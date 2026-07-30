# minisforum-v3-tabletmode

Automatic tablet mode and working screen auto-rotation on the **Minisforum V3**
(Linux, tested on Arch/EndeavourOS with KDE Plasma 6 on Wayland, kernel 7.1).

Two independent problems are fixed here.

## 1. Auto-rotate never starts

The V3's LSM6DS3TR-C accelerometer (ACPI `SMO8B30`) has **no IIO trigger** — its
interrupt line isn't described in the firmware's ACPI tables. Recent kernels made
`st_lsm6dsx` advertise buffer support anyway, so udev tags the device as both
`iio-poll-accel` and `iio-buffer-accel`, and `iio-sensor-proxy` prefers the
buffered driver. It enables the ring buffer and then blocks forever waiting for
samples that can never arrive. Because the daemon defers its D-Bus reply until
the first reading, `ClaimAccelerometer` never returns and the compositor never
receives an orientation.

Symptoms: `AccelerometerOrientation` stuck at `"undefined"` while
`HasAccelerometer` is `true`; `ClaimLight` returns instantly but
`ClaimAccelerometer` hangs; `buffer/enable` flips to `1` during a claim; the
journal logs `Could not find trigger name associated with ...`. Raw
`in_accel_*_raw` sysfs reads work fine throughout — the hardware is healthy.

`81-iio-sensor-proxy-force-poll.rules` reassigns the udev property so only the
polling driver remains.

## 2. Folding the keyboard doesn't enable tablet mode

The V3 is a *detachable*. Detaching the keyboard puts the compositor into tablet
mode natively, but folding the folio behind the screen does not — and none of the
usual signals react to it:

| Checked | Result when folded |
| --- | --- |
| `SW_TABLET_MODE` on `gpio-keys` | never leaves `0` |
| USB presence / power / interfaces | unchanged |
| evdev, all devices, all event types | nothing |
| EC RAM (256 B, controlled A/B/A) | drift only |
| all 215 AMD GPIO pin levels | completely stable |
| udev / kernel events | none |

The folio does know, though: it stops sending keystrokes entirely when folded
(69 key reports open vs **0** folded). It announces the change as a **vendor HID
report on its own interface**, which the kernel maps to no input event at all:

```
02 00 00 40   folded behind the screen
02 00 00 08   opened to laptop position
```

Edge-triggered, silent while idle. Raw HID is the only place it is visible, which
is why every layer above it is blind to the fold.

`v3-tabletmode-daemon` watches for those reports and republishes them as a
virtual `SW_TABLET_MODE` switch via uinput, which compositors do understand.

## Behaviour

| Position | Rotation |
| --- | --- |
| Keyboard folded behind screen | enabled (this package) |
| Keyboard detached | enabled (native) |
| Normal laptop position | disabled |

When not asserting "folded" the daemon destroys its virtual switch instead of
reporting `0`, so it can never contradict the native detach detection. The folio
is located by `HID_ID` rather than by `hidrawN`, so renumbering across reboots is
harmless, and the nodes are reopened on hotplug.

## Install

```sh
makepkg -si
systemctl enable --now v3-tabletmode.service
```

Then set **Auto-rotate → In tablet mode** in *System Settings → Display &
Monitor*. Note that KWin rewrites `~/.config/kwinoutputconfig.json` from memory
on shutdown, so edit that setting through the GUI, not the file.

For the on-screen keyboard, install `maliit-keyboard` and set
`VirtualKeyboardEnabled=true` under `[Wayland]` in `~/.config/kwinrc`.

## Manual override

Automatic detection is the default; the override exists as a fallback.

```sh
v3-tabletmode auto      # follow the folio (default)
v3-tabletmode on        # force tablet mode
v3-tabletmode off       # force it off
v3-tabletmode toggle    # flip between auto and on
v3-tabletmode status
```

A *Toggle Tablet Mode* launcher is installed for touch use.

## Hardware notes

- Keyboard folio: USB `05af:326a`, `HID_ID=0003:000005AF:0000326A`
- Accelerometer: `lsm6ds3tr-c_accel`, ACPI `SMO8B30`
- The report bytes above were captured on one unit; if yours differs, dump
  `/dev/hidraw*` while folding and adjust `REPORT_FOLDED` / `REPORT_OPENED`.

## See also

- [`minisforum-v3-dsdt`](https://aur.archlinux.org/packages/minisforum-v3-dsdt) —
  DSDT patch that exposes the accelerometer at all
- [`minisforum-v3-accelerometer`](https://aur.archlinux.org/packages/minisforum-v3-accelerometer) —
  correct mount matrix

## License

MIT

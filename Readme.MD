# spearhead-led

Control the LED light bar of the **1MORE Spearhead VR** (USB `0d8c:0172`) on Linux, without the vendor's Windows-only software.

Audio, the microphone, the volume wheel and the mic-mute switch already work on Linux through the standard `snd-usb-audio` driver. The only thing that needs the vendor software is the light bar, and this tool talks to it directly. The protocol was reverse-engineered from USB traffic; see [PROTOCOL.md](PROTOCOL.md).

> **Read this first.** The tool writes to a vendor-specific HID channel of a C-Media chip whose protocol is not publicly documented. It was developed and tested on **one single unit** of the Spearhead VR. Use it at your own risk. See [Safety](#safety).

## What it can do

| Mode | Options |
|---|---|
| `off` | none |
| `constant` | any RGB colour |
| `breathe` | any RGB colour, speed 10 (fast) to 100 (slow) |
| `heartbeat` | any RGB colour (the speed setting of the vendor panel does not appear on the wire for this mode) |

There is a small Tk window and a command-line interface.

## Status and limitations

- Tested on Arch Linux with a single unit: USB `0d8c:0172`, `bcdDevice 1.16`, vendor driver package "H1005 SpearheadVR DRIVER 2.14 WHQL".
- Verified on hardware: all four modes, colours (red, green, pink, cyan, blue), breathe speed.
- **Not verified:** whether the setting survives unplugging the headphones, how `heartbeat` reacts to audio, and anything about other models or firmware revisions.
- The vendor panel also has an equalizer, a virtual 7.1 mode and other settings. Those are **not** handled here. On Linux, equalizer and virtual surround are usually done in software (PipeWire filter-chain, EasyEffects).
- Not affiliated with 1MORE or C-Media. No vendor software or firmware is included or needed.

## Install

Requires Python 3 and, for the window only, Tk (`tk` on Arch, `python3-tk` on Debian/Ubuntu).

```bash
sudo install -m755 spearhead-led /usr/local/bin/spearhead-led
```

Optional: let your user write to the device without root.

```bash
sudo install -m644 99-spearhead.rules /etc/udev/rules.d/99-spearhead.rules
sudo udevadm control --reload && sudo udevadm trigger
# then replug the headphones and log out/in
```

Without the udev rule the window asks for your administrator password and retries the write through `sudo`. The password is kept in memory only while the window is open.

## Usage

```bash
spearhead-led                               # open the window
spearhead-led constant  00ff00 --send
spearhead-led breathe   ff0000 50 --send    # speed 10..100
spearhead-led heartbeat 00c8ff --send
spearhead-led off --send
```

The command line does a **dry run** and writes nothing unless you pass `--send`.

## Safety

Before every write the tool checks that the device matches the one the protocol was captured from:

1. USB ID is `0d8c:0172`;
2. the USB product string is exactly `1MORE Spearhead VR` (so the VRX model is rejected);
3. the HID report descriptor is byte-identical to the one from the tested unit.

If anything differs, the tool refuses, explains what did not match and exits with code 3. To bypass the check you must pass `--i-know-that-this-will-ruin-my-headphones`. That flag exists on purpose: sending data that was captured from another device into an unknown firmware may misconfigure it. Do not use it unless you understand the protocol and accept that risk.

The tool only replays the frames the vendor panel was observed to send for lighting. It never touches firmware-update functionality.

## How it works

The panel sends a short sequence of 16-byte HID output reports (report ID `0xFF`) through the control endpoint: a start frame, a header that selects the mode, a 128-entry RGB brightness ramp, and an end frame. Details, the exact frame layout and how it was captured are in [PROTOCOL.md](PROTOCOL.md).

## Contributing

Captures from other units or models are very welcome, especially: other `bcdDevice` revisions, the VRX, and whether lighting settings persist after a replug. Open an issue with `lsusb -v -d 0d8c:0172` output and, if you can, a usbmon capture of the vendor panel changing the light settings.

## License

MIT. See `LICENSE`.

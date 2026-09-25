# Update or restore SD Card for Yaw 3

This stock image has a more recent version of RaspberryPi OS, and I had to redevelop portions of the Yaw software to work with changes in the OS, I also added some performance improvements!

If you want to modify the chair to support [Wifi6E then skip to here](wifi6e.md)

Prior to doing this you should execute the following on the chair:

SSH into the device, substitute with the IP of your simulator on your network:

`ssh pi@192.168.0.120` The password is pi

Then execute the following command and store the output:
`sed -n 's/^License=//p' /usr/local/etc/virtualhere/config.ini`

The output is your VirtualHere license, save this! you will need this if your SD ever gets corrupted and you need to reflash the SD, without it you will probably need to re-purchase VirtualHere, as long as you have this you can relicense VH on the same physical Raspberry Pi.

## Updated RaspberyPi OS Stock Image

Flash the following image to a high quality <a href="https://amzn.to/4cguMio" target="_blank">32GB Micro SD card</a> using <a target="_blank" href="https://etcher.balena.io/">Balena Etcher</a>

<a target="_blank" href="https://www.dropbox.com/scl/fi/1t0k8cg5xr9hvy8ix8unp/image_2026-09-25-yaw3-standard.img.xz?rlkey=a2849wc766jmmfyypcbkzudgp&st=l7b2jd1w&e=1&dl=1">2026-09-25 YawIII-RaspberryPIOS12-clean-optimized</a>

## 2026-09-26: 10 hours of development and testing

- Fixed multiple issues that predate my original image
-Reduced SD image sizes by 50%

## Ride behavior & safety

- **Per-axis deadband compensation** at the motor-command stage — the constant buzz/vibration while seated and stationary is gone; motors no longer idle at their "will it move?" threshold. Ride tracking verified unaffected by A/B test.
- **Calibration now always captures yaw**, regardless of what the app requests — Calibrate means "wherever I'm facing now is center." Previously the missing yaw reference made every Start rotate the chair ~45° sideways after a reflash.

## Crash & leak fixes

- `setSpeed` memory leak fixed (~104 MB/day from the control loop) — long sessions can't run the rig out of memory.
- Web-interface socket leak + two thread-safety races fixed (app peer list, last-packet timestamp) — removes rare crashes/corruption when app, game, and debugger connect concurrently.
- TCP control socket binds with `reuseAddress` — the server reliably survives quick restarts instead of crash-looping on an in-use port.
- Exception type mismatch in the updater fixed — network failures handled cleanly instead of escaping as uncaught errors.
- Wi-Fi signal-info memory leak fixed (leaked on *every* call).
- Angle normalization fixed for values below −180° — rare large roll/yaw swings can no longer produce wrong motor commands.
- Ride-recording zipper follows the configured CSV location — recording to permanent storage no longer crashes after every ride.
- Service stop timeout capped at 10 s (was 90 s) — restarts and updates no longer hang for minutes in "deactivating".

## Performance (measured, not guessed)

- **Allocation-free status send path**: status messages built with `snprintf` into persistent buffers; JSON only for developer tools. Memory-allocation churn with the app connected dropped ~40,800 → ~410 per second (−99%), idle down 22%. Output proven byte-identical on the wire against golden captures.
- Control loop no longer copies PID parameters every tick — cheaper 100 Hz loop, less jitter.
- Telemetry CSV streams stay open across ticks — no open/close syscall storm while recording.

## Device bring-up

- GPIO chip discovered structurally instead of by number — works regardless of kernel chip numbering.
- Kernel console & serial login kept off the RS422 bus UART — console spew can no longer corrupt motor-bus communication.
- Hostname follows the rig name (case preserved, defaults to YAWIII) — rig shows up on the network under its proper name.
- BLE advert payload fix + provisioning repairs — phone can discover and provision the rig reliably.
- Self-updater neutralized (it pointed at the **Yaw 2** channel — bricked arm64 rigs on first boot) and disabled by default.
- Startup script's gpiod v2 fallback repaired — boot works on newer libgpiod too.

## One-line version

The rig now calibrates correctly, holds still when parked, shouldn't enter flat spin, survives flashes without going feral, leaks nothing, allocates 99% less while streaming, boots under its own name, and the whole build→flash→measure→deploy pipeline is reproducible from a Mac.

I don't have a job/income at the moment your support appreciated 
<div align="center">
<script type="text/javascript" src="https://cdnjs.buymeacoffee.com/1.0.0/button.prod.min.js" data-name="bmc-button" data-slug="ItsVRK" data-color="#FFDD00" data-emoji="" data-font="Cookie" data-text="Buy me a coffee" data-outline-color="#000000" data-font-color="#000000" data-coffee-color="#ffffff" ></script>
</div>

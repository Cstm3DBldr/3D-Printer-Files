# Full Home Backup — SC350 — 2026-04-19
**Voron StealthChanger 350×350×400 — 6 Toolhead Build**

Taken: 2026-04-19 at 23:02  
State: Working autoloader, fixed screen bug (screen is now GPU driven / correct orientation)

---

## What Is and Is NOT Included

Python virtual environments (klippy-env, moonraker-env, .KlipperScreen-env),
log files, and gcode files are excluded — they are either re-creatable via KIAUH
or too large to store in git. Everything else from /home/pi is included.

---

## Folder Contents

### `printer_data/config/`
All Klipper configuration files exactly as they live on the printer.
Restore path: `/home/pi/printer_data/config/`

Includes:
- `printer.cfg` — main printer config
- `macros.cfg`, `homing.cfg`, `crowsnest.conf`, `moonraker.conf`, `timelapse.cfg`
- `KlipperScreen.conf`
- `Toolchanger/` — toolchanger.cfg, calibrate-offsets.cfg, tool_detection.cfg, T0–T5
- `stealth-autoloader/` — hardware.cfg, macros.cfg, pin_aliases.cfg, parameters.cfg, variables.cfg

### `printer_data/database/`
Moonraker database (print history, saved settings, variables).
Restore path: `/home/pi/printer_data/database/`

### `printer_data/systemd/`
Environment files for all Klipper-related services.
Restore path: `/home/pi/printer_data/systemd/`

### `stealth-autoloader/`
Full Stealth Autoloader repo content (minus .git). Contains Python backend extras
and hardware config source files.
Restore path: `/home/pi/stealth-autoloader/`

After restoring, re-create symlinks:
```bash
ln -sf ~/stealth-autoloader/klipper/extras/stealth_autoloader.py ~/klipper/klippy/extras/
ln -sf ~/stealth-autoloader/klipper/extras/sa_encoder.py ~/klipper/klippy/extras/
ln -sf ~/stealth-autoloader/klipper/extras/sa_sequences.py ~/klipper/klippy/extras/
ln -sf ~/stealth-autoloader/klipper/extras/sa_motion.py ~/klipper/klippy/extras/
ln -sf ~/stealth-autoloader/klipper/extras/sa_calibration.py ~/klipper/klippy/extras/
```

### `klipper-toolchanger/klipper/extras/`
Custom Klipper Python extras for the StealthChanger toolchanger system.
These must be symlinked into klipper after install.
Restore path: `/home/pi/klipper-toolchanger/klipper/extras/`

Key files: tool.py, toolchanger.py, tool_probe.py, tool_probe_endstop.py,
tools_calibrate.py, multi_fan.py, manual_rail.py, rounded_path.py, bed_thermal_adjust.py

### `tapchanger/Klipper/`
Tapchanger README reference.
Restore path: `/home/pi/tapchanger/Klipper/`

### `KlipperScreen-config/`
Custom KlipperScreen menu configuration files.
Restore path: `/home/pi/KlipperScreen/config/`

### `System-Services/`
Systemd service unit files for all printer services.
Restore path: `/etc/systemd/system/`

Files: klipper.service, moonraker.service, KlipperScreen.service,
crowsnest.service, sonar.service

After restoring:
```bash
sudo systemctl daemon-reload
sudo systemctl enable klipper moonraker KlipperScreen crowsnest sonar
```

### `Screen-Fix/`
**See Screen-Fix/README.MD for full restore instructions.**

Files that fixed the BTT M8P CM4 + PITFT43 DSI display:
- `boot_config.txt` → `/boot/config.txt` (GPU driver + rotation)
- `DISP0/dt-blob.bin` → `/boot/dt-blob.bin` (CM4 DSI device tree blob)
- `KlipperScreen.service` → `/etc/systemd/system/KlipperScreen.service` (BACKEND=X)

### `CM4_dt_blob_Source/`
Source DTS files and compiled dt-blob.bin for both DISP0 and DISP1.
Also includes CM4_dt_blob_Source.zip (original download).
Stored for reference if dt-blob.bin ever needs to be recompiled.

### `dotfiles/`
Pi user dotfiles: .bashrc, .profile, .gitconfig, .kiauh.ini
Restore path: `/home/pi/`

---

## Recovery Order (full rebuild)

1. Flash MainsailOS to SD/eMMC
2. Install software via KIAUH: Klipper, Moonraker, KlipperScreen, Crowsnest, Sonar
3. Restore `printer_data/config/` → `/home/pi/printer_data/config/`
4. Restore `printer_data/database/` → `/home/pi/printer_data/database/`
5. Restore `printer_data/systemd/` → `/home/pi/printer_data/systemd/`
6. Clone stealth-autoloader repo or restore from `stealth-autoloader/`, create symlinks
7. Clone klipper-toolchanger repo or restore extras, create symlinks
8. Restore `KlipperScreen-config/` → `/home/pi/KlipperScreen/config/`
9. Follow Screen-Fix/README.MD for display setup
10. `sudo systemctl daemon-reload && sudo reboot`

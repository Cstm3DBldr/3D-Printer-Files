# Screen Fix — BTT M8P CM4 + PITFT43 DSI Touchscreen
## GPU-Driven Rendering + Correct Orientation

This folder contains the files that fixed two issues:
1. Screen rendering was software-driven (slow/glitchy) — fixed by enabling vc4-fkms-v3d GPU driver
2. Screen was rotated 180° — fixed by display_lcd_rotate=2 in config.txt

---

## Files and Where They Go

### `boot_config.txt`
**Restore to:** `/boot/config.txt`

The critical lines for the screen fix are:
```
dtoverlay=vc4-fkms-v3d      ← enables GPU driver (NOT vc4-kms-v3d — use fkms)
max_framebuffers=2           ← required for DSI display
display_lcd_rotate=2         ← rotates screen 180° (0=normal, 1=90°, 2=180°, 3=270°)
dtoverlay=rpi-ft5406         ← enables touch input on the DSI display
```

**WARNING:** Do NOT change `vc4-fkms-v3d` to `vc4-kms-v3d` — the full KMS driver
breaks the DSI touchscreen on CM4. The fake-KMS (fkms) driver is required.

---

### `DISP0/dt-blob.bin`
**Restore to:** `/boot/dt-blob.bin`  
Copy command: `sudo cp dt-blob.bin /boot/dt-blob.bin`

This is the compiled device tree blob that maps the CM4's DSI0 port to the
PITFT43 display. Without this file the display may not initialize correctly on
CM4. Source DTS files are in CM4_dt_blob_Source/ if you need to recompile.

---

### `KlipperScreen.service`
**Restore to:** `/etc/systemd/system/KlipperScreen.service`  
Copy command: `sudo cp KlipperScreen.service /etc/systemd/system/`  
Then reload: `sudo systemctl daemon-reload && sudo systemctl restart KlipperScreen`

The key line is `BACKEND=X` in the Environment — this forces X11 (not Wayland/Cage)
which is required for the vc4-fkms-v3d driver to render KlipperScreen correctly.

---

## Full Restore Sequence (if rebuilding from scratch)

```bash
# 1. Copy boot config
sudo cp boot_config.txt /boot/config.txt

# 2. Copy dt-blob
sudo cp DISP0/dt-blob.bin /boot/dt-blob.bin

# 3. Copy KlipperScreen service
sudo cp KlipperScreen.service /etc/systemd/system/KlipperScreen.service
sudo systemctl daemon-reload
sudo systemctl enable KlipperScreen

# 4. Reboot
sudo reboot
```

After reboot the screen should display correctly with GPU rendering and correct orientation.

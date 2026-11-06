# Discord on Oracle Cloud ARM64

Run the Linux Discord desktop app on an Oracle Cloud Ampere A1 VM (Ubuntu 24.04 ARM64). The Linux client is x86_64, so FEX-Emu is used to run it inside an x86_64 Ubuntu RootFS.

## Setup

Install the desktop packages:

```bash
sudo apt install -y xfce4 xfce4-goodies xrdp xorgxrdp dbus-x11 squashfs-tools
```

Install FEX-Emu and get an Ubuntu 24.04 x86_64 RootFS image. The image used here was:

```
/opt/fex-emu/RootFS/Ubuntu_24_04.sqsh
```

Extract it to a writable directory:

```
sudo unsquashfs -no-progress \
  -d /opt/fex-emu/RootFS/Ubuntu_24_04-writable \
  /opt/fex-emu/RootFS/Ubuntu_24_04.sqsh
```

Discord also needs the AMD64 `libatomic1` library inside the guest RootFS. Download the Ubuntu 24.04 amd64 `.deb` (not the ARM64 host package), then extract it into the RootFS. The package used during this setup was `14.2.0-4ubuntu2~24.04.1`:

```
ROOTFS=/opt/fex-emu/RootFS/Ubuntu_24_04-writable
sudo dpkg-deb -x /tmp/libatomic1_14.2.0-4ubuntu2_24.04.1_amd64.deb "$ROOTFS"
```

This extracts the library files into the guest filesystem; it does not install the package in the guest package database.

The Discord x86_64 `.deb` was unpacked rather than installed on the ARM64 host:

```
mkdir -p ~/discord-amd64/root
dpkg-deb -x ~/discord-amd64/discord.deb ~/discord-amd64/root
```

The versioned app directory used by the launch command is `/home/streamdesk/config/discord/app-1.0.168/`. Make sure the extracted Discord app files are in that directory and that the `Discord` executable is present there.

## Connect to the desktop

In `/etc/xrdp/xrdp.ini`, under `[Globals]`, set XRDP to listen on localhost only:

```
port=tcp://127.0.0.1:3389
```

Restart XRDP and check that it is listening on `127.0.0.1:3389`, not `0.0.0.0:3389`:

```
sudo systemctl restart xrdp
sudo ss -ltnp | grep 3389
```

From Windows, open an SSH tunnel to the VM:

```
ssh -i "PATH_TO_PRIVATE_KEY" -N -L 3390:127.0.0.1:3389 ubuntu@YOUR_VM_PUBLIC_IP
```

Leave that terminal open, then open Remote Desktop and connect to `localhost:3390`. Log in to the XFCE session as `streamdesk`.

The desktop was set to 1280x720 and XFCE compositing was turned off.

## Start Discord

Run this from a terminal inside the `streamdesk` desktop session:

```
FEX_ROOTFS=/opt/fex-emu/RootFS/Ubuntu_24_04-writable \
FEX /home/streamdesk/config/discord/app-1.0.168/Discord \
  --disable-setuid-sandbox \
  --enable-gpu \
  --use-gl=angle \
  --use-angle=swiftshader \
  --disable-site-isolation-trials
```

`--disable-setuid-sandbox` was needed to start Electron in this setup. `--disable-site-isolation-trials` was used to get past the login/CAPTCHA issue at the time; it may not be needed with a different Discord build.

## Notes

- This is x86_64 emulation on ARM64, not a native ARM build of Discord.
- The VM has no physical GPU. SwiftShader handles graphics in software.
- Streaming was low-FPS on a 2 OCPU VM. Lowering the desktop resolution helped, but did not fix it completely.
- Audio may need more setup. PulseAudio showed `auto_null` devices in this configuration.
- The paths and launch flags above are for Discord 1.0.168. Other versions may need changes.
- Log in, join the voice channel and start the stream manually.

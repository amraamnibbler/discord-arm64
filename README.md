# Discord on Oracle ARM64

Run the Linux Discord desktop app on an Oracle Cloud Ampere A1 VM (Ubuntu 24.04 ARM64) using FEX-Emu. Discord's Linux app is x86_64, so it runs through emulation rather than as a native ARM app.

## Setup

- Create an Ubuntu 24.04 ARM64 VM.
- Install FEX-Emu and an Ubuntu 24.04 x86_64 RootFS.
- Extract the RootFS to a writable folder. The path used below is `/opt/fex-emu/RootFS/Ubuntu_24_04-writable`.
- Make sure the guest RootFS has the AMD64 version of `libatomic1` installed. The Discord app needs `libatomic.so.1`.
- Install the Linux Discord client in `/home/streamdesk/config/discord/`. The version used here was 1.0.168.
- Set up XFCE and XRDP for the desktop session.

## Start Discord

Run it from the desktop session:

```bash
FEX_ROOTFS=/opt/fex-emu/RootFS/Ubuntu_24_04-writable \
FEX /home/streamdesk/config/discord/app-1.0.168/Discord \
  --disable-setuid-sandbox \
  --enable-gpu \
  --use-gl=angle \
  --use-angle=swiftshader \
  --disable-site-isolation-trials
```

The sandbox flag is needed for Electron to start in this setup. The site-isolation flag was used to get through the login/CAPTCHA issue.

## Connect to the desktop

Keep XRDP listening locally and connect through an SSH tunnel. From Windows:

```powershell
ssh -i "PATH_TO_PRIVATE_KEY" -N -L 3390:127.0.0.1:3389 ubuntu@YOUR_VM_PUBLIC_IP
```

Leave that terminal open, then open Remote Desktop and connect to `localhost:3390`.

The desktop was set to 1280x720 and XFCE compositing was turned off.

## Notes

- FEX is emulating x86_64 on ARM64; this is not a native ARM build of Discord.
- There is no physical GPU on the VM. SwiftShader handles graphics in software.
- Streaming was low-FPS on a 2 OCPU VM. Dropping the desktop resolution helped, but did not fix it completely.
- Audio may need more setup. PulseAudio showed `auto_null` devices in this configuration.
- The commands and paths above are for Discord 1.0.168. Other versions may need changes.
- Log in, join the voice channel and start the stream manually.

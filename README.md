# Ad-Player

```markdown
# 📺 Raspberry Pi CLI Ad-Player (No-GUI Ultimate Guide)

A lightweight, rock-solid, and completely unattended digital signage / advertisement player solution tailored for **Raspberry Pi OS Lite (Debian 13 Bookworm)**. 

By utilizing low-level system services and the kernel framebuffer, this setup bypasses heavy desktop environments to boot **directly into a fullscreen image loop within seconds**—completely free of mouse cursors, login prompts, desktop panels, or screen blanking issues.

---

## 🛠️ Prerequisites

* **Hardware:** Any Raspberry Pi connected to a monitor/TV via HDMI.
* **OS:** Raspberry Pi OS Lite (No-GUI version, Bookworm or newer).
* **Default Username:** `user`
* **Media:** An advertising image named `ads.jpg` placed directly in the `/home/user/` directory.
  > 💡 *Verify the file path via SSH or terminal using: `ls /home/user/ads.jpg`*

---

## 🚀 Step-by-Step Implementation

### Step 1: Install the Framebuffer Image Viewer (`fbi`)
`fbi` is a dedicated CLI tool that draws image pixels directly onto the system's screen framebuffer without requiring an X11/Wayland desktop UI server.

```bash
sudo apt update
sudo apt install fbi -y

```

### Step 2: Configure Password-Free `sudo` Privileges

To allow the startup script to call the administrative framebuffer layout smoothly without hanging at a `[sudo] password for user:` prompt, you must grant passwordless privileges:

1. Open the secure sudoers editor:
```bash
sudo visudo

```


2. Scroll to the **very bottom** of the file and append the following line:
```text
user ALL=(ALL) NOPASSWD: ALL

```


3. Save and exit (`Ctrl + O` ➔ `Enter` ➔ `Ctrl + X`).

### Step 3: Enforce Systemd Console Autologin

We will manually override the default low-level Getty system service to bypass the interactive username/password terminal prompt on the main physical display (`tty1`):

1. Create the systemd service drop-in configuration directory and inject the autologin parameters with this single string command:
```bash
sudo mkdir -p /etc/systemd/system/getty@tty1.service.d/ && echo -e "[Service]\nExecStart=\nExecStart=-/sbin/agetty --autologin user --noclear %I \$TERM" | sudo tee /etc/systemd/system/getty@tty1.service.d/autologin.conf

```


2. Reload the systemd daemon manager and lock the service behavior:
```bash
sudo systemctl daemon-reload
sudo systemctl enable getty@tty1.service

```



### Step 4: Configure Boot Display Script & Anti-Screen Blanking

We harness the user's `.bashrc` execution file to kill power-saving triggers and spawn our media layout the precise millisecond the local terminal auto-login phase fires.

1. Open the user profile configuration:
```bash
nano /home/user/.bashrc

```


2. Scroll to the **absolute bottom** of the file and paste this standard logic script block:
```bash
# Ensure this only triggers on the local physical screen (tty1), preventing SSH glitches
if [ "$(tty)" = "/dev/tty1" ]; then
    # Completely disable terminal blanking, energy savers, and panel power-downs
    setterm -blank 0 -powersave off -powerdown 0

    # Launch fbi: force fullscreen (-T 1), auto-zoom (-a), and hide software UI (-noverbose)
    sudo fbi -d /dev/fb0 -T 1 -noverbose -a /home/user/ads.jpg
fi

```


3. Save and exit (`Ctrl + O` ➔ `Enter` ➔ `Ctrl + X`).

### Step 5: Clean up the Boot Splash (Optional)

To hide standard kernel log prints and system messages during startup for a seamless commercial look:

1. Open the boot loader core command-line file:
```bash
sudo nano /boot/firmware/cmdline.txt

```


*(Note: On legacy Debian deployments, if this path is absent, edit `/boot/cmdline.txt` instead)*
2. Append these arguments at the **very end of that single long line** (Do **NOT** press enter to create a new line, use a single space separator):
```text
quiet splash logo.nologo vt.global_cursor_default=0

```


3. Save and exit (`Ctrl + O` ➔ `Enter` ➔ `Ctrl + X`).

---

## 🏁 Deployment Test

Unplug your configuration mouse, verify that only the target monitor and a setup keyboard are connected, and issue a hard restart command:

```bash
sudo reboot

```

**Expected Behavior:** Your Raspberry Pi will cycle boot, skip diagnostic print screens, automatically log into the `user` profile without credentials, and instantly throw your `ads.jpg` asset into a 100% full-screen presentation.

---

## 💡 Maintenance & Extensions

### 🛑 Exiting the Playback Loop

Need to reconfigure or modify files locally? Simply connect a keyboard to the machine and tap `Q` or `Esc`. The `fbi` display instance will terminate safely, dropping you back into your standard `user@display1:~ $` terminal prompt.

### 🖼️ Updating the Display Content

To refresh the advertisement banner, close the stream or log in via SSH, replace the current image block located at `/home/user/ads.jpg` with your new graphic asset, and reboot.

### 🔄 Enabling a Multi-Image Slideshow Loop

If your deployment demands cycling through multiple advertising graphics sequentially:

1. Create a dedicated image warehouse directory:
```bash
mkdir /home/user/images

```


2. Dump all your target image files (`.jpg`, `.png`, etc.) directly inside that directory.
3. Open your `/home/user/.bashrc` profile, head back to the bottom, and replace the solitary `fbi` launch statement with this directory parsing command:
```bash
sudo fbi -d /dev/fb0 -T 1 -noverbose -a -t 5 /home/user/images/*

```


> 💡 *The `-t 5` flag dictates the slide transition tempo interval in seconds. Tweak this value to match your desired duration.*



```

```

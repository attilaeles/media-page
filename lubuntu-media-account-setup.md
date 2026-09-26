# Lubuntu Media Account Setup

A lightweight dedicated media/kiosk account for a Linux tablet or
low-resource PC.

## Goal

Create a separate Linux user named `media` that:

-   is isolated from the normal development/admin account;
-   has no `sudo` privileges;
-   uses its own Firefox profile and cookies;
-   stays logged in to streaming providers such as Netflix, Max, Prime
    Video, and YouTube;
-   automatically starts Firefox in fullscreen/kiosk mode;
-   can optionally log in automatically when the device boots;
-   keeps the normal Lubuntu desktop available when needed.

This setup is particularly suitable for low-resource hardware such as a
Surface Go running Lubuntu/LXQt.

## Architecture

``` text
Surface Go / Lubuntu
|
+-- Normal user
|   +-- password protected
|   +-- sudo/admin access
|   +-- Python / Jupyter
|   +-- GCP / BigQuery
|   +-- SDR tools
|   +-- personal files and credentials
|
+-- media user
    +-- no sudo access
    +-- separate Firefox profile
    +-- streaming-service sessions
    +-- Firefox kiosk/fullscreen startup
```

## 1. Create the Media User

Log in to the normal administrator account and open a terminal.

``` bash
sudo adduser media
```

Follow the prompts and assign a password. Do **not** add the account to
`sudo`.

Check its groups:

``` bash
groups media
```

If `sudo` appears unexpectedly:

``` bash
sudo deluser media sudo
```

## 2. Log In to the Media Account Once

Log out of the normal account and select `media` at the Lubuntu login
screen.

This creates `/home/media/` and a separate Firefox profile.

## 3. Configure Firefox

Launch Firefox as `media`.

Recommended configuration:

-   no development extensions;
-   no unnecessary extensions;
-   do not automatically delete cookies/site data;
-   avoid Firefox Sync with your main personal browser unless desired.

### Enable DRM

Open:

``` text
Settings -> General -> Digital Rights Management (DRM) Content
```

Enable **Play DRM-controlled content**.

Firefox can then use Widevine DRM for supported streaming services.

## 4. Log In to Streaming Providers

Sign in normally to Netflix, Max, Prime Video, YouTube, Spotify Web,
etc.

You normally do **not** need Firefox to save the actual passwords.
Persistent provider cookies/tokens can keep the device signed in.
Providers may periodically require reauthentication.

## 5. Test Streaming Before Kiosk Mode

Verify:

1.  DRM playback;
2.  audio;
3.  fullscreen video;
4.  touchscreen controls;
5.  Wi-Fi stability.

Do not automate kiosk startup until ordinary playback works.

## 6. Test Firefox Kiosk Mode

``` bash
firefox --kiosk
```

Or launch Netflix directly:

``` bash
firefox --kiosk https://www.netflix.com/
```

## 7. Optional Media Homepage

A local homepage can provide large touch-friendly links to all
providers.

``` bash
mkdir -p ~/media-home
```

Create:

``` text
~/media-home/index.html
```

Then launch:

``` bash
firefox --kiosk file:///home/media/media-home/index.html
```

## 8. Start Firefox Automatically

While logged in as `media`:

``` text
Preferences -> LXQt Settings -> Session Settings -> Autostart
```

Add:

``` text
Name: Media Browser
Command: firefox --kiosk
```

Or:

``` text
Name: Media Browser
Command: firefox --kiosk file:///home/media/media-home/index.html
```

Log out and back in to test.

## 9. Optional Automatic Login

Auto-login gives:

``` text
Power on -> Lubuntu -> media -> Firefox kiosk
```

**Security:** anyone with physical access can access authenticated
streaming services. Keep `media` unprivileged and free of work
credentials.

Confirm the display manager:

``` bash
cat /etc/X11/default-display-manager
```

If it is SDDM, inspect available sessions:

``` bash
ls /usr/share/xsessions/
```

Create/edit:

``` bash
sudo nano /etc/sddm.conf.d/autologin.conf
```

Example:

``` ini
[Autologin]
User=media
Session=Lubuntu
```

The session name can vary by Lubuntu release; use the appropriate
installed LXQt/Lubuntu session name.

Reboot:

``` bash
sudo reboot
```

## 10. Keep the Normal Account Available

Recommended design:

``` text
Surface Go
|
+-- media
|   +-- no sudo
|   +-- Firefox kiosk
|   +-- streaming services
|
+-- normal user
    +-- password
    +-- full Lubuntu desktop
    +-- Python/Jupyter
    +-- GCP/BigQuery
    +-- RTL-SDR
```

Exit Firefox/log out of `media` to switch to the normal account.

## 11. Optional Local Video Playback

Install lightweight `mpv` from the administrator account:

``` bash
sudo apt install mpv
```

Example:

``` bash
mpv /path/to/video.mp4
```

Use Firefox rather than VLC/mpv for DRM streaming services.

## 12. Resource-Saving Recommendations

For 4 GB RAM:

-   keep one browser open;
-   minimize extensions and tabs;
-   do not run Jupyter/SDR/development tools in the media session;
-   keep zram enabled;
-   keep earlyoom enabled if configured;
-   prefer hardware video decoding where supported.

Applications installed system-wide are shared between users. The second
user mainly adds its own profile, cache, settings, and files.

## 13. Credentials and Privacy

Good:

``` text
media
+-- Netflix session
+-- Max session
+-- Prime Video session
+-- media-only YouTube session
```

Avoid:

``` text
media
+-- GCP credentials       NO
+-- SSH private keys      NO
+-- API keys              NO
+-- sensitive work files  NO
+-- admin credentials     NO
```

## 14. Disable Auto-Login

From the administrator account:

``` bash
sudo rm /etc/sddm.conf.d/autologin.conf
sudo reboot
```

## 15. Remove Firefox Autostart

As `media`:

``` text
Preferences -> LXQt Settings -> Session Settings -> Autostart
```

Remove/disable `Media Browser`.

## 16. Remove the Media User

Preserve its home directory:

``` bash
sudo deluser media
```

Remove account and home directory:

``` bash
sudo deluser --remove-home media
```

Only use the second command when `/home/media` contains nothing you
need.

## Final Recommended Configuration

``` text
Surface Go
|
+-- Lubuntu / LXQt
+-- zram
+-- earlyoom
|
+-- normal user
|   +-- password protected
|   +-- sudo
|   +-- Python / Jupyter
|   +-- GCP
|   +-- RTL-SDR
|
+-- media
    +-- no sudo
    +-- optional auto-login
    +-- Firefox auto-start
    +-- Firefox kiosk mode
    +-- DRM enabled
    +-- persistent streaming sessions
    +-- optional local media homepage
```

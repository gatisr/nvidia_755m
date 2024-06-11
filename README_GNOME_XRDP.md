Here is what I found that works well for configuring XRDP with Gnome without having to modify global /etc files. When properly configured, it runs reasonably fast, although is not as zippy as Xfe.

Install xrdp and gnome-session:

```bash
sudo apt update
sudo apt install xrdp gnome-session
```

Allow log in after xrdp setup:

```bash
sudo adduser xrdp ssl-cert
```

Specify Gnome for your remote session:

```bash
echo "gnome-session" | tee ~/.xsession
```

Then set your ~/.xsessionrc file using:

```bash
echo "export XAUTHORITY=${HOME}/.Xauthority" | tee ~/.xsessionrc
echo "export GNOME_SHELL_SESSION_MODE=ubuntu" | tee -a ~/.xsessionrc
echo "export XDG_CONFIG_DIRS=/etc/xdg/xdg-ubuntu:/etc/xdg" | tee -a ~/.xsessionrc
```

Here the XAUTHORITY line solves the Display error when opening Firefox and other apps that rely on root privileges.

Optionally, if you want the regular Gnome desktop with vertical menu on the side rather than Activities menu, do this:

```bash
echo "export XDG_CURRENT_DESKTOP=ubuntu:GNOME" | tee -a ~/.xsessionrc
```

Finally do this:

```bash
sudo systemctl restart xrdp
```

Then log out and connect with a Remote Desktop client.

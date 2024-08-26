---
title: Launching GNOME applications on SSH session
tags:
  - GNOME
id: '1359'
categories:
  - - Linux
date: 2021-06-22 22:01:26
---

TL;DR:

Add these 2 lines to your `.bashrc`:

export $(dbus-launch --exit-with-session)
gnome-keyring-daemon -c pkcs11,secrets
<!-- more -->
Those 2 lines took me quite some Googling to figure out..

So I have X11 `DISPLAY` environment variable configured on my SSH session, and GUI applications like gvim runs ok, even QtCreator and Quartus etc. can show their GUI just fine. gedit takes a while but will eventually show up.

Then I was trying to launch `gnome-terminal` on my SSH session, which after a while gives me this error:

```
# Error constructing proxy for org.gnome.Terminal:/org/gnome/Terminal/Factory0: Error calling StartServiceByName for org.gnome.Terminal: Timeout was reached
```

`gnome-terminal` is special that it actually asks the `gnome-terminal-server` daemon to display the GUI, `gnome-terminal` itself will just exit afterwards.

That error, now that I have found out the solution, I think it means it's trying to launch the service/application though dbus, but was unable to connect. To fix it, either launch it through `dbus-launch`:

```
dbus-launch gnome-terminal
```

Or declare dbus session environment variables as in TL;DR:  
`--exit-with-session` is necessary otherwise `dbus-daemon` will be persistent after SSH session exits.

```
export $(dbus-launch --exit-with-session)
```

Now the terminal UI shows up, but it took quite some time. It must be waiting on something that eventually timed out.  
Only for the first time in each SSH session though, on subsequent launches the GUI shows up straight away.  
To debug this, declare environment variable `G_DBUS_DEBUG`:

```
G_DBUS_DEBUG=all gnome-terminal
```

```
......
GDBus-debug:Call:
 >>>> ASYNC org.freedesktop.DBus.Properties.GetAll()
      on object /org/gnome/Terminal/Factory0
      owned by name :1.1 (serial 7)
......
GDBus-debug:Call:
 <<<< ASYNC COMPLETE org.freedesktop.DBus.Properties.GetAll()
      FAILED: Timeout was reached
......
```

While watching `/var/log/syslog`:

```
Jun 22 21:41:44 zhiyb-Linux dbus-daemon[565554]: [session uid=1000 pid=565552] Activating service name='org.freedesktop.secrets' requested by ':1.3' (uid=1000 pid=565616 comm="/usr/libexec/xdg-desktop-portal ")
Jun 22 21:42:09 zhiyb-Linux xdg-desktop-por[565616]: Failed to create secret proxy: Error calling StartServiceByName for org.freedesktop.secrets: Timeout was reached
Jun 22 21:42:09 zhiyb-Linux xdg-desktop-por[565616]: No skeleton to export
Jun 22 21:42:09 zhiyb-Linux dbus-daemon[565554]: [session uid=1000 pid=565552] Successfully activated service 'org.freedesktop.portal.Desktop'
```

So, some sort of secret proxy timed out.. Turns out that's the gnome-keyring sevice.  
To fix this, launch `gnome-keyring-daemon`.  
I noticed that it prints something like this:  
`SSH_AUTH_SOCK=/run/user/1000/keyring/ssh  
`Which is environment variable declaration for SSH agent. However, the agent seems to be problematic when multiple SSH sessions are open and multiple `gnome-keyring-daemon` instances are running, so disable the SSH component:

```
gnome-keyring-daemon -c pkcs11,secrets
```

Anyway, now `gnome-terminal` launches almost immediately every time, so I'm happy 🙂.

* * *

Some extra stuff:

Launching `virt-manager` and trying to connect to `qemu:///system` gives error:

```
Unable to connect to libvirt qemu:///system.

authentication unavailable: no polkit agent available to authenticate action 'org.libvirt.unix.manage'
```

To fix this, launch a GNOME polkit agent in the background.  
I'm using debian 11, so it's here:

```
/usr/lib/policykit-1-gnome/polkit-gnome-authentication-agent-1 &
```

* * *

To force GPG to use console-mode pinentry, install `pinentry-curses`, then

```
sudo update-alternatives --config pinentry
```
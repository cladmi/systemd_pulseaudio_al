Running PulseAudio as a daemon on a headless server with SystemD
================================================================


My goal is to centralize the music playing on my home server from various devices.

For that I wanted to run PulseAudio as a daemon on my home server and enable the 'over the air' tcp module.

And I finally got it working!


Context
-------

Arch Linux with SystemD, no X11 server, no direct login if it's not for last change troubleshooting.


Tutorial state
--------------

All the given tutorial pages are quite short, and quickly understandable, so read them!

I got a lot of trial and errors, so I'm not sure anymore of what was required or not. Or where I got the information.
But I will post all I have from now, so if I should redo it later, I will be able.

### TODO ###

* Add error messages
* try to find source of information
* reinstall it from zero to check all required steps



Prequisites
===========

Making pulseaudio and tcp communication work when pulseaudio is run as a regular user.
Start with `pulseaudio --start` and see the documentation to know how to do it. The arch linux one is always quite good.

https://wiki.archlinux.org/index.php/PulseAudio/Examples#PulseAudio_over_network


Iptables
--------

I should open the port 4713 on tcp to get it working.


Testing
-------

### On the server ###

The sound should work when run from the server.


### On the client ###

The `PULSE_SERVER` environment variable should point to the remote server. And in this case, the sound should come out of the server speakers.

    PULSE_SERVER=tcp:server_ip_address_or_hostname:4713 mplayer epic_sax_guy.mp3

If not, go back and RTFM.

A quick way to test it too, is to try a telnet on 'server:4713'.
If it fails, there is a problem, if it works, try the real one to verify.



Running PulseAudio as a Daemon with SystemD
===========================================


http://www.freedesktop.org/wiki/Software/PulseAudio/Documentation/User/WhatIsWrongWithSystemWide/

Do you want to do it anyway ? Just read it one more time, to really understand, and find what you need to know.

> You are on your own.
> You need to know you way around, be able to write init scripts, dbus policies, to fix up device permissions, and unix users, you need to pass around security cookies and more.

Some official documentation to read before going to the next part.

http://www.freedesktop.org/wiki/Software/PulseAudio/Documentation/User/SystemWide/


Systemd service file
--------------------

Read the `pulseaudio.service` attached file, and copy it to `/etc/systemd/system/pulseaudio.service`.

And try to start pulseaudio, it should fail. In two different terminals:

    # journalctl -xn -f
    # systemctl --system daemon-reload; systemctl restart pulseaudio


Users permissons
----------------

You should see an error message saying 'use pulse not found' or something. So create it as told in the previous documentation and retry.




Dbus
----

The new error message is now something like 'org.dbus could not acquire service org.pulseaudio.Server'  blablabla.

dbus should be allowed to own the pulseaudio service, and for that, copy the `pulseaudio.conf` file to  `/etc/dbus-1/system.d/pulseaudio.conf`



PulseAudio starts but still no remote sound
-------------------------------------------


You should be able to start pulseaudio with systemctl command, no error message, only some warnings.

    pulseaudio[9128]: N: [pulseaudio] main.c: Running in system mode, forcibly disabling SHM mode!
    pulseaudio[9128]: N: [pulseaudio] main.c: Running in system mode, forcibly disabling exit idle time!
    pulseaudio[9130]: [pulseaudio] main.c: OK, so you are running PA in system mode. Please note that you most likely shouldn't be doing that.
    pulseaudio[9130]: [pulseaudio] main.c: If you do it nonetheless then it's your own fault if things don't work as expected.
    pulseaudio[9130]: [pulseaudio] main.c: Please read http://pulseaudio.org/wiki/WhatIsWrongWithSystemMode for an explanation why system mode is usually a ba

But it still cannot be accessed from the outside.

You stop it, restart it as a user with `pulseaudio --start` it works, but as a daemon it does not.
It took me some time to find out why. Finally I run the `pactl list` command as the `pulse` user, and, found no informations about `tcp` modules.
The configuration has 4 files, two of them are `default.pa` and `system.pa`. The configuration should go in the second one, not in the first one…

So add the `load-module module-native-protocol-tcp auth-anonymous=1` line, or the more personalized one, to the `system.pa` file.


Finally
-------

Now, restart two more times to see no errors on start up, and test it works when appropriate.

The final command being `systemctl enable pulseaudio.service` and then having fun.



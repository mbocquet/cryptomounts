# cryptremote

This is a Debian based initscript used to open LUKS volume(s) using a keyphrase
located on remote(s) server(s).

## Why ?

- to mount LUKS volumes automatically without using a local keyfile, USB stick
  or interactive password.
- to be able to disable remotely the next automount of volume(s).
- to protect data hosted in a remote datacenter you can't trust, in case of
  server / storage compromision or theft.

## How does it works ?

The initscript looks for a file containing LUKS UUID and the URL of the
keyphrase is used to open LUKS volume(s). As volumes are opened, they can be
mounted via autofs or fstab. Volume(s) unmounting is done via the standard
shutdown process.

If autofs is used to mount these volumes, its initscript must be modified to add 'cryptremote'
as a dependency

## Dependencies

- curl
- a proper .netrc file to access remote URL over https without interactions

## Installation
### Debian based distributions with sysinit
- copy `sysinit/cryptremote.sysinit` to `/etc/init.d/cryptremote`  
  `cp sysinit/cryptremote.sysinit /etc/init.d/cryptremote`
- make the script executable  
  `chmod +x /etc/init.d/cryptremote`
- copy `cryptremote.default` to `/etc/default/cryptremote`  
  `cp cryptremote.default /etc/default/cryptremote`
- Customize `/etc/default/cryptremote`
- Insert script into sysinit process  
  `insserv -d cryptremote`
- Add cryptremote dependency to autofs  
`/etc/init.d/autofs`  
Replace :
<pre>
...
-# Required-Start: $network $remote_fs $syslog
-# Required-Stop: $network $remote_fs $syslog
...
</pre>
by
<pre>
...
+# Required-Start: $network $remote_fs $syslog cryptremote
+# Required-Stop: $network $remote_fs $syslog cryptremote
...
</pre>

### Debian based distributions with systemd
- copy `systemd/cryptremote.service` to `/etc/systemd/system/`  
`cp systemd/cryptremote.service /etc/systemd/system/`
- copy `systemd/cryptremote` to `/usr/local/sbin`  
`cp systemd/cryptremote /usr/local/sbin/`
- make the script executable  
`chmod +x /usr/local/sbin/cryptremote`
- copy `cryptremote.default` to `/etc/default/cryptremote`  
`cp cryptremote.default /etc/default/cryptremote`
- Customize `/etc/default/cryptremote`
- Add cryptremote dependency to autofs  
`mkdir /etc/systemd/system/autofs.service.d/`  
`/etc/systemd/system/autofs.service.d/local.conf`
<pre>
[Unit]
After=cryptremote.service
Requires=cryptremote.service
</pre>
Note: 'Unit' is not 'unit', respect case !
- Reload systemd daemon  
`systemctl daemon-reload`
- Enable `cryptremote.service` Unit  
`systemctl enable cryptremote.service`
- Reload autofs.service  
  `systemctl reload autofs.service`
- Check the status of autofs local.conf  
  `systemctl status autofs.service`
  <pre>
  ● autofs.service - LSB: Automounts filesystems on demand
     Loaded: loaded (/etc/init.d/autofs)
    Drop-In: /etc/systemd/system/autofs.service.d
             └─local.conf
     Active: active (running) since mer. 2016-03-16 22:07:59 CET; 5s ago
    Process: 2332 ExecStop=/etc/init.d/autofs stop (code=exited, status=0/SUCCESS)
    Process: 2340 ExecStart=/etc/init.d/autofs start (code=exited, status=0/SUCCESS)
     CGroup: /system.slice/autofs.service
             └─2347 /usr/sbin/automount --pid-file /var/run/autofs.pid
  </pre>

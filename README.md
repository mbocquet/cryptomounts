# cryptremote

This is a Debian based initscript used to open LUKS volume(s) using a keyphrase
located on remote(s) server(s).

## Why ?

- to mount LUKS volumes automatically without using a local keyfile or a USB
  stick
- to be able to disable remotely the next automount of volume(s)

## How does it works ?

The initscript looks for a file containing LUKS UUID and the URL of the
keyphrase is used to open LUKS volume(s). As volumes are opened, they can be
mounted via autofs or fstab. Volume(s) unmounting is done via the standard
shutdown process.

If autofs is used to mount these volumes, its initscript must be modified to add 'cryptremote'
as a dependency

## Dependencies

- curl
- a proper .netrc file to access remote URL over https

## Installation
### Debian based distributions with sysinit
- copy `sysinit/cryptremote.sysinit` to `/etc/init.d/cryptremote`

<code>
cp sysinit/cryptremote.sysinit /etc/init.d/cryptremote
</code>

- make the script executable

<code>
chmod +x /etc/init.d/cryptremote
</code>

- copy `cryptremote.default` to `/etc/default/cryptremote`

<code>
cp cryptremote.default /etc/default/cryptremote
</code>

- Customize `/etc/default/cryptremote`

- Insert script into sysinit process

<code>
insserv -d cryptremote
</code>

- Add cryptremote dependency to autofs

/etc/init.d/autofs

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

<code>
cp systemd/cryptremote.service /etc/systemd/system/
</code>

- copy `systemd/cryptremote` to `/usr/local/sbin`
- make the script executable

<code>
chmod +x /usr/local/sbin/cryptremote
</code>

- copy `cryptremote.default` to `/etc/default/cryptremote`

`cp cryptremote.default /etc/default/cryptremote`

- Customize `/etc/default/cryptremote`

- Add cryptremote dependency to autofs

/lib/systemd/system/autofs.service

Replace :
<pre>
...
-After=network.target ypbind.service sssd.service network-online.target
-Wants=network-online.target
...
</pre>

by

<pre>
...
-After=network.target ypbind.service sssd.service network-online.target cryptremote.service
-Wants=network-online.target cryptremote.service
...
</pre>

- Reload systemd daemon

<code>
systemctl daemon-reload
</code>

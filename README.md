# cryptomounts

This is a Debian based initscript used to automount LUKS volume(s) using
a keyphrase located on remote(s) server(s).

## Why ?

- to mount LUKS volumes automatically without using a local keyfile or a USB
  stick
- to be able to disable remotely the next automount of volume(s)

## How does it works ?

The initscript looks for a file containing LUKS UUID and the URL of the
keyphrase is used to open LUKS volume(s). As volumes are opened, they can be
mounted via autofs or fstab. Volume(s) unmounting is done via the standard
shutdown process.

In case autofs is used, its initscript must be modified to add 'cryptomounts'
as a dependency :

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
+# Required-Start: $network $remote_fs $syslog cryptomounts
+# Required-Stop: $network $remote_fs $syslog cryptomounts
...
</pre>

## Dependencies

- curl
- a proper .netrc file to access remote URL over https

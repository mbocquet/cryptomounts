# cryptomounts

This is a Debian based initscript used to automount LUKS volume(s) using
a keyphrase located on remote(s) server(s).

Why ?
- to mount LUKS volumes automatically without using a local keyfile or a USB
  stick
- to be able to disable remotely the next automount of volume(s)

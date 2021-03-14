# Web Prompt Secret

## What is this

This script starts a web server that hosts a web `<form>` which essentially includes nothing but a `<textarea>` and a `<submit>` (but also some text, which is slightly configurable).

When the `<form>` is submitted, a program, which is supplied as a parameter to the script, is launched and the data entered into the `<textarea>` is sent to the STDIN of that program.

The scripts waits for the program to exits and if the exit code is zero, the web server will shut down and the script will exit.

## What is it for

I have a Raspberry Pi, with a large external hard disk attached.

The hard disk is encrypted.

But I want the data on this encrypted partition to be available to programs which are started on boot (like PostgreSQL).

The Raspberry Pi does not have a monitor attached.

This is script is how I get the passphrase during the boot process so the disk can be decrypted.

**NOTE** This script does not deal with encryption. I use this at home and trust my network (when I'm at home). If you use this on an untrusted network you will need to take precautions, like running an NGINX SSL proxy on the same host.

## How

I execute this script on boot like the following:

#### /usr/local/sbin/luks-open

```bash
#!/bin/bash

if ! cryptsetup isLuks "/dev/sdb1"; then
	exit 1
fi

if ! cryptsetup status "/dev/sdb1"; then

    web-prompt-secret --secret 'LUKS PASSPHRASE' --process BOOT --label passphrase --port "8080" cryptsetup open "/dev/sdb1" "encrypted" -
	exit $?
fi
exit 0
```

#### /etc/systemd/system/luks-open.service

```systemd
[Unit]
Description=Luks Web Mount
After=network.target network-online.target
Wants=network-online.target

[Service]
Type=oneshot
ExecStart=luks-open
TimeoutStartSec=0
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```

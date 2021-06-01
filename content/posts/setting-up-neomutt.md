---
layout: post
title: "Setting Up neomutt with notmuch, gmail(lieer) and gpg signing"
<!-- date: Thu, 18 Feb 2021 22:02:13 +0530 -->
categories: neomutt notmuch gmail gpg
---

- [start](#before-we-begin)
- [install](#installing)
- [notmuch](#notmuch)
  - [hooks](#notmuchhooks)
    - [pre-new](#pre-new)
    - [post-new](#post-new)
- [lieer](#lieer-gmail)
- [neomutt](#neomutt)
  - [mailcap](#neomuttmailcap)
  - [vim-keys.rc](#neomuttvim-keysrc)
  - [dracula](#neomuttdracula)
- [encrypt/sign](#gpg)
- [systemd/timers](#systemd-timers)

## Before we begin

First of all I don't like dotfiles inside the home folder.  
And I wish all programs would follow the [XDG Base directory standard](https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html).  
But since they don't we have to manually export environment variables to specify the location of config files.  
ArchWiki has an excellent list of programs that support the XDG Base directory spec [here](https://wiki.archlinux.org/index.php/XDG_Base_Directory)

```bash
# Notmuch
export NOTMUCH_CONFIG="$XDG_CONFIG_HOME"/notmuch/notmuchrc
export NMBGIT="$XDG_DATA_HOME"/notmuch/nmbug
```

You need to make sure that the `XDG_CONFIG_HOME` and `XDG_DATA_HOME` variables are already exported.

```bash
# XDG Dirs
export XDG_CONFIG_HOME="/home/<username>/.config"
export XDG_DATA_HOME="/home/<username>/.local/share"
```

## Installing

- Arch Linux

```bash
sudo pacman -S notmuch neomutt
yay -S gmailieer
```

You can also install gmailieer from pip

## Notmuch

I'm setting my maildir to `~/Mail`.  
You can run `notmuch init` or you can directly just edit the `~/.config/notmuch/notmuchrc` file and put in the following.

```bash
[database]
path=/home/<username>/Mail

[user]
name=<Your Name>

[new]
tags=unread;inbox;
ignore=/.*[.](json|lock|bak)$/

[search]
exclude_tags=deleted;spam;

[maildir]
synchronize_flags=true
```

#### notmuch/hooks

Notmuch hooks are executed pre and post a notmuch command

the hooks are stored in

```
~/Mail/.notmuch/hooks
```

in my case.

#### pre-new

Adding a pre-new hook with the contents

```bash
#!/bin/sh
cd ~/Mail
gmi sync
```

in `~/Mail/.notmuch/hooks/pre-new`

This will execute before notmuch new is executed

#### post-new

And adding post-new

```bash
#!/bin/sh
notmuch tag -inbox -unread tag:sent AND tag:inbox AND tag:unread
unread=$(notmuch search tag:unread | wc -l)
if [[ $unread -gt 0 ]];then
    notify-send "$unread new emails"
fi
```

This will execute after notmuch new

Notmuch marks sent messages a unread and inbox tags.  
The post-new will remove inbox and unread tags from sent messages.  
This will also send a notification with the count of unread emails.

## Lieer (Gmail)

First run

```bash
cd ~/Mail
gmi init your.email@gmail.com
```

This will open an OAuth window In your browser  
Authorize it and then run.  
If you want the `personal`, `updates`,`social`,`forums` tags from gmail then run

```bash
gmi set --ignore-tags-remote ""
```

and then

```bash
cd ~/Mail
gmi pull
```

This will take a while (took me around 8 minutes for 15K emails)

## neomutt

Edit the neomutt config file like so.

```bash

# Read password
set my_pass = "`pass Email/neomutt`"

# Sending mail
set smtp_url = "smtp://<email.username>@smtp.gmail.com:587"
set smtp_pass = $my_pass

# Creds
set from = "<email.username>@gmail.com"
set signature = "~/.config/neomutt/signature"

# Sign with gpgme
set crypt_use_gpgme=yes
set postpone_encrypt = yes
set pgp_self_encrypt = yes
set crypt_use_pka = no
set crypt_autoencrypt = no
set crypt_autopgp = yes
set crypt_autosign = yes
set crypt_verify_sig = yes
set pgp_sign_as=0xYOUR123GPG

# Enable virtual mailboxes
set virtual_spoolfile=yes

virtual-mailboxes "Inbox" "notmuch://?query=tag:inbox"
virtual-mailboxes "Unread" "notmuch://?query=tag:unread"
virtual-mailboxes "Personal" "notmuch://?query=tag:personal"
# virtual-mailboxes "Social" "notmuch://?query=tag:social"
virtual-mailboxes "Updates" "notmuch://?query=tag:updates"
virtual-mailboxes "Forums" "notmuch://?query=tag:forums"
virtual-mailboxes "Draft" "notmuch://?query=tag:draft"
virtual-mailboxes "Sent" "notmuch://?query=tag:sent"
virtual-mailboxes "Spam" "notmuch://?query=tag:spam"
# virtual-mailboxes "Trash" "notmuch://?query=tag:trash"

set postponed = "~/Mail/Drafts"
set record = ""

# Sidebar
set sidebar_visible
set sidebar_format = "%D%* [%N]%*"
set mail_check_stats

# Activate caching, as it can greatly improve speed
set header_cache = "~/.cache/neomutt/headers"
set message_cachedir = "~/.cache/neomutt/bodies"


# Sort by threads
set sort = threads
# Sort threads by last date recieved - newest first
set sort_aux = reverse-last-date-received
# Show date in year/month/day hour:minute format
set date_format="%d/%m/%y %I:%M%p"

# mailcap and auto_view
set mailcap_path = "~/.config/neomutt/mailcap"
auto_view text/html

source ~/.config/neomutt/vim.muttrc
source ~/.config/neomutt/dracula/dracula.muttrc
```

If you don't use vim keys or the dracula theme remove the last two lines.

#### neomutt/mailcap

To view html emails use mailcap with w3m  
The mailcap file `~/.config/neomutt/mailcap`

```
text/html; w3m -I %{charset} -T text/html ; copiousoutput; nametemplate=%s.html
```

#### neomutt/vim-keys.rc

The vim.muttrc file from [vim-keys.rc](https://github.com/neomutt/neomutt/blob/master/contrib/vim-keys/vim-keys.rc)

```bash
# Moving around
bind attach,browser,index       g   noop
bind attach,browser,index       gg  first-entry
bind attach,browser,index       G   last-entry
bind pager                      g  noop
bind pager                      gg  top
bind pager                      G   bottom
bind pager                      k   previous-line
bind pager                      j   next-line

# Scrolling
# bind attach,browser,pager,index \CF next-page
# bind attach,browser,pager,index \CB previous-page

bind attach,browser,pager,index \Cu half-up
bind attach,browser,pager,index \Cd half-down
bind browser,pager              \Ce next-line
bind browser,pager              \Cy previous-line
bind index                      \Ce next-line
bind index                      \Cy previous-line

bind pager,index                d   noop
bind pager,index                dd  delete-message

# bind index                      \Cm list-reply # Doesn't work currently


# Threads
bind browser,pager,index        N   search-opposite
bind pager,index                dT  delete-thread
bind pager,index                dt  delete-subthread
bind pager,index                gt  next-thread
bind pager,index                gT  previous-thread
bind index                      za  collapse-thread
bind index                      zA  collapse-all # Missing :folddisable/foldenable
```

### neomutt/dracula

```bash
cd ~/.config/neomutt/
git clone https://github.com/dracula/mutt dracula
```

## GPG

Set your gpg key in neomutt in the neomuttrc as

```bash
gpg --keyid-format 0xlong -K --fingerprint
```

then set the gpg key in neomutt

```
set pgp_sign_as=0xYOUR123GPG
```

### Systemd Timers

We can use user systemd units to auto run notmuch new and sync emails automatically.

Systemd timers and services don't seem to have the environment variables when run.
So we need to export them before we run the services.

`checkmail`

```bash
#!/usr/bin/bash
export NOTMUCH_CONFIG=/home/<username>/.config/notmuch/notmuchrc
export NMBGIT=/home/<username>/.local/share/notmuch/nmbug
notmuch new
```

Put checkmail in `~/.local/bin/checkmail`

`checkmail.service`

```systemd
[Unit]
Description=Check Mail
RefuseManualStart=no
RefuseManualStop=yes

[Service]
Type=oneshot
ExecStart=/home/<username>/.local/bin/checkmail
```

`checkmail.timer`

```systemd
[Unit]
Description=Check Mail every fifteen minutes
RefuseManualStart=no
RefuseManualStop=no

[Timer]
Persistent=false
OnBootSec= 5min
OnUnitActiveSec= 15min
Unit=checkmail.service

[Install]
WantedBy=default.target
```

put `checkmail.timer` and `checkmail.service` in `~/.config/systemd/user/` folder

Now you can enable checkmail.timer to start on login by

```bash
systemctl --user enable checkmail.timer
```

You can check the status of `checkmail.timer` by

```bash
systemctl --user status checkmail.timer
```

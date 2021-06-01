---
layout: post
title: "Setting up neomutt with notmuch, mbsync and gpg signing"
<!-- date: Sat, 03 Apr 2021 23:50:31 +0530 -->
categories: neomutt notmuch gmail gpg disroot
---

This is an update from my previous post [here](https://uttarayan21.github.io/2021/02/setting-up-neomutt).

- [start](#before-we-begin)
- [install](#installing)
- [notmuch](#notmuch)
  - [hooks](#notmuchhooks)
    - [pre-new](#pre-new)
    - [post-new](#post-new)
- [mbsync](#mbsync)
  - [gmail](#gmail)
  - [disroot](#disroot)
- [neomutt](#neomutt)
  - [mailcap](#neomuttmailcap)
  - [vim-keys.rc](#neomuttvim-keysrc)
  - [dracula](#neomuttdracula)
- [encrypt/sign](#gpg)
- [systemd/timers](#systemd-timers)

## Before we begin

I recently move to [disroot](disroot.org) and wanted to change from gmi to mbsync.
First of all I don't like dotfiles inside the home folder.  
And I wish all programs would follow the [XDG Base directory standard](https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html).  
But since they don't we have to manually export environment variables to specify the location of config files.  
ArchWiki has an excellent list of programs that support the XDG Base directory spec [here](https://wiki.archlinux.org/index.php/XDG_Base_Directory)

```bash
# Notmuch
export NOTMUCH_CONFIG="$XDG_CONFIG_HOME/notmuch/notmuchrc"
export NMBGIT="$XDG_DATA_HOME/notmuch/nmbug"
alias mbsync -c "$XDG_CONFIG_HOME/isync/config"
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
sudo pacman -S notmuch neomutt isync
```

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
ignore=/.*[.](json|lock|bak|uidvalidity|mbsyncstate|)$/

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
mbsync -a -c ~/.config/isync/config
```

in `~/Mail/.notmuch/hooks/pre-new`

This will execute before notmuch new is executed

#### post-new

And adding post-new

```bash
#!/bin/sh

gmail_unread=$(notmuch count tag:unread to:"*gmail.com")
proton_unread=$(notmuch count tag:unread to:"*protonmail.com")
disroot_unread=$(notmuch count tag:unread to:"*disroot.org")
unread=$(expr $gmail_unread + $proton_unread + $disroot_unread)

if [[ $unread -gt 0 ]];then
    label=""
    if [[ $gmail_unread -gt 0 ]];then
       label+="\n$gmail_unread in gmail"
    fi
    if [[ $proton_unread -gt 0 ]];then
        label+="\n$proton_unread in protonmail"
    fi
    if [[ $disroot_unread -gt 0 ]];then
        label+="\n$disroot_unread in disroot"
    fi
    notify-send "$unread unread emails" "$label"
fi

```

This will execute after notmuch new

Notmuch marks sent messages a unread and inbox tags.  
The post-new will remove inbox and unread tags from sent messages.  
This will also send a notification with the count of unread emails.

## mbsync

Passwords are saved using the `pass` command.  
I use gnome-keyring to unlock the gpg keyring and cache it to gpg-agent on unlock.  
You can try to use [cruegge/pam-gnupg](https://github.com/cruegge/pam-gnupg) if you don't want to use gnome-keyring.

## gmail

```
IMAPAccount <username>@gmail.com
    Host imap.gmail.com
    User <username>@gmail.com
    PassCmd "pass Email/mbsync"
    SSLType IMAPS
    AuthMechs LOGIN

IMAPStore gmail-remote
    Account <username>@gmail.com
    UseNamespace yes

MaildirStore gmail-local
    SubFolders Verbatim
    Inbox ~/Mail/gmail/Inbox
    Path ~/Mail/gmail

Channel gmail-inbox
    Create Both
    Expunge Both
    SyncState *
    Sync Pull All Push All
    Far :gmail-remote:
    Near :gmail-local:
    Patterns "INBOX"

Channel gmail-sent
    Create Both
    Expunge Both
    SyncState *
    Sync Pull All Push All
    Far :gmail-remote:"[Gmail]/Sent Mail"
    Near :gmail-local:/Sent

Channel gmail-trash
    Create Both
    Expunge Both
    SyncState *
    Sync Pull All Push All
    Far :gmail-remote:"[Gmail]/Trash"
    Near :gmail-local:/Trash

Channel gmail-drafts
    Create Both
    Expunge Both
    SyncState *
    Sync Pull All Push All
    Far :gmail-remote:"[Gmail]/Drafts"
    Near :gmail-local:/Drafts

Channel gmail-spam
    Create Both
    Expunge Both
    SyncState *
    Sync Pull All Push All
    Far :gmail-remote:"[Gmail]/Spam"
    Near :gmail-local:/Spam

Group gmail
    Channel gmail-inbox
    Channel gmail-drafts
    Channel gmail-sent
    Channel gmail-trash
    Channel gmail-spam
```

## disroot

```
IMAPAccount <username>@disroot.org
    Host disroot.org
    Port 993
    User <username>@disroot.org
    PassCmd "pass user.disroot.org/<username>"
    SSLType IMAPS
    AuthMechs LOGIN

IMAPStore disroot-remote
    Account <username>@disroot.org
    UseNamespace yes

MaildirStore disroot-local
    SubFolders Verbatim
    Inbox ~/Mail/disroot/INBOX
    Path ~/Mail/disroot/

Channel disroot
    Far :disroot-remote:
    Near :disroot-local:
    Patterns *
    Create Both
    Expunge Both
    SyncState *
    Sync Pull All Push All
```

## neomutt

Edit the neomutt config file like so.

```
# Gmail
source ~/.config/neomutt/gmail/<usrname>
folder-hook 'gmail' 'source ~/.config/neomutt/gmail/<usrname>'

# Disroot
source ~/.config/neomutt/disroot/<usrname>
folder-hook 'disroot' 'source ~/.config/neomutt/disroot/<usrname>'

# Macros for switching accounts
macro index <f2> '<sync-mailbox><refresh><enter-command>source ~/.config/neomutt/gmail/<usrname><enter><change-folder>!<enter>'
macro index <f3> '<sync-mailbox><refresh><enter-command>source ~/.config/neomutt/disroot/<usrname><enter><change-folder>!<enter>'

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

# Sidebar
set sidebar_visible
# set sidebar_format = "%B%?F? [%F]?%* %?N?%N/?%S"
set sidebar_format = "%D%* [%N]%*"
set mail_check_stats

# Activate caching, as it can greatly improve speed
set header_cache = "~/.cache/neomutt/headers"
set message_cachedir = "~/.cache/neomutt/bodies"

# Disable saving outgoing mail since Gmail saves them by default.
# set record = ""

# Sort by threads
set sort = threads
# Sort threads by last date recieved - newest first
set sort_aux = reverse-last-date-received
# Show date in year/month/day hour:minute format
set date_format="%d/%m/%y %I:%M%p"

# mailcap and auto_view
set mailcap_path = "~/.config/neomutt/mailcap"
auto_view text/html

source ~/.config/neomutt/vimkeys.muttrc
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
Description=Check Mail every five minutes
RefuseManualStart=no
RefuseManualStop=no

[Timer]
Persistent=false
OnBootSec= 5min
OnUnitActiveSec= 5min
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

# jailkit-inits
[jailkit](https://olivier.sessink.nl/jailkit/) is an awesome tool to manage your chroot environments given the rise in complexity of softwares.

I've compiled a list of common chroot environments that I generally need, such as `nodejs`, to be able to run these disasters in a secure environment.

This list is specific for OpenBSD but should be informative for other operating systems as well.

PRs are highly welcome, the more tools that can be made secure, the easier it is for OpenBSD chroots to be used.

### Basic environment
Things which I have found are needed on almost all chroots and are very light, so should almost always be present.  
My configs for making a base are split up so that they are a bit more modular, incase someobody needs only some of the funcationality
```
[uidbasics]
comment = common files for all jails that need user/group information
paths = /lib/libnsl.so.1, /lib64/libnsl.so.1, /lib/libnss*.so.2, /lib64/libnss*.so.2, /lib/i386-linux-gnu/libnsl.so.1, /lib/i386-linux-gnu/libnss*.so.2, /lib/x86_64-linux-gnu/libnsl.so.1, /lib/x86_64-linux-gnu/libnss*.so.2, /lib/arm-linux-gnueabihf/libnss*.so.2, /lib/arm-linux-gnueabihf/libnsl*.so.1, /etc/nsswitch.conf, /etc/ld.so.conf
devices = /dev/tty, /dev/null, /dev/urandom

[coreutils]
comment = sanitized coreutils like for general functionality + important shell utilities
paths = [, awk, basename, bunzip2, bzip2, cat, chmod, chown, clear, cp, cpio, cut, date, dd, df, du, echo, egrep, expr, false, fgrep, find, grep, gunzip, gzcat, gzip, head, id, install, kill, ldd, less, ln, ls, md5, mkdir, mkfifo, mknod, mktemp, more, mv, nice, pgrep, pkill, pwd, rm, rmdir, sed, sh, sha1, sha256, sha512, sleep, sort, sync, tail, tar, touch, tr, true, tty, uname, uncompress, wc, what which, who, whoami, zcat, zegrep, zfgrep, zgrep

[base]
comment = essential shells and their configs needed for almost every chroot
paths = sh, csh, ksh, bash, zsh, env, /usr/libexec/ld.so, /var/run/ld.so.hints, /etc/motd, /etc/issue, /etc/bash.bashrc, /etc/bashrc, /etc/ksh*, /etc/profile, /usr/lib/locale/en_US.utf8
users = root
groups = root
includesections = uidbasics, coreutils

[netbase]
comment = common files for all jails that need any internet connectivity
paths = /lib/libnss_dns.so.2, /lib64/libnss_dns.so.2, /lib/libnss_mdns*.so.2, /etc/resolv.conf, /etc/host.conf, /etc/hosts, /etc/protocols, /etc/services
includesections = base
```

### Editors
```
[editors]
comment = vi, vim, and nano
paths = ex, nano, vi, vim, rvim, view, rview, vimdfiff, vimtutor, /etc/vimrc, /usr/share/vim
includesections = base
```

### Limited shell
```
[jk_lsh]
comment = Jailkit limited shell
paths = /usr/local/sbin/jk_lsh, /etc/jailkit/jk_lsh.ini
users = root
groups = root
includesections = uidbasics, logbasics

[limitedshell]
comment = alias for jk_lsh
includesections = jk_lsh
```

### Logging using syslog in chroot
```
[logbasics]
comment = timezone information and log sockets
paths = /etc/localtime
need_logsocket = 1
```

### RCS systems
```
[cvs]
comment = Concurrent Versions System
paths = cvs
devices = /dev/null

[git]
comment = Fast Version Control System
paths = /usr/bin/git*, /usr/lib/git-core, /usr/bin/basename, /bin/uname, /usr/bin/pager
includesections = editors, perl
```

### SSH and other utils

```
[scp]
comment = ssh secure copy
paths = scp
includesections = netbase
devices = /dev/urandom

[sftp]
comment = ssh secure ftp
paths = /usr/lib/sftp-server, /usr/libexec/openssh/sftp-server, /usr/lib/misc/sftp-server, /usr/libexec/sftp-server, /usr/lib/openssh/sftp-server
includesections = netbase
devices = /dev/urandom, /dev/null

[ssh]
comment = ssh secure shell
paths = ssh
includesections = netbasics, uidbasics
devices = /dev/urandom, /dev/tty, /dev/null

[rsync]
paths = rsync
includesections = netbasics, uidbasics
```
### Utilities
```
[procmail]
comment = procmail mail delivery
paths = procmail, /bin/sh
devices = /dev/null

[midnightcommander]
comment = Midnight Commander
paths = mc, mcedit, mcview, /usr/local/share/mc
includesections = basicshell, terminfo

[netutils]
comment = several internet utilities like wget, ftp, rsync, scp, ssh
paths = wget, lynx, ftp, host, rsync, smbclient
includesections = netbase, ssh, sftp, scp

[apacheutils]
comment = htpasswd utility
paths = htpasswd

[extshellplusnet]
comment = alias for extendedshell + netutils + apacheutils
includesections = extendedshell, netutils, apacheutils

[openvpn]
comment = jail for the openvpn daemon
paths = /usr/local/sbin/openvpn
users = root,nobody
groups = root,nogroup
devices = /dev/urandom, /dev/random, /dev/net/tun
includesections = netbasics, uidbasics
need_logsocket = 1

[apache]
comment = the apache webserver, very basic setup, probably too limited for you
paths = /usr/local/apache
users = root, www-data
groups = root, www-data
includesections = netbasics, uidbasics

[perl]
comment = the perl interpreter and libraries
paths = perl, /usr/lib/perl, /usr/lib/perl5, /usr/share/perl, /usr/share/perl5

[xauth]
comment = getting X authentication to work
paths = /usr/X11R6/bin/xauth, /usr/X11R6/lib/X11/rgb.txt

[xclients]
comment = minimal files for X clients
paths = /usr/X11R6/lib/X11/rgb.txt
includesections = xauth

[vncserver]
comment = the VNC server program
paths = Xvnc, Xrealvnc, /usr/X11R6/lib/X11/fonts/
includesections = xclients

[ping]
comment = Ping program
paths_w_setuid = /bin/ping

#[xterm]
#comment = xterm
#paths = /usr/X11R6/bin/xterm, /usr/share/terminfo, /etc/terminfo
#devices = /dev/pts/0, /dev/pts/1, /dev/pts/2, /dev/pts/3, /dev/pts/4, /dev/ptyb4, /dev/ptya4, /dev/tty, /dev/tty0, /dev/tty4
```

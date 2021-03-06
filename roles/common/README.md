ansible::common
================

The common role is used to create a suitable environment for ansible to manage the system in.

This role handles a few basic setup tasks.

Create local user for ansible to use
-------------------------------------

This is done in order to restrict (or deny) root logins (notably via SSH).

The part of the process does the following:

* looks for a suitable id_rsa.pub under ~/.ssh/ (of the user running ansible).
* creates an unprivileged user (ansible:deploy, by default) on the target system
* populates authorised keys for that user based on the id_rsa.pub found previously, or based on variable data.
* creates a sudoers.d file allowing tty-less, password-less access to ALL

Install EPEL
-------------

For RedHat-likes, install the epel-release package.

This will trigger a full system update (via the `common | packages | update all` handler).

The main reason for it doing so is that certain other roles (openssh, notably) will no work correctly otherwise.
In order to use certain ciphers, this is in effect necessary.

Configure systemd-timesyncd and enable it
------------------------------------------

There seem to be 4 basic choices on Linux for synching time:

* ntpd, the default server/client for linux
* ntpdate, which (apparently) comes with ntpd, and is client-only
* sntp, client-only, which supposedly supercedes ntpdate
* systemd-timesyncd (and timedatectl), another client-only baked in to systemd.

Most documented best practice seems to agree ntpd is a bad choice unless you _want_ a server.

Of the remaining 3 options, sntp seems to be recommended over ntpdate.
sntp itself generally requires installation and a cronjob/systemd timer.
systemd-timesyncd seems to do what sntp does, without the cruft.

This role will configure timesyncd (to use x.pool.ntp.org, for 0 < x < 4).

It will also call `timedatectl set-ntp true` to enable and run the NTP client.

Please note this aspect of the role will likely change.
For one, ntpsec should be appearing some time soon.
It may also be further/better research will end up saying sntp is a better choice.

See https://wiki.archlinux.org/index.php/Time#Time_synchronization for additional info.


NB: It turns out that for Centos7, the above is incorrect.
timesyncd relies on the ntp daemon to sync time.
The relevant task therefore takes care of installing it.

Install unzip
--------------

Needed to extract zip files, e.g. for consul.


Create sysconfig directory
---------------------------

/etc/sysconfig provides a place to store things like environment files for systemd services.
On RedHat-like systems, it also provides somewhere to stay ifup configs, and other things.
I believe /etc/default is used on some other distributions.

The LSB (or FHS) is silent on the use of either.

So it seemed easiest to just create /etc/sysconfig if absent, and make other roles use it.



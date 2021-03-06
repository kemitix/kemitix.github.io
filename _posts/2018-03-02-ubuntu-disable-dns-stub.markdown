---
layout: post
title: "Disable Ubuntu's DNS stub resolver"
date: 2018-03-02 17:19:00 +0000
---
I'm starting to experiment with [Rancher](https://rancher.com/) to manage docker instances. However, when trying to starting the rancher-agent, 
the logs were showing that it didn't like the lookback address being used for DNS resolution. It then promptly refused to go
any further.

The stub resolver is intended to make things easier for users who move between networks and need a more dynamic method to
select their DNS resolution server. Whether that is because they are a laptop user or they need to connect to a VPN 
periodically.

For a more static instance, such as my development machine, a fixed DNS resolution address will do me fine.

The following instructions are from [Bastian Voigt's](https://askubuntu.com/users/680732/bastian-voigt) answer on 
[askUbuntu.com](https://askubuntu.com/):

[How to disable systemd-resolved in Ubuntu?](https://askubuntu.com/questions/907246/how-to-disable-systemd-resolved-in-ubuntu):

> Disable the systemd-resolved service and stop it:
> ```bash
> sudo systemctl disable systemd-resolved.service
> sudo service systemd-resolved stop
> ```
> Put the following line in the `[main]` section of your `/etc/NetworkManager/NetworkManager.conf`:
> ```none
> dns=default
> ```
> Delete the symlink `/etc/resolv.conf`
> ```bash
> sudo rm /etc/resolv.conf
> ```
> Restart network-manager
> ```bash
> sudo service network-manager restart
> ```

Then I simply restarted my rancher-agent docker instance and it's happy to continue.

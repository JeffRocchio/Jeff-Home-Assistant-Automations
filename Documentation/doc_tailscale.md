# Purpose

Provide info and notes concerning my setup and configuration of Tailscale for remote access to Home Assistant.

# HA Instance Tailscale URL

The following URL is used to access Home Assistant on the Raspberry Pi: <see in my hidden .secrets folder>

# Tailscale Ref

See Home Assistant page [](https://www.home-assistant.io/integrations/tailscale/)

# Connect Devices

Done! Your devices can now connect from anywhere.

Every device in your Tailscale network has a private 100.x.y.z IP address that you can reach no matter where you are. And every protocol works — SSH, RDP, HTTP, Minecraft — use whatever you want while Tailscale is running.

What’s next? Whatever you want.

Connect all your devices, or read some of our guides to learn how to take advantage for all that Tailscale has to offer.

Set up DNS
Tailscale gives devices 100.x.y.z IP addresses, but you can also set up DNS to more easily connect to your devices.

Share a node
Sharing lets you give a Tailscale user on another network access to a device within your network, without exposing it publicly.

Set up access controls
By default, every device can talk to every device. You can define rules to limit which devices can talk to each other.


# Install on My PC

Install Tailscale and sign in as JeffRocchio@github. Once you sign in on a device, it will be automatically added to your Tailscale network.

jroc@jeffFedora:~$ curl -fsSL https://tailscale.com/install.sh | sh
Installing Tailscale for fedora, using method dnf
+ '[' 3 = 3 ']'
+ sudo dnf install -y 'dnf-command(config-manager)'
[sudo] password for jroc:
Last metadata expiration check: 1:46:21 ago on Sun 26 Jan 2025 02:00:13 PM EST.
Package dnf-plugins-core-4.10.0-1.fc40.noarch is already installed.
Dependencies resolved.
Nothing to do.
Complete!
+ sudo dnf config-manager --add-repo https://pkgs.tailscale.com/stable/fedora//tailscale.repo
Adding repo from: https://pkgs.tailscale.com/stable/fedora//tailscale.repo
+ sudo dnf install -y tailscale
Tailscale stable                                                                                                                                                                                     5.3 kB/s | 832  B     00:00
Tailscale stable                                                                                                                                                                                      41 kB/s | 3.1 kB     00:00
Importing GPG key 0x957F5868:
 Userid     : "Tailscale Inc. (Package repository signing key) <info@tailscale.com>"
 Fingerprint: 2596 A99E AAB3 3821 893C 0A79 458C A832 957F 5868
 From       : https://pkgs.tailscale.com/stable/fedora/repo.gpg
Tailscale stable                                                                                                                                                                                      37 kB/s |  11 kB     00:00
Dependencies resolved.
=====================================================================================================================================================================================================================================
 Package                                               Architecture                                       Version                                                 Repository                                                    Size
=====================================================================================================================================================================================================================================
Installing:
 tailscale                                             x86_64                                             1.78.1-1                                                tailscale-stable                                              29 M

Transaction Summary
=====================================================================================================================================================================================================================================
Install  1 Package

Total download size: 29 M
Installed size: 53 M
Downloading Packages:
tailscale_1.78.1_x86_64.rpm                                                                                                                                                                           26 MB/s |  29 MB     00:01
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                                                                                                                 26 MB/s |  29 MB     00:01
Tailscale stable                                                                                                                                                                                      37 kB/s | 3.1 kB     00:00
Importing GPG key 0x957F5868:
 Userid     : "Tailscale Inc. (Package repository signing key) <info@tailscale.com>"
 Fingerprint: 2596 A99E AAB3 3821 893C 0A79 458C A832 957F 5868
 From       : https://pkgs.tailscale.com/stable/fedora/repo.gpg
Key imported successfully
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                                                                                                                                                             1/1
  Installing       : tailscale-1.78.1-1.x86_64                                                                                                                                                                                   1/1
  Running scriptlet: tailscale-1.78.1-1.x86_64                                                                                                                                                                                   1/1

Installed:
  tailscale-1.78.1-1.x86_64

Complete!
+ sudo systemctl enable --now tailscaled
Created symlink /etc/systemd/system/multi-user.target.wants/tailscaled.service → /usr/lib/systemd/system/tailscaled.service.
+ set +x
Installation complete! Log in to start using Tailscale by running:

sudo tailscale up
jroc@jeffFedora:~$



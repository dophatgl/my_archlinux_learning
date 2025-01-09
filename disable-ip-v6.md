To disable IPv6 on your system, you can modify the kernel parameters or update the configuration files. Here’s how to do it temporarily and permanently:

Temporary Method (Until Reboot)

Use the following commands to disable IPv6 without rebooting:

sudo sysctl -w net.ipv6.conf.all.disable_ipv6=1
sudo sysctl -w net.ipv6.conf.default.disable_ipv6=1
sudo sysctl -w net.ipv6.conf.lo.disable_ipv6=1

Permanent Method (Persists After Reboot)
	1.	Edit the sysctl.conf file:

sudo nano /etc/sysctl.conf


	2.	Add these lines to the end of the file:

net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1


	3.	Apply the changes:

sudo sysctl -p



Alternative Method (GRUB Configuration)
	1.	Edit the GRUB configuration:

sudo nano /etc/default/grub


	2.	Add ipv6.disable=1 to the GRUB_CMDLINE_LINUX line:

GRUB_CMDLINE_LINUX="... ipv6.disable=1"


	3.	Update GRUB:

sudo update-grub


	4.	Reboot the system:

sudo reboot



This will disable IPv6 on your system at the kernel level.
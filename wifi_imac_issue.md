# Ubuntu 20.04 with Wifi on iMac 2019 (MacOS 11 - Big Sur)

The purpose of this document is to summarize some of the things I had to figure out to successfully install Ubuntu on my iMac '19 27" 5K, a.k.a. iMac 19,1.

## t2linux 

From what I could find, iMac's do not have the t2 security chip. However, it does share hardware (and firmware) with contemporary Macbook Pros. For this reason, the [t2linux.org website](https://t2linux.org/) contains lots of helpful information about how to install Ubuntu on an iMac.

However, the guides on the t2linux website did not work for me when I ran it, so I had to mix and match some old pages from the wiki to make things work. Also, most issues are focused on Macbooks, not iMacs.

## Downloading the ISO and installing Ubuntu

Download the iso from [here](https://github.com/AdityaGarg8/mbp-ubuntu/releases/tag/v20.04-5.17.1).

The ISO can be burned to an USB with Balena Etcher.

Follow this [Installation Guide](https://github.com/t2linux/wiki/blob/6e9e8aa56c7bdff3d584bdc4eb7bfb22247c01ce/docs/distributions/ubuntu/installation.md#install-procedure).

## Wifi setup

Once you've successfully booted into Ubuntu, Wifi will not be working. This is the tricky part.

Neither the [new](https://github.com/t2linux/wiki/blob/6e9e8aa56c7bdff3d584bdc4eb7bfb22247c01ce/docs/guides/wifi.md) or [old Wifi guide](https://github.com/t2linux/wiki/blob/a4b46a7cfbe7efcbb6a0b6111e22172b0f5c4a77/docs/guides/wifi.md) worked for me. I had to mix the two to get things working properly.

iMac (and Macbooks) use proprietary Wifi (and Bluetooth) drivers which are not available on Linux. So, you must copy these over to Ubuntu to get Wifi to work. Bluetooth appears to work out of the box for the ISO in the previous section.

### Finding driver files and making them available on Ubuntu

Reboot your Mac, hold the Option key when the Mac logo appears, and select the macOS bootloader.

Once on Mac, run `ioreg -l | grep RequestedFiles` to list the Wifi drivers loaded by macOS. Here's some example output:

```bash
"RequestedFiles" = ({
    "Firmware"="C-4364s-B2/midway-X3.trx",
    "TxCap"="C-4364s-B2/midway-X3.txcb",
    "Regulatory"="C-4364s-B2/midway-X3.clmb",
    "NVRAM"="C-4364s-B2/P-midway-X3_M-HRPN_V-u__m-7.5.txt"
})
```

These files live in `/usr/lib/firmware/wifi`.

If you have a USB stick, simply copy over the files to the USB stick and reboot back into Ubuntu. I would recommend also putting the output of `ioreg -l | grep RequestedFiles` into a file on the USB as well.

If you do not have a USB stick, you can mount the EFI partition with `sudo diskutil mount disk0s1` then move files to `/Volumes/EFI`. Be very careful not to break your EFI partition!

```bash
sudo diskutil mount disk0s1
tar -czvf /Volumes/EFI/wifi.tar.gz /usr/share/firmware/wifi
ioreg -l | grep RequestedFiles > /Volumes/EFI/ioreg-output.txt
sudo diskutil unmount disk0s1
```

### Adding drivers to Ubuntu

Reboot back to Ubuntu (again by holding the Option key and selecting the Ubuntu bootloader).

Different Ubuntu kernels use different names for the expected drivers, and I found it easiest to keep reloading and checking `dmesg | grep brcm` output to find the right target names.

For example, in my Ubuntu installation, dmesg outputs

```
[ 2505.669998] brcmfmac 0000:03:00.0: Direct firmware load for brcm/brcmfmac4364b2-pcie.apple,midway.bin failed with error -2
[ 2505.670065] brcmfmac 0000:03:00.0: Direct firmware load for brcm/brcmfmac4364b2-pcie.bin failed with error -2
```

As you rename the macOS files and move them to the expected location, keep reloading with `sudo modprobe -r brcmfmac && sudo modprobe brcmfmac` and check output to see if you are making progress.

The table below should help you with renaming.

| macOS | Ubuntu |
| --- | --- |
| .trx | .bin |
| .clmb | .clm_blob |
| .txt | .txt |
| .txcb | .txcap_blob |

For my iMac19,1, the requested files in `ioreg` were called `midway-X3` etc.

If you used the EFI partition to transfer data, mount the partition and go to that directory:

```bash
sudo su
lsblk # check name of EFI partition, mine was nvme0n1p1
mkdir /mnt/efi
mount /dev/nvme0n1p1 /mnt/efi
```

Then copy the `ioreg` files into the locations requested in dmesg. This is the sequence of commands I ran to figure out what to do

```bash
# dmesg complained first about two .bin files:
# [...] brcm/brcmfmac4364b2-pcie.apple,midway.bin failed with error -2
# [...] brcm/brcmfmac4364b2-pcie.bin failed with error -2
dmesg | grep brcm

# cat ioreg output to find the right filename
cat /mnt/efi/ioreg-output.txt

# copy .trx to .bin
cp /mnt/efi/wifi-drivers/midway-X3.trx /usr/lib/firmware/brcm/brcmfmac4364b2-pcie.bin
cp /mnt/efi/wifi-drivers/midway-X3.trx /usr/lib/firmware/brcm/brcmfmac4364b2-pcie.apple,midway.bin
modprobe -r brcmfmac && sudo modprobe brcmfmac

# check dmesg again
# it complained about three new files:
# [...] brcm/brcmfmac4364b2-pcie.txt failed with error -2
# [...] brcm/brcmfmac4364b2-pcie.clm_blob failed with error -2
# [...] brcm/brcmfmac4364b2-pcie.txcap_blob failed with error -2
dmesg | grep brcm

cp /mnt/efi/wifi-drivers/midway-X3.clmb /usr/lib/firmware/brcm/brcmfmac4364b2-pcie.clm_blob
cp /mnt/efi/wifi-drivers/P-midway-X3_M-HRPN_V-u__m-7.5.txt /usr/lib/firmware/brcm/brcmfmac4364b2-pcie.txt
cp /mnt/efi/wifi-drivers/midway-X3.txcb /usr/lib/firmware/brcm/brcmfmac4364b2-pcie.txcap_blob

# reload once again
modprobe -r brcmfmac && sudo modprobe brcmfmac

# Success! Wifi is now available and dmesg shows successful output
# [ 2862.539050] brcmfmac: brcmf_c_process_txcap_blob: TxCap blob found, loading
# [ 2862.539868] brcmfmac: brcmf_c_preinit_dcmds: Firmware: BCM4364/3 wl0: Jul 12 2021 19:26:30 version 9.30.464.0.32.5.76 FWID 01-45ccefcd
dmesg | grep brcm
```

### Cleanup (EFI)

Remove the drivers from the EFI partition and unmount:

```
sudo rm -r /mnt/efi/wifi-drivers
sudo rm /mnt/efi/ioreg-output.txt
sudo unmount 
```
ref: https://gist.github.com/sebnyberg/ccf6b6c37df99a125bc2346977284622

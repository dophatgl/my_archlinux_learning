To get bluetooth to automatically reconnect my paired devices I did the following:
  1. set bluetooth service to start on boot: systemctl enable bluetooth.service
  2. set bluetooth adapter to automatically power on: edit /etc/bluetooth/main.conf and set AutoEnable=true
  3. set paired devices as trusted *this is what I was missing*: from the bluetoothctl util, enter trust XX:XX:XX:XX:XX:XX for each paired device (replace XX... with mac address)

https://bbs.archlinux.org/viewtopic.php?id=223949

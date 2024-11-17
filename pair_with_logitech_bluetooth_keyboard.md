pair with my logitech bluetooth keyboard
https://github.com/bluez/bluez/issues/531
echo -e  "# Fix BLE mouse issue\nuhid" | sudo tee /etc/modules-load.d/uhid.conf

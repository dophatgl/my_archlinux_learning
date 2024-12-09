

I know this is a 5 year old thread but I want to drop a solution here in case anyone else needs it because it actually took me a while even though a fix feels blatantly obvious

First enable your bluetooth service on startup like this:

sudo systemctl enable --now bluetooth

Do this command to find out your device's mac address and copy it:

bluetoothctl devices

Then go to your desktop environment's startup script or any startup script you want that works well. I use Hyprland so for me this was ~/.config/hypr/hyprland.conf
Here you want to make 3 commands run on startup, in Hyprland the config prompt for that is "exec-once =", use the prompt that's needed for your DE's config

exec-once = bluetoothctl trust (device's MAC)
exec-once = bluetoothctl power on
exec-once = bluetoothctl connect (device's MAC)

Now reboot and your device should auto connect on startup, do give it a bit of time though cause it takes like 20 seconds usually
Afterwards make sure you go through all of your bluetooth manager configs and check if all the auto connect options are enabled so it can reconnect to your device when you turn it on and off while your pc is active

The reason why bluetooth devices don't connect on startup is because something like blueman only auto connects devices once the computer is already started up, aka if you turn off your device and then turn it back on it will automatically connect to it. There is no actual script in blueman or bluez that connects your device to the computer on startup, you have to do that yourself.

Hope this helps someone

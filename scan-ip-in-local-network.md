

To find all local IP addresses in your home network, including those of your 6 PCs, you can use various methods. Here are a few approaches depending on your operating system and setup:

---

### **1. Using Your Router**
- **Log in to your router's admin panel:**
  1. Open a web browser and enter your router's IP address (often `192.168.1.1` or `192.168.0.1`).
  2. Log in with your admin credentials (check the label on the router or its manual if unsure).
- **View connected devices:**
  - Look for a section like **Connected Devices**, **DHCP Clients**, or **Device List**. It will display the local IP addresses of all connected devices.

---

### **2. Using Command-Line Tools**
#### **On Linux or macOS**
- **Find the network range:**
  ```bash
  ip route
  ```
  Look for a line like `192.168.1.0/24` (this is your subnet).
- **Scan the network with `nmap`:**
  If you have `nmap` installed:
  ```bash
  sudo nmap -sn 192.168.1.0/24
  ```
  This will list all active devices on your network with their IPs and hostnames.

  If `nmap` isn't installed:
  ```bash
  sudo apt install nmap   # On Ubuntu-based systems
  sudo pacman -S nmap     # On Arch-based systems
  ```

#### **On Windows**
- **Find your network range:**
  1. Open Command Prompt (Win + R, type `cmd`, and press Enter).
  2. Run:
     ```cmd
     ipconfig
     ```
     Note the `Subnet Mask` and `Default Gateway`.
- **Scan the network with `arp`:**
  Run:
  ```cmd
  arp -a
  ```
  This lists the IP addresses of devices connected to your network.

---

### **3. Using Mobile Apps**
- Use network scanning apps like **Fing** or **PingTools** (available for Android and iOS). They can scan your home network and display all connected devices.

---

### **4. Using Python (Optional)**
If you're comfortable with Python, you can create a simple script:
```python
import os

subnet = "192.168.1."  # Replace with your subnet
for i in range(1, 255):
    response = os.system(f"ping -c 1 {subnet}{i} > /dev/null 2>&1")
    if response == 0:
        print(f"Device found: {subnet}{i}")
```
Run this script to find all active devices.

---

### **Notes:**
- Ensure all PCs are powered on and connected to the network.
- For wireless devices, confirm they're connected to the same Wi-Fi network.
- Static IP assignments (configured manually) won't always show up in the DHCP client list of the router.

Let me know if you need help with any specific method!

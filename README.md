# RFD-Pi-Nodes

Objective: Setup two or more Pi 5 devices with RFD 900 Radios to send basic long range communication between each node.


Needs:
- Two or more Pi 5's with your OS of choice (I used Pi OS)
- Power supplies for the Pis
- Setup SSH connection between Pis over the home network. 

  Detailed SSH setup
  1. Enable SSH on each Pi
     - On Raspberry Pi OS: run sudo raspi-config -> Interfacing Options -> SSH -> Enable
     - Or, if you have an SD card mounted on another machine, create an empty file named ssh in the /boot partition to enable SSH on first boot.

  2. Ensure network connectivity and find the Pi's IP address
     - Connect Pis to your home network (Ethernet or Wi‑Fi).
     - Use your router's admin page or run ip a on the Pi to find its IP address (e.g. 192.168.1.42).
     - You can also use mDNS: ping raspberrypi.local (or the hostname you set) from another machine.

  3. Update the system (recommended)
     sudo apt update && sudo apt upgrade -y

  4. Create an SSH key pair on your workstation (if you don't already have one)
     - Use a modern key type, for example Ed25519:
       ssh-keygen -t ed25519 -C "pi@yourdomain" -f ~/.ssh/id_ed25519
     - Press Enter to accept defaults and protect the key with a passphrase if desired.

  5. Copy your public key to each Pi for passwordless login
     - Use ssh-copy-id (recommended):
       ssh-copy-id -i ~/.ssh/id_ed25519.pub pi@<PI_IP_ADDRESS>
     - Or manually append the public key to ~/.ssh/authorized_keys on the Pi and set permissions:
       mkdir -p ~/.ssh && chmod 700 ~/.ssh
       echo "<your-public-key>" >> ~/.ssh/authorized_keys
       chmod 600 ~/.ssh/authorized_keys

  6. Test SSH login
     ssh -i ~/.ssh/id_ed25519 pi@<PI_IP_ADDRESS>
     - If you used ssh-copy-id and default keys, ssh pi@<PI_IP_ADDRESS> should connect without a password.

  7. Optional: simplify access with ~/.ssh/config on your workstation
     Host pi-node1
       HostName 192.168.1.42
       User pi
       IdentityFile ~/.ssh/id_ed25519

     Then connect with: ssh pi-node1

  8. Optional: harden SSH (enable only key auth)
     - Edit /etc/ssh/sshd_config on the Pi and set:
       PasswordAuthentication no
       PermitRootLogin no
     - Restart SSH: sudo systemctl restart ssh
     - Keep an active session open while testing so you don't lock yourself out.

  9. Troubleshooting tips
     - If you get "Permission denied": check that permissions on ~/.ssh (700) and ~/.ssh/authorized_keys (600) are correct.
     - If connection times out: confirm the Pi has the IP you expect, check network/cables, and verify ssh service: sudo systemctl status ssh.
     - Use verbose ssh output for debugging: ssh -vvv pi@<PI_IP_ADDRESS>

- We are going to write this project in C or Bash scripting


Wiring (example)

There are two common connection methods:

1. UART TTL connection (direct to Pi UART or via level shifter)
   - RFD TX -> Pi RX (GPIO 15 / UART RX)
   - RFD RX -> Pi TX (GPIO 14 / UART TX)
   - Ground -> Ground
   - Make sure voltage levels are compatible or use a level shifter.

2. USB connection
   - Use the radio's USB/serial adapter and connect to Pi USB port.
   - Identify the device (e.g., /dev/ttyUSB0 or /dev/ttyAMA0)

Always consult your radio's datasheet for power and voltage requirements.
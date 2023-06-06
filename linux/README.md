## Get MAC from default network interface

```bash
ip link show | grep -E "$(ip route | grep default | awk '{print $5}')" -A1 | awk '/ether/ {print $2}'
```

## Get ID from the SO installation drive

```bash
sudo hdparm -I $(df --output=source / | tail -1 | awk '{gsub(/[0-9]+$/, ""); print}') | grep "Serial Number" | xargs | awk '/:/ {print $3}'
```

**Kill process running in port**

This is useful to kill process when for example you close VS Code while running a NodeJs server and it do not closes well. The console shows an error something like “Another process is running that port” and you know that is the previous process without exiting.

1. You have to have lsof binary installed

   ```bash
   # Debian distributions
   sudo apt install lsof

   # RHEK/Centos
   dnf install lsof
   ```

2. Place this command into .zshrc

   ```bash
   function killPort {
       if [ -z $1 ]; then
           echo -e "\n[*] Usage: $0 {port}"
       fi
       echo -e "\n[*] Killing process related to port $1...\n"
       kill -9 $(lsof -i -P -n | grep $1 | xargs | awk {'print $2'}) 2> /dev/null
       echo "[*] Done"
   }
   ```

3. Reload terminal
4. Enjoy

   ```bash
   # Usage
   killPort 3000
   ```

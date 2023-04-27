## Get MAC from default network interface

```bash
ip link show | grep -E "$(ip route | grep default | awk '{print $5}')" -A1 | awk '/ether/ {print $2}'
```

## Get ID from the SO installation drive

```bash
sudo hdparm -I $(df --output=source / | tail -1 | awk '{gsub(/[0-9]+$/, ""); print}') | grep "Serial Number" | xargs | awk '/:/ {print $3}'
```

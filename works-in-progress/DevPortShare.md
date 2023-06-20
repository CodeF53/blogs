Web dev you will want to test on your phone.

You can do that by
1. allowing ports through firewall
2. forwarding network traffic to WSL

## Manual
### Firewall
Start by opening "Windows Defender Firewall with Advanced Security"

Click "Inbound Rules"

Click "New Rule"

Select "Port"

Select "TCP"

Allow specific ports, for example:\
- 3000 (Ruby on Rails server)
- 4000 (Vite)
- 5500-5520 (Live Server VSCode extension)

That would be formatted as 3000, 4000, 5500-5520

Select "Allow the connection"

Leave all boxes checked

Give it a good name

If you host your stuff in Windows and not WSL, you are already done.

### Forward to WSL
Get your current internet connection ip.
Open an Admin command prompt.

Run `ipconfig`, you want the value after `IPv4 Address`. It typically starts with `10.` or `172.`

Repeat this command for all the ports you opened in firewall, replacing `LocalIP` and `PORT` with relevant values
```
netsh interface portproxy add v4tov4 listenport=PORT listenaddress=LocalIP connectport=PORT connectaddress=127.0.0.1
```

## Automating WSL forwarding
If your IP changes frequently because you move networks or don't have a static IP assigned on your preferred network, doing the last step can get tedious.

### Script to auto-adjust ports
Somewhere on your computer dedicated to scripts, create a new file called `update-portforwarding.ps1`, replacing WslIP with your WSLIP

```ps1
# Retrieve the current Wi-Fi IPv4 address
$wifiIPv4 = (Get-NetIPAddress -InterfaceAlias Wi-Fi -AddressFamily IPv4).IPAddress

# Define the ports you want to forward
$ports = @(3000, 4000, 5500)

# Remove existing port forwarding rules
netsh interface portproxy reset

# Create new port forwarding rules with the updated Wi-Fi IPv4 address
foreach ($port in $ports) {
  netsh interface portproxy add v4tov4 listenport=$port listenaddress=$wifiIPv4 connectport=$port connectaddress=127.0.0.1
```

I was gonna say add it to Task Scheduler, but nah, just put your scripts folder in path and run it when you need it.
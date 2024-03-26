# wgjailr
This is a very simple script to help with network namespaces. There are also some systemd unit files included to help
run the script as a service.

## Dependencies
* linux distribution with network namespaces
* systemd
* `socat` package
* `iproute2` packages
* `wireguard-tools` packages

## Installation
Copy `bin/wgjailr` to `/usr/bin` and make sure it is executable:
```
sudo chmod +x bin/wgjailr
sudo cp bin/wgjailr /usr/bin
```

## Usage
The script has three main commands that can be executed: `up`, `down`, and `e`.

The `up` command creates a new network namespace and adds a loopback interface, a VPN interface, and sets the default route in the new namespace. It also sets the `resolv.conf` in the network namespace to the specified DNS server.

The `down` command deletes the network namespace and cleans up the resolvconf.

The `e` command allows executing a command in the network namespace as the current user.

## Script Configuration

The following variables are configurable, just change them at the top of the script:
* `NETWORK_NAMESPACE_NAME`: The name of the network namespace. Default is `vpn`.
* `VPN_CONFIG_PATH`: The path to your VPN configuration file. Default is `/opt/wireguard.conf`.
* `VPN_DNS_SERVER`: The DNS server to be used in the network namespace. If not set, will be read from the wireguard .conf file.
* `VPN_INTERFACE_NAME`: The name of the VPN interface in the network namespace. Default is `tun0`.
* `VPN_LOCAL_IP`: The local IP address assigned to the VPN interface in the network namespace. If not set, will be read from the wireguard .conf file.

## Systemd Unit Files
In addition to the shell script, there are several Systemd unit files that are used to run the script on system start-up, forward a port from the root network namespace to the namespaced network, and run a service in the network namespace, effectively jailing the service to the network namespace.
* `netns-vpn.service`: This unit file is used to create the network namespace and set up the VPN interface. The ExecStart and ExecStop directives runs the `up` and `down` commands in the `wgjailr` script.
* `netns-vpn-port-forward-@.service`: This unit file is used to forward a port from the root network namespace to the namespaced network. The `ExecStart` directive uses the `socat` utility to listen on a specified port and forward the connection to the namespaced network.
* `qBittorrent-nox.service`: This unit file is an example of how to run a service, in this case `qBittorrent-nox`, in the network namespace. The `ExecStart` directive runs the `e` command in the `wgjailr` script, followed by the qBittorrent command line client.

To use these unit files, you must copy them to the `/etc/systemd/system/` directory.

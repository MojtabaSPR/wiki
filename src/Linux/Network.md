# Network

<details>
<summary>Check IP</summary>

**Different ways to check ip addresses:**
```bash
 ~: ifconfig
 ~: ifconfig ens33
 ~: ip addr show
 ~: ip addr show ens33
 ~: cat /etc/netplan/*.yml
```    
**A brief way:**
```bash
 ~: networkctl status
```
</details>

<br/>

<details>
<summary>Default Gateway</summary>

**To check routes and DG:**
```bash
 ~: route -n
 
 ~: networkctl status
 
 ~: ip route list
```
 
 **How to change DG:**
 1. From /etc/netplan/*.yml file
 2. From ***ip route*** command
 
 **Change DG from ip command:**
 ```bash
 ip route add default via 192.168.1.1
 ```
 
 ```bash
 route add default gw 192.168.1.1
 ```
 
 **Delete A route:**
 ```bash
 sudo ip route del default via 192.168.1.1
 ```
</details>

<br/>

<details><summary>DNS</summary>
    
**To check dns servers:**
```bash
cat /etc/resolve.conf
```

در فایل بالا قبلا می توانستیم آیپی دی ان اس سرورهایمان را معرفی کنیم اما در نسخه های جدیدتر لینوکس این فایل توسط پکیج و دستور زیر مدیریت می شود و نباید این فایل را تغییر داد.

**systemd-resolve:**
```bash
systemd-resolve —status
```

This command will give dns servers and status per interface.

```bash
networkctl status
```
</details>

<br/>

<details><summary>DHCP</summary>

To Release current ip:
```bash
sudo dhclient -v -r
```
To get new ip address from dhcp server:
```bash
sudo dhclient -v 
```
If the nic already has an ip the above command will add new address to the nic, So if you want to renew you should first run the ***dhclient*** command with ***-r*** option.
</details> 

<br/>

<details><summary>RHEL</summary>

***IP Address configuration:***

 *If the NetworkManager service is running, this command displays the state of the system's physical network interfaces,*
```bash
nmcli device status
```

```bash
ip addr show
ip addr show dev eth1
ifconfig 
```
    
***Add Static or DHCP address:***

1. *Edit network file*
```bash
vim /etc/sysconfig/network For old versions
```

*2.Edit below file and set ip and gateway*
```bash
vim /etc/sysconfig/network-scripts/ifcfg-eth0
```

*3.Set DNS Server configuration*
```bash
vim /etc/resolv.conf
```

*4.Restart network service*
```bash
/etc/init.d/network restart [On SysVinit]

systemctl restart network [On SystemD]
```
</details>

<br/>

<details><summary>Static/Dynamic IP</summary>

In new version of ubuntu the netplan package and method is being used to manage networking.

There is also two rendering system that linux or netplan are using to manage the networking and they are:

systemd-networkd , NetworkManager

By default on server edition the systemd-networkd is being used.

The ip address configurations are saved in some yaml files in below directory: /etc/netplan

By default there is one yaml file for all network adapters by the name e.g: ***50-cloud-init.yaml***

**Rendering method:**

In this yaml file the rendering method is defined. for example for NetworkManager we have:

```bash
cat /etc/netplan/50-cloud-init.yaml
```

```bash
network:
	version: 2
	renderer: NetworkManager
```

If the rendere is not mentioned by default systemd-networkd will be used.

**To check either dhcp or static method:**

```bash
cat /etc/netplan/50-cloud-init.yaml
```


<details><summary>DHCP Mode Example</summary> 

```bash
sudo vim /etc/netplan/50-cloud-init.yaml
```

```bash
network:
	ethernets:
		ens33:
			dhcp4: true
	version: 2
```

```bash
sudo netplan apply
```
</details> <!-- End 1 -->


<details>
<summary>Static IP Example</summary>

```bash
sudo vim /etc/netplan/50-cloud-init.yaml
```

```bash
network: 
	version: 2 
	renderer: networkd 
	ethernets: 
		enp3s0: 
			addresses:
				- 192.168.174.10/24
			gateway4: 192.168.174.1
			nameservers: 
				search: [mydomain, otherdomain] 
				addresses: [192.168.174.1, 1.1.1.1]
```

```bash
sudo netplan apply
```
</details><!-- End 1 -->
</details>

<br/>

<details><summary>networkd</summary>

systemd-networkd is a system daemon that manages network configurations. It detects and configures network devices as they appear; it can also create virtual network devices. This service can be especially useful to set up complex network configurations for a container managed by systemd-nspawn or for virtual machines. It also works fine on simple connections.

networkd is part of Systemd. In other word systemd-networkd used to manage network connections using Systemd. On the other hand NetworkManager is a GUI tool for configuring networking options.

You can tell Netplan to use NetworkManager and it is useful for Linux desktop or laptop users. By default all network devices get handled by systemd-networkd. 

```bash
systemctl status systemd-networkd
```
</details>

<br/>

<details><summary>network manager</summary>
    
NetworkManager attempts to keep an active network connection available at all times.

The point of NetworkManager is to make networking configuration and setup as painless and automatic as possible. If using DHCP, NetworkManager is intended to replace default routes, obtain IP addresses from a DHCP server and change nameservers whenever it sees fit. In effect, the goal of NetworkManager is to make networking Just Work.

Whilst it was originally targeted at desktops, it has more recently been chosen as the default network management software for some non-Debian server-oriented Linux distributions. If you have special needs, the upstream's developers would like to hear about them, but understand that NetworkManager is not intended to serve the needs of all users.

```bash
systemctl status network-manager
```
</details>

<br/>

<details><summary>Netplan</summary>

- 01- Netplan
        
    The network configuration abstraction renderer
    
    Netplan is a utility for easily configuring networking on a linux system. You simply create a YAML description of the required network interfaces and what each should be configured to do. From this description Netplan will generate all the necessary configuration for your chosen renderer tool.
    
    ## How does it work?
    
    Netplan reads network configuration from `/etc/netplan/*.yaml` which are written by administrators, installers, cloud image instantiations, or other OS deployments. During early boot, Netplan generates backend specific configuration files in `/run` to hand off control of devices to a particular networking daemon.
    
    Netplan currently works with these supported renderers:
    
    - NetworkD
    - Network Manager
    
    ### Configuration
    
    Obviously, without configuration, netplan will not do anything. The most useful configuration snippet (to bring up things via dhcp) is as follows:
    
    ```bash
    network:
        version: 2
        renderer: NetworkManager
    ```
    
    This will make NetworkManager manage all devices (and by default, any ethernet device will come up with DHCP once carrier is detected).
    
    Using networkd as a renderer does not let devices automatically come up using DHCP; each interface needs to be specified in a file in `/etc/netplan` for its configuration to be written and for it to be used in networkd.
    
    Netplan uses a set of subcommands to drive its behavior:
    
    - **netplan generate**: Use `/etc/netplan` to generate the required configuration for the renderers.
    - **netplan apply**: Apply all configuration for the renderers, restarting them as necessary.
    
- 02- Examples | Netplan
    
    Below are a collection of example netplan configurations for common scenarios. If you see a scenario missing or have one to contribute, please file a bug against this documentation with the example using the links at the bottom of this page. Thank you!
    
    To configure netplan, save configuration files under `/etc/netplan/` with a `.yaml` extension (e.g. `/etc/netplan/config.yaml`), then run `sudo netplan apply`. This command parses and applies the configuration to the system. Configuration written to disk under `/etc/netplan/` will persist between reboots.
    
    ## Using DHCP and static addressing
    
    To let the interface named ‘enp3s0’ get an address via DHCP, create a YAML file with the following:
    
    ```bash
    network:
        version: 2
        renderer: networkd
        ethernets:
        enp3s0:
            dhcp4: true
    ```
    
    To instead set a static IP address, use the addresses key, which takes a list of (IPv4 or IPv6), addresses along with the subnet prefix length (e.g. /24). Gateway and DNS information can be provided as well:
    
    ```bash
    network:
        version: 2
        renderer: networkd
        ethernets:
        enp3s0:
            addresses:
                
            gateway4: 10.10.10.1
            nameservers:
                search: [mydomain, otherdomain]
                addresses: [10.10.10.1, 1.1.1.1]
    ```
    
    ## Connecting multiple interfaces with DHCP
    
    Many systems now include more than one network interface. Servers will commonly need to connect to multiple networks, and may require that traffic to the Internet goes through a specific interface despite all of them providing a valid gateway.
    
    One can achieve the exact routing desired over DHCP by specifying a metric for the routes retrieved over DHCP, which will ensure some routes are preferred over others. In this example, ‘enred’ is preferred over ‘engreen’, as it has a lower route metric:
    
    ```bash
    network:
        version: 2
        ethernets:
        enred:
            dhcp4: yes
            dhcp4-overrides:
            route-metric: 100
        engreen:
            dhcp4: yes
            dhcp4-overrides:
            route-metric: 200
    ```
    
    ## Connecting to an open wireless network
    
    Netplan easily supports connecting to an open wireless network (one that is not secured by a password), only requiring that the access point is defined:
    
    ```bash
    network:
        version: 2
        wifis:
        wl0:
            access-points:
            opennetwork: {}
            dhcp4: yes
    ```
    
    ## Connecting to a WPA Personal wireless network
    
    Wireless devices use the ‘wifis’ key and share the same configuration options with wired ethernet devices. The wireless access point name and password should also be specified:
    
    ## Connecting to WPA Enterprise wireless networks
    
    It is also common to find wireless networks secured using WPA or WPA2 Enterprise, which requires additional authentication parameters.
    
    For example, if the network is secured using WPA-EAP and TTLS:
    
    Or, if the network is secured using WPA-EAP and TLS:
    
    Many different modes of encryption are supported. See the [Netplan reference](https://netplan.io/reference) page.
    
    ## Using multiple addresses on a single interface
    
    The addresses key can take a list of addresses to assign to an interface:
    
    Interface aliases (e.g. eth0:0) are not supported.
    
    ## Using multiple addresses with multiple gateways
    
    Similar to the example above, interfaces with multiple addresses can be configured with multiple gateways.
    
    Given that there are multiple addresses, each with their own gateway, we do not specify `gateway4` here, and instead configure individual routes to 0.0.0.0/0 (everywhere) using the address of the gateway for the subnet. The `metric` value should be adjusted so the routing happens as expected.
    
    DHCP can be used to receive one of the IP addresses for the interface. In this case, the default route for that address will be automatically configured with a `metric` value of 100. As a short-hand for an entry under `routes`, `gateway4` can be set to the gateway address for one of the subnets. In that case, the route for that subnet can be omitted from `routes`. Its `metric` will be set to 100.
    
    ## Using Network Manager as a renderer
    
    Netplan supports both networkd and Network Manager as backends. You can specify which network backend should be used to configure particular devices by using the `renderer` key. You can also delegate all configuration of the network to Network Manager itself by specifying only the `renderer` key:
    
    ## Configuring interface bonding
    
    Bonding is configured by declaring a bond interface with a list of physical interfaces and a bonding mode. Below is an example of an active-backup bond that uses DHCP to obtain an address:
    
    Below is an example of a system acting as a router with various bonded interfaces and different types. Note the ‘optional: true’ key declarations that allow booting to occur without waiting for those interfaces to activate fully.
    
    ## Configuring network bridges
    
    To create a very simple bridge consisting of a single device that uses DHCP, write:
    
    A more complex example, to get libvirtd to use a specific bridge with a tagged vlan, while continuing to provide an untagged interface as well would involve:
    
    Then libvirtd would be configured to use this bridge by adding the following content to a new XML file under `/etc/libvirtd/qemu/networks/`. The name of the bridge in the tag as well as in need to match the name of the bridge device configured using netplan:
    
    ## Attaching VLANs to network interfaces
    
    To configure multiple VLANs with renamed interfaces:
    
    ## Reaching a directly connected gateway
    
    This allows setting up a default route, or any route, using the “on-link” keyword where the gateway is an IP address that is directly connected to the network even if the address does not match the subnet configured on the interface.
    
    For IPv6 the config would be very similar, with the notable difference being an additional scope: link host route to the router’s address required:
    
    ## Configuring source routing
    
    Route tables can be added to particular interfaces to allow routing between two networks:
    
    In the example below, ens3 is on the 192.168.3.0/24 network and ens5 is on the 192.168.5.0/24 network. This enables clients on either network to connect to the other and allow the response to come from the correct interface.
    
    Furthermore, the default route is still assigned to ens5 allowing any other traffic to go through it.
    
    ## Configuring a loopback interface
    
    Networkd does not allow creating new loopback devices, but a user can add new addresses to the standard loopback interface, lo, in order to have it considered a valid address on the machine as well as for custom routing:
    
    ## Integration with a Windows DHCP Server
    
    For networks where DHCP is provided by a Windows Server using the dhcp-identifier key allows for interoperability:
    
    ## Connecting an IP tunnel
    
    Tunnels allow an administrator to extend networks across the Internet by configuring two endpoints that will connect a special tunnel interface and do the routing required. Netplan supports SIT, GRE, IP-in-IP (ipip, ipip6, ip6ip6), IP6GRE, VTI and VTI6 tunnels.
    
    A common use of tunnels is to enable IPv6 connectivity on networks that only support IPv4. The example below show how such a tunnel might be configured.
    
    Here, 1.1.1.1 is the client’s own IP address; 2.2.2.2 is the remote server’s IPv4 address, “2001:dead:beef::2/64” is the client’s IPv6 address as defined by the tunnel, and “2001:dead:beef::1” is the remote server’s IPv6 address.
    
    Finally, “2001:cafe:face::1/64” is an address for the client within the routed IPv6 prefix:
    
- 03- Design | Netplan
    
    [https://netplan.io/design](https://netplan.io/design)
    
    There are central network config files for Snappy, Server, Client, MAAS, cloud-init in `/etc/netplan/*.yaml`. All installers only generate such a file, no /etc/network/interfaces any more. There is also a netplan command line tool to drive some operations.
    
    Systems are configured during early boot with a “network renderer” which reads `/etc/netplan/*.yaml` and writes configuration to /run to hand off control of devices to the specified networking daemon.
    
    - Any other configured devices get handled by networkd by default, unless explicitly marked as managed by a specific manager ([NetworkManager](https://wiki.ubuntu.com/NetworkManager))
    - Devices that are not covered by the network config do not get touched at all.
    
    ## Key goals
    
    - Usable in initramfs (few dependencies and fast)
    - No persistent generated config, only original YAML config
    - Default policy applies with no config file present
    - Parser supports multiple config files to allow applications (libvirt, lxd) to package up expected network config (virbr0, lxdbr0), or to change the global default policy to use [NetworkManager](https://wiki.ubuntu.com/NetworkManager) for everything.
    - Retains the flexibility to change backends/policy later or adjust to “apt purge network-manager” as generated configuration is ephemeral
    
    ## Requirements
    
    - Initial configuration of network on a host from an oracle: MAAS, Cloud, Installer (Subiquity/Ubiquity), WebDM
    - Machine-parseable, but human readable configuration: YAML
    - Allowing full control over links and ip configuration (link properties and ip network configuration options) bond-* bridge-* ipv6-* , etc
    - Mtu, wake-on-lan, ethtool settings
    - Rule-based application of configuration
    - Related to smart defaults, e.g. Ubuntu Core might want to always run DHCP on any on-board interface (it doesn’t care which one)
    - Smart defaults, with predictable fallback configuration
    - Support VMs in cloud (EC2, and Azure variants)
    - Support containers in LXD
    - Support servers booting on baremetal (ISO)
    - Support servers booting via MAAS
    - Automatic appropriate backend selection without any config, but pluggable (config.d) policy for desktops to use NM for everything
    - Compatibility with older images which have existing 70-persistent-net.rules or boot with “net.ifnames=0” or similar.
    
    ## Design overview
    
    ![https://assets.ubuntu.com/v1/a1a80854-netplan_design_overview.svg](https://assets.ubuntu.com/v1/a1a80854-netplan_design_overview.svg)
    
    ## Network Config format
    
    This is called “version 2”, as current MAAS/curtin already use a [different YAML format](http://curtin.readthedocs.io/en/latest/topics/networking.html) that is called “version 1”.
    
    ## General structure
    
    The top-level node is a network: mapping that contains “version: 2” (the YAML currently being used by curtin, MAAS, etc. is version 1), and then device definitions grouped by their type, such as “ethernets:”, “wifis:”, or “bridges:”. These are the types that our renderer can understand and are supported by our backends.
    
    Each type block contains device definitions as a map where the keys (called “configuration IDs”) are defined as below.
    
    ## Device Configuration IDs
    
    The key names below the per-device-type definition maps (like “ethernets:”) are called “ID”s. They must be unique throughout the entire set of configuration files. Their primary purpose is to serve as anchor names for composite devices, for example to enumerate the members of a bridge that is currently being defined.
    
    There are two physically/structurally different classes of device definitions, and the ID field has a different interpretation for each:
    
    *Examples: ethernet, wifi*
    
    These can dynamically come and go between reboots and even during runtime (hotplugging). In the generic case, they can be selected by “match:” rules on desired properties, such as name/name pattern, MAC address, driver, or device paths. In general these will match any number of devices (unless they refer to properties which are unique such as the full path or MAC address), so without further knowledge about the hardware these will always be considered as a group.
    
    With specific knowledge (taken from the admin, a gadget snap, etc.), or by using unique properties such as path or MAC, match rules can be written so that they only match one device. Then the “set-name:” property can be used to give that device a more specific/desirable/nicer name than the default from udev’s ifnames. Any additional device that satisfies the match rules will then fail to get renamed and keep the original kernel name (and dmesg will show an error).
    
    It is valid to specify no match rules at all, in which case the ID field is simply the interface name to be matched. This is mostly useful if you want to keep simple cases simple, and it’s how we have done network device configuration since the mists of time.
    
    If there are match: rules, then the ID field is a purely opaque name which is only being used for references from definitions of compound devices in the config.
    
    *Examples: veth, bridge, bond*
    
    These are fully under the control of the config file(s) and the network stack. I. e. these devices are being created instead of matched. Thus “match:” and “set-name:” are not applicable for these, and the ID field is the name of the created virtual device.
    
    ## Complex example
    
    This shows most available features that are planned for the initial 16.10 implementation:
    
    ```bash
    network:
        version: 2
    
        # if specified globally, can only realistically have that value, as
        # networkd cannot render wifi/3G. This would be shipped as a separate
        # config.d/ by desktop images
        # it can also be specified by-type or by-device
        renderer: network-manager
        ethernets:
        # opaque ID for physical interfaces with match rules
        # only referred to by other stanzas
        id0:
            match:
            macaddress: 00:11:22:33:44:55
        wakeonlan: true
        dhcp4: true
        addresses:
            - 192.168.14.2/24
            - 2001:1::1/64
        lom:
            # example for explicitly setting a backend (default would be networkd)
            match:
            driver: ixgbe
            # you are responsible for setting tight enough match rules
            # that only match one device if you use set-name
            set-name: lom1
            dhcp6: true
        switchports:
            # all cards on second PCI bus
            # unconfigured by themselves will be added to br0 below
            match:
            name: enp2*
            mtu: 1280
        wifis:
            all-wlans:
            # useful on a system where you know there is
            # only ever going to be one device
            match: {}
            access-points:
            "Joe's home":
                # mode defaults to "managed" (client), key type to wpa-psk
                password: "s3kr1t"
            # this creates an AP on wlp1s0 using hostapd
            # no match rules, thus ID is the interface name
            wlp1s0:
            access-points:
                "guest":
                mode: ap
                channel: 11
                # no WPA config implies default of open
        bridges:
            # the key name is the name for virtual (created) interfaces;
            # no 'match' or 'set-name' attributes are allowed.
            br0:
            # IDs of the components
            # switchports expands into multiple interfaces
            interfaces: [wlp1s0, switchports]
            dhcp4: true
            routes:
            - to: 0.0.0.0/0
            via: 11.0.0.1
            metric: 3
            nameservers:
            search: [foo.local, bar.local]
            addresses: [8.8.8.8]
    ```
    
    ## Commands
    
    `Generate`  Runs during early boot and will read config, and write files  `Apply`  Kicks the various backends to realize network config  `List` `Update` `Config`  **Set** Capture existing config on an interface into the equivalent YAML. **Show** Merge and display all the current available configuration on the system.  
    
    ## Use cases
    
    Some of the possible ways of using netplan are captured below.
    
    Cathy boots the latest Ubuntu Server Image on her Intel NUC via USB key. The NUC has both a wired ethernet port and a wireless NIC builtin and has added a third USB-NIC as well, though it’s not currently connected. During the installation, the installer presents Cathy with the 3 interfaces. Cathy selects the wired ethernet device; it was the only one with an IP configured via DHCP. After the installation is complete Cathy reboots her NUC and finds the following in `/etc/netplan/01-install.yaml:`
    
    ```bash
    network:
        version: 2
        ethernets:
        # configured by subiquity
        eno1:
            dhcp4: true
    ```
    
    The installer wrote out a config which included elements for the explicitly chosen/configured wired device. The other NICs are unconfigured, as the installer should not make any assumptions about their future usage or configuration.
    
    ### Data oracle (cloud-init, MAAS, gadget snap)
    
    A user creates a cloud-init userdata file for a cloud instance that will get two network cards attached. That cloud-init file contains a network definition that configures a bond device from these two.
    
    The gadget snap for a router specifies that all ethernet interfaces on the second PCI bus belong to the builtin switch, and configures a bridge for them. The other ethernet interface is renamed to “eth-uplink” to provide a more specific and useful name for it.
    
    In general there are many situations where users (via the installer), hardware vendors, or cloud management software can make assumptions about the nature, number, and purpose of network interfaces – these should be expressible in terms of specific interface names and vendor-provided configuration for them.
    
    Alice plugs a USB nic into her Intel NUC after installing Ubuntu Server. The default policy for server uses networkd, and does not attempt to do any networking configuration for devices that are not already present in `/etc/netplan/`.
    
    ### Using a previously configured device
    
    Sally plugs in USB NIC that previously had been configured and expects that it retains the same configuration despite being connected in a different port.
    
    On device removal, no config change has been made. If the server was rebooted, then when processing e/n/c, we will write out the config again in /run/network, including MATCH rules for .LINK to retain the name and likely the same MATCH rules .NETWORK to apply the same config.
    
    (Note: This behaviour is already how NM/networkd/ifupdown etc. behave – this use case is just for a complete behaviour description.)
    
    ### Update network config on package install
    
    lxd, libvirt, and others might include network config by shipping a /{etc,lib}/netplan/*.yaml snippet, for e. g. setting up a bridge (virbr0, lxdbr0).
    
    ### Device configuration removal
    
    Alice is done with the usb0 device that was added and now wants to purge the configuration from the system.
    
    ### Show current config
    
    Configuration is a merge of all `/etc/netplan/\*.yaml`. The `netplan config show` command will read the current on-disk configuration and merge any configs in `/etc/netplan/\*.yaml` merge them and produce a YAML formatted output to stdout.
    
    - [Use cases](https://netplan.io/design)
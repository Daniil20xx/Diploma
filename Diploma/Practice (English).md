### Table of Contents

- [[#Overview|Overview]]
- [[#Private Network A:|Private Network A:]]
	- [[#Private Network A:#MicroTik 1|MicroTik 1]]
	- [[#Private Network A:#Host 1 (Linux)|Host 1 (Linux)]]
- [[#Private Network B:|Private Network B:]]
	- [[#Private Network B:#MicroTik 2|MicroTik 2]]
	- [[#Private Network B:#Host 2 (Linux)|Host 2 (Linux)]]
- [[#MicroTik 1 and MicroTik 2 connection|MicroTik 1 and MicroTik 2 connection]]
		- [[#Host 2 (Linux)#MicroTik 1|MicroTik 1]]
		- [[#Host 2 (Linux)#MicroTik 2|MicroTik 2]]
		- [[#Host 2 (Linux)#Connection between MicroTik 1 and MicroTik 2|Connection between MicroTik 1 and MicroTik 2]]
		- [[#Host 2 (Linux)#Check that NAT is working|Check that NAT is working]]
- [[#Public Host|Public Host]]
- [[#Download Coturn on PublicHost|Download Coturn on PublicHost]]
- [[#Set Up Coturn|Set Up Coturn]]
- [[#Set up a STUN server|Set up a STUN server]]
	- [[#Set up a STUN server#Test the STUN Server from Host1|Test the STUN Server from Host1]]

### Overview
In the virtual environment, we will set up the following test stand:
![[Pasted image 20261101191238.png]]
### Private Network A:
#### MicroTik 1
MicroTik 1 is a router for the internal subnet, so it should have 2 interfaces:
1) the first one for the local network
2)  the second one for Internet access
To route traffic, we'll configure NAT

**MicroTik1**
Use following command to find out which interfaces **MikroTik1** has:
```python
interface/print
```

![[Pasted image 20261101150146.png]]

`ether1` -  our local network. In the MikroTik settings, this interface is assigned to the **host1** LAN segment.
![[Pasted image 20261101150804.png]]

`ether2` - public network. In the MikroTik settings, this interface belongs to the **network** LAN segment.
![[Pasted image 20261101150931.png]]

Add an address pool to the local network interface and specify the gateway `192.168.1.1`.
```python
ip/address/add address=192.168.1.1/24 interface=ether1
ip/address/print
```

![[Pasted image 20261101151335.png]]

#### Host 1 (Linux)
In the virtual machine settings, the **Network Adapter** is set to **LAN Segment**, and the `host1` segment is selected.
![[Pasted image 20261101153639.png]]

Open the network connection settings:
![[Pasted image 20261101152935.png]]

Let's move on to the connection settings:
![[Pasted image 20261101152950.png]]

Open the IPv4 settings:
![[Pasted image 20261101153003.png]]

Select **"Manual"** for the IPv4 method.
**Address:** `192.168.1.10`
**Subnet mask:** `255.255.255.0`
**Gateway:** `192.168.1.1`

Apply the changes:
![[Pasted image 20261101153209.png]]

After that, restart the `NetworkManager` service:
![[Pasted image 20261101153709.png]]

Verify that a connection to the **MikroTik1** exists using the `ping` command:
![[Pasted image 20261101153812.png]]

![[Pasted image 20261101162202.png]]
In my case, the ping goes through, so everything is set up correctly.

### Private Network B:

#### MicroTik 2
**MicroTik2** is a router for the internal subnet, so it should have two interfaces:
1) the first for the local network
2) the second for Internet access
To route traffic, we'll configure NAT

As with MicroTik 1, we have two interfaces:
`ether1` - local network (host2).
`ether2` - public network (network).
```python
interface/print
```

![[Pasted image 20261101154342.png]]

Add an address pool to the `ether1` interface:
```python
ip/address/add address=192.168.1.1/24 interface=ether1
ip/address/print
```

![[Pasted image 20261101154609.png]]

#### Host 2 (Linux)
Let's make sure that the interface is configured correctly in the settings for the **Host 2** virtual machine:
![[Pasted image 20261101155047.png]]

Since we configured a static IP address on **Host 1** using the graphical interface, we'll do the same on **Host 2** using the terminal:

Use the `ip -c a` command to list the interfaces:
![[Pasted image 20261101155452.png]]
We're interested in interface 1s **ens33**. It currently has no IP address.

Open the `00-installer-config.yaml` file::
```bash
sudo nano /etc/netplan/00-installer-config.yaml 
```

Since we're using a static IPv4 address, we'll disable `dhcp` for both IPv4 and IPv6.
We'll add the address `192.168.1.10` and specify the subnet mask as `/24`.
Add gateway information using `routes`. Add information specifying which daemon should handle network management—in our case, **NetworkManager**.

The file should end up with the following content:
![[Pasted image 20261101161148.png]]

```python
network:
  ethernets:
    ens33:
      dhcp4: false
      dhcp6: false
      match:
        macaddress: 00:0c:29:a4:ff:53
      set-name: ens33
      addresses:
        - 192.168.1.10/24
      routes:
        - to: default
          via: 192.168.1.1
  version: 2
```
**!!! It is important that `macaddress` matches the MAC address of the network interface (use the `ip -c a` command)*

Apply the changes using the following command:
```bash
sudo netplan apply
```
![[Pasted image 20261101161206.png]]

Let's check to see if the IP address has appeared:
![[Pasted image 20261101161828.png]]

We verify that the MikroTik and Linux systems are connected:

Linux:
![[Pasted image 20261101161945.png]]

MicroTik:
![[Pasted image 20261101162040.png]]


### MicroTik 1 and MicroTik 2 connection

##### MicroTik 1
Add IP addresses to the `ether2` interface:
```python
ip/address/add address=203.0.113.1/24 interface=ether2
ip/address/print
```

![[Pasted image 20261101170155.png]]

Now, configure NAT:
```python
/ip/firewall/nat/add action=masquerade chain=srcnat out-interface=ether2
```

![[Pasted image 20261101171534.png]]

- `/ip/firewall/nat/add`: Add a new NAT rule.
- `action=masquerade`: Replace the packet's source IP address with the IP address of the outgoing interface. Change the port as well, if necessary.
- `chain=srcnat`: Apply the rule to outgoing traffic, that is, change the sender's address.
- `out-interface=ether2`: Apply the rule only to packets that exit through the ether2 interface.

##### MicroTik 2

Add IP addresses to the `ether2` interface:
```python
ip/address/add address=203.0.113.2/24 interface=ether2
ip/address/print
```

![[Pasted image 20261101172018.png]]

Now, configure NAT:

```python
/ip/firewall/nat/add action=masquerade chain=srcnat out-interface=ether2
```

![[Pasted image 20261101172059.png]]

##### Connection between MicroTik 1 and MicroTik 2
Verify that **MicroTik1** and **MicroTik2** are connected using the `ping` command:

**MicroTik1** ping:
![[Pasted image 20261101172653.png]]

**MicroTik2** ping:
![[Pasted image 20261101172725.png]]

The ping is successful, which means there is a connection.
##### Check that NAT is working

Let's try pinging **MicroTik2** from **Host1**, and then **MicroTik1** from **Host2**.

**Host1:**
![[Pasted image 20261101173008.png]]

**Host2:**
![[Pasted image 20261101173050.png]]
****Ping** is successful, which means **NAT** is working.


### Public Host

In the virtual machine settings, under **Network Adapter**, set the **LAN segment** to `network`:
![[Pasted image 20261101173559.png]]

In the **Public Host** settings, set a static address:

![[Pasted image 20261101173724.png]]

Let's try pinging **Microtik1** and **Microtik2**:
![[Pasted image 20261101173831.png]]

Let's try pinging **PublicHost** from **Host1**:
![[Pasted image 20261101173949.png]]

Let's try pinging **PublicHost** from **Host2**:
![[Pasted image 20261101174241.png]]

Let's try pinging **Host1**/**Host2** from **PublicHost**:
![[Pasted image 20261101174329.png]]
*Network Unreachable.*

### Download Coturn on PublicHost
To download the file, we'll need a real Internet connection.
To do this, go to the virtual machine settings and add another network adapter:
![[Pasted image 20261101174645.png]]

Let's leave it in NAT mode:
![[Pasted image 20261101174724.png]]


Inside the virtual machine, we verify that we have Internet access by running a `ping` command:
```bash
ping 8.8.8.8
```

![[Pasted image 20261101174810.png]]

Now, install `coutun`:
```bash
sudo apt update
sudo apt install coturn
```

![[Pasted image 20261101175157.png]]

After successful installation, we can remove the network adapter:
![[Pasted image 20261101175354.png]]

### Set Up Coturn
Let's configure `turnserver` to start automatically as a system service when Ubuntu boots. To do this, edit the `/etc/default/coturn` file and uncomment the following line:
![[Pasted image 20261101180025.png]]

### Set up a STUN server

On **PublicHost**, open `/etc/turnserver.conf` and add the following settings:

```python
listening-ip=203.0.113.10
listening-port=3478
fingerpring
```

``listening-ip`: Specify the IP address from which we will accept requests.
`listening-port`: Specify the port on which requests will be received

Open the ports in the firewall:
```bash
sudo ufw allow 3478/udp
sudo ufw allow 3478/tcp
```

![[Pasted image 20261101181339.png]]

Restart `coturn` using the following command:
```bash
sudo systemctl restart coturn
```

#### Test the STUN Server from Host1
In the virtual machine settings, add a network adapter of the NAT type:
![[Pasted image 20261101182248.png]]

Install **coturn**:
```bash
sudo apt update
sudo apt install coturn
```

![[Pasted image 20261101182549.png]]

Removing a network adapter:
![[Pasted image 20261101182638.png]]

Send a request to an external server (STUN):
```bash
turnutils_stunclient -p 3478 203.0.113.10
```

![[Pasted image 20261101182912.png]]
We can see that the **STUN server** provided us with our public IP address and the port assigned by the NAT configured on **MicroTik1**.

### Set up a TURN server

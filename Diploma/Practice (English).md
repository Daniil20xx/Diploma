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
- [[#Installing Wireshark|Installing Wireshark]]
- [[#Analyze Packets Using Wireshark (STUN)|Analyze Packets Using Wireshark (STUN)]]
	- [[#Analyze Packets Using Wireshark (STUN)#Analyze STUN|Analyze STUN]]
		- [[#Analyze STUN#Binding Request|Binding Request]]
			- [[#Binding Request#Message Type|Message Type]]
			- [[#Binding Request#Message Length|Message Length]]
			- [[#Binding Request#Message Cookie|Message Cookie]]
			- [[#Binding Request#Transaction ID|Transaction ID]]
		- [[#Analyze STUN#Binding Success Response|Binding Success Response]]
			- [[#Binding Success Response#Attributes|Attributes]]
- [[#Set up a TURN server|Set up a TURN server]]
- [[#Analyze Packets Using Wireshark (TURN)|Analyze Packets Using Wireshark (TURN)]]
- [[#Other setup (JISTI) not complete, does not good solution to analyze turn.|Other setup (JISTI) not complete, does not good solution to analyze turn.]]


### Overview
In the virtual environment, we will set up the following test stand:
![Pasted image 20261101191238](images/Pasted%20image%2020261101191238.png)
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

![Pasted image 20261101150146](images/Pasted%20image%2020261101150146.png)

`ether1` -  our local network. In the MikroTik settings, this interface is assigned to the **host1** LAN segment.
![Pasted image 20261101150804](images/Pasted%20image%2020261101150804.png)

`ether2` - public network. In the MikroTik settings, this interface belongs to the **network** LAN segment.
![Pasted image 20261101150931](images/Pasted%20image%2020261101150931.png)

Add an address pool to the local network interface and specify the gateway `192.168.1.1`.
```python
ip/address/add address=192.168.1.1/24 interface=ether1
ip/address/print
```

![Pasted image 20261101151335](images/Pasted%20image%2020261101151335.png)

#### Host 1 (Linux)
In the virtual machine settings, the **Network Adapter** is set to **LAN Segment**, and the `host1` segment is selected.
![Pasted image 20261101153639](images/Pasted%20image%2020261101153639.png)

Open the network connection settings:

![Pasted image 20261101152935](images/Pasted%20image%2020261101152935.png)

Let's move on to the connection settings:

![Pasted image 20261101152950](images/Pasted%20image%2020261101152950.png)

Open the IPv4 settings:

![Pasted image 20261101153003](images/Pasted%20image%2020261101153003.png)

Select **"Manual"** for the IPv4 method.
**Address:** `192.168.1.10`
**Subnet mask:** `255.255.255.0`
**Gateway:** `192.168.1.1`

Apply the changes:

![Pasted image 20261101153209](images/Pasted%20image%2020261101153209.png)

After that, restart the `NetworkManager` service:

![Pasted image 20261101153709](images/Pasted%20image%2020261101153709.png)

Verify that a connection to the **MikroTik1** exists using the `ping` command:

![Pasted image 20261101153812](images/Pasted%20image%2020261101153812.png)

![Pasted image 20261101162202](images/Pasted%20image%2020261101162202.png)

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

![Pasted image 20261101154342](images/Pasted%20image%2020261101154342.png)

Add an address pool to the `ether1` interface:
```python
ip/address/add address=192.168.1.1/24 interface=ether1
ip/address/print
```

![Pasted image 20261101154609](images/Pasted%20image%2020261101154609.png)

#### Host 2 (Linux)
Let's make sure that the interface is configured correctly in the settings for the **Host 2** virtual machine:

![Pasted image 20261101155047](images/Pasted%20image%2020261101155047.png)

Since we configured a static IP address on **Host 1** using the graphical interface, we'll do the same on **Host 2** using the terminal:

Use the `ip -c a` command to list the interfaces:

![Pasted image 20261101155452](images/Pasted%20image%2020261101155452.png)

We're interested in interface 1s **ens33**. It currently has no IP address.

Open the `00-installer-config.yaml` file:

```bash
sudo nano /etc/netplan/00-installer-config.yaml 
```

Since we're using a static IPv4 address, we'll disable `dhcp` for both IPv4 and IPv6.
We'll add the address `192.168.1.10` and specify the subnet mask as `/24`.
Add gateway information using `routes`. Add information specifying which daemon should handle network management - in our case, **NetworkManager**.

The file should end up with the following content:

![Pasted image 20261101161148](images/Pasted%20image%2020261101161148.png)

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
![Pasted image 20261101161206](images/Pasted%20image%2020261101161206.png)

Let's check to see if the IP address has appeared:
![Pasted image 20261101161828](images/Pasted%20image%2020261101161828.png)

We verify that the MikroTik and Linux systems are connected:

Linux:
![Pasted image 20261101161945](images/Pasted%20image%2020261101161945.png)

MicroTik:
![Pasted image 20261101162040](images/Pasted%20image%2020261101162040.png)


### MicroTik 1 and MicroTik 2 connection

##### MicroTik 1
Add IP addresses to the `ether2` interface:
```python
ip/address/add address=203.0.113.1/24 interface=ether2
ip/address/print
```

![Pasted image 20261101170155](images/Pasted%20image%2020261101170155.png)

Now, configure NAT:
```python
/ip/firewall/nat/add action=masquerade chain=srcnat out-interface=ether2
```

![Pasted image 20261101171534](images/Pasted%20image%2020261101171534.png)

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

![Pasted image 20261101172018](images/Pasted%20image%2020261101172018.png)

Now, configure NAT:

```python
/ip/firewall/nat/add action=masquerade chain=srcnat out-interface=ether2
```

![Pasted image 20261101172059](images/Pasted%20image%2020261101172059.png)

##### Connection between MicroTik 1 and MicroTik 2
Verify that **MicroTik1** and **MicroTik2** are connected using the `ping` command:

**MicroTik1** ping:
![Pasted image 20261101172653](images/Pasted%20image%2020261101172653.png)

**MicroTik2** ping:
![Pasted image 20261101172725](images/Pasted%20image%2020261101172725.png)

The ping is successful, which means there is a connection.
##### Check that NAT is working

Let's try pinging **MicroTik2** from **Host1**, and then **MicroTik1** from **Host2**.

**Host1:**
![Pasted image 20261101173008](images/Pasted%20image%2020261101173008.png)

**Host2:**
![Pasted image 20261101173050](images/Pasted%20image%2020261101173050.png)
****Ping** is successful, which means **NAT** is working.


### Public Host

In the virtual machine settings, under **Network Adapter**, set the **LAN segment** to `network`:
![Pasted image 20261101173559](images/Pasted%20image%2020261101173559.png)

In the **Public Host** settings, set a static address:

![Pasted image 20261101173724](images/Pasted%20image%2020261101173724.png)

Let's try pinging **Microtik1** and **Microtik2**:
![Pasted image 20261101173831](images/Pasted%20image%2020261101173831.png)

Let's try pinging **PublicHost** from **Host1**:
![Pasted image 20261101173949](images/Pasted%20image%2020261101173949.png)

Let's try pinging **PublicHost** from **Host2**:
![Pasted image 20261101174241](images/Pasted%20image%2020261101174241.png)

Let's try pinging **Host1**/**Host2** from **PublicHost**:
![Pasted image 20261101174329](images/Pasted%20image%2020261101174329.png)
*Network Unreachable.*

### Download Coturn on PublicHost
To download the file, we'll need a real Internet connection.
To do this, go to the virtual machine settings and add another network adapter:
![Pasted image 20261101174645](images/Pasted%20image%2020261101174645.png)

Let's leave it in NAT mode:
![Pasted image 20261101174724](images/Pasted%20image%2020261101174724.png)


Inside the virtual machine, we verify that we have Internet access by running a `ping` command:
```bash
ping 8.8.8.8
```

![Pasted image 20261101174810](images/Pasted%20image%2020261101174810.png)

Now, install `coutun`:
```bash
sudo apt update
sudo apt install coturn
```

![Pasted image 20261101175157](images/Pasted%20image%2020261101175157.png)

After successful installation, we can remove the network adapter:
![Pasted image 20261101175354](images/Pasted%20image%2020261101175354.png)

### Set Up Coturn
Let's configure `turnserver` to start automatically as a system service when Ubuntu boots. To do this, edit the `/etc/default/coturn` file and uncomment the following line:
![Pasted image 20261101180025](images/Pasted%20image%2020261101180025.png)

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

![Pasted image 20261101181339](images/Pasted%20image%2020261101181339.png)

Restart `coturn` using the following command:
```bash
sudo systemctl restart coturn
```

#### Test the STUN Server from Host1
In the virtual machine settings, add a network adapter of the NAT type:
![Pasted image 20261101182248](images/Pasted%20image%2020261101182248.png)

Install **coturn**:
```bash
sudo apt update
sudo apt install coturn
```

![Pasted image 20261101182549](images/Pasted%20image%2020261101182549.png)

Removing a network adapter:
![Pasted image 20261101182638](images/Pasted%20image%2020261101182638.png)

Send a request to an external server (STUN):
```bash
turnutils_stunclient -p 3478 203.0.113.10
```

![Pasted image 20261101182912](images/Pasted%20image%2020261101182912.png)
We can see that the **STUN server** provided us with our public IP address and the port assigned by the NAT configured on **MicroTik1**.

### Installing Wireshark
Let's install Wireshark so we can reconstruct the chronology of connections and packet transmissions. The installation process for Wireshark is the same on all Ubuntu hosts.

```bash
sudo apt install wireshark
```

Select "Yes":

![Pasted image 20261109154746](images/Pasted%20image%2020261109154746.png)

### Analyze Packets Using Wireshark (STUN)
#### Analyze STUN
Use the `ip -c a` command to find out the interface name. In my case, it's `ens33`:

![Pasted image 20261109155344](images/Pasted%20image%2020261109155344.png)

Run **Wireshark** with administrator privileges:

![Pasted image 20261109155213](images/Pasted%20image%2020261109155213.png)

Select the `ens33` interface:

![Pasted image 20261109155634](images/Pasted%20image%2020261109155634.png)

Packet capture begins:

![Pasted image 20261109155706](images/Pasted%20image%2020261109155706.png)

In the new terminal, we send a request to the STUN server:

```bash
turnutils_stunclient -p 3478 203.0.113.10
```

![Pasted image 20261109155755](images/Pasted%20image%2020261109155755.png)

Go back to Wireshark and filter the packets by the server's IP and STUN:

```wireshark
ip.addr == 203.0.113.10
```

![Pasted image 20261109155956](images/Pasted%20image%2020261109155956.png)

There is two packages. Let's take a closer look at them:

##### Binding Request
The client sends the first packet. It is called `Binding Request`.
![Pasted image 20261109160334](images/Pasted%20image%2020261109160334.png)
![Pasted image 20261109161754](images/Pasted%20image%2020261109161754.png)

A STUN packet is structured as follows:
![Pasted image 20261109160917](images/Pasted%20image%2020261109160917.png)
Source: https://www.rfc-editor.org/info/rfc8489/#section-5

The first two bits, `00`, indicate that this is a STUN packet.

###### Message Type
The next 14 bits are the **STUN Message Type**. This is the message type. It is divided into a class (2 bits) and a method (12 bits).
![Pasted image 20261109165443](images/Pasted%20image%2020261109165443.png)
Source: https://www.rfc-editor.org/info/rfc8489/#section-5

**Classes**
Two bits of the class field determine the direction and nature of the transaction (whether it is a request, a response, or simply a notification):
    - **0b00** (Request): The client sends a request to the server and expects a mandatory response.
	- **0b01** (Indication): A message is sent in one direction without expecting a response (for example, to maintain a NAT session: Keep-Alive).
	- **0b10** (Success Response): The server confirms that the request was processed successfully and returns data (for example, your public IP).
    - **0b11** (Error Response): The server reports that something went wrong.

**Methods**
In pure STUN, there is only one basic method:
    - **0x001** (Binding): A request to determine the external address.

If STUN is extended to the TURN protocol (RFC 8656, used for relaying traffic through a proxy server), new methods are added:
    - **0x003** (Allocate): A request to allocate a relay address on the server.
	- **0x004** (Refresh): Extends the lifetime of an allocated address.
    - **0x006** (Send): Sends data through the server.
    - **0x007** (Data): Receives data from the server.
	- **0x009** (CreatePermission): Request permission to exchange data with a specific peer.

**Message Method Assignment**
This bit indicates whether we are using officially documented methods or specific, third-party methods.
A value of 0 indicates that we are using the official standard, while 1 indicates the opposite.

In our case, the Message Class is **00**, which means we sent a `Request` class.
Our method is `0x001`, i.e., `Binding`.
Our Message Method Assignment is 0, which means we are using the official standard; therefore, Wireshark has determined that we have a `Binding Request`.

###### Message Length
**Message Length** specifies the length of **only the additional information (attributes)** that follows immediately after the main section of the 20-byte header.
In our case, the length is 0, since we do not have a section containing additional information.

###### Message Cookie
A fixed value of `0x2112A442` in network byte order. This helps distinguish `STUN` from other protocols.

###### Transaction ID
A 96-bit identifier that uniquely identifies a request/response pair. This helps match request and response pairs.
The user generates this randomly and sends the request to the server. When the server sends a response, it uses the exact same transaction ID. When the user receives a response from the server, they can establish an exact match between the request and the response.

##### Binding Success Response
In the response, we can see that:
**Message Type**:
- Class: `0x10` - Success Response
- Method: `0x001` - Binding (this is the only one for STUN)
**Message Length**: is 36 bytes because the Attributes field has been added.
**Message Transaction ID**: `136137a7cf6942644904ef50`, which is the same as the Message Transaction ID in the request. In other words, the server sent back the same Message Transaction ID that the user generated.

![Pasted image 20261109171830](images/Pasted%20image%2020261109171830.png)

###### Attributes
An “Attributes” field has appeared in the response. Let's take a closer look at it:
![Pasted image 20261109172744](images/Pasted%20image%2020261109172744.png)
Source: https://www.rfc-editor.org/info/rfc8489/#section-14

At the top level, each attribute has a fixed structure:
- **Attribute Type** (16 bits / 2 bytes): Type identifier.
- **Length** (16 bits / 2 bytes): Length of the *Value* field only, in bytes.
- **Value** (variable length): Payload.

The 16-bit Attribute Type field contains important rules for the program:
- **Bit 0** (Comprehension): Determines what to do if the type is unknown.
	- 0 means “Required” - a mandatory attribute that must be read. If it cannot be read, the packet is considered corrupted.
    - 1 means “Optional” - an optional attribute that can be ignored; therefore, if it cannot be read, it can simply be skipped.
- **Bit 1** (Assignment):
    - 0 means “IETF Review” (an official standard from an RFC).
    - 1 means “Vendor-specific” (a custom attribute from a specific manufacturer).
- **Bits 2–15** (14 bits): The sequence number of the attribute itself.

2. Analyzing an Actual Dump from Wireshark

In our case, the server sent a response with two attributes:
1) The XOR-MAPPED-ADDRESS attribute
2) The SOFTWARE attribute

**XOR-MAPPED-ADDRESS Attribute**:
The attribute type `0x0020` appeared in the dump. In binary, this is `0000 0000 0010 0000`.
- **The very first bit is 0**, which means that this attribute **must be read**.
- **The second bit is 0**, which means that the attribute is an official RFC standard.
- **Bits 2–15** store the numeric code `0x0020`. For the program, this is a clear signal: “The port and IP address are masked inside.”
- **Length**: Equals 8. This means that the payload in the Value field occupies exactly 8 bytes.
- **Value**: Contains information about the IPv4 protocol version (`0x01`), as well as the port and IP address, which are hidden using an XOR mask.
- **Port Decryption**: The packet contains the bytes `e00b`. To determine the actual port, the program performs an XOR operation with the most significant bytes of the Magic Cookie (`0x2112`). The result of XORing `0xe00b` with `0x2112` yields the actual port **49433**.
- **IP Decryption**: The packet contains the bytes `ea12d543`. The program performs an XOR operation with the entire Magic Cookie (`0x2112A442`). The result of `0xea12d543` XOR `0x2112A442` yields the net IP address **203.0.113.1**.

**SOFTWARE Attribute:**
The attribute type `0x8022` appeared in the dump. In binary, this is `1000 0000 0010 0010`.
- **The very first bit is 1**, which means that this attribute **is optional**. If our application were older and didn’t recognize this attribute, it would simply ignore it and continue running.
- **The second bit is 0**, which means that the attribute is **an official RFC standard**.
- **Bits 2–15 store** the numeric code `0x0022`. Based on this code, the program recognizes that the field contains the text name of the software.
- **Length 20**: This means that the payload in the `Value` field occupies exactly 20 bytes.
- **Value**: Contains the plain text "Coturn-4.6.1 'Gorst'". Since the text is exactly 20 bytes long and is divisible by 4 without a remainder, the server did not need to add empty bytes at the end for padding.

### Set up a TURN server

Just as you downloaded the **cuturn** client on **host1** in the [Set Up Coturn on Host1](#Set%20Up%20Coturn%20on%20Host1) section, download the client on **host2**.

On **PublicHost**, we change the turn server configuration:

```bash
sudo nano /etc/turnserver.conf 
```

```conf
listening-ip=203.0.113.10
listening-port=3478
fingerpring

# Add Relay port range 
min-port=49000
max-port=50000
lt-cred-mech
realm=testturnserver
```

![](images/Pasted%20image%2020261110172403.png)

We've added a couple of new fields:
**Min-port** and **Max-port** define the range of ports that will be assigned to users.

**lt-cred-mech** stands for **Long-Term Credential Mechanism**. Simply put: it involves verifying the client’s username and password when connecting to the TURN server.

**realm** is the authentication realm. Simply put, it’s the name of the realm to which the credentials belong.

Next, we update the firewall rules to allow traffic to the appropriate ports.

```bash
sudo ufw allow 49000:50000/udp
```

![](images/Pasted%20image%2020261110162214.png)

Now let's add some users.

```bash
sudo turnadmin -a -u host1 -p password123 -r testturnserver
sudo turnadmin -a -u host2 -p password123 -r testturnserver
```

![](images/Pasted%20image%2020261110172733.png)
![](images/Pasted%20image%2020261110172928.png)

Restart the `coturn` service:

``` bash
sudo systemctl restart coturn
```

![](images/Pasted%20image%2020261110162354.png)

We verify that the service has started without errors and is running:

![](images/Pasted%20image%2020261110162528.png)

To test whether the TURN server is working properly, we'll use the same utility, `turnutils_uclient`- this command simulates a call or chat and begins sending TURN packets continuously.

```bash
turnutils_uclient -y -u host1 -w password123 -p 3478 203.0.113.10
```

Let's break down what's happening here:
The `-y` flag indicates that you don't need to specify who to send packets to. The utility will automatically create a pair of clients and connect them via TURN. Traffic is guaranteed to go through the relay in both directions.

`-u host1 -w password123` specifies the user account created on the TURN server.

`-p 3478 203.0.113.10` specifies the port and address of the TURN server.

![](images/Pasted%20image%2020261111182405.png)

Everything is working.

### Analyze Packets Using Wireshark (TURN)

Launch **Wireshark** on **Host1** and select the internal interface:

![](images/Pasted%20image%2020261111182539.png)

In the terminal, run the `turnutils_uclient` utility in **client-to-client** mode.

![](images/Pasted%20image%2020261111182731.png)

Let's filter packets based on the public server's IP address:

```wireshark
ip.addr == 203.0.113.10
```

![](images/Pasted%20image%2020261111182959.png)

It appears that the first request was an `Allocate Request`.

### Other setup (JISTI) not complete, does not good solution to analyze turn.

We'll check the availability of the **TURN server** using the **Jisti** app.

We're deploying **Jisti** on **PublicHost**. I'll be installing it via Docker, so first let's install the necessary packages:

```bash
sudo apt update
sudo apt install docker.io docker-compose-v2 curl unzip
sudo systemctl enable docker
sudo systemctl start docker
```

![](images/Pasted%20image%2020261111170417.png)

Create a folder and download the latest version of Jisti:
```bash
mkdir ~/jisti
cd ~/jisti
```

```bash
wget -q -O - https://api.github.com/repos/jitsi/docker-jitsi-meet/releases/latest > jisti.json
wget $(cat jisti.json | grep zip | cut -d '"' -f 4)
rm jisti.json
```

After that, we end up with an archive named **jisti**:

![](images/Pasted%20image%2020261111170913.png)

Extract the archive:

```bash
unzip ./*
```

![](images/Pasted%20image%2020261111171018.png)

Let's create a configuration:

```bash
cd jitsi-docker-jitsi-meet-*
cp env.example .env
./gen-passwords.sh
```

`./gen-passwords.sh`: automatically generates internal passwords for **Jitsi** components.

Now, edit the configuration:

```bash
nano .env
```

```bash
PUBLIC_URL=https://203.0.113.10:8443
ENABLE_LETSENCRYPT=0
JVB_ADVERTISE_IPS=203.0.113.10
```

`ENABLE_LETSENCRYPT=0` disables the retrieval of a **Let's Encrypt** certificate. For an isolated lab environment, we use a standard self-signed certificate.

`JVB_ADVERTISE_IPS` specifies the address that Jitsi Videobridge will provide to clients for transmitting media traffic.

---

.... Add information about starting Docker and additional settings (No, because a problem came up, and I didn't write it down until I fixed it so I wouldn't lose track of it)

I thought about using `jisti` as an example of a TURN, where traffic would be forwarded using `coturn`, but I couldn’t get it to work that way. Besides, I found an easier way to analyze TURN traffic using the `turnutils_...` utility

---

On Host1, open the website:

```
https://203.0.113.10:8443/
```

![](images/Pasted%20image%2020261111173808.png)

Create a room and join it:

![](images/Pasted%20image%2020261111173855.png)

On **Host2**, we connect to the room:

![](images/Pasted%20image%2020261111173957.png)

As you can see, both participants are in the same room:

![](images/Pasted%20image%2020261111174047.png)



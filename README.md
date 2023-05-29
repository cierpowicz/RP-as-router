# How to use a Raspberry Pi to provide WiFi for Ethernet-only devices


![image](https://github.com/cierpowicz/RP-as-router/assets/106453032/12c24901-35b9-4798-9179-f4489f416510)

![image](https://github.com/cierpowicz/RP-as-router/assets/106453032/353eb2b9-6278-4d71-a7c5-ac5e3915cb15)


This project is about to create secure playground for network testing (difrent type of routing like OSPF RIP EIGRP etc.) and also last but not least the main goal is to take control on MY OWN old laptop Dell. E6320 witf Win7 6.1 SP1  [here](https://github.com/cierpowicz/Win7-testing) 

* [Step By Step](#step-by-step)
* [Troubleshooting](#troubleshooting)
* [Resources](#resources)

## Step By Step

### Step 0: Configure Pi

Alright I guess that you have runing RaspberryPi connectet to Wifi. Befour start lets login in to our Pi using SSH from my main computer. To do this we have to enable SSH service on Pi.
```
sudo raspi-config
```
![image](https://github.com/cierpowicz/RP-as-router/assets/106453032/0d5d83b6-ee26-47dd-8306-5efdb4df734e)

![image](https://github.com/cierpowicz/RP-as-router/assets/106453032/2b326dff-8f95-4e6e-85ac-e7a1a78ed9c2)

Now Im eable to connect to pI using SSH from my PC

![image](https://github.com/cierpowicz/RP-as-router/assets/106453032/14c9e951-c5c7-44b1-91a6-b2a2715d4dfa)


### Step 1: Setup DHCP
Befour set DHCP we have to set static IP on our eth0 port. There is couple ways to do this but i will show you two options:

* (FIRST)Type in terminal:
  ``` sudo interface eth0 192.168.34.1/24```
* (SECOND)Edit file (REMEMBER hash(#) all lines except these two) :
  ``` sudo nano /etc/dhcpcd.conf```
  ![image](https://github.com/cierpowicz/RP-as-router/assets/106453032/814fadf7-918c-4174-9aec-fcf52ccf3a71)

Now lets download and instqall DHCP server:
``` sudo apt-get install isc-dhcp-server ```

Dont worry about this failer:

![image](https://github.com/cierpowicz/RP-as-router/assets/106453032/d206d38c-ead9-478f-8793-88c28a703f79)


Next edit ```/etc/dhcp/dhcpd.conf``` add on the down file:
``` authoritative;
subnet 192.168.34.0 netmask 255.255.255.0 {
 range 192.168.34.10 192.168.34.250;
 option broadcast-address 192.168.34.255;
 option routers 192.168.34.1;
 default-lease-time 600;
 max-lease-time 7200;
 option domain-name "local-network";
 option domain-name-servers 8.8.8.8, 8.8.4.4;
}
```

Now, edit ``` etc/default/isc-dhcp-server ``` ass bellow:

```INTERFACESv4="eth0"```

![image](https://github.com/cierpowicz/RP-as-router/assets/106453032/dcd20635-33a1-49c3-b923-005152e758ce)




### Step 2: configure IP forwarding

Now is necessary to configure IP forwarding. We want to forwart all traffic coming from eth0 to wlan0. Command bellow can do this ;) : 
```
sudo iptables -t nat -A POSTROUTING -o wlan0 -j MASQUERADE
sudo iptables -A FORWARD -i wlan0 -o eth0 -m state --state RELATED,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -i eht0 -o wlan0 -j ACCEPT
```
To make thous command live we have to add one more line in file ```/etc/sysctl.conf```

```net.ipv4.ip_forward=1```

### Step 3: Set the WiFi network as the main route

Just pass in the command line this: 

```
DEFAULT_IFACE=`route -n | grep -E "^0.0.0.0 .+UG" | awk '{print $8}'`
if [ "$DEFAULT_IFACE" != "wlan0" ]
then
  GW=`route -n | grep -E "^0.0.0.0 .+UG .+wlan0$" | awk '{print $2}'`
  echo Setting default route to wlan0 via $GW
  sudo route del default $DEFAULT_IFACE
  sudo route add default gw $GW wlan0
fi
```

### Step 4: configure one more file ```sh /etc/rc.local ```

Add in file ``` /etc/rc.local``` this line:

![image](https://github.com/cierpowicz/RP-as-router/assets/106453032/da8e77fd-db42-463d-988d-264c92d08421)

### Step 5: Test ``` ping 8.8.8.8```

Ok, now we kan test ```ping.8.8.8.8``` on DHCP Client:

![image](https://github.com/cierpowicz/RP-as-router/assets/106453032/7b649f4c-3d13-4ff5-924b-08b48416cde9)
 
 ITS WORK !!!

## Troubleshooting

In this lab i found only 3 issues.

1. SSH connection problem:
  ![image](https://github.com/cierpowicz/RP-as-router/assets/106453032/770bc050-d762-471a-8d7a-574a24bf37c3)
  Is because (i guess) RasberryPi after hard reset gets same IP from router. and during ssh conection i folder 
  ``` /etc/.ssh.known_hosts ``` found 2 ssh keys from same IP address !!!
2. After reboot the isc-dhcp-server service still dont work, is becouse you have to hash all lines in file
  ```/etc/dhcpcd.conf ``` is supostu looks like this: 
  ![image](https://github.com/cierpowicz/RP-as-router/assets/106453032/5bdeaadf-ee69-42d5-b42f-6d694d63c0b5)
3. On my target PC (Dell.) i have IP address from dhcp from RBpi BUT still cant ```ping 8.8.8.8```
   So you have to add one more line in  file on RBpi```/etc/rc.local```
   
   ``` iptables -t nat -A POSTROUTING -o wlan0 -j MASQUERADE```
  
  File supostu looks like this
  ![image](https://github.com/cierpowicz/RP-as-router/assets/106453032/395ac48f-dcb9-4347-b5b5-ff8defb6e127)

  
  


## Resources

* https://www.youtube.com/watch?v=TtLNue7gzZA
* https://www.youtube.com/watch?v=h0sR7tKuI-U&list=LL&index=2&t=83s
* https://gist.github.com/Konamiman/110adcc485b372f1aff000b4180e2e10












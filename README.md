<!-- ABOUT THE PROJECT -->
# problemy
1. pierwszy:
  ![image](https://github.com/cierpowicz/RP-as-router/assets/106453032/770bc050-d762-471a-8d7a-574a24bf37c3)
2. po reboot servwer dhcp dalej nie dizala 
  trzeba usunac wszystkie hasze z pliku /etc/dhcpcd.conf i zostawic tylko ta linijke co dodalismy LOL
3. Oczywiscie internet nie dziala. ale wystarczy fallow this tutrial i bedzie git. 
# BRUDNOPIS
RaspberrypiOS jest zainstalowany na swierzo

Po podczeniu do wifi naley jeszcze odpalic usluge SSH.
w tym celu nalezy:
 ```sh
   sudo raspi-config 
  ```
  ![image](https://github.com/cierpowicz/RP-as-router/assets/106453032/0d5d83b6-ee26-47dd-8306-5efdb4df734e)

  ![image](https://github.com/cierpowicz/RP-as-router/assets/106453032/2b326dff-8f95-4e6e-85ac-e7a1a78ed9c2)

Dobra wszyscy dadza rade izi pizi

### Use systemctl to Enable SSH

 ```sh
sudo systemctl enable ssh
```
 ```sh
   sudo systemctl start ssh
  ```
  
## Oki teraz mozemy wskoczyc na naszego komputerka

Teraz mozemy poczyc sie z Pi przez ssh:
```sh
   sudo raspi-config 
  ```
  
  tera trzeba skonfigurwac co nalezy
nalezy skonfigurowac plik /etc/dhcpcd.conf, ale zanim to zrobimy lepiej zrobic kopie pliku na wszyelki wypadek:
  ```sh
sudo cp /etc/dhcpcd.conf /etc/dhcpcd.conf.orginal 
  ```
Teraz przejzdzimy do konfiguracji pliku dhcpcd.conf
  ```sh
   sudo nano /etc/dhcpcd.conf
   ```
 Znajdz tera taki cos i zrob jak na focie:
 ![image](https://github.com/cierpowicz/RP-as-router/assets/106453032/d0a6d557-14cc-4126-99a0-87f0197ad3ae)

  
  Trzeba pobrac co potrzebne 
  
  ```sh
   sudo apt-get install isc-dhcp-server 
  ```
Po zainstalowaniu okaze sie ze jest lipa serwer nie dziala i spoko tak ma byc:
![image](https://github.com/cierpowicz/RP-as-router/assets/106453032/180d80fe-1d2c-4d40-945c-54957a8b6953)


Ale tera trzeba dokonczyc reszte konfiguracji jazda jazda: 

  Trzeba skonfigurowac plika /etc/dhcp/dhcpd.conf ale ofcors najpierw kopia x 
  
  ```sh
   sudo cp /etc/dhcp/dhcpd.conf /etc/dhcp/dhcpd.conf.orginal 
  ```
  Tera konfiguracja trzba wkleic NA DOLE do pliku /etc/dhcp/dhcpd.conf ponizsza configuracje,
   ```sh
   authoritative;
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
Tera konfigurujemy plika /etc/defoult/isc-dhcp-server
  
```sh
    INTERFACESv4="eth0"  
  ```
  
 ![image](https://github.com/cierpowicz/RP-as-router/assets/106453032/6463c204-6da8-4551-a31e-70a8d2c37598)

## Yoo colejny step jakies ip forwarding 

Tera trzeba przeforwardowac rucha

```sh
   sudo iptables -t nat -A POSTROUTING -o wlan0 -j MASQUERADE
  sudo iptables -A FORWARD -i wlan0 -o eth0 -m state --state RELATED,ESTABLISHED -j ACCEPT
  sudo iptables -A FORWARD -i eht0 -o wlan0 -j ACCEPT 
  ```

 Tera trzeba znow plika zaladowac: /etc/sysctl.conf jazda:
 
 ```sh
    net.ipv4.ip_forward=1 
  ```
## kolejny setup  ze niby wifi main route

zaladowac do konsoli:
 ```sh
    DEFAULT_IFACE=`route -n | grep -E "^0.0.0.0 .+UG" | awk '{print $8}'`
if [ "$DEFAULT_IFACE" != "wlan0" ]
then
  GW=`route -n | grep -E "^0.0.0.0 .+UG .+wlan0$" | awk '{print $2}'`
  echo Setting default route to wlan0 via $GW
  sudo route del default $DEFAULT_IFACE
  sudo route add default gw $GW wlan0
fi 
  ```
## Konicowka

Tera gdy odpalimy znow isc-dhcp-service to znow nie dziala a to dlatego ze trzeba usunac wszyskie hasze z 
pliku /etc/dhcpcd.conf i zstawic tylko to co dodalismy. 

![image](https://github.com/cierpowicz/RP-as-router/assets/106453032/d097bbb2-2107-4eec-a9f1-9dda98304b36)

tera mozna odpalac i powinno byc git

```sh
    sudo service isc-dhcp-server start
  ```
oczywiscie w moim przypadku cos nie dziala a to dlatego ze zapomnialem zeydtowac plika, etraz dziala: 
![image](https://github.com/cierpowicz/RP-as-router/assets/106453032/2a72add4-4fd7-4a91-91d7-f19793476a86)


## ALE CZY NAPEWNO ??? 

Wyglda na to ze dziala oto polecenie ifconfig na komputerku podlaczonym do RasberyPi: 
adres jaki dostalismy jak widac jest

## Teraz czy internet dziala ??? Oczywscie ze nie dziala 
nic dizwnego ze nie zawsze cos nie dziala ... 

aby to naprawic nalezy na rasbery pi zetytowac jeszcze jednego plika /etc/rc.local ale najpierw znobmy jego copie 

![image](https://github.com/cierpowicz/RP-as-router/assets/106453032/d5861e66-171b-4ac7-a5bd-b4296e22921d)

szybki reboot 

tera wszystko dzial 

![image](https://github.com/cierpowicz/RP-as-router/assets/106453032/49338227-6cad-44fd-8d6a-5256d5721196)



## About The Project


![image](https://github.com/cierpowicz/RP-as-router/assets/106453032/12c24901-35b9-4798-9179-f4489f416510)

__ Pomoce naukowe:
https://www.youtube.com/watch?v=h0sR7tKuI-U&list=LL&index=2&t=83s
https://gist.github.com/Konamiman/110adcc485b372f1aff000b4180e2e10
https://www.youtube.com/watch?v=TtLNue7gzZA
__
celem jest stworzenie bezpiecznego srodowiska do testowania sieci, analizy ruchu, odtworzanie roznych scenariuszy.


<p align="right">(<a href="#readme-top">back to top</a>)</p>

### Sprzecik

RasberyPi3 
ASUS RT-N65U
Dell Latitiude 15 E6320
<p align="right">(<a href="#readme-top">back to top</a>)</p>

## Getting Started


### konfiguracja RasberyPi 

Tutaj przygotujemy sobie nawszego RB zeby dizalalo
oto kilka pierwszy komend tak o zeby bylo ladnie :) 

Klasyk czyli update && ubgrade:
```sh
  sudo apt-get update
  sudo apt-get upgrade
```
Instalowanie niezbednych itemsow:
```sh
  sudo apt-get update
  sudo apt-get upgrade
```


### podlaczenie do wifi

Aby podlaczyc sie do wifi trzeba skonfigurowac nastepujace pliki:
/etc/cos
/etc/cos
/etc/cos

Aby podlaczyc sie do wifi podazaj dalej LUB [skorzystaj z poradnika](https://github.com/cierpowicz/RaspberryPi3-wifi-conection)

1. Step 1: Clone the repo 
   ```sh
   git clone https://github.com/your_username_/Project-Name.git
   ```
2. Step 2: run main.py
   ```sh
   python3 siemanko
   ```
4. Step 3: check if the ip is corect: 
   ```sh
   python3 siemanko
   ```
   
### glowna konfiguracja 



### test
Trzeba se spingowa
```sh
  sudo ping 8.8.8.8
```


## Conclusion
W powyzszym labolatorium korzystajac z RBpi3 i tak dalej 

## sesorces
- https://www.youtube.com/watch?v=CgPyTv5AAE0







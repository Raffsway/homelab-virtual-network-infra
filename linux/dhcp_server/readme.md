
## Linux Configuration for DHCP Server - Steps

1. **Set Static IP**  
   Edit the Netplan configuration file:
   ```bash
   sudo nano /etc/netplan/50-network-manage-all.yaml
2. **EDIT** the Netplan configuration file:
   ```yaml
   network:
     version: 2
     ethernets:
       eth0:
         dhcp4: no
         addresses:
           - 10.0.3.3/29
         routes:
           - to: default
             via: 10.0.3.1
         nameservers:
           addresses: [10.0.3.3, 8.8.8.8]
3. **Install dhcp Server**: 
    ```bash
    sudo apt-get install isc-dhcp-server
4. **Configure ``dhcpd.conf``** in ``sudo nano /etc/dhcp/dhcpd.conf``
-  ```bash
    # This is a very basic subnet declaration.
    # LAN-1
    option domain-name "homelab.net";  # Alterado para domínio criado no bind9
    option domain-name-servers 10.0.3.3;  # Apontando o Server Local
    subnet 10.0.1.0 netmask 255.255.255.0 {
        range 10.0.1.100 10.0.1.254;
        option subnet-mask 255.255.255.0;
        option routers 10.0.1.1;
        option broadcast-address 10.0.1.255;
    }
    # LAN-2
    subnet 10.0.2.0 netmask 255.255.255.0 {
        range 10.0.2.100 10.0.2.254; 
        option subnet-mask 255.255.255.0;
        option routers 10.0.2.1;
        option broadcast-address 10.0.2.255;
        }
    # LAN-3-SRV
    subnet 10.0.3.0 netmask 255.255.255.248 {
        range 10.0.3.3 10.0.3.6;
        option subnet-mask 255.255.255.248;
        option routers 10.0.3.1;
        option broadcast-address 10.0.3.7;
    }
4. **Check for interface errors**
    ```bash
    dhcpd -t
5. **Specify interfaces to use in ``sudo nano /etc/default/isc-dhcp-serve``**
    ```bash
    #       Separete multiple interfaces with spaces, e.g "eth0 ethe1".
    INTERFACESv4="ens3"
    INTERFACESv6=""
6. **Restart service dhcp**
    ```bash
    service isc-dhcp-server restart
    ```
## Linux Configuration for DNS Server - Steps

1. **Install BIND**  
   Install BIND DNS server using the following command:
   ```bash
   sudo apt-get update
   sudo apt-get install bind9
   ```
2. **Configure Static IP Address**  
   Edit the Netplan configuration file to set a static IP address:
   ```bash
   sudo nano /etc/netplan/50-cloud-init.yaml
   ```
   Edit the file as follows:
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
           addresses: [8.8.8.8, 8.8.4.4]
   ```
3. **Configure DNS Zones**  
   Edit the local BIND configuration file and create directory:
   ```bash
   sudo nano /etc/bind/named.conf.local
   ```
   Add the following zone configurations:
   ```bash
   // Zona de Pesquisa Direta
   zone "homelab.net" {
       type master;
       file "/etc/bind/db.homelab.net";
    };
    // Zona de Pesquisa Reversa para 10.0.3.0/29
    zone "3.0.10.in-addr.arpa" {
            type master;
            file "/etc/bind/db.10.0.3.0-29";
    };
    // Zona de Pesquisa Reversa para 10.0.2.0/24
    zone "2.0.10.in-addr.arpa" {
            type master;
            file "/etc/bind/db.10.0.2-24";
    };
    // Zona de Pesquisa Reversa para 10.0.1.0/24
    zone "1.0.10.in-addr.arpa" {
            type master;
            file "/etc/bind/db.10.0.1.0-24";
    };
    // Zona de Pesquisa Reversa para 192.168.100.200/32
    zone "200.100.168.192.in-addr.arpa" {
            type master;
            file "/etc/bind/db.192.168.100.200-32";
    };
   ```
4. **Create Forward Zone File**  
   Create the zone file for `homelab.net`: ``sudo nano /etc/bind/db.homelab.net``
   Add the following content:
   ```bash
      ;
      ; BIND data file for local loopback interface
      ;

      $TTL    604800
      @       IN      SOA     raffsway.homelab.net. root.homelab.net. (
                                 100         ; Serial
                              604800         ; Refresh
                               86400         ; Retry
                             2419200         ; Expire
                              604800 )       ; Negative Cache TTL
      ;
      ;
      homelab.net.    IN      NS      raffsway.homelab.net.
      homelab.net.    IN      A       10.0.3.3
      @               IN      A       127.0.0.1
      raffsway        IN      A       10.0.3.3
      SWLAN3          IN      A       10.0.3.2
      SWLAN2          IN      A       10.0.2.2
      SWLAN1          IN      A       10.0.1.2
      RT-DC           IN      A       192.168.100.200
      www             IN      CNAME   homelab.net.
   ```
5. **Create Reverse Zone File**
   - Create the reverse zone file for the `10.0.3.0/29` subnet:  ``sudo nano /etc/bind/db.10.0.3.0-29``.  
      Add the following content:
   ```bash
      ; BIND reverse data file for local network 10.0.3.0/29
      ;
      $TTL    604800
      @       IN      SOA     raffsway.homelab.net. root.homelab.net. (
                                   1         ; Serial
                              604800         ; Refresh
                               86400         ; Retry
                             2419200         ; Expire
                              604800 )       ; Negative Cache TTL
      ;
              IN      NS      raffsway.homelab.net.
      2       IN      PTR     SWLAN3.homelab.net.
      3       IN      PTR     raffsway.homelab.net.
   ```
   - Create the reverse zone file for the `10.0.2.0/24` subnet: ``sudo nano /etc/bind/db.10.0.2.0-24``.  
      Add the following content:
   ```bash
      ; BIND reverse data file for local network 10.0.2.0/24
      ;
      $TTL    604800
      @       IN      SOA     raffsway.homelab.net. root.homelab.net. (
                                   1         ; Serial
                              604800         ; Refresh
                               86400         ; Retry
                             2419200         ; Expire
                              604800 )       ; Negative Cache TTL
      ;
              IN      NS      raffsway.homelab.net.
      2       IN      PTR     SWLAN2.homelab.net.
   ```
   - Create the reverse zone file for the `10.0.1.0/24` subnet: ``sudo nano /etc/bind/db.10.0.1.0-24``.  
      Add the following content:
   ```bash
      ;
      ; BIND reverse data file for local network 10.0.1.0/24
      ;
      $TTL    604800
      @       IN      SOA     raffsway.homelab.net. root.homelab.net. (
                                    1         ; Serial
                               604800         ; Refresh
                                86400         ; Retry
                              2419200         ; Expire
                               604800 )       ; Negative Cache TTL
      ;
              IN      NS      raffsway.homelab.net.
      2       IN      PTR     SWLAN1.homelab.net.
   ```
   - Create the reverse zone file for the `192.168.100.200/32` subnet: ``sudo nano /etc/bind/db.192.168.100.200-32``.  
      Add the following content:
   ```bash
      ;
      ; BIND reverse data file for local network 192.168.100.200/32
      ;
      $TTL    604800
      @       IN      SOA     raffsway.homelab.net. root.homelab.net. (
                                    1         ; Serial
                               604800         ; Refresh
                                86400         ; Retry
                              2419200         ; Expire
                               604800 )       ; Negative Cache TTL
      ;
              IN      NS      raffsway.homelab.net.
      1       IN      PTR     RT-DC.homelab.net.
   ```
6. **Check BIND Configuration**  
   Before restarting BIND, check for syntax errors in the configuration files:
   ```bash
   sudo named-checkzone homelab.net /etc/bind/db.homelab.net
   sudo named-checkzone 3.0.10.in-addr.arpa /etc/bind/db.10.0.3.0-29
   sudo named-checkzone 2.0.10.in-addr.arpa /etc/bind/db.10.0.2.0-24
   sudo named-checkzone 1.0.10.in-addr.arpa /etc/bind/db.10.0.1.0-24
   sudo named-checkzone 200.100.168.192.in-addr.arpa /etc/bind/db.192.168.100.200-32
   ```
7. **Configure DNS Options file**  
   Add the following commands to the file /etc/bind/named.conf.options:
   ```bash
   options {
        directory "/var/cache/bind";

        forwarders { // Para permitir que consultas sejam encaminhadas para internet
                8.8.8.8;
                8.8.4.4;
      };
        forward only;  // Para permitir encaminhamendo de consultas
        allow-query {any; }; // Para permitir consultas a partir de outras redes

        dnssec-validation auto;
        auth-nxdomain no;
        listen-on-v6 { any; };
      };
   ```
8. **Restart BIND Service**  
   Restart the BIND service to apply the changes:

   ```bash
   sudo systemctl restart bind9
   ```
9. **Test the DNS Configuration**  
   Use `dig` to verify the DNS resolution:  
    ```bash
    dig @127.0.0.1 homelab.net
    dig @127.0.0.1 -x 10.0.3.3
    ```
10. **Troubleshooting BIND9 DNS Server**  
   Here’s the procedure in English if you encounter issues with localhost not being set to 127.0.0.1 in ``/etc/resolv.conf``:  
-  **Verify the Localhost Line**
   Ensure the file includes the following line. If it’s missing, add it to the file:
   ```bash 
   nameserver 127.0.0.1
   ```
   If restarting the BIND9 service causes the configuration to revert to its previous state, it may be necessary to remove the ``resolv.conf`` file."
-  **Disable systemd-resolved**  
    If you don’t need `systemd-resolved`, you can disable it by running the following commands:
    ```bash
    sudo systemctl stop systemd-resolved
    sudo systemctl disable systemd-resolved
    ```
-  **Remove the Symbolic Link**  
    Remove the symbolic link pointing to systemd-resolved:

    ```bash
    sudo rm /etc/resolv.conf
    ```
-  **Create a New resolv.conf File**  
    Create a new `resolv.conf` file to point directly to BIND9:

    ```bash
    sudo nano /etc/resolv.conf
    ```

-  **Add the Following Line**  

    ```bash
    nameserver 127.0.0.1
    ```

-  **Restart BIND9**  
    Finally, restart the BIND9 service to ensure the settings are applied:

    ```bash
    sudo systemctl restart bind9
    ```


## Syslog Server Setup on Linux  
1. **Update and Install Syslog Server Packages**
   - Update package lists:
     ```bash
     sudo apt update
     ```
   - Install `rsyslog`:
     ```bash
     sudo apt install rsyslog
     ```
   - Verify if the service is running:
     ```bash
     sudo systemctl status rsyslog
     ```
2. **Configure Syslog Server**
   - Edit the rsyslog configuration file to enable remote logging:
     ```bash
     sudo nano /etc/rsyslog.conf
     ```
   - Uncomment or add the following lines to enable UDP and TCP reception (optional if not required):
     ```bash
     # Provides UDP syslog reception
     module(load="imudp")
     input(type="imudp" port="514")
     
     # Provides TCP syslog reception
     module(load="imtcp")
     input(type="imtcp" port="514")
     ```
3. **Create Log Directory (Optional)**
   - Set up a specific directory for Syslog files if needed:
     ```bash
     sudo mkdir /var/log/syslog
     ```
4. **Restart Syslog Service**
   - Restart the rsyslog service to apply the new configuration:
     ```bash
     sudo systemctl restart rsyslog
     ```
   - Confirm the service is running:
     ```bash
     sudo systemctl status rsyslog
     ```
5. **Configure Remote Devices to Send Logs**
   - On the remote device (such as a router or switch), configure it to send logs to the Syslog server’s IP and port 514 (adjust as necessary).

6. **Viewing Syslog Entries**
   - Check the Syslog server logs by viewing the log file:
     ```bash
     tail -f /var/log/syslog
     ```

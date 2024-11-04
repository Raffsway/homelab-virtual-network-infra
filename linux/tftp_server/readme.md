
# TFTP Server Configuration Guide

## TFTP Server Setup on Linux
1. **Update and Install TFTP Server Packages**
   - Update package lists:
     ```bash
     sudo apt update
     ```
   - Install `tftpd-hpa`:
     ```bash
     sudo apt install tftpd-hpa
     ```
   - Verify the service status:
     ```bash
     sudo systemctl status tftpd-hpa
     ```
2. **Configure TFTP Server**
   - Edit the TFTP configuration file:
     ```bash
     sudo nano /etc/default/tftpd-hpa
     ```
     Now, I just want to change the ``TFTP_DIRECTORY to /tftp`` and ``add the –create`` option to the ``TFTP_OPTIONS``. Without the –create option, you will not be able to create or upload new files to the TFTP server. You will only be able to update existing files. So I think the –create option is very important.

     The final configuration file should look like this. Now, press + x followed by y and to save the changes.
     ```yaml
     # /etc/default/tftpd-hpa

     TFTP_USERNAME="tftp"
     TFTP_DIRECTORY="/tftp"
     TFTP_ADDRESS=":69"
     TFTP_OPTIONS="--secure --create"
     ```
3. **Create TFTP Root Directory**
   - Set up the directory for TFTP file transfers:
     ```bash
     sudo mkdir /tftp
     ```
     - Now, change the owner and group of the /tftp directory to tftp with the following command:
     ```bash
     sudo chown tftp:tftp /tftp
     ```
4. **Restart TFTP Service**
   - Restart the `tftpd-hpa` service to apply changes:
     ```bash
     sudo systemctl restart tftpd-hpa
     ```
   - Confirm the service is running:
     ```bash
     sudo systemctl status tftpd-hpa
     ```
5. **Install TFTP Client (Optional)**
   - For testing the server, you can install `tftp-hpa`:
     ```bash
     sudo apt install tftp-hpa
     ```
6. **Using TFTP Client Commands**
   - Connect to the TFTP server (replace ``10.0.3.3`` with the server's IP address):
     ```bash
     tftp 10.0.3.3
     ```
   - Once connected, use the following commands:
     - Enable verbose mode for detailed output:
       ```bash
       tftp> verbose
       ```
     - **Exit** the TFTP session:
       ```bash
       tftp> quit
       ```
---

Assignment 3 Part 1

This assignment sets up a web server that shows system information (like the kernel release, OS version, and installed packages) in a webpage.
It also includes a service to generate the system’s info daily and services it by using Nginx, with firewall protection.

Requirements
    Nginx installed on the system
    UFW (Uncomplicated Firewall) installed

Overview
	1. System User Creation: We create a special system user (webgen) to run the web server.
	2. Systemd Service & Timer: A service runs a script to generate the system information and a timer that runs it daily.
	3. Nginx Web Server: Nginx is configured to serve the generated index.html file.
	4. Firewall: We set up a firewall to allow SSH and HTTP traffic securely

Setup Instructions

 1. Create the webgen User
  Create the webgen system user and set up its home directory, this user will be isolated from other system processes 
  for better security when running the script and managing the files in regard to what the user can accesss:

```bash
sudo useradd --system -s /usr/bin/nologin webgen
sudo usermod -d /var/lib/webgen -m webgen
mkdir -p /var/lib/webgen/bin /var/lib/webgen/HTML
sudo chown -R webgen:webgen /var/lib/webgen
```
 
 
 2. Install the Script and Configuration Files
  Clone the `generate_index` script from the repoistory, which will generates the HTML document with the system information, to the webgen user's bin directory
  - Place in `/var/lib/webgen/bin/` and allow webgen executable privileges:

```bash
sudo mv /file_path/generate_index /var/lib/webgen/bin/
sudo chmod +x /var/lib/webgen/bin/generate_index
```
  - Next, we manage the service and timer files in the `/etc/systemd/system/` directory. First, ensure that you have moved the `generate_index` script to the the proper location.
  Use:
    ```bash
    ls -l /var/lib/webgen/bin/generate_index
    ```
  - Onve the file is confirmed to be in the right directory, we can start using the necessary files. When creating any service/ timer file, these must be placed in the `/etc/systemd/system/` directory. 
  - Run the following to complete this task:

  ```bash
  sudo cp generate-index.service /etc/systemd/system/
  sudo cp generate-index.timer /etc/systemd/system/ 
  ```

  - Now that these files are in the correct path, they can be enables using the systemd service manager. Run the following:
  ```bash
  sudo systemctl daemon-reload
  sudo systemctl enable generate-index.timer
  ```
  - When using any service on linux, remember to reload them, then you can start working on them, here we enable the timer to work daily at 5:00 am.
  
3. Nginx Configuration
  Copy the Nginx configuration files:
  - Create a directory for the server block config file to go in, such as the `conf.d` that was used in the example below, this is typical convention.

   - Global configuration file (`nginx.conf`) should be placed in `/etc/nginx/`.
   - Server block file (`system_info_server.conf`) should go in `/etc/nginx/conf.d/`.

 
  ```bash
  sudo cp nginx.conf /etc/nginx/
  sudo cp system_server.conf /etc/nginx/server_block_dir/
  ```

  - The server-block acts like a failsafe ensuring that if we made a mistake with our nginx.conf file, than all our nginx services may get affected. 
  - The server-block prevents this issue, making it safer for when we apply changes and might need to troubleshoot

 - Test the Nginx configuration with:
  ```bash
  sudo nginx -t 
  ```
 - Ensure the syntax is confirmed to be error free and that the test was successful once you run this command

 - Now, restart the Nginx

```bash
sudo systemctl restart nginx
```

4. Configure Firewall

Enable and configure the firewall to allow only SSH (port 22) and HTTP (port 80) traffic. SSH rate limiting is enabled for added security.

```bash
sudo ufw enable
sudo ufw allow ssh
sudo ufw allow http
sudo ufw limit ssh
```

- IMPORTANT: Do NOT forget to enable ssh to ensure the droplet's server won't lock us out.

Verify the setup by checking the status of the timer, run the following:

```bash
sudo systemctl list-timers | grep generate-index.timer
systemctl status generate-index.timer
```

 - Now check that everying is working on the server end locally by running:
 ```bash
 curl http://localhost
 ```

 - In a browser, check the succesful connection from the server by using your public ip address and check that the system information is displayed. 
 `http://[ip_address]`

 - Finally check your firewall settings use:
 ```bash
 sudo ufw status verbose
 ```

 - And make sure it looks like this;

 To                         Action      From
--                         ------      ----
22                         ALLOW IN    Anywhere
80                         ALLOW IN    Anywhere
22 (v6)                    ALLOW IN    Anywhere (v6)
80 (v6)                    ALLOW IN    Anywhere (v6)


5. General

 General enhancements for the generate_index script could involve further error handling such as the inclusion of an log file that can help to troubleshoot any issues during the setup by using a log_error function. The script could also include further information on our system information by running command like `hostnamectl` and `lscpu` which could return the number of cores, threads, or socketes on a machinealong with the hostname, hardware vender, or hardware model.


6. Solutions

 Cloud Provider Firewall:

 Perhaps DigitalOcean needs some confirguration of their separate firewall, I did however try to configure the network security settings to include all inboung traffic on the HTTP port in addition to the SSh one. Might be it was not done correctly.

 The server block might have issues accessing the server through server name and that localhost is not the correct one we need, and instead we should make it the public ip address to get a direct reference.

 External Network Issues on my own side where port 80 was being blocked.

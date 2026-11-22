# Docker-Project
- In PROXMOX, I cloned my Ubuntu VM and gave it a new name  
- Make sure there is no desktop environment and that you are able too SSH from your own computer, which in this case I did it through my Ubuntu on my VMware  
- Before I put down any commands, I made sure to go to the Snapshots tab of the VM and take a snapshot, naming it pre-docker
# Installation
- I installed the Docker program using the following commands:
- sudo apt update && sudo apt install -y ca-certificates curl
- sudo install -m 0755 -d /etc/apt/keyrings
- sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
  /etc/apt/keyrings/docker.asc
  echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] \
https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo $VERSION_CODENAME) stable" \
| sudo tee /etc/apt/sources.list.d/docker.list
- sudo apt update
- sudo apt install -y docker-ce docker-ce-cli containerd.io \
docker-buildx-plugin docker-compose-plugin
# Post-Install Checks
- I ran some commands to ensure that docker is running and that it is running properly
 - sudo systemctl enable --now docker
 - systemctl is-enabled docker 
 - sudo docker run --rm hello-world
 - docker compose version
- Later on I ensured the app can reboot and not just crash when I command sudo reboot
# Application (Uptime-Kuma)
- My application of choice was Uptime-Kuma, and I kind of chose it out of random
- Their installation website is here: https://uptimekuma.co/install-uptime-kuma-docker/#google_vignette
- I downloaded through Docker Compose using the following commands:
- mkdir -p ~/uptime-kuma && cd ~/uptime-kuma (to make a new directory and be in it)
- nano docker-compose.yml (Scripting)
- We will be in the nano mode and this is what it will contain:  
  services:    
  uptime-kuma:  
  image: louislam/uptime-kuma:2  
    container_name: uptime-kuma  
    restart: always  
    ports:  
      - "3001:3001"  # This maps the container port "3001" to the host port "3001"  
    volumes:  
      - /path/to/data:/app/data  # Configuring persistent storage  
    environment:  
      - TZ=UTC  # Set the timezone (change to your preferred local timezone so monitoring times are the same)  
      - UMASK=0022  # Set your file permissions manually  
    networks:  
      - kuma_network  # add your own custom network config  
    healthcheck:  
      test: ["CMD", "curl", "-f", "http://localhost:3001"]  
      interval: 30s  
      retries: 3  
      start_period: 10s  
      timeout: 5s  
    logging:  
      driver: "json-file"  
      options:  
        max-size: "10m"  
        max-file: "3"  

  networks:  
  kuma_network:  
    driver: bridge  
- I ran sudo docker-compose up-d so that I can start the container
- Visted http://10.30.42.110:3001 and it work
# Securing the VM
- I opened a new window to start the SSH key portion
- I used this command: ssh-keygen -t ed25519 to generate a public and a private key
- The important part was copying it to my vm using: ssh-copy-id ett1501@10.30.42.110
- After confirming that I was able to ssh without using a password through multiple windows, I was able to disbale password authentication.
- (To know when you are successful, you should be able to ssh name@<ip_of_vm> and not have to input a password and be in the shell
- I was able to disable the password authentication by nano /etc/ssh/ssh_config and scrolling into you see a "PasswordAuthentication yes". You edit the yes to a no
- I installed a firewall for the second task of this section using: sudo apt update && sudo apt install -y ufw
- I enabled the firewall where only the SSH and the Uptime-Kuma ports are open
- As stated from the assignment, I installed unattended-upgrades(ubuntu) so that I can get upgrades automatically
  sudo apt update ** sudo apt install -y unattended-upgrades  
  sudo dpkg-reconfigure -plow unattended-upgrades (enabling)
# Backups and Restore Drill
- I made a script that allows me to store data using tar into a backup directory
- I would have to stop the app, store data, and start the app
- It starts with making a directory in home, which i named ~/backup.sh and then scripting it using nano
- Code:  
  #!/bin/bash  
  mkdir -p ~/backups  
  cd /home/ett1501/uptime-kuma  
  sudo docker compose stop  
  sudo tar -czf ~/bakcups/bakcup-$(date +%s).tar.gz data  
  sudo docker compose start  
  ls -t ~/backups/bakcup-*.tar.gz | tail -n +8 | xargs -r rm
  - after making the script I would Backup the VM on proxmox
    Left side, Backup > Backup now
  -  I went to my online program and edited around a couple of areas
  -  I ran my nano program and stopped the process of the program
  -  I removed the data that was stored and started the program back up again, leading me to a brand new page  
    

  
  
    
  

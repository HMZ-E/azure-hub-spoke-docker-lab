# Azure Hub-and-Spoke Lab: ADDS, Docker & Multi-Tier Web App

This project is a hands-on Azure infrastructure lab where I built a secure Hub-and-Spoke network topology. I deployed a Windows Server acting as a Bastion Host and Domain Controller (ADDS/DNS) to securely manage an isolated Ubuntu Docker backend hosting a multi-tier WordPress application.

## Project Architecture Diagram

<img width="1024" height="559" alt="watermarked_img_16635784561564119234" src="https://github.com/user-attachments/assets/c1dc0c49-9651-4385-819a-68a4d427e188" />

## Phase 1: Virtual Network & ADDS Bastion Host

I started by building the core network and identity infrastructure.
* Created an Azure Virtual Network (`Hub-VNet`) using the `10.0.1.0/24` address space.
* Deployed a Windows Server 2022 VM (`DC01`) with a static internal IP (`10.0.1.4`).
* Installed Active Directory Domain Services (ADDS) and promoted the server to a Domain Controller.
* Configured the DNS Server role and updated the Azure VNet to use `10.0.1.4` for all internal custom DNS resolution.

## Phase 2: Linux Backend & NSG Firewall

Next, I set up the secure backend server for containerized workloads.
* Provisioned an Ubuntu 24.04 LTS instance (`Linux-Docker-Host`) on `10.0.1.5`.
* Configured Azure Network Security Groups (NSGs) to strictly control traffic.
* Opened SSH (22), MySQL (3306), and Portainer (9000) for internal management from DC01 only.
* Opened HTTP (80) to the public internet for the web application.

## Phase 3: Docker & Container Deployment

I connected to the Linux host internally from my DC01 Bastion Host to install the runtime engine and spin up the backend services.

- on DC01:
    ```text
    # ssh sysadmin@10.0.1.5
    ```

- on Linux-Docker-Host:
    ```text
    # curl -fsSL [https://get.docker.com](https://get.docker.com) -o get-docker.sh
    # sudo sh get-docker.sh
    # sudo systemctl enable docker
    # sudo systemctl start docker
    # sudo docker run --name lab-mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=Login.20010920 --restart=always -d mysql:8.0
    # sudo docker volume create portainer_data
    # sudo docker run -d -p 9000:9000 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:latest
    ```

## Phase 4: Remote Database Management

Instead of managing the database via CLI, I set up a proper client-server management connection.
* Installed DBeaver Community Edition on `DC01`.
* Resolved Windows Server network security blocks by manually downloading the MySQL JDBC driver.
* Overrode default MySQL 8.0 `caching_sha2_password` restrictions to allow unencrypted local network connections.
* Executed SQL queries remotely to provision the `wordpress` database schema.

## Phase 5: WordPress Frontend Deployment

For the final tier, I deployed the frontend web server and linked it to the backend database.
* Accessed Portainer via `http://10.0.1.5:9000`.
* Deployed the `wordpress:latest` container and mapped port `80` to the host.
* Injected database connection parameters using Docker environment variables (`WORDPRESS_DB_HOST`, `WORDPRESS_DB_USER`, `WORDPRESS_DB_PASSWORD`, `WORDPRESS_DB_NAME`).
* Successfully tested public access to the WordPress site via the Azure public IP.

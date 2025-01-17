# Complete Setup Guide for Word2Vec Application

## 1. Configuring Iranian DNS Resolvers

### Manual Configuration
1. Open the DNS resolver configuration file:
```bash
sudo nano /etc/resolv.conf
```

2. Add these DNS server addresses ([source](https://shecan.ir/)) to the top of the file:
```
nameserver 178.22.122.100
nameserver 185.51.200.2
```

3. Save and exit (Ctrl + X, then Y, then Enter)

### Important Note
This configuration is temporary and will reset after system reboot.

## 2. System Preparation and Docker Installation

### Update System
```bash
sudo apt update
sudo apt upgrade -y
```

### Install Docker
1. Remove old Docker versions if present:
```bash
sudo apt remove docker docker-engine docker.io containerd runc
```

2. Install prerequisites:
```bash
sudo apt install apt-transport-https ca-certificates curl software-properties-common -y
```

3. Add Docker's GPG key and repository:
```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

4. Install Docker:
```bash
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io -y
```

5. Verify installation:
```bash
sudo docker --version
```

### Install Docker Compose
```bash
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker-compose --version
```

### Configure Docker
```bash
sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -aG docker $USER  # Optional: Add user to docker group
```

Note: Log out and back in after adding user to docker group

## 3. Application Setup

### Clone Repositories
```bash
git clone https://github.com/icm-project/word2vec-tree.git
git clone https://github.com/icm-project/word2vec.git
```

### Create Docker Compose Configuration
Create a new file named `docker-compose.yml` and add the following content:

```yaml
version: '3.9'
services:
  word2vec-tree:
    build:
      context: ./word2vec-tree/
      dockerfile: Dockerfile
      args:
        BACKEND_DOMAIN: ${BACKEND_DOMAIN:-http://87.247.176.61:443/}
    container_name: word2vec-tree
    ports:
      - "80:3000"
    depends_on:
      - word2vec
    restart: unless-stopped
    networks:
      - custom_network
    environment:
      - NODE_ENV=production
  word2vec:
    build:
      context: ./word2vec/
      dockerfile: Dockerfile
    container_name: word2vec
    ports:
      - "443:8000"
    restart: unless-stopped
    networks:
      - custom_network
networks:
  custom_network:
    driver: bridge
```

### Deploy Application
Build and start the containers:
```bash
docker compose up --build -d
```

The application will be accessible on:
- Frontend: `http://localhost:20000`
- Backend: `http://localhost:20001`

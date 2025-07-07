# AWS Setup for Docker and Nginx & Certificate with Certbot with n8n



### Step 1: Connect to AWS EC2 Instance via SSH
```bash
ssh -i yourkey.pem ec2-user@publicip
```

### Step 2: Update the instance and install Docker
```bash
sudo yum update -y
sudo yum install -y docker
```

### Step 3: Start and enable Docker service
```bash
sudo systemctl start docker
sudo systemctl enable docker
```

### Step 4: Add user to the Docker group for non-root access
```bash
sudo usermod -aG docker ec2-user
```

### Step 5: Exit and log back in so the changes can take effect
```bash
exit
```

### Step 6: Connect to AWS EC2 Instance via SSH
```bash
ssh -i yourkey.pem ec2-user@publicip
```

### Step 7: Install Docker Compose
```bash
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

### Step 8: Run N8N Docker Container
```bash
sudo docker run -d --restart unless-stopped -it \
--name n8n \
-p 5678:5678 \
-e N8N_HOST="your-domain-name" \
-e WEBHOOK_TUNNEL_URL="https://your-domain-name/" \
-e WEBHOOK_URL="https://your-domain-name/" \
-v ~/.n8n:/root/.n8n \
n8nio/n8n
```

### Step 9: Install and Configure Nginx
```bash
sudo dnf install -y nginx
sudo systemctl start nginx
sudo systemctl enable nginx
```

### Step 10: Configure Nginx for N8N Reverse Proxy
```bash
sudo nano /etc/nginx/conf.d/n8n.conf
```
Add the following content in `n8n.conf`:
```nginx
server {
    listen 80;
    server_name your-domain-name;

    location / {
        proxy_pass http://localhost:5678;
        proxy_http_version 1.1;
        chunked_transfer_encoding off;
        proxy_buffering off;
        proxy_cache off;

        # Headers for WebSocket support
        proxy_set_header Connection 'Upgrade';
        proxy_set_header Upgrade $http_upgrade;

        # Additional headers for forwarding client info
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```
Save and exit using:
```bash
CTRL+O, ENTER, CTRL+X
```

### Step 11: Test Nginx Configuration and Restart Service
```bash
sudo nginx -t
sudo systemctl restart nginx
```

### Step 12: Set Up SSL Certificate with Certbot
```bash
sudo dnf install -y certbot python3-certbot-nginx
sudo certbot --nginx -d your-domain-name
sudo systemctl restart nginx
```

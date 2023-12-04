# SSL on NGINX

Owner: Mojtaba Safaei
Verification: Verified

**First, add the repository:**

```bash
sudo add-apt-repository ppa:certbot/certbot
```

**Install Certbot’s Nginx package with apt:**

```bash
sudo apt install python-certbot-nginx
```

**Run certbot and follow the steps to secure the specific site:**

```bash
sudo certbot
```
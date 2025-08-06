# Nginx Deployment for PHP-Powered eCommerce Website

## Installation

### 1. Install Required Packages
```bash
sudo apt update
sudo apt install -y nginx php8.4 php8.4-fpm php8.4-mysql mariadb-server git
```

### 2. Stop Conflicting Services
```bash
sudo systemctl stop apache2 php8.3-fpm 2>/dev/null || true
sudo systemctl disable apache2 php8.3-fpm 2>/dev/null || true
```

### 3. Configure Database
```bash
sudo mysql
```
```sql
CREATE DATABASE ecomdb;
CREATE USER 'ecomuser'@'localhost' IDENTIFIED BY 'ecompassword';
GRANT ALL PRIVILEGES ON *.* TO 'ecomuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

### 4. Load Sample Data
```bash
cat > db-load-script.sql <<-EOF
USE ecomdb;
CREATE TABLE products (id mediumint(8) unsigned NOT NULL auto_increment,Name varchar(255) default NULL,Price varchar(255) default NULL, ImageUrl varchar(255) default NULL,PRIMARY KEY (id)) AUTO_INCREMENT=1;
INSERT INTO products (Name,Price,ImageUrl) VALUES ("Laptop","100","c-1.png"),("Drone","200","c-2.png"),("VR","300","c-3.png"),("Tablet","50","c-5.png"),("Watch","90","c-6.png"),("Phone Covers","20","c-7.png"),("Phone","80","c-8.png"),("Laptop","150","c-4.png");
EOF

sudo mysql < db-load-script.sql
```

### 5. Configure PHP-FPM (Port 9001)
```bash
sudo sed -i 's/listen = .*/listen = 127.0.0.1:9001/' /etc/php/8.4/fpm/pool.d/www.conf
```

### 6. Configure Nginx
```bash
sudo tee /etc/nginx/conf.d/default.conf > /dev/null <<EOF
server {
    listen 80;
    root /var/www/html;
    index index.php;
    location / { try_files \$uri \$uri/ /index.php?\$args; }
    location ~ \.php\$ {
        fastcgi_pass 127.0.0.1:9001;
        fastcgi_param SCRIPT_FILENAME \$document_root\$fastcgi_script_name;
        include fastcgi_params;
    }
}
EOF

sudo rm -f /etc/nginx/sites-enabled/default
```

### 7. Download Code
```bash
sudo git clone https://github.com/MostafaAbdulazziz/nginx-hosted-php-ecommerce.git /var/www/html/
```

### 8. Create Environment File
```bash
sudo tee /var/www/html/.env > /dev/null <<EOF
DB_HOST=localhost
DB_USER=ecomuser
DB_PASSWORD=ecompassword
DB_NAME=ecomdb
EOF
```

### 9. Set Permissions
```bash
sudo chown -R www-data:www-data /var/www/html/
```

## Running

### Start Services
```bash
sudo systemctl start mariadb php8.4-fpm nginx
sudo systemctl enable mariadb php8.4-fpm nginx
```

### Test
```bash
curl http://localhost
```


Vagrant.configure("2") do |config|

  # Налаштування веб-сервера
  config.vm.define "web" do |web|
    web.vm.box = "ubuntu/focal64"
    web.vm.network "forwarded_port", guest: 80, host: 8080
    web.vm.network "private_network", ip: "192.168.56.10"
    
    web.vm.provision "shell", inline: <<-SHELL
      # Оновлення системи
      sudo apt update && sudo apt upgrade -y
      
      # Встановлення Nginx, PHP та необхідних модулів
      sudo apt install -y nginx php-fpm php-mysql 

      sudo add-apt-repository -y ppa:ondrej/php
      sudo apt-get update -y && sudo apt-get install ca-certificates apt-transport-https software-properties-common
      sudo apt-get install -y php8.3-fpm
      
      # Налаштування PHP-FPM
      sudo sed -i 's|fastcgi_pass unix:/run/php/php7.4-fpm.sock;|fastcgi_pass unix:/run/php/php8.3-fpm.sock;|g' /etc/nginx/sites-available/default
      
      # Створення PHP-скрипту для підключення до бази даних
      sudo tee /var/www/html/index.php > /dev/null <<EOL
<?php
\$conn = new mysqli("192.168.56.11", "vagrant", "vagrant", "test_db");
if (\$conn->connect_error) {
    die("Connection failed: " . \$conn->connect_error);
}
\$result = \$conn->query("SELECT * FROM users");
echo "<h1>Users List</h1>";
while (\$row = \$result->fetch_assoc()) {
    echo "<p>" . \$row["id"] . ". " . \$row["name"] . "</p>";
}
\$conn->close();
?>
EOL
      
      # Перезапуск Nginx
      sudo systemctl restart nginx
    SHELL
  end

  # Налаштування сервера бази даних
  config.vm.define "db" do |db|
    db.vm.box = "ubuntu/focal64"
    db.vm.network "forwarded_port", guest: 3306, host: 3306
    db.vm.network "private_network", ip: "192.168.56.11"
    
    db.vm.provision "shell", inline: <<-SHELL
      # Оновлення системи
      sudo apt update && sudo apt upgrade -y
      
      # Встановлення MySQL
      sudo apt install -y mysql-server
      
      # Налаштування MySQL для зовнішніх підключень
      sudo sed -i 's/bind-address.*/bind-address = 0.0.0.0/' /etc/mysql/mysql.conf.d/mysqld.cnf
      
      # Перезапуск MySQL
      sudo systemctl restart mysql
      
      # Створення бази даних та користувача
      sudo mysql <<EOL
CREATE DATABASE test_db;
USE test_db;
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL
);
INSERT INTO users (name) VALUES ('John'), ('Bill'), ('Cris');

CREATE USER 'vagrant'@'%' IDENTIFIED BY 'vagrant';
GRANT ALL PRIVILEGES ON test_db.* TO 'vagrant'@'%';
FLUSH PRIVILEGES;
EOL
    SHELL
  end
end
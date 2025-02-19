Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/bionic64"
  # Machine Web
  config.vm.define "web" do |web|
    web.vm.network "private_network", ip: "192.168.56.20"
    web.vm.network "forwarded_port", guest: 8086, host: 8086 # Correction du port
    web.vm.hostname = "web-node"
    web.vm.provider "virtualbox" do |vb|
      vb.name = "web-server"
      vb.memory = "2048"
      vb.cpus = 2
    end
    
    web.vm.provision "shell", inline: <<-SHELL
  # ... (garder les parties précédentes inchangées)

  # Construction du projet
  cd /home/vagrant/admin-app
  chmod +x mvnw
  ./mvnw clean package -DskipTests

  # Vérification de la génération du JAR
  if [ ! -f "target/admin-app-0.0.1-SNAPSHOT.jar" ]; then
    echo "ERREUR : Le fichier JAR n'a pas été généré!"
    exit 1
  fi

  # Configuration du service systemd
  echo "[Unit]
Description=Spring Boot Application
After=network.target

[Service]
User=vagrant
WorkingDirectory=/home/vagrant/admin-app
ExecStart=/usr/lib/jvm/java-17-openjdk-amd64/bin/java -jar /home/vagrant/admin-app/target/admin-app-0.0.1-SNAPSHOT.jar
Restart=always
Environment=\"DB_HOST=192.168.56.21\"
Environment=\"DB_NAME=admin_app_db\"
Environment=\"DB_USERNAME=admin\"
Environment=\"DB_PASSWORD=password\"

[Install]
WantedBy=multi-user.target" | sudo tee /etc/systemd/system/springboot.service

  sudo systemctl daemon-reload
  sudo systemctl enable springboot.service
  sudo systemctl start springboot.service
SHELL
  end

  # Machine DB
  config.vm.define "db" do |db|
    db.vm.network "private_network", ip: "192.168.56.21"
    db.vm.hostname = "db-node"
    db.vm.provider "virtualbox" do |vb|
      vb.name = "db-server"
      vb.memory = "1024"
      vb.cpus = 1
    end

    db.vm.provision "shell", inline: <<-SHELL
      # Installation de MySQL
      sudo apt-get update
      sudo apt-get install -y mysql-server

      # Configuration MySQL
      sudo sed -i 's/127.0.0.1/0.0.0.0/' /etc/mysql/mysql.conf.d/mysqld.cnf
      
      # Création de la BDD et de l'utilisateur
      sudo mysql -e "CREATE DATABASE admin_app_db;"
      sudo mysql -e "CREATE USER 'admin'@'%' IDENTIFIED WITH mysql_native_password BY 'password';"
      sudo mysql -e "GRANT ALL PRIVILEGES ON admin_app_db.* TO 'admin'@'%';"
      sudo mysql -e "FLUSH PRIVILEGES;"

      # Redémarrage des services
      sudo systemctl restart mysql
    SHELL
  end
end
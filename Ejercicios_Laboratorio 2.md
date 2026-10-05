# Ejercicios 1, 2 y 3
```bash title="Vagrantfile"
# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
        config.vm.box = "ubuntu/focal64"
        config.vm.hostname = "servidor-web"
  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
        config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
        config.vm.synced_folder "./compartido", "/vagrant_data"

  # Disable the default share of the current code directory. Doing this
  # provides improved isolation between the vagrant box and your host
  # by making sure your Vagrantfile isn't accessible to the vagrant box.
  # If you use this you may want to enable additional shared subfolders as
  # shown above.
  # config.vm.synced_folder ".", "/vagrant", disabled: true

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
        config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
                vb.gui = false
  #
  #   # Customize the amount of memory on the VM:
                vb.memory = "512"
                vb.cpus = 1
        end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Enable provisioning with a shell script. Additional provisioners such as
  # Ansible, Chef, Docker, Puppet and Salt are also available. Please see the
  # documentation for more information about their specific syntax and use.
        config.vm.provision "shell", inline: <<-SHELL
                apt-get update
                apt-get install -y apache2
                { ip a; systemctl status apache2 } > /vagrant_data/informe_despliegue.txt
        SHELL
end
```

# Ejercicio 4
```bash title="Vagrantfile"
# vi: set ft=ruby :
BOX_IMAGE="ubuntu/focal64"
NODE_COUNT = 2

Vagrant.configure("2") do |config|

  (1..NODE_COUNT).each do |i|
    config.vm.define "web#{i}" do |subconfig|
      subconfig.vm.box = BOX_IMAGE
      subconfig.vm.hostname = "web#{i}"
      dir_ip="192.168.56.#{19+i}"
      print dir_ip+ "\n"
      subconfig.vm.network "private_network", ip: dir_ip
      #subconfig.vm.network "private_network", type: "dhcp"
      subconfig.vm.provision "shell", inline: <<-SHELL
                apt-get update
                apt-get install -y apache2
                mkdir -p /vagrant_data
                { ip a; systemctl status apache2; } > /vagrant_data/informe_despliegue_$(hostname).txt
        SHELL
    end
  end

  config.vm.box = BOX_IMAGE
  config.vm.synced_folder "./compartido", "/vagrant_data"

  config.vm.provider "virtualbox" do |vb|
    vb.gui = false
    vb.memory = "512"
  end

end
```

Para comprobar el ssh y curl:
```bash
vagrant ssh web1

ping 192.168.56.21

curl -I http://192.168.56.21 
# para obtener únicamente los encabezados (headers) de respuesta HTTP de un servidor, sin descargar el cuerpo ni el contenido HTML de la página
```

# Ejercicio 5
``` title="Vagrantfile"
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

        config.vm.box = "ubuntu/focal64"
        # Servidor web
        config.vm.define "nodo-web" do |web|
                web.vm.hostname = "nodo-web"
                web.vm.network "private_network", ip: "192.168.56.20"
                web.vm.provision "shell", inline: <<-SHELL
                        apt-get update
                        apt-get install -y apache2 mariadb-client
                SHELL
        end

        # Servidor db
        config.vm.define "nodo-db" do |db|
                db.vm.hostname = "nodo-db"
                db.vm.network "private_network", ip: "192.168.56.30"
                db.vm.synced_folder "./db_dumps", "/backup_db"
                db.vm.provision "shell", inline: <<-SHELL
                        apt-get update
                        apt-get install -y mariadb-server

                        mysql -e "DROP DATABASE IF EXISTS prueba_db"
                        mysql -e "CREATE DATABASE prueba_db"
                        mysql -e "CREATE TABLE prueba_db.users (id INT AUTO_INCREMENT PRIMARY KEY, nombre VARCHAR(255));"
                        mysql -e "INSERT INTO prueba_db.users (nombre) VALUES ('Admin'), ('Ubuntu'), ('Cimsi');"

                        FECHA=$(date +%Y%m%d_%H%M%S)
                        ARCHIVO="/backup_db/dump_prueba_${FECHA}.sql"
                        mysqldump prueba_db > ${ARCHIVO}
                SHELL
        end
end
```

``` title="Vagrantfile_jose"
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/focal64"

# Servidor Base de Datos
  config.vm.define "nodo-db" do |db|
    db.vm.hostname = "nodo-db"
    db.vm.network "private_network", ip: "192.168.56.30"
	db.vm.synced_folder "./db_dumps", "/backup_db"
    
    db.vm.provision "shell", inline: <<-SHELL
      apt-get update
      apt-get install -y mariadb-server
		
	  # Apertura del socket para permitir peticiones del nodo-web
      sed -i 's/bind-address.*/bind-address = 0.0.0.0/' /etc/mysql/mariadb.conf.d/50-server.cnf
      systemctl restart mariadb

      mysql -e "DROP DATABASE IF EXISTS prueba_db;"
      mysql -e "CREATE DATABASE prueba_db;"
      mysql -e "CREATE TABLE prueba_db.users (id INT AUTO_INCREMENT PRIMARY KEY, nombre VARCHAR(255));"
      mysql -e "INSERT INTO prueba_db.users (nombre) VALUES ('Admin'), ('Ubuntu'), ('Cimsi');"
      
      # Para que webuser pueda realizar peticiones a la base de datos
      mysql -e "CREATE USER 'webuser'@'192.168.56.20' IDENTIFIED BY 'password_webuser';"
      mysql -e "GRANT ALL PRIVILEGES ON prueba_db.* TO 'webuser'@'192.168.56.20';"
      mysql -e "FLUSH PRIVILEGES;"

	  FECHA=$(date +%Y%m%d_%H%M%S)
	  ARCHIVO="/backup_db/dump_prueba_${FECHA}.sql"
	  mysqldump prueba_db > ${ARCHIVO}

    SHELL
  end


# 2. Servidor Web
  config.vm.define "nodo-web" do |web|
    web.vm.hostname = "nodo-web"
    web.vm.network "private_network", ip: "192.168.56.20"
    
    web.vm.provision "shell", inline: <<-SHELL
      apt-get update
      apt-get install -y apache2 mariadb-client
      
      # Ejemplo de peticion a la base de datos
      # mysql -h 192.168.56.30 -u webuser -ppassword_segura -e "SELECT * FROM prueba_db.users;" > /var/www/html/db_test.txt
    SHELL
  end
end
```
# Ejercicio 6
``` title="Vagrantfile"
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/focal64"

  # Servidor Base de Datos
  config.vm.define "nodo-db" do |db|
    db.vm.hostname = "nodo-db"
    db.vm.network "private_network", ip: "192.168.56.30"
    db.vm.synced_folder "./db_dumps", "/backup_db"
    
    db.vm.provision "shell", inline: <<-SHELL
      apt-get update
      apt-get install -y mariadb-server

      # Apertura del socket para permitir peticiones del nodo-web
      sed -i 's/bind-address.*/bind-address = 0.0.0.0/' /etc/mysql/mariadb.conf.d/50-server.cnf
      systemctl restart mariadb

      mysql -e "DROP DATABASE IF EXISTS prueba_db;"
      mysql -e "CREATE DATABASE prueba_db;"
      mysql -e "CREATE TABLE prueba_db.users (id INT AUTO_INCREMENT PRIMARY KEY, nombre VARCHAR(255));"
      mysql -e "INSERT INTO prueba_db.users (nombre) VALUES ('Admin'), ('Ubuntu'), ('Cimsi');"
      
      # Creación del usuario remoto (contraseña unificada)
      mysql -e "CREATE USER 'webuser'@'192.168.56.20' IDENTIFIED BY 'password_webuser';"
      mysql -e "GRANT ALL PRIVILEGES ON prueba_db.* TO 'webuser'@'192.168.56.20';"
      mysql -e "FLUSH PRIVILEGES;"

      FECHA=$(date +%Y%m%d_%H%M%S)
	  ARCHIVO="/backup_db/dump_prueba_${FECHA}.sql"
	  mysqldump prueba_db > ${ARCHIVO}
	  
    SHELL
  end

 
  # Servidor Web y Pruebas de Rendimiento
  config.vm.define "nodo-web" do |web|
    web.vm.hostname = "nodo-web"
    web.vm.network "private_network", ip: "192.168.56.20"
    
    # Montaje de la carpeta compartida para los informes de ab
    web.vm.synced_folder "./resultados", "/resultados", create: true
    
    web.vm.provision "shell", inline: <<-SHELL
      apt-get update
      # apache2-utils es obligatorio para el binario 'ab'
      apt-get install -y apache2 mariadb-client apache2-utils curl
      
      # 1. Prueba de conectividad cliente-servidor DB
      mysql -h 192.168.56.30 -u webuser -ppassword_webuser -e "SELECT * FROM prueba_db.users;" > /var/www/html/db_test.txt

      # 2. Control de flujo: Esperar a que el daemon HTTP responda en la IP asignada
      until curl -s http://192.168.56.20/ > /dev/null; do
        sleep 1
      done

      # 3. Creación del script de análisis de rendimiento
      cat << 'EOF' > /usr/local/bin/run_benchmark.sh
#!/bin/bash
ARCHIVO="/resultados/informe_rendimiento.txt"
IP_OBJETIVO="192.168.56.20"

echo "   ANÁLISIS DE RENDIMIENTO - APACHE BENCH" >> "${ARCHIVO}"
echo "=============================================" >> "${ARCHIVO}"
echo "Fecha de ejecución: $(date)" >> "${ARCHIVO}"
echo "Servidor objetivo: http://${IP_OBJETIVO}/" >> "${ARCHIVO}"
echo -e "\n" >> "${ARCHIVO}"

# Iteración sobre los niveles de concurrencia exigidos
for c in 1 2 4 8 16; do
    echo ">>> Prueba: n=1000, c=${c}" >> "${ARCHIVO}"
    echo "--------------------------------------------------------" >> "${ARCHIVO}"
    
    ab -n 1000 -c ${c} http://${IP_OBJETIVO}/ >> "${ARCHIVO}" 2>&1
    
    echo -e "\n" >> "${ARCHIVO}"
    # Pausa técnica para evitar la saturación de puertos efímeros en TIME_WAIT
    sleep 2
done
EOF

      # 4. Ejecución del test de estrés
      chmod +x /usr/local/bin/run_benchmark.sh
      /usr/local/bin/run_benchmark.sh
    SHELL
  end
end
```


# Ejercicio 7


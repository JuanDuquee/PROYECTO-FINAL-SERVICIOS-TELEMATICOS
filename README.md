# PROYECTO-FINAL-SERVICIOS-TELEMATICOS
BALANCEO DE CARGA CON MYSQL + MYSQLROUTER + DOCKER + SYSBENCH

USAMOS EL SIGUIENTE VAGRANTFILE:

# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  config.vm.define :mysqlrouter do |mysqlrouter|
    mysqlrouter.vm.box = "generic/centos8"
    mysqlrouter.vm.network :private_network, ip: "192.168.50.2"
    mysqlrouter.vm.provision "shell", path: "aprovision.sh.txt"
    mysqlrouter.vm.hostname = "mysqlrouter"
  end
  config.vm.define :mysql1 do |mysql1|
    mysql1.vm.box = "generic/centos8"
    mysql1.vm.network :private_network, ip: "192.168.50.3"
    mysql1.vm.provision "shell", path: "aprovision.sh.txt"
    mysql1.vm.hostname = "mysql1"
  end
   config.vm.define :mysql2 do |mysql2|
     mysql2.vm.box = "generic/centos8"
     mysql2.vm.network :private_network, ip: "192.168.50.4"
     mysql2.vm.provision "shell", path: "aprovision.sh.txt"
     mysql2.vm.hostname = "mysql2"
   end
   config.vm.define :mysql3 do |mysql3|
     mysql3.vm.box = "generic/centos8"
     mysql3.vm.network :private_network, ip: "192.168.50.5"
     mysql3.vm.provision "shell", path: "aprovision.sh.txt"
     mysql3.vm.hostname = "mysql3"
   end
   config.vm.define :cliente do |cliente|
     cliente.vm.box = "generic/centos8"
     cliente.vm.network :private_network, ip: "192.168.50.6"
     cliente.vm.provision "shell", path: "aprovision.sh.txt"
     cliente.vm.hostname = "cliente"
   end
end

CON ESTE VAGRANTFILE CENTOS 8 GENERIC YA VIENE INSTALADO EL VIM.

=======================================================================================================================================================

LO PRIMERO QUE DEBEMOS HACER AL INICIAR LAS MAQUINAS ES DESACTIVAR EL SELINUX EN CADA MAQUINA:

sudo vim /etc/selinux/config
SELINUX=disabled

SIEMPRE QUE INICIEMOS LAS MAQUINAS RECORDAR DESACTIVAR EL SERVICIO FIREWALLD EN TODAS LAS MAQUINAS!!!

=======================================================================================================================================================

INSTALACIÓN DE MYSQL (MAQUINAS MYSQLROUTER, MYSQL1, MYSQL2, MYSQL3) Y MYSQL SHELL (MYSQL1, MYSQL2, MYSQL3)

sudo yum update -y
sudo wget https://dev.mysql.com/get/mysql80-community-release-el7-7.noarch.rpm
sudo rpm -Uvh mysql80-community-release-el7-7.noarch.rpm
sudo yum install mysql-server -y
sudo yum install mysql-shell -y
sudo systemctl start mysqld
sudo systemctl status mysqld

mysql --version
mysqlsh --version

=======================================================================================================================================================

EMPEZAMOS POR LA CONFIGURACIÓN DE CADA MAQUINA:

==================EN LA MAQUINA MYSQL1===============================

En estas rutas vamos a asignar las IP y las maquinas las cuales van a estar comunicadas en el cluster:

vim /etc/hosts
127.0.0.1 mysql1 mysql1 reemplazar por 192.168.50.3 mysql1 mysql1
127.0.0.1 mysql2 mysql2 reemplazar por 192.168.50.4 mysql2 mysql2
127.0.0.1 mysql3 mysql3 reemplazar por 192.168.50.5 mysql3 mysql3

vim /etc/my.cnf
agregar report_host = mysql1
agregar report_host = mysql2
agregar report_host = mysql3

reiniciamos la maquina para efectuar cambios:
sudo reboot

como reiniciamos la maquina ya sabemos que debemos desactivar el firewall de nuevo:
service firewalld stop

iniciamos mysql:
sudo systemctl start mysqld

entramos a mysql shell con el usuario root sin contraseña:
mysqlsh --uri root@localhost (SIN CONTRASEÑA)

Configuramos el cluster con los siguientes pasos:
dba.configureInstance()
2 (la opción que debemos elegir)
innodbcluster (como se llamara la instancia)
contraseña_fuerte1! (la contraseña de la instancia)
y
y

Observamos la configuración de la instancia:
dba.checkInstanceConfiguration('innodbcluster@mysql1')
contraseña_fuerte1!
Y

Entramos a la shell depende de la maquina:
\c innodbcluster@mysql1:3306
contraseña_fuerte1!
Y

creamos nuestro cluster con un nombre:
var mycls= dba.createCluster('FUTURESOFT_CLS')

creamos una variable para obtener el cluster y con la cual obtendremos los datos:
var cls=dba.getCluster('FUTURESOFT_CLS')
cls.status()
cls.describe()

{
comandos que nos pueden servir:
dba.getCluster()
dba.dropMetadataSchema()
dba.rebootClusterFromCompleteOutage()
}

{
Para cuando volvamos a iniciar las maquinas y queramos obtener el cluster que creamos antes:
dba.rebootClusterFromCompleteOutage()
dba.getCluster()
var cls=dba.getCluster('FUTURESOFT_CLS') 
}

ahora agregamos las otras dos instancias, es decir, las otras dos maquinas:
cls.addInstance('mysql2:3306')
contraseña_fuerte1!
Y
(default clone): LE DAMOS ENTER

cls.addInstance('mysql3:3306')
contraseña_fuerte1!
Y
(default clone): LE DAMOS ENTER

Ahora vemos el estado del cluster:
cls.status() (deberiamos ver las tres maquinas, la primera con privilegios R/W y las demas R/O)

Asigmanos la instancia que tendra el mysql router:
cls.setupRouterAccount('mysqlrouter')
contraseña_fuerte1!

=======================================================================================================================================================

==================EN LA MAQUINA MYSQL2===============================


En estas rutas vamos a asignar las IP y las maquinas las cuales van a estar comunicadas en el cluster:

vim /etc/hosts
127.0.0.1 mysql1 mysql1 reemplazar por 192.168.50.3 mysql1 mysql1
127.0.0.1 mysql2 mysql2 reemplazar por 192.168.50.4 mysql2 mysql2
127.0.0.1 mysql3 mysql3 reemplazar por 192.168.50.5 mysql3 mysql3

vim /etc/my.cnf
agregar report_host = mysql1
agregar report_host = mysql2
agregar report_host = mysql3

reiniciamos la maquina para efectuar cambios:
sudo reboot

como reiniciamos la maquina ya sabemos que debemos desactivar el firewall de nuevo:
service firewalld stop

iniciamos mysql:
sudo systemctl start mysqld

entramos a mysql shell con el usuario root sin contraseña:
mysqlsh --uri root@localhost (SIN CONTRASEÑA)

Configuramos el cluster con los siguientes pasos:
dba.configureInstance()
2 (la opción que debemos elegir)
innodbcluster (como se llamara la instancia)
contraseña_fuerte1! (la contraseña de la instancia)
y
y

Observamos la configuración de la instancia:
dba.checkInstanceConfiguration('innodbcluster@mysql2')
contraseña_fuerte1!
Y

=======================================================================================================================================================

==================EN LA MAQUINA MYSQL3===============================

En estas rutas vamos a asignar las IP y las maquinas las cuales van a estar comunicadas en el cluster:

vim /etc/hosts
127.0.0.1 mysql1 mysql1 reemplazar por 192.168.50.3 mysql1 mysql1
127.0.0.1 mysql2 mysql2 reemplazar por 192.168.50.4 mysql2 mysql2
127.0.0.1 mysql3 mysql3 reemplazar por 192.168.50.5 mysql3 mysql3

vim /etc/my.cnf
agregar report_host = mysql1
agregar report_host = mysql2
agregar report_host = mysql3

reiniciamos la maquina para efectuar cambios:
sudo reboot

como reiniciamos la maquina ya sabemos que debemos desactivar el firewall de nuevo:
service firewalld stop

iniciamos mysql:
sudo systemctl start mysqld

entramos a mysql shell con el usuario root sin contraseña:
mysqlsh --uri root@localhost (SIN CONTRASEÑA)

Configuramos el cluster con los siguientes pasos:
dba.configureInstance()
2 (la opción que debemos elegir)
innodbcluster (como se llamara la instancia)
contraseña_fuerte1! (la contraseña de la instancia)
y
y

Observamos la configuración de la instancia:
dba.checkInstanceConfiguration('innodbcluster@mysql3')
contraseña_fuerte1!
Y

=======================================================================================================================================================

==================EN LA MAQUINA MYSQLROUTER=========================

para entrar al bash de mysql shell:
mysqlsh (CTRL + D para salir)

instalamos mysql router:
sudo yum install mysql-router -y

mysqlrouter --version

En estas rutas vamos a asignar las IP y las maquinas las cuales van a estar comunicadas con mysql router:

vim /etc/hosts
127.0.0.1 mysqlrouter mysqlrouter reemplazar por 192.168.50.2 mysqlrouter mysqlrouter
127.0.0.1 mysql1 mysql1 reemplazar por 192.168.50.3 mysql1 mysql1
127.0.0.1 mysql2 mysql2 reemplazar por 192.168.50.4 mysql2 mysql2
127.0.0.1 mysql3 mysql3 reemplazar por 192.168.50.5 mysql3 mysql3

vim /etc/my.cnf
agregar report_host = mysqlrouter
agregar report_host = mysql1
agregar report_host = mysql2
agregar report_host = mysql3

iniciamos mysql:
sudo systemctl start mysqld

iniciamos mysql router:
mysqlrouter --bootstrap innodbcluster@mysql1 --user mysqlrouter (abrira puertos de lectura y escritura)
contraseña_fuerte1!

systemctl start mysqlrouter 

=======================================================================================================================================================

==================EN LA MAQUINA MYSQL1===============================

Maquina Master! con privilegios R/W (Read and Write)

Ahora vamos a crear una base de datos y una tabla para comprobar el funcionamiento del cluster en mysql:

mysqlsh --uri root@localhost (enter. sin contraseña)

\sql

select @@hostname; (debera salir mysql1)

CREATE DATABASE UAO;
USE UAO;
CREATE TABLE if not exists UAO.estudiantes(id int primary key auto_increment, nombre varchar(100), carrera varchar(100), semestre int);
INSERT UAO.estudiantes(nombre, carrera, semestre) values ('Juan Duque','Ing.Informatica', 8);
INSERT UAO.estudiantes(nombre, carrera, semestre) values ('David Ordoñez','Ing.Informatica', 6);
INSERT UAO.estudiantes(nombre, carrera, semestre) values ('Santiago Castaño','Ing.Informatica', 10);
INSERT UAO.estudiantes(nombre, carrera, semestre) values ('Javier Vazquez','Ing.Informatica', 7);
SELECT * FROM UAO.estudiantes;

=======================================================================================================================================================

==================EN LA MAQUINA MYSQL2===============================

Maquina Esclava! con privilegios R/O (Read Only)

Comprobamos en lectura la creación de lo que hizo la maquina master con privilegios de escritura y lectura:

mysqlsh --uri root@localhost

\sql

select @@hostname; (debera salir mysql2)

SELECT * FROM UAO.estudiantes;

INSERT UAO.estudiantes(nombre, carrera, semestre) values ('Prueba','Ing.Informatica', 5); (NO DEBERIA PORQUE LA MAQUINA TIENE PRIVILEGIOS R/O)

=======================================================================================================================================================

==================EN LA MAQUINA MYSQL3===============================

Maquina Esclava! con privilegios R/O (Read Only)

Comprobamos en lectura la creación de lo que hizo la maquina master con privilegios de escritura y lectura:

mysqlsh --uri root@localhost

\sql

select @@hostname; (debera salir mysql3)

SELECT * FROM UAO.estudiantes;

INSERT UAO.estudiantes(nombre, carrera, semestre) values ('Prueba','Ing.Informatica', 5); (NO DEBERIA PORQUE LA MAQUINA TIENE PRIVILEGIOS R/O)

=======================================================================================================================================================

============AHORA EMPEZAMOS CON LA PRUEBA DE MYSQL ROUTER USANDO DOCKER=================

curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh
docker --version
systemctl start docker
systemctl status docker

docker network create innodbnet (d3df2d1dc42f74f55b1d60d29ac63b443e7d78822571f415356f8c002d02c9d9)

for N in 1 2 3
do docker run -d --name=mysql$N --hostname=mysql$N --net=innodbnet \
-e MYSQL_ROOT_PASSWORD=root mysql/mysql-server:8.0
done

docker images

docker ps -a

for N in 1 2 3
do docker exec -it mysql$N mysql -uroot -proot \
-e "CREATE USER 'clusteradmin'@'%' IDENTIFIED BY 'admin';" \
-e "GRANT ALL privileges ON *.* TO 'clusteradmin'@'%' with grant option;" \
-e "reset master;"
done

for N in 1 2 3
do docker exec -t mysql$N mysql -uclusteradmin -padmin \
-e "select @@hostname;" \
-e "SELECT user FROM mysql.user where user = 'clusteradmin';"
done

docker exec -it mysql1 mysqlsh -uroot -proot -S/var/run/mysqld/mysqlx.sock

\sql
select @@hostname;

\js
dba.checkInstanceConfiguration("clusteradmin@mysql1:3306")
admin
N

dba.configureInstance("clusteradmin@mysql1:3306")
admin
y
y

ahora afuera de mysql-shell

docker ps -a
docker start fe8a5cb488c1 (que es el mysql1 que esta exited) 

docker exec -it mysql1 mysqlsh -uroot -proot -S/var/run/mysqld/mysqlx.sock
dba.configureInstance("clusteradmin@mysql2:3306")
admin
y
y

dba.configureInstance("clusteradmin@mysql3:3306")
admin
y
y

docker ps -a
docker start a79ae1cb5967 (que es el mysql2 que esta exited) 
docker start 6521215f5d86 (que es el mysql3 que esta exited)

docker exec -it mysql1 mysqlsh -uroot -proot -S/var/run/mysqld/mysqlx.sock
dba.checkInstanceConfiguration("clusteradmin@mysql1:3306")
admin
N
dba.checkInstanceConfiguration("clusteradmin@mysql2:3306")
admin
N
dba.checkInstanceConfiguration("clusteradmin@mysql3:3306")
admin
N

{
docker restart mysql1 mysql2 mysql3 (si se necesita)
docker exec -it mysql1 mysqlsh -uroot -proot -S/var/run/mysqld/mysqlx.sock
}

vim /etc/hosts
127.0.0.1 mysqlrouter mysqlrouter reemplazar por 192.168.50.2 mysqlrouter mysqlrouter
127.0.0.1 mysql1 mysql1 reemplazar por 192.168.50.3 mysql1 mysql1
127.0.0.1 mysql2 mysql2 reemplazar por 192.168.50.4 mysql2 mysql2
127.0.0.1 mysql3 mysql3 reemplazar por 192.168.50.5 mysql3 mysql3
127.0.0.1 mysql3 mysql3 reemplazar por 192.168.50.6 cliente cliente

vim /etc/my.cnf
agregar report_host = mysqlrouter
agregar report_host = mysql1
agregar report_host = mysql2
agregar report_host = mysql3
agregar report_host = cliente

reboot

service firewalld stop

systemctl start docker
docker ps -a
docker start mysql1 mysql2 mysql3

LA IP QUE PROPORCIONAMOS A CONTINUACIÓN ES LA IP DE LA MAQUINA MYSQLROUTER QUE ES LA QUE HACE EL BALANCEO!!!

while [ 1 ]
do 
sleep 1 
docker exec -it mysql1 mysql -h 192.168.50.2 -P 6446 -uinnodbcluster -pcontraseña_fuerte1! -e "SELECT * FROM UAO.estudiantes;"
done

PARA PROBAR LA MAQUINA WRITE:
while [ 1 ]
do 
sleep 1 
docker exec -it mysql1 mysql -h 192.168.50.2 -P 6446 -uinnodbcluster -pcontraseña_fuerte1! -e "SELECT @@hostname;"
done

PARA PROBAR LAS MAQUINAS READ:
while [ 1 ]
do 
sleep 1 
docker exec -it mysql1 mysql -h 192.168.50.2 -P 6447 -uinnodbcluster -pcontraseña_fuerte1! -e "SELECT @@hostname;"
done

=======================================================================================================================================================

============AHORA HACEMOS UNA PRUEBA CUANDO UNA DE LAS MAQUINAS SE APAGA=================

-----------------------------------------------------
EN LA MAQUINA MYSQL1:

systemctl stop mysqld
-----------------------------------------------------
EN LA MAQUINA MYSQL2:

mysqlsh --uri root@localhost
\c innodbcluster@mysql2:3306
contraseña_fuerte1!

var cls=dba.getCluster('FUTURESOFT_CLS')
cls.status()
-----------------------------------------------------
HACEMOS LAS PRUEBAS EN LA MAQUINA CLIENTE CON DOCKER
-----------------------------------------------------
EN LA MAQUINA MYSQL1 NUEVAMENTE:

systemctl start mysqld
-----------------------------------------------------
EN LA MAQUINA MYSQL2 NUEVAMENTE:
cls.status()

CON ESTA PRUEBA LA MAQUINA MYSQL2 O MYSQL3 SERIA LA QUE QUEDARIA CON PRIVILEGIOS R/W
MIENTRAS QUE LAS MAQUINAS MYSQL1 Y MYSQL2 O MYSQL3 QUEDARIAN CON PRIVILEGIOS R/O

=======================================================================================================================================================
=======================================================================================================================================================
=======================================================================================================================================================

NOTA PARA CUANDO SE EJECUTE EL PROYECTO OTRO DIA. HACER ESTOS PASOS QUE NO SE OLVIDEN!!!!!!!!!!!!!!

INICIAR LAS MAQUINAS

En las maquinas mysql1 2 y 3

service firewalld stop
sudo systemctl start mysqld
mysqlsh --uri root@localhost (SIN CONTRASEÑA)
dba.checkInstanceConfiguration('innodbcluster@mysql1') (cambia de numero dependiendo de la maquina)
contraseña_fuerte1!
Y

\c innodbcluster@mysql1:3306 (cambia de numero dependiendo de la maquina)
contraseña_fuerte1!
Y

TENER TODAS LAS MAQUINAS PRENDIDAS CON LOS SERVICIOS PRENDIDOS!!

para la maquina master:
dba.rebootClusterFromCompleteOutage()
dba.getCluster()
var cls=dba.getCluster('FUTURESOFT_CLS') 
cls.status()

\c innodbcluster@mysql1:3306 (cambia de numero dependiendo de la maquina)
\sql
SELECT * FROM UAO.estudiantes;

MYSQL ROUTER ========================================

service firewalld stop
sudo systemctl start mysqld
sudo systemctl start mysqlrouter

mysqlrouter --bootstrap innodbcluster@mysql1 --user mysqlrouter
contraseña_fuerte1!

CLIENTE DOCKER ========================================

service firewalld stop

systemctl start docker
docker ps -a
docker start mysql1 mysql2 mysql3 (Revisar que esten en estado healthy)

LA IP QUE PROPORCIONAMOS A CONTINUACIÓN ES LA IP DE LA MAQUINA MYSQLROUTER QUE ES LA QUE HACE EL BALANCEO!!!

while [ 1 ]
do 
sleep 1 
docker exec -it mysql1 mysql -h 192.168.50.2 -P 6446 -uinnodbcluster -pcontraseña_fuerte1! -e "SELECT * FROM UAO.estudiantes;"
done

PARA PROBAR LA MAQUINA WRITE:
while [ 1 ]
do 
sleep 1 
docker exec -it mysql1 mysql -h 192.168.50.2 -P 6446 -uinnodbcluster -pcontraseña_fuerte1! -e "SELECT @@hostname;"
done

PARA PROBAR LAS MAQUINAS READ:
while [ 1 ]
do 
sleep 1 
docker exec -it mysql1 mysql -h 192.168.50.2 -P 6447 -uinnodbcluster -pcontraseña_fuerte1! -e "SELECT @@hostname;"
done

=======================================================================================================================================================

====================CONFIGURACIÓN SYSBENCH=======================================

sudo yum install sysbench -y

sysbench --version

sysbench --test=cpu --cpu-max-prime=20000 --num-threads=1 run

MODOS DE USAR SYSBENCH:

oltp_read_only
port = 3306
port router = 6446
--tables=10 
--table-size=100
--events=0 
--rate=5000
--time=30 
--threads=10

sysbench oltp_read_write --tables=100 --table-size=100000 --db-driver=mysql --mysql-host=localhost --mysql-db=UAO --mysql-user=root --mysql-port=6446 --time=30 --threads=16 prepare (preparar las pruebas)

sysbench oltp_read_write --tables=100 --table-size=100000 --db-driver=mysql --mysql-host=localhost --mysql-db=UAO --mysql-user=root --mysql-port=6446 --time=30 --threads=16 run (correr las pruebas)

sysbench oltp_read_only --tables=10 --table-size=100 --db-driver=mysql --mysql-host=localhost --mysql-db=UAO --mysql-user=root --mysql-port=6446 --time=30 --threads=10 run

sysbench oltp_read_write --tables=100 --table-size=100000 --db-driver=mysql --mysql-host=localhost --mysql-db=UAO --mysql-user=root --mysql-port=6446 --time=30 --threads=16 cleanup (para borrar las pruebas)

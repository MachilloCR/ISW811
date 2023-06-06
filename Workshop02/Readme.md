# Workshop02

## ¿Qué necesitamos para desplegar una aplicación?

- Servidor
- Dominio
- Una IP
- Una App (Backend, Frontend, fullstack)
- Una base de datos
- Presupuesto
- Seguridad
- Firewall
- SEO
- Analítica

## Implementación de servidor LAMP

## Iniciamos la maquina virtual

Para iniciar la maquina virtual hay que ir a la carpeta donde este ubicado el **Vagrantfile**
y poner el comando **vagrant up**.

![Vagrant Up]( images/vagrant_up.PNG "Vagrant Up")

## Conectarse a la maquina virtual por SSH
Una vez ya la maquina esta iniciada podemos conectarnos a la misma con el comando **vagrant ssh**

![Vagrant ssh]( images/vagrantssh.PNG "Vagrant ssh")

## Cambiar el hostname de la maquina virtual

Para cambiar el nombre del equipo ejecutamos el siguiente comando **'sudo hostnamectl set-hostname webserver'** dentro de la maquina virtual a la que se desea cambiar el nombre. y para ver reflejado el cambio debemos salirnos y volver a conectarnos a la maquina virtual.

```bash 
 sudo hostnamectl set-hostname webserver
 exit
 vagrant ssh
```
## Actualizar el hostname en el archivo hosts

Para finalizar el cambio del hostname hay que actualizar el nombre de la maquina en el archivo **hosts**. En GNU/Linux esta archivo se encuentra en /etc/hosts.

```bash
nano /etc/hosts
```
![Vagrant hostname]( images/changehostname.PNG "Vagrant hostname")

## Actualizar la lista de paquetes elegibles

Primeramente antes de instalar cualquier paquete hay que actualizar la lista de paquetes disponibles en la maquina virtual con el comando 'apt-get update'.
```bash
sudo apt-get update
```
## Instalar paquetes

Luego instalamos los paquetes necesarios, en este caso Vim, cURL, apache2, MySQL y PHP.

```bash
sudo apt-get install vim vim-nox 
curl git apache2 mariadb-server mariadb-client 
php7.4 php7.4-bcmath php7.4-curl php7.4-json 
php7.4-mbstring php7.4-mysql php7.4-xml
```
## Comprobar la ip del servidor

Desde la maquina anfitriona verificamos la ip definida en el Vagrantfile el parámetro **private_network** y le hacemos un **ping** para estar seguros.

![ip confirm]( images/ipconfirm.PNG "ip confirm")

```bash
ping 192.168.33.10

Haciendo ping a 192.168.33.10 con 32 bytes de datos:
Respuesta desde 192.168.33.10: bytes=32 tiempo=46ms TTL=64
Respuesta desde 192.168.33.10: bytes=32 tiempo=1ms TTL=64
Respuesta desde 192.168.33.10: bytes=32 tiempo=1ms TTL=64
Respuesta desde 192.168.33.10: bytes=32 tiempo=1ms TTL=64
```

## Agregar la entrada (para resolver el dominio que se desea simular)

En windows la dirección del archivo **hosts** es 'windows\system32\drivers\etc' y editamos el archivo hosts.

```bash
cd \
cd \windows\system32\drivers\etc
notepad hosts
```

en el archivo **hosts** agregamos la entrada correspondiente para simular la resolución del dominio deseado.**¡importante no modificar ninguna de las entradas ya existentes dentro del archivo!**

192.168.56.10 esteban.isw811.xyz

y verificamos que sirva poniendo en una ventana en incognito el nombre del dominio.

![Apache example]( images/apacherunning.PNG "Apache running")

## Habilitar módulos (para soportar varios sitios y https)

Ahora en la maquina virtual vamos a habilitar los módulos necesarios para poder soportar los **hosts** virtuales y certificados SSL con el siguiente comando.

```bash
sudo a2enmod vhost_alias rewrite ssl
```
y luego reiniciamos el apache2 con el comando

```bash
sudo systemctl restart apache2
```
## Crear carpeta de sitios

Para facilitar el trabajo creamos un folder en la maquina anfitriona y lo sincronizamos contra la ruta /home/vagrant/sites de la maquina virtual.

Debemos editar en el **Vagrantfile** la linea 46 o preferiblemente copiar y pegar la linea abajo y modificarla con lo siguiente.

![Sites Folder]( images/foldersites.PNG "Sites Folder")

una vez ya modificamos el **Vagrantfile** debemos reiniciar la maquina virtual para los cambios surtan efecto.

```bash
exit
vagrant halt
vagrant up
vagrant ssh
```
## Crear el archivo conf para el sitio

Por cada sitio que deseemos en el servidor web es necesario crearle un archivo **.conf**,
entonces primero creamos el folder **confs** desde la maquina anfitriona en el directorio de la maquina invitada y dentro de este folder **confs** creamos los archivos **.conf** 
```bash
mkdir confs
cd confs
touch esteban.isw811.xyz.conf
```
una vez ya creado el archivo lo abrimos y dentro copiamos lo siguiente
```html
<VirtualHost *:80>
	ServerAdmin webmaster@mizaq.isw811.xyz
	ServerName esteban.isw811.xyz

	# Indexes + Directory Root.
	DirectoryIndex index.php index.html
	DocumentRoot /home/vagrant/sites/esteban.isw811.xyz

	<Directory /home/vagrant/sites/esteban.isw811.xyz>
		DirectoryIndex index.php index.html
		AllowOverride All
		Require all granted
	</Directory>

	ErrorLog ${APACHE_LOG_DIR}/esteban.isw811.xyz.error.log
	LogLevel warn
	CustomLog ${APACHE_LOG_DIR}/esteban.isw811.xyz.access.log combined
</VirtualHost>
```

## Copiar conf a site-available 

Ahora debemos copiar el archivo **.conf** anteriormente creado y pegarlo en **sites-available** en apache2.

```bash
sudo cp /vagrant/confs/esteban.isw811.xyz.conf
/etc/apache2/sites-available
```

Después de agregar el archivo **.conf** verificamos que no exista ningún error con

```bash
sudo apache2ctl -t
```

y si todo esta bien nos devolverá **"Syntax OK"**. 

¡Este proceso se debe repetir cada que agregue un nuevo archivo **.conf**!

## Configurar parámetro ServerName

Si al momento de probar la configuración de apache nos muestra el siguiente error *Could not reliably
determine the server's fully qualified domain name*, hay que ejecutar el el siguiente comando, el cual agrega agrega la directiva de **ServerName** al archivo de configuración general de Apache.

```bash
echo "ServerName webserver" | sudo tee -a
/etc/apache2/apache2.conf
```

y volvemos a verificar que no exista ningún error con

```bash
sudo apache2ctl -t
```  
y si todo sale bien, podemos habilitar el sitio con 
```bash
sudo a2ensite esteban.isw811.xyz.conf
``` 
y volvemos a reiniciar Apache para ver los cambios.

```bash
sudo systemctl restart apache2.service
``` 

## Probar el nuevo sitio

Si todo salio bien podemos visualizar el sitio desde la maquina anfitriona visitando el URL
**http://esteban.isw811.xyz** y en lugar de cargar la pagina por defecto de apache cargará la del nuevo sitio


![New site]( images/new-site-up.PNG "New site")
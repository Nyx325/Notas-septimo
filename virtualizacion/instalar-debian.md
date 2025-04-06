# Pasos pa crear VM

## Crear vm

1. Seleccionar iso de instalación
2. Desactivar `instalación desatendidad`
3. Seleccionar ram, cpu y disco a como quieras

## Iniciar por primera vez la VM en el ISO

1. Seleccionar graphical install
2. Seleccionar idioma (Inglés recomendao)
3. Seleccionar localización (México)
4. Seleccionar distribución teclado (latam)
5. Elegir nombre de la maquina
6. Colocar vacio el dominio
7. Colocar la contraseña de root
8. Colocar nombre de usuario personal
9. Crear contraseña para ese usuario creado
10. Seleccionar region de reloj (Eastern supongo)
11. Seleccionar usar el disco completo para la instalación
12. Seleccionar el disco a usar
13. Indicar que todas los archivos estén en la misma partición
14. Confirmar particionado
15. Marcar que si a escribir los cambios en el disco
16. Marcar a que no a escanear `extra installation media`
17. Marcar que sí a configurar package manager
18. Marcar que no a lo de participar en package usage survey
19. Marcar solo standard system utilities
20. Marcar que si a la instalación de grub
21. Seleccionar /dev/sda

## Primer encendido post instalacion

1. Meterse a `/etc/apt/sources.list` y activar
   los paquetes privativos agregando `contrib` y
   `non-free` a los mirrors existentes y
   comentar el mirror de la imagen iso
2. Actualizar repositorios con `apt update` y actualizar
   los paquetes instalados si es necesario con `apt upgrade`
3. Instalar `sudo` con `apt`
4. Agregar el usuario personal al archivo `/etc/sudoers`

## Configurar SSH

1. Instalar `openssh-server` y `openssh-client`

```bash
sudo apt update && sudo apt upgrade # Buscar actualizaciones en caso de haber
sudo apt install openssh-server openssh-client
```

2. Comprobar que el estado esté activo

```bash
sudo systemctl status mariadb
# O
sudo service mariadb status
```

Si no está inactivo

```bash
sudo systemctl enable mariadb # Iniciar servicio en arranque
sudo systemctl start mariadb # Iniciar servicio ahora

# o
sudo service mariadb start # Iniciar servicio ahora
sudo update-rc.d mariadb enable # Iniciar servicio en arranque
```

3. Conectarse desde otra compu con

```bash
ssh usuario@IpVm
```

## Instalar y configurar Mariadb

1. Instalar el servidor y cliente de mariadb

```bash
sudo apt update && sudo apt upgrade # Buscar actualizaciones en caso de haber
sudo apt install mariadb-server mariadb-client
```

2. Comprobar si el servicio está activo.

```bash
sudo systemctl status mariadb
# O
sudo service mariadb status
```

3. Ingresar como root a mariadb

```bash
sudo mariadb
```

4. Crear un usuario con privilegios

```sql
CREATE USER 'usuario'@'%' IDENTIFIED BY 'contraseña'
```

- `%` Signfica cualquier IP

5. Añadirle privilegios especificos `SELECT`, `UPDATE`, `DELETE`, etc a una BD y tabla especificos

```sql
GRANT SELECT, UPDATE, DELETE ON DB.Table TO 'usuario'@'%' WITH GRANT OPTION
```

o añadirle todos los permisos a todas las BDs y Tablas

```sql
GRANT ALL PRIVILEGES ON *.* TO 'usuario'@'%' WITH GRANT OPTION;
```

Refrescar los privilegios, de otro modo tendríamos que apagar y volver a prender el servicio

```sql
FLUSH PRIVILEGES;
```

6. Validar acceso

```bash
mariadb -u usuario -p
```

7. Configurar para permitir conexión remota a `mariadb`, para eso modificamos
   el archivo

```bash
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
```

Tendremos algo como esto:

```text
# Instead of skip-networking the default is now to listen only on
# localhost which is more compatible and is not less secure.
bind-address            = 127.0.0.1
```

Podemos comentar la unica linea no comentada o cambiarlo por:

```text
bind-address            = 0.0.0.0
```

posteriormente reiniciamos el servicio

```bash
sudo systemctl restart mariadb
# O
sudo service mariadb restart
```

## Configurar apache

1. Instalar dependencias Apache

```bash
sudo apt-get install apache2 apache2-utils php8.2 \
php8.2-bcmath php8.2-cgi php8.2-cli php8.2-curl  \
php8.2-enchant php8.2-imagick php8.2-http \
php8.2-imagick php8.2-intl php8.2-maxminddb \
php8.2-mbstring php8.2-mcrypt php8.2-mysql php8.2-ps \
php8.2-pspell php8.2-psr php8.2-readline \
php8.2-sqlite3 php8.2-xml php8.2-xmlrpc php8.2-xsl \
php8.2-zip libapache2-mod-php
```

2. Revisar que el servicio esté activo

```bash
systemctl status apache2
```

3. Abrimos la url `http://ip-maquina` y deberia salir algo

4. Validamos que PHP funcione creando un archivo cualquiera, ej
   `prueba.php` dentro de la carpeta `/var/www/html`

```php
<?php
//prueba.php
phpinfo()
```

Y entrar a `http://ip-maquina/prueba.php` debe salir una pagina mostrando info de php

Todo lo que pongamos en la carpeta de `html` será accesible desde
la url

5. Crear una carpeta local para poner nuestros proyectos sin
   necesidad de ser super usuarios (usuario root o usar sudo)

```bash
# ir a la carpeta de nuestro usuario, este comando lo pueden
# poner desde cualquier carpeta y funciona
cd /home/nombre-usuario-personal

# o
cd
```

Crear una carpeta

```bash
mkdir web-projects
cd web-projects
```

Crear una carpeta en `html`

```bash
sudo mkdir /var/www/html/projects
```

Montamos la carpeta de nuestro usuario `web-projects` en la
carpeta `html/projects`. Cuando apaguemos la maquina, se
desmontará la carpeta y tendremos que volverlo a hacer

```bash
sudo mount --bind /home/nombre-usuario-personal/web-projects /var/www/html/projects
```

## Subir archivos al server mediante FTP

Usar algo como `filezzilla` de host colocas la IP del server, usuario
el usuario personal y la contraseña de ese usuario, y puerto `22`

con eso ya puedes subir archivos mediante FTP

# Maestro esclavo

Cuando hablamos de replicaciones en una arquitectura maestro
esclavo, hablamos de una clonación total de mis máquinas virtuales
además debe poder recibir cambios en tiempo real, es decir, si creo
una base de datos en el servidor maestro, todos los esclavos deben
recibir esos cambios en tiempo real, sin embargo estos esclavos
aunque realicen cambios, no van a replicar estos cambios en los demás
esclavos o en el maestro

**Ventajas**

- Si obtenemos algun error en el servidor maestro considerando el caso
  de una BD, dejamos de poder admitir inserciones pero seguimos permitiendo
  hacer consultas

## Crear arquitectura con VM's

### En maestro

Vamos a clonar dos maquinas virtuales de nuestra base.

1. Como ambas computadoras tienen el mismo hostname vamos a entrar
   a `/etc/hostname` y modificaremos el archivo para colocar el nuevo
   nombre de la VM.
2. Haremos el mismo cambio en el archivo `/etc/hosts`
3. Reiniciamos ambas maquinas
4. En el archivo `/etc/mysql/mariadb.conf.d/50-server.cnf` agregamos lo siguiente
   debajo de las lineas que indican algo como `sudo mkdir`

```config
server-id = 1
log_bin = /var/log/mysql/mysql-bin.log
```

5. Despues ejecutamos los comandos que menciona el archivo

```
sudo mkdir -m 2750 /var/log/mysql
sudo chown mysql /var/log/mysql
```

6. Reiniciamos el servicio
7. Entramos a `mariadb` y creamos un usuario de replicación

```sql
CREATE USER 'replica'@'%' IDENTIFIED BY 'replica';
GRANT REPLICATION SLAVE ON *.* TO 'replica'@'%';
FLUSH PRIVILEGES;
```

8.

```sql
show master status \g;
```

Debe salir algo como:

```sql
+------------------+----------+--------------+------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------------+----------+--------------+------------------+
| mysql-bin.000001 |      766 |              |                  |
+------------------+----------+--------------+------------------+
```

### En el esclavo

1. En el archivo `/etc/mysql/mariadb.conf.d/50-server.cnf` agregamos lo mismo
   que en el maestro cambiando el `server-id` por `2`

```config
server-id = 2
log_bin = /var/log/mysql/mysql-bin.log
```

2. Creamos las mismas carpetas

```
sudo mkdir -m 2750 /var/log/mysql
sudo chown mysql /var/log/mysql
```

3. Reiniciamos el sistema

4. En `mariadb`:

```sql
#Detener los hilos esclavos:
STOP SLAVE;

#Colocamos el server maestro
CHANGE MASTER TO  MASTER_HOST='ip del maestro',
MASTER_USER='replica',
MASTER_PASSWORD='replica',
MASTER_LOG_FILE='mysql-bin.000001',
MASTER_LOG_POS=328;
```

Ejemplo:

```sql
#Detener los hilos esclavos:
STOP SLAVE;

#Colocamos el server maestro
CHANGE MASTER TO  MASTER_HOST='172.23.50.189',
MASTER_USER='replica',
MASTER_PASSWORD='replica',
MASTER_LOG_FILE='mysql-bin.000001',
MASTER_LOG_POS=766;
```

5. Reiniciamos el servicio

Con esto con crear una base de datos o tabla en el maestro
se ven reflejados los registros en la máquina esclavo

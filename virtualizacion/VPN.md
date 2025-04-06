Instalar `wireguard` para configurar túneles, `wireguard` facilita la instalación
pero complica la administracion, recomendable para no más de 10 host, para cosas
más complicadas recomiendan `openvpn`

```text
$ sudo apt install wireguard wireguard-tools openresolv iptables iptables-persistent
```

Los archivos de configuración se encuentra en `/etc/wireguard`, se encontrará vacío,
debemos generar la clave y candado como las puertas de nuestra casa

```text
# cd /etc/wireguard
# wg genkey | tee privatekey | wg pubkey > publickey
```

Esto generará una clave pública y privada en dos archivos, repetir el proceso `1+n`
veces siendo `n` el número de host, almacenar las claves en donde no se pierdan:

```text
# Claves servidor
pri -> sOs9zH613w18HUD6NVIPRU6Jvql4LVuQCcznZghqr14=
pub -> mNn0YO7A+7M5ZnusyFiJAAkOLhz7466XSx0hScUu0jk=

# Claves del cliente 01
pri -> GMcGWfOz3weMC1PZvInxBVO2mS4iD3Hj56Q/SITseUM=
pub -> yWXJIEvVDMwPXEUtx85vQ5lGnjfMpkitHcZ3j4Afhis=

# Claves del cliente 02
pri -> 6J1RxXz+j4Na87qBrWlqct3fj+46PFACDaW2i9R4mXc=
pub -> IM3AVPvyd+D481wjhLvbHkEnS6TBMY/cn5oAmi1gkW0=

```

A las `vpn` se les suele configurar una IP clase A subneteada como clase C.
Para configurar esto vamos a configurar lo siguiente:

```
[Interface]
Address = {ipVpnServer}
PrivateKey = {privateKeyServer}
ListenPort = 51820

#Rutas para manejar el trafico y tener internet
PostUp = sysctl -w net.ipv4.ip_forward=1;  iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT; iptables -t nat -A POSTROUTING -o {interfazDeRed} -j MASQUERADE
PostDown = sysctl -w net.ipv4.ip_forward=0; iptables -D FORWARD -i %i -j ACCEPT; iptables -D FORWARD -o %i -j ACCEPT; iptables -t nat -D POSTROUTING -o {interfazDeRed} -j MASQUERADE

# PC Cliente 1
[Peer]
PublicKey = {publickeyCliente}
AllowedIPs = {ipVpnCliente}

# PC Cliente 2
[Peer]
PublicKey = {publickeyCliente}
AllowedIPs = {ipVpnCliente}
```

- **ipVpnServer:** Ip que tendrá el servidor, por ej `10.0.0.1`
- **interfazDeRed:** La interfaz con la que accede a internet el servidor
- **publickeyCliente** Public key del cliente a configurar
- **ipVpnCliente** Ip del cliente a configurar, ips a partir de la `.1`

Ejemplo:

```
[Interface]
Address = 10.0.0.1
PrivateKey = sOs9zH613w18HUD6NVIPRU6Jvql4LVuQCcznZghqr14=
ListenPort = 51820

#Rutas para manejar el trafico y tener internet
PostUp = sysctl -w net.ipv4.ip_forward=1;  iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT; iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE
PostDown = sysctl -w net.ipv4.ip_forward=0; iptables -D FORWARD -i %i -j ACCEPT; iptables -D FORWARD -o %i -j ACCEPT; iptables -t nat -D POSTROUTING -o enp0s3 -j MASQUERADE

# PC cliente 1
[Peer]
PublicKey = yWXJIEvVDMwPXEUtx85vQ5lGnjfMpkitHcZ3j4Afhis=
AllowedIPs = 10.0.0.2

# PC cliente 2
[Peer]
PublicKey = IM3AVPvyd+D481wjhLvbHkEnS6TBMY/cn5oAmi1gkW0=
AllowedIPs = 10.0.0.3

```

Guardar en `/etc/wireguard/wgvpn01.conf`

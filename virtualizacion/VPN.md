# Configuracion VPN con Wireguard

## Configuracion del lado del servidor

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
Para configurar esto vamos a crear un archivo con lo siguiente:

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

### Administrar el servicio

- Iniciar → `systemctl start wg-quick@wgvpn01`
- Estatus → `systemctl status wg-quick@wgvpn01`
- Detener → `systemctl stop wg-quick@wgvpn01`

## Configuracion de lado del cliente

```
[Interface]
PrivateKey = {privatekeyDelCliente}
Address = {ipVpnDelCliente}
DNS = 8.8.8.8
ListenPort = 51820

[Peer]
PublicKey = {publickeyDelServer}
AllowedIPs = 0.0.0.0/0
Endpoint = {ipRealDelServer}:{puertoDelServer}
PersistentKeepalive = 30
```

- Donde `ipRealDelServer` hace referencia a la IP pública
  de nuestro servidor, en caso de ser un equipo en local o
  tu red doméstica, deberás configurar una regla `NAT` o
  `Port Forwarding` para que tu router redireccione las
  solicitudes al puerto de wireguard a tu equipo que es el
  sevidor VPN

## Notas extra

### Error 1 No se pudo configurar el DNS

Estando en `archlinux` obtuve el siguiente error al
activar mi conexión:

```
# wg-quick up wgvpn01_client.conf
[#] ip link add wgvpn01_client type wireguard
[#] wg setconf wgvpn01_client /dev/fd/63
[#] ip -4 address add 10.0.0.2 dev wgvpn01_client
[#] ip link set mtu 1420 up dev wgvpn01_client
[#] resolvconf -a wgvpn01_client -m 0 -x
Failed to set DNS configuration: Could not activate remote peer 'org.freedesktop.resolve1': activation request failed: unknown unit
[#] ip link delete dev wgvpn01_client
```

**Mi fuente confiable indica lo siguiente:**

El error que estás viendo se debe a que WireGuard (a través
de wg-quick) intenta configurar el DNS usando systemd-resolved,
pero en tu sistema ese servicio no está activo o instalado.
Para solucionarlo tienes dos opciones:

1. **Habilitar systemd-resolved:**  
   Si quieres que wg-quick configure automáticamente el DNS,
   debes asegurarte de que el servicio systemd-resolved esté
   activo. En Debian 12 puedes activarlo con:

   ```bash
   sudo systemctl enable --now systemd-resolved
   ```

Esto debería permitir que la línea `DNS = 8.8.8.8` se aplique
sin problemas.

2. **Eliminar la línea de DNS:**  
    Si prefieres no usar systemd-resolved o ya gestionas el DNS
   de otra forma, puedes quitar (o comentar) la línea
   `DNS = 8.8.8.8` en tu archivo de configuración. De esta manera,
   wg-quick no intentará configurar el DNS y evitarás ese error.

Ambas soluciones son válidas; elige la que mejor se adapte a tu configuración y necesidades.

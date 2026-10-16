## 1. Objetivo de la práctica

El objetivo de esta práctica es configurar un servidor DNS en Ubuntu para que los equipos de nuestra red puedan comunicarse utilizando nombres de dominio en lugar de tener que recordar sus direcciones IP.

# Configuración de Servidor DNS BIND9 y Cliente en Ubuntu

Esta guía detalla la implementación, configuración y comprobación de un servicio de nombres de dominio (DNS) utilizando **BIND9** en la red con IP del servidor `172.16.6.170`, cliente `172.16.6.171` y puerta de enlace `172.16.6.1`.

---

## 1. Topología y Estructura de la Red

- **Servidor DNS / DHCP**: IP `172.16.6.170`
- **Cliente**: IP `172.16.6.171`
- **Puerta de enlace**: `172.16.6.1`
- **Subred**: `172.16.6.0/24`

---

## 2. Configuración en la Máquina Servidor (BIND9)

### Paso 2.1: Instalación de BIND9
Actualiza los repositorios e instala BIND9 y sus utilidades de comprobación:
    sudo apt update
    sudo apt install bind9 bind9-utils -y

### Paso 2.2: Configuración de Opciones Globales (`named.conf.options`)
Edita el archivo de configuración global:
    sudo nano /etc/bind/named.conf.options
Configura los parámetros de escucha, consultas y reenvío (*forwarders*):
    options {
        directory "/var/cache/bind";

        listen-on { 127.0.0.1; 172.16.6.170; };  # IP del Servidor DNS
        allow-query { any; };
        recursion yes;
        allow-recursion { 172.16.6.0/24; localhost; };

        forwarders {
            8.8.8.8;
            1.1.1.1;
        };

        dnssec-validation auto;
    };

### Paso 2.3: Declaración de la Zona Local (`named.conf.local`)
Añade la definición del dominio maestro en el archivo local:
    sudo nano /etc/bind/named.conf.local
Añade el siguiente bloque:
    zone "tudominio.local" {
        type master;
        file "/etc/bind/db.tudominio.local";
    };

### Paso 2.4: Creación del Archivo de Registros (`db.tudominio.local`)
Crea y edita el archivo de zona para definir los registros de recursos:
    sudo nano /etc/bind/db.tudominio.local
Introduce la siguiente estructura de registros:
    $TTL 86400
    @   IN  SOA ns1.tudominio.local. admin.tudominio.local. (
                  2026101301 ; Serial
                  600        ; Refresh
                  120        ; Retry
                  604800     ; Expire
                  86400 )    ; Minimum TTL
    
    @       IN  NS  ns1.tudominio.local.
    @       IN  A   172.16.6.170   ; IP del servidor DNS
    ns1     IN  A   172.16.6.170   ; Nombre del servidor DNS
    cliente IN  A   172.16.6.171   ; IP de la máquina cliente

### Paso 2.5: Verificación y Reinicio del Servicio
Comprueba que la sintaxis sea correcta y reinicia el servicio BIND9:
    sudo named-checkconf
    sudo named-checkzone tudominio.local /etc/bind/db.tudominio.local
    sudo systemctl restart named
    sudo systemctl enable named

---

## 3. Configuración del Servidor DHCP

Para automatizar la entrega de IP y DNS a los clientes de la red:
    sudo nano /etc/dhcp/dhcpd.conf
Asegura los parámetros de la subred:
    subnet 172.16.6.0 netmask 255.255.255.0 {
      range 172.16.6.171 172.16.6.180;
      option routers 172.16.6.1;
      option domain-name-servers 172.16.6.170;
      option domain-name "tudominio.local";
    }
Reinicia el servicio DHCP:
    sudo systemctl restart isc-dhcp-server

---

## 4. Configuración y Comprobación en la Máquina Cliente

### Paso 4.1: Configurar el Servidor DNS en el Cliente
En la máquina cliente (`172.16.6.171`), asegúrate de que el DNS primario apunte a la IP del servidor (`172.16.6.170`). Si utilizas `systemd-resolved`:
    sudo resolvectl dns <tu_interfaz_de_red> 172.16.6.170
    sudo resolvectl domain <tu_interfaz_de_red> tudominio.local

### Paso 4.2: Pruebas de Verificación
Desde la terminal de la máquina cliente, ejecuta las siguientes comprobaciones para validar el correcto funcionamiento:

1. Resolver el registro del cliente:
       dig @172.16.6.170 cliente.tudominio.local

2. Resolver el dominio raíz:
       dig @172.16.6.170 tudominio.local

3. Comprobar resolución externa (Internet vía Forwarders):
       dig @172.16.6.170 google.com

4. Verificación rápida con `nslookup`:
       nslookup cliente.tudominio.local 172.16.6.170
       dig @172.16.5.216 google.com

4. Verificación rápida con `nslookup`:
       nslookup navarro.navarro.edu 172.16.5.216

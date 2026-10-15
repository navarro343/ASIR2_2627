## 1. Objetivo de la práctica

El objetivo de esta práctica es configurar un servidor DNS en Ubuntu para que los equipos de nuestra red puedan comunicarse utilizando nombres de dominio en lugar de tener que recordar sus direcciones IP.

Paso 1: Instalar BIND9
En la terminal de tu máquina servidor, ejecuta:

Bash
sudo apt update
sudo apt install bind9 bind9utils -y
Paso 2: Configurar las Opciones Globales (named.conf.options)
Abre el archivo de opciones:

Bash
sudo nano /etc/bind/named.conf.options
Asegúrate de que quede configurado así (reemplazando la IP por la de tu servidor):

Fragmento de código
options {
    directory "/var/cache/bind";

    listen-on { 127.0.0.1; 172.16.5.216; };  # IP de tu servidor DNS
    allow-query { any; };
    recursion yes;
    allow-recursion { 172.16.0.0/16; localhost; };

    forwarders {
        8.8.8.8;
        1.1.1.1;
    };

    dnssec-validation auto;
};
(Guarda con Ctrl + O, Enter y sal con Ctrl + X).

Paso 3: Declarar la Zona Local (named.conf.local)
Abre el archivo de configuración local:

Bash
sudo nano /etc/bind/named.conf.local
Añade la zona de tu dominio:

Fragmento de código
zone "mired.local" {
    type master;
    file "/etc/bind/db.mired.local";
};
Paso 4: Crear el Archivo de Zona (db.mired.local)
Crea y edita el archivo de registros de la zona desde cero:

Bash
sudo nano /etc/bind/db.mired.local
Pega la siguiente estructura (aquí puedes incluir tu equipo cliente o el servidor):

Plaintext
$TTL 86400
@   IN  SOA ns1.mired.local. admin.mired.local. (
              2026101301 ; Serial
              600        ; Refresh
              120        ; Retry
              604800     ; Expire
              86400 )    ; Minimum TTL

@       IN  NS  ns1.mired.local.
@       IN  A   172.16.5.216   ; IP del servidor DNS
ns1     IN  A   172.16.5.216   ; Nombre del servidor DNS
navarro IN  A   172.16.5.171   ; IP de tu máquina cliente (ej: navarro)
(Guarda y sal).

Paso 5: Validar y Reiniciar el Servicio
Comprueba que no hay errores de sintaxis y reinicia el servicio:

Bash
sudo named-checkconf
sudo named-checkzone mired.local /etc/bind/db.mired.local
sudo systemctl restart named
Paso 6: Abrir el Firewall
Permite el tráfico de DNS en el cortafuegos del servidor:

Bash
sudo ufw allow Bind9
PARTE 2: Configuración en la Máquina Cliente (Ubuntu Cliente)
Para que tu máquina cliente (navarro, con IP 172.16.5.171) utilice este servidor DNS en lugar de los públicos, tienes dos opciones: configurarlo manualmente (vía Netplan o configuración de red) o mediante DHCP. Para la prueba directa, configúralo en el cliente:

Apunta el DNS al servidor: En la configuración de red de tu cliente (o mediante Netplan en /etc/netplan/...), cambia el servidor DNS primario a la IP de tu servidor (172.16.5.216).

Alternativa rápida temporal (si usas systemd-resolved en el cliente):

Bash
sudo resolvectl dns <tu_interfaz_de_red> 172.16.5.216
sudo resolvectl domain <tu_interfaz_de_red> mired.local
PARTE 3: Comprobación y Verificación desde la Máquina Cliente
Ve a la terminal de tu máquina cliente y realiza las siguientes pruebas para comprobar que todo funciona:

Comprobar la resolución del dominio local (ej: el propio servidor o el cliente):

Bash
dig @172.16.5.216 navarro.mired.local
Resultado esperado: Deberá mostrar status: NOERROR y en la sección ANSWER SECTION la IP 172.16.5.171.

Comprobar la resolución del dominio raíz local:

Bash
dig @172.16.5.216 mired.local
Resultado esperado: Debe mostrar la IP del servidor (172.16.5.216).

Comprobar la navegación/resolución externa (gracias a los forwarders):

Bash
dig @172.16.5.216 google.com
Resultado esperado: Debe resolver la IP pública de Google, confirmando que tu servidor DNS redirige correctamente las consultas de internet.

Prueba general con nslookup:

Bash
nslookup navarro.mired.local 172.16.5.216

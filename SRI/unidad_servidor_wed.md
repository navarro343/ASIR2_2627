# Documentación Completa: Red Interna en Ubuntu con BIND9, Apache (Puerto 8080) y Cliente

Esta guía detalla paso a paso la implementación completa de un entorno de red interno sobre máquinas virtuales Ubuntu, integrando un servidor DNS con **BIND9**, un servidor web **Apache** sirviendo una página personalizada en el **puerto 8080** bajo el dominio oficial (`navarro.asir2.iespoveda.local`), y una máquina cliente configurada para la resolución de nombres y acceso al servicio.

---

## 1. Topología y Parámetros de la Red

- **Subred**: `172.16.6.0/24`
- **Servidor (DNS y Apache)**: `172.16.6.170`
- **Cliente**: `172.16.6.171`
- **Puerta de Enlace (Gateway)**: `172.16.6.1`
- **Dominio Oficial**: `navarro.asir2.iespoveda.local`

---

## 2. Configuración en la Máquina Servidor (`172.16.6.170`)

### Paso 2.1: Instalación de BIND9 y Apache2
Actualiza los repositorios e instala las herramientas de red, BIND9 y Apache:
```bash
sudo apt update
sudo apt install bind9 bind9-utils apache2 -y
```

### Paso 2.2: Configuración Global de BIND9 (`named.conf.options`)
Edita el archivo de opciones globales para configurar los reenvíos y restricciones de red:
```bash
sudo nano /etc/bind/named.conf.options
```
Añade la siguiente configuración:
```named
options {
    directory "/var/cache/bind";

    listen-on { 127.0.0.1; 172.16.6.170; };
    allow-query { any; };
    recursion yes;
    allow-recursion { 172.16.6.0/24; localhost; };

    forwarders {
        8.8.8.8;
        1.1.1.1;
    };

    dnssec-validation auto;
};
```

### Paso 2.3: Declaración de la Zona Local (`named.conf.local`)
Registra la zona maestra para tu dominio:
```bash
sudo nano /etc/bind/named.conf.local
```
Añade el siguiente bloque:
```named
zone "navarro.asir2.iespoveda.local" {
    type master;
    file "/etc/bind/db.navarro.asir2.iespoveda.local";
};
```

### Paso 2.4: Creación del Archivo de Registros de Zona
Crea y edita el archivo de mapeo de nombres:
```bash
sudo nano /etc/bind/db.navarro.asir2.iespoveda.local
```
Introduce los registros de recursos (SOA, NS y registros A):
```text
$TTL 86400
@   IN  SOA ns1.navarro.asir2.iespoveda.local. admin.navarro.asir2.iespoveda.local. (
              2026101301 ; Serial
              600        ; Refresh
              120        ; Retry
              604800     ; Expire
              86400 )    ; Minimum TTL

@       IN  NS  ns1.navarro.asir2.iespoveda.local.
@       IN  A   172.16.6.170   ; IP del servidor DNS y Web
ns1     IN  A   172.16.6.170   ; Nombre del servidor DNS
www     IN  A   172.16.6.170   ; Resolución para el servidor web
cliente IN  A   172.16.6.171   ; IP de la máquina cliente
```

### Paso 2.5: Verificación y Reinicio de BIND9
Comprueba la sintaxis de los archivos de zona y arranca el servicio:
```bash
sudo named-checkconf
sudo named-checkzone navarro.asir2.iespoveda.local /etc/bind/db.navarro.asir2.iespoveda.local
sudo systemctl restart named
sudo systemctl enable named
```

---

## 3. Configuración del Servidor Web Apache en el Puerto 8080

### Paso 3.1: Configuración de Puertos de Apache (`ports.conf`)
Indica a Apache que escuche las peticiones en el puerto `8080`:
```bash
sudo nano /etc/apache2/ports.conf
```
Asegúrate de que contenga:
```apache
Listen 8080
```

### Paso 3.2: Configuración del VirtualHost (`000-default.conf`)
Edita el archivo de configuración por defecto del sitio web:
```bash
sudo nano /etc/apache2/sites-available/000-default.conf
```
Configura el bloque para que escuche en el puerto `8080` y vincule tu dominio:
```apache
<VirtualHost *:8080>
    ServerAdmin webmaster@localhost
    ServerName navarro.asir2.iespoveda.local
    ServerAlias www.navarro.asir2.iespoveda.local
    DocumentRoot /var/www/html

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

### Paso 3.3: Creación de la Página Web Limpia (`index.html`)
Elimina la página por defecto preinstalada y crea un archivo limpio desde cero:
```bash
sudo rm /var/www/html/index.html
sudo nano /var/www/html/index.html
```
Introduce una estructura HTML clara:
```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Servidor Web en Puerto 8080</title>
</head>
<body>
    <h1>¡Bienvenido a mi servidor web!</h1>
    <p>Sitio alojado en el servidor y servido mediante Apache en el puerto 8080.</p>
</body>
</html>
```

### Paso 3.4: Reinicio de Apache
Aplica los cambios en el servidor web:
```bash
sudo systemctl restart apache2
sudo systemctl enable apache2
```

---

## 4. Configuración y Comprobación en la Máquina Cliente (`172.16.6.171`)

### Paso 4.1: Configuración del Servidor DNS
En la máquina cliente, apunta la interfaz de red (reemplaza `enp0s3` por el nombre real de tu tarjeta de red si difiere) al servidor DNS local:
```bash
sudo resolvectl dns enp0s3 172.16.6.170
sudo resolvectl domain enp0s3 navarro.asir2.iespoveda.local
```

### Paso 4.2: Pruebas de Verificación
1. **Comprobar la resolución de nombres:**
   ```bash
   dig www.navarro.asir2.iespoveda.local
   ```
   *(Debe retornar la IP del servidor `172.16.6.170` en la respuesta).*

2. **Acceso al sitio web:**
   Abre el navegador web en la máquina cliente e introduce la URL completa incluyendo el puerto:
   ```text
   http://www.navarro.asir2.iespoveda.local:8080
   ```











   # Configuración de Réplica DNS BIND9 (Servidor Principal y Servidor Secundario)

Esta guía documenta únicamente los pasos necesarios para habilitar la réplica de zona (Master/Slave) entre el servidor DNS principal y el nuevo servidor secundario clonado.

---

## 1. Parámetros de los Servidores
- **Servidor DNS Principal (Master)**: `172.16.6.170`
- **Servidor DNS Secundario (Slave - Clon)**: `172.16.6.172` *(Ajustar según la IP real asignada al clon)*
- **Dominio**: `navarro.asir2.iespoveda.local`

---

## 2. Ajustes en el Servidor DNS Principal (`172.16.6.170`)

### Paso 2.1: Permitir la transferencia de zona en `named.conf.local`
Edita la configuración de la zona en el servidor principal:
```bash
sudo nano /etc/bind/named.conf.local
```
Añade las directivas `allow-transfer` y `also-notify` apuntando a la IP del servidor secundario:
```named
zone "navarro.asir2.iespoveda.local" {
    type master;
    file "/etc/bind/db.navarro.asir2.iespoveda.local";
    allow-transfer { 172.16.6.172; }; // IP del servidor secundario
    also-notify { 172.16.6.172; };    // Notificación automática de cambios
};
```

### Paso 2.2: Actualizar el archivo de registros de zona
Añade el registro NS del servidor secundario e incrementa el número de serie (`Serial`):
```bash
sudo nano /etc/bind/db.navarro.asir2.iespoveda.local
```
```text
$TTL 86400
@   IN  SOA ns1.navarro.asir2.iespoveda.local. admin.navarro.asir2.iespoveda.local. (
              2026101302 ; Serial (Incrementar para forzar la réplica)
              600        ; Refresh
              120        ; Retry
              604800     ; Expire
              86400 )    ; Minimum TTL

@       IN  NS  ns1.navarro.asir2.iespoveda.local.
@       IN  NS  ns2.navarro.asir2.iespoveda.local.
@       IN  A   172.16.6.170
ns1     IN  A   172.16.6.170   ; Servidor Master
ns2     IN  A   172.16.6.172   ; Servidor Slave
www     IN  A   172.16.6.170
cliente IN  A   172.16.6.171
```

### Paso 2.3: Reiniciar BIND9 en el Principal
```bash
sudo systemctl restart named
```

---

## 3. Configuración en el Servidor DNS Secundario (`172.16.6.172`)

### Paso 3.1: Instalar BIND9 en la máquina clonada
```bash
sudo apt update
sudo apt install bind9 bind9-utils -y
```

### Paso 3.2: Configurar `named.conf.options`
Abre la configuración global:
```bash
sudo nano /etc/bind/named.conf.options
```
Configura la escucha en la dirección IP del clon:
```named
options {
    directory "/var/cache/bind";

    listen-on { 127.0.0.1; 172.16.6.172; };
    allow-query { any; };
    recursion yes;
    allow-recursion { 172.16.6.0/24; localhost; };

    forwarders {
        8.8.8.8;
        1.1.1.1;
    };

    dnssec-validation auto;
};
```

### Paso 3.3: Declarar la zona como Esclava (`named.conf.local`)
Abre el archivo de zonas locales:
```bash
sudo nano /etc/bind/named.conf.local
```
Añade la zona indicando la IP del servidor maestro:
```named
zone "navarro.asir2.iespoveda.local" {
    type slave;
    file "/var/cache/bind/db.navarro.asir2.iespoveda.local";
    masters { 172.16.6.170; };
};
```

### Paso 3.4: Iniciar el servicio BIND9 en el Secundario
```bash
sudo systemctl restart named
sudo systemctl enable named
```

---

## 4. Verificación de la Transferencia de Zona
En la máquina secundaria, ejecuta los siguientes comandos para confirmar que ha recibido los datos del servidor principal:

1. **Revisar los registros de transferencia:**
   ```bash
   sudo journalctl -u named -e
   ```
   *(Debe indicar que la transferencia de zona desde `172.16.6.170` finalizó correctamente).*

2. **Probar una consulta DNS contra el Secundario:**
   ```bash
   dig @172.16.6.172 www.navarro.asir2.iespoveda.local
   ```


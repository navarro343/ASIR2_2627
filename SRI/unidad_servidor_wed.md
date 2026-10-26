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

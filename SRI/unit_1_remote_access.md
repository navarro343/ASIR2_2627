# unit 1-Remote Access Management(2ºASIR)
markdown# 1. Instalación del servidor SSH

Por defecto, Ubuntu suele incluir solo el cliente SSH. Para permitir conexiones entrantes, instala el paquete `openssh-server`:

```bash
**sudo apt update**
**sudo apt install openssh-server -y**
```

---

# 2. Verificar que el servicio esté activo

Comprueba el estado del Demonio SSH (SSHD):

```bash
**sudo systemctl status ssh**
```

Si no está activo, inícialo y habilítalo para que arranque con el sistema:

```bash
**sudo systemctl start ssh**
**sudo systemctl enable ssh**
```

**Verificación:** Intenta conectarte localmente o desde otro equipo ejecutando `ssh usuario@IP_DE_TU_UBUNTU`. Si pide la contraseña y te deja acceder, el servicio básico funciona.

---

# 3. Configuración del archivo sshd_config

El archivo de configuración principal se encuentra en `/etc/ssh/sshd_config`. Realiza una copia de seguridad antes de editarlo:

```bash
**sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak**
**sudo nano /etc/ssh/sshd_config**
```

### Modificaciones recomendadas para mayor seguridad:

* **Cambiar el puerto por defecto (opcional):** Busca `#Port 22` y cámbialo por un puerto no estándar (por ejemplo, `Port 2222`).
* **Deshabilitar el acceso del usuario root directamente:** Busca `PermitRootLogin` y asegúrate de que esté configurado en:
  ```text
  PermitRootLogin no
  ```
* **Limitar los intentos de autenticación:**
  ```text
  MaxAuthTries 3
  ```
* **Restringir el acceso a usuarios específicos (opcional):** Añade al final del archivo los nombres de los usuarios permitidos:
  ```text
  AllowUsers tu_usuario
  ```

---

# 4. Configurar autenticación mediante claves SSH (Sin contraseña)

Es el método más seguro para conectarte a tu servidor.

Desde la máquina cliente (tu ordenador local), genera un par de claves:

```bash
**ssh-keygen -t ed25519 -C "comentario_opcional"**
```

Copia la clave pública al servidor Ubuntu:

```bash
**ssh-copy-id tu_usuario@IP_DEL_SERVIDOR**
```
*(Si cambiaste el puerto por defecto, añade `-p puerto`).*

**Deshabilitar la autenticación por contraseña** (Opcional pero recomendado tras probar las claves): Abre de nuevo `/etc/ssh/sshd_config` en el servidor y ajusta:

```text
PasswordAuthentication no
```

---

# 5. Reiniciar el servicio y ajustar el cortafuegos (UFW)

Para aplicar cualquier cambio realizado en la configuración:

```bash
**sudo systemctl restart ssh**
```

Si tienes el cortafuegos ufw activado, habilita el tráfico en el puerto correspondiente:

```bash
# Si usas el puerto por defecto (22):
**sudo ufw allow ssh**

# Si cambiaste el puerto (ejemplo 2222):
**sudo ufw allow 2222/tcp**

# Aplica los cambios en el firewall
**sudo ufw reload**
```





# Configuración de Red en VirtualBox

Cada opción de la pestaña **"Red"** en VirtualBox define cómo se comunicará tu máquina virtual (por ejemplo, tu servidor Ubuntu) con tu ordenador real (anfitrión), con otras máquinas virtuales y con el mundo exterior (Internet).

---

### Tabla Comparativa Rápida

| Modo de Red | ¿Tiene Internet? | ¿Se ve con el PC Real? | ¿Se ve con otras Máquinas Virtuales? |
| :--- | :---: | :---: | :---: |
| **1. NAT** *(Por defecto)* | 🔒 Sí | ❌ No (Requiere Reenvío) | ❌ No |
| **2. Adaptador puente** | 🟢 Sí | 🟢 Sí | 🟢 Sí (Si están en la misma red) |
| **3. Red interna** | ❌ No | ❌ No | 🟢 Sí |
| **4. Adaptador sólo-anfitrión** | ❌ No | 🟢 Sí | 🟢 Sí (Mismo adaptador) |
| **5. Red NAT** | 🔒 Sí | ❌ No (Requiere Reenvío) | 🟢 Sí |

---

## Explicación Detallada de cada Modo

### 1. NAT (Network Address Translation)
* **¿Qué es?:** Es la opción que viene activada por defecto.
* **¿Cómo funciona?:** La máquina virtual se "esconde" detrás de la conexión de tu ordenador real. El router de tu casa no sabe que la máquina virtual existe.
* **Uso ideal:** Excelente para navegar por internet o descargar paquetes de forma inmediata (`sudo apt update`). Sin embargo, es incómodo para usar SSH porque tu PC real no puede iniciar una conexión directa con la máquina sin configurar mapeos.

### 2. Adaptador puente (Bridged Adapter)
* **¿Qué es?:** Convierte a la máquina virtual en un ordenador independiente dentro de tu red local.
* **¿Cómo funciona?:** Tu router de casa le asignará una dirección IP propia (por ejemplo, `192.168.1.45`), exactamente igual que si conectaras un móvil o un portátil real al Wi-Fi.
* **Uso ideal:** **Es la mejor opción si estás montando un servidor SSH.** Permite que te conectes desde tu PC real (o cualquier otro dispositivo de tu casa) usando directamente la dirección IP asignada.

### 3. Red interna (Internal Network)
* **¿Qué es?:** Una red totalmente aislada, invisible para el mundo exterior.
* **¿Cómo funciona?:** No tiene acceso a internet y tu ordenador real tampoco puede comunicarse con ella. Solo sirve para que las máquinas virtuales creadas dentro del mismo entorno hablen entre sí.
* **Uso ideal:** Pruebas de malware, entornos de hackeo ético o simulación de redes complejas sin riesgos externos.

### 4. Adaptador sólo-anfitrión (Host-Only)
* **¿Qué es?:** Un cable de red directo y exclusivo entre tu ordenador real y la máquina virtual.
* **¿Cómo funciona?:** Tu PC real y la máquina virtual se comunican de forma bilateral impecable, pero la máquina virtual se queda completamente desconectada de Internet.
* **Uso ideal:** Si necesitas conectarte por SSH desde tu ordenador para administrar la máquina, pero quieres máxima seguridad asegurándote de que nada entre ni salga a internet.

### 5. Controlador genérico (Generic Driver)
* **¿Qué es?:** Una opción avanzada y muy poco habitual.
* **¿Cómo funciona?:** Permite usar controladores de red personalizados o herramientas de simulación de redes externas muy específicas (como entornos VDE).

### 6. Red NAT (NAT Network)
* **¿Qué es?:** Una evolución del NAT tradicional pensado para múltiples sistemas.
* **¿Cómo funciona?:** Permite crear un grupo de máquinas virtuales donde todas tienen salida a internet y, al mismo tiempo, pueden comunicarse libremente entre sí.

### 7. Red en la nube (Cloud Network)
* **¿Qué es?:** Una característica experimental.
* **¿Cómo funciona?:** Permite conectar la tarjeta de red interna de la máquina virtual con un servicio de infraestructura en la nube compatible (como Oracle Cloud infrastructure).

### 8. No conectado
* **¿Qué es?:** El equivalente a desenchufar físicamente el cable de red.
* **¿Cómo funciona?:** La tarjeta de red sigue estando presente en el sistema operativo, pero no recibe ningún tipo de señal ni datos.

---

## Configuración para Servidor SSH

Si estás siguiendo una guía para montar un servidor SSH en tu máquina virtual, tienes dos caminos principales según lo que elijas aquí:

1. **Si eliges Adaptador Puente:** En tu Ubuntu ejecutas `ip a` para saber tu dirección IP local y te conectas directamente desde la terminal de tu PC real con:  
   `ssh usuario@IP_DE_TU_UBUNTU`
2. **Si mantienes NAT:** Deberás hacer clic en el botón inferior **"Reenvío de puertos"** (Port Forwarding) y añadir una regla mapeando un puerto libre de tu PC (ej: `2222`) hacia el puerto `22` de tu máquina virtual. Te conectarías usando:  
   `ssh usuario@127.0.0.1 -p 2222`

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

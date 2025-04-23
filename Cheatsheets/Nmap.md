# 🛰️ Cheatsheet Nmap

Este cheatsheet recoge los comandos que más utilizo en Nmap para la fase de reconocimiento.
---

## 🔍 Escaneos básicos

### 🔹 Escaneo completo de puertos TCP (rápido)
```bash
nmap -sS -v -n -p- -oN firstscan --min-rate=5000 <IP> -Pn
```
- **-sS**: SYN Stealth Scan (rápido y menos ruidoso)
- **-n**: no resuelve DNS
- **-p-**: escanea todos los 65535 puertos
- **--min-rate**: aumenta la velocidad del escaneo
- **-Pn**: ignora host discovery (tratamos todos como vivos)

### 🔹 Detección de versiones y servicios
```bash
nmap -sCV -p<puertos> -oN targetscan <IP>
```
- **-sC**: scripts por defecto (equivalente a --script=default)
- **-sV**: versión de los servicios
- **-p**: puertos concretos descubiertos en el escaneo anterior

---

## 🧪 Scripts útiles por protocolo

### 🔸 SMB (puerto 445)
```bash
nmap --script smb-vuln* -p445 <IP>
```
- Detecta vulnerabilidades comunes en SMB, como MS17-010 (EternalBlue).

### 🔸 FTP (puerto 21)
```bash
nmap --script ftp* -p21 <IP>
```
- Enumera banners, usuarios anónimos permitidos, archivos expuestos, etc.

### 🔸 HTTP (puerto 80 o 8080)
```bash
nmap --script http-enum -p80 <IP>
```
- Realiza enumeración básica de rutas, tecnologías y paneles comunes.

---

📘 Archivo creado por **igneosec** como parte de la preparación para OSCP.
📁 Repositorio: [https://github.com/igneosec/oscp-preparation-lab](https://github.com/igneosec/oscp-preparation-lab)

# 🖥️ [Nombre de la Máquina]

> 🔊 Plataforma: [TryHackMe / HackTheBox / Otra]
> 
> 
> 🛡️ Nivel de dificultad: [Fácil / Media / Difícil]
> 
> 💡 Preparación OSCP - **igneosec**
> 

---

## 📅 Información general

- **Nombre de la máquina**:
- **Dirección IP**:
- **Sistema operativo**:
- **Tipo de acceso / Vector de entrada**:

---

## 🔍 Enumeración

### 🔹 Escaneo de puertos

```bash
nmap -sS -v -n -p- -oN firstscan --min-rate=5000 <IP> -Pn
```

> Puertos detectados:
> 
- [puerto]/tcp → [servicio]

---

### 🔹 Detección de versiones y servicios

```bash
nmap -sCV -p[puertos] -oN targetscan <IP>
```

> Resultados clave:
> 
- OS:
- Servicios relevantes:
- Banners, hostnames, workgroup, etc.

---

## 🧠 Enumeración útil

- Posibles vulnerabilidades:
- Usuarios identificados:
- Servicios expuestos o inseguros:
- Rutas sospechosas o ficheros sensibles:

---

## 🎯 Explotación

### 🔥 Confirmación de vulnerabilidad (si aplica)

```bash
<script o herramienta>
```

> Descripción de la vulnerabilidad y CVE (si hay)
> 

---

### 💀 Explotación manual

```bash
<exploit>
```

> Resultado / error / acceso obtenido
> 

---

## 📦 Post-Explotación

### 🔐 Extracción de credenciales / hashes

```bash
<hashdump / smbclient / credenciales>
```

- Usuario: hash / contraseña

### ⚡️ Crackeo (si aplica)

```bash
john --wordlist=<ruta> <hashfile>
```

> Contraseña crackeada: [opcional]
> 

---

## 🏁 Flags encontradas

### ✅ Flag 1

- **Ruta**:
- **Contenido**:

### ✅ Flag 2

- **Ruta**:
- **Contenido**:

(...)

---

## 📚 Lecciones aprendidas

- Herramientas usadas y combinadas
- Obstáculos resueltos durante la sesión
- Buenas prácticas observadas

---

📄 Plantilla OSCP-style creada por **igneosec**

📊 Parte del laboratorio ofensivo: [https://github.com/igneosec](https://github.com/igneosec)
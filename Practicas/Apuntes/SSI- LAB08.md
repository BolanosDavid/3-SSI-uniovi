## Descubrimiento de red y enumeración inicial de hosts

El primer paso de la práctica consistió en identificar el rango de direcciones utilizado en el laboratorio. A partir de una dirección observada dentro del entorno, en nuestro caso `192.168.8.72`, se planteó como hipótesis que la red podía pertenecer al rango `192.168.8.0/24`. Aunque una única IP no permite conocer con certeza la máscara de red, esta suposición resultaba coherente con el escenario proporcionado y fue posteriormente validada mediante un escaneo completo del rango.

Para comprobarlo, se realizó un barrido con `nmap` sobre toda la subred:

```bash
sudo nmap 192.168.8.0/24
```

El escaneo identificó un total de **8 hosts activos** dentro del rango analizado. Los resultados obtenidos fueron los siguientes:

| Dirección IP | Nombre detectado                    |               Puertos abiertos | Observaciones                           |
| ------------ | ----------------------------------- | -----------------------------: | --------------------------------------- |
| 192.168.8.1  | -                                   |                         22/tcp | Host con servicio SSH expuesto          |
| 192.168.8.2  | lab8_dns.lab_08_lab8_net            |                         53/tcp | Servidor DNS del laboratorio            |
| 192.168.8.13 | lab8_obsolete.lab_08_lab8_net       |         22/tcp, 23/tcp, 80/tcp | Host con SSH, Telnet y HTTP             |
| 192.168.8.34 | lab8_eii.lab_08_lab8_net            |                Ninguno visible | Todos los puertos aparecen filtrados    |
| 192.168.8.48 | lab8_epi.lab_08_lab8_net            |                 22/tcp, 80/tcp | Host con SSH y HTTP                     |
| 192.168.8.51 | lab8_ssh.lab_08_lab8_net            |                         22/tcp | Host dedicado a SSH                     |
| 192.168.8.69 | lab8_ssh_vulnerable.lab_08_lab8_net | 21/tcp, 22/tcp, 23/tcp, 80/tcp | Host con múltiples servicios expuestos  |
| 192.168.8.73 | lab8_kali                           |                Ninguno visible | Máquina atacante, con puertos filtrados |

A partir de estos resultados se puede concluir que la hipótesis inicial era correcta y que el laboratorio se encuentra desplegado efectivamente en la subred **`192.168.8.0/24`**.

Además, esta primera enumeración ya permite extraer varias observaciones relevantes. En primer lugar, se identifica claramente un **servidor DNS** en `192.168.8.2`, lo que indica que la resolución de nombres puede formar parte importante de la práctica. En segundo lugar, se observan varios hosts con servicios clásicos de administración remota y publicación web, como **SSH (22/tcp)** y **HTTP (80/tcp)**. También destaca la presencia de **Telnet (23/tcp)** y **FTP (21/tcp)** en algunos equipos, servicios que suelen considerarse inseguros o desaconsejados en entornos modernos y que, por tanto, resultan especialmente interesantes desde el punto de vista de la enumeración y el análisis de superficie de exposición.

Uno de los equipos más llamativos es `192.168.8.69`, ya que presenta simultáneamente los servicios **FTP, SSH, Telnet y HTTP**, lo que sugiere una máquina deliberadamente más expuesta o vulnerable que el resto. Por otro lado, el host `192.168.8.34` responde como activo, pero todos los puertos aparecen como **filtered**, lo que indica la posible existencia de reglas de filtrado o cortafuegos que impiden obtener información precisa mediante un escaneo TCP básico. Algo similar ocurre con la máquina Kali (`192.168.8.73`), que aparece activa pero con los puertos filtrados.

En esta fase, el uso de `nmap` permitió por tanto cumplir dos objetivos fundamentales: **confirmar el rango de red del laboratorio** e **identificar los hosts activos junto con una primera aproximación a los servicios expuestos**. Esta información servirá como base para fases posteriores de enumeración más detallada, orientadas a identificar versiones, configuraciones y posibles debilidades de cada uno de los sistemas detectados.

---

## Procedimiento realizado

En primer lugar, observamos que una de las direcciones del entorno pertenecía al rango `192.168.8.x`, concretamente `192.168.8.72`. A partir de ello, planteamos que el laboratorio probablemente estuviese desplegado sobre la subred `192.168.8.0/24`. Para verificar esta hipótesis realizamos un escaneo de red con `nmap` sobre todo el rango.

El comando utilizado fue:

```bash
sudo nmap 192.168.8.0/24
```

Tras completar el barrido, `nmap` detectó 8 hosts activos. Gracias a este escaneo inicial pudimos identificar qué máquinas estaban encendidas y qué servicios TCP básicos exponía cada una de ellas. Entre ellas destacaban un servidor DNS, varios servidores con SSH y HTTP, y una máquina especialmente expuesta con FTP, SSH, Telnet y HTTP abiertos.

---

## Enumeración adicional mediante scripts NSE por defecto

Una vez identificados los hosts activos y sus puertos abiertos, se realizó una segunda fase de enumeración utilizando los **scripts por defecto de Nmap**, con el objetivo de obtener más información sobre los servicios detectados, como banners, claves públicas SSH, títulos HTTP o versiones de software.

El comando ejecutado fue el siguiente:

```bash
sudo nmap 192.168.8.0/24 --script default
```

Este escaneo mantuvo la detección de hosts observada previamente, pero además permitió obtener información más detallada sobre varios de los servicios expuestos.

### Resultados obtenidos

#### 192.168.8.2 – Servidor DNS

En el servidor DNS se obtuvo información de versión mediante el script correspondiente:

- **53/tcp open domain**
    
- **bind.version: 9.18.30-0ubuntu0.20.04.2-Ubuntu**
    

Esto permitió identificar que el servicio DNS del laboratorio está basado en **BIND 9.18.30 sobre Ubuntu 20.04**, lo cual resulta útil para posteriores tareas de fingerprinting o búsqueda de posibles vulnerabilidades asociadas a la versión.

#### 192.168.8.13 – Host obsolete

En este equipo se confirmó la exposición de varios servicios:

- **22/tcp open ssh**
    
- **23/tcp open telnet**
    
- **80/tcp open http**
    

Además, `nmap` recuperó las **claves públicas del servidor SSH** (DSA, RSA, ECDSA y ED25519), lo que confirma la disponibilidad real del servicio y aporta información de huella digital del host. En el puerto HTTP se identificó como título de la página:

- **Apache2 Ubuntu Default Page: It works**
    

Esto sugiere que el servidor web mantiene la **página por defecto de Apache**, lo cual suele indicar una configuración mínima o poco personalizada. Desde el punto de vista de seguridad, también destaca especialmente la presencia de **Telnet**, ya que se trata de un protocolo obsoleto e inseguro al transmitir la información sin cifrar.

#### 192.168.8.48 – Host epi

En este equipo se detectaron los siguientes servicios:

- **22/tcp open ssh**
    
- **80/tcp open http**
    

Al igual que en otros hosts con SSH, se obtuvieron sus claves públicas. En el servidor web, el título recuperado fue:

- **EPI Gijón – Ingeniería industrial, informática y de telecomunicación**
    

Esto indica que la página alojada no es la página por defecto del servidor, sino un contenido web ya desplegado y personalizado.

#### 192.168.8.51 – Host ssh

Este host mantiene únicamente el puerto:

- **22/tcp open ssh**
    

En este caso, la información adicional obtenida corresponde a las **claves host SSH**, lo que confirma que se trata de un sistema centrado en ofrecer acceso remoto seguro.

#### 192.168.8.69 – Host vulnerable

Este equipo volvió a destacar como el más expuesto de la red, presentando los siguientes servicios:

- **21/tcp open ftp**
    
- **22/tcp open ssh**
    
- **23/tcp open telnet**
    
- **80/tcp open http**
    

El servidor HTTP devolvió el mismo título que el host `192.168.8.48`, concretamente:

- **EPI Gijón – Ingeniería industrial, informática y de telecomunicación**
    

También se recuperaron las claves públicas SSH. Sin embargo, lo más relevante de este análisis apareció en los resultados posteriores del escaneo: `nmap` detectó que las **claves SSH de 192.168.8.69 y 192.168.8.48 son idénticas**, tanto para RSA como para ECDSA y ED25519.

### Detección de hosts duplicados o clonados

En la sección final del escaneo, `nmap` mostró el aviso:

- **ssh-hostkey: Possible duplicate hosts**
    

y señaló que los hosts `192.168.8.48` y `192.168.8.69` comparten exactamente las mismas claves SSH.

Este dato es especialmente interesante, ya que suele indicar una de estas situaciones:

- ambos equipos proceden de una **misma imagen clonada**
    
- se ha copiado la configuración de SSH entre máquinas
    
- o bien son sistemas desplegados a partir de una plantilla sin regenerar las host keys
    

Desde el punto de vista de la enumeración, este hallazgo es importante porque permite **relacionar dos máquinas aparentemente distintas**, sugiriendo que pueden compartir una base común de configuración o de software. Además, reutilizar claves host SSH entre sistemas diferentes es una mala práctica de seguridad, ya que rompe la unicidad esperada de la identidad criptográfica de cada servidor.

### Interpretación de los resultados

Esta segunda fase permitió enriquecer notablemente la información obtenida en el barrido inicial. Ya no solo se conocían los hosts activos y sus puertos abiertos, sino también detalles concretos sobre los servicios:

- versión del servidor DNS
    
- títulos de las páginas web alojadas
    
- huellas criptográficas de varios servidores SSH
    
- indicios de configuración por defecto en Apache
    
- y evidencias de reutilización de configuración entre hosts
    

En conjunto, esta información resulta muy útil para orientar las siguientes fases de enumeración, ya que permite priorizar los objetivos más interesantes. En particular, el host `192.168.8.69` sigue siendo el más llamativo por la cantidad de servicios expuestos, mientras que la coincidencia de claves SSH entre `192.168.8.48` y `192.168.8.69` aporta una pista adicional sobre la relación entre ambos sistemas.

---

Y si quieres dejar una versión más compacta, más estilo “qué hicimos”, podrías poner también esto:

## Procedimiento realizado

Después del escaneo inicial, ejecutamos `nmap` con los scripts NSE por defecto para obtener más detalles sobre los servicios descubiertos:

```bash
sudo nmap 192.168.8.0/24 --script default
```

Gracias a esta fase obtuvimos información adicional como la versión del servidor DNS, los títulos de varias páginas web y las claves públicas de distintos servicios SSH. Entre los resultados más relevantes destacó la identificación de **BIND 9.18.30** en el servidor DNS, la detección de la **página por defecto de Apache** en el host `192.168.8.13` y el hallazgo de que los hosts `192.168.8.48` y `192.168.8.69` comparten las mismas claves SSH, lo que sugiere que podrían haberse desplegado a partir de una misma imagen o configuración base.

---

El siguiente paso lógico en esta práctica suele ser uno de estos:

- **escaneo de versiones** con `nmap -sV`
    
- **detección de sistema operativo** con `-O`
    
- **enumeración DNS** contra `192.168.8.2`
    
- o **escaneos TCP alternativos** para comparar cómo responde `192.168.8.34`
    

Pásame el siguiente comando o resultado y te sigo montando la memoria en orden.
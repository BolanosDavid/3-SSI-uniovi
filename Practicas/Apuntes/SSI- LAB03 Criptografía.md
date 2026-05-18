# Resumen de la Práctica de Criptografía - Laboratorios 3-4

## Contexto General

Esta práctica consiste en un CTF (Capture The Flag) de criptografía para la asignatura de Seguridad de Sistemas Informáticos de la Universidad de Oviedo. Cada estudiante tiene recursos individualizados y debe completar varios pasos relacionados con conceptos criptográficos.

  

---

  

## Paso 1: Cracking de archivo ZIP protegido

### Objetivo

Descifrar un archivo ZIP protegido por contraseña que se descargó mediante una URL personalizada proporcionada por correo electrónico.
### Información proporcionada

- **URL personalizada**: `https://156.35.163.140/api/dd44e5e5` 

- **Credenciales**: `UO302313:pass_ECpPY`

- **Pista**: La contraseña tiene máximo 6 caracteres (letras mayúsculas/minúsculas y números)

### Proceso realizado

1. **Conexión a la VPN de la Universidad**

   - Requisito: Conectarse a GlobalProtect VPN desde el sistema host (no funciona en la MV)

   - Manual: https://torres.epv.uniovi.es/centon/acceso-forticlient.html

  

2. **Descarga del archivo ZIP**

   ```bash

   curl -u UO302313:pass_ECpPY https://156.35.163.140/api/dd44e5e5 --insecure --output UO302313.zip

   ```

  

3. **Cracking de la contraseña**

   - **Herramienta utilizada**: `fcrackzip`

   - **Comando**:

   ```bash

   fcrackzip -b -c 'aA1' -l 1-6 -u archivo.zip

   ```

   - Parámetros:

     - `-b`: Modo brute force

     - `-c 'aA1'`: Charset (a=minúsculas, A=mayúsculas, 1=números)

     - `-l 1-6`: Longitud de 1 a 6 caracteres

     - `-u`: Verificación con unzip

  

4. **Extracción del contenido**

   ```bash

   unzip archivo.zip

   ```

  

### Resultado

Archivo ZIP descifrado exitosamente, obteniendo múltiples archivos:

- Imágenes (image_0.jpg, image_1.jpg, etc.)

- Archivo de instrucciones (step2.txt)

- Archivo cifrado (UO302313_mensaje.enc)

- Vector de inicialización (UO302313_IV)

- Claves criptográficas

  

---

  

## Paso 2: Esteganografía con Steghide

  

### Objetivo

Identificar una imagen específica mediante su hash SHA-384, extraer un mensaje oculto usando steghide, y crackear la contraseña de esteganografía.

### Información proporcionada

- **Hash objetivo**: `a8f4c98dc979e4823bba78e93eb90ba04ed83eea3d080a27556ee244acf4a28e378140070c516bd744bc997e447931f8`

- **URL para diccionario**: `http://156.35.163.140:777`

- **Pista**: La contraseña está entre las palabras de esa página web

### Proceso realizado

1. **Identificación de la imagen correcta**

   ```bash

   # Calcular hash SHA-384 de todas las imágenes

   sha384sum *.jpg *.png

  

   # Buscar la que coincide con el hash objetivo

   sha384sum *.jpg | grep "a8f4c98dc979e4823bba78e93eb90ba04ed83eea3d080a27556ee244acf4a28e378140070c516bd744bc997e447931f8"

   ```

2. **Generación del diccionario con CeWL**

   ```bash

   # Conectado a la VPN, extraer palabras de la página web

   cewl http://156.35.163.140:777 -m 1 -d 4 --with-numbers -w diccionario.txt

   ```

   - Parámetros:

     - `-m 1`: Palabras desde 1 carácter

     - `-d 4`: Profundidad de rastreo

     - `--with-numbers`: Incluir palabras con números

  

3. **Cracking de la contraseña de steghide**

   - **Herramienta utilizada**: `stegseek` (mucho más rápida que otras alternativas)

   ```bash

   stegseek imagen_correcta.jpg diccionario.txt

   ```
  
4. **Resultado del cracking**

   - **Contraseña encontrada**: `Contact`

5. **Extracción del mensaje oculto**

   ```bash

   # Modo interactivo

   steghide extract -sf imagen_correcta.jpg

   # Introducir contraseña: Contact

  

   # O modo directo

   steghide extract -sf imagen_correcta.jpg -p "Contact"

   ```


6. **Lectura del contenido extraído**

   ```bash

   cat step3.txt

   ```

### Resultado

Mensaje extraído exitosamente conteniendo las **instrucciones para los pasos 3 y 4**:

- Clave de descifrado: `ee5dd8c351b1c5d089c7793c50e874b6`

- Archivo a descifrar: `uo302313.enc

- Requisito: Probar diferentes algoritmos de cifrado simétrico

- El mensaje descifrado estará codificado y contendrá una pregunta simple

- Respuesta esperada: un número

  

## Paso 3 y 4: Descifrado simétrico y decodificación

## Objetivo

Descifrar un archivo cifrado simétricamente probando diferentes algoritmos, decodificar el resultado, responder a una pregunta y encontrar instrucciones ocultas en el mensaje.

## Información disponible

- **Archivo cifrado**: `UO302313_mensaje.enc` (80 bytes, binario)
    
- **Clave**: `49d32a54f7d47a048c45a826ddf8843d` (128 bits / 32 caracteres hex)
    
- **IV**: `f7ab03abeca859e33bcf577713d855ea` (del archivo UO302313_IV)
    

## Proceso realizado

1. **Probar diferentes algoritmos de cifrado simétrico**
    
    Lista de algoritmos candidatos según el PDF:
    
    - AES-192-CBC, AES-192-CTR
        
    - AES-256-CBC, AES-256-ECB, AES-256-OFB, AES-256-CTR
        
    - ARIA-256-CBC, ARIA-256-CTR
        
    - Camellia-256-CBC
        
    - ChaCha20
        
    
    **Comando para descifrar con AES-192-CBC** (el correcto):
    
    bash
    
    `openssl enc -d -aes-192-cbc -in UO302313_mensaje.enc -out mensaje_descifrado.txt -K 49d32a54f7d47a048c45a826ddf8843d -iv f7ab03abeca859e33bcf577713d855ea`
    
    - `-d`: Modo descifrado
        
    - `-aes-192-cbc`: Algoritmo (probar hasta encontrar el correcto)
        
    - `-K`: Clave en hexadecimal (mayúscula)
        
    - `-iv`: Vector de inicialización
        
2. **Decodificación del mensaje**
    
    El mensaje descifrado estaba codificado en **Base64**:
    
    bash
    
    `cat mensaje_descifrado.txt # Verificar si es Base64 base64 -d mensaje_descifrado.txt > mensaje_final.txt cat mensaje_final.txt`
    
3. **Responder a la pregunta**
    
    El mensaje contenía una pregunta matemática simple cuya respuesta debía subirse al formulario.
    

## Resultado

✅ Mensaje descifrado y decodificado correctamente  
✅ Respuesta a la pregunta obtenida  
✅ Instrucciones para los siguientes pasos reveladas

---

## Paso 5: Acceso SSH con clave privada

## Objetivo

Acceder mediante SSH a una máquina remota usando la clave privada proporcionada y obtener un archivo de respuesta.

## Información proporcionada

- **IP**: 156.35.163.140
    
- **Puerto**: 2222
    
- **Usuario**: UO302313
    
- **Clave privada**: `clave_privada_UO302313_id_rsa` (del ZIP inicial)
    
- **Archivo objetivo**: `respuesta_UO302313_step5`
    

## Proceso realizado

1. **Configurar permisos de la clave privada**
    
    bash
    
    `chmod 600 clave_privada_UO302313_id_rsa`
    
    _Nota_: SSH requiere que las claves privadas tengan permisos restrictivos (solo lectura/escritura para el propietario)
    
2. **Conexión SSH**
    
    bash
    
    `ssh -i clave_privada_UO302313_id_rsa -p 2222 UO302313@156.35.163.140`
    
    - `-i`: Especifica la clave privada
        
    - `-p 2222`: Puerto no estándar
        
3. **Obtener el archivo de respuesta**
    
    bash
    
    `ls cat respuesta_UO302313_step5`
    
4. **Localizar el hash para el siguiente paso**
    
    bash
    
    `cat local_hash`
    

## Resultado

✅ Acceso SSH exitoso usando criptografía asimétrica (RSA)  
✅ Contenido de `respuesta_UO302313_step5` obtenido  
✅ Hash del usuario local encontrado: `12829da7d7e6146a0506cc410ffeeb0f88be99b3035dbb78ee56347d09d492bf`

---

## Paso 6: Cracking de contraseña de usuario Linux

## Objetivo

Crackear el hash de contraseña de un usuario Linux, acceder a ese usuario, y obtener un archivo token.

## Información proporcionada

- **Usuario objetivo**: `local_UO302313`
    
- **Hash SHA-256**: `12829da7d7e6146a0506cc410ffeeb0f88be99b3035dbb78ee56347d09d492bf` (del archivo `local_hash`)
    
- **Archivo objetivo**: `token`
    

## Proceso realizado

1. **Identificación del tipo de hash**
    
    El hash tiene 64 caracteres hexadecimales → **SHA-256**
    
    Herramientas para identificar hashes:
    
    - CyberChef: [https://gchq.github.io/CyberChef/](https://gchq.github.io/CyberChef/)
        
    - Hashid: `hashid hash.txt`
        
    - Hash Analyzer: [https://www.tunnelsup.com/hash-analyzer](https://www.tunnelsup.com/hash-analyzer)
        
2. **Cracking del hash**
    
    **Opción 1: CrackStation (online, recomendado para empezar)**
    
    text
    
    `URL: https://crackstation.net/ Pegar el hash y resolver el captcha`
    
    **Opción 2: Hashcat (offline, para hashes complejos)**
    
    bash
    
    `# Ataque con diccionario hashcat -m 1400 -a 0 local_hash /usr/share/wordlists/rockyou.txt # Ataque de fuerza bruta hashcat -m 1400 -a 3 local_hash`
    
    - `-m 1400`: Modo SHA-256
        
    - `-a 0`: Ataque con diccionario
        
    - `-a 3`: Ataque de fuerza bruta
        
    
    **Opción 3: John the Ripper**
    
    bash
    
    `john local_hash --format=raw-sha256 --wordlist=/usr/share/wordlists/rockyou.txt`
    
    **Resultado del cracking**:
    
    - **Contraseña encontrada**: `ATHFCHARACTERS`
        
3. **Acceso al usuario local**
    
    bash
    
    `su local_UO302313 # Introducir contraseña: ATHFCHARACTERS`
    
4. **Navegación y obtención del token**
    
    bash
    
    `# Cambiar al directorio home correcto cd ~ # o simplemente cd # Listar archivos ls # Leer el token cat token`
    

## Resultado

✅ Hash crackeado exitosamente con CrackStation  
✅ Contraseña obtenida: `ATHFCHARACTERS`  
✅ Acceso al usuario `local_UO302313` logrado  
✅ Contenido del archivo `token` obtenido

## Entrega requerida para Paso 6

- Contraseña del usuario `local_UO302313`: `ATHFCHARACTERS`
    
- Contenido del archivo `token`
    

---

## Paso 7: Generación de certificado SSL con CSR

## Objetivo

Generar un par de claves RSA, crear un CSR (Certificate Signing Request), y obtener un certificado firmado por la CA de la universidad.

## Información proporcionada

- **URL de la CA**: [http://156.35.163.140:5000](http://156.35.163.140:5000/)
    
- **Requisito**: Usar OpenSSL 3.0 o superior
    

## Proceso realizado

1. **Verificar versión de OpenSSL**
    
    bash
    
    `openssl version # Debe ser 3.0 o superior`
    
2. **Generar clave privada y CSR simultáneamente**
    
    bash
    
    `openssl req -newkey rsa:2048 -keyout mi_clave_privada.key -out mi_csr.csr -nodes`
    
    - `-newkey rsa:2048`: Genera clave RSA de 2048 bits
        
    - `-keyout`: Nombre del archivo de clave privada
        
    - `-out`: Nombre del archivo CSR
        
    - `-nodes`: No cifrar la clave privada con contraseña (opcional)
        
3. **Datos solicitados durante la generación**
    
    text
    
    `Country Name (2 letter code): ES State or Province Name: Asturias Locality Name: Oviedo Organization Name: Universidad de Oviedo Organizational Unit Name: (opcional) Common Name: UO302313 Email Address: UO302313@uniovi.es Challenge password: (dejar en blanco) Optional company name: (dejar en blanco)`
    
4. **Subir el CSR a la CA**
    
    - Abrir navegador (Chrome, Firefox, Edge)
        
    - Ir a: `http://156.35.163.140:5000`
        
    - Subir el archivo `mi_csr.csr`
        
    - Descargar el certificado firmado (formato `.pem`)
        
    - Guardar como `certificado_firmado.pem`
        

## Resultado

✅ Par de claves RSA generado (2048 bits)  
✅ CSR creado y enviado a la CA  
✅ Certificado firmado obtenido en formato PEM

## Archivos obtenidos

- `mi_clave_privada.key` - Tu clave privada RSA (guardar segura)
    
- `mi_csr.csr` - Certificate Signing Request (ya no es necesario)
    
- `certificado_firmado.pem` - Tu certificado firmado por la CA
    

---

## Paso 8: Cifrado y firma del archivo token

## Objetivo

Cifrar el archivo token con la clave pública de los profesores (criptografía asimétrica) y firmarlo digitalmente con tu certificado.

## Información proporcionada

- **Clave pública profesores**: `public_key_profesores.pem` (del ZIP inicial)
    
- **Archivo a cifrar**: `token` (del Paso 6)
    
- **Tu clave privada**: `mi_clave_privada.key` (del Paso 7)
    

## Proceso realizado

1. **Cifrar el archivo token con la clave pública de los profesores**
    
    **Opción 1: Cifrado directo con pkeyutl** (para archivos pequeños)
    
    bash
    
    `openssl pkeyutl -encrypt -pubin -inkey public_key_profesores.pem -in token -out token_cifrado.enc`
    
    - `-encrypt`: Modo cifrado
        
    - `-pubin`: Indica que la entrada es una clave pública
        
    - `-inkey`: Clave pública de los profesores
        
    - `-in`: Archivo a cifrar
        
    - `-out`: Archivo cifrado resultante
        
    
    **Opción 2: Usar CMS** (alternativa más moderna)
    
    bash
    
    `openssl cms -encrypt -binary -aes-256-cbc -in token -out token_cifrado.enc -outform DER -recip public_key_profesores.pem`
    
    **Opción 3: Método híbrido AES + RSA** (para archivos grandes)
    
    bash
    
    `# Generar clave simétrica aleatoria openssl rand -out clave_temp.bin 32 # Cifrar el token con AES openssl enc -aes-256-cbc -salt -in token -out token_aes.enc -pass file:clave_temp.bin -pbkdf2 # Cifrar la clave simétrica con RSA openssl pkeyutl -encrypt -pubin -inkey public_key_profesores.pem -in clave_temp.bin -out clave_temp.bin.enc # Combinar ambos (si es necesario) cat clave_temp.bin.enc token_aes.enc > token_cifrado.enc`
    
2. **Firmar el archivo cifrado con tu clave privada**
    
    bash
    
    `openssl dgst -sha256 -sign mi_clave_privada.key -out token_cifrado.sig token_cifrado.enc`
    
    - `dgst`: Comando de digest/hash
        
    - `-sha256`: Algoritmo de hash
        
    - `-sign`: Firmar con clave privada
        
    - `-out`: Archivo de firma resultante
        
3. **Verificar los archivos generados**
    
    bash
    
    `ls -lh token_cifrado.enc token_cifrado.sig certificado_firmado.pem`
    

## Resultado

✅ Archivo `token` cifrado con la clave pública de los profesores  
✅ Archivo cifrado firmado digitalmente con tu clave privada  
✅ Tres archivos listos para la entrega

## Entrega requerida para Pasos 7 y 8

1. **token_cifrado.enc** - Token cifrado con clave pública de profesores
    
2. **token_cifrado.sig** - Firma digital del archivo cifrado
    
3. **certificado_firmado.pem** - Tu certificado firmado por la CA
    

---

## Conceptos criptográficos aplicados

## Criptografía asimétrica (RSA)

- **Pares de claves**: Clave pública (cifrar/verificar) y privada (descifrar/firmar)
    
- **CSR**: Solicitud de firma de certificado para PKI
    
- **Certificados digitales**: Vinculan identidad con clave pública
    
- **Cifrado RSA**: Usado para archivos pequeños o claves simétricas
    
- **Firma digital**: Garantiza autenticidad e integridad
    

## Infraestructura de clave pública (PKI)

- **CA (Certificate Authority)**: Entidad que firma certificados
    
- **Cadena de confianza**: Certificados verificados por CA confiable
    
- **Formato PEM**: Codificación Base64 de certificados/claves
    

## Cracking de contraseñas Linux

- **Shadow file**: Almacena hashes de contraseñas en Linux
    
- **Rainbow tables**: Tablas pre-computadas de hashes
    
- **SHA-256**: Función hash unidireccional
    
- **Dictionary attack**: Más eficiente que fuerza bruta
    

## SSH con autenticación por clave pública

- **Autenticación sin contraseña**: Más segura que contraseñas
    
- **Pares de claves SSH**: RSA, DSA, ECDSA, Ed25519
    
- **Permisos restrictivos**: Claves deben tener chmod 600
    

---

## Comandos de referencia rápida

Cracking ZIP fcrackzip -b -c 'aA1' -l 1-6 -u archivo.zip 
 Generar diccionario web cewl http://URL -m 1 -d 4 --with-numbers -w diccionario.txt
Cracking steghide stegseek imagen.jpg diccionario.txt 
Descifrado simétrico openssl enc -d -aes-192-cbc -in archivo.enc -out archivo.txt -K clave_hex -iv iv_hex 
SSH con clave privada ssh -i clave_privada.key -p 2222 usuario@ip 
Cracking SHA-256 hashcat -m 1400 -a 0 hash.txt wordlist.txt  
Generar CSR openssl req -newkey rsa:2048 -keyout clave.key -out csr.csr -nodes 
Cifrar con clave pública openssl pkeyutl -encrypt -pubin -inkey publica.pem -in archivo -out cifrado.enc 
Firmar archivo openssl dgst -sha256 -sign privada.key -out firma.sig archivo`

---


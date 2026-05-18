docker run
docker stop
docker exec
docker build -> 
docker bash -> linea de comandos del contenedor


Para encontrar la primera contraseña miramos las cabeceras las peticiones HTTP y nos damos cuenta que la contraseña está en el debug token
SSI_P1_2ddcc781905bfe6a
Para encontrar la segunda contraseña buscamos 
la palabra ROBOT y nos damos cuenta que hay 2 carpetas disallowed. Ponemos la de backup y encontramos la segunda.
SSI_P2_e8611d13d2aa4adf
Para la tercera contraseña necesitamos hacer una conexion FTP al servidor y analizamos los metadatos de la imagen y la encontramos en el Comentario

SSI_P3_b659f42158bea1be

La ultima contraseña:
MIIEoQ

bda511

2ddcc781905bfe6ae8611d13d2aa4adfb659f42158bea1beMIIEoQbda511
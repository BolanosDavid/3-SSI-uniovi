## 1. Objetivo de la práctica

El objetivo de esta práctica ha sido identificar y analizar vulnerabilidades de seguridad en una aplicación, utilizando tanto revisión manual como herramientas automáticas de análisis estático y dinámico. Además, se ha comprobado el comportamiento de dependencias vulnerables dentro de un pipeline básico de ejecución y validación.

## 2. Material de partida

Como punto de partida se descargó un archivo `.zip` desde el campus virtual. En dicho material se incluía una aplicación desarrollada en C#, utilizada en la práctica para trabajar la seguridad de aplicaciones.

La aplicación analizada solicita unas credenciales al usuario, realiza una operación de hash sobre la contraseña, interactúa con una base de datos para simular el registro de un usuario y, posteriormente, pide una ruta de fichero para mostrar por pantalla el contenido del archivo indicado. A partir de este comportamiento se realizó el análisis de seguridad.

## 3. Detección manual de vulnerabilidades

En una primera fase se llevó a cabo una revisión manual del código de la aplicación para localizar errores y huecos de seguridad. Finalmente, se recogieron las siguientes vulnerabilidades:

### 3.1. URL de conexión a la base de datos hardcodeada

La cadena de conexión de acceso a datos está embebida directamente en el código fuente. Esto supone un riesgo, ya que cualquier persona con acceso al código puede conocer los datos de conexión y tratar de acceder a la base de datos.

Como medida correctiva, esta información debería almacenarse fuera del código, en un entorno externo y controlado, y posteriormente inyectarse en la aplicación durante la ejecución.

### 3.2. Inyección SQL en la consulta

La consulta SQL se construye directamente a partir de datos introducidos por el usuario. Esto permite que un atacante pueda alterar la consulta original e inyectar instrucciones SQL maliciosas.

La solución consiste en parametrizar la consulta, evitando concatenar directamente los valores recibidos.

### 3.3. Función de hashing vulnerable

La aplicación emplea una función de hashing considerada insegura. El uso de algoritmos con vulnerabilidades conocidas reduce notablemente la protección de la información sensible.

La solución propuesta es utilizar un algoritmo de hashing moderno y robusto, del que no se conozcan vulnerabilidades prácticas relevantes para este contexto.

### 3.4. Excepciones silenciadas

Durante la ejecución, algunas excepciones se capturan pero no se gestionan adecuadamente, quedando ocultas. Esto dificulta la detección de errores reales, complica el mantenimiento y puede encubrir situaciones de ataque.

La solución es mantener un entorno controlado en el que al usuario solo se le muestren los errores previstos, mientras que el error real quede registrado en un sistema de logs.

### 3.5. Variable no utilizada

Se detectó la presencia de una variable que no está siendo utilizada. Aunque pueda parecer un problema menor, mantener elementos innecesarios en el código puede facilitar usos incorrectos futuros o dar lugar a errores de diseño.

La solución es eliminar esa variable si no resulta necesaria o, en caso de estar pensada para una funcionalidad concreta, utilizarla correctamente.

### 3.6. Path no controlado al leer ficheros

La aplicación permite introducir una ruta de fichero sin aplicar un control adecuado sobre ella. Esto puede facilitar accesos a archivos sensibles o no autorizados.

La solución consiste en validar y restringir el path, permitiendo únicamente rutas seguras y controladas.

## 4. Análisis SAST

Tras la revisión manual, se utilizó una herramienta de análisis estático de seguridad (SAST) con el fin de localizar posibles vulnerabilidades no detectadas inicialmente.

La herramienta seleccionada fue **SonarAnalyzer.CSharp**. En caso de no estar instalada previamente en el proyecto, puede añadirse mediante el siguiente comando:

```bash
dotnet add package SonarAnalyzer.CSharp
```

El objetivo de este análisis era complementar la revisión manual y comprobar si existían problemas adicionales detectables de forma automática a nivel estático.

## 5. Simulación de pipeline y comprobación de dependencias vulnerables

A continuación, se simuló un entorno de ejecución con sus correspondientes pipelines de validación. En este caso, se trabajó con un pipeline básico orientado a comprobar que no existieran errores de seguridad relevantes.

Para verificar su funcionamiento, se instaló una librería con vulnerabilidades conocidas: **Newtonsoft.Json 9.0.1**. El resultado fue especialmente interesante, ya que en este caso el análisis con Sonar no reportó el problema, pero el pipeline sí logró detectarlo.

Esto demuestra que los controles en pipeline pueden descubrir debilidades que no siempre aparecen en un análisis estático convencional.

La medida correctiva en este caso consiste en actualizar la dependencia a una versión más moderna y segura, o bien sustituir la librería por otra alternativa que no presente vulnerabilidades conocidas.

Como comprobación adicional, también se puede utilizar el siguiente comando para listar paquetes vulnerables dentro del proyecto:

```bash
dotnet list package --vulnerable
```

## 6. Análisis DAST

Posteriormente, se comentó que la vulnerabilidad relacionada con el control del path no estaba siendo detectada por el SAST. Esto se debe a que se trata de un problema que, en muchos casos, se aprecia mejor en tiempo de ejecución que mediante análisis estático.

Por ese motivo, se recurrió a un análisis dinámico de seguridad (DAST). Dado que en este escenario no se estaba utilizando base de datos ni otros servicios adicionales, el análisis dinámico se centró en el servidor Apache expuesto.

Para ello se empleó **OWASP ZAP** mediante Docker. Primero se descargó la imagen estable:

```bash
docker pull zaproxy/zap-stable
```

Una vez descargada, se lanzó el análisis baseline sobre la URL expuesta en Apache, generando además un informe en HTML:

```bash
docker run --network host -v $(pwd):/zap/wrk/:rw -t zaproxy/zap-stable zap-baseline.py -t http://localhost:1080 -r informeZAP.html
```

En este caso, `http://localhost:1080` correspondía a la dirección expuesta por el servidor Apache.

Tras revisar el informe generado, se comprobó que la instalación por defecto de Apache presentaba **dos vulnerabilidades de severidad media** detectadas durante el análisis.

## 7. Conclusiones

La práctica ha permitido comprobar que la seguridad de una aplicación no debe evaluarse únicamente mediante inspección manual, sino que es necesario complementarla con herramientas automáticas y con controles en diferentes fases del ciclo de desarrollo y despliegue.

Por un lado, la revisión manual permitió detectar varias debilidades importantes, como la presencia de credenciales hardcodeadas, una consulta vulnerable a inyección SQL, el uso de una función de hashing insegura, el silenciamiento de excepciones, la existencia de una variable no utilizada y la falta de control sobre las rutas de acceso a ficheros.

Por otro lado, el uso de SAST ayudó a reforzar el análisis, mientras que la simulación de pipelines permitió detectar dependencias vulnerables que no habían sido señaladas por otras herramientas. Finalmente, el uso de DAST con OWASP ZAP puso de manifiesto que ciertos problemas solo se observan adecuadamente en ejecución, además de mostrar vulnerabilidades presentes incluso en configuraciones por defecto del entorno, como ocurrió con Apache.

En conjunto, la práctica pone de relieve la importancia de combinar revisión manual, análisis estático, validación de dependencias, pipelines de seguridad y análisis dinámico para conseguir una evaluación más completa de la seguridad de una aplicación.
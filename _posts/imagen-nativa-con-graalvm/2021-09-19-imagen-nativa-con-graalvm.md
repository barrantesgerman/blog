---
layout: post
title: "Imagen Nativa con GraalVM"
date: 2021-09-19 15:00:20 -0600
categories: java
---

Tradicionalmente el diseño de Software se ha realizado con arquitecturas monoliticas, pero con el surgimiento de la nube se ha visto que este tipo de arquitectura no escala de manera efectiva. Por lo que han surgido nuevas arquitecturas para solventar estos problemas modernos, como lo son la arquitectura de microservicios y la arquitectura sin servidor.

Para estas nuevas arquitecturas es indispensable el desarrollo de aplicaciones que consuman poco espacio en disco, con tiempos de inicio muy reducidos, con un buen rendimiento y que consuman poca memoria. [GraalVM][ref1] es un nuevo proyecto de software que nos permite alcanzar estos objetivos.

## ¿Qué es GraalVM?

[GraalVM][ref1] es una nueva máquina virtual de Oracle Labs, de la cual se  pueden destacar cuatro características fundamentales:

* **Es un JDK completo**: que ha pasado todos los test de certificación ([TCK][ref2]) de Oracle, por lo que puede remplazar cualquier instalación de JDK u OpenJDK de manera transparente.
* **Cuenta con un compilador [JIT][ref3] mejorado**: que ha mostrado un mejor desempeño que el propio compilador JIT incluido en el JDK de Oracle.
* **Es una Máquina Virtual Políglota**: que nos permite no solo trabajar con código Java y otros basados en JVM (Groovy, Scala, Kotlin), sino también Javascript, Ruby, Python, R entre otros.
* **Puede crear Imágenes Nativas**: mediante un compilador [AOT][ref4] para producir código nativo, que no requiere la JVM para ejecutarse. *El cual va ser el tema central de está publicación*.

Para comprender mejor los beneficios que nos provee la creación de estas imágenes nativas vamos a crear dos imagenes de [Docker](https://www.docker.com/) con un microservicio muy simple, la primera versión se ejecutará sobre  JRE 11 de Java y la segunda versión será con [imagen nativa][ref5], para posteriormente compararlas.

El código fuente del microservicio se puede descargar desde [Github](https://github.com/barrantesgerman/graalvm-sample-rest-service).

## Instalación

Lo primero a realizar, es instalar GraalVM, al día de hoy la última versión es la **21.2.0**, en el repositorio de [Github](https://github.com/graalvm/graalvm-ce-builds/releases/tag/vm-21.2.0) del proyecto se pueden bajar las diferentes versiones para [Windows](https://github.com/graalvm/graalvm-ce-builds/releases/download/vm-21.2.0/graalvm-ce-java11-windows-amd64-21.2.0.zip), [Linux](https://github.com/graalvm/graalvm-ce-builds/releases/download/vm-21.2.0/graalvm-ce-java11-linux-amd64-21.2.0.tar.gz) y [MacOS](https://github.com/graalvm/graalvm-ce-builds/releases/download/vm-21.2.0/graalvm-ce-java11-darwin-amd64-21.2.0.tar.gz). En el caso de Linux/MacOS se puede utilizar [SDKMAN](https://sdkman.io/) para facilitar está tarea, ejecutando:

```shell
$ sdk install java 21.2.0.r11-grl
```

El utilitario de [native-image][ref5] requerido para crear la imágenes nativas, no viene como parte de la instalación de GraalVM, pero se puede instalar con la herramienta **GraalVM Updater**, ejecutando el comando:

```shell
$ gu install native-image
```

> Para el caso de **Linux** se requiere adicionalmente instalar `glibc-devel` y `zlib-devel`.
>
> Para el caso de **Windows** se requiere la instalación Microsoft Visual C++ (MSVC) que viene con Visual Studio 2017 15.5.5 o superior y la configuración de `x64 Native Tools Command Prompt`.

## Creación del Microservicio

Para el desarrollo del microservicio seleccioné [Gradle](https://gradle.org/) como herramienta de construcción del proyecto y [Spark](https://sparkjava.com/) para el desarrollo del servicio web, por lo que se deben agregar las siguientes dependencias al proyecto:

```gradle
dependencies {
    implementation 'com.sparkjava:spark-core:2.9.3'
    implementation 'org.slf4j:slf4j-simple:1.7.25'
}
```

Se agrega la clase `src/main/java/com/example/App.java` que contendrá la lógica del servicio web. La cual responderá con el mensaje `Hello World` cuando se haga una petición GET a `http://localhost:4567/hello`.

```java
package com.example;
import static spark.Spark.*;
public class App {
    public static void main(String[] args) {
        get("/hello", (req, res) -> "Hello World");
    }
}
```

Con esto ya está listo el microservicio, para compilar el proyecto como un único JAR con todas sus dependencias (un FatJar) agregamos el plugin `id 'com.github.johnrengelman.shadow' version '7.0.0'` en la configuración del proyecto y realizamos la compilación del mismo.

```shell
$ ./gradlew build
Starting a Gradle Daemon, 1 incompatible Daemon could not be reused, use --status for details

BUILD SUCCESSFUL in 14s
10 actionable tasks: 9 executed, 1 up-to-date
```

Ya con esto tenemos un JAR con todas sus dependencias el cual podemos ejecutar con:

```shell
$ java -jar build/libs/graalvm-sample-rest-service-1.0.0-all.jar
[Thread-0] INFO org.eclipse.jetty.util.log - Logging initialized @154ms to org.eclipse.jetty.util.log.Slf4jLog
[Thread-0] INFO spark.embeddedserver.jetty.EmbeddedJettyServer - == Spark has ignited ...
[Thread-0] INFO spark.embeddedserver.jetty.EmbeddedJettyServer - >> Listening on 0.0.0.0:4567
[Thread-0] INFO org.eclipse.jetty.server.Server - jetty-9.4.31.v20200723; built: 2020-07-23T17:57:36.812Z; git: 450ba27947e13e66baa8cd1ce7e85a4461cacc1d; jvm 11.0.12+6-jvmci-21.2-b08
[Thread-0] INFO org.eclipse.jetty.server.session - DefaultSessionIdManager workerName=node0
[Thread-0] INFO org.eclipse.jetty.server.session - No SessionScavenger set, using defaults
[Thread-0] INFO org.eclipse.jetty.server.session - node0 Scavenging every 600000ms
[Thread-0] INFO org.eclipse.jetty.server.AbstractConnector - Started ServerConnector@57285265{HTTP/1.1, (http/1.1)}{0.0.0.0:4567}
[Thread-0] INFO org.eclipse.jetty.server.Server - Started @265ms
```

Ahora con el comando `native-image` procedemos a crear la imagen nativa en nuestro equipo:

```shell
$ native-image -jar build/libs/graalvm-sample-rest-service-1.0.0-all.jar
[graalvm-sample-rest-service-1.0.0-all:25197]    classlist:   1,323.67 ms,  0.96 GB
[graalvm-sample-rest-service-1.0.0-all:25197]        (cap):     633.48 ms,  0.96 GB
[graalvm-sample-rest-service-1.0.0-all:25197]        setup:   2,100.08 ms,  0.96 GB
[graalvm-sample-rest-service-1.0.0-all:25197]     analysis:  10,708.20 ms,  1.77 GB
Fatal error:com.oracle.graal.pointsto.util.AnalysisError$ParsingError: Error encountered while parsing org.eclipse.jetty.util.TypeUtil.<clinit>() 
Parsing context: <no parsing context available>
...
```

El error que obtenemos es debido a la forma de inicialización de las clases en la generación de imágenes nativas de GraalVM, Christian Wimmer ha escrito un par de artículos al respecto para comprender mejor este comportamiento ([Parte 1][ref6] y [Parte 2][ref7]), pero en resumen se requiere indicarle a GraalVM que inicialice en tiempo de compilación los bloques estáticos de código, que en nuestro caso se encuentran en las dependencias de [Spark](https://sparkjava.com/), por lo que debemos ejecutar `native-image` de la siguiente forma:

```shell
$ native-image \
  -H:+ReportUnsupportedElementsAtRuntime \
  -H:TraceClassInitialization=true \
  --enable-http \
  --static \
  --no-fallback \
  --initialize-at-build-time=org.eclipse.jetty,org.slf4j,javax.servlet,org.sparkjava \
  -jar build/libs/graalvm-sample-rest-service-1.0.0-all.jar
[graalvm-sample-rest-service-1.0.0-all:27341]    classlist:   1,489.61 ms,  0.96 GB
[graalvm-sample-rest-service-1.0.0-all:27341]        (cap):     702.42 ms,  0.96 GB
[graalvm-sample-rest-service-1.0.0-all:27341]        setup:   2,253.13 ms,  0.96 GB
[ForkJoinPool-2-worker-1] INFO org.eclipse.jetty.util.log - Logging initialized @11521ms to org.eclipse.jetty.util.log.Slf4jLog
[graalvm-sample-rest-service-1.0.0-all:27341]     (clinit):     394.06 ms,  3.80 GB
[graalvm-sample-rest-service-1.0.0-all:27341]   (typeflow):  14,132.81 ms,  3.80 GB
[graalvm-sample-rest-service-1.0.0-all:27341]    (objects):  11,593.03 ms,  3.80 GB
[graalvm-sample-rest-service-1.0.0-all:27341]   (features):     949.99 ms,  3.80 GB
[graalvm-sample-rest-service-1.0.0-all:27341]     analysis:  27,852.26 ms,  3.80 GB
[graalvm-sample-rest-service-1.0.0-all:27341]     universe:   1,667.32 ms,  3.80 GB
[graalvm-sample-rest-service-1.0.0-all:27341]      (parse):   2,079.10 ms,  3.82 GB
[graalvm-sample-rest-service-1.0.0-all:27341]     (inline):   2,737.85 ms,  3.83 GB
[graalvm-sample-rest-service-1.0.0-all:27341]    (compile):  28,728.56 ms,  4.30 GB
[graalvm-sample-rest-service-1.0.0-all:27341]      compile:  35,150.84 ms,  4.30 GB
[graalvm-sample-rest-service-1.0.0-all:27341]        image:   3,858.71 ms,  4.31 GB
[graalvm-sample-rest-service-1.0.0-all:27341]        write:   1,573.59 ms,  4.31 GB
[graalvm-sample-rest-service-1.0.0-all:27341]      [total]:  74,079.75 ms,  4.31 GB
```

En esta ocasión logramos realizar la compilación de nuestra aplicación en nativo y procedemos a la ejecución de la misma:

```shell
$ ./graalvm-sample-rest-service-1.0.0-all
[Thread-0] ERROR spark.Spark - ignite failed
java.lang.IllegalStateException: java.lang.NoSuchMethodException: no such method: org.eclipse.jetty.server.ForwardedRequestCustomizer$Forwarded.handleCipherSuite(HttpField)void/invokeVirtual
...
```

El error que obtenemos es debido a que GraalVM al generar la imagen nativa, realiza un análisis estático del código. Sin embargo, este análisis tiene algunas limitaciones a la fecha y es incapaz de predecir todos los usos de la interfaz nativa de Java ([JNI](https://docs.oracle.com/en/java/javase/11/docs/specs/jni/intro.html)), el uso de [Reflection](https://www.oracle.com/technical-resources/articles/java/javareflection.html), los proxy dinámicos (`java.lang.reflect.Proxy`) o los recursos en el class path (`Class.getResource`).

Los usos no detectados de estas características dinámicas deben proporcionarse a la herramienta de `native-image` en forma de archivos de configuración en formato JSON, preferiblemente dentro de la carpeta `META-INF/native-image` en los recursos del proyecto.

Para estos casos GraalVM proporciona un [Agente][ref8] que nos facilita la tarea de crear estos archivos, ejecutando el siguiente comando, el agente realizará un rastreo de las características dinámicas usadas:

```shell
java \
  -agentlib:native-image-agent=config-output-dir=src/main/resources/META-INF/native-image \
  -jar build/libs/graalvm-sample-rest-service-1.0.0-all.jar
```

Para que el agente sea capaz de detectar todos los usos de características dinámicas, se deben ejecutar todos los flujos de la aplicación, en este caso realizando la petición GET a `http://localhost:4567/hello`.

Procedemos a realizar la compilación nuevamente:

```shell
$ ./gradlew build
...
$ native-image \
  -H:+ReportUnsupportedElementsAtRuntime \
  -H:TraceClassInitialization=true \
  --enable-http \
  --static \
  --no-fallback \
  --initialize-at-build-time=org.eclipse.jetty,org.slf4j,javax.servlet,org.sparkjava \
  -jar build/libs/graalvm-sample-rest-service-1.0.0-all.jar
[graalvm-sample-rest-service-1.0.0-all:25276]    classlist:   1,414.79 ms,  0.96 GB
[graalvm-sample-rest-service-1.0.0-all:25276]        (cap):     569.28 ms,  0.96 GB
[graalvm-sample-rest-service-1.0.0-all:25276]        setup:   2,106.27 ms,  0.96 GB
[ForkJoinPool-2-worker-7] INFO org.eclipse.jetty.util.log - Logging initialized @11914ms to org.eclipse.jetty.util.log.Slf4jLog
[graalvm-sample-rest-service-1.0.0-all:25276]     (clinit):     369.62 ms,  3.77 GB
[graalvm-sample-rest-service-1.0.0-all:25276]   (typeflow):  11,851.15 ms,  3.77 GB
[graalvm-sample-rest-service-1.0.0-all:25276]    (objects):  14,694.05 ms,  3.77 GB
[graalvm-sample-rest-service-1.0.0-all:25276]   (features):     937.61 ms,  3.77 GB
[graalvm-sample-rest-service-1.0.0-all:25276]     analysis:  28,633.32 ms,  3.77 GB
[graalvm-sample-rest-service-1.0.0-all:25276]     universe:   1,958.40 ms,  3.77 GB
[graalvm-sample-rest-service-1.0.0-all:25276]      (parse):   1,934.04 ms,  3.79 GB
[graalvm-sample-rest-service-1.0.0-all:25276]     (inline):   2,966.97 ms,  3.83 GB
[graalvm-sample-rest-service-1.0.0-all:25276]    (compile):  27,998.57 ms,  4.34 GB
[graalvm-sample-rest-service-1.0.0-all:25276]      compile:  34,444.06 ms,  4.34 GB
[graalvm-sample-rest-service-1.0.0-all:25276]        image:   3,869.79 ms,  4.32 GB
[graalvm-sample-rest-service-1.0.0-all:25276]        write:     736.48 ms,  4.32 GB
[graalvm-sample-rest-service-1.0.0-all:25276]      [total]:  73,406.30 ms,  4.32 GB
```

Procedemos a la ejecución de la imagen nativa:

```shell
./graalvm-sample-rest-service-1.0.0-all
[Thread-0] INFO spark.embeddedserver.jetty.EmbeddedJettyServer - == Spark has ignited ...
[Thread-0] INFO spark.embeddedserver.jetty.EmbeddedJettyServer - >> Listening on 0.0.0.0:4567
[Thread-0] INFO org.eclipse.jetty.server.Server - jetty-9.4.31.v20200723; built: 2020-07-23T17:57:36.812Z; git: 450ba27947e13e66baa8cd1ce7e85a4461cacc1d; jvm 11.0.12
[Thread-0] INFO org.eclipse.jetty.server.session - DefaultSessionIdManager workerName=node0
[Thread-0] INFO org.eclipse.jetty.server.session - No SessionScavenger set, using defaults
[Thread-0] INFO org.eclipse.jetty.server.session - node0 Scavenging every 660000ms
[Thread-0] INFO org.eclipse.jetty.server.AbstractConnector - Started ServerConnector@7c4b47c2{HTTP/1.1, (http/1.1)}{0.0.0.0:4567}
[Thread-0] INFO org.eclipse.jetty.server.Server - Started @6ms
```

Realizamos la petición GET con [cURL](https://curl.se/) y observamos que en esta ocasión obtenemos una respuesta satisfactoria:

```shell
$ curl -X GET -w '\n' http://localhost:4567/hello
Hello World
```

## Crear imágenes de Docker

Para la creación de la imagen con JRE 11 creamos el archivo [Dockerfile.jre](https://github.com/barrantesgerman/graalvm-sample-rest-service/blob/main/Dockerfile.jre) y para la imagen nativa creamos el archivo [Dockerfile.native](https://github.com/barrantesgerman/graalvm-sample-rest-service/blob/main/Dockerfile.native), ambas con [multistage-build](https://docs.docker.com/develop/develop-images/multistage-build/) para garantizar el menor tamaño en la imagen final, creamos las imágenes ejecutando:

```shell
$ docker build -t graalvm-sample-rest-service-jre -f Dockerfile.jre .
$ docker build -t graalvm-sample-rest-service-native -f Dockerfile.native .
```

Ejecutamos ambos contenedores con:

```shell
$ docker run -d -p 4567:4567 --name native graalvm-sample-rest-service-native
$ docker run -d -p 4568:4567 --name jre graalvm-sample-rest-service-jre
```

Revisamos el tamaño resultante de las imágenes:

```shell
$ docker image ls
REPOSITORY                           TAG                  IMAGE ID       CREATED          SIZE
graalvm-sample-rest-service-jre      latest               16bdf8457704   2 minutes ago   225MB
graalvm-sample-rest-service-native   latest               b97d8eca3f1a   3 minutes ago   33.6MB
```

Revisamos el consumo de recursos de las imágenes:

```shell
$ docker stats native jre
CONTAINER ID   NAME      CPU %     MEM USAGE / LIMIT     MEM %     NET I/O           BLOCK I/O         PIDS
8a285db0a1c4   native    0.00%     2.047MiB / 11.58GiB   0.02%     4.89kB / 1.59kB   0B / 0B           12
be3f83b4f2b0   jre       0.12%     49.59MiB / 11.58GiB   0.42%     4.63kB / 1.59kB   36.7MB / 3.35MB   28
```

En la siguiente tabla podemos comparar los valores obtenidos del contenedor JRE contra el contenedor con imagen nativa:

|                     | JRE            | Imagen Nativa |
| ------------------- | -------------- | ------------- |
| Tamaño de imagen    | 225MB          | 33.6MB        |
| Velocidad de inicio | 415ms promedio | 3ms promedio  |
| RAM usada           | 49.59MB        | 2.05MB        |

## Conclusiones

La imágenes nativas son ideales para aplicaciones que requieran una gran capacidad para escalar horizontalmente, permitiendo iniciar o finalizar numerosas instancias en muy poco tiempo, siguiendo el principio de [desechabilidad][ref9] de **"twelve-factor"**. A pesar de ser un servicio web muy simple, en la tabla se aprecia lo rápido que inicia la aplicación nativa en comparación a la que usa JRE.

La disminución del tamaño del la imagen de docker también es significativa, muy a pesar que el JAR con sus dependencias pesa solo 4,6MB, al momento de crear la imagen del contenedor con JRE se usa como base `openjdk:11-jre-slim-buster` la cual le suma 221MB del sistema operativo más la instalación del JRE para correr el JAR. Por su parte la imagen nativa pesa 33MB pero cuenta con todo lo necesario para ejecutarse por si misma, por lo que se puede usar como base la imagen `scratch` para el contenedor de docker que prácticamente no le suma nada al peso final.

Por otra parte, en esta publicación realizamos un servicio web con [Spark](https://sparkjava.com/), que no cuenta con integración "**Out-of-the-Box**" con GraalVM con el propósito de ver las [limitaciones](https://www.graalvm.org/reference-manual/native-image/Limitations/) y como tratar con ellas. Pero en el caso particular de Java, es muy común la utilización de frameworks para el desarrollo de microservicios como lo son [Spring Boot](https://spring.io/projects/spring-boot), [Micronaut](https://micronaut.io/), [Quarkus](https://quarkus.io/) o [Helidon](https://helidon.io/), que nos facilitan la integración con GraalVM y nos ahorran en la mayoría de los casos mucho tiempo y trabajo, ya que se encargan por nosotros de indicarle a la herramienta de `native-image` como realizar la compilación y como resolver las características dinámicas (JNI, Reflection, Dynamic Proxy, Class Path Resources) que son usadas por el framework y por las librerías de terceros con las que se integran.

## Referencias

* [GraalVM - Native Images](https://www.graalvm.org/docs/getting-started/#native-images)
* [GraalVM Team Blog](https://medium.com/graalvm)
* [Víctor Orozco - GraalVM](https://www.youtube.com/watch?v=ChviUG0C_p0&list=PL4fXIf4GRmkWg5aGtdf8zBcWMbHS7GOc2)
* [Understanding Class Initialization in GraalVM Native Image Generation](https://medium.com/graalvm/understanding-class-initialization-in-graalvm-native-image-generation-d765b7e4d6ed)
* [Updates on Class Initialization in GraalVM Native Image Generation](https://medium.com/graalvm/updates-on-class-initialization-in-graalvm-native-image-generation-c61faca461f7)
* [GraalVM: Making JVM Languages Performant for Microservices](https://medium.com/geekculture/graalvm-making-jvm-languages-performant-for-microservices-fcb054259ce8)

[ref1]: https://www.graalvm.org/docs/introduction/	"Introduction to GraalVM"
[ref2]: https://en.wikipedia.org/wiki/Technology_Compatibility_Kit	"Technology Compatibility Kit"
[ref3]: https://en.wikipedia.org/wiki/Just-in-time_compilation	"Just-in-time compilation"
[ref4]: https://en.wikipedia.org/wiki/Ahead-of-time_compilation	"Ahead-of-time compilation"
[ref5]: https://www.graalvm.org/reference-manual/native-image/	"Native Image"
[ref6]: https://medium.com/graalvm/understanding-class-initialization-in-graalvm-native-image-generation-d765b7e4d6ed	"Understanding Class Initialization in GraalVM Native Image Generation"
[ref7]: https://medium.com/graalvm/updates-on-class-initialization-in-graalvm-native-image-generation-c61faca461f7	"Updates on Class Initialization in GraalVM Native Image Generation"
[ref8]: https://www.graalvm.org/reference-manual/native-image/Agent/	"Assisted Configuration with Tracing Agent"
[ref9]: https://12factor.net/disposability	"Disposability"

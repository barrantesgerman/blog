---
layout: post
title: "Maven Wrapper"
date: 2021-09-26 17:35:20 -0600
categories: java
---

[Maven Wrapper](https://github.com/takari/maven-wrapper) es una herramienta que nos permite descargar y ejecutar una versión específica de [Maven](https://maven.apache.org/) de manera automatizada, es especialmente útil en desarrollos donde se usa [integración continua](https://en.wikipedia.org/wiki/Continuous_integration).

## Instalación

Para instalar el plugin, solo debemos ejecutar en la raíz del proyecto (donde el archivo `pom.xml` se encuentra)

```shell
$ mvn -N io.takari:maven:0.7.7:wrapper
```

El parámetro `-N` significa `-non-recursive` (no recursivo), por lo que solo se aplicará al proyecto principal en el directorio actual, y no a ningún submodulo del proyecto.

También podemos indicar la versión específica de Maven.

```shell
$ mvn -N io.takari:maven:wrapper -Dmaven=3.5.2
```

Luego de ejecutar el comando, se crean dos archivos (`mvnw` y `mvnw.cmd`) y un directorio oculto (`.mvn`).

```
mvnw
mvnw.cmd
.mvn
└── wrapper
    ├── MavenWrapperDownloader.java
    ├── maven-wrapper.jar
    └── maven-wrapper.properties
```

## Uso

En lugar de usar el tradicional comando `mvn`, usamos `mvnw` para Linux o `mvnw.cmd` para el caso de Windows.

```shell
# Forma tradicional
$ mvn clean install
# Equivalente en Linux
$ ./mvnw clean install
# Equivalente en Windows
$ mvnw.cmd clean install
```

La primera vez que se ejecute, el script intentará usar `.mvn/wrapper/maven-wrapper.jar` para descargar [Maven](https://maven.apache.org/) en `./m2/wrapper/dists` dentro del directorio raíz del usuario, si no encuentra a `maven-wrapper.jar`, intentará usar [cURL](https://curl.se/) o [Wget](http://www.gnu.org/software/wget/) y como último recurso compilará la clase en `.mvn/wrapper/MavenWrapperDownloader.java` para realizar la descargar.

La ruta para descargar [Maven](https://maven.apache.org/) la obtiene del archivo `.mvn/wrapper/maven-wrapper.properties` del atributo `distributionUrl`.

```properties
distributionUrl=https://repo.maven.apache.org/maven2/org/apache/maven/apache-maven/3.6.3/apache-maven-3.6.3-bin.zip
wrapperUrl=https://repo.maven.apache.org/maven2/io/takari/maven-wrapper/0.5.6/maven-wrapper-0.5.6.jar
```

## Documentación

* [Maven Wrapper Plugin](https://github.com/takari/maven-wrapper)

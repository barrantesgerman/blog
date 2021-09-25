---
layout: post
title: "Jar Ejecutable con Gradle"
date: 2021-09-25 10:23:20 -0600
categories: java
---

Los archivos `.jar` son empaquetados de clases de Java, se podría comparar con un simple archivo `.zip` que contiene todos los `.class` de nuestra aplicación. En el pueden o no haber una o varias clases principales.

Las clases principales son las que contienen un método con la firma `public static void main(String[] args)`, por ejemplo:

```java
package com.example.executable;
public class Main {
    public static void main(String[] args) {
        System.out.println("Hello World");
    }
}
```

La manera tradicional de ejecutar esta clase principal es:

```shell
$ java -cp example.jar com.example.executable.Main
Hello World
```

Si damos doble clic o ejecutamos un `java -jar nombre-archivo-jar` obtendremos un error:

```shell
$ java -jar example.jar
no hay ningún atributo de manifiesto principal en example.jar
```

Para poder ejecutarlo con doble clic o `java -jar nombre-archivo-jar` se debe incluir dentro del JAR el archivo de manifiesto, localizado en `META-INF/MANIFEST.MF` y en su contenido el atributo `Main-Class` indicar el nombre de la clase principal que se debe ejecutar.

> **NOTA:** El archivo `META-INF/MANIFEST.MF` tiene que terminar con una línea en blanco para que sea correctamente interpretado por Java.

Adicionalmente, el proyecto puede requerir dependencias de terceros, estas dependencias se debe agregar también el atributo `Class-Path` con la ruta de los JARs requeridos, separando cada uno con espacio, dentro del archivo de manifiesto.

Por ejemplo, si agregamos dependencias a [SLF4J](http://www.slf4j.org/) con [Log4j](http://logging.apache.org/log4j/1.2/) en el [build.gradle](https://github.com/barrantesgerman/executable-jar-gradle/blob/main/example/build.gradle)

```gradle
dependencies {
    implementation 'org.slf4j:slf4j-simple:1.7.25'
}
```

Modificamos la clase principal `com.example.executable.Main`:

```java
package com.example.executable;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
public class Main {
    private static final Logger LOG = LoggerFactory.getLogger(Main.class);
    public static void main(String[] args) {
        LOG.info("Hello World");
    }
}
```

Y luego modificamos el archivo `META-INF/MANIFEST.MF`:

```properties
Manifest-Version: 1.0
Class-Path: libs/slf4j-log4j12-1.7.30.jar libs/slf4j-api-1.7.30.jar libs/log4j-1.2.17.jar
Main-Class: com.example.executable.Main

```

Se deben copiar todas las dependencias en una carpeta llamada *libs* junto al archivo JAR.

Ya con esto podemos ejecutar la aplicación dando doble clic en ella o ejecutando  `java -jar nombre-archivo-jar` sin problemas.

## Copiando Dependencias

[Maven](https://maven.apache.org/) cuenta con plugins que nos facilitan la tarea de crear el archivo de manifiesto y copiar las dependencias.

Con `maven-jar-plugin` podemos indicar la clase principal en el atributo `mainClass` y el nombre de la carpeta de dependencias en el atributo `classpathPrefix`.

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-jar-plugin</artifactId>
    <version>3.2.0</version>
    <configuration>
        <archive>
            <manifest>
                <addClasspath>true</addClasspath>
                <classpathPrefix>libs/</classpathPrefix>
                <mainClass>com.example.executable.Main</mainClass>
            </manifest>
        </archive>
    </configuration>
</plugin>
```

Por su parte, el plugin `maven-dependency-plugin` nos ayuda a copiar todas las dependencias del proyecto en la carpeta indicada.

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-dependency-plugin</artifactId>
    <version>3.2.0</version>
    <executions>
        <execution>
            <id>copy-dependencies</id>
            <phase>prepare-package</phase>
            <goals>
                <goal>copy-dependencies</goal>
            </goals>
            <configuration>
                <outputDirectory>
                    ${project.build.directory}/libs
                </outputDirectory>
            </configuration>
        </execution>
    </executions>
</plugin>
```


## Generar un FatJar con Plugin

Un **FatJar** (también conocido como **UberJar**) es un archivo JAR que contiene no solo sus clases, sino que también contiene dentro de él mismo todas sus dependencias. Su ventaja consiste en ser un único archivo (.jar) para distribuir la aplicación (no se requiere copiar la carpeta *libs* como en el caso anterior).

Existen varias formas en Gradle para generar un FatJar.

### Usando el plugin de java

Se debe contar con el plugin `java` y agregar la siguiente configuración en la tarea `jar` del proyecto:

```gradle
plugins {
    id 'java'
}

jar {
    manifest {
        attributes(
            'Main-Class': 'com.example.executable.Main'
        )
    }
    from {
        configurations.runtimeClasspath.collect { it.isDirectory() ? it : zipTree(it) }
    }
}
```

De esta forma al compilar el proyecto con `gradle build` se construirá un FatJar.

### Creando una tarea propia

Igual que el anterior, se debe contar con el plugin `java` y creamos una tarea propia, en este ejemplo la tarea se llama `fatJar`:

```gradle
plugins {
    id 'java'
}

task fatJar(type: Jar) {
    manifest {
        attributes(
            'Main-Class': 'com.example.executable.Main'
        )
    }
    baseName = project.name + '-all'
    from {
        configurations.runtimeClasspath.collect { it.isDirectory() ? it : zipTree(it) }
    }
    with jar
}
```

Para crear el FatJar en este caso, se requiere llamar a la tarea con `gradle fatJar`.

### Usando un plugin de terceros

En el repositorio de Gradle existen varios plugins para crear FatJar's, uno de los más populares es el siguiente:

```gradle
plugins {
    id 'com.github.johnrengelman.shadow' version '7.0.0'
}
```

Con este [plugin](https://plugins.gradle.org/plugin/com.github.johnrengelman.shadow), no requerimos realizar ninguna configuración adicional, basta con compilar el proyecto para generar el JAR original y el FatJar.

## Código Fuente de Ejemplo

📦 [executable-jar-gradle](https://github.com/barrantesgerman/executable-jar-gradle)

## Documentación de los Plugins

* [Gradle Shadow Plugin](https://imperceptiblethoughts.com/shadow/)

* [Gradle – Create a Jar file with dependencies](https://mkyong.com/gradle/gradle-create-a-jar-file-with-dependencies/)

* [How to make a Jar file with dependencies by Gradle 7.0+?](https://stackoverflow.com/questions/59367435/how-to-make-a-jar-file-with-dependencies-by-gradle-7-0)

* [Creating a Fat Jar in Gradle](https://www.baeldung.com/gradle-fat-jar)

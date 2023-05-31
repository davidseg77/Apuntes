# Desarrollo de microservicios nativos de Red Hat Cloud con Quarkus (DO378)

### Creación de proyectos

Hay dos opciones para crear la estructura de un nuevo proyecto de Quarkus:

**Complemento Maven**

El plug-in de Quarkus Maven proporciona un comando para crear una nueva aplicación. Este plug-in agrega configuraciones específicas y ejemplos para las extensiones proporcionadas (Codestarts).

```
[user@host ~]$ mvn com.redhat.quarkus.platform:
quarkus-maven-plugin:2.13.5.Final-redhat-00002:create \ 1
    -DprojectGroupId=com.redhat.training \
    -DprojectArtifactId=getting-started \ 2
    -Dextensions=resteasy 3
[INFO] Scanning for projects...
...output omitted...
applying codestarts... 4
...output omitted...
```

1

Crear comando

2

ID y directorio del proyecto

3

Extensión inicial

4

Extensiones aplicadas codestarts

**Web Starter**

Red Hat Build of Quarkus proporciona un Iniciador web , que puede utilizar para crear proyectos con una interfaz gráfica de usuario. La URL de este iniciador web es https://code.quarkus.redhat.com/. Esta aplicación web en línea permite una selección rápida de herramientas de compilación, versiones de Java y extensiones iniciales de Quarkus.

--------------------------------------------------------------
Abra el proyecto con VSCodium o cualquier otro editor de su elección.

```
[student@workstation tenther]$ codium .
```

Ejecute la aplicación en modo de desarrollo:

```
[student@workstation tenther]$ mvn quarkus:dev
```

------------------------------------------------------------------






# seguridad-docker
Para aprender sobre seguridad en docker vamos a seguir esta guía: https://github.com/maximofernandezriera/seguridad-docker/blob/main/README.md

## PARTE A. Seguridad host
Para empezar, debe tenerse en cuenta los permisos de usuario. Lo que se recomienda es crear un grupo de docker para limitar los accesos.

![image](https://user-images.githubusercontent.com/91556405/170104516-d54f4873-2441-4b99-9c4b-ab5218b2f5f1.png)
![image](https://user-images.githubusercontent.com/91556405/170104712-214ca895-d9ab-4b3d-b74b-0b29354b7e23.png)

## PARTE B. Seguridad en el demonio
Para impedir que los usuarios no toquen la configuración indicamos en el archivo ```/etc/docker/daemon.json```:

![image](https://user-images.githubusercontent.com/91556405/170108806-54e6cfe5-df6e-43f9-afe7-1ec9dac9edd1.png)

## PARTE C. Seguridad en contenedores
Podemos usar ulimit para limitar los recursos que usa un contenedor.

![image](https://user-images.githubusercontent.com/91556405/170109536-c8dbbb8c-7994-4c8e-b862-6c460286b125.png)

Si se retira el flag de ulimits, daría los de por defecto. Se puede modificar el archivo de configuración para fijar unos límites pequeños y luego modificarlo en tiempo de ejecución.

Ejemplo de un comando sin ulimit:
```docker run -it --cpus=".5" ubuntu /bin/bash```

Otra configuración interesante es reiniciar en el fallo, poniendo en el comando ```--restart=on-failure```

### Privilegios
Se puede cambiar el usuario por defecto ```root``` al correr la imagen, y podemos forzar a ello con el flag ```-u uuid```.

Si utilizamos el siguiente comando, nos devuelve que no tenemos acceso:

```
docker run -u 4400 alpine ls /root

ls: can't open '/root': Permission denied
```

Otro flag relacionado con permisos es ```--privileged```:

```
docker run -it --privileged ubuntu

mount -t tmpfs none /mnt

df -h
```

Hay que darle privilegios porque sino nos deniega el acceso.

Otra cosa es montar el socket de docker en un contenedor. Por ejemplo, levantar un docker dentro de docker. Esto se ve mucho en CD/CI.

Por ejemplo:
```docker run -v /var/run/docker.sock:/var/run/docker.sock -it docker```

### Seguridad en imágenes
Es más fiable confiar en una imagen de la que tengo acceso al `Dockerfile` que una que ya viene construida. Puede tener malware si la imagen no es confiable.

Con docker commit se crea una imagen, aunque su uso es desaconsejable. ya que no se puede auditar. Sólo confiar en las imágenes de Docker, o de fuente confiable como Google Cloud o Microsoft Azure.

Por ejemplo, si buscamos un imagen de Wodpress encontraremos unas 8000 imágenes. La más popular es la imagen oficial que es la generada por Docker. En el caso de que la haya generado otro desarrollador estará indicado en la imagen. Por ejemplo `Multicontainer WordPress` de **Microsoft**

Se puede hacer una imagen a partir de un repositorio en GitHub como por ejemplo, esta [imagen](https://hub.docker.com/r/irespaldiza/whoami) alojada en https://github.com/irespaldiza/whoami por lo que se puede clonar o crearlo tú a partir del Dockerfile.

El servicio que lo soporta es Notary que es una de las aplicaciones a las que da soporte Docker.

Esta variable de entorno es muy aconsejable tenerla a true. Por ejemplo se puede incluir en el `bash.rc`.

A partir de una imagen es parecido a compilar a partir de la definición para generar las capas y guardarla en un registry como `docker.hub`

La imagen de alpine sólo pesa  5.61 MB porque comparte el kernel con el host.

Los comandos se ejecutan en tiempo de compilación en la preparación del entorno y ENTRYPOINT y CMD en tiempo de ejecución.

Lo normal es poner en ENTRYPOINT un comando y en CMD los parámetros (que se pueden sobrescribir)

También existe el comando ADD que es muy parecido a COPY. La diferencia es que COPY sólo permite copiar desde el equipo y ADD desde una url.

Y otra diferencia es que ADD puede copiar y descomprimir archivos, pero la capa no se podrá cachear.

Es una mala praxis no usar la versión al utilizar un paquete. Por ejemplo, `alpine`

para buscar imágenes se usa `docker search alpine`, pero no nos muestra tag

De esta forma, se puede comprobar si tiene vulnerabilidades o si dentro de un tiempo la vuelvo a generar puede dar problemas de compatibilidad.

Es  bastante normal que el código deba estar compilado lo que añade más superficie a ser atacada porque no interesa tener un compilador en la imagen ya que si nuestro contenedor es atacado el atacante podría compilar programas en nuestro sistema, cosa que está totalmente prohibida.

Por ejemplo

```dockerfile
FROM golang:alpine AS builder
ENV GO111MODULE=auto
COPY whoami.go /app/
WORKDIR /app
RUN go build -o whoami

FROM alpine
WORKDIR /app
COPY --from=builder /app/whoami /app/
ENTRYPOINT ./whoami
```

El concepto `pot` es de **Kubernetes** que son un grupo de contenedores que comparten en el mismo espacio de puertos

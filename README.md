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

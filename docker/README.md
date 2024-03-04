# Docker Avanzado

- aprenderemos a gestionar recursos fisicos, cpi, memoria, etc
- storage drivers, etc.
- politica de reinicio de contenedores
- log driver
- otros conceptos
- [los recursos de contenedores lo encuentras aqui](https://docs.docker.com/config/containers/resource_constraints/)
- stress para hacer pruebas de estres en linux
- [en docker compose, la documentacion oficial se puede ubicar aqui](https://docs.docker.com/compose/compose-file/)
- [docker storage drivers](https://docs.docker.com/storage/storagedriver/select-storage-driver/)


### ========================================================================================

# Docker Registry

Basicamente para crear repositorios privados en servidores locales o empresariales

- primero se instala el registry

[Aqui](https://hub.docker.com/_/registry)

luego, la imagen ya lista mediante un dockerfile se taggea de la siguiente manera
 - docker tag image_name host:port/image
 - docker tag image_name localhost:5000/imagen

luego, se procede a subirla

- docker push localhost:5000/image

 para descargar la imagen:

- docker pull localhost:5000/image


### ========================================================================================

# Contextos de Docker

- para ver mas del docker [daemon.json] -> (https://docs.docker.com/config/daemon/)

### ========================================================================================

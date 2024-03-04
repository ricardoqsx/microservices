|# **Docker Swarm**

## ***es un orquestador de contenedores***

- esta pensado para entornos complejos

- permite que los contenedores se expandan a traves de multiples maquinas en forma de cluster

- aprovecha mejor la capacidad de conputo distribuyendo la carga entre distintos nodos

- Permite crear un cluster de uno o mas nodos docker

- Permite disponer de "Servicios" que se despliegan de forma replicada a lo largo de los nodos del cluster

## ***Managers***

- Son los coodrinadores, los gestores del cluster para garantizar que funcione
- se encargan de repartir las tareas
- se puede tener uno o varios managers
(es lo habitual para tener HA)

*Cada nodo puede ser una maquina fisica o virtual*

## ***Workers***

- Son los que hacen el trabajo sucio
- es donde se despliegan los contenedores a partir de imagenes y donde se van a ejecutar estos servicios.

## ***Servicios***

- Cuando desplegamos una imagen en el docker engine se crea un servicio.

- un servicio identifica tareas en el contexto de una aplicacion, ejemplo un web server, una base de datos, etc.

- siempre debemos indicar la imagen a usar para crear los contenedores y que comandos lanzar dentro. 

- para esquematizarlo:

en un Manager se define un servicio: ejempplo -> [Servicio web HTTP],
Ese servicio se va a desplegar en 2 workers y en cada nodo worker tendre una tarea http y se desplegara un contenedor correspondiente a ese tipo de tareas

# ====================

# Crear Clusters swarm
contunto de maquinas con docker instalados que interactuan entre ellas

el cluster se inicia primero creando un nodo tipo manager que va a ser el inicial del cual partira el resto.

**docker swarm init --advertise-addr *<ip_address>***

- con esto se inicializa el cluster
- se le asigna un nombre (el cual es una cadena larga)

- seguidamente te da el comando para unir workers:

**docker swarm join -- token *<id_de_token ip_address:2377>***

- para añadir un manager al swarm recien inicializado, hay que correr el siguiente comando:

**docker swarm join-token manager**

- y seguir las instrucciones.

# ====================

# Gestion de nodos

- se utiliza el comando docker node para visualizar opciones del nodo

**docker node ls**

- con este se pueden visualizar los nodos que estan activos.

- los managers se pueden ver en la columna **manager status**, ya que si es manager, tiene un status leader, de lo contrario no aparece nada e implica que es un worker

estos comandos de gestion solo pueden ser utilizados desde un manager, desde un worker no se puede ya que trabaja como esclavo

- En caso de que se pierda de vista el token, se puede usar el siguiente comando:

**docker swarm manage join-token <node_type>**

- puede ser manager o worker, osea que puede ser asi:
**docker swarm manage join token manager**
**docker swarm manage join tocker worker**

- seguidamente el prompt enviara un comando junto con el token que se utilizara para agregarse al manager

- Adicional, habran diferentes tipos de estados al tratarse de managers, ejemplo:

1. Leader
2. Reachable
3. Unavailable

Si cae el cluster lider, habra un consenso entre los nodos lider para poner a un nuevo lider

- otra cosa: tmbn se puede cambiar el tipo de nodo, de worker a manager y viceversa.

- Al ver **docker info**, al final envia un mensaje de que correr swarm en 2 nodos managers, ya que no posee tolerancia a fallos y hay que configurar un numero impar, en este caso 3. si solo hay 2 existe posiblidad de split brain ya que si se cae el lider y no se ven uno al otro, no se pueden poner de acuerdo cual tomara el rol de manager/lider

Implementacion de Tipo Raft.
modo de trabajo de cluster, en el cual recomienda tener un numero impar de nodos maestros en un cluster 

## Caidas de nodo manager

- Se reorganiza automaticamente, en caso de caidas otro nodo toma el lugar del lider
- en caso de que el antiguo lider se recupere, el nodo que toma el liderazgo lo mantiene
- Para garantizar la HA, es necesario que docker este activo en mas de la mitad de los operadores
(ejemplo 3, se cae 1 quedan 2, mas del 50%)

## Promover y degradar nodos

- se utiliza el comando **docker node** para esto

- promote: para promover uno o mas nodos, se usa el ID

- demote: para degradar uno o mas nodos, se usa el ID 

- IMPORTANTE: tratar de mantener a toda costa los 3 nodos para evitar inconvenientes.

adicional, existe el comando **docker node update --role (worker|manager)**  el cual realizaria la misma funcion

## **HINT**: docker no recomienda tener mas de 7 manager, ya que no aporta mucho, o 3 o 5 o 7 max

## etiquetar un nodo:

esto para evitar que un cluster envie tareas ahi, o para tenerlas bien definidas
se usa el comando 
<docker service update --labe-add key=val nodo>


- si se agrega una etiqueta se puede restringir que si se crea un servicio, solo se despliegue en la etiqueta mencionada

<docker service create servicio --constraint node.label.key==label nginx>

- aqui evita la etiqueta
<docker service create servicio --constraint node.label.key!=label nginx>

## nodos preferentes


- asi se ejecuta un nodo preferente, una especie de condicion or, ya que si se ejecuta, prefiere buscar la etiqueta, y si no la encuentra igual avanza
<docker service create servicio --placement-pref spread=node.label.key!=label --replicas=3 nginx>

## Quitar nodos del cluster

<docker swarm leave> <- con esto desde el nodo se abandona el cluster desde el nodo
<docker node rm nombre_nodo> con esto desde un manager se borra el nodo del cluster

# ====================

# Servicios y replicas

- Se usan para desplegar aplicaciones.

- Un servicio es un microservicio, es decir, un componente individual que pertenece a un app mas grande.

- Podemos por ejemplo, considerar un servicio a un servidor web, una base de datos o cualquier otro componente de mi aplicacion que este ofreciendo algun recurso al resto.

- Cuando creo un servicio debo indicar una serie de caracteristicas asociadas al mismo:

1. Numero de replicas del servicio que se van a ejecutar
2. configuracion de CPU y memoria
3. una posible red de tipo "overlay"
4. El puerto por el que vamos a poder acceder al servicio

## Task ó Tareas

- Cuando se crea un servicio en realidad lo que se hace es lanzar una o varias tareas de tipo replica en los nodos

- Las tareas se ejecutan de manera completamente independiente, unas de otras.

la logica de ejecucion va mas o menos de esta manera:

1. primero esta el servicio
2. el servicio ejecuta una tarea
3. la tarea ejecuta el contenedor

- Importante tener en cuenta que cada contenedor es independiente y es swarm el que gestiona

## Tipos de servicios

1. Replicados: son tareas que se distribuyen en los nodos necesarios
2. Globales: ejecuta una tarea en cada nodo del cluster

- para crear un servicio global

<docker service create --name web4 -p 8006:80 --mode global httpd>

## Crear un servicio

- Se usa el comando:

<docker service create --name apache httpd>

- Igualmente, para ver los servicios creados:

<docker service ls>

- Para ver los contenedores del servicio

<docker service ps service_name>

- para ver informacion del servicio:

<docker service inspect service_name>

- algo interesante de este ultimo es la opcion --pretty, que muestra el output mas "bonito"

## Acceder a un servicio

- similar a lo visto de publicar puertos en docker
- ejemplo:

<docker service create --name servicio_web --publish published=9000,target=80 httpd>

- cuando se publica un puerto, este en auto sera abierto y replicado en todos los nodos del cluster

## replicas en los servicios

es la posibilidad dada de clonar para que se pueda dar servicios en varios servidores, permite sobre todo mejorar el rendimiento y tener una mayor disponibilidad de nuestra app

<docker service create --name web3 -p 8003:80 --replicas=3 imagen>

- se crea a lo largo del custer, y se puede ver con service ls

## luego de la creacion de un servicio; comando scale

docker service scale servicio web1=5

con esto se modifican la cantidad de replicas e inmediatamente el cluster ejecuta la tarea de tener 5 replicas del servicio dicho a lo largo y ancho de todos los nodos del cluster

esto funciona a modo de ajuste, por lo cual si se quieren tener 5 replicas y luego 3, solo se ejecuta el comando con el cambio en la opcion.

importante mencionar tambien que, en el caso de lanzar un servicio global y luego agregar un nodo, este en auto se replicara en este nodo.

importante tener en cuenta que las replicas se dan es desde la parte del servicio y no tanto desde cosas del backend, ejemplo una base de datos, no es lo mismo tener 3, 5 contenedores recibiendo trafico, que varias bases de datos todas teniendo informacion distintas

para temas de replicas en bases de datos, al menos en el caso de mysql hay que usar aplicaciones como mysql galera.

a swarm no le importa que hay dentro de un contenedor ya que solo recibe la orden, y una replica es una orden de copiar por lo que hay que tener en cuenta estas caracteristicas

## actualizar o modificar un servicio

- se hace con docker service update

### ejemplo:

- si tengo un servicio sin replicas:

docker service scale servicio=5

- esto aumentaria el numero de replicas a 5, luego modificaremos el limite de RAM que puede usar un servicio:

docker service update --limit-memory 10m servicio

- aqui se logra apreciar que debe actualizar todas las replicas

# ====================

# Redes Overlays

es una red distribuida que se expande entre todos los host de todos los clusters, permite que los contenedores sean accesibles desde cualquier sitio, generalmente esto swarm lo gestiona de manera automatica, en auto se activa  el swarm se activan estas redes

- ingress: esta por encima de la red bridge, gestiona el control del trafico de los servicios

- docker_gwbridge: es la que permite interconectar distintos demonios docker individuales con otros daemons docker dentro del cluster, basicamente conecta entre si los servicos docker de las maquinas del cluster, osea funciona como gateway

# ====================

# Otros conceptos

## impedir que un nodo reciba tareas

- por defecto, todos los nodos reciben tareas, incluso los managers.
- dentro del comando docker node update hay una opcion llamada --availability y esta cuenta con 3 opciones:

1. active: permite tareas en el nodo
2. pause: pausa las tareas para luego continuarlas
3. drain: no permite tareas en el nodo

- el comando seria:

<docker node update --availability drain nombre_nodo>



# ====================

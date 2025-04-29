# Apuntes Curso Kubernetes

## Seccion 1: Minikube

### Instalacion de minikube

Sobre como instalar minikube, la info la tienes [aqui](https://minikube.sigs.k8s.io/docs/start/)

Segun tu sistema, y luego de tener las dependencias necesarias instaladas (segun el caso, ya sea docker o virtualbox), para iniciar el cluster de kubernetes se puede iniciar con el siguiente comando

`minikube start --vm-driver=docker`

a partir de aqui, tu cluster deberia estar arriba.

aparte a esto, tambien se necesita de una herramienta para llevar a cabo la administracion del cluster, en este caso, `kubectl`

### Instalacion de kubectl

Segun el sistema operativo, la instalacion debe seguirse dados los pasos en su [pagina oficial](https://kubernetes.io/docs/tasks/tools/), en todo caso para linux se debe instalar de la siguiente forma:

1. Descargar el respectivo binario con el siguiente comando (este es para x86):
`curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"`

2. validarlo de la siguiente forma:


- descargar el checksum con: `curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"`

- validar el binario contra el archivo checksum:

`echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check`

3. instalar kubectl

`sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl`

- si no tienes acceso root en el sistema objetivo, puedes aun instalarlo en el directorio `~/.local/bin/`

4. probar para asegurar que la version instalada este actualizada:

`kubectl version --client`

- o usa la vista detallada:

`kubectl version --client --output=yaml`

### Comandos utiles para minikube

- `minikube` trae todas las opciones posibles
- `minikube status` para ver el estado del cluster
- `minikube logs` para ver los mensajes de logs del cluster
- `minikube ip` para ver la ip del cluster
- `minikube ssh` para entrar al cluster

### Archivos de minikube

hay 2 directorios con los que se trabaja en minikube

```
~/.kube
~/.minikube
```

.kube contiene un archivo de configuracion de conexion de otros cluster de kubernetes

### Crear otros clusters de minikube

para ver la cantidad de nodos corriendo en la maquina fisica y otros datos relevantes `minikube profile list`

para iniciar distintos nodos con minikube `minikube start --driver=docker -p cluster1 --node=2`

### Cambiar la configuracion del cluster

Cambiar la memoria al cluster: `minikube config set memory 2G -p minikube`

### Cambiar de cluster

Se cambia con contextos

`minikube profile nombre_cluster`

o editando el archivo

`~/.kube/config`

en la linea de contextos

### Dashboard de minikube

Es una herramienta web para trabajar con kubernetes, se activa con:

`minikube dashboard`

### Trabajar con otro container-runtime

se hace con el siguiente comando:

`minikube start --container-runtime=docker`


### Borrar el Cluster

se hace con:

`minikube delete`

si tiene un nombre distinto a minikube se usa la opcion `-p nombre`

## Seccion 2: pods en Kubernetes

### ciclo de vida de una aplicacion:

Tipicamente, en un entorno de contenedores, primero debe convertirse en una imagen que luego pueda desplegarse en forma de contenedor.

Hay 2 formas, modo imperativo y modo declarativo:

- imperativo: similar a usar docker run
- declarativo: similar a docker compose

en el modo declarativo se crea un manifest de tipo yaml para dar las reglas sobre como distribuir mi aplicacion, todo se guarda en ectcd (la base de datos de kubernetes)

Estado deseado de la aplicacion: k8s continuamente pregunta si el estado real coincide con el estado deseado de la aplicacion, por ende, si en la realidad solo hay 2 replicas y se pidieron 5, este trabajara en eso en automatico

### Que es un Pod

un pod conforman el objeto minimo que tenemos en kubernetes, un pod en realidad es un contenedor, una burbuja que le da una serie de caracteristicas y funcionalidades a los contenedores de kubernetes

Este contenedor se guarda dentro de un pod, le da funcionalidades adiciones que no tiene un contenedor por si mismo.

Se puede tener mas de un contenedor dentro de un pod

que funciones le da?

- direcciones ip
- puertos
- hostnames
- sockets
- memoria
- volumenes
- etc.

todo esto englobado en una sola ip.

Este pod se despiega dentro de un nodo del cluster (un worker).

Otras caracteristicas:
- los pods ademas se pueden replicar contenedores.
- por defecto son stateless
- no tienen estado
- no se debe guardar informacion en ellos

### Multiples contenedores en PODS

En el ambito de microservicios, es importante tener en cuenta que habran casos donde no es aconsejable tener varios contenedores en un mismo pod, debido a casos de aplicativos con bases de datos, para evitar que choque cosas como:

- ciclo de vida
- backups
- reinicios (rebotes en España)
- actualizaciones

todo esto entre contenedores, es preferible mantenerlas separadas, siguiendo la logica de los microservicios

Se juntan los pods cuando las aplicaciones contenerizadas estan intimamente relacionadas

### Crear el primer pod:

se crean con el comando

`kubectl run nginx1 --image=nginx`

basicamente dice:

- kubectl: cliente kubernetes
- run: corre
- nginx: imagen con nombre nginx
- -- image=nginx: y usa la image nginx

luego se puede visualizar si ya esta creado con

`kubectl get pods`

aqui te da info como:
- nombre del pod
- si esta listo
- estado
- reinicios
- edad, osea el tiempo de vida del pod

otra opcion interesante para obtener mas info:

`kubectl get pods -o wide`

añade:
- ip
- que nodo lo ejecuta
- modo nominated
- readiness gates
 
### Ver las propiedades de un pod, comando `describe`

seria de la siguiente manera, siguiendo el ejemplo anterior:

`kubectl describe pod/nginx1`

con esto se obtiene toda la informacion relevante sobre ese pod

importante describir el recurso al que se refiere, ejemplo si es un pod, un load balancer, deployment, namespace, etc.

### Ejecutar comandos contra un pod, comando `exec`


es similar a docker, con la diferencia de que se esta invocando a un pod y no a un contenedores

`kubectl exec nginx1 -- ls`

importante agregar los -- 2 guiones antes del comando.

el modo interactivo es similar

`kubectl exec -it -- bash`


### Ver logs de un pod y modificar su contenido

ejemplo, si se lanza un pod:

`kubectl run apache --image=httpd --port=8080 --generator=run-pod/v1`

con el comando logs se puede ver el pod

`kubectl logs pod/apache`

si se desea debuggear, con la opcion -f se ajusta a esa necesidad

`kubectl logs -f pod/apache`


para ver determinada cantidad de lineas en los logs

`kubectl logs apache --tail=numero`


### probar el pod con `kubectl proxy`

`kubectl proxy`

esto solo abre en localhost un puerto donde se pueden comprobar caracteristicas y recursos

es un tema de recursos mas que de otra cosa, si le encuentro alguna utilidad mas adelante, la anotare en esta seccion

es para ver los objetos y navegar por ellos

### Probar POD con un servicio

para exponer servicios al exterior:

`kubectl expose pod nginx1 --port=80 --name=nginx-svc --type=LoadBalancer`

por defecto kubernetes agrega puertos a partir del 30k

con minikube con el siguiente comando se puede obtener una url para ver el servicio expuesto:

`minikube service apache-svc --url`

-- por algun motivo este metodo no me deja ver los servicios expuestos desde mi servidor a mi maquina local, pendiente investigar como solucionar esto.

### Port Forwarding con PODs

algo asi como los puertos de docker

kubectl port-forward nginx1 9999:80

con esto lo que se deberia hacer es traducir el puerto origen con el destino similar a docker, y se accede desde la ip local

### ver los PODS desde un nodo del cluster

en este video se refiere a acceder al nodo de minikube con ssh

`minikube ssh`

y con `docker ps` se pueden ver todos los contenedores de ese nodo

### introduccion a YAML

- ver archivo `01-introduccion+a+YAML.txt`

### crear un POD mediante un manifest

basicamente cuando se refiere a un manifest, es a traves de un archivo yaml que tiene todos los requerimientos para ese servicio/pod/etc, se hace asi:

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    zone: prod
    version: v1
spec:
  containers:
   - name: nginx   
     image: apasoft/nginx:v1
```

luego solo se aplica l manifiesto con el siguiente comando:

`kubectl create -f nginx.yaml`

### generar la configuracion de un pod en json o yaml

es una opcion del get que permite extraer toda la configuracion 

`kubectl get pod nginx -o yaml`

tambien es posible exportarlo en json

### opciones interesantes sobre borrar pods

se pueden borrar multiples pods solo poniendo comas

`kubectl delete pod pod1, pod2, pod3`

tambien se puede mediante etiquetas o darle un periodo de gracia en el cual el pod estara vivo

`kubectl delete pod pod1 --grace-period=10`

serian 10 segundos en este caso.

otra opcion es el --now para hacerlo inmediatamente

`kubectl delete pod pod1 --now`

para borrarlos todos

`kubectl delete pods --all`

otra cosa, funciona para todos los tipos de objetos y/o servicios

si se usa:

`kubectl delete all --all`

borra todos los pods, servicios, todo, mas o menos como docker system prune

### pods multicontenedores

para esto empezamos con 2 contenedores, un nginx y otro que monitorice que nginx este arriba:

```
apiVersion: v1
kind: Pod
metadata:
  name: multi
spec:
  containers:
  - name: web
    image: nginx
    ports:
    - containerPort: 80  
  - name: frontal
    image: alpine
    command: ["watch", "-n5", "ping",  "localhost"]
```
tener en cuenta que a pesar de ser solo 2 contenedores dentro de un pod, este solo tendra 1 direccion ip ya que el pod es como si fuese una maquina fisica donde estan dentro los contenedores

genealmente se usaria el comando `kubectl logs multi` para ver el log del pod, pero el problema con este es que solo permite ver el de 1, con lo cual hay que usar otra opcion:

`kubectl logs multi -c frontal`

con la opcion -f se queda observando

con esto se refiere al pod y luego al contenedor

otra opcion seria

`kubectl exec multi -c frontal -- date`

### Comando apply, trabajando de forma declarativa

basicamente lo que hace es que aplica configuraciones por ejemplo, si se hacen cambios en vivo se apliquen sin necesidad de bajar y subir el pod

- ejemplo:

`kubectl apply -f nginx.yaml`

la diferencia con el create, es que se guardan los estados

generalmente se usa cuando haces cambios y quieres aplicarlos, y tambien se puede usar a partir de cero

en el caso de los deployments mantiene un historico completo de los cambios que se van haciendo

### rebotar POD (Restart)

son politicas de restart (politicas de rebote)

hay 3 opciones

- Always
- OnFalure
- Never

a los pods que  no se les pone la propiedad, reinician por defecto

```
apiVersion: v1
kind: Pod
metadata:
  name: tomcat
  labels:
    app: tomcat
spec:
  containers:
   - name: tomcat     
     image: tomcat
  restartPolicy: OnFailure
```

## Seccion 3: Labels, selectors, anotaciones, etiquetas en objetos

### Labels

Labels son etiquetas, es una forma de establecer relaciones entre distintos objetos de kubernetes

- no hay una determinada forma de llamar a una label, algunas son de sistemas

en la carpeta deployments de ejecuta un `kubectl apply -f deployments/tomcat.yaml` con lo cual se crea un pod con un contenedor tomcat dentro y una etiqueta estado

el comando para ver las etiquetas seria:

`kubectl get pod tomcat --show-labels`

si adicional se usa la opcion `-L` se agregaria una columna (al parecer se trata de una especie clave:valor)
para este caso espeifico mostraria todos los contenedores que tengan un label estado, adicional con una coma (,) se agrega otra label para ver que objetos pertenecen a ese label

`kubectl get pod tomcat --show-labels -L estado`

### Trabajar con labels

hay una forma imperativa de agregar, modificar o borrar etiquetas

los nombres de las etiquetas para las key:

- pueden empezar por numero o por letra
- no pueden tener mas de 63 caracteres de largo
- admiten puntos y guiones
- no admiten espacios en blanco y otros caracteres raros
- debe haber una nomenclatura bien definida para la empresa

para actualizar hay que sobreescribir con `--overwrite`

- ejemplo:
`kubectl label --overwrite pod/tomcat estado=test`

- para borrar:
`kubectl label pod/tomcat responsable-`

no hay un responsable, osea elimina la etiqueta

### Selectors

los selectores como como wheres que se usan para localizar objetos que tengan determinadas etiquetas

- un ejemplo: buscar todos los pods que tengan la etiqueta "desarrollo"

tiene valores como `=, == ó !=` casi que igualito que un where en sql, otro ejemplo

`kubectl get pods --show-labels -l 'estado in(desarrollo, testing)'`

con esto funciona igual que la clausula in en sql, que trae los valores incluidos en el parentesis

- un ejemplo practico o caso de uso:

`kubectl delete pods -l estado=desarrollo`

borrara todos los pods que tengan la etiqueta desarrollo


### Anotaciones

las anotaciones son similares porque tambien son clave:valor pero en realidad son mas bien descriptivas, permiten poner por ejemplo documentacion, comentarios, vamos que sirven para describir y documentar nuestros componentes

## Seccion 4: Deployments

### Workloads y Controllers

Son pods que funcionan teniendo los contenedores dentro, pero tienen determinadas caracteristicas (que no tiene un pod tipicamente) que se necesitan dentro de un cluster como por ejemplo:

- creacion de nuevas replicas
- escalamiento
- crear nuevas replicas si estan fallan
- etc.

todos los componentes o workloads estan intimamente relacionados, son como envolturas que toman uno o varios pods que haran que se ejecuten de determinada manera

los controller tienen como trabajo estar continuamente comprobando y controlando que el cluster este correcto y que los pods y workloads se adapten a lo que yo quiero (son componentes de tipo loop), los pods se comenzaran a crear mediante otros componentes, ejemplo deployments 

#### Deployment: 
son componentes que envuelven a los pods y que les dan una serie de caracteristicas, ejemplo hacer updates y rollbacks de manera sencilla, ademas de comprobar que funciona correctamente.

#### Reprica Set: 
se encargan de hacer el escalado de los pods cuando es necesario, de acuerdo a lo que el cluster tenga indicado.

#### Replication Controller:
esta reemplazando al replica set

Stateful Set: objeto que gestiona el despliegue y escalado de los pods, garantiza el orden y unicidad de esos pods

#### Daemon Sets
asegura que todos los nodos del cluster van a tener una copia de un pod

#### Jobs
Es un componente que crea uno o varios pods y se asegura que un determinado numero de ellos terminen satisfactoriamente (osea que se cumpla el ciclo completo)

#### CronJob
muy parecido pero ejecuta una parte planificada con scheduling


### Deployments
Es la forma habitual en la que se despliegan aplicaciones
Que es? el objeto inicial mas basico que puedo tener dentro de kubernetes, es un contenedor, pero este no puede lanzarse sin mas sino que debe lanzarse dentro de un pod, este es a su vez el objeto minimo que se puede ejecutar en kubernetes, pero los pods tienen distintos problemas:

- No Escalan
- no se autocuran (no se recuperan ante caidas)
- los updates y rollbacks se vuelven complicados

Por esto, se utilizan los deployments, es una especie de burbuja que engloba unas propiedades o caracteristicas que resuelven los problemas de los pods anteriormente comentados

Cuando se crea un deployment, en automatico tambien se crea otro objeto, un replicaSet, este va de la mano con los deployments y es raro que se manejen de forma independiente.

Es decir, cuando tengo un deployment, tengo:

- El Deployment
- El ReplicaSet
- El Pod
- Los Contenedores (que pueden ser uno o mas)

### Modo imperativo

un ejemplo del modo imperactivo

```
kubectl create deployment apache --image=httpd
```
y para ver los replicaset
```
kubectl get rs
```

### Ver info de un deployment

se puede ver con:

```
kubectl describe [tipo de objeto, si es un deploy, rs, etc] nombre [del objeto]

ejemplo:

kubectl describe deploy httpd
```

arroja toda la informacion sobre el deployment
trae informacion sobre:

- el pod template con los pods que tiene ese deployment
  - esto seria una especie de caracteristicas que quiero tengan los pods asociados a ese deployment
- etiquetas
- el namespace donce esta corriendo
- el numero de replicas
- el selector
- el tipo de estrategia (StrategyType)
- viejos y nuevos ReplicaSet

recordar que tambien se puede solicitar esta info
en formato yaml o json asi:

`
kubectl get deploy apache -o yaml [o json]
`

dara todas las propiedades del deployment

con esto se saca la configuracion actual del deployment y el estado

proximamante se hablara de los selflinks

### Dashboards, deployments y ReplicaSets

crearemos el archivo:

```
apiVersion: apps/v1 # i se Usa apps/v1beta2 para versiones anteriores a 1.9.0
kind: Deployment
metadata:
  name: nginx-d
spec:
  selector:   #permite seleccionar un conjunto de objetos que cumplan las condicione
    matchLabels:
      app: nginx
  replicas: 2 # indica al controlador que ejecute 2 pods
  template:   # Plantilla que define los containers
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```

se crea con el comando:

`kubectl create -f deploy-nginx.yaml`

pero hay un detalle con el comando create, crea el objeto de modo imperativo, por lo cual si se hacen modificaciones habria que volver a crear el objeto, entonces para trabajar es mejor el apply:

`kubectl apply -f deploy-nginx.yaml`

apply automaticamente crea el objeto si no existe, y lo almacena en etcd de forma que luego se puede modificar el objeto sin volver a crearlo

para buscar pods en una etiqueta se da con la opcion `-l` (de labels) asi:

`
kubectl get pods -l apps=nginx
`

y para tener los datos en una columna propia:

`
kubectl get pods -L app
`

asi traeria todos los pods que tienen la etiqueta app
 y finalmente, para hacer un filtro mayor:

`
kubectl get pods -l app=nginx -L app
`

esto para agregar un filtro adicional y solo buscar pods (u otros recursos) que sean de tipo app y que tengan la etiqueta nginx

para ver elementos especificos:

`
kubectl get pods,deploy,rs
`

### Comando Edit

para ver un deploy (en este caso el deploy que hicimos mas arriba) se usaria el comando:

`
kubectl describe deploy nginx-d
`

porque nginx-d? porque es como se llama el deployment, esta descrito en `metadata`, el comando describe da toda la informacion disponible del objeto, en este caso el deployment `nginx-d`

importante es, tener los archivos de configuracion en un lugar seguro para poder trabajarlo luego, de lo contrario entonces se podria modificar el deploy con el comando

`
kubectl edit deploy eploy-nginx.yaml
`

con esto se abrira en el editor de texto por defecto y permitiria modificarlo de forma dinamica

en caso de que se quiera obtener la informacion del deploy para luego trabajarla se puede usar la opcion `-o`

`kubectl get deploy nginx-d -o yaml`

### Escalar un deployment

otra forma (ademas de la que ya vimos) para escalar deployments, por medio de comandos podria ser asi:

`kubectl scale deploy nginx-d --replicas=3`

otra forma util de hacerlo con las etiquetas

`kubectl scale deploy -l estado=1 --replicas=10`

ya que en el deploy hemos agregado como propiedad al deploy una label llamada estado y un valor igual a 1 (entre comillas)

### Consideraciones sobre el escalado

un escalado basicamente es una copia de pods
lo cual lo hace valido para por ejemplo aplicaciones ya que por ejemplo, una aplicacion basada en apache, y un volumen sirviendo los datos con los archivos html puede crecer xq a partir de N cantidad de replicas puede simplemente leer el volumen y servir los datos

En el caso de las bases de datos esto no funciona debido a su naturaleza centralizada (si se hiciera, kubernetes es agnostico de lo que exista dentro del pod, el le hara su deploy pero esto no implica que funcionara, ya que el motor de base de datos gestiona los datos de tal manera que no permite una lectura conjunta), con lo cual se debe recurrir a soluciones como MySQL cluster o Mariadb Galera Cluster para proporcionarle esa capacidad de escalado

## Seccion 5: Servicios, o como conectarnos a los recursos

#### Que es un servicio?

Es un recurso u objeto que dan la capacidad de conectarnos a nuestros contenedores

#### Tipos de Servicio

- ClusterIP: Accesible solo desde dentro del cluster
- NodePort: Accesible desde afuera del cluster
- LoadBalancer: Accesible desde afuera del cluster e integrado con LoadBalancers de terceros (AWS, Azure, GCP)

Se asocian los servicios con los deployments y pods por medio de los selectores (que son una especie de where o condicion para decir como se asocian los objetos entre si), labels, etc.

#### Crear Servicios

Primero creamos un deployment

`kubectl create deployment --image=httpd`

luego lo exponemos con este comando:

`kubectl expose deploy apache50 --port=80 --type=NodePort`

a partir de aqui se puede acceder desde fuera del cluster al servicio de kubernetes viendo el puerto asociado y la ip del mismo

con `kubectl describe svc apache50` podremos ver  el endpoint a donde esta asociado ese pod, adicional, si no se le especifica un label, kubernetes le agrega un label igualito que el nombre del deploy, en este caso `apache50`

#### Gestionar y configurar Servicios

creando y desployments y luego su respectivo servicio:

```
# deployment master
kubectl create deployment redis-master --image=redis
# Deployment Cliente
kubectl create deployment redis-cli --image=redis
# Exposicion de servicio mediante ClusterIP
kubectl expose deploy redis-master --port=6359 --type=CusterIP
```

luego al conectarse al pod mediante `kubectl exec -it <nombre-pod> -- bash` al ejecutar redis-cli -h redis-master, si se fijan el expose esta exponiendo el redis-master, pero mas importante aun, si se le hace un `kubectl describe svc redis-master` se podran percatar de que tiene la label `redis-master`

con esto se logra que entre servicios se logren ver

como se puede agregar mas de un objeto en un manifiesto yaml? simple, con 3 guiones
`---` a modo de separar cada una de las secciones

con el comando `kubectl describe endpoints <nombre-servicio>` con lo cual se puede ver informacion mas detallada del las ip del pod, los selectores, labels, etc.

importante destacar que kubernetes agrega ciertas variables de entorno para el servicio relacionado, se puede ver ejecutando ENV dentro del pod

#### Rolling Updates
#### Rollbacks
#### Recreate

## Seccion 5: Namespaces

### Que es un namespace?

Que es? los espacios de nombre es una division logica del cluster de kubernetes para separarlos por zonas virtuales

- Por defecto todo se crea en default
- kube-system, es el que maneja kubernetes, 
- kube-public se usa por todos los usuarios, funciona mas o menos como la carpeta public de windows, donde todo lo que se deja ahi es visible para otros usuarios

con `kubectl describe ns default` se puede obtener informacion como:

- labels o etiquetas
- anotaciones
- status o estado
- si hay quotas habilitadas
- o si hay limites de rango activos

si uso `kubectl get pods -n <namespace>` puedo ver los pods que existen dentro de ese namespace

### Crear y gestionar namespaces

existen formas imporativas y declarativas para crearlos:

La Imperativa (osea con un comando)

`kubectl create namespace n1`

y la delcarativa (con un manifiesto):

```
apiVersion: v1
kind: Namespace
metadata:
  name: dev1
  labels:
    tipo: desarrollo
```

adicional, tambien esta el comando
`kubectl rollout history deploy elastic -n dev1`

para ver el historial de cambios en el objeto

### Namespace por defecto (contextos)

se puede cambiar de 2 formas, en el archivo de configuracion dentro de `.kube/config` cambiar la linea que dice:

- current-context: Default

la otra manera para verlo, con el comando:

`kubectl config view`

luego se usaria el comando:

`kubectl config set-context --current --namespace=dev1`

### Configurar Recursos a un namespace

esto seria para poner limites a los pods que se creen dentro del namespace, tenemos el siguiente tipo de objeto que es el LimitRange

```
apiVersion: v1
kind: LimitRange
metadata:
  name: recursos
spec:
  limits:
  - default:
      memory: 512Mi
      cpu: 1
    defaultRequest:
      memory: 256Mi
      cpu: 0.5
    max:
      memory: 1Gi
      cpu: 4
    min:
      memory: 128Mi
      cpu: 0.5
    type: Container

```

con esto se establecen ciertos limites como los rangos minimos y maximos que pueden tomar tanto a nivel de namespace como a nivel de cluster

### Eventos en un namespace

basicamente es todo lo que sucede en ese namespace, se puede ver con:

`kubectl get events -n <namespace>`

si solo nos interesa saber por ejemplo, los warnings:

`kubectl get events --field-selector type='Warning'`

otra opcion es con -w

`kubectl get events -n dev1 -w`

es como un tail -f

## Seccion 6: Rolling Updates

### conceptos sobre Updates

basicamente seria un metodo o objeto mediante el cual se pueden hacer actualizaciones sin perder la disponibilidad, en caso de que por ejemplo, si tienes un deployment de 10 replicas, y actualizas, en lugar de reiniciarlas todas al actualizar algun valor (cambio de nodeport a clusterip por ejemplo) van actualizandose de manera evolutiva, para que de esta manera siempre se garantice la disponibilidad de la aplicacion.

en este caso, en spec tenemos 2 objetos o politicas para esto:

```
spec:
  strategy:
  type: RollingUpdate
```
si se necesitan agregar ciertos limites
```
  strategy:
     type: RollingUpdate
     rollingUpdate:
        maxUnavailable: 1
        maxSurge: 1
  minReadySeconds: 2
  ```

ó en vez de usar los rolling updates, usar Recreate

```
spec:
  strategy:
  type: Recreate
```

### Comprobaciones sobre el proceso de rolling

importante aclarar que un update se produce cuando se hacen cambios inherentes al puerto, por ejemplo, el cambio de puerto, de label, o asi

con 
`kubectl rollout history deploy nginx-d`
 logramos ver el historial de cambios que se han hecho sobre ese deploy, si se usa `kubectl rollout history deploy nginx-d --revision=2` se puede ver el cambio que se dio en esa revision, adicional en cada revision se puede ver el hash del pod que estuvo activo en ese momento, si se hace un kubectl get pods se puede ver que tiene el hash de la ultima revision

### Deshacer Cambios - Rollback

 si se necesita volver a una revision que ya funcionaba debido a que quiza haya un problema y no se quiere volver a hacer todo desde cero, se puede usar el comando `kubectl rollout undo deploy nginx-d --to-revision=2` y se puede revisar si ya regreso a la revision 2 con `kubectl rollout status deploy nginx-d`, si se revisa con el comando `kubectl rollout history deploy nginx-d` se podra constatar que lo que sucede es que crea una nueva revision y borra la que se 'regreso'

### Recrear pods en el update

esto utiliza la estrategia recreate, la cual es algo mas agresiva pues elimina todos los pods de golpe y los recrea

## Seccion 7: variables, ConfigMaps

### Variables:

Estas son utilizadas por las aplicaciones dentro del contenedor (un ejemplo clarisimo seria mysql o wordpress) que utilizan estas variables para automatizar lo que tiene que ver con el uso de la aplicacion como tal, un ejemplo para llevar las vartiables a un contenedor dentro de un pod mediante manifiesto seria:

```
apiVersion: v1
kind: Pod
metadata:
  name: var-ejemplo
  labels:
    app: variables 
spec:
  containers:
  - name: contenedor-variables
    image: gcr.io/google-samples/node-hello:1.0
    env:
    - name: NOMBRE
      value: "CURSO DE KUBERNETES"
    - name: PROPIETARIO
      value: "Apasoft Training"

```

en el cual se puede ver que en spec dentro de containers se agrega el valor env y luego name, value, name, value, los cuales significan el nombre de la variable, y el valor de esta sucesivamente.

### ConfigMaps

cuando hay muchas variables se utilizan los configMaps, es un archivo basicamente con puras variables

se configuran asi:

con comandos: `kubectl create configmap cf1 --from-literal=usuario=user1 --from-literal=password=passwd`, asi mismo, para ver los configmaps se usa el comando get `kubectl get cm`, otra cosa interesante es que si vemos el configmap en formato yaml con `kubectl get cm cf1 -o yaml` se ve mas o menos como se puede ir creando un configmap

```
apiVersion: v1
data:
  password: passwd
  usuario: user1
kind: ConfigMap
metadata:
  creationTimestamp: "2024-11-13T02:21:40Z"
  name: cf1
  namespace: default
  resourceVersion: "9338008"
  uid: 5693a3ff-ce34-48a3-b33a-27a97c787b8f
```

### ConfigMaps desde archivos

hay que tener en cuenta que no es lo mismo cargar datos que cargar variables de entorno, una manera rapida de cargar un configmap desde un archivo es usando el comando: `kubectl create cm datos-mysql --from-file=datos_mysql.properties`, pero algo curioso que ocurre aqui es que al ejecutar `kubectl get cm` en la columna data aparecera un 1, cuando en el archivo se crearon 4, lo cual implica que no sirve para ser una variable de entorno, pero si para otras cosas

una manera de declararlo dentro de un manifiesto:

```
apiVersion: v1
kind: Pod
metadata:
  name: pod1
spec:
  containers:
    - name: test-container
      image: gcr.io/google-samples/node-hello:1.0
      env:
        # Define las variables de entorno desde un archivo
        - name: DATOS_MYSQL
          valueFrom:
            configMapKeyRef:
              name: datos-mysql
              key: datos_mysql.properties
  restartPolicy: Never
```

### Cargar variables con ConfigMap

agregando la directiva `--from-env-file` le da las capacidades de leer cada variable
- un ejemplo:
`kubectl create configmap datos-mysql-env --from-env-file <file.properties>`

a partir de aqui, a diferencia del `from-file` agrega correctamente las variables de entorno

### ConfigMaps y Volumenes

una caracteristica muy interesante de los configmaps es dejarlos o ponerlos en un determinado sitio dentro del contenedor como si fuesen un archivo, lo que en realidad significa es que hablamos de un volumen, un ejemplo:

## Secrets

ver [secrets.pdf](para mayor informacion)

se crean con el siguiente comando:

`kubectl create secret generic passwords --from-literal=pass-root=devuser --from-literal=pass-usu=kubernetes`

aqui se esta creando un secreto de contraseña generica, con lo cual con las directivas `--from-literal` le asigna un usuario y contraseña, ambas con valor kubernetes

algo interesante es que con la directiva 

```
env:
  - name: MYSQL_ROOT_PASSWORD
    valueFrom:
      secretKeyRef:
        name: passwords
        key: pass-root
```

se le puede asignar el valor de una variable del secret a una variable necesaria del contenedor, basicamente estaria reasignando variables.

otra opcion, mediante archivos

## Secrets mediante archivos

se usa el comando `kubectl create secret generic datos --from-file=datos.txt` esto lo que realizara es que inyectara el contenido de datos dentro del pod, con `kubectl get secret` se obtienen los secretos creados, y finalmente con `kubectl get secret datos -o yaml` se puede apreciar como convierte el texto en una cadena formateada

## secrets de forma declarativa

los secretos tienen generalmente esta estructura:

```
apiVersion: v1
kind: Secret
metadata:
  name: secreto2
type: Opaque
stringData:
  usuario2: 'usu2'
  usu2-pass: 'password-usu2'
```
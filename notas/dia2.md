# ElasticSeach

Orquestador de lucenes.
Queremos tener varios lucenes... muchisimos lucenes para conseguir:
- Alta disponibilidad
- Escalabilidad

Lucene es un indexador / motor de búsqueda, que tiene potencia para trabajar no 
solo con indices normales (directos) sino tambien con indices invertidos.

Podemos cargar documentos en ES dentro de INDICES

> Documentos: SOLO DOCUMENTOS JSON

# Índices

Colección de shards que permiten indexar (opcionalmente también guardar) una colección de documentos.

Dentro de los shards (lucenes) que tenemos en un índices, había 2 tipos:
- primarios:    Escalabilidad
- replicas:     Alta disponibilidad

Nunca ES va a meter 2 shards quivalentes:
Primario A x 2 Replica A1 + Replica A2
Nunca ES meteria de esas 3 2 en el mismo nodo

# Tipos de nodos

ElasticSearch es una herramienta DISTRIBUIDA. 
Repartimos tareas de DISTINTA NATURALEZA que serán procesadas por distintos componentes (NODOS)... además en cluster!

- Maestros: EN UN MOMENTO DADO, SOLO 1 NODO ACTUA COMO MAESTRO!
    - Por un lado, están controlando/"monitorizando" el resto de nodos.
    - Determinan en que nodos se deben ubicar los shards de un índice.
    
    Para Alta disponibilidad como poco 2... Pero con 2 no tengo ya HA? aunque sea un poquito.
        Pero... Quiero evitar el fenómeno llamado BRAIN SPLITTING... y eso me obliga a meter número impar...
        para garantizar que siempre tendré un maestro elegido por MAYORIA ABSOLUTA !
- Datos:
    - Son lo que albergan los lucenes (shards)
      Para Alta disponibilidad como poco 2

- Coordinadores: Este rol no se puede quitar a un nodo. Puedo crear un coordinador PURO si a un nodo le QUITO TODOS LOS DEMAS ROLES
    La misión de los nodos coordiandores es AGRUPAR los resultados de búsqueda antes de devolverlos a quién haya hecho la petición
    Que necesita ese nodo (en cuanto a recursos? CPU-ordenaciones y RAM-para recibir y procesar los datos de todos los servidores !)
    Cómo resolvemos el que los maestros o los datas no hagan esta función... si no la puedo quitar?
    Simplemente MANDANDO PETICIONES SOLO A LOS NODOS QUE ME INTERESE.

    Los coordinadores son los que EXPONGO en BALANCEO DE CARGA para búsquedas.
- Ingesta

    Esto son máquinas (se usan poco) que son capaces de hacer la parte dura de los procesos de ingesta
    Vamos a intentar mandar los datos a ES muy masticados (Logstash)

- Machine learning
    Se usan para aplicar procedimientos de ML... si he pagado la licencia oportuna !

Un cluster puede vivir sin ciertos tipos de nodos... por narices voy a tener al menos nodos de tipo:
- Maestros
- Datos
Lo que hacemos habitualmente es configurar 2 nodos maestros. Y a uno de los nodos de datos le damos POTESTAD para VOTAR al maestro.
En ES esa configuración es un poco especial.

Cuando levantamos un nodo (proceso corriendo de ES... no es una máquina!) le asigno los roles que puede ejercer:
NODO 1: MAESTRO
NODO 2: MAESTRO
NODO 5: MAESTRO
NODO 3: DATOS       , + MAESTRO(pero un maestro de mentirijilla... solo puede votar, no ejercer como)
NODO 4: DATOS       , + MAESTRO(pero un maestro de mentirijilla... solo puede votar, no ejercer como)

El escalado de estos sistemas hoy en día lo hacemos en AUTOMATICO -> Kubernetes (K8S, Openshift, Tamzu)
---
Antiguamente montábamos sistemas / aplciaciones MONOLITICAS... que no iban con esta filosofía.
Esta forma de trabajo (arquitectura de software) está obsoleta.... en favor de arquitecturas DESACOMPLADAS.
La más famosa y usada mayoritariamente a dia de hoy es la ARQUITECTURA DE MICROSERVICIOS.
Esto me permite:
- Escalar de forma individual CADA COMPONENTE
---
Eso nos pasa con TODOS los sistemas que almacenan datos:
- Base de datos:         MariaDB
- Indexador:             ElasticSearch
- Sistema de mensajería: KAFKA, RabbitMQ

Esas herramientas tiene que hablar continuamente entre si.... para tomar decisiones.
Estas herramientas pueden sufrir un problema que llamamos BRAIN SPLITTING -> lobotomia y separaban el lado izquierdo del lado derecho del cerebro.

Servidor MariaDB 1*               < INSERT nuevo USUARIO 'Iván'
    TABLA USUARIOS
        1   Iván 
    |
    Red .... y si pierdo conexión de red .. ambas se pondrían como maestras.,... VUELVE LA RED: Diversión garantizada !
    |
Servidor MariaDB 2*              < INSERT nuevo USUARIO 'Lucas'                 Servidor Maestro
    TABLA USUARIOS
        1   Lucas

Para evitar esto, los servidores eligen al maestro por votación... quorum... Necesitan mayoría absoluta
3 -> Cuantos votos necesito? 2

RED
|--- Servidor 1 *
|--- Servidor 2
|-x- Servidor 3

El hecho de meter nodos impares en sistemas de almacenamiento (MAESTROS) es para evitar el BRAIN SPLITTING
No para conseguir la HA... que con 2 serviría.
---

Tengo 2 discos duros de 5400 vueltas : 1 Tb
Si escribo en cada uno por separado: 100Mb/s

Si tengo 2 discos que puedo montar: RAID -> Presentar esos 2 HDD al SO como uno solo.

- RAID 0: AUMENTO DE CAPACIDAD: 1 HDD de qué capacidad? 2Tb
    Las escrituras se pueden hacer ahora usando las agujas de los 2 discos duros simultaneamente: Velocidad: 200Mb/s
                                                                                -> ESCALABILIDAD
- RAID 1: ESPEJO:               1 HDD de qué capacidad? 1Tb... pierdo 1 Tb      -> ALTA DISPONIBILIDAD
    Cada escritura se hace en los 2 discos... Velocidad escritura: 100Mb

- RAID 10: 4 discos de 1Tb, los presento como 1 solo de 2Tb
    -> ESCALABILIDAD            (más rendimiento)
    -> ALTA DISPONIBILIDAD      (tolerancia a fallas)

2 discos... copiados entre si... si uno se rompe, que pasa con mi HA? No.

---

cluster Servidores web:
- Servidor 1 - 50% CPU, memoria
- Servidor 2 - 50% CPU, memoria ... si este cae... qué pasa? Si este cae, el otro cae. 


---
DATA 1
    A0' A1 A2'' A3' A4 A5

DATA 2
    A0 A1' A2' A3 A4' A5'

DATA 3
    A0'' A1'' A2 A3'' A4'' A5''

Indice A con 6 shards primarios x 3 replicas 

A0 A1 A2 A3 A4 A5
A0' A1' A2' A3' A4' A5'
A0'' A1'' A2'' A3'' A4'' A5''
A0''' A1''' A2''' A3''' A4''' A5''' Estos shards iban a quedar DESASIGNADOS !
                                    Y el cluster quedaría DEGRADADO !           

En ES el estado del cluster se me representa en 3 colores:
- VERDE - OK                    Estamos cumpliendo con objetivos definidos
- AMARILLO - DEGRADADO          Estoy operativo... pero estás en la cuerda floja... ya que no puedo garantizar tus requerimientos
- ROJO - NOK
 

---
DATA 1
                                                                                A0  A1' B0  B1' CAPUT !
DATA 2
        A0'   A1'     B0'       B1'
DATA 3   v     ^       v         ^      Voy a mover 800 Mbs de una máquina a otra. Los HDD los peto... La red la peto!
        A0    A1      B0        B1

Indice A con 2 shards x 1 replicas
Indice B con 2 shards x 1 replicas


Si a ES se le ocurre la brillante idea de ponerse a copiar los shards... Adios cluster !

Aquí habrá confioguraciones en ES... Ese proceso le puedo pedir a ES que lo haga... pero con un DELAY importante.
5 minutos sigues sin conexión... empieza a copiar.

---
Donde va a estar corriendo el elastic? En un conjunto de máquinas FISICAS? Poco habitual

O lo tengo en MAQUINAS VIRTUALES... que sigue siendo raro ya a día de hoy
O lo tengo en CONTENEDORES... gestionados por un KUBERNETES

                                                                                SISTEMA DE ALMACENAMIENTO EXTERNO AL CLUSTER x3
MAQUINA FISICA KUBERNETES 1                                                     Cábina
    ES-Data 1                                                                   NFS
MAQUINA FISICA KUBERNETES 2* CATAPLOF                                           Cloud
MAQUINA FISICA KUBERNETES 3
    ES-Data 2
MAQUINA FISICA KUBERNETES 4
    ES-Master 1
    ES-Data 3 (Donde están los datos? en la máquina física? NO)
MAQUINA FISICA KUBERNETES 5
    ES-Coordinador1
...
MAQUINA FISICA KUBERNETES N
    ES-Master 2
    ES-Coordinador2


Felipe ! Dpto BI > quiero hacer una búsqueda en ES
Kibana !

    A Felipe o a Kibana les doy la dirección de un Balanceador de carga ! Y ese balanceador apuntará a unos nodos del cluster... 
    a cuales? a los SOLO coordinadores
    
---

# Contenedor

Proceso aislado donde corre una aplicación                                      Nasti de plasti.. un contenedor puede ejecutar montonoes de procesos.
Contiene el código del programa, bibliotecas, configuración ...                 VERDAD.... realmente quien contiene el código es la IMAGEN DEL CONTENEDOR
Tiene un Sistema operativo... es mucho más ligero                               ES IMPOSIBLE MONTAR UN SO dentro de un contenedor.
Facilmente compartible                                                          NI DE BROMA. Lo que si compartiomos fáclmente son las IMAGENES DE CONTENEDOR
Se puede ejecutar en cualquier SO?                                              NO... solo en LINUX .

Entorno aislado dentro de un SO con kernel Linux donde ejecutar procesos.
AISLADO:
- Su propia configuración de red: -> sus propias IP
- Su propio FileSystem (sistema de archivos propio)
- Sus propias variables de entorno
- Puede tener limitaciones de acceso a recursos HW

Los contenedores los creamos desde imágenes de contenedor.

# IMAGEN DE CONTENEDOR

Un triste fichero comprimido (tar) que contiene una estructura de carpetas según
el estandar POSIX( /bin /var /etc/ tmp). Eso ni siquiera es obligatorio... pero habitual.

Dentro de esas carpetas encontramos INSTALADOS programas... montones de programas.
Y sus correspondientes ficheros de configuración.

Además, una imagen de contenedor se acompaña con algunos METADATOS:
- Que comando es el que debe ejecutarse cuando un contenedor creado desde esa imagen de contenedor sea iniciado
- Qué puertos son los que están levantando los procesos que ahí dentro corren por defecto
- En qué carpetas esos procesos van a guardar su información.

Las imágenes de contenedor las encontramos en REGISTRIES DE REPOS DE IMAGENES DE CONTENEDOR:
- Docker hub
- Quay.io           > Redhat

# Gestores de contenedores:
- Docker
- Podman (REDHAT)
- Containerd
- CRIO

# Sistema de archivos de un contenedor

Se crea mediante la superposición de CAPAS

VOLUMENES . Capas adicionales
    /var/log                -> HOST: /datos/logs/contenedor/nginx1
                            -> CLOUD AWS: VolId: 18274528273534782746
CAPA DEL CONTENEDOR. Capa 1
CAPA BASE. Capa de nivel 0. Capa de la imagen < Esta capa es INMUTABLE

HOST
/
    bin/
    tmp/
    var/    DATOS
        lib/
            docker/
                container/...
                            minginx1/
                                    /var/log/nginx.log
                images/...
                        nginx/  <<<< . Esta es la que ofrecemos a los procesos que corren dentro del contenedor como carpeta RAIZ. chroot
                                                                                                                                    change root
                                bin/
                                    ls
                                    mv
                                    cp
                                    mkdir
                                    chmod
                                tmp/
                                var/
                                    www/index.html
                                etc/    
                                    nginx/nginx.conf
                                usr/
                                    bin/
                                        nginx
                                opt/
                                root/
                                home/                            
    etc/    CONFIGRUACIONES
    root/
    home/
    
USOS DE LOS VOLUMENES AL TRABAJAR CON CONTENEDORES:
- Persistencia de datos tras el borrado de un contenedor
- Compartir datos entre contenedores
- Inyectar ficheros de configuración
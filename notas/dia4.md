Si tengo una máquina/Contenedor para el ES
y esa máquina tiene 16Gbs... cúanta memoria le pongo al ES (JAVA)?
50-60% 8-10Gbs -> ES

Por qué? 
Buffers de IO gestionados a nivel de SO.

---

ElastiSearch lo que ofrece es un API REST
No tiene interfaz gráfica en ES.

Kibana es una Interfaz gráfica WEB.... y nos deja hacer algunas operaciones contra ES.
Básicamente operaciones de gestión/administración y de búsqueda de datos.

Cargar datos, crear indices, crear plantillas de indices.... TODO ESTO NO me da formularios para hacerlo KIBANA.

---

API REST? 

API? Application Programming Interface

A todos los programas les hacemos peticiones... Para hacer esas peticiones necesito invocar unas:
- funciones
- comandos
- procedimientos
- botones

Los programas ofrecen una INTERFAZ de comunicación con ellos:
- INTERFAZ GRAFICA
- INTERFAZ de linea de comandos
- Otro tipo de interfaces: API REST

Interfaz REST, se basa en peticiones que hacemos por protocolo HTTP

---

World Wide Web?  WEB? 
Un servicio que funciona sobre INTERNET, básado en:
PROTOCOLO HTTP y transmisión de ficheros HTML
Lo monta Tim Berners-Lee. 199?  -> Funda el W3C
    Definir HTTP y HTML
    HyperText : Textos que tenían vinculos (enlaces) entre si.
    
    Los ficheros HTML van orientados a su visualización por seres humanos.
    Los HTML tienen dentro MARCAS: 
        Encabezados de distintos niveles: H1... H6
        Table
        Lista ordenada
        Imagen
        Negrita
        Parrafo
        
---

Arquitectura Cliente-Servidor

Clientes                                
 COMPUTADORA                                            SERVIDOR
 Programas                                              Programas
    
 Representar                                            Gestión y persistencia de una DATOS gestionados por un programa
 Hacer peticiones para que los datos se modifiquen
 
 
 Para hacer comunicaciones entre cliente y servidor necesitamos:
    - Emisor:   Cliente  / Servidor
    - Receptor: Servidor / Cliente
    - Canal:    Red
    - Mensaje:  DATOS < - Lenguaje (idioma)
    - Protocolo: Conjunto de normas de reglas para que la comunicación se pueda llevar a cabo.
                 Puedo incluso tener distintos protocolos que se superpongan entre si.
                 
Persona                         HTTP + HTML
    Navegador   ------------------ quiero un muñeco de Naruto ------->
    
                                                                        Programa 1
Programa de UPS                                                        Servidor de amazon
            SEUR -- quero saber qué paquetes hemos de entregar hoy -->  Programa 2 < - SERVICIO WEB
                        HTML? XML, JSON, YAML, CSV (Excel) Lenguaje neutro
                        Protocolo? HTTP
                                        SOAP -> XML, WSDL, UDDI
                                        Restricciones en el uso de HTTP: REST -> JSON
                                        
---
# Entendeis como funciona el protocolo HTTP? 

Se base en el concepto de REQUEST -> RESPONSE 
No tengo comunicación BIDIRECCIONAL pura
CLIENTE emite un REQUEST (Petición)
El SERVIDOR emite un RESPONSE(respues) a un REQUEST concreto
Cada rquest lleva asociado su RESPONSE

Tanto al hacer el resquest, como al hacer el response, tengo opción de mandar datos: BODY REQUEST | BODY RESPONSE
Esos datos irán en un determinado lenguaje... el que sea. Originalmente HTML... hoy en día.. lo que me de la gana.

Esos datos, se meten en una caja, como cuando AMAZON me manda el muñeco del Naruto.
Por fuera de la caja, se pone una PEGATINA, con metadatos (datos accesorios a los adtos principales que mando)

Esos metadatos en HTTP se denominan HEADERS

Al hacer un REQUEST hay 2 metadatos importantes:
 METODO: Lo que quiero que hagas con eso. HTTP define más de 20 metodos. EN REST usamos básicamente 5:
    GET     Que me dé algo
    POST    Para que guarde algo en el servidor (algo nuevo)... A veces éste y el de abajo ... se mezclan un poco.
    PUT     Para que modifique algo en el servidor
    DELETE  Para que algo se borre en el servidor
    HEAD    Para saber si algo existe en el servidor
 Tipo de Contenido: (lo que hay dentro de la caja): JSON, PNG, MPEG, HTML
 
Hay veces que tanto en un REQUEST como en un RESPONSE no mando DATOS... solo metadatos.
 
El servidor contesta a ese request con un response... que puede incluir datos... e incluirá siempre METADATOS:
 CODIGO DE RESPUESTA
    2XX Que ha ido bien
    3XX Redirección
    4XX Errores de cliente
    5XX Errores de servidor
 Tipo de Contenido: (lo que hay dentro de la caja): JSON, PNG, MPEG, HTML

Adicionalmente cuando hago un REQUEST lo hago contra una URL

URL:    Una URL es una dirección que identifica un RECURSO UNICO (en nuestro caso - REST) un servicio

        protocolo  ://  servidor        :   puerto      Contexto/EndPoint
        http            dominio (DNS)       80
        https                               443
                                                        La ruta a la que accedo dentro del servidor -> 
                                                            /index.html
                                                            
                                                            / La RAIZ del sistema de archivos
                                                        También, hoy en día, me permite identificar un PROGRAMA
                                                        
                                                            /login.php  -> Programa que tengo en HDD
                                                            /login.jsp
                                                            /login
Apache:
    Document ROOT: /var/www/site1/
                                  index.html
                                  images/logo.png
    
    ^^^^
    
    http://Apache:80/images/logo.png
    
    
    /var/www/site1/index.html
    
    
---
Cluster:

RED DOCKER          COMUNICACIONES EXTERNAS             COMUNICACIONES INTERNAS ENTRE LOS NODOS
|- Maestro1               9200                              9300
|- Maestro2
|- Datos1
|- Datos2
|- Datos3
|- Coordinador1           9200 > Mapeo a nivel del host 9200
|- Coordinador2           9200 > Mapeo a nivel del host 9201



data

data_content
    Abiertos en los que ademas de APPEND hacemos UPDATE -> Uso ES como indexador de documentos (gestor documental)
data_hot
    Abiertos son en los que hacemos APPEND en el índice -> LOGS
    Congelados de alta consulta
    NVME + SSD
data_warm
    Congelados de baja consulta
data_cold
    Cerrados
    
---
En ES, cuando hacemos una búsqueda, los resultado se ordenan en base a una puntuación que se genera para cada documento
El valor máximo de esa puntuación a proiri NO ESTA LIMITADO.

Dame los documentos del indice USUARIOS en los que el nombre sea Iván o la edad 33

nombre Ivan y edad 33 PUNTUACION MAS ALTA
nombre Ivan           PUNTUACION MEDIA
edad 33               PUNTUACION MEDIA

---

<p>Hola <strong>amigo</strong></p>
<p>Hola mi amigo strong</p>

Hola amigo

strong

---

data1
    A0  A1  A2  A3                          A4'                 A8'
data2
    A4  A5  A6  A7      A0'     A1'                             A8''
data3
    A8  A9              A0''    A1''        A4''

INDICE A 10 shards x 3 replicas

---

APACHE              
    V
    access_log
        ^
        Filebeat    >   Logstash    >   ElasticSearch   < Kibana
        
        Cada linea del fichero de log mandarla a Logstash 
                        
                        Procesar esa linea para mandarla a ElasticSearch
                        
                        
---

Apache: 
 Contenedor
 Maquina

FileBeat: Cada Apache va a llevar su filebeat
    Si Apache está en un contenedor, filebeat lo metere en un contenedor anexo (KUBERNETES -> Apache y FileBeat van en el mismo POD)
    Si Apache fuera montado a nivel de host: Filebeat podría montarlo a nivel de host o de contenedor
    
    
Apache1 -> access_log1 < - filebeat1 ---> Logstash
Apache2 -> access_log2 < - filebeat2 -------^


---

clusters activo-activo      ha y escalabilidad
clusters activo-pasivo      ha
    DOS NODOS, uno levantado y otro abajo... a la espera
    

logstash


TODO ESTO AL FINAL VA A UN KUBERNETES

50 apaches - 50 filebeats ---x-> logstash? En general con 1 nos da pa mucho
                                   ->        logstash1
                                BC ->        logstash2
                                
                                            1 replica HA en Kubernetes?
O acumulo mensajes
O borro los viejos - 2 ROTADOs de 50kb
                    100Kbs -> Memoria RAM



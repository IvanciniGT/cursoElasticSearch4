- Instalar un cluster más serio de ES
- Dar una vuelta a Lucene => Operaciones de gestión de índices que hacemos en ES
    - Merge indice
    - Freeze

---

# Índice de ES es:

Coleccion de shards (lucenes) donde puedo indexar, y buscar documentos JSON

## Qué ocurre cuando cargo un conjunto de documentos JSON en un Indice

    INDICE: usuarios
    Usuario: 1
    {
        "nombre": "Ivan",
        "apellidos": "Osuna Ayuste",
        "email": "ivan.osuna.ayuste@gmail.com",
        "edad": 44
    }
    
1.  Lo primero será el asignar a este documento un IDENTIFICADOR.
    Al cargar un documento, podemos nosotros DARLE un identificador.
        Esto tiene sentido cuando indexo documentos que se gestionan por fuera de ES.
            Que saco de una BBDD
            Páginas web que tengo
            Documentos en unas carpetas de un HDD
    También puedo dejar a ES que le asigne un ID en AUTOMATICO.
        Esto tiene sentido cuando ES es quién me va a gestionar el documento... pero
            ES no es ni una BBDD ni un gestor documental, ni nada de eso.
            No tiene esas capacidades.
            SOLO cuando trabajo con documentos que no requieren GESTION, más allá de su almacenamiento: LOGs
    
2.  ES debe determinar en qué SHARD se guarda el documento... ya que un índice puede tener muchos SHARDS
    Eso se hace mediante un algoritmo de ROUTING.
    Por defecto, cuando llega un docuemnto, se mira su ID (el suministrado o el autogenerado)
    Ese ID se transforma en un número -> Aplicar un algoritmo de HASH (huella) SHA, MD5
    Ese número, elastic lo divide entre el número de SHARD primarios que tiene el INDICE...
    Y también se queda con el RESTO... 0... número de shards primarios -1 = NUMERO DE SHARD AL QUE VA EL DOCUMENTO

        0-> lo meto en el shard 0    
        7-> lo meto en el shard 7
    
    Servidores en distintos Centros de datos.
    Quiero usar como campo para el algoritmo de Enrutamiento el CENTRO DE DATOS (Madrid, Barcelona, Sevilla)
    Esto me asegura que? Todos los documentos que vengan del mismo centro de datos se guardan en el mismo SHARD
    Y las búsquedas, cuando las haga por centro de datos, van a ir como un tiro!

3.  Le pido al Lucene que lo indexe...
    Y aquí viene fiesta : ->  Segmentos (Ficheros en los que Lucene va guardando los índices (directos e inversos))


    DOCUMENTO con id 1:
    {
        "nombre": "Ivan",
        "apellidos": "Osuna Ayuste",
        "email": "ivan.osuna.ayuste@gmail.com",
        "edad": 44
    }
    
    INDICE nombre:

    ivan      1
    
    INDICE apellidos:

    osuna    1(1)
    ayuste   1(2)
    
    INDICE edad:

    44       1
    
    INDICE email:

    ivan.osuna.ayuste@gmail.com       1
    
    ¿Y esa información, dónde se genera? Esa información se genera en la MEMORIA RAM de la computadora
    Pero... me sirve con qué esté ahí?   Es necesario PERSISTIRLA (llevarla al HDD)
    
    Lucene guarda esa información en unos ficheros en el HDD. Qué estructura genera LUCENE para esos ficheros.
    
    Una cosa es en la memoria RAM... Ahí según le llegan datos nuevos, tiene flexibilidad para ponerlos donde tocan.
    El problema es al llevarlos al HDD... No puede meter esos datos en medio de un fichero
    
    Lucene directamente los datos nuevos que le llegan los va añadiendo al final del fichero.
    Con lo cual... en la memoria los datos están ORDENADOS... pero en el fichero NO
    
    Esos ficheros que va generando se llaman SEGMENTOS
    
    Si los datos no están en memoria, Lucene debe leer los datos del HDD.
    Siempre que se hace una búsqueda, el INDICE de lucene debe estar en MEMORIA... y el proceso de cargar los datos
    de un fichero a memoria puede ser tedioso. (tardar un huevo)
    
    Cuando un índice no está en uso... y lucene necesita espacio de memoria, Lucen va a eliminar ese indice de la memoria.
    Ya lo tiene en un ficheor para cuando necesite cargarlo.
    
    Cuanta más RAM tenga, más indices entran en memoria... pero... tendré un límite siempre
    
    En un momento dado, me puede interesa reorganizar esos ficheros (SEGMENTOS) para dejarlos más sencillos de leer.
    ESTA OPERACION EN ES SE DENOMINA MERGE ! Agrupar ficheros de SEGMENTOS, y reorganizar los datos que contienen
    para que sean más facilmente accesibles (llevarlos a la RAM más rapido)
    Es sencilla o es pesada? PESADA... Me obliga a reescribir todos los ficheros de segmentos.
        ¿Cuando me interesa hacerlo? 
            Cuando no tenga actividad... por las noches. Esto podría hacerlo...
                Si es que tengo un sistema que descanse.... WEB
                Pero aún así... cuanto me serviría ese trabajo? HAsta que hubiera metido muchos más datos.
            Otra opción es cuando tenga ASEGURADO que ya no me van a entrar más datos.
                Si he generado indices para rangos de fechas: INDICE_ENERO .
                El dia 1 de febrero es indice ya no se toca... en la vida.
                    Es el momento ideal para hacer ese trabajo.

    OPEN            Puedo hacer búsquedas y actualizaciones de datos (LECTURA y ESCRITURA)
    CLOSE           No se usa el índice para nada (Ni lectura, ni escritura)
                        Me interesa guardar los datos... para por si acaso en el futuro...
    FREEZE          Puedo hacer búsquedas... pero no meter datos nuevos.
                        Esto es lo que hacemos en cuanto pasa el periodo de tiempo para el que hemos creado el indice.
    DESCONGELAR     Pues lo contrario... dejarlo abierto
    MERGE           Agrupar y optimizar los ficheros de segmento.
    REFRESH         Lucene va procesando los documentos que le llegan en RAM (generando las entradas que tiene que incluir en los indices)
                            vvv REFRESH (permite que los datos de los nuevos documentos estén disponibles para búsquedas YA)
                                        Esto está configurado por defecto en el orden de decimas de segundo.
                                        Si no me es suficiente, puedo formarlo para que se haga más rápido.
                                        Realmente el uso de esta operación está limitada a casos muy excepcionales (vinculados a cargas masivas)
                    Acto seguido, añade esas entradas en el INDICE completo que tiene en RAM
                            vvv FLUSH
                    GUARDAR LAS ENTRADAS EN EL HDD al final del archivo de segmento actual
    FLUSH
    SPLIT           DIVIDIR los shards primarios de un índice
                    Si tengo un indice que he creado con 4 shards, puedo pasar a 8 shards... pero nunca podría pasar a 5 shards
    SHRINK          Agrupar shards primarios de un indice De 4 paso a 2.... eso si, nunca a 3

---

Trabajando con ficheros. 2 formas de trabajar con ficheros en una computadora. Cuando abro un archivo a nivel de SO,
sobre ese archivo puedo hacer 3 operaciones referidas a la escritura:
- Añadir cosas al final del archivo (append)
- Reescribir un archivo completo
- Acceso aleatorio al archivo (poner la aguja del HDD en una determinada posición del archivo) y escribir ahí,
    machacando lo que hubiera. <<<<< ES LA QUE USAN TODAS LAS BBDD


        Nombres                 Apellidos           Edad
    1|  Ivan.                |   Osuna           |  45                   |                   |
    2|  Felipe               |                   |  22                   |                   |
    3|  eric                 |                   |                       |                   |
    4|                       |                   |                       |                   |
    5|                       |                   |                       |                   |
    6|                       |                   |                       |                   |










---

Algoritmos de HASH:

Todos los dias desde que teneis 14-15 años.

Letra del DNI = Algoritmo de huella.
99.999.999 - LETRA 
La letra es una HUELLA del número.
Cada número genera una HUELLA, siempre la misma, al aplciarle un determinado procedimiento(transformación, algoritmo)
- Tomo el número y lo divido entre 23
    23.000.001 | 23
     0 0       ----------
                1.000.000 < Este número, me la pela
             1 <<<< ESTE EL QUE ME IMPORTA 
                    Este número estará entre 0 y 22
- El ministerio de interior tiene una tabla asignando a cada resto posible una letra.
    ->   Este DNI lleva la letra R



Tengo cargados datos de los log de un apache:
- Codigo de respuesta
- url
- servidor
- agente (el programa que hizo la peticion)
- so del cliente
- ip

Cúantos shards me interesa tener en ese índice?
De qué depende? De cuantos datos me estén entrando... y la cantidad de trabajadores que necesite para procesarlos

En general, me interesa valores altos o bajos? 
    - Para inserción: ALTOS
    - Para consulta? Si recupero los datos de uno en uno: ME DA IGUAL... aunque esta operación es POCO o NADA habitual en ES
                     Si quiero hacer una búsqueda? BAJOS. Necesito consolidar menos información GANANCIA
                                                   ALTOS. Voy a tener muchos tios leyendo los datos a la vez
                        
                        Búsqueda de los erores 505 de mi sitio web - 10M de accesos.. guardaditos en unos ficheros de mi HDD
                            El lucene si no está usando un indice, lo quita de la RAM a la mínima que me descuido.
                        10 shards... Y por ende 10 ficheros de 1M de accesos... que soy capaz de leer en paralelo.

    El tener más replicas, me da más sitios donde buscar... a coste de repetir y reptir y repetir los datos en muchos HDD = $$$
                    -> Me dan HA
    El tener más primarios, también me da más sitios donde buscar... sin tener que repetir y repetir y repetir lso datos en muchos HDD = MAS BARATO

Pero esto se complica aún más.

Imaginad que tengo un indice con un shard... y ese shard ocupa                  10   Gbs
Si lo divido en 2 shards... cuanto ocupará cada shard?                           6-7 Gbs
    A tomar por culo!
    2 Gbs más x 2 = 4 Gbs más. es decir un 40% más de espacio... más replicas...
        4 Gbs más en HDD... pero 4 Gbs más en RAM en servidores para cargar el shard
        
    Y por qué?
        En el shard (el lucene), no guarda solo las UBICACIONES de los datos... TAMBIEN GUARDA los términos
        
        
    Entrada 1 de un log de un servidor apache. Codigo respuesta: 200
    Y como ese tengo 50.
    
        Lo tengo en un shard:
            TERMINOS:       UBICACIONES
                200         1, 2, 3, 4, 5, 6, 7, 8, 9, 10
                            11, 12, 13, 14, 15, 16, 17, 18, 19, 20
                201         21
                404         22
                500         23
        
        Y lo paso a 2 shards
            
            SHARD1:
                            TERMINOS:       UBICACIONES
                                200         1, 2, 3, 4, 5, 6, 7, 8, 9, 10.... 1M
                                404         22
                                500         23
            SHARD2:
                            TERMINOS:       UBICACIONES
                                200         11, 12, 13, 14, 15, 16, 17, 18, 19, 20... 1M
                                201         21
    
            Que he guardo ahora duplciado? "200"
            Si tengo pocos términos que se repiten mucho... No hay mucho problema
            
            Si en lugar de eso guardo DNIs de personas que han comprado algo en Amazon.
            
            SHARD1:
                11111111A:  1, 2 ....??? Pocos datos
                ..
                ..
                ..
                Me dio millon de DNIs
            SHARD2
                11111111A:  3, 4 ....??? Pocos datos
                Me dio millon de DNIs pocos datos.
                
            Que me pasa aquí cuando divido el shard? Que la mayoría de los DNIs se me duplcian.
            Me ocupan más en el índice LOS TERMINOS que las ubicaciones.
            
            Si los términos ocupan mucho dentro del índice... cuantos shards me interesan? pocos.. si creo muchos empiezo a duplicar muchos datos..
                A no ser que cambie el arlgoritmo de routing... para que todos las compras del mismo DNIs se guarden juntas en el mismo shard...
                Claro que eso me sirve para los terminos del DNI... pero no para los códigos de los productos... que serán los que ahora se repitan
            Si los términos ocupan poco.... (porcentualmente), me da igual tener muchos shards.                

Los shards deben estar entre 10 - 50 Gbs
En cada shard solo hay un hilo haciendo simultaneamente busquedas o actualizaciones

---

Igual que las BBDD, ElasticSearch cúanta memoria RAM quiere usar a priori?      INFINITA !
A priori, querrá tener todos los shards en RAM.
Y las búsquedas que haga, las cachea, para la siguiente vez, devolverlas más rápidas
Y los planes de ejecución de esas búsquedas, los cachea.

Y eso hay que limitarlo.
---
Pero aquí hay un PROBLEMA. ElasticSearch está desarrollado en JAVA !
JAVA es horrendo controlando la memoria RAM. 

    String texto = "HOLA";
    
    Crear un dato de tipo String(TEXTO) en memoria RAM con el valor HOLA

    texto = "ADIOS";
    
    Crea otro datos de tipo String(TEXTO) en otro sitio de la memoria RAM, con el valor "ADIOS"
    Y hace que la variable "texto" que antes apuntaba al valor HOLA ahora apunte al valor ADIOS.
    
    Llegados a este punto, cúantas cosas hay en la RAM? 2 cosas: HOLA y el ADIOS
    Pero el HOLA, ya no tiene niguna variable que le apunte... SE HA CONVERTIDO EN BASURA... ocupando espacio de la memoria RAM.
    
    Esto no es ni bueno ni malo... ES UN FEATURE (una característica de diseño de JAVA)
    Quiero decir... ya teniamos un lenguaje MUY SIMIIAR a java que gestionaba la memoria RAM dpm, (C++)
    
    Y eso por qué? porque gestionar la RAM bien, es muy complicado... Y los desarrolladores le tiene que meter horas por un tubo.
    Programar en C o C++ es mucho más complejo (costoso) que en JAVA
     
    Y en JAVA lo que tenemos es lo que llamamos un RECOLECTOR DE BASURA... que de vez en cuando, especialmente cuando no queda memoria
    la librera (borra toda la basura)
---

Java es un lenguaje interpretado ".java" -> ".class" -> Son interpretados, para que se puedan ejecutar. Eso se hace en tiempo real: JIT

Ese JIT, junto con el resto de cositas que necesito para ejecutar un programa JAVA, se encuentran dentro de una MAQUINA VIRTUAL

java | java.exe = JVM (JRE)
                    Java Virtual Machine -> Java Runtime Environment
                    
Cuando ejecuto ese programa levanto una máquina virtual... como cuando la levanto con VirtualBox, Citrix, VMWare, KVM, HyperV

Y al maquina virtual, a nivel de HOST le tengo que configurar: Recursos a los que PUEDE TENER ACCESO (entre ellos, la memoria)
Pero cuando arranco la maquina virtual, cuánta memoria se usa realmente? Según quien?
- Según el host? la que haya pedido
- Según la MV? dependiendo de los programas y datos que tenga dentro.

No me vale con monitorizar el host (al menos solamente) .... necesito monitorizar la memoria de dentro de la maquina virtual:
En java, una parte de esa memoria es lo que llamamos el HEAP, que es la gorda... y la que guarda los datos.

Una cosa es que a la MV le haya dado 10 Gbs
Y otra cosa es que mi programa (ElasticSearch) esté usando los 10Gbs... que a lo mejor solo usa 5 Gbs... en cuyo caso? 
    ESTOY HACIENDO EL PRINGAO ! Estoy tirando 5 Gbs
    
No os preocupeis que con Elastic Eso no pasa... se lo come to'
Lo que me interesa es poner la cantidad de memoria adecuada.... además de pasta (si pongo más memoria de la cuenta) 
esto impacta DIRECTAMENTE en el rendimiento!

La memoria RAM (HEAP) se usa para varias cosas:
- Para tener datos de acceso rápido (que no necesite ir a disco)... CACHE ! 
        Estos datos ... no son vitales.
        Si no los tengo, el sistema sigue en funcionamiento... aunque más lento
- Para tener los datos de trabajo.
        Estos datos tienen que entrar si o si. VA EN FUNCION DE LA CARGA DE TRABAJO !




RED INTERNA  de DOCKER              ALGUIEN DE FUERA DE LA RED(kubernetes)
|
|- 172.18.0.2 maestro1
|       9200 < https
|       9300 < https
|- 172.18.0.3 maestro2
|       9200
|       9300
|- 172.18.0.4 datos1
|       9200
|       9300
|- 172.18.0.5 datos2
|       9200
|       9300
|- 172.18.0.6 datos3
|       9200
|       9300
|- 172.18.0.7 coordinador1 *
|       9200                        > 9200 (del host)
|       9300
|- 172.18.0.8 coordinador2 *
|       9200                        > 9201 (del host)
|       9300
|
|- 172.18.0.9 Kibana
|       5601                        > 8080 (del host) http < https
|           https://coordinador1
|
|
|

# ElasticSearch

Un producto de software de tipo: MOTOR DE BÚSQUEDA / INDEXADOR

Desde ya:
- ElasticSearch NO ES UNA HERRAMIENTA DE MONITORIZACION!
- ElasticSearch NO ES UNA BASE DE DATOS

Ahora bien: 
- Permite hacer búsquedas basada en INDICES invertidas
- Permite hacer búsquedas muy potentes y eficientes
- Permite montar un sistema de monitorización, cuando lo juntamos con otras herramientas de software que fabrica 
  la empresa ELASTIC inc: Kibana, Logstash, Beats: 
  El stack ELK < Entre otras cosas sirve para montar un sistema de monitorización.
- Me permite crear un repositorio centralizado de logs
- En según que circustancias, me sirve como motor de búsqueda de Bases de datos

Por resumirlo mejor. Si tuviera que elegir otra herramienta equivalente a ES: GOOGLE!


## MOTOR DE BÚSQUEDA / INDEXADOR

Me permite:
- Indexar información: Guardar información dentro de INDICE
- Hacer búsquedas que aprovechen la potencia de esos índices

## INDICE

Llevais usando índices desde que teneís 8 años - 9 años. En cualquier libro de texto.
Entiendo un índice exactamente igual que en un libro... qué es un índice?
Es un conjunto ORDENADO de datos, donde asociado a cada dato tengo su UBICACION !
- Capitulo1: Cómo freir un huevo .........pag 17
- Capitulo2: Cómo hacer pan ..............pag 34

Para qué nos sirven los índices ... en un libro mismo?

Localizar un dato de forma eficiente.

---

## Bases de datos relacionales (tradicionales)

En las bases de datos relacionales, guardamos la información en TABLAS: (una tabla EXCEL de datos).
Dentro de una tabla tenemos: FILAS(registros), COLUMNAS(campos)

### RECETAS_DE_COCINA 

| Titulo             | Dificultad   | Tipo de plato | ID |
| ------------------ | ------------ | ------------- | -- |
| Paella             | Media        | principal     |  1 |
| Lentejas estofadas | Baja         | primero       |  2 |
| Pasta carbonara    | Media        | primero       |  3 |
| Pasta boloñesa     | Media        | primero       |  4 |
| Lasaña boloñesa    | Baja         | segundo       |  5 |
| Pizza carbonara    | Baja         | pizza         |  6 |
| Bacalao al pil pil | Alta.        | principal     |  7 |

En las bases de datos, cómo se llamaría a la operación que debe hacer la BBDD para realizar en esa tabla 
una búsqueda del tipo: Dame las recetas de platos de tipo "primero"? FULL SCAN

Es eficiente un FULL SCAN? No es nada eficiente.... Es la forma menos EFICIENTE que tenemos de buscar los datos.
Ya que me obliga a ir DATO A DATO.

Se os ocurre cómo podríamos mejorar esa búsqueda?
Si ordenamos los datos.... podemos aplicar un ALGORITMO DE BUSQUEDA más eficiente: BUSQUEDA BINARIA !

Usamos un algoritmo de búsqueda binaria también desde que tenemos 10 años: 
Es el que usamos cada vez que abrimos un DICCIONARIO !
En los diccionarios tenemos los términos ORDENADOS ALFABETICAMENTE.
Cuando quiero buscar algo, qué hago? 
Abro el diccionario a la mitad.... y miro que letra hay en esa página:

- amarillo
- **blanco**   <--- Abro el diccionario por aquí la tercera vez: ENCONTRADO!
- calendadrio  <--- Abro el diccionario por aquí la segunda vez
- ~~melón~~
- rama  <---------- Abro el diccionario por aquí la primera vez
- ~~sal~~
- ~~zapato~~

Al aplicar una búsqueda binaria, con solo 3 intentos, he encontrado la palabra-

1.000.000
  500.000
  250.000
  125.000
   63.000
   32.000
   16.000
    8.000
    4.000
    2.000
    1.000
      500
      250
      125
       63
       32
       16
        8
        4
        2
        1
En 21 operaciones he encontrado un dato entre 1 millón de elementos. ESTO ES EFICIENTE!

Realmente, si tengo que buscar la palabra BLANCO en un diccionario no abro a la mitad.... Ya se que BLANCO
aparece al principio (la B)... Realmente la primera apertura no me discrimina la mitad de las palabras... sino muchas más.

Eso lo puedo hacer, porque conozco en mi idioma, cómo se distribuyen las palabras.
Esto también lo hacen las BBDD ---> ESTADISTICAS DE LA BBDD

Claro... esto está muy bonito en un diccionario.... que tiene los datos PREORDENADOS....
Y si no tengo los datos preordenados,
me puede interesa ORDENAR los datos antes de hacer una búsqueda para hacerla más eficiente?

Qué tal se le da a un ordenador, ORDENAR datos? Es de las peores cosas que le puedo pedir a una COMPUTADORA.
Las computadoras son MUY INEFICIENTES ORDENANDO DATOS.

Me sale más caro (computacionalmente) ORDENAR los datos que hacer un FULLSCAN!

Lo que me podría interesar es tener los datos PREORDENADOS... igual que en un diccionario.

### Problema 1

Si guardo los datos preordenados... cuantos criterios de ORDENACION puedo tener? 

| Titulo             | Dificultad   | Tipo de plato |
| ------------------ | ------------ | ------------- |
| Pizza carbonara    | Baja         | pizza         |
| Lentejas estofadas | Baja         | primero       |
| Pasta carbonara    | Media        | primero       |
| Pasta boloñesa     | Media        | primero       |
| Paella             | Media        | principal     |
| Lasaña boloñesa    | Baja         | segundo       |

Esa ordenación de arriba está guay para buscar por TIPO... pero ya no me vale para buscar por dificultad.

### Problema 2

He decidido apostar todo a un campo.. y dejar los datos ordenados por ese campo: DIFICULTAD

| Titulo             | Dificultad   | Tipo de plato | 
| ------------------ | ------------ | ------------- |
| Pizza carbonara    | Baja         | pizza         |   
| Lentejas estofadas | Baja         | primero       |
| Lasaña boloñesa    | Baja         | segundo       |
| Pasta carbonara    | Media        | primero       |
| Pasta boloñesa     | Media        | primero       |
| Paella             | Media        | principal     |

Me es facil mantener el orden cuando lleguen datos nuevos?

Qué pasa si ahora quiero meter:

| Pollo asado        | Baja         | principal     |

Qué impacto tendría hacer esto (el "INSERTAR" ese datos por ahí en medio)?
Pongamos que lo quiero meter en la posición que menos impacto tenga: POSICION 4
El problema es que tengo que mover las de abajo.

| Pollo asado        | Baja         | principal     |

....
Las bases de datos RELACIONALES llevan decadas tratando con estos problemas. Y se han vuelto MUY EFICIENTES en su gestión.

Cada vez que un dato nuevo se añade en una tabla de una BBDD, el dato se guarda SIEMPRE al final de la tabla!
Ahora... esto destroza cualquier PREORDENACION QUE YO HAYA HECHO.

Qué tienen las BBDD para poder hacer búsquedas eficientes, aplicando ese algoritmo llamado BUSQUEDA BINARIA? INDICES

INDICE en una BBDD?? Lo mismo que en papel (que un libro)
Una COPIA ordenada de los datos, donde cada dato tiene asociada su UBICACION

## GENERO EL INDICE TITULO

TITULO                          UBICACION: IDENTIFICADOR

Bacalao al pil pil              7
Ensalada                        8
Lasaña boloñesa                 5
Lentejas estofadas              2
-
Paella                          1
-
Pasta boloñesa                  4
-
Pasta carbonara                 3
-
Pizza carbonara                 6


Ahora que tengo ese INDICE CREADO, imaginad que quiero meter Bacalao al pil pil!
En la tabla lo pongo al final!

Claro.. aquí llegamos a un compromiso (que es configurable en las BBDD?)
El ratio de blanco (huecos) que quiero.
- Si dejo muchos huecos? Tiro mucho disco duro = Tirar mucha pasta
- Si dejo pocos huecos?   Con más frecuencia tengo que reescribir el INDICE

---

El almacenamiento hoy en día es BARATO o CARO? DE LAS COSAS MAS CARAS EN LOS ENTORNOS DE PRODUCCION
3Tbs para casa: 60€
Para la empresa: 150€

El dato SIEMPRE ES LO MAS VALIOSO DE TODO.
Y en un entorno de producción cual es estandar? 
Cada dato, en cuantos sitios se guarda independientes? Al menos en 3 (RAID5)
El dato lo quiero al menos en 3 sitios. Pero físicamente en sitios diferentes.
Para la empresa los 3Tbs son 3HDD guays : 450€

---

Pero hasta aquí, todo lo que os he contado, ya lo hacen las BBDD relacionales.
Se han vuelto expertas en estos temas.

Qué necesidad tengo de una herramienta como elasticSearch: INDEXADOR/Buscador?

LAs BBDD relacionales son expertas en tratar con:
- Información MUY ESTRUCTURADA!
- No son nada buenas, para ciertos TIPOS DE BUSQUEDAS.

--- 

TITULO                          UBICACION: IDENTIFICADOR

Bacalao al pil pil              7
Ensalada                        8
Lasaña boloñesa                 5
Lentejas estofadas              2
Paella                          1
Pasta boloñesa                  4
Pasta carbonara                 3
Pizza Carbonara                 6


Término de búsqueda: "CARBONARA".

En cúantas de esas recetas tengo la palabra "CARBONARA"? De hecho en ninguna!
Me temo que lo que aparece es "carbonara"

El índice me podría ser de un poco de ayuda. Sobre el índice que algoritmo he de aplicar en este caso? FULL SCAN del INDICE

DIFICULTAD:

Término                     Ubicaciones:
Baja                        2, 5, 6
Media                       1, 3, 4
Alta                        7



| Titulo             | Dificultad   | Tipo de plato | ID |
| ------------------ | ------------ | ------------- | -- |
| Paella             | Media        | principal     |  1 |
| Lentejas estofadas | Baja         | primero       |  2 |
| Pasta carbonara    | Media        | primero       |  3 |
| Pasta boloñesa     | Media        | primero       |  4 |
| Lasaña boloñesa    | Baja         | segundo       |  5 |
| Pizza carbonara    | Baja         | pizza         |  6 |
| Bacalao al pil pil | Alta.        | principal     |  7 |

## Problemas adicionales con las búsquedas:

De cada receta guardo:
- Titulo
- Ingredientes
- Tiempo necesario
- Dificultad
- Tipo de plato
- ....


Pero quiero hacer una búsqueda por un criterio que no está ahí!
Tipo de cocina:
- Mediterranea
- Italiana
- Asiatica
- India

Una cosa es lo que viene el cada registro (datos de los que dispongo) 
y otra cosa es las búsquedas que yo quiera hacer.
Y a veces, **quiero hacer búsquedas por datos que no vienen en los registros**.

A lo mejor me creo un índice llamado TIPO DE COCINA: 
Y relleno ese índice con la salida de un PROGRAMA: 
- Si la receta habla de pizza -> ITALIANA
- Si contiene paella -> ESPAÑOLA

## Ejemplo más concretio de esto:

Tengo los accesos a un servidor web:
- Hora
- Usuario
- IP
- URL

Puedo hacer una búsqueda por PAIS del que accede?
Quizás puedo cear un INDICE para hacer esta búsqueda, obteniendo el PAIS? 
De una BBDD de geoposicionamiento de IPs

---

bacalao-*-pil-pil                   7
ensalad                             8
ensalad -rus                        9
lasana-bolon                        5
lentej -estof                       2
paell                               1
past-bolon.                         4
past-carbon-ivan-*-style            3
pizz-carbon-recet-*-*-abuel.        6


Término de búsqueda: "CARBONARA".

En casos como este, echamos mano de los INDICES INVERTIDOS.
sobre cada dato original, aplico un algoritmo (programa), para generar otros datos, que son lo que guardo en el INDICE.
1- Poner todo en minusculas y quitar tildes
2- Tokenizar: Espacios en blanco, barras bajas, guiones, signos de puntuación, parentesis, corchetes
3- Eliminar palabras vacias de significado (stop words)
4- quitar plurales... géneros o incluso diminutivos, aumentativos, superlativos... quedarme con la RAIZ ETIMOLOGICA de la palabra
5- extraigo los terminos que me han quedado y los ordeno

TERMINOS DEL INDICE             UBICACIONES
abuel                           6(6ª)
bacalao                         7(1ª)
bolon                           5(2ª), 4(2ª)
carbon                          3(2ª), 6(2ª)
ensalad
estof
ivan
lasana
lentej
paell
past
pil
recet
style

Y ESTO ES LO QUE SE LLAMA UN INDICE INVERTIDO

Si ahora me piden buscar CARBONARA: 
1- Aplicar a el término o érminos de búsqueda el MISMO procedimiento que aplico cuando se añaden nuevos datos al conjunto
  CARBONARA -> carbon
2- Ahora si puedo aplicar una búsqueda binaria sobre el índice

En las BBDD relacionales... por defecto no se hace NADA De esto. Hay algunas que me permiten EXPLICITAMENET hacer este tipo de
tranformaciones apra generar INDICES INVERTIDOS. NO todas lo soportan. 
Y aún así, las funcionaldiades que me dan suelen ser bastante pobres.
Además, especialmente hoy en día, hay mucha información que NO SOY CAPAZ DE METER EN UNA BBDD relacional.
Porque no es estructurada.

# EJEMPLO

Monto una web... AMAZON !
Y quiero guardar los datos de un producto que vendo.
Los meto en ORACLE !!!!

En mi tabla PRODUCTOS:
- Nombre del producto
- Tipo
- Precio
- Marca
- Modelo
- Tallas disponibles?
- Potencia?
- Peso?
- Dimensiones?
- Número de páginas?
- Tipo de portada?
- Material


---

## ElasticSearch.

Creeis que ES sabe crear índices normales? DIRECTOS ... NO
Creeis que ES sabe crear índices invertidos?        ... NO

ES no sabe crear ni un solo índice. VAYA RUINA !

ES usa una herramienta Opensource capaz de generar todo tipo de índices: LUCENE y es un proyecto de APACHE
Básicamente es una librería desarrollada JAVA que es capaz de crear, mantener y usar índices para realizar búsquedas.

Qué es pues ES?

Un orquestador de LUCENEs.

De hecho no es el único orquestador de LUCENES que hay en el mercado: 
hay otro proyecto de APACHE Opensource y gratuito que hace lo mismo: SolR

Cuando instalamos un ElasticSearch, siempre montamos un CLUSTER:

Cluster? Conjunto de procesos, que pueden estar en el mismo/distintos servidores físicos.
Habítualmente esos procesos (programas en ejecución) los tendremos en distintos servidores físicos (HA, Escalabilidad)

Esos procesos que forman un cluster irán indexando documentos, y permitirán posterioremnte hacer búsquedas sobre ellos.
Para ello, usan procesos de LUCENE !

### HA???

High availibity: Alta disponibilidad?
- Tratar de garantizar la no pérdida de la información.
- Tratar de garantizar que mi sistema (el acceso al dato) está disponible una determinada cantidad de tiempo. SLA

### ESCALABILIDAD???

Capacidad de ajustar mi infraestructura a las necesidades de cada momento -> AUMENTAR
Hablando de datos... cada vez... según pasa el tiempo qué tengo? MAS DATOS

De esto se encarga ES. Montar muchos LUCENES que se encargan de guardar los INDICES.
Cada dato lo guardará en varios LUCENES                 -> lo que ofrece HA
Y puedo según me interese meter más LUCENES a funcionar -> ESCALABILIDAD

---

En ES usamos el término INDICE...
pero, en ES, el término INDICE no significa nada NI PARECIDO a lo que significa en las BBDD relacionales.

## ¿Qué es un índice en ES?

Es "el equivalente" en una BBDD relacional a una tabla.

> Es un conjunto de LUCENES que guardan un conjunto de documentos sobre los que puedo hacer búsquedas.

Documentos? que tipo de documentos?
- Fichero .log?     NO
- Documento WORD?   NO
- Documento pdf?    NO

ES lo único que guarda son documentos JSON!
Entonces?
- Si quiero guardar un fichero word, que tengo que hacer? Convertirlo a JSON
- Si quiero guardar un fichero pdf, que tengo que hacer?  Convertirlo a JSON
- Si quiero guardar un fichero log, que tengo que hacer?  Convertirlo a JSON

Hace ElasticSearch esas transformaciones? NO. Necesitaré de otros programas que me ayuden con estas transformaciones.
---

# JSON?

Todo eso son documentos JSON per se... sin nada más

---
true
---
"hola"
---
3.9
---
{
    "usuario": {
        "nombre": "Ivan",
        "edad": 44
    }
}
---
[
    "usuario": {
        "nombre": "Ivan",
        "edad": 44
    },
    "usuario": {
        "nombre": "Juan",
        "edad": 21
    }
 ]
---
[1,2,3,4,5]
---

JSON: JS Object notation. Notación para definir objertos en el lenguaje JS.

--- 

### INDICES EN ELASTICSEARCH

> Colección de Lucenes que guardan INDICES TRADICIONALES (directos e invertidos) generados a partir de un conjunto de 
documentos, sobre los que realizar búsquedas.

INDICE EN ELASTICSEARCH: RECETAS
 SHARDs PRIMARIOS   SHARDs de REPLICACION
- LUCENE 1          LUCENE 1 copia1 , LUCENE 1 copia2
- LUCENE 2          LUCENE 2 copia1 , LUCENE 2 copia2
- LUCENE 3          LUCENE 3 copia1 , LUCENE 3 copia2

SHARD = LUCENE

{
    "titulo":       "Paella mixta",
    "dificultad":   "media",
    "ingredientes": [
        {
            "nombre": "arroz",
            "cantidad": "1kg"
        },
        {
            "nombre": "pollo",
            "cantidad": "500g",
            "detalle": "troceadito"
        }
    ]
}
EL DATO, el DOCUMENTO, lo tendré que tener guardado en una BBDD Externa a ES. ES no lo va a guardar por mi.

Guarda ES el documento original? tal y cual viene? A priori NO.
En algún sitio tendré guardado: "Paella mixta con pollo" ? NO le interesa este dato para hacer la búsqueda... le interesa:
- paella
- mixta
- pollo

Igual que google puede guardar una copia del documento, tal y como está en el momento de indexarlo,
ElasticSearch puede hacer lo mismo (por defecto lo hace)...

Eso si, lo que tengo es una copia, no el original.... y además, 
una colección de INDICES para poder buscar RAPIDITO ese documento.


Cada documento irá a un LUCENE... El lucene al que irá se determina mediante un proceso llamado ENRUTAMIENTO (ROUTING)
Que me aporta el tener varios LUCENES? ESCALABILIDAD
El poder tener a mas hombrecillos trabajando y poder ser capaces de procesar MAS DATOS/DOCUMENTOS

Ahora bien... en cualquier entorno de producción que se precie... además de ESCALABILIDAD he de asegurar LA ALTA DISPONIBILIDAD.
Para ello, lo rimero que necesito es que cada DATO esté replicado... almacenado en varios lugares FISICOS.

Cada uno d esos LUCENES van a tener asociados OTROS LUCENES que guardarán una REPLICA de los datos
---
NOTA: 

Imaginaos un fichero de LOG de APACHE WEB SERVER!
Qué datos se guardan ahí:
- URL,
- Fecha,
- IP,
- Codigo respuesta,
- BYTES

Si quiero que esa información se INDEXE en ES... que es lo primero que necesito hacer?
- Convertirla a JSON. El fichero entero? NO... de hecho... en algún momento hay "UN FICHERO DE LOG ENTERO"

En el fichero yo voy escribiendo cosas (entradas del log)... en algún momento puedo decir que el fichero está completo? NO

Eso si... cada entrada del fichero:
- URL,
- Fecha,
- IP,
- Codigo respuesta,
- BYTES

puedo convertirla a JSON y mandarla como UN DOCUMENTO a ELASTIC SEARCH 

Pero si os fijais... los documentos de LOG (entradas de un log) tienen una característica muy especial...
Son datos que están muertos -> NO CAMBIAN NUNCA

Puede cambiar una entrada de un log? NO... es un evento que ocurrió en el pasado. Hasta que no descubramos las maquinas para
viajar en el tiempo, será imposible que un evento pasado que haya registrado cambie.

Podré añadir datos nuevos. Pero no modificar los que tengo.

Y... si un documento no va a cambiar (ES IMPOSIBLE POR SU NATURALEZA) en el futuro...
Si guardo una copia del mismo en el momento de indexarlo... esa copia va a ser SIEMPRE VALIDA!
Y en este Y SOLO EN ESTE escenario, ES se convierte no solo en un idexador, sino también en un REPOSITORIO de documentos (BBDD)

Curiosamente este el el gran USO que le damos a ES. Como repositorio / motor de busqueda / indexador de documentos de LOG.

MONITORIZACION:
- Recoger esa información                                                       Agentes BEATS y Logstash    - 15%
- Generar un repo de información (eventos que han ocurrido)                     ElasticSearch ***           - 50%
- Visualizar, rocesar, generar alertas, sobre esos datos                        Kibana                      - 25-30%
                                                                        +   ----------------------------------
                                                                                Elastic Stack

---

Servidores web                          ENRUTAMIENTO / TRANSFORMACION

Servidor 1 - Apache
    access.log                                                              Cluster
Servidor 2 - Apache          KAFKA       Logstash        Logstash           ELASTIC SEARCH                  <<< KIBANA
    access.log                                                              5 nodos !
Servidor 3 - Apache                                                             Monitorización
    access.log
...                                                      Logstash           Cluster ES-2
Servidor N - Apache                                                             Negocio / Explotación de datos
    access.log
        Filebeat (Agente BEAT de Elastic)
        FluentD
        
        
KAFKA - Sistema de mensajería: WHATSAPP

---

Hay un problemón con ES: Sin tener ni puñetera idea de ES, es muy facil montar algo como lo que he definido ahí arriba
Pero... cagada... en cuanto pase 1 año.

ES, a pesar de que me permite empezar a trabajar con muy poco conocimiento, el problema es que las decisiones que TOME en un 
momento dato tendrán un IMPACTO ENORME a furuto... Y el remedio a FUTURO es MUY DOLOROSO.

Dicho de otra forma, más me vale tener suerte (o conocimiento) y acertar con las decisiones...

---

Al definir un índice en ELASTIC SEARCH, hemos de dar una serie de datos:
- Configuración:
    - Cuantos lucenes(shards) primarios quiero
    - Cuantos lucenes(shards) de replica quiero
    - Donde van esos lucenes dentro del cluster(nodos)
    - ....
- Mappings:
    - El tipo de dato
    - Cómo procesar ese dato (quiero quitar stopwrods, que tokenizer, ignore case)
- Alias del índice. A un índice me puedo referir con distintos nombres.

---
1º cuestion. ES quiere manejar poco o muchos INDICES? Prefiere índices muy grandes o más pequeños? PEQUEÑOS !
Y nosotros queremos indices PEQUEÑOS.

Las operaciones de mantenimiento de ES se realizan a nivel de INDICE... Y ESTO ES BASICO !

Quiero guardar los logs de 5 servidores APACHE que tengo. Cómo los guardo? Cúantos índices quiero usar?
1 I 
1 al día
5 , 1 por servidor

Lo índices el ES siempre SIEMPRE, SIEMPRE, van en función al TIEMPO

Los índices para que sirven? Para hacer búsquedas... Y cual es el primer dato que uso al hacer una búsqueda? FECHA !
Voy a hacer búsquedas del tipo... DEMA LOS LOG DE LOS ULTIMOS 17 años? DE TODA LA VIDA? Pocas veces.
Mucho menos habitual: Dame los logs de los ultimos 5 minutos, 5 horas, día.
Yo puedo desactivar INDICES en una búsqueda.
Puedo cerrar indices.
Puedo congelar índices.

En función de si recibo más o menos datos, y de las búsquedas que quiera hacer en el futuro, querré en los apaches:
- 1 indice cada hora
- 1 indice cada día
- 1 indice cada semana
- 1 indice cada mes.

Acabaré con algo como:          ALIAS
- Log_Apache_Enero_2022 + Log_Middleware_Apache_Enero_2022
- Log_Apache_Febrero_2022
- Log_Apache_Marzo_2022
- Log_Apache_Abril_2022
- Log_Apache_Mayo_2022
- ...
- Log_Apache_Diciembre_2022


- Log_Nginx_Enero_2022 + Log_Middleware_Nginx_Enero_2022


Esto es lo que nos permite aplicar una buena política de mantenimiento de INDICES.
Y ES se ha diseñado para tener índices creados de esta forma.

Pero... cuando hago una búsqueda, en ocasiones me interesa juntar datos de distintos INDICES.
ES permite el uso de PATRONES de nombres de índices al hacer BUSQUEDAS:
    Log_Apache_*            Qué le pasaría a esta búsqueda según pase el tiempo?
        Cuando un Indice tenga más de 1 año, sácalo de las búsquedas por defecto.
    Log_Apache_*_2022

Pero además en un momento dado, me puede interesar una búsqueda del tipo:
    Log_Middleware_*_Enero_2022 -> Apache, Nginx, Tomcat, Weblogic...
    
    
Imaginad que tengo datos de log de apache. Un indice por mes.

Eso significa que al menos a cúantos LUCENES tengo que preguntar para sacar los datos de todo un año? Al menos a 12

Y cuando hago un query... que necesita hacer ES?
Quiero todos los errores de 2022, de tipo 503 que he tenido en el servidor apache.
- Le tiene que preguntar a los 12 lucenes....
- Juntar todos esos datos -> Puede que este proceso requiera REORDENAR los datos.
- Esto puede tener graves consecuencias en rendimiento. SEGURO QUE REQUIERE MONTON DE RECURSOS: CPU + MEMORIA RAM

ENERO:

503  Nivel 1 Lunes
503  Nivel 2 Jueves
503  Nivel 2 Sabado

+

FEBRERO:

503  Nivel 1 Lunes
503  Nivel 2 Miercoles
503  Nivel 3 Domingo
-------------------------

503  Nivel 1 Lunes
503  Nivel 1 Lunes
503  Nivel 2 Jueves
503  Nivel 2 Sabado
503  Nivel 2 Miercoles
503  Nivel 3 Domingo
---
Y de hecho es habitual dentro del cluster de ES dedicar máquinas SOLO a esta tarea!

Y aquí nos aparece un concepto nuevo. Y es que los nodos de un cluster no son todos iguales entre si. 
Cada uno tiene su proia misión:
- Nodos maestros
- Nodos de datos: Los que tienen los lucenes
    - Datos en caliente
    - Datos en templados
    - Datos en frio
    - Datos congelados
- Nodos coordinadores
- Nodos de ML
- Nodos de ingesta.

De donde saco la información de CUANTOS NODOS ME INTERESA DE CADA TIPO? MONITORIZACION !
Monitorizar el Cluster de ES >> Obtengo información de cómo configurar el cluster, 
e irlo adaptando a las necesidades según pase el tiempo.


En kubernetes a un POD:
resources:
    limit:
        memory: Maximo que puedo pedir
    request:
        memory: Minimo garantizado
        
---

FILEBEAT

Va leyendo linea a linea de ese fichero.

47.61.49.162 - - [16/Mar/2023:12:05:19 +0000] "GET / HTTP/1.1" 304 - "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/109.0.0.0 Safari/537.36"

Le manda a Logstash un documento JSON... según esquema BEATS

- Quien está mandando un determinado mensaje: Filebeat, MetricBEat
- Donde está corriendo
- Fecha
- Mensaje

---

XML > Estandar del W3C. Sale un estandar nuevo XSD: Schemas XML

<usuario>
    <nombre>Ivan</nombre>
    <edad>44</edad>
</usuario>

<coche>
    <fabricante>bmw</fabricante>
    <modelo>serie 4</modelo>
</coche>

JSON también tenemos esquemas
YAML tambien tenemos esquemas
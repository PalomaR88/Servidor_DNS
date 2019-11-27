# Servidor DNS
Vamos a instalar un servidor dns que nos permita gestionar la resolución directa e inversa de nuestros nombres. Cada alumno va a poseer un servidor dns con autoridad sobre un subdominio de nuestro dominio principal gonzalonazareno.org, que se llamará tu_nombre.gonzalonazareno.org.

Indica al profesor el nombre de tu dominio para que pueda realizar la delegación en el servidor DNS principal papion-dns.

El servidor DNS se va a instalar en el servidor1 (croqueta). Y en un primer momento se configurará de la siguiente manera:

-El servidor DNS se llama croqueta.tu_nombre.gonzalonazareno.org y va a ser el servidor con autoridad para la zona tu_nombre.gonzalonazareno.org.
- El servidor debe resolver el nombre de los tres servidores.
- El servidor debe resolver los distintos servicios (virtualhost y servidor de base de datos).
- Debes determinar si la resolución directa se hace con dirección ip fijas o flotantes del cloud depediendo del servicio que se este prestando.
- Debes considerar la posibilidad de hacer dos zonas de resolución inversa para resolver ip fijas o flotantes del cloud.

Instalación de bind9:
~~~
debian@croqueta:~$ sudo apt install bind9
~~~

Hay que añadir la siguiente línea al fichero de configuración /etc/bind/named.conf.options:
~~~
        allow-query{172.22.0.0/16; 192.168.202.0/24;};
~~~

Configuración del fichero /etc/bind/named.conf.local para indicar el fichero donde estarán los ficheros de resolución directa e inversa:
~~~
zone "paloma.gonzalonazareno.org" {
        type master;
        file "db.paloma.gonzalonazareno.org";
};
zone "22.172.in-addr.arpa" {
        type master;
        file "db.200.22.172";
};
zone "0.0.10.in-addr.arpa" {
        type master;
        file "db.0.0.10";
};
~~~


Se configuran los ficheros anteriormente citados: db.paloma.gonzalonazareno.org, db.200.22.172 y db.0.0.10.

Configuración de la resolución directa en db.paloma.gonzalonazareno.org:
~~~
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     croqueta.paloma.gonzalonazareno.org. palomagarciacampon08.gonzalonazareno.org. (
                              1         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN              NS      croqueta.paloma.gonzalonazareno.org.

$ORIGIN paloma.gonzalonazareno.org.
croqueta        IN  A       172.22.200.124
croqueta-int    IN  A       10.0.0.12
tortilla        IN  A       172.22.200.144
tortilla-int    IN  A       10.0.0.11
salmorejo       IN  A       172.22.200.133
salmorejo-int   IN  A       10.0.0.5
www             IN  CNAME   salmorejo
cloud           IN  CNAME   salmorejo
mysql           IN  CNAME   tortilla-int
~~~

Configuración de la resolución inversa de la ip externa en /var/cache/bind/db.200.22.172:
~~~
$TTL    86400
@               IN      SOA     croqueta.paloma.gonzalonazareno.org. palomagarciacampon08.gonzalonazareno.org (
                              1         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                          86400 )       ; Negative Cache TTL
;
@               IN      NS      croqueta.paloma.gonzalonazareno.org.

$ORIGIN 200.22.172.in-addr.arpa.
200.124             IN      PTR	croqueta.paloma.gonzalonazareno.org.
200.144             IN      PTR	tortilla.paloma.gonzalonazareno.org.
200.133             IN      PTR	salmorejo.paloma.gonzalonazareno.org.

~~~

Por último, la configuración de la zona inversa de la red interna, db.0.0.10:
~~~
$TTL    86400
@               IN      SOA     croqueta.paloma.gonzalonazareno.org. palo$
                              1         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                          86400 )       ; Negative Cache TTL
;
@               IN      NS      croqueta.paloma.gonzalonazareno.org.

$ORIGIN 0.0.10.in-addr.arpa.
12             IN      PTR      croqueta.paloma.gonzalonazareno.org.
11             IN      PTR      tortilla.paloma.gonzalonazareno.org.
5             IN      PTR       salmorejo.paloma.gonzalonazareno.org.
~~~




Entrega el resultado de las siguientes consultas a los servidores de nuestra red :
- El servidor DNS con autoridad sobre la zona del dominio tu_nombre.gonzalonazareno.org

paloma@coatlicue:~/DISCO2/CICLO II/SERVICIO DE RED E INTERNET/DNS$ dig ns paloma.gonzalonazareno.org
~~~
; <<>> DiG 9.11.5-P4-5.1-Debian <<>> ns paloma.gonzalonazareno.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 59711
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 70900943a191dcf6ada33b9b5ddb98d155ccfbe76faf85c0 (good)
;; QUESTION SECTION:
;paloma.gonzalonazareno.org.	IN	NS

;; ANSWER SECTION:
paloma.gonzalonazareno.org. 604766 IN	NS	croqueta.paloma.gonzalonazareno.org.

;; Query time: 0 msec
;; SERVER: 192.168.202.2#53(192.168.202.2)
;; WHEN: lun nov 25 10:03:13 CET 2019
;; MSG SIZE  rcvd: 106
~~~


- La dirección IP de algún servidor

~~~
paloma@coatlicue:~/DISCO2/CICLO II/SERVICIO DE RED E INTERNET/DNS$ dig tortilla.paloma.gonzalonazareno.org

; <<>> DiG 9.11.5-P4-5.1-Debian <<>> tortilla.paloma.gonzalonazareno.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 56042
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 4d9712031054ebab4f5d53f05ddb99a2c107a67bcb9bdabc (good)
;; QUESTION SECTION:
;tortilla.paloma.gonzalonazareno.org. IN	A

;; ANSWER SECTION:
tortilla.paloma.gonzalonazareno.org. 604800 IN A 172.22.200.144

;; AUTHORITY SECTION:
paloma.gonzalonazareno.org. 604557 IN	NS	croqueta.paloma.gonzalonazareno.org.

;; Query time: 4 msec
;; SERVER: 192.168.202.2#53(192.168.202.2)
;; WHEN: lun nov 25 10:06:42 CET 2019
;; MSG SIZE  rcvd: 131
~~~

- Una resolución de un nombre de un servicio

~~~
paloma@coatlicue:~/DISCO2/CICLO II/SERVICIO DE RED E INTERNET/DNS$ dig www.paloma.gonzalonazareno.org

; <<>> DiG 9.11.5-P4-5.1-Debian <<>> www.paloma.gonzalonazareno.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 5482
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 1efb192e507de971ac3fcd4b5ddb9a15589df826797ece16 (good)
;; QUESTION SECTION:
;www.paloma.gonzalonazareno.org.	IN	A

;; ANSWER SECTION:
www.paloma.gonzalonazareno.org.	604800 IN CNAME	salmorejo.paloma.gonzalonazareno.org.
salmorejo.paloma.gonzalonazareno.org. 604800 IN	A 172.22.200.133

;; AUTHORITY SECTION:
paloma.gonzalonazareno.org. 604442 IN	NS	croqueta.paloma.gonzalonazareno.org.

;; Query time: 5 msec
;; SERVER: 192.168.202.2#53(192.168.202.2)
;; WHEN: lun nov 25 10:08:37 CET 2019
;; MSG SIZE  rcvd: 150
~~~

- Un resolución inversa de IP fija, y otra resolución inversa de IP flotante. (Esta consulta la debes hacer directamente preguntando a tu servidor).

~~~
paloma@coatlicue:~/DISCO2/CICLO II/SERVICIO DE RED E INTERNET/DNS$ dig @croqueta -x 172.22.200.133

; <<>> DiG 9.11.5-P4-5.1-Debian <<>> @croqueta -x 172.22.200.133
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 2465
;; flags: qr aa rd; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 2
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 77e8cd42c4082b063442e2a85ddb9a67b721165451aa6c49 (good)
;; QUESTION SECTION:
;133.200.22.172.in-addr.arpa.	IN	PTR

;; ANSWER SECTION:
133.200.22.172.in-addr.arpa. 86400 IN	PTR	salmorejo.paloma.gonzalonazareno.org.

;; AUTHORITY SECTION:
200.22.172.in-addr.arpa. 86400	IN	NS	croqueta.paloma.gonzalonazareno.org.

;; ADDITIONAL SECTION:
croqueta.paloma.gonzalonazareno.org. 604800 IN A 172.22.200.124

;; Query time: 3 msec
;; SERVER: 172.22.200.124#53(172.22.200.124)
;; WHEN: lun nov 25 10:09:59 CET 2019
;; MSG SIZE  rcvd: 173
~~~


~~~
paloma@coatlicue:~/DISCO2/CICLO II/SERVICIO DE RED E INTERNET/DNS$ dig @croqueta -x 10.0.0.5

; <<>> DiG 9.11.5-P4-5.1-Debian <<>> @croqueta -x 10.0.0.5
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 38905
;; flags: qr aa rd; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 2
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 4395816ce064c3345568f1495ddb9b0eb17226b9785aca1e (good)
;; QUESTION SECTION:
;5.0.0.10.in-addr.arpa.		IN	PTR

;; ANSWER SECTION:
5.0.0.10.in-addr.arpa.	86400	IN	PTR	salmorejo.paloma.gonzalonazareno.org.

;; AUTHORITY SECTION:
0.0.10.in-addr.arpa.	86400	IN	NS	croqueta.paloma.gonzalonazareno.org.

;; ADDITIONAL SECTION:
croqueta.paloma.gonzalonazareno.org. 604800 IN A 172.22.200.124

;; Query time: 1 msec
;; SERVER: 172.22.200.124#53(172.22.200.124)
;; WHEN: lun nov 25 10:12:46 CET 2019
;; MSG SIZE  rcvd: 167
~~~


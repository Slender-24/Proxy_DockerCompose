# Pràctica: Proxies amb Nginx (Infraestructura amb Docker Compose)



Aquest repositori conté el codi i la memòria de l'evolució de la pràctica d'alta disponibilitat implementant un Proxy Invers amb memòria cau i balanceig de càrrega utilitzant contenidors Docker.

---

## Fases de Desenvolupament

Al llarg del projecte, s'ha construït la infraestructura de forma incremental complint cada requeriment indicat a la rúbrica d'avaluació.

### Fase 1: Un sol node web
Com a pas inicial, es va redactar un fitxer elemental `index.html`, agrupant els recursos multimèdia indicats (tres imatges i un vídeo reduït).
Es va crear una versió primigènia de l'arxiu orquestrador `docker-compose.yml` que només contenia un servidor web *Apache* utilitzant la imatge oficial `httpd:latest`.
**Problemes trobats**: Cap. La configuració clàssica d'Apache funcionava directament provant sobre el port obert per defecte a localhost.

Captura de la Fase 1:
<img width="1856" height="1033" alt="image" src="https://github.com/user-attachments/assets/4acf2ec8-808a-4f8c-8906-dd7e4bf4d5b0" />


### Fase 2: Segon node web
A continuació es va introduir a l'orquestració de Docker Compose un segon backend anomenat `apache2`, configurant-li exactament la mateixa imatge que al primer.
Per tal de distingir quin node responia cada petició en el futur es va planificar inicialment investigar logs, per posteriorment descobrir i implementar l'enviament de l'adreça IP del servidor upstream mitjançant capçaleres configurades en un pass frontal, que ens identificaren cadascú de forma directa al propi inspector del navegador (solució implementada més endavant).
**Problemes trobats**: En provar de testejar en l'avaluació local els dos servidors webs paral·lels, hi havia un xoc lògic assignant els ports (els dos pretenien escoltar directament el `80:80` obrint conflictes). Es va solucionar tapant-los amb `expose: "80"` deixant els ports només reeixits per la xarxa interna de docker, i preparant el terreny per al Nginx frontal final.

### Fase 3: Volum compartit
L'objectiu següent era mantenir la consistència entre el servidor `apache1` i `apache2` sense arribar a utilitzar la clònica de materials per construir les imatges prefabricades (estalviant pes). Es va generar un directori de treball extern `/html/` d'on vam localitzar l'estructura de fitxers desitjada, i utilitzant **Volums de Docker (Volumes)** al fitxer compose (`./html:/usr/local/apache2/htdocs`), vam forçar als dos apaches a absorbir el servei simultàniament d'una sola arrel unida actualitzable al vol.

### Fase 4: Proxy Invers amb balanceig (Round Robin)
El gruix de l'operació va consistir a posar en actiu de proxy centralitzador Nginx de la família `nginx:latest`. S'hi va connectar un volum d'arxiu en només lectura a `/etc/nginx/nginx.conf` i obrint exclusivament com a única exposició externa en ell el port 80 de client d'admissió en lloc dels contenidors apache ja aïllats.

Mitjançant la directiva **`upstream`** al fitxer del proxy, definírem com a destinataris de processament el grup lògic sota `apache1:80` i `apache2:80`. Nginx fa natural i per defecte el balanceig **Round Robin** a parts iguals entre aquestes peticions. Utilitzant l'ordre de `proxy_pass` enviem el flux final al *backend*.

A més a més, vam emmagatzemar finalment una línia especial per afegir la petició d'identificació de resposta de fase 2: `add_header X-Backend-Server $upstream_addr;`.  

**Problemes trobats**: Errors de latència o resolució en un estret marge a la construcció si s'inicia el Nginx abans que els DNS dels altres contenidors del compostat estiguessin preparats; resolt creant un procés seqüencial segur amb `depends_on`.

En el test actual comprovem la funcionalitat d'aquesta rotació al desgranar el flux a l'inspector del proveïdor amb la captura rotant cada clic de F5 demostrant de qui extreu informació:

CAPTURA DE ROUND ROBIN:
<img width="1533" height="323" alt="image" src="https://github.com/user-attachments/assets/b607dc7c-8d3b-4650-b984-d396934a2f47" />


### Fase 5: Memòria cau (Proxy Cache)
Finalment, un bloqueig de memòria virtual cau va ser dissenyat establint una assignació a `/var/cache/nginx` del directori Nginx utilitzant `proxy_cache_path` amb les delimitacions obligatòries limitant l'extensió on s'allotjava al màxim assequible fins `max_size=500m`. Habilitant zones de memòria 200 en `location` amb la directiva `proxy_cache mi_cache;`. 

Com es prova i evidencia amb el header addicional dissenyat per monitoratge general implementat explícitament `X-Cache-Status` podent extreure de l'inspector `HIT`, `MISS` i `EXPIRED` (amb latència o absència depenent en la recàrrega):

CACHE HIT:
<img width="810" height="230" alt="image" src="https://github.com/user-attachments/assets/ef9137f3-5324-4cbd-a6f6-81f24c5309ed" />

---

### Instruccions Tècniques Curtes per Avaluador (Desplegament i Eliminació de Xarxa docker)
1. Col·locar per primer cop el grup d'emmagatzematge multimèdia local intern `/html/` correctament els (`imagen1.jpg`, `imagen2.jpg`, `imagen3.jpg` i `video.mp4`).
2. S'activa a la xarxa sencera a segon pla per l'escriptori / port extern `sudo docker compose up -d` 
3. S'accedeix via un ordinador de visualització al host port exposat des d'un navegador amb http per resoldre IP en `http://localhost`.
4. El sistema s'entera local per tancar amb  `sudo docker compose down`.

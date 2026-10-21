# NFS

# Activitat AA1: Configuració d'un servidor NFS

## Presentació de l'activitat

El problema que planteja **DevOptimize Solutions** és la necessitat d'evitar que el codi font, els documents de disseny, la documentació tècnica, etc., estiguin fora de control als equips dels desenvolupadors. Tampoc volen dependre d'un servei de cloud per compartir fitxers.

Per això, teniu l'encàrrec de mostrar al CEO de l'startup el funcionament d'un servidor NFS per a la compartició de fitxers en una xarxa local. El client ha insistit que actualment treballa sense un entorn d'autenticació centralitzada i que, de moment, no té previst implementar-ne cap.

Per mostrar al client com quedarà la solució proposada a partir de les seves demandes i poder mostrar també les seves limitacions, se t'encarrega fer una demostració del sistema.

Crearàs un servidor NFS (NFSv3) i un client Linux que consumeixi els recursos compartits. Hauràs de crear usuaris i grups per simular l'entorn del client i demostrar el control d'accés utilitzant les opcions d'exportació (`/etc/exports`) i els permisos del sistema de fitxers (`chmod`, `chown`).

## Durada de l'activitat

La durada prevista de l'activitat és de **8 hores a classe**.

## Objectius específics de l'activitat

La finalitat de la tasca és que l'alumnat configuri un servidor de fitxers NFS amb diversos nivells d'accés i prepari un client Linux per accedir automàticament als recursos compartits.

## Competències treballades

- **f)** Instal·lar, configurar i mantenir serveis multiusuari, aplicacions i dispositius compartits en un entorn de xarxa local, atenent les necessitats i els requeriments especificats.
- **g)** Realitzar proves funcionals en sistemes microinformàtics i xarxes locals, localitzant i diagnosticant disfuncions per comprovar-ne i ajustar-ne el funcionament.
- **o)** Utilitzar els mitjans de consulta disponibles, seleccionant el més adequat en cada cas, per resoldre en un temps raonable supòsits no coneguts i dubtes professionals.

## Resultats d'aprenentatge i criteris d'avaluació

**RA4.** Gestiona els recursos compartits del sistema, interpretant especificacions i determinant nivells de seguretat.

- **4.1** Reconeix la diferència entre permís i dret.
- **4.2** Identifica els recursos del sistema que es compartiran i en quines condicions.
- **4.3** Assigna permisos als recursos del sistema que es compartiran.
- **4.5** Utilitza l'entorn gràfic per compartir recursos.
- **4.6** Estableix nivells de seguretat per controlar l'accés del client als recursos compartits en xarxa.

## Continguts

- **4.1** Permisos i drets.
- **4.2** Compartir arxius i directoris a través de la xarxa.
- **4.3** Configuració dels permisos dels recursos compartits.

## Capacitats clau treballades

- Autonomia.
- Innovació.
- Relació interpersonal.
- Organització del treball.
- Responsabilitat.
- Treball en equip.
- Resolució de problemes.

## Semàfor d'ús de la IA

🟠 **Aquesta activitat permet un ús parcial o restringit.**

Està permès utilitzar la IA com a eina de suport per millorar la redacció dels informes, fer cerques preliminars d'informació, estructurar idees o explicar conceptes teòrics complexos.

**Condicions:** Cal processar, entendre i validar sempre els resultats rebuts. Està totalment prohibit copiar l'enunciat d'un exercici directament al xat de la IA i enganxar la resposta generada per al lliurament final sense treball propi ni anàlisi crítica.

---

# Enunciat de l'activitat

## Requeriments previs

Cal que tingueu un servidor Linux (Ubuntu Server) instal·lat i configurat de l'activitat anterior. En cas que sigui necessari, torneu a desplegar el servidor a partir del fitxer `.OVA` que vau generar.

## Fase 1. Preparació de l'entorn

Per al client, utilitzarem **Zorin OS 18** amb configuració de xarxa NAT.

El servidor tindrà una interfície configurada amb **xarxa només amb amfitrió** i l'altra amb **xarxa NAT**. Això ens permetrà que el client tingui accés a Internet i que el servidor pugui comunicar-se amb el client.

Ens assegurarem que ambdues màquines tinguin accés a Internet i que es puguin comunicar entre elles a través de la xarxa només amb amfitrió. També actualitzarem els sistemes amb les últimes actualitzacions disponibles.

## Fase 2. Preparació del servidor

Abans de compartir res, hem de preparar els usuaris i els directoris al servidor.

### 2.1. Creació de grups

Crear dos grups per al client:

- `devs`
- `admins`

### 2.2. Creació d'usuaris

Crear els usuaris següents:

- `dev01`, membre del grup `devs`.
- `admin01`, membre del grup `admins`.

### 2.3. Creació de directoris al servidor

Crear els directoris següents:

- Directori per als projectes de desenvolupament: `/srv/nfs/dev_projects`
- Directori per a les eines d'administració: `/srv/nfs/admin_tools`

### 2.4. Permisos del servidor

Aquest és el punt clau de la configuració:

- Es vol que els desenvolupadors tinguin control total sobre els seus projectes.
- Es vol que els administradors tinguin control sobre les seves eines.
- En tots dos casos, l'usuari propietari serà `root`.

Com a pas final, s'instal·laran els paquets necessaris per al servei NFS al servidor i es configurarà l'exportació dels directoris amb les opcions adequades.

> **Nota:** Perquè aquesta pràctica funcioni correctament, heu de replicar aquests usuaris i grups al client o, idealment, assegurar-vos que els UID i GID (els números d'identificació) coincideixin a les dues màquines.

## Fase 3. L'exportació d'administració: el dilema del `root_squash`

El client necessita que el directori `/srv/nfs/admin_tools` sigui accessible per a l'equip d'administradors. A vegades, l'usuari `root` del client, que sou vosaltres, els consultors, necessitarà escriure en aquest directori per instal·lar eines. Aquí mostrarem un error típic i la seva solució.

### 3.1. Prova 1: l'error comú

1. Exportar el directori `/srv/nfs/admin_tools` amb les opcions `rw,sync`.
2. Des del client, muntar aquest recurs compartit a `/mnt/admin_tools`.
3. Com a `root` del client, intentar crear un fitxer dins d'aquest directori muntat.
4. Verificar quin és el propietari del fitxer creat.

**Pregunta:** Per què el fitxer té aquest propietari? Justifiqueu la resposta amb una explicació tècnica de l'opció `root_squash`.

### 3.2. Prova 2: la solució

1. Modificar l'exportació del directori `/srv/nfs/admin_tools` per incloure l'opció `no_root_squash`.
2. Al client, desmuntar i tornar a muntar el recurs compartit.
3. Com a `root` del client, intentar crear un fitxer dins d'aquest directori muntat.
4. Observar quin és el propietari del fitxer creat aquesta vegada.

**Pregunta:** Ha canviat alguna cosa? Justifiqueu la resposta amb una explicació tècnica de l'opció `no_root_squash`.

## Fase 4. L'exportació de desenvolupament: permisos `rw` vs. `ro`

Editar el fitxer `/etc/exports` per afegir dues exportacions per al mateix directori.

El client vol que:

- La xarxa d'administració, per exemple `192.168.56.0/24`, hi pugui escriure.
- La xarxa de consultors, que simularem amb una altra IP, per exemple `192.168.56.100`, només hi pugui llegir.

### Proves des del client

1. Muntar el recurs compartit a `/mnt/dev_projects`.
2. Provar d'escriure-hi com a usuari `dev01`. Hauria de funcionar.
3. Desmuntar el recurs i canviar manualment la IP del client a `192.168.56.100`.
4. Tornar a provar d'escriure al directori muntat com a usuari `dev01` i comprovar que només funciona la lectura.
5. Canviar d'usuari al client a `admin01` i tornar a provar d'escriure al directori muntat.

**Pregunta:** Per què `admin01` no pot escriure al directori? Comproveu que no és membre del grup `devs` i expliqueu com hi influeixen els permisos locals del sistema de fitxers.

## Fase 5. Muntatge automàtic amb `/etc/fstab`

És evident que els usuaris no poden estar muntant manualment els recursos compartits cada vegada que reinicien el sistema. Per això, es configurarà el muntatge automàtic mitjançant el fitxer `/etc/fstab` al client.

### Procediment

1. Editar el fitxer `/etc/fstab` al client per afegir les entrades necessàries per muntar automàticament els recursos compartits NFS als directoris següents:
   - `/mnt/admin_tools`
   - `/mnt/dev_projects`
2. Executar la comanda `mount -a` per provar les entrades sense reiniciar.
3. Reiniciar el client.
4. Verificar que els recursos compartits s'han muntat correctament.

---

# Lliurament de l'activitat

L'activitat es pot lliurar de dues maneres.

## Opció 1. Documentació en format PDF

El document ha d'incloure:

- Captures de pantalla.
- Explicacions detallades del procés seguit.
- Respostes a les preguntes plantejades.
- Una conclusió raonada.

## Opció 2. Repositori de GitHub

El repositori ha de tenir l'estructura de fitxers següent:

```text
.
├── README.md
├── AA1-NFS.md
└── images/
    ├── screenshot1.png
    └── screenshot2.png
```

- `README.md`: contindrà l'enunciat de l'activitat.
- `AA1-NFS.md`: serà el fitxer principal amb la descripció de l'activitat.
- `images/`: carpeta on es guardaran les captures de pantalla.

## Documentació del procés

Documenta tot el procés seguint les fases descrites anteriorment.

Per a les comandes, escriu-les com a codi perquè les puguis copiar posteriorment amb facilitat.

Per exemple:

```bash
sudo apt update
```

Inclou captures de pantalla per demostrar el funcionament correcte de la prova de concepte.

Per inserir una imatge en Markdown, utilitza la sintaxi següent:

```markdown
![Descripció de la imatge](images/nom_de_la_imatge.png)
```

Respon les preguntes plantejades en les diferents fases.

### Conclusió

Redacta una conclusió raonada amb les teves recomanacions per al client. Cal plantejar els avantatges de la solució proposada, així com les limitacions i les possibles millores futures.

> **Nota:** Els termes «redacta» i «raonada» indiquen clarament que s'espera que escriguis amb les teves pròpies paraules i que aportis un pensament crític al teu treball. Pensa que, si les conclusions les redacta directament una IA, ens plantejarem seriosament substituir-te per ella.

## Material de suport

- Material propi del mòdul: **NF1.UD4.A1 Compartició d'arxius i carpetes**.
- Ruiz, P. *Capítulo 11: Instalar y configurar NFS en Ubuntu*. Sistemas Operativos en Red (Actualizado). SomeBooks.es, juny de 2022. [Enllaç](#).

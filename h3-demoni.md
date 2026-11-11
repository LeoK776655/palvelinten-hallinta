# H3 - Demoni

## Lue ja tiivistä

### Apache installed with Ansible

- Ydinkonsepti: package, file, service
- Rakenne koostuu "master" kopioista konfiguraatiotiedostosta, varsinaisesta suorittavasta YAML-tiedostosta ja handlerista joka restarttaa daemonin.

Jos netti kaatuisi kesken asennuksen, mitä loppuprosessille käy?

### Handlers: running operations on change

- Käsittelee aivan samoja tehtäviä kuin muutkin
- On passiivinen niin kauas kunnes saa kutsun aktivoitua
- Oletuksena ajetaan vasta playbookin päätteeksi
- notify (kutsu) tulee olla tismalleen sama nimi kuin handlerilla

Jos on monta eri handleria voiko ne saada tekemään tehtävänsä vielä tietyssä järjestyksessä kun heidän vuoronsa suorittaa tulee?

### ansible-doc service

- Työkalu jolla ohjelmia voidaan käynnistää ja sammuttaa riippumatta init-järjestelmästä käytetään
- `enabled` määrittää käynnistyykö kyseinen palvelu automaattisesti startupissa tai palvelimen käynnistyksessä (yes tai no)
- `name` määrittää nimen ohjelmalle jota halutaan hallita
- `state` määrittää mihin tilaan palvelu halutaan laittaa. `started` varmistaa sen olevan päällä. `stopped` sammuttaa. `restarted` käynnistää uudelleen ja `reloaded` lataa sen uudestaan käynnistämättä.

Esimerkki

```
- name: Start service httpd, if not started
  ansible.builtin.service:
    name: httpd
    state: started
```

Onko loppujen lopuksi `restarted` ja `reloaded` minkäänlaista merkittävää eroa?

## a) Apache

Päivitetään sovellukset ja asennetaan apache komennoilla `sudo apt update` `sudo apt install apache2 -y`

Vaihdetaan etusivun omistajuutta jotta normaali käyttäjä voi muokata sitä ilman sudo tai root oikeuksia. 

```
sudo chown $USER:$USER /var/wwww/html -R
```
Linuxissa $USER on muuttuja joka toimii samana kuin sinun sillä hetkellä sisäänkirjautunut käyttäjä. Sama asia kuin käyttäisi `leo:leo`. -R eli recursive meinaa että komento omistajan muutosta menee rekursiivisesti jokaiseen alihakemistoon ja tiedostoon. 

Seuraavaksi testataan. Laitetaan pääsivulle jotain sisältöä

```
echo "<h1>Terve tämä on testi</h1>. > /var/wwww/html/index.html
```

Käydään sivuilla. Tarkistetaan ip osoite `ip a` ja kirjoitetaan se selaimeen. Toinen vaihtoehto on käyttää `curl localhost/index.html`

<img width="442" height="48" alt="demoni_testilapi" src="https://github.com/user-attachments/assets/f3c59bbb-1ec0-40a0-8e43-c66291ff4a9b" />

<img width="669" height="163" alt="demoni_testilapi2" src="https://github.com/user-attachments/assets/1a811b52-df56-48df-ae24-9e7a91709085" />

Testit onnistuneet, Apache haki oikean tiedoston.

## b) Nginx

Laitetaan Apache pois päältä ja varmistetaan että se ei lähde automaattisesti käyntiin kun tietokone käynnistetään.

<img width="625" height="166" alt="demoni_apachepois" src="https://github.com/user-attachments/assets/ca2a1e57-7195-45e6-9562-d77b611b2def" />

Asennetaan nginx. `sudo apt update` ja `sudo apt install nginx -y`

Nginx käyttää samaa document root kansiota, joten sen omistajuus on jo kunnossa.

Voimme suoraan testata toimiiko nginx. Kirjoittelin uuden viestin pääsivun sisällöksi.

<img width="719" height="76" alt="demoni_nginxtesti" src="https://github.com/user-attachments/assets/8a318671-b9c5-4663-b1b7-803044457ae6" />

Testi onnistunut, Nginx haki oikean tiedoston.

## c) Automatisointi

Poistetaan ensin Nginx jotta ansiblella olisi jotain ladattavaa. 
```
sudo purge nginx nginx-common -y
```
Testataan

<img width="719" height="59" alt="demoni_nginxveke" src="https://github.com/user-attachments/assets/9783c93f-aac0-48ac-a1c6-563f3b761dbd" />

Hyvä, poisto onnistui.

Tehdään uusi rooli ansibleen jotta järjestys pysyy kasassa. `roles/nginx/`

<img width="205" height="87" alt="Näyttökuva 2026-11-10 231450" src="https://github.com/user-attachments/assets/4f6e9c80-cdbc-439e-9048-708b433380f6" />

Ja roolin alle teemme kolme alihakemistoa: files, handlers sekä tasks.

Lisätään nginx site.yml tiedostoon.

<img width="414" height="156" alt="kuva" src="https://github.com/user-attachments/assets/47d134d1-a013-4b66-aa9e-64310fa482a5" />

Teron ohjeita hyödyntäen tehdään YAML-tiedostot handlers sekä tasks hakemistoihin. Files hakemistoon määritetään konfiguraatio.

Alla olevassa kuvassa `- apt:` asentaa Nginxin jos sitä ei ole vielä asennettu.

`- copy:` Siirtää minun tekemän konfiguraatiotiedoston paikallisesta koneesta kohdekoneeseen.

`- file:` Luo symlinkin joka aktivoi sivuston

`notify: restart nginx` Aktivoi handlerin, joka lopuksi käynnistää nginxin uudestaan jotta konfiguraatiomuutokset tulevat voimaan.

<img width="705" height="366" alt="Näyttökuva 2026-11-11 002158" src="https://github.com/user-attachments/assets/6ffaf282-c2ad-4ad3-896d-78bbfd00adb7" />

Alla olevasta kuvasta löytyy handlerin oma YAML-pätkä. Se aktivoituu ylemmän kuvan `notify` kohdasta.

<img width="585" height="99" alt="Näyttökuva 2026-11-10 235105" src="https://github.com/user-attachments/assets/f7d9f7f1-383b-445a-b60f-dc0aa6e427d7" />

Alla olevasta kuvasta näkyy vielä konfiguraatioasetukset jotka kopioidaan kaikille kohdekoneille asennuksen yhteydessä.

Tiedostoon määritetään kuunneltava portti, palvelimen nimi, kansio josta tiedostot haetaan ja määritetään mikä tiedosto toimii pääsivuna.

<img width="643" height="137" alt="Näyttökuva 2026-11-11 003413" src="https://github.com/user-attachments/assets/41b55719-7ba5-4768-ab06-8cfffbdab393" />

Testataan. `ansible-playbook site.yml -K`

<img width="724" height="134" alt="Näyttökuva 2026-11-10 234712" src="https://github.com/user-attachments/assets/8037e340-4bd0-4441-a316-9e6042433fac" />

Ihme ja kumma, meni ensimmäisellä yrityksellä läpi. Kokeillaan vielä `curl localhost`. Se vaatii nginxin toimiakseen.

<img width="429" height="59" alt="Näyttökuva 2026-11-10 234727" src="https://github.com/user-attachments/assets/57cfbf35-d260-48ef-9761-7f6e458c50fb" />

Hurraa se toimii.


## Lähteet
  
Apache. 2026. Luettavissa: https://httpd.apache.org/docs/2.4/. Luettu 10.11.2026

Karvinen, T. 2026. Apache installed with Ansible. Luettavissa: https://terokarvinen.com/apache-ansible/. Luettu 10.11.2026

Ansible Community Documentation. 2026. Handlers: running operations on change. Luettavissa: https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_handlers.html. Luettu 10.11.2026

Tekoälyä hyödynnetty materiaalin ja rakenteiden ymmärtämiseen.

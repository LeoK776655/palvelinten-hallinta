# h2 - Voileipä

## Lue ja tiivistä

### ansible-doc

copy

- Kopioi tiedosto tai hakemistorakenne koneelta määrättyyn paikkaan etäkoneelle
- Kun kopioidaan tiedosto mestarilta orjalle määritetään komennossa tiedoston lähde, päämäärä, omistaja, ryhmä ja käyttöoikeudet
- `content` määrittää sen sisällön joka kopioidaan
- `src` määrittää sen mistä se sisältö joka kopioidaan tulee
- `owner, group, mode` määrittävät kopioidulle tiedostolle omistajan, ryhmän sekä oikeudet

Content parametrillä se toimii tekstieditorina, src parametrillä se vain kopioi.

esimerkki

```
ansible.builtin.copy:
 src: /esim/tiedosto/siirretävä.conf TAI content: diiba-daaba
 dest: /etc/siirretävä.conf
 owner: foo
 group: foo
 mode: '0644'
```

apt

- Advanced Packet Tool hallitsee esim. Debianin paketteja (asentaa, päivittää, poistaa)
- `name` Määrität minkä nimisen paketin lataat/päivität/poistat. Voidaan myös tarkentaa saamaan haluttu tietty versio, esim. `foo=1.0`. Aliaksina toimii `package` sekä `pkg`.
- `state` Määrittää halutun paketin tilan. `latest` antaa uusimman version, `fixed` koittaa korjata systeemin rikkinäiset riippuvuudet.
- `update_cache` Ajaa saman komennon kuin `apt-get update`, mutta päivittää varaston ennen operaatioita

Kirjoittamalla parametrin update_cache: true varmistamme paketin viimeisimmän version
saamisen päivitettynä

esimerkki

```
ansible.builtin.apt:
  name: nginx
```
file

- Määrittää tiedostojen attribuutit, hakemistot tai viitaaukset toisiin hakemistoihin ja niiden kohteet (symlink)
- Voi myös poistaa yllämainitut asiat
- `path` Määrittää polun tiedostoon tai hakemistoon
- `recurse` Kun kohde on hakemisto (`state=directory`) määrittää tietyn tiedoston attribuutit rekursiivisesti muihin tiedostoihin.
- `src` Lähdetiedoston polku, käytetään vain kun luodaan linkkejä (`state=link` tai `state=hard`)
- `state` Määrittää tilan/tyypin sille mitä luodaan. Esim. `state=hard` luo tai muuttaa kovaa linkkiä. `state=absent` poistaa tiedoston/hakemiston.
- `owner, group, mode` Määrittävät kopioidulle tiedostolle omistajan, ryhmän sekä oikeudet

File moduulissa on erittäin helppoa vaihtaa tiedostojen parametrejä, kuten vaihtaa omistajaa tai ryhmää

esimerkki symbolisen linkin luomisesta

```
ansible.builtin.file:
  src: /tiedosto/johon/linkata/
  dest: /reitti/johon/symlinkata
  owner: foo
  group: foo
  state: link
```

jos luon käyttäjälle kotihakemiston `create_home` avulla ja myöhemmin poistan käyttäjän `state: absent`, mitä käy käyttäjän kotihakemistolle?

user

- Hallitsee käyttöjärjestelmän tilejä. Se voi luoda, poistaa tai muokata eri tilejä.
- `name` Kertoo nimen käyttäjälle
- `create_home` Määrittää luodaanko käyttäjälle kotihakemisto esim. `/home/marsilainen/`. Oletuksena päällä, saa poiskin.
- `comment` Kuvaus käyttäjästä
- `groups` Lista ryhmiä joissa käyttäjä on osana.
- `shell` Määrittää shellin johon käyttäjä kirjautuu, esim. `/bin/bash/`
- `state` Tila onko käyttäjä olemassa vai ei. `state=absent` tai `state=present`
- `system` Jos erikseen laitetaan `yes`, tekee käyttäjästä system-tason tilin eikä normikäyttäjää.

esimerkki luo käyttäjä matti meikäläinen kotihakemiston kanssa

```
ansible.builtin.user:
  name: mattimeikalainen
  create_home: yes (vapaaehtoinen, päällä jo oletuksena)
```

authorized_key

- Poista tai lisää SSH authorisoituja avaimia tietyille käyttäjille.
- `user` Käyttäjän nimi jonka avaimia halutaan muokata
- `key` SSH julkinen avain merkkijonona

Miten moduulin skaalautuminen onnistuu jos avaimia tulee enemmän?

esimerkki määritä authorisoitu avain tiedostolta

```
ansible.posix.authorized_key:
  user: mattimeikalainen
  state: present
  key: "{{lookup('file', '/home/mattimeikalainen/.ssh/id_rsa.pub') }}"
```

### Sudo without password

- kun muokataan sudoersien oikeuksia, käytetään sudo visudo.
- luodaan salasanattomille sudon käyttäjille oma ryhmä (sudoless)
- uudelle ryhmälle voi määrittää haluamansa säännöt hyödyntämällä erilaisia tokeneita
- varmista oma backup ja avaa uuteen terminaaliin root komennolla sudo -i

Mietintänä onko mahdollista rajata salasanattomuutta vain tiettyihin komentoihin? Eikö ole suuri
tietoturvariski antaa täysi sudo-oikeus ilman salasanaa?

### Passwordless sudo with ansible

- määritetään roolille omat tehtävät, moduulit ja parametrit joiden avulla automatisoidaan kirjautuminen
- julkisen ssh-avaimen voi myös saada `lookup` komennolla
- jos ajaa `ansible-playbook site.yml -K (--ask-become-pass)` ilman salasanatonta roolia pyytää ansible sinulta aina salasanan. Tämä johtuu `become: true`, se antaa komennolle sudo-oikeudet

Pohdin että olisi varmaan helpompaa skaalautuvuuden takia nimetä rooli joksikin yleiseksi, kuten "kouluusers".

### xkcd sarjakuva

- ilman sudoa voi olla hankalaa saada mitä haluaa
- sudon kanssa voi olla helpompaa saada mitä haluaa

Normaalisti `sudo` tuppaa kysymään salasanaa tai varmistusta

## a.) Sudoless

Ensiksi luodaan uusi käyttäjä. 

```
$sudo adduser leo2
```

Käyttäjälle määritetään salasana sekä tarkemmat tiedot. Tälle käyttäjälle annoin vain salasanan ja jätin tarkemmat tiedot tyhjiksi.

Kun käyttäjä on luotu, luodaan uusi ryhmä: `sudoless`

```
$sudo groupadd sudoless
```

Uusi ryhmä on nyt luotu, lisätään uusin käyttäjämme `leo2` ryhmään.

```
$sudo adduser leo2 sudoless
```
Ensimmäinen osa on tehty. Seuraavaksi tehdään uusi sääntö joka koskee "sudoless" ryhmän jäseniä. Sääntö määrittää sen että jäsenien ei tarvitse syöttää salasanaa käyttäessään sudo komentoja.

```
$sudo visudo /etc/sudoers.d/sudoless
```

`sudo visudo` on komento jolla voidaan muokata "sudoers" asetustiedostoja.

Tiedoston sisälle kopiomme ja liitämme Teron ohjeista valmiin koodipätkän.

```
%sudoless ALL = (ALL) NOPASSWD: ALL
```

<img width="303" height="20" alt="sudoless_komento" src="https://github.com/user-attachments/assets/d484b2cc-ac98-4292-9e5b-fd7efb51442b" />

Tallenetaan muutokset ja suljetaan nano. Siirrytään testaamaan toimiiko mikään.

<img width="777" height="158" alt="sudoless_testionnistunut" src="https://github.com/user-attachments/assets/641dd102-127d-49ae-bed7-d9d9646cef89" />

Kuvan mukaan kaikki näyttäisi olevan kondiksessa. Varmistin vielä SSH-yhteyden toimivuuden ja testasin sen jälkeen sudoa. `sudo -k` nollaa sudon muistaman salasanan, joten kuvassa ei ole voinut huijata.

### Relevantti lähde

Karvinen, T, 2026. Sudo without password. Luettavissa: https://terokarvinen.com/passwordless-sudo/. Luettu 3.11.2026


## b.) Antero (leo2)

***********************************************************

Tehdään uudelle käyttäjälle uusi hakemisto ansibleen.

```
ansible$ mkdir roles/leo2/tasks
```

<img width="263" height="201" alt="kuva" src="https://github.com/user-attachments/assets/7cfdb385-c4e3-4928-a5c8-fa66c8d680de" />

Ohjeita seuraamalla tehdään uusi YAML-tiedosto. 

```
tasks$ micro main.yml
```
Lisätään sinne kuvan mukaiset asetukset.

<img width="765" height="298" alt="passwordless_main yml" src="https://github.com/user-attachments/assets/6dcf7ccb-c316-4d95-ae9c-c088a928b6a8" />

Kiroillaan ja ihmetellään vartti mikä on pielessä kun ei toiminut. Koitetaan `lookup` ja haetaan SSH-avain suoraan hakemistosta. Rupeaa kaikki rullaamaan, virhe taisi olla SSH-avaimen saamisessa (tod. näk. kirjoitusvirhe)


<img width="489" height="280" alt="passwordless_vitunidiootti" src="https://github.com/user-attachments/assets/febc1534-e780-4ec3-8197-cdc958253d72" />


### Relevantit lähteet

Karvinen, T, 2026. Passwordless Sudo with Ansible. Luettavissa: https://terokarvinen.com/passwordless-sudo-with-ansible/. Luettu 3.11.2026

Tekoälyltä (Gemini Pro) kysytty mikä on tiedostossa voinut olla pielessä

## c.) Package

Asennetaan kaksi eri pakettia ansiblella. 

Tehdään uusi rooli. Lisätään myös rooli `site.yml` tiedostoon.

```
ansible$ mkdir roles/packages/tasks/
```

ja sinne `main.yml` tiedosto, jonne määritetään tehtävät.

<img width="300" height="132" alt="kuva" src="https://github.com/user-attachments/assets/cb4f1021-e3e2-4cc6-8d3e-8106da9450ee" />

Lopussa ajetaan `ansible-playbook site.yml -K` ja katsotaan mitä tulee.

<img width="955" height="563" alt="kuva" src="https://github.com/user-attachments/assets/4bf13786-22db-4a91-8625-7cae5f64efbe" />

Kaikki näyttää olevan ok. Lopuksi testataan. 

Kokeillaan ensimmäistä pakettia komennolla `sl`.

<img width="693" height="544" alt="lataus_juna" src="https://github.com/user-attachments/assets/86ac7e3c-9013-48e6-95e9-d48cc8bc7d67" />

sl, eli Steam Locomotive näyttäisi toimivan kuten pitäisikin. Seuraavaksi testataan cowsay komennolla `cowsay (teksti)`.

<img width="491" height="197" alt="latauksetjees_lehma" src="https://github.com/user-attachments/assets/9c0cea13-468c-4b31-a39b-8e69d79e9dd4" />

Näyttää Mansikki puhuvan asiaa. Lataukset ovat onnistuneet.

### Relevantti lähde `ansible-doc apt`

## d.) File

Ollaan laiskoja ja lisätään juuri tehtyyn `/roles/packages/tasks/main.yml` perään uusi koodinpätkä.

```
- name: Orja luo useamman rivin tiedoston
  ansible.builtin.copy:
    dest: "/tmpt/esimerkki.txt/
    content: |
      Hei.
      Toivon että tämä toimisi ensimmäisellä yrityksellä.
      Kiitos.
    owner: "leo"
    group: "leo"
    mode: "0644"
```
Nyt tiedosto näyttää tältä. Poistin myös viime osassa ladanneeni paketit. `state: present` -> `state: absent`

<img width="618" height="373" alt="orja_main yml" src="https://github.com/user-attachments/assets/9f89f553-0255-4fb9-80e8-23bb9dc8e580" />

Testataan toimiiko ensimmäisellä yrityksellä ajamalla `ansible-playbook site.yml -K`

<img width="600" height="137" alt="orja_toimi" src="https://github.com/user-attachments/assets/ecb28d16-f18b-4ed8-a33e-ab0e56adf908" />

Hurraa! Toimi heti. Kuvassa näkyvät oikeudet `-rw-r--r--` menee näin:

read = 4  write = 2  execute = 1

- `rw-` omistaja: Lukuoikeus sekä kirjoitusoikeus. 4 + 2 = 6
- `r--` ryhmä: Ryhmään kuuluvat voivat lukea. 4
- `r--` muut: Sama kuin ryhmä, lukuoikeus. 4

### Relevantti lähde: `ansible-doc copy`

## e.) Jotain muuta

Päätin asettaa itselleni MOTD eli Message of the Day. Sen pitäisi tulla aina kun kirjautuu sisään tai vaihtaa käyttäjää. Ideoiden palloitteluun käytin tekoälyä.

Käytän tässä tehtävässä moduulia `ansible.builtin.blockinfile`. Se omaa valmiiksi olemassa olevasta tiedostot osan itselleen ja kirjaa siihen sen mitä käsketään. Toinen vaihtoehto olisi käyttää `copy` moduulia, mutta se tekisi aina uuden tiedoston.

```
- name: Aseta MOTD
  ansible.builtin.blockinfile:
    path: /etc/motd
    block: "Hyvää huomenta, hyvää päivää sekä hyvää yötä!"
    state: present
```

<img width="595" height="98" alt="MOTD_KOODI" src="https://github.com/user-attachments/assets/fe914c1d-da91-4a4f-83fa-687cfe387cb3" />

Ajetaan `ansible-playbook site.yml -K` ja testataan toimiiko mikään. Vaihdetaan käyttäjää ja katsotaan tuleeko motd näytölle.

<img width="801" height="457" alt="MOTD" src="https://github.com/user-attachments/assets/9c30754f-f120-4447-b40a-30332d35dd1d" />

Näyttäisi olevan kohdillaan. Kokeilu onnistui! Huomasin että `blockinfile` tulostaa myös `#  BEGIN ANSIBLE MANAGED BLOCK` sekä `# END ANSIBLE MANAGED BLOCK`. Niitä saisi muokattua `marker_begin` ja `marker_end` parametreillä.

## Lähteet

Karvinen, T, 2026. SSH public key - Login without password. Luettavissa: https://terokarvinen.com/ssh-public-key-login-without-password/. Luettu 3.11.2026

Karvinen, T, 2026. Sudo without password. Luettavissa: https://terokarvinen.com/passwordless-sudo/. Luettu 3.11.2026

Karvinen, T, 2026. Passwordless sudo with Ansible. Luettavissa: https://terokarvinen.com/passwordless-sudo/. Luettu 3.11.2026

ansible.builtin, 2026. Luettavissa: https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/index.html. Luettu 3.11.2026

ansible.builtin.blockinfile, 2026. Luettavissa: https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/blockinfile_module.html. Luettu: 4.11.2026

Tekoälyä (Gemini Pro) käytetty idean löytämiseen e.) osiossa sekä ongelmanratkaisussa b.) osiossa. 

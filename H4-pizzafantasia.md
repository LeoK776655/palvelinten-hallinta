# h4 Pizza Fantasia

## Lue ja Tiivistä

### Karvinen 2023: Configuration Management of Distributed Systems over Unreliable and Hostile Networks

#### Size and Complexity of some DSLs

- Saltilla on 510 eri tilafunktiota joiden dokumentaatio on yli 75000 sanaa pitkä
- Puppetilla on vakiona 113 funktioita
- Puppetin rakenne on erilainen muista yleisistä ohjelmointikielistä

DSL näyttää skaalautuvan laajasti.

#### Use of DSL Functions in Case Configuration

- Pieni määrä funktioita riittää vaikka kokonaisuus olisi laaja
- Demonit asennetaan usein käyttämällä package-file-service kaavaa
- Asiakassovellukset käyttävät pelkkää package-file

#### Dependencies Between Main Functions

- On tärkeää että tila tulee olla idempotentti
- Yleisimmät funktiot voidaan jakaa eri kategorioihin
- Linuxissa lähes kaikki asetukset tallennetaan pelkkänä tekstinä

### a) Demonin asennus

Teen tehtävän demonin MariaDB kanssa. 

Asennetaan demoni.

```
sudo apt update
sudo apt install mariadb-server
```

Lataus on onnistunut. Tarkistetaan onko demoni päällä ja testataan luomalla uusi database.

<img width="793" height="262" alt="kuva" src="https://github.com/user-attachments/assets/65a372ac-9061-43af-8121-6de30eae4395" />

<img width="635" height="206" alt="kuva" src="https://github.com/user-attachments/assets/0a84577d-8e76-4ab0-b468-068889d8ace1" />

Kuvista voi nähdä että demoni on päällä ja uuden databasen luominen onnistui. Demoni vaikuttaisi olevan toiminnassa.

### b) Automatisointi

Luodaan uusi rooli ansible kansioon `mariadb`. Roolin alle luodaan alihakemisto `tasks` ja sinne YAML tiedosto `main.yml`.

Lisätään myös uusi rooli `site.yml` tiedostoon.

<img width="420" height="171" alt="kuva" src="https://github.com/user-attachments/assets/f91c345e-d7a2-45c9-87e2-214af390eea3" />

Kirjoitetaan sinne alla oleva koodinpätkä, jotta MariaDB-server asentuu kun ajetaan playbook uudella koneella ensimmäistä kertaa.

<img width="628" height="77" alt="h4_automatisointi" src="https://github.com/user-attachments/assets/51ff123f-24ed-4ca1-9f3b-1b17e142f205" />


Testataan toimiiko playbook. Ensiksi poistetaan MariaDB jotta ansiblella olisi jotain ladattavaa.

```
sudo apt purge mariadb-server
```
Kun demoni on poistettu voidaan suorittaa ansiblen playbook 

```
ansible-playbook -K site.yml
```

<img width="779" height="161" alt="kuva" src="https://github.com/user-attachments/assets/2febf542-f973-4317-907d-644ecf40920e" />

Muutoksia tulee yksi. Testataan onko demoni nyt ladattuna.

```
sudo mariadb -e "SHOW DATABASES;"
```

<img width="702" height="166" alt="h4_mariaisback" src="https://github.com/user-attachments/assets/c2ce3b49-e0ed-48df-93ef-bd02309660ac" />

Demoni on palannut palvelimelle. Komento toimii.

### c) Asetusten muutokset

Asetuksena vaihdan demonin porttia. Kysyin tekoälyltä mitä tiedostoa minun pitää muokata jotta portti saadaan vaihdettua. Vastauksena sain `/etc/mysql/mariadb.conf.d/50-server.cnf`.

Kopioidaan kyseinen tiedosto `ansible/roles/mariadb/files/`. Avataan se ja lisätään sinne toivottavasti oikeaan kohtaan `port = 3333`.

<img width="654" height="258" alt="h4_porttimuutos" src="https://github.com/user-attachments/assets/4b2b0f4e-8855-4ffd-b601-45c4458c6edf" />

Lisätään suoritettavaan YAML-tiedostoon kyseinen koodinpätkä. 

<img width="645" height="231" alt="kuva" src="https://github.com/user-attachments/assets/1ec8bca1-22ff-4712-bc4b-f27e5899ad32" />

Tehdään handleri jotta asetusten muutosten jälkeen demoni käynnistetään uudelleen ja asetusten muutokset astuvat voimaan.

<img width="611" height="92" alt="kuva" src="https://github.com/user-attachments/assets/1a0b76ca-a187-4678-a87b-7075ee1eb8c8" />

Testataan toimiiko mikään. Ajetaan playbook. `ansible-playbook -K site.yml`

<img width="799" height="194" alt="h4_asetusmuutokset" src="https://github.com/user-attachments/assets/45d70edd-d746-4c5c-a658-7a25b9d08a44" />

Muutoksia on tapahtunut muutama. Katsotaan onko demonin portti muuttunut.

```
sudo ss -tulpen | grep 3333
```

<img width="874" height="118" alt="h4_3333" src="https://github.com/user-attachments/assets/106e0e00-b89a-406c-a379-c19c2a9ff8f6" />

Kuvassa näyttäisi olevan demoni hengailemassa kyseisellä portilla. Testaus on onnistunut.

### d) Rikotaan.

Poistetaan demoni `sudo apt purge mariadb-server` jotta pääsemme testaamaan toimiiko asennus ja asetusten vaihto nollasta.

<img width="809" height="348" alt="kuva" src="https://github.com/user-attachments/assets/60b64479-e138-4100-ac2c-f9bcfe82db02" />

Ajetaan playbook jotta saadaan demoni takaisin.

<img width="873" height="256" alt="kuva" src="https://github.com/user-attachments/assets/2894799b-f182-4f05-8044-d6bfbcb70fb1" />

Muutoksia on taas tapahtunut. Tarkistetaan ensiksi onko demoni ladattuna ja sen jälkeen tarkastetaan onko porttimuunnos mennyt läpi.

<img width="877" height="451" alt="kuva" src="https://github.com/user-attachments/assets/2244801c-ac6b-4b3f-b1bc-8bec5c690298" />

Demoni on asennettu ja asetusten muutos on onnistunut. Kaikki vaikuttaisi toimivan kuten pitääkin.

### e) Idempotentti

Testataan idempotenssi. Ajetaan playbook. Jos mitään ei muutu ja tuloksena on pelkkää vihreää tila on idempotensissa.

<img width="867" height="187" alt="kuva" src="https://github.com/user-attachments/assets/40c4fae9-c169-458b-af94-30c90751084d" />

Muutoksia ei tehty ja handleri ei pyöri turhaan. Tila on idempotentti.


## Lähteet

Tekoälyä käytetty (Gemini Pro):

päättämään mihinkä demoniin tehtävä perustuu. Prompt: Anna 5 eri demonia johon voisin perustaa tämän tehtävän. Liittenä kuvankaappaus tehtävänannosta.

Kysytty mihin tiedostoon pitää tehdä muutos portiin. Prompt: Mikä on mariadbn konfiguraatiotiedosto jossa voin vaihtaa porttia?

MariaDB Documents. 2026. Luettavissa: https://mariadb.com/docs/server/mariadb-quickstart-guides. Luettu 18.11.2026

Karvinen, T. Apache installed with Ansible. 2026. Luettavissa: https://terokarvinen.com/apache-ansible/. Luettu 18.11.2026

Karvinen T. Configuration management of distributed systems over unreliable and hostile networks. 2024. Luettavissa: https://terokarvinen.com/palvelinten-hallinta/Karvinen-2023-Configuration-Management-of-Distributed-Systems-over-Unreliable-and-Hostile-Networks.pdf. Luettu 18.11.2026

# H1 Hei Ansiblen maailma

# a.) Asenna Debian 13

Asensin Debian 13 xfce-version Oraclen VirtualBoxiin. Asennusprosessi meni eteni ilman ongelmia, annoin käyttäjälle nimen sekä salasanat. Määritin oikean kielen näppäimistölle sekä käyttöjärjestelmälle. Mitään muuta asennuksessa en vaihtanut.

<img width="800" height="600" alt="debian asennus" src="https://github.com/user-attachments/assets/afcb9cd0-dfd2-49ec-beac-86993d2cca77" />


## Ongelma: asennuksen jälkeen virheilmoitus

Ongelma tuli kuitenkin vastaan virtuaalikoneen asennuksen jälkeen ensimmäisen rebootin mukana. Sain avauksen myötä kolme erilaista virheviestiä. Googlaamalla ongelmaviestit löysin muutaman mahdollisen ratkaisun.

<img width="1280" height="800" alt="ongelma debian asennus" src="https://github.com/user-attachments/assets/4875f71d-4e38-4924-8fd7-bada7d863d0b" />


Ratkaisu 1: Anna Debianille muutama minuutti aikaa, jos boottaus onnistuisi. 

Lopputulos: Noin 5 minuutin odottamisen jälkeen, mikään ei ollut edistynyt.

Ratkaisu 2: Vaihda VirtualBoxin Display asetuksista virtuaalikoneen graphics controller VMSVGA -> VBoxVSGA

Lopputulos: Mikään ei muuttunut, sama ongelma jatkui.

Ratkaisu 3: Nosta virtuaalikoneen Video Memory 16MB -> 128MB.

Lopputulos: Toimi. Virtuaalikone väläytti avauksessa samoja viestejä arviolta noin sekunnin, mutta lähti siitä rullamaan.

## Ongelma: "leo is not in sudoers file.

<img width="1280" height="800" alt="sudo-ongelma" src="https://github.com/user-attachments/assets/6f16aeae-dc6b-49e1-b0b7-dd0887167cfc" />

Googlettamalla löytyi ratkaisu.

Ratkaisu: Vaihdoin käyttäjän terminaalissa rootiksi, jonka avulla lisäsin käyttäjäni "leo" sudoereihin komennolla "sudo adduser leo sudo". Lopussa vaihdoin taas käyttäjää komennolla "su - leo" ja tuplachcheckkasin asian komennolla "whoami". 

Lopputulos: Käyttäjä "leo" lisättiin sudoereihin onnistuneesti.



# b.) Asenna SSH-demoni

Seuraavan tehtävän tarkoitus oli asentaa SSH-demoni, sekä testata sen toimivuus. Toimivuutta testasin ottamalla yhteyden localhostiin.

<img width="854" height="273" alt="sshtesti2" src="https://github.com/user-attachments/assets/7fd71859-8fa8-41a5-a94a-2477e9ff14cc" />


# c.) Automatisoi SSH-kirjautuminen

Seuraavaksi on tarkoitus automatisoida ssh-kirjautuminen käyttämällä julkista avainta. Seurasin suoraan Karvisen ohjeita SSH public key - Login without password artikkelista. Asennus ja testaus tapahtui ilman ongelmia. Kuvat valitettavasti otettu ensimmäisen asennuksen jälkeen.

<img width="536" height="111" alt="sshkeygen olemassa" src="https://github.com/user-attachments/assets/d15a362d-728e-410b-b6c5-623a196f4a5f" />


# d.) Hei Ansible

Viimeisen tehtävän ideana oli saada tuloste: "Hei maailma!" tehtyä ansiblella. Omassa tiedostossa lukee perinteisen hei maailman tilalla "See you on Thursdays!". 

Ensin tehtiin "hosts.ini" tiedosto, joka määrittää minulle kaikki hostit jota voin kontrolloida. 

<img width="321" height="60" alt="hosts ini sisältö" src="https://github.com/user-attachments/assets/59b4784d-9fef-4bca-bf75-8adc0d3d309f" />

Testasin ansiblea sekä hostin toimivuutta alapuolen kuvassa olevalla komennolla.

<img width="1276" height="321" alt="ansiblekomento" src="https://github.com/user-attachments/assets/46bff2ff-a6c3-4842-b7fb-cb7acc8aa360" />

Häiritsevä python version varoitusteksti saatiin poistettua lisäämällä "hosts.ini" tiedostoon tietty python komento. Kuvassa näkyvä [all:vars] varmistaa sen että komento pätee jokaiseen hostiin. Tuleviin sekä nykyiseen.

<img width="516" height="155" alt="pythonhavitetty" src="https://github.com/user-attachments/assets/240128f3-6ad7-4058-99b2-56a2d7223c06" />

Jotta jokaiseen ansiblen komentoon ei tarvitsisi kirjoittaa "-i hosts.ini", tehdään uusi konfiguraatio-tiedosto ansiblelle. Lisätään sinne alla olevan kuvan mukainen sääntö.

<img width="344" height="65" alt="ansiblecfg" src="https://github.com/user-attachments/assets/9cf0eb58-477d-4816-9417-ed36995cd8a6" />

Seuraava askel on määrittää mitkä tietokoneet saa mitkäkin konfiguraatiot. Koska haluamme että kaikki tietokoneemme saavat tietyn roolin, käytetään "- hosts: all". Olin kirjoittanut koodin hieman vinoon, korjasin sen mutta huomaan myös sen että roolia "hello" ei ole edes olemassa.

<img width="803" height="366" alt="ansibleongelma1" src="https://github.com/user-attachments/assets/39f75fbe-dd65-4c86-b207-043706c82b4c" />

Seuraavaksi tehdään rooli "hello" ja annetaan sille tehtäväksi tehdä tiedosto "hello-ansible" /tmp/ hakemistoon orjatietokoneelle.

<img width="463" height="78" alt="rooli hello" src="https://github.com/user-attachments/assets/e07e740b-1990-474e-ac1f-e61756ce6770" />

Testataan uusinta komentoa.

<img width="1283" height="229" alt="site yml toimii" src="https://github.com/user-attachments/assets/7c3a8da2-3a0f-4b32-b501-77ebf631afb5" />

<img width="530" height="92" alt="hei maailma" src="https://github.com/user-attachments/assets/3161acce-1c0e-4b2b-b928-3c72887ef274" />

Kaikki toimii. Minun "hei maailmani" on tulostettu onnistuneesti. Tehtävässä seurasin vain tarkasti Karvisen ohjeita Hello Ansible artikkelista.

# x) Lue ja tiivistä

## Hello Ansible

- Ansible on konfiguraatiotyökalu, tarkoitettu helpompaan hallintaan.
- Ansiblella tehdään koodia infrasta (IaC).
- "hosts.ini" Määrittää ansiblelle kaikki hallittavat tietokoneet
- "ansible.cfg" määrittää itse ansiblelle omat asetukset ja sääntönsä

Miksi YAML-tiedostot ovat niin tarkkoja sisennysvirheiden kanssa?

## SSH public key - Login without password 

- Salasanaton kirjautuminen on helpompaa ja turvallisempaa
- Se mahdollistaa automaation esim. ansible-komennoille
- Luodaan omalla koneella avainpari ja kopioidaan se kohdepalvelimelle

Miksi salasanaton kirjautuminen on turvallisempaa?


# Lähteet

Karvinen, T 2026. SSH public key - Login without password. Luettavissa: https://terokarvinen.com/ssh-public-key-login-without-password/#install-ssh. Luettu 28.10

Karvinen, T. 2026. Hello Ansible. Luettavissa: https://terokarvinen.com/hello-ansible/. Luettu 28.10.2026

Punnala, T. 2025. H1-installing-linux. Luettavissa: https://github.com/punnalathomas/linux-course/blob/main/h1-installing-linux.md Luettu 28.10.2026

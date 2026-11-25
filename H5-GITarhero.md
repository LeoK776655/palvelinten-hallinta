# H5 GITar hero
## x) Lue ja tiivistä

- Git toimii melkein pelkästään omalta koneelta, yhteyttä tarvitaan vain esim. synkronisointiin.
- Git tallentaa jokaisen commitin snapshottina, jolloin muuttumattomat tiedostot eivät tule kopioiduiksi uudelleen.
- Pitää olla varovainen sillä jos tiedostoja voi helposti menettää `git reset --hard` komennolla

- `git add --all` Käskee tekemään kaikki muutokset koko varastosta
- `git commit` Tallentaa valmiina olevat tiedostot pysyviksi versioiksi paikalliseen varastoon
- `git pull` Hakee uusimmat muokkaukset etävarastosta ja yhdistää ne paikallisesti
- `git push` Lähettää paikalliset commitit etävarastoon
- `&&` Toinen komento menee läpi vain jos ensimmäinen menee läpi
- `;` Ajaa seuraavat komennot tämän merkin jälkeen riippumatta siitä mitä aikaisemmille tapahtui

## a) Online

Asennetaan Git virtuaalikoneelle. 

```
sudo apt update
sudo apt install -y git
```

Tehdään GitHubiin uusi varasto. Painetaan oikealla ylhäällä olevaa + merkkiä ja sieltä `new repository`.

<img width="539" height="312" alt="h5_uusrepo" src="https://github.com/user-attachments/assets/ebb21d70-f229-463a-906e-438f6029a9b9" />

Määritetään uudelle repolle nimi, pieni kuvaus sekä lisätään `READme.md` sekä `GNU General Public License 3` lisenssi. Nimessä sekä kuvauksessa piti olla sana `sunshine` jossain.

<img width="885" height="781" alt="h5_valmisrepo" src="https://github.com/user-attachments/assets/9efbd82a-0e6c-4d79-a92b-dae2ad044924" />

## b) Dolly, kloonataan GitHub varasto

Katsotaan oma julkinen ssh-avain ja lisätään se GitHuB käyttäjälle. Testataan samalla toimiiko se.

<img width="993" height="514" alt="h5_ssh_gittiin" src="https://github.com/user-attachments/assets/72056af1-4099-4fa6-a1e7-875b59a2456a" />

<img width="925" height="42" alt="g5_git_sshyhteyscheck" src="https://github.com/user-attachments/assets/29b5f11c-a1aa-4092-8ff8-0c787c5033fe" />

Kloonataan juuri a) kohdassa tehty repo. 

<img width="755" height="149" alt="h5_gitkloonattu" src="https://github.com/user-attachments/assets/b6471ae0-1ce7-465c-b634-484d9f88fe8b" />

Siirrytään oikeaan hakemistoon `cd sunshine-enihsnus` ja testataan muokata `READme.md` tiedostoa. 

```
echo "Tämä testimuutos tehdään virtuaalikoneelta." >> READme.md
```

Sitten lisätään kaikki tiedostoihin tehdyt muutokset push-jonoon. Testataan samalla voiko virtuaalikoneelta lisätä commit viestin. `git push` jotta muutokset tulevat läpi.

<img width="796" height="245" alt="h5_gitcommitjne" src="https://github.com/user-attachments/assets/16dcc941-14b8-41f6-8445-c575e86580ca" />

Näyttäisi toimivan, katsotaan vielä omasta reposta.

<img width="944" height="584" alt="kuva" src="https://github.com/user-attachments/assets/cbe8fc01-d50f-4f55-8076-9593e65b1bb4" />

Muutokset ovat menneet läpi. Kaikki ok.

## c) Doh! rikotaan jotain

Lisätään `echo` komennolla jotain tyhmää ja turhaa tekstiä `READme.md` tiedostoon.

<img width="801" height="363" alt="h5_tyhmämuutos" src="https://github.com/user-attachments/assets/33783a0b-bee8-4abc-952c-9c48e594df8a" />

Päätetään ettei haluta näitä muutoksia mihinkään. Ajetaan komento `git reset --hard`

<img width="787" height="250" alt="h5_gitreset" src="https://github.com/user-attachments/assets/515a432e-a171-469b-85a9-687d2ff77a4a" />

Se tuhosi tyhmät muutokset, kaikki on taas ok. 

## d) Tukki. tarkastellaan logeja

Avataan Gitin logit `git log`

<img width="822" height="378" alt="h5_gitlokit" src="https://github.com/user-attachments/assets/9485669c-4020-4059-9605-f47cb366582b" />

Logeista näkyy juuri ne muutokset joita olen puskenut virtuaalikoneelta tässä tehtävässä. Nimi ja sposti näyttää olevan ihan oikein. Date näyttää moneltako muutokset on tehty.

## e) Gitansible

Siirrytään ansiblen hakemistoon `cd ansible`. Sieltä tehdään uusi tyhjä `.git` varasto. 

```
git init
git add --all
git commit -m "Testi jne"
```

Muokataan jotain olemassa olevaa YAML tiedostoa vähän jotta voidaan testata Gittiä. Päätin laittaa pienen viestin `hello` roolin `main.yml` tiedostoon.

<img width="306" height="67" alt="kuva" src="https://github.com/user-attachments/assets/b9d2f130-ff86-475c-b5dc-b1047379e468" />

Ajetaan playbook `ansible-playbook site.yml -K`

Todistetaan muutokset (niitä oli 1, ei kuvaa)

Ajetaan `git diff` ja katsotaan huomataanko muutokset.

<img width="807" height="269" alt="h5_ansiblegitmuutos" src="https://github.com/user-attachments/assets/6c464637-9f89-41a2-8fe8-42fb87313cf1" />

Muutokset ovat näkyvissä, ajetaan vielä `git add --all` ja perään `git commit`

<img width="626" height="59" alt="h5_vikacommit" src="https://github.com/user-attachments/assets/fbb51095-a698-4fc0-8096-c88599b9b462" />

Muutokset ovat menneet läpi, kaikki ok.

## Lähteet

Git, Luettavissa: https://git-scm.com/book/en/v2. Luettu 25.11.2026

Git - What is Git?, luettavissa: https://git-scm.com/book/en/v2/Getting-Started-What-is-Git%3F. luettu 25.11.2026

Git - push, luettavissa: https://git-scm.com/docs/git-push luettu 25.11.2026

Git - pull, luettavissa: https://git-scm.com/docs/git-pull luettu 25.11.2026

Git - commit, luettavissa: https://git-scm.com/docs/git-commit luettu 25.11.2026

git - add, luettavissa: https://git-scm.com/docs/git-add luettu 25.11.2026

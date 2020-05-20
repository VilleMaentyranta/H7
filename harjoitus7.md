# Harjoitus 7.

Laite, jolla tehtävät suoritettiin: Apple MacBook Pro – MacOS Catalina 10.15.4 (19E287) Oracle VirtualBox 6.1.4 Xubuntu 18.04.4 + guest additions 2048 MB RAM

## Tehtävät:

### h7 Oma moduuli

b) Oma moduli (iso tehtävä). Ratkaise jokin oikean elämän tai keksitty tarve omilla tiloilla/moduleilla. Voit käyttää Salttia tai muuta valitsemaasi modernia keskitetyn hallinnan ohjelmaa. Raportoi modulisi tarkoitus, koodi ja testit

## B) Oma moduuli

Halusin aloittaa ihan puhtaalta pöydältä, joten päätin luoda uuden virtuaalikoneen Tero Karvisen ohjeita käyttäen ([Linkki](http://terokarvinen.com/2020/remote-learning-tools-for-my-courses/)). Asensin siihen myös Guest Additionssit (ohjeet edellisessä linkissä).

### Salt 
Kun kone oli asennettuna asensin sille Saltin. Asennus tapahtui samalla tavalla kuin Harjoitus 1. ([Linkki](https://vimlinux.wordpress.com/2020/04/07/harjoitus-1/)).


Ajetaan peruskomento 
```
    sudo apt-get update
```
Asennetaan virtuaalikoneelle salt-master ja salt-minion komennoilla
   ```
    sudo apt-get -y install salt-master
    sudo apt-get -y install salt-minion
```
Selvitetään koneen IP-osoite, jotta salt-minionin konfiguraatiotiedostoa voidaaneditoida sisältämään Masterin IP 
```
    hostname -I
    sudoedit /etc/salt/minion

```

Lisätään konfiguraatioon Masterin IP ja nimi Minionille 
```
master: 10.0.2.15 
id: orja  
```
Käynnistetään salt-minion uudestaan sudo systemctl restart salt-minion.service Hyväksytään juuri luotu minion sudo salt-key -A y Testataan Masterin ja Minionin välisen yhteyden toimivuus sudo salt ’*’ cmd.run ‘whoami’

### Apache2 
Seuraavaksi päätin asentaa Apachen. Apache asennettiin Harjoitus 2. varten, joten seurasin sen ohjeita ([Linkki](https://vimlinux.wordpress.com/2020/04/14/harjoitus-2/)).


Asenna Apache2
```
    sudo apt-get install apache2
```
Tehdään reikä palomuurin porttiin 80
```
    sudo ufw allow 80/tcp
```
Avataan verkkoselaimella localhost-sivusto ja varmistetaan Apache2 toimivuus
Testataan localhost sivun muokkaamista.
```
    echo ”hello world” |sudo tee /var/www/html/index.html
```
Päivitetään verkkosivu ja havaitaan muutokset.
Luodaan nyt Apachelle oma kansio /srv/salt -polkuun.
```
    sudo mkdir /srv/salt
    sudo mkdir /srv/salt/apache
```
Apache kansion sisään luodaan init.sls tiedosto
```
    sudoedit /srv/salt/apache/init.sls
```
Tiedostoon määritykset Tero Karvisen [sivuilta](http://terokarvinen.com/2018/apache-user-homepages-automatically-salt-package-file-service-example)
Seuraavaksi luodaan default html -sivusto, jonne kirjoitetaan haluttu teksti

    sudoedit /srv/salt/apache/default-index.html 

Viedään nämä asetukset minioneille

    sudo salt ‘*’ state.apply apache


### SSH 
Seuraavaksi asennetaan SSH, jotta voimme ottaa koneelle SSH yhteyden tarvittaessa. Apuna käytin Tero Karvisen tarjoamaa ohjetta ([Linkki](http://terokarvinen.com/2018/pkg-file-service-control-daemons-with-salt-change-ssh-server-port)).

Luodaan SSHD kansio

    mkdir /srv/salt/sshd
Luodaan /srv/salt/sshd polkuun init.sls tiedosto
 
    cd /srv/salt/sshd
    sudoedit init.sls
Tiedoston sisään kirjoitetaan seuraavat tiedot
```
openssh-server: pkg.installed /etc/ssh/sshd_config: file.managed:

-   source: salt://sshd/sshd_config sshd: service.running:
-   watch:
    -   file: /etc/ssh/sshd_config 
   ```
Seuraavaksi luomme sshd_config tiedoston

```
sudoedit sshd_config

```
```
#DON'T EDIT - managed file, changes will be overwritten
Port 8888 Protocol 2 HostKey /etc/ssh/ssh_host_rsa_key HostKey /etc/ssh/ssh_host_dsa_key HostKey /etc/ssh/ssh_host_ecdsa_key HostKey /etc/ssh/ssh_host_ed25519_key UsePrivilegeSeparation yes KeyRegenerationInterval 3600 ServerKeyBits 1024 SyslogFacility AUTH LogLevel INFO LoginGraceTime 120 PermitRootLogin prohibit-password StrictModes yes RSAAuthentication yes PubkeyAuthentication yes IgnoreRhosts yes RhostsRSAAuthentication no HostbasedAuthentication no PermitEmptyPasswords no ChallengeResponseAuthentication no X11Forwarding yes X11DisplayOffset 10 PrintMotd no PrintLastLog yes TCPKeepAlive yes AcceptEnv LANG LC_* Subsystem sftp /usr/lib/openssh/sftp-server UsePAM yes
```
Ajetaan tila minionille

```
sudo salt '*' state.apply sshd

```

Testataan ottaa SSH yhteys

```
ssh -p 8888 ville@10.0.2.15

```

### Curl ja Tree

Seuraavaksi asennan Curlin ja Treen. Apuna käytin Saltin ohjeita ([Linkki](https://docs.saltstack.com/en/getstarted/fundamentals/states.html)).

Luodaan /srv/salt -polkuun programs-kansio

    sudo mkdir /srv/salt/programs
Sen sisään luodaan init.sls-tiedosto

    cd /srv/salt/programs
    sudoedit init.sls
Init.sls tiedoston sisään kirjoitamme seuraavan tekstin
```
install_network_packages: pkg.installed: - pkgs: - tree - curl
```
Ajetaan minionille
    sudo salt '*' state.apply programs



Testataan Curlia Tero Karvisen sivulle komennolla: curl terokarvinen.com 
Testataan Treetä /srv/salt -polkuun komennolla: tree /srv/salt Molemmat toimii halutulla tavalla.

### Git

Seuraavaksi asensin Gitin. Gitin asennus tehtiin aiemmin harjoitus 3. ([Linkki](https://vimlinux.wordpress.com/2020/04/21/harjoitus-3/)).

Asennetaan git

    sudo apt-get install git -y

Kirjauduin sen jälkeen Githubiin ja loin siellä uuden repositorion. Laitoin nimeksi H7, tein siitä julkisen ja valitisin, että repositorioon luodaan README tiedosto. Lisenssiksi valitsin GNU GPL v3.0. Tämän jälkeen palataan terminaaliin.


Määritetään config tiedot gittiin
```
    git config –global user.email ”omasäpo”
    git config –global user.name ”nimesi”
```

Kloonataan Githubista repositorio komennolla

```
git clone https://github.com/VilleMaentyranta/H7.git
```

Siirrytään kansioon

```
cd H7
```

Luodaan harjoitus7.md tiedosto ja muokataan sitä haluamalla tavalla.

```
nano harjoitus7.md
```
Annetaan seuraava komento, joka päivittää tiedoston Githubiin
```
git add . && git commit; git pull && git push
```

Kurkataan Githubista, että tiedosto on varmasti päivittynyt sinne ja sehän on.

#### Orjakone

Seuraavaksi loin uuden koneen virtualboxiin samalla tavalla kuin aiemmin.  
Kun kone oli valmis, yhdistin sen samaan verkkoon Master koneen kanssa.

-   Orjakoneella
    -   `sudo apt-get update`
    -   `sudo apt-get install salt-minion`
        -   `master: 10.0.2.15`
        -   `id: uusiorja`
-   Käynnistetään minion uudelleen
    -   `sudo systemctl restart salt-minion.service`

-   Masterilla etsitään löytyykö uusia orjia ja hyväksytään se
    -   sudo salt-key -A
    -   Y
    
Nyt kun meillä on uusiorja hallinnassa, voimme kokeilla asentaa sille suoraan kaikki aiemmin asennetut sovellukset `sudo salt '*' state.highstate` -komennolla.

Kaikki näyttää menneen läpi, joten kokeillaan nyt vielä orjalla että kaikki toimii ja nehän toimii.

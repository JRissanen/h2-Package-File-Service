# h2-Package-File-Service

## Salt Quickstart - Salt Stack Master and Slave on Ubuntu Linux

Saltin avulla on mahdollista hallita tuhatta tietokonetta.</br>
Vain Master-koneella täytyy olla julkinen palvelin ja tunnettu osoite.</br>
Slave-koneiden sijainnilla ei ole hallinnan kannalta merkitystä, ne voivat olla esimerkiksi:
* NAT:in takana
* Palomuurin takana
* Tuntemattomassa osoitteessa

__Master- ja Slave-koneiden asentaminen ja käyttö.__</br>
Seuraavissa komennoissa kun viitataan Master-koneella tehtäviin komentoihin käyttäjä on "master$" ja kun viitataan Slave-koneeseen käyttäjä on "slave$".</br>

Master-koneen asentaminen:</br>
`master$ sudo apt-get update`</br>
`master$ sudo apt-get -y install salt-master`</br>

Slave-koneen asentaminen:</br>
`slave$ sudo apt-get update`</br>
`slave$ sudo apt-get -y install salt-minion`</br>

Jotta yhteys Master- ja Slave-koneiden välillä toimisi, Slave-koneen tulee tietää Master-koneen ip-osoite.</br>
Slave-koneille on myös mahdollista määrittää nimi (id). Jos nimeä ei määritä, se muodostuu automaattisesti käyttäjän nimestä.</br>
`master$ hostname -I`</br>
`10.0.0.88`</br>

`slave$ sudoedit /etc/salt/minion`</br>
`master: 10.0.0.88`</br>
`id: Slave1`</br>

Asetusten jälkeen toimivuuden kannalta on olennaista uudelleenkäynnistää Slave-kone sekä hyväksyä Slave-koneen avain Master-koneella.</br>
`slave$ sudo systemctl restart salt-minion.service`</br>

`master$ sudo salt-key -A`</br>
`Unaccepted Keys:`</br>
`Slave1`</br>
`Proceed? [n/Y]`</br>
`Key for minion Slave1 accepted.`</br>

Tämän jälkeen on hyvä tarkistaa onko yhteys Master- ja Slave-koneiden välillä asetettu oikein, ajamalla jokin komento.</br>
`master$ sudo salt '*' cmd.run 'whoami'`</br>
`Slave1:`</br>
`root`</br>

(Karvinen,T. 2018.)

## Understanding SaltStack

__Salt Approach__

* Real-Time Communication
  * Slave-koneet saavat komennot samanaikaisesti.
  * Ero 10 ja 10 000 koneen samanaikaisessa päivittämisessä ei ole suuri.
  * Salt menetelmä infrastruktuurista tiedonkeräämiseen on tehdä reaaliaikaisia kyselyjä Slave-koneille.
  * Kyselyjä on mahdollista tehdä tuhansia muutamissa sekunneissa.

* No Freeloaders!
  * Slave-koneet tekevät omat tehtävänsä.
  * Ohjeet tehtäviin tulevat Master-koneelta.
  * Slave-koneet päättävät itse ohjeiden perusteella vastaavatko komentoihin.
  * Kaikkien Slave-koneiden tarvitsemat komennot ovat jo paikallisessa muistissa nopeaa vastaamista varten.

* Salt Loves To Scale
  * Salt on suunniteltu korkealla suorituskyvyllä ja skaalattavuudella.
  * Salt Master- ja Slave-koneiden välillä toimiva ZeroMQ tai pelkkä TCP, antavat Saltille suorityskyvyssä edun, kilpailijoihin verrattuna.
  * Viestit sarjallistetaan MessagePack:in avulla.
  * Salt käyttää Python Tornado:a tahdistamattomana verkkokirjastona.

* Normalize everything
  * Salt komennot ajetaan samalla tavalla Slave-koneen käyttöjärjestelmästä riippumatta.
  * Salt tiivistää jokaisen käyttöjärjestelmän yksityiskohdat, laitteistot ja järjestelmän työkalut.

* Manage everything
  * Saltia voi ajaa lähes kaikkialla, missä voi myös ajaa Pythonia.
  * Ainoa vaatimus Saltin käyttöön on, että laitee tukee jotain verkko protokollaa. (Proxy Minion System).

* Automate Everything
  * Salt Master-kone käyttää ohjeina erilaisia tiloja (state) Slave-koneille, joten järjestelmän oletusasetuksetkin on mahdollista määrittää tiloilla.

* Programming optional
  * Saltin käyttäminen ei vaadi minkään erityisen ohjelmointikielen opettelemista tai osaamista.
  * Saltia kirjoitetaan infrastruktuuri koodina.

(Saltstack. 2016.)

__Plug-ins__

* Pluggable Subsystems
  * Salt pitää sisällään yli 20 liitännäistä alijärjestelmää.
  * Yleisimpiin alijärjestelmiin kuuluu esimerkiksi:
   * Tunnistautuminen
   * Tiedostopalvelin
   * Turvallinen tietopankki
   * Konfigurointi

* Subsystem During A Job Run
  * Tehtävien ajaminen tapahtuu monen alijärjestelmän avulla.

* Salt Components
  * Jokainen Saltin komponentti on liitännäinen alijärjestelmä.

* Virtual Modules
  * Koska moni käyttöjärjestelmä on hyvin erilainen, Salt hyödyntää virtuaali moduuleja tiivistäessään käyttöjärjestelmien eri ominaisuuksia.

(Saltstack. 2016.)

__Communication__

* Architecture Model
  * Salt käyttää server-agent-mallia, eli Master- ja Slave-koneita.
  * Master-koneet lähettävät ohjeita Slave-koneille.

* Communcation Model
  * Salt käyttää publish-subscribe mallia.
  * Salt Master-kone käyttää portteja 4505 ja 4506, joiden täytyy olla auki, hyväksyäkseen saapuvat pyynnöt.

* Salt Minion Authentication
  * Ensimmäisen käynnistyksen yhteydessä Slave-kone etsii järjestelmän nimeltä salt.
  * Löytäessään saltin, Slave-kone lähettää oman julkisen avaimensa Master-koneelle.
  * Master-koneen täytyy hyväksyä avain.
  * Hyväksymisen jälkeen Master-kone antaa oman julkisen avaimensa sekä kiertävän AES-avaimen.
  * AES-avainta käytetään koneiden välisen viesetinnän salaamiseen.

(Saltstack. 2016.)

__Remote Execution__

* Remote Execution
  * Saltin alkuperäinen tarkoitus oli olla etäsuoritusta tukeva työkalu.
  * Komennolla: `salt '*' pkg.install git` pystyy asentamaan jokaiselle Slave-koneelle git:in.

(Saltstack. 2016.)

__States__

* State System
  * Tila komennot on suunniteltu toimimaan eri käyttöjärjestelmien ja alustojen välillä.
  * Kaikki kohde järjestelmät pystyvät ajamaan komennot samaanaikaan.
  * Salt hyödyntää satoja Python kirjastoja eri moduulien konfiguraatioiden hallinnassa.

(Saltstack. 2016.)

__Runners__

* Salt Runners
  * Ruuner-alijärjestelmä sisältää moduulit, jotka ajetaan Master-koneella.
  * Runner moduuleita kutsutaan komennolla: `salt-run`.

(Saltstack. 2016.)

__System Data__

* Grains
  * Hyödynnetään kun kerätään tietoa järjestelmistä.
  * Kerätään automaattisesti kun Slave-kone käynnistetään.

* Salt Pillar
  * Hyödynnetään kun toimitetaan tietoa järjestelmiin.
  * Data salataan kohde Slave-koneen julkisella avaimella.

* Salt Mine
  * Hyödynnetään kun jaetaan dataa Slave-koneiden välillä.
  * Sisältää yleisiä ohjeita, joita moni Slave-kone voi hyödyntää.

(Saltstack. 2016.)

__Python__

* Python
  * Pythonia ei tarvitse osata kirjoittaa, mutta sitä on hyvä osata lukea.
  * Jokainen Saltin liitännäinen alijärjestelmä on myös Python moduuli.
  * Kaikki moduulit sijaitsevat salt-kansiossa.
  * Python moduulin pääte on: `.py`.

(Saltstack. 2016.)

## SaltStack Fundamentals

__Demo Environment__

* Demo Environment
  * SaltStackin käytön oppii parhaiten itse tekemällä, pelkän lukemisen sijaan.
  * Kokeilut kannattaa tehdä turvallisessa ympäristössä, esim. virtuaalikoneella.

(Saltstack. 2016.)

__Execute Commands__

* Salt commands
  * Salt komentoja voi ajaa syntaksilla: `salt 'target' module.function arguments`.

(Saltstack. 2016.)

__Targeting__

* Targeting
  * Salt komennoilla on mahdollista ajaa komentoja yhdelle tai useammalle Slave-koneelle.
  * 'target'- määrittää aina kohteen.
  * Esimerkiksi komento: `salt-call '*'` kutsuu jokaista Slave-konetta.
  * Komento: `salt-call 'Slave1'` kutsuu vain Slave1-nimistä Slave-konetta.

(Saltstack. 2016.)

__Create A Salt State__

* Create A Salt State
  * Kaikki Salt tilat sijaitsevat hakemistossa: `/srv/salt`, joka pitää itse luoda.
  * Esimerkki tilasta:
   ```
  install_network_packages:
    pkg.istalled:
      - pkgs:
        - rsync
        - lftp
        - curl
  ```
  * Sisennykset ovat aina kaksi välilyötniä.
  * Tilat tallennetaan aina `init.sls`-tiedostoina.
  * Tilat, jotka ovat nimeltään: `top.sls`, mahdollistavat monen eri tilan samanaikasen ajamisen.

(Saltstack. 2016.)

## SaltStack Configuration Management - Functions

__Salt State Functions__

* Syntax
  * Tilat määritellään YAML:lla.
  * Ensimmäinen rivi on tilan nimi, eli id.
  * Seuraavalle riville tulee moduuli ja functio, joka ajetaan kutsuttaessa.
  * Lopuksi määritellään muuttujat/arvot ja ensimmäinen arvo on aina nimi.
  * Esimerkki tilan rakenteesta:
    ```
    install vim:
      pkg.istalled:
        - name: vim
    ```
  * Tiloja voi tehdä lukuisia määriä eri tarkoituksia varten.

(Saltstack. 2016.)

## Tehtävät

Asensin tehtäviä varten puhtaan Linux Ubuntu 22.04.1 LTS järjestelmän Virtual Box Versioon 6.1.40. </br>
Kaikki tehtävät on tehty edellä mainitussa ympäristössä. </br>
Puhtaan virtuaalikoneen käyttäjäksi määritin "julle" ja koneen nimeksi "julle-TestiUbuntu".</br>
Ssh-yhteyden testaamista varten käytin myös Linux Ubuntu 22.04.1 LTS Live-Server virtuaalikonetta, jonka käyttäjä on myös "julle" ja koneen nimi "julleserver". </br>
Tehtävät on tehty 8.11.2022 klo 13:00 - 23:00, sekä 9.11.2022 klo 15:30 - 23:00.

__a) 
Demonin asetukset. Säädä jokin demoni (asenna+tee asetukset+testaa) package-file-service -rakenteella.</br>
Ensin käsin: muista tehdä ja raportoida asennus ensin käsin, vasta sitten automatisoiden.</br>
Jos osaat hyvin, voit tehdä jonkin eri asetuksen kuin tunnilla.</br>
Harjoitusta varten tulee siis tehdä alusta ja raportoida samalla.__

__b) 
Asenna Salt herra-orja niin, että molemmat ovat samalla koneella. </br>
Voit tehdä ne esimerkiksi uudelle virtuaalikoneelle, niin pääset kokeilemaan puhtaalta pöydältä.

Asensin tehtävässä a) openssh-serverin ja vaihdoin ssh:n portin.</br>
Käytin mallina Tero Karvisen artikkelia: Pkg-File-Service – Control Daemons with Salt – Change SSH Server Port. </br>
Luettavissa: https://terokarvinen.com/2018/pkg-file-service-control-daemons-with-salt-change-ssh-server-port/

Ensimmäinen komentoni oli `sudo apt-get update`.</br>
Sitten asensin Salt-Masterin ja Mionionin komennoilla: `sudo apt-get install salt-master` ja `sudo apt-get install salt-minion`. </br>
¨Sitten tein hakemiston: `mkdir /srv/salt/ssh`.</br>
Loin uuden sls-tiedoston ssh kansioon komennolla: `sudoedit init.sls`, joka asentaa openssh-serverin.</br>
Kopioin sshd_config-tiedoston ssh-kansioon: komennolla: `sudo cp sshd_config /srv/salt/ssh`.

![Screenshot 2022-11-09 160255](https://user-images.githubusercontent.com/116954333/200852135-d035dfe4-93ad-4f8f-a133-8a3a75f90d88.png)

Seuraavaksi vaihdoin sshd_config-tiedostosta alkuperäisen portin 22 -> porttiin 8888. Komento oli: `sudoedit sshd_config` ja tiedostoa pystyi muokkaamaan.

Tilaa ajaessa kohtasin muutaman ongelman koskien init.sls tiedoston rakennetta, joka on ensimmäisestä kuvasta havaittavissa. </br>
Tekemäni muutokset näkee alla olevasta toisesta kuvasta, mutta piti siis ottaa turha `ssh:` tiedoston alusta pois. </br> 
Sen lisäksi piti myös korjata sisennysvirheet, ottaa `pkg.installed:` riviltä ":"-merkki pois, koska sekin oli turha. </br>
Lopuksi ongelma jonka löytämiseen meni eniten aikaa oli älytä laittaa `- source: salt://sshd_config` riville vielä oma kansioni, jossa tiedosto oli, eli ssh.

![Screenshot 2022-11-09 163326](https://user-images.githubusercontent.com/116954333/200858604-08489d51-0575-4d26-a16b-15fc8a9ba6d0.png)

Korjaustoimenpiteiden jälkeen tila kuitenkin toimi oletetusti.

![Screenshot 2022-11-09 163514](https://user-images.githubusercontent.com/116954333/200860894-b51acc05-cd7e-433f-a117-c7706d60bf4d.png)

Testasin ssh yhteyden toimivuuden käyttämällä toista virtuaalikonetta. </br>
Testausta varten molempien virtuaalikoneiden piti olla verkossa "bridged adapter" asetuksella, jotta yhteys toimisi. </br>
Käytin komentoa: `ssh -p 8888 julle@julle-TestiUbuntu`, yhteyden muodostamiseen juuri configuroidun portin 8888-kautta. </br>
Yhteys näytti toimivan, joten ajoin vielä komennon: `ls`, jotta varmistin, että samat kansiot tulivat näkyviin. Ja tulihan ne, joten homma selvä.

![Screenshot 2022-11-09 172859](https://user-images.githubusercontent.com/116954333/200871742-00dab812-4b1c-43ef-ab6f-a985d2a03c76.png)

__c) 
Aja jokin tila paikallisesti ilman master-slave arkkitehtuuria. </br>
Analysoi debug-tulostetta. 'sudo salt-call --local state.apply hellotero -l debug'__

Tarkistin ensin oman "helloworld" tiedostoni komennolla: `cat helloworld`. </br>
Sen jälkeen tulostin debug login komennolla: `sudo salt-call --local state.apply helloworld -l debug`.

![Screenshot 2022-11-09 175045](https://user-images.githubusercontent.com/116954333/200878283-da958e25-8c17-4c4b-aee3-c53229624189.png)



## Lähteet

Karvinen, T. 2018. Salt Quickstart - Salt Stack Master and Slave on Ubuntu Linux. </br>
Luettavissa: https://terokarvinen.com/2018/salt-quickstart-salt-stack-master-and-slave-on-ubuntu-linux/ </br>
Luettu: 8.11.2022.

Karvinen, T. 2018. Pkg-File-Service – Control Daemons with Salt – Change SSH Server Port. </br>
Luettavissa: https://terokarvinen.com/2018/pkg-file-service-control-daemons-with-salt-change-ssh-server-port/ </br>
Luettu: 8.11.2022.

Saltstack. 2016. SaltStack Get Started. </br>
Luettavissa: https://docs.saltproject.io/en/getstarted/ </br>
Luettu: 8.11.2022.

Saltstack. 2016. Understanding SaltStack. </br>
Luettavissa: https://docs.saltproject.io/en/getstarted/system/index.html </br>
Luettu: 8.11.2022.

Saltstack. 2016. SaltStack Fundamentals. </br>
Luettavissa: https://docs.saltproject.io/en/getstarted/fundamentals/index.html </br>
Luettu: 8.11.2022.

Saltstack. 2016. SaltStack Configuration Management. Functions. </br>
Luettavissa: https://docs.saltproject.io/en/getstarted/config/functions.html </br>
Luettu: 8.11.2022.


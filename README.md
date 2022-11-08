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
  * 









## Lähteet

Karvinen, T. 2018. Salt Quickstart - Salt Stack Master and Slave on Ubuntu Linux. </br>
Luettavissa: https://terokarvinen.com/2018/salt-quickstart-salt-stack-master-and-slave-on-ubuntu-linux/ </br>
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


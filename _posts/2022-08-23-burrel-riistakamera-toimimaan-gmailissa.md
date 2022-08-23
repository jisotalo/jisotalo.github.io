---
layout: post
title: Burrel-riistakamera toimimaan Gmailissa
---

Hankin Burrel S12 HD+SMS III -riistakameran mökille valvontakamerakäyttöön. Kyseistä mallia oli kehuttu kohtalaisen paljon, joten ajattelin sen hoitavan homman mallikkaasti. Saas nähdä...

![image](https://user-images.githubusercontent.com/13457157/186179662-17106a3d-12bc-439f-8adb-94913beda363.png)

 Kaverilla on vastaavia useampia ja kirosi alimpaan helvettiin kaikki riistakamerat. Erityisesti siksi ettei Gmaili enää toimi ja pelkkää ongelmaa muutenkin. No, Gmailin kanssa taistelu on tullut jo aiemmin vastaan parin projektin kanssa, joten osasin lähteä etsimään syytä Gmailin päästä.

**Lopputulos:** Ei tarvikaan heittää vielä jorpakkoon ja kaverikin on tyytyväinen. Oli jo lähettämässä huoltoon...

## Ongelma
Gmail on jatkuvasti tiukentanut ns. ei-turvallisten sovellusten pääsyä sähköpostiin. Tästä on ollut paljonkin [keskustelua](https://news.ycombinator.com/item?id=30513825). 30.05.2022 alkaen ei enää sähköpostin lähetys ole toiminut perinteisellä tavalla, eli käyttäen tunnusta ja salasanaa.

[https://support.google.com/accounts/answer/6010255?hl=fi](https://support.google.com/accounts/answer/6010255?hl=fi)

Jos Burrelin riistakamerassa (ja varmasti muissakin kameroissa) asettaa Gmailin käytettäväksi sähköpostiksi, laite kyllä yrittää lähettää mutta ei onnistu. Ja se siinä, ei mitään selkeää syytä miksi (*nice*).

## Ratkaisu

Jotta Gmail-lähetys toimii, täytyy tehdä seuraavat asiat:
1. Gmail: Aktivoida kaksivaiheinen tunnistautuminen
2. Gmail: Luoda uusi sovelluksen salasana
3. Burrel: Konfiguroida riistakamera lähettämään sovelluksen salasanan avulla

### Kaksivaiheisen tunnistautumisen aktivoiminen
1. Kirjaudu Gmailiisi normaalisti
2. Valitse oikeasta yläkulmasta nimikirjaimesi alta asetusvalikko ja valitse `Ylläpidä Google-tiliäsi`.

![image](https://user-images.githubusercontent.com/13457157/186183976-2525d55e-5898-4d82-aa58-a5ae99197f89.png)

3. Valitse `Tietoturva`
4. `Googleen kirjautuminen` -osiossa klikkaa `Kaksivaiheinen vahvistus` -riviä
4. Ota vahvistus käyttöön ohjeiden mukaan.

### Sovellussalasanan luonti

Lisätietoja: [https://support.google.com/accounts/answer/185833](https://support.google.com/accounts/answer/185833)

1. Kirjaudu Gmailiisi normaalisti
2. Valitse oikeasta yläkulmasta nimikirjaimesi alta asetusvalikko ja valitse `Ylläpidä Google-tiliäsi`.
3. Valitse `Tietoturva`
4. `Googleen kirjautuminen` -osiossa klikkaa `Sovellussalasanat`-riviä
5. Lisää uusi valitsemalla `Valitse sovellus` ja sen alta `Muu (omavalintainen nimi)`. Kirjoita nimeksi esim. Riistakamera ja paina `Luo`

![image](https://user-images.githubusercontent.com/13457157/186186074-f6a47940-359d-406f-ab0b-5f6cb0e02286.png)

6. Ota itsellesi salasana talteen. Tämä on nyt riistakameran käyttämä salasana.

![image](https://user-images.githubusercontent.com/13457157/186186655-9e940cba-eef8-415e-af79-6df2b035999c.png)

7. Nyt uusi sovellussalasana on luotu ja sitä voidaan käyttää ei-turvallisissa ohjelmissa

![image](https://user-images.githubusercontent.com/13457157/186187495-7df4d48f-5187-4cc6-9939-e2d67118af91.png)

### Riistakameran konfigurointi

1. Konfiguroi riistakamera kuten ennekin
2. Käytä salasanana oman Gmail-salasanasi sijasta äsken luotua sovelluksen salasanaa.

![image](https://user-images.githubusercontent.com/13457157/186188671-0556fec7-b228-4d5f-bd95-581ce5ac4adb.png)

*Esimerkki konfigurointiohjelmasta, tietysti sama homma jos konffaa riistakameran näytöltä* 

## Lopuksi

Loppupeleissä homma on tehty harvinaisen fiksuksi. Tietysti myös tietoturva paranee: jos joku pääsee käsiksi riistakameran asetuksiin, eipä tarvitse kuin poistaa sovellussalasana ja tili on turvassa.

Seuraillaan kuinka kamera ja Gmail toimii, toistaiseksi näyttää oikein hyvältä.

![camera3](https://user-images.githubusercontent.com/13457157/186180405-c86dd4dd-f6b1-4583-8780-9f259289e0c0.gif)

---
layout: post
title: VHS-nauhojen digitointi itse tehtynä halvalla
---

Jo jonkin aikaa on nurkissa lojunut vanhoja VHS-kotivideoita odottaen siirtämistä digitaaliseen muotoon. Nauhat eivät meinaa kestää aikaa ja lisäksi VHS-laitteet alkavat olla jo vähän harvinaisempia. Onneksi nauhuri löytyi vielä nurkista.

## Selvitys

Digitointiin on kolme vaihtoehtoa, joita ovat

1. työn teettäminen alan liikkeessä,
2. itse tekeminen kirjastossa tai
3. itse tekeminen omilla laitteilla.

Ajattelin aluksi ulkoistaa tämän alan liikkeelle, mutta heti ensimmäisen nettihaun jälkeen alkoi se tuntua liian kalliilta (parikymppiä per nauha). Ei ole edes tietoa onko nauhoissa sitä mitä pitäisi.

Seuraavaksi tuli mieleen kirjastot. [Esimerkiksi Tampereen kirjastossa tätä voi tehdä ilmaiseksi kunhan varaa ajan](https://www.tampere.fi/kirjastot/kirjastojen-tietokoneet-ja-laitteet/digitointi-ja-editointi-kirjastoissa). Tämä alkoi houkuttaa kunnes ajatus tuntien istumisesta kirjastossa odottaen alkoi kuulostaa työläältä, vaikka muuten kirjastoissa viihdynkin. **Kahden tunnin videon digitointihan vie sen kaksi tuntia aikaa.**

Viimeisenä sitten itse tekeminen. Rahaa ei liikaa kiinnostanut laittaa projektiin, joten lähdin budjetti edellä selvittämään. Onneksi edellisjouluksi tuli hankittua SCART-HDMI -adapteri vanhojen videoiden katseluun. Eli tarve olisi ainoastaan HDMI-kuvan tallentamiselle.

Suomesta löyty muutamaa laitetta, esimerkiksi Verkkokauppa.comista löytyi [Magix Video Saver! 2022 - videokaappari USB-liitäntään](https://www.verkkokauppa.com/fi/product/715712/Magix-Video-Saver-2022-videokaappari-USB-liitantaan). Silti yli 60 euron hinta tuntui aika kovalta joten ei muuta kun Aliexpressiin...

Aliexpressistä löysin fiksun oloisen HAGIBIS-merkkisen laitteen ([Hagibis Video Capture Card USB 3.0 4K HDMI-compatible Video Game Grabber ](https://www.aliexpress.com/item/1005001773724519.html)). Hintaa tuottelle oli noin 20 euroa ja turhien *"Received the product, haven't tested 5/5 awesome"* arvosteluiden seassa oli myös oikeita arvosteluja, joiden perusteella tuote vaikutti jopa toimivalta. Ei muuta kun tilaukseen ja toivotaan parasta... 

![image](https://user-images.githubusercontent.com/13457157/201733293-13fa9a4a-109f-4fd4-8735-6c13176c1986.png)

## Rauta

Laite tuli lopulta postissa ja oli ihan fiksun olonen. SCART-HDMI-muuntimena käytin Suomesta ostamaani muunninta ([Scart -> HDMI adapteri](https://holvi.com/shop/digiankka/product/039777a9792405e2bdc698a4860cd20e/)). Näitä lienee miljoona erilaista mutta tuon ostin aikanaan itse.

![image](https://user-images.githubusercontent.com/13457157/201735217-a93fdabc-52b2-464e-8850-fdb0d02255e3.png)

Setuppi oli siis lopulta:
```
Sony VHS-nauhuri
      |
SCART-HDMI-muunnin
      |
HDMI-USB-muunnin
      |
   Läppäri
      |
Ulkoinen kiintolevy
```

Ja tässä vielä kuva setupista:

![image](https://user-images.githubusercontent.com/13457157/201735556-4cf7c8eb-3924-40df-956c-e886d263d383.png)


## Digitointi

HAGIBIS-laitteen mukana tulleessa ohjeessa neuvottiin käyttämään [OBS Studio](https://obsproject.com/) -nimistä ohjelmaa. Tämä vaikuttikin oikein pätevältä.

### Asetuksia

Alussa lisäsin uuden kuvalähteen **Sources**-kohdassa ja valitsin **Video Capture Device**.

**Video capture device -asetukset**

![image](https://user-images.githubusercontent.com/13457157/201736933-f781aa1d-263f-452d-9c60-80cb6c243386.png)

Sitten poistin kaikki muut audiolähteet paitsi Video Capture Device:n, jotta esimerkiksi läppärin mikrofonin ääntä ei käytetä. Tämän tein menemällä asetuksiin ja asettamalla kaikki laitteet **disabled** -tilaan.


![image](https://user-images.githubusercontent.com/13457157/201738238-e488da78-e962-4444-bf0d-6defaf73e3b1.png)

![image](https://user-images.githubusercontent.com/13457157/201737981-f4850c54-e094-4b76-bec8-da3c520e79a4.png)


Muutin myös asetuksista resoluution pienemmäksi (koska VHS).

![image](https://user-images.githubusercontent.com/13457157/201738600-6543b5ab-f1d9-4a75-804e-1bfad7556a8d.png)

Havaitsin että videolla välillä kuuluva rätinä aiheutti äänen pätkimistä. Lisäämällä kaksi filteriä se tuntui helpottavan. Eli klikkasin oikealla Sources-kohdan alla Video Capture Device -laitetta ja valitsin **Filters**. 

Lisäsin kaksi suodatinta: **Gain** johon laitoin arvoksi -5.20 dB sekä **Limiter** arvolla -6.00 dB.

![image](https://user-images.githubusercontent.com/13457157/201737836-b99311b8-b367-421c-97fc-a2af79c29633.png)


![image](https://user-images.githubusercontent.com/13457157/201737890-9152ea85-e2f1-4be3-9dc6-9463ee13cd82.png)


## Nauhoitus

Sitten vaan kasetti nauhuriin, sopivan kohdan haku ja video päälle. Samalla painoin OBS Studiosta **Start Recording**. Näin syntyy kivasti digitoitua VHS-videota mkv-tiedostoon.

![image](https://user-images.githubusercontent.com/13457157/201743170-76cd3936-7592-413d-b312-aaa63a5835c0.png)

## Kevennys

Yhdeltä kasetilta löytyi MTV3-kanavalta vuonna 2001 tullutta yöchattia. Laitoin sen YouTubeen koska onhan se nyt legendaarista kulttuuria 2000-luvun alusta.

[https://www.youtube.com/watch?v=F-UQ1juNCB8](https://www.youtube.com/watch?v=F-UQ1juNCB8)

<iframe width="560" height="315" src="https://www.youtube.com/embed/F-UQ1juNCB8" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Lopuksi

VHS-nauhojen digitointi olikin aika helppo homma. Hintaa tuli yhteensä noin 30 euroa. Ja nyt on jatkossakin laitteet valmiina.

Uskallan myös suositella tuota Aliexpressin muunninta/videokaapparia. Hoiti homman mallikkaasti.

Muutama video ei enää meinannut toimia, joten kannattaa hoitaa homma alta pois ajoissa..

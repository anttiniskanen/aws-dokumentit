# Pysyva verkkosivu

Tässä esimerkissä luodaan pysyvä verkkosivu. Amazonin Yksinkertaista Tietovarasto Palvelua (Amazon S3) käytetään tietovarastona kaikelle tietoudelle, joista pysyvä verkkosivu koostuu: HTML-tiedostoille, kuville, CSS-määrittelyille, videoille ja JavaScript-koodille. Jokainen tietue varastoidaan Amazon S3:een objektina, paikkaan jota kutsutaan ämpäriksi.

Tämä ohjeistus käsittää:
-Ämpärin luonnin
-Ämpärin määrittelyn
-Verkkosivun levityksen
-Siivouksen

Yksinkertaisuuden vuoksi tässä ohjeistuksessa ei luoda sivustolle pysyvää toimialuenimeä. Tätä varten vaadittaisiin ainakin:
-Toimialuenimen rekisteröinti
-Sen liittäminen sivustoon

## Esivalmistelut

Ansible sisältää valmiin kokoelman moduuleja AWS-ympäristön hallintaan. Tässä selitetään lyhyesti, mitä toimenpiteitä näiden käyttöönottoon liittyy. Näihin ei enää palata myöhemmissä esimerkeissä.

Ansiblen AWS-moduulit vaativat Pythonin boto-paketin. Ainakin tämä pitää siis olla asennettuna järjestelmässä, josta Ansiblea ohjataan. Boton voi asentaa komentamalla `pip install boto` (jos tätä ei löydy, voi yrittää vaikkapa `apt install python-pip`...).

Tavallisesti Ansible-moduuleja suoritetaan yksittäisiä koneita tai koneryhmiä vastaan. Pilvipalveluja ohjaavat toimenpiteet suoritetaan kuitenkin paikallisessaa järjestelmässä (`localhost`), joten ainakaan tässä esimerkissä ei tarvise huolehtia Ansiblen hosts-tiedostosta ym. inventaarion yksityiskohdista.

Käyttäjän henkilöyden todennus pilvipalvelujen automatisoinnissa onnistuu käyttäjätiliin liitetyn avaimen tunnisteella ja siihen liittyvällä salaisella osalla. Nämä voi paljastaa Ansiblelle esimerkiksi ympäristömuuttujina:
```
export AWS_ACCESS_KEY_ID='AK123'
export AWS_SECRET_ACCESS_KEY='abc123'
```
Muitakin vaihtoehtoja on. Näistä kannattaa valita tietoturvan ja käytettävyyden suhteen paras kompromissi.

## Ämpärin luonti

Jos edelliset kohdat ovat kunnossa, voidaan luoda uusi ämpäri. Rakennetaan Ansible-pelikirja `01-01-amparit.yml`

Pelikirja aloitetaan määrittämällä pilvipalveluille ominainen, paikallinen suoritusympäristö. Testiympäristössä emme tarvitse myöskään faktojen keräilyä koneistamme.
```
---
- hosts: localhost
  connection: local
  gather_facts: False
```

Määritellään ainakin tehtävälle sopiva nimi.
```
tasks:
    - name: esimerkki-ampari
      s3_bucket:
```

Tämän alle määritellään vastaavasti ämpäriä kuvaava nimi.
```
         name: esimerkki.fi
```

Harjoituksen vuoksi laitetaan ämpäri heti aluksi oikealle vyöhykkeelle, lähelle kehittäjiä.
```
         region: eu-west-1
```

Tehtävän tulos tallennetaan muuttujaan mahdollista jatkokäsittelyä varten.
```
      register: s3_bucket
```

Vastaavasti rakennetaan ämpäriautomaatiot myös esineille `www.esimerkki.fi` ja `logs.esimerkki.fi`. Jos tässä käytettäisiin oikeita toimialuenimiä, nimien pitäisi vastata toisiaan, ts. ämpärit pitää luoda kerralla oikein toimialuetta varten, sillä niiden muuttaminen jälkikäteen ei ole mahdollista.

Näilä lisäyksillä ämpärin luonnin mahdollistava automaatio on valmis. Asian voi todentaa ajamalla `ansible-playbook 01-01-amparit.yml` ja tarkistamalla esimerkiksi AWS:n S3-konsolista näiden olemassaolon.

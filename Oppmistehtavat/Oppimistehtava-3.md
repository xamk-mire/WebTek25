# Oppimistehtävä 3: PostgreSQL + Prisma & migraatiot

Päivitä backend toteutus käyttämään PostgreSQL tietkokantaa Prisma ORM:n avulla.

#### 🎯 Tavoitteet:

-  Korvataan aiemmin käytetty **muistissa pidetty data** oikealla **PostgreSQL-tietokannalla**.
    
- Käytetään **Prisma ORM**:ää tietokantayhteyksiin.
    
- Säilytetään **REST-rajapinnan sopimus**, jotta frontend toimii muuttamatta.
    
- Otetaan käyttöön **migraatiot** skeeman hallintaan.
    
- Lisätään **siemendata** (seed) kehitysympäristön testaukseen.
    

---
#### 📂 Esimerkkiprojektin rakenne

 Oppimistehtävän lopussa, backend-projektin tulisi näyttää tältä:

```
backend/
├─ prisma/                     # Prisma ORM: skeema ja siementiedot
│  ├─ schema.prisma            # Tietokannan skeema (Plant, Device, Status)
│  └─ seed.ts                  # Kehitysdatan seedit
│
├─ src/
│  ├─ app.ts                   # Express-sovelluksen luonti ja middlewaret
│  ├─ server.ts                # Käynnistyspiste (käynnistää palvelimen)
│  │
│  ├─ db/
│  │  └─ prisma.ts             # PrismaClient-instanssi
│  │
│  ├─ models/
│  │  └─ types.ts              # Tyypit (Plant, Device, AddPlantBody) API:lle
│  │
│  ├─ repositories/            # Tietokantakerros
│  │  └─ plants.repo.ts        # Prisma-kyselyt: listaus, haku, luonti, toggle
│  │
│  ├─ services/                # Sovelluslogiikka
│  │  └─ plants.service.ts     # Kutsuu repositorya ja muuntaa tiedot API-muotoon
│  │
│  ├─ controllers/             # HTTP-kontrollerit
│  │  └─ plants.controller.ts  # Vastaa pyyntöihin, validointi, statuskoodit
│  │
│  └─ routes/
│     └─ plants.routes.ts      # Rajapintapolut (Express Router)
│
├─ .env                # Ympäristömuuttujat
├─ .gitignore
├─ package.json
├─ tsconfig.json
└─ README.md
```

> Tämä rakenne noudattaa selkeää kerrosarkkitehtuuria:
> 
> - **controllers** huolehtii HTTP-tason asioista ja kutsuu **services**-kerrosta.
>     
> - **services** sisältää sovelluslogiikan ja muuntaa tietokantatulokset API-muotoon.
>     
> - **repositories** tekee varsinaiset tietokantakyselyt Prismalla.
>     
> - **db** sisältää yhden PrismaClient-instanssin, jota repositorit käyttävät.
>     

---

## Tehtävä 1 – Tietokannan asennus ja ympäristö

### Vaatimukset

1. Asenna ja käynnistä **PostgreSQL** paikallisesti (natiiviasennus tai Docker).
    
2. Luo erillinen **käyttäjä** ja **tietokanta** sovellukselle (älä käytä oletus-`postgres` käyttäjää).
    
3. Konfiguroi yhteys **ympäristömuuttujalla**:
    
    - Käytä nimeä `DATABASE_URL`.
        
    - Tallenna arvo backend-projektin **`.env`-tiedostoon** (salainen, ei versionhallintaan).
        
    - Luo **`.env.example`** tiedosto, jossa on esimerkkimuotoinen rivi ilman oikeita salasanoja.
        
4. Dokumentoi README-tiedostoon, miten tietokanta yhteys toimii env muuttujien avulla.
    

### Hyväksymiskriteerit

- Sovellus pystyy ottamaan yhteyden tietokantaan käyttäen `DATABASE_URL`-muuttujaa.
    
- Salaisia arvoja ei ole versionhallinnassa.
    

---

## Tehtävä 2 – Prisma käyttöönotto

### Vaatimukset

1. Asenna Prisma backend-projektiin.
    
2. Alusta Prisma niin, että:
    
    - **`prisma/schema.prisma`** sisältää tietolähteen (`datasource`) ja `client`-generaattorin.
        
    - Tietolähde käyttää **PostgreSQL**-provideria ja `url` tulee ympäristömuuttujasta.
        
3. Lisää npm-skriptit `package.json` tiedostoon:
    
    - `prisma generate` – luo tyypitetyn clientin
        
    - `prisma migrate dev` – kehitysmigraatiot
        
    - `prisma migrate deploy` – tuotantoon
        
    - `prisma studio` – graafinen tarkastelu
        
4. Kirjoita README-tiedostoon ohje Prisma-työskentelyyn:
    
    - miten client generoidaan
        
    - miten migraatiot suoritetaan kehityksessä ja tuotannossa
        

### Hyväksymiskriteerit

- `prisma generate` onnistuu.
    
- Prisma tunnistaa tietokannan määritetyn `DATABASE_URL`-muuttujan avulla.
    

---

### Tehtävä 3 – Tietomallin luonti ja migraatiot

### Vaatimukset

1. Määritä **Prisma-skeemassa** kolme perusrakennetta jotka heijastavat `models/types.ts` tyyppimalleja:
	
    - **Enum** `Status` arvoilla: `ok`, `fault`, `offline`.
        
    - **Plant**-malli:
        
        - `id`
            
        - `name` 
            
        - `species` 
            
        - `location` 
            
        - `notes`
            
        - `createdAt` 
            
        - `updatedAt` 
            
        - 1:1-suhde `Device`-malliin
            
    - **Device**-malli:
        
        - `id` 
            
        - `status` 
            
        - `battery` 
            
        - `watering` 
            
        - `moisture`
            
        - `createdAt`, `updatedAt`
            
        - `plantId` (kokonaisluku, **uniikki**, viittaa Plant-malliin)

>[!NOTE]
> 1. Muista määrittää malleille pääavain
> 2. Käytä id:n luonnissa juoksevaa id:n generointia (autoincremented)
> 3. Huomioi mitkä mallien kentistä ovat **nullableja** sekä määrittele tarvittaessa **default arvot**

2. Luo ja suorita **ensimmäinen migraatio**, joka luo tarvittavat taulut ja suhteet.
    
3. Tallenna migraatiot versionhallintaan (tulisi tapahtua automaattisesti kun ajat migraatiot).
    
4. Dokumentoi kehityksen ja tuotannon **migraatiotyönkulku**:
    
    - kehitys: muokkaa skeemaa → `migrate dev` → clientin generointi
        
    - tuotanto: `migrate deploy`
        

### Hyväksymiskriteerit

- Migraatiot luovat taulut ja suhteet onnistuneesti.
    
- Prisma-client heijastelee uusia malleja.
    
- Toisen kehittäjän on mahdollista kloonata repo, ajaa migraatiot ja saada toimiva skeema.
    

---
### Tehtävä 4 – Kehitysdatan siemenet (seed)

### Vaatimukset

> [!NOTE]
> Tutustu ennen tehtävän tekoa kuinka seedaus toimii prismassa: [Seeding | Prisma Documentation](https://www.prisma.io/docs/orm/prisma-migrate/workflows/seeding)

1. Luo **siemendata/seed**, joka:
    
    - Poistaa olemassa olevan datan kehitysympäristössä ennen lisäystä.
        
    - Lisää vähintään **3 kasvia**, joilla kaikilla on oma laite.
        
    - Käyttää realistisia arvoja kosteudelle ja akulle.
        
2. Toteuta npm-skripti seedien suorittamista varten.
    
3. Kirjoita README-tiedostoon, milloin seedejä ajetaan (esim. ensimmäinen käyttöönotto, tietokannan nollauksen jälkeen).
    
>[!NOTE]
> Kehitysympäristö saattaa madhollisesti herjata "virheellisestä" tyyppivirheistä, joista ei välttämättä tarvitse välittää
> -> sovellus kääntyy ja toimii oikein "tyyppivirheestä" huolimatta, esim. `Cannot find name 'process'. Do you need to install type definitions for node? Try "npm i --save-dev @types/node"` 

### Hyväksymiskriteerit

- Seedien ajaminen luo tietokantaan odotetut rivit.
    
- Seedien ajaminen useita kertoja kehityksessä ei tuota duplikaatteja (koska vanhat rivit poistetaan ensin tai logiikka on idempotentti).
    

---

### Tehtävä 5 – Repository- ja Service-kerrokset

### Vaatimukset

1. Luo **repository-kerros** `src/repositories`, joka vastaa ainoastaan tietokantayhteyksistä Prisma-clientin avulla.
        
2. Toteuta repository-funktiot:
    
    - **Listaa kasvit**:
        
        - Palauttaa kaikki kasvit liittyvine laitetietoineen.
            
        - Järjestää tulokset nousevan `id`-arvon mukaan.
            
        - Tukee valinnaisesti `offset` ja `limit` sivutusparametreja.
            
    - **Hae kasvi id:n perusteella**:
        
        - Palauttaa kasvin ja sen laitteen tai `null`, jos ei löydy.
            
    - **Vaihda laitteen kastelutilaa**:
        
        - Suoritetaan **transaktiossa**: haetaan nykytila, käännetään se (true ↔ false) ja tallennetaan.
            
        - Palauttaa päivitetyn kasvin tai `null`, jos ei löydy.
            
    - **Luo kasvi**:
        
        - Ottaa vastaan luontipyynnön kentät.
            
        - Luo aina **yhden laitteen** uudelle kasville.
            
        - Käyttää oletusarvoja (status = `ok`, battery = 100, watering = false, moisture = 50).
            
        - Rajaa kosteuden ja akun arvot välille 0–100.
            
3. Päivitä **service-kerros** niin, että se kutsuu repository-kerroksen funktioita:
    
    - Palauttaa tiedot samassa muodossa kuin ennen, jotta kontrollereihin ei tarvita muutoksia.
        

### Hyväksymiskriteerit

- Service-funktiot palauttavat JSON-sarjallistettavia kasvi- ja laite-olioita.
    
- Tietokantakyselyt on eristetty repository-kerrokseen.
    
- Kastelutilan vaihto on atominen (ei kilpajuoksuongelmia -> tila vaihtuu vaikka kyselyitä olisi useampia).
    

---

### Tehtävä 6 – Kontrollerit, API-sopimus ja dokumentaatio

### Vaatimukset

1. **Säilytä REST-rajapinnan sopimus**:
    
    - `GET /api/plants` palauttaa listan kasveista (array-muodossa, jotta frontend ei hajoa).
        
    - `GET /api/plants/:id` palauttaa yksittäisen kasvin tai 404-virheen.
        
    - `POST /api/plants/:id/water` vaihtaa laitteen kastelutilaa ja palauttaa päivitetyn kasvin.
        
    - `POST /api/plants` luo uuden kasvin ja palauttaa sen; vastaa koodilla 201 ja sisältää Location-otsakkeen.
        
2. Toteuta validointi:
    
    - Hylkää virheellinen `id` polkuparametreissa (status 400).
        
    - Hylkää virheellinen syöte luontipyyntöön (status 400 ja kuvaava virheviesti).
        
3. Päivitä **Swagger/OpenAPI-dokumentaatio**:
    
    - Rajapinnat ja skeemat vastaavat tietokannan tuottamaa dataa.
        
    - Sisällytä kentät kasville ja laitteelle, sekä mahdolliset virhevastausten kuvaukset.
        

### Hyväksymiskriteerit

- Rajapinnat toimivat ja palauttavat odotetun datan.
    
- Swagger-dokumentaatio vastaa toteutusta.
    
- Frontend voi käyttää samoja API-päätepisteitä kuin aiemmin ilman muutoksia.
    


---

## ✅ Lopulliset hyväksymiskriteerit

-  PostgreSQL on käytössä ja saavutettavissa.
    
-  Prisma on alustettu ja migraatiot luovat tarvittavat taulut.
    
-  Siemenet tuottavat odotetut kasvi- ja laiterivit.
    
-  Repository- ja service-kerrokset toimivat ja käyttävät tietokantaa.
    
-  Rajapinnat toimivat ja palauttavat samanlaisen datan kuin ennen.
    
-  Swagger-dokumentaatio on ajan tasalla.
    

---

## ⚠️ Huomioitavaa

- Muista ajaa `prisma generate` aina skeeman muutosten jälkeen.
    
- Älä tallenna salasanoja tai `.env`-tiedostoa versionhallintaan.
    
- Käytä tarvittaessa `prisma migrate reset` vain paikallisessa kehityksessä tietokannan tyhjentämiseen ja siementen uudelleenajoon.
    

# Oppimistehtävä 2: Plants API (Express.js + TypeScript + Swagger)

Toteuta REST-rajapinta, jota aiemmin rakennettu React-frontend voi kutsua.  
Teknologiat: **Node.js (Express) + TypeScript + Swagger**.

Pääpiirteet:

- ID:t ovat **numeerisia** (`number`)
    
- Toteutetaan rajapinnat kasvien listaamiseen, yksittäisen kasvin hakemiseen ja kastelun päälle/pois-kytkemiseen
    
- Tarjotaan OpenAPI (Swagger UI) -dokumentaatio
    
- Käytetään REST-tyylistä API:a `/api`-etuliitteellä
    

---

## 📁 Esimerkki projektirakenne

```
plants-api/
├─ src/
│  ├─ server.ts               # palvelimen käynnistys
│  ├─ app.ts                  # express-sovelluksen määritys
│  ├─ routes/
│  │  └─ plants.routes.ts     # REST-reitit
│  ├─ controllers/
│  │  └─ plants.controller.ts # käsittelijät
│  ├─ services/
│  │  └─ plants.service.ts    # sovelluslogiikka, in-memory data
│  ├─ models/
│  │  ├─ types.ts             # Plant & Device -tyypit
├─ package.json
├─ tsconfig.json
├─ .gitignore                 # määritetään mitä tiedostoja git:n tulee seurata
└─ README.md
```

---

## Tehtävä 1 – Projektin alustus (TypeScript + Express)

**Tavoite:** Luo uusi TS/Node-projekti Expressillä ja kehitystyökaluilla.

1. Alusta ja asenna:
    

```bash
mkdir plants-api && cd plants-api
npm init -y
npm i express cors
npm i -D typescript ts-node-dev @types/express @types/node @types/cors
```

2. `tsconfig.json` (perus)
    

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "CommonJS",
    "rootDir": "src",
    "outDir": "dist",
    "esModuleInterop": true,
    "forceConsistentCasingInFileNames": true,
    "strict": true,
    "skipLibCheck": true
  },
  "include": ["src"]
}
```

3. `package.json` skriptit
    

```json
{
  "scripts": {
    "dev": "ts-node-dev --respawn --transpile-only src/server.ts",
    "build": "tsc",
    "start": "node dist/server.js"
  }
}
```

4. Pienin mahdollinen palvelin esimerkki:
    

**`src/app.ts`**

```ts
import express from "express";
import cors from "cors";

export function createApp() {
  const app = express();
  app.use(cors());
  app.use(express.json());
  
  // Poista kommentointi 2 tehtävän jälkeen
  // app.use('/api/plants', plantsRouter); 

  app.get("/api/health", (_req, res) => res.json({ status: "ok", ts: new Date().toISOString() }));

  return app;
}
```

**`src/server.ts`**

```ts
import { createApp } from "./app";
const PORT = process.env.PORT ?? 8001;
createApp().listen(PORT, () => console.log(`API käynnissä portissa ${PORT}`));
```


5. Lisää backend projektisi juurikansioon .gitignore tiedosto
	
	- Voit tarvittaessa kopioida tiedoston olemassaolevasta frontend toteutuksesta


✅ **Valmis, kun:** `GET /api/health` palauttaa JSONin.

---

## Tehtävä 2 – Tietomalli ja in-memory palvelu

**Tavoite:** Määritä TypeScript-tyypit ja tee yksinkertainen muistiin tallennettu palvelu.

**Tyypit (`src/models/types.ts`):**

- `DeviceStatus = "ok" | "fault" | "offline"`
    
- `Device`
	- id - numero
	- status - DeviceStatus
	- battery - numero
	- watering - boolean
	- moisture - numero
	- created_at - string
	- updated_at - string
    
- `Plant`
	- id - numero
	- name - string
	- species - string
	- location - string tai nullable
	- notes - string tai nullable
	- device - Device tyyppi
    

**Palvelu (`src/services/plants.service.ts`):**

Lisää tiedosto ja lisää aluksi tiedoston sisälle alla oleva esimerkki koodi jota käytetään mockuppaamiseen

```ts
const now = () => new Date().toISOString();

// Randomizer metodi, jonka avulla generoidaan esimerkki dataa
const rnd = (a: number, b: number) => +(a + Math.random() * (b - a)).toFixed(1); 

  
// Lista esimerkki kasveista laitteineen joka toimii väliaikaisena muistivarastona
const plants: Plant[] = [

  {

    id: 1,

    name: 'Monstera - Office',

    species: 'monstera',

    location: 'Office',

    device: {

      id: 1,

      status: 'ok',

      battery: 92,

      watering: false,

      moisture: rnd(45, 60),

      created_at: now(),

      updated_at: now(),

    },

  },

  {

    id: 2,

    name: 'Fiddle Leaf Fig',

    species: 'ficus',

    location: 'Living Room',

    device: {

      id: 2,

      status: 'ok',

      battery: 87,

      watering: false,

      moisture: rnd(35, 50),

      created_at: now(),

      updated_at: now(),

    },

  },

  {

    id: 3,

    name: 'Cactus',

    species: 'cactus',

    location: 'Kitchen',

    device: {

      id: 3,

      status: 'ok',

      battery: 96,

      watering: false,

      moisture: rnd(15, 30),

      created_at: now(),

      updated_at: now(),

    },

  },

];
```

### Palvelufunktiot


#### 2.1) `listPlants(): Plant[]`

**Tarkoitus:** Palauttaa **kaikki** kasvit nykyisestä muistivarastosta.

**Vaatimukset**

- Palauttaa **uudet oliot** (ei suoria viitteitä sisäiseen taulukkoon).
    
- **Järjestys**: vapaa, mutta suositellaan nousevaa `id`-järjestystä, jotta vastaukset ovat deterministisiä.
    

**Virhetilanteet**

- Ei virheitä, myös tyhjä lista on sallittu palautusarvo.
    

---

#### 2.2) `getPlant(id: number): Plant | undefined`

**Tarkoitus:** Palauttaa **yksittäisen kasvin** annetulla `id`:llä.

**Vaatimukset**

- **Parametri**: `id` on **kokonaisluku** ≥ 1.
    
- **Haku**: etsi kasvi täsmäävällä `id`:llä.
    
- Palauta **kopio** kasvin datasta.
    

**Virhetilanteet**

- Jos `id` ei ole kelvollinen numero, palauta `undefined`.
    
- Jos kasvia ei löydy, palauta `undefined`.
    
- Tulos tulkitaan kontrollerissa: 404 Not Found.
    

---

#### 2.3) `toggleWatering(id: number): Plant | undefined`

**Tarkoitus:** **Vaihda** kasviin liitetyn laitteen `watering`-arvoa `true ↔ false` ja päivitä `updated_at`.

**Vaatimukset**

- **Parametri**: `id` viittaa kasviin (ei laitteeseen).
    
- Etsi kasvi; jos löytyy:
    
    - **Vaihda** `plant.device.watering` vastakkaiseksi arvoksi.
        
    - **Päivitä** `plant.device.updated_at` nykyhetkeen (ISO).
        
    - **Säilytä** `plant.device.created_at` muuttumattomana.
        
- **Palauta** päivitetyn kasvin tiedot.
    

**Virhetilanteet**

- Jos kasvia `id`:llä ei löydy, palauta `undefined`.
    
- Jos laitteen tila on jo sama kuin “uusi” arvo (toggle määrittää aina vastakkaisen), se on silti oikea — tila vain vaihtuu.
    

---

### Tehtävä 3 - Kasvien lisäys toiminnallisuuden alustus

#### 3.1) Laajenna tyypitykset (models)

**Tiedosto:** `src/models/types.ts`

1. Lisää **request body** kasvin luontiin:
    

```ts
export type AddPlantBody = {
  name: string;
  species: string;
  location?: string | null;
  notes?: string | null;
  device?: Device;
};
```

**Vaatimukset**

- `name` ja `species` ovat pakollisia (ei tyhjiä).
    
- `device` on valinnainen; jos annetaan, sen kentät ovat valinnaisia ja rajataan: `status`, `battery`, `watering`, `moisture`.

#### 3.2) Lisää palvelufunktio `addPlant(input: AddPlantBody): Plant`

- Pidä kirjaa juoksevista id-arvoista:
    

```ts
// Esimerkki lista sisältää 3 kasvia laitteineen joten aloitus arvon on oltava vähintään 4
let nextPlantId = /* max(plants.id)+1 tai 4 */; 
let nextDeviceId = /* max(devices.id)+1 tai 4 */;
```

- Aseta **oletukset** laitteelle, jos `input.device` puuttuu:
    
    - `status: "ok"`, `battery: 100`, `watering: false`, `moisture: 50`
        
    
- **Auto-increment** sekä kasvin että laitteen id:lle.
    
- Aseta `created_at` sekä `updated_at` arvot käyttämään `now` muuttujaa
	 
- Työnnä uusi kasvi in-memory taulukkoon.
    
- Palauta lopuksi **Luotu kasvi**
    
- Virhe

>[!NOTE]
>Voit tarvittaessa käyttää `api.mock.ts` tiedoston toteutusta apuna esimerkkinä

---
### Tehtävä 4 – Kontrollerit ja reitit

**Tavoite:** Liitä in-memory -palvelu (Tehtävä 2) HTTP-rajapinnaksi Expressin avulla. Toteuta **kasvien REST-päätepisteet** yhtenäisen `/api`-etuliitteen alle.

Toteuta kontrollerit siten, että ne kutsuvat tehtävissä 2 sekä 3 toteutettuja metodeja.

**Reitit (`src/routes/plants.routes.ts`):**

- `GET /api/plants` → hakee listan kasveista
    
- `GET /api/plants/:id` → hakee yksittäisen kasvi
    
- `POST /api/plants/:id/water` → togglettaa kastelun yksittäiselle kasville
    
- `POST /api/plants → luo uuden kasvin laitteineen

---

### Tehtävä 5 – Swagger/OpenAPI dokumentaatio

**Tavoite:** Laadi **Swagger/OpenAPI-dokumentaatio** kasvi-API:lle, jotta kehittäjät näkevät selkeästi mitä rajapintoja on tarjolla, mitä ne palauttavat ja millaisia virhevastauksia voi tulla.

Dokumentaatio toteutetaan **Swagger UI**:n avulla, joka näyttää rajapinnan selainkäyttöliittymänä (`/api/docs`) ja tarjoaa myös raakamuotoisen OpenAPI JSON -tiedoston (`/api/openapi.json`).


Asenna aluksi tarvittavat kirjastot sekä typescript tyypit:

```bash
npm i swagger-ui-express swagger-jsdoc
npm i -D @types/swagger-ui-express @types/swagger-jsdoc
```

 
 Lisää seuraavat koodit `src/app.ts`

```ts
import swaggerUi from "swagger-ui-express";
import { openapiSpec } from "./docs/openapi";

app.get("/api/openapi.json", (_req, res) => res.json(openapiSpec));
app.use("/api/docs", swaggerUi.serve, swaggerUi.setup(openapiSpec));
```


#### 5.1 Dokumentoitavat endpointit
#### `GET /api/plants`

- **Summary:** Listaa kaikki kasvit.
    
- **Responses:**
    
    - `200 OK` → taulukko `Plant`-olioita.
        
    - `500 Internal Server Error` → jos odottamaton virhe tapahtuu.
        

#### `GET /api/plants/{id}`

- **Summary:** Palauttaa yksittäisen kasvin `id`:n perusteella.
    
- **Parameters:**
    
    - `id`: path-parametri, tyyppi `integer`, vaadittu.
        
- **Responses:**
    
    - `200 OK` → yksi `Plant`.
        
    - `400 Bad Request` → jos `id` ei ole kelvollinen numero.
        
    - `404 Not Found` → jos kasvia ei löydy.
        
    - `500 Internal Server Error`.
        

#### `POST /api/plants/{id}/water`

- **Summary:** Vaihtaa (toggle) kastelun päälle tai pois.
    
- **Parameters:**
    
    - `id`: path-parametri, tyyppi `integer`, vaadittu.
        
- **Responses:**
    
    - `200 OK` → palauttaa päivitetyn `Plant`.
        
    - `400 Bad Request` → jos `id` virheellinen.
        
    - `404 Not Found` → jos kasvia ei löydy.
        
    - `500 Internal Server Error`.
        

#### `POST /api/plants:

- Summary: Lisää uuden kasvin järjestelmään
	
- **Parameters:**
    
	- `requestBody` (name, species pakolliset; optional: location, notes, device{ status, battery, watering, moisture })
		
- **Responses:**
    
    - `200 OK` → palauttaa lisätyn `Plant`.
        
    - `400 Bad Request` → jos virhe.
        
    - `500 Internal Server Error`.
        

---
#### 5.2 Schemat (esimerkkirakenteet)

Voit katsoa esimerkkiä Schemojen (Data Models:n) toteuttamisesta Swagger:n dokumentaatiosta: [Data Models (Schemas) | Swagger Docs](https://swagger.io/docs/specification/v3_0/data-models/data-models/) (Hyvä esimerkki löytyy esim. Nested Objecst osiosta).

#### `Plant`

- **Kentät:**
    
    - `id` (integer, example: 1)
        
    - `name` (string, example: "Monstera - Office")
        
    - `species` (string, example: "monstera")
        
    - `location` (string, nullable, example: "Office")
        
    - `notes` (string, nullable, example: "Needs bright light")
        
    - `device` (`Device`)
        

#### `Device`

- **Kentät:**
    
    - `id` (integer, example: 101)
        
    - `status` (string, enum: `ok`, `fault`, `offline`)
        
    - `battery` (integer, 0–100, example: 92)
        
    - `watering` (boolean, example: false)
        
    - `moisture` (number, example: 47.5)
        
    - `created_at` (string, date-time, example: "2025-09-23T10:15:00Z")
        
    - `updated_at` (string, date-time)
        

#### AddPlantBody -> ei tarvitse toteuttaa tässä vaiheessa


✅ **Valmis, kun:** Swagger UI näkyy osoitteessa `/api/docs`.

---

### Tehtävä 6 – Frontend-integraatio

**Tavoite:** Saat frontin käyttämään backendin API:a.


Lisää (tai varmista) proxy-asetus:
```ts
// vite.config.ts
server: {
  proxy: { "/api": { target: "http://localhost:8001", changeOrigin: true } }
}
```
**Miksi?** Frontti voi aina kutsua **suhteellista** polkua `/api/...`, selain ei tee CORS-kutsua → ei CORS-virheitä.


Jos kutsut täysiä URL:eja, käytä `cors`-asetusta backend:ssä:

```ts
app.use(cors({ origin: ["http://localhost:5173"] }));
```


Muokkaa `frontend/src/lib/api.base.ts` tiedostoa, toteuttamalla tiedoston sisälle uudet metodit, joidenka avulla frontend käyttää backend:n tarjoamia API endpointteja mockattujen metodien sijaan.

```ts
export const API = {

  listPlants: listPlants, // ennen mock.listPlants

  getPlant: getPlant, // ennen mock.getPlant

  togglePlantWatering: togglePlantWatering, // ennen mock.togglePlantWatering

  addPlant: addPlant, // ennen mock.addPlant

};
```

Lisää toteutukset yllä mainituille metodeille siten, että metodit kutsuvat vastaavia endpointteja

- listPlants kutsuu `GET /api/plants` 
	
- getPlant kutsuu `GET /api/plants/{id}`
	
- togglePlantWatering `POST /api/plants/{id}/water`
	
- addPlant `POST /api/plants`


#### Testaa järjestelmän toimivuus

- Käytä apuna swagger UI:ta kasvien luonnissa ja varmista, että luotu kasvi tulee näkyviin frontend:n käyttöliittymään
	
	- Voit joutua päivittämään selaintasi, jotta swaggerilla lisätty kasvi ilmestyy näkyviin


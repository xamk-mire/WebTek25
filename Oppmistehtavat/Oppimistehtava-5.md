
# 🌿 **Oppimistehtävä 5: Autentikointi, Docker ja haku/suodatus/lajittelu**

Tässä harjoituksessa jatketaan **kasvien kastelusovelluksen** kehittämistä.  
Tavoitteena on lisätä käyttäjäautentikointi, mahdollistaa sovelluksen ajaminen Dockerissa ja toteuttaa haku-, suodatus- sekä lajittelutoiminnot.

---

## 🟩 **Tehtävä 1 – Käyttäjäautentikointi (Backend)**

### 🎯 Tavoite

Rakennetaan **autentikointijärjestelmä** backend-sovellukseen.  
Käyttäjät voivat rekisteröityä, kirjautua sisään ja käyttää suojattuja API-reittejä JWT-tunnisteella.

---

### ⚙️ Asennettavat kirjastot

Asenna tarvittavat kirjastot backend-hakemistossa:

```bash
npm install bcrypt jsonwebtoken
npm install -D @types/bcrypt @types/jsonwebtoken
```

---

### 🪜 Vaiheittain

#### 1️⃣ Luo käyttäjämalli tietokantaan

**Tiedosto:** `backend/prisma/schema.prisma`

Lisätään `User`-malli Prisma-skeemaan, joka tallentaa sähköpostin, salasanan ja rekisteröintiajan.

```prisma
model User {
  id        Int      @id @default(autoincrement())
  email     String   @unique
  password  String
  name      String?
  createdAt DateTime @default(now())
}
```

> 💡 Tämä määrittely mahdollistaa useamman käyttäjän, mutta varmistaa että sähköpostiosoite on aina yksilöllinen.

Aja muutokset:

```bash
npx prisma generate
npx prisma migrate dev --name add_user_model
```

---

#### 2️⃣ Luo autentikointikontrolleri

**Tiedosto:** `backend/src/controllers/authController.ts`

Tämä tiedosto sisältää kaksi toimintoa:

- `registerUser(req, res)` — rekisteröi käyttäjän, hashataan salasana `bcrypt`illä
    
- `loginUser(req, res)` — tarkistaa salasanan ja luo JWT-tokenin
    

> 💬 **Miksi:**  
> Erottamalla logiikka controlleriin pidämme koodin selkeänä.  
> Jokainen reitti (`/register`, `/login`) kutsuu omaa funktiotaan.

---

#### 3️⃣ Määrittele autentikointireitit

**Tiedosto:** `backend/src/routes/authRoutes.ts`

Reitit:

- `POST /api/auth/register`
    
- `POST /api/auth/login`
    

Reitit tuodaan käyttöön `app.ts`:ssa:

```ts
import authRoutes from "./routes/authRoutes";
app.use("/api/auth", authRoutes);
```

---

#### 4️⃣ Lisää JWT-tarkistusmiddleware

**Tiedosto:** `backend/src/middleware/auth.ts`

Middleware varmistaa, että pyynnöissä on voimassa oleva token.  
Jos token puuttuu tai on virheellinen, palautetaan `401 Unauthorized`.

```ts
import jwt from "jsonwebtoken";

export function requireAuth(req, res, next) {
  const header = req.headers.authorization;
  if (!header) return res.status(401).json({ error: "Missing token" });

  const token = header.split(" ")[1];
  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.user = decoded;
    next();
  } catch {
    res.status(401).json({ error: "Invalid token" });
  }
}
```

> 💬 **Miksi:**  
> Tämä suojaa kasvien hallinnan reitit siten, että vain kirjautuneet käyttäjät voivat käyttää niitä.

---

## 🟦 **Tehtävä 2 – Autentikointi käyttöliittymässä (Frontend)**

### 🎯 Tavoite

Rakennetaan kirjautumis- ja rekisteröintisivut sekä käyttöliittymään konteksti, joka hallitsee käyttäjätilaa ja tokenin tallennusta.

---

### 🪜 Vaiheittain

#### 1️⃣ Luo AuthContext

**Tiedosto:** `frontend/src/auth/AuthContext.tsx`

AuthContext sisältää käyttäjän tilan (`user`, `token`) ja tarjoaa funktiot:

- `login(email, password)`
    
- `register(name, email, password)`
    
- `logout()`
    

> 💬 **Miksi:**  
> Context mahdollistaa tokenin ja käyttäjätiedon jakamisen kaikkien komponenttien välillä ilman propseja.

---

#### 2️⃣ Luo kirjautumissivu

**Tiedosto:** `frontend/src/auth/LoginPage.tsx`

Sivulla on:

- sähköposti- ja salasanakentät
    
- “Kirjaudu” -painike
    
- virheilmoitukset (`alert alert-error`) ja lataus (`btn loading`)
    

---

#### 3️⃣ Luo rekisteröintisivu

**Tiedosto:** `frontend/src/auth/RegisterPage.tsx`

Samanlainen kuin kirjautumissivu, mutta sisältää myös `name`-kentän.

---

#### 4️⃣ Lisää PrivateRoute

**Tiedosto:** `frontend/src/auth/PrivateRoute.tsx`

Tämä komponentti tarkistaa, onko käyttäjä kirjautunut sisään.  
Jos token puuttuu → ohjaa käyttäjän `/login`-sivulle.

> 💬 **Miksi:**  
> Tämä estää pääsyn suojatuille sivuille (kuten kasvilistaan) ilman kirjautumista.

---

#### 5️⃣ Lisää reitit sovellukseen

**Tiedosto:** `frontend/src/main.tsx`

```tsx
<BrowserRouter>
  <AuthProvider>
    <Routes>
      <Route path="/login" element={<LoginPage />} />
      <Route path="/register" element={<RegisterPage />} />
      <Route path="/" element={<PrivateRoute><PlantsPage /></PrivateRoute>} />
    </Routes>
  </AuthProvider>
</BrowserRouter>
```


---

## 🟨 **Tehtävä 3 – Backendin haku, suodatus ja lajittelu**

### 🎯 Tavoite

Laajenna kasvien hakutoimintoa siten, että käyttäjä voi hakea, suodattaa ja lajitella kasveja.

---

### 🪜 Vaiheittain

#### 1️⃣ Muokkaa kasvien reittiä

**Tiedosto:** `backend/src/routes/plantsRoutes.ts`

Lisää GET-pyyntö, joka tukee query-parametreja:  
`q`, `status`, `watering`, `species`, `sort`, `order`, `offset`, `limit`.

#### 2️⃣ Lisää logiikka kontrolleriin

**Tiedosto:** `backend/src/controllers/plantsController.ts`

Hae query-parametrit:

```ts
const { q, status, watering, species, sort, order, offset, limit } = req.query;
```

Luo dynaaminen Prisma-haku `where` ja `orderBy`-ehdoilla.  
Palauta tulos:

```json
{ "total": 123, "offset": 0, "limit": 10, "items": [...] }
```

> 💬 **Miksi:**  
> Näin voimme hakea vain ne kasvit, jotka vastaavat käyttäjän hakua — esim. tietyn lajin tai kosteustason perusteella.

---

## 🟫 **Tehtävä 4 – Frontendin haku, suodatus ja lajittelu**

### 🎯 Tavoite

Lisätään hakukentät ja suodatusvalinnat kasvilistaan, ja kytketään ne backendin API:in.

---

### 🪜 Vaiheittain

#### 1️⃣ Muokkaa PlantsPage-tiedostoa

**Tiedosto:** `frontend/src/pages/PlantsPage.tsx`

Lisää:

- `useSearchParams` hakemaan query-parametrit URL:ista
    
- DaisyUI `input` ja `select` -elementit hakua ja lajittelua varten
    
- “Tyhjennä suodattimet” -painike
    

> 💬 **Miksi:**  
> URL-parametrit mahdollistavat suodatusten tallentamisen selaimen osoiteriville, jolloin näkymä voidaan jakaa muille käyttäjille.

---

## 🟪 **Tehtävä 5 – Sivutus ja käyttöliittymän viimeistely**

### 🎯 Tavoite

Viimeistele sovelluksen käyttöliittymä lisäämällä sivutus, virhe- ja lataustilat sekä yhtenäinen tyyli.

---

### 🪜 Vaiheittain

#### 1️⃣ Lisää sivutusnäkymä

**Tiedosto:** `frontend/src/pages/PlantsPage.tsx`

- “Edellinen / Seuraava” -napit (`btn btn-sm`)
    
- Näytä “N–M / total” -tieto (esim. 1-6/20 -> näkymässä nätetään kasvit 1-6, ja kasveja on yhteensä 20)
    

#### 2️⃣ Lisää tilaviestit (mikäli viestit puuttuvat)

Käytä DaisyUI-komponentteja:

```tsx
{isLoading && <div className="alert alert-info">Ladataan kasveja...</div>}
{error && <div className="alert alert-error">Virhe haussa</div>}
```

#### 3️⃣ Viimeistele ulkoasu 

- Käytä `grid-cols-1 sm:grid-cols-2 lg:grid-cols-3` (pitäisi olla jo käytössä)
    
- Varmista, että komponentit pysyvät siisteinä myös mobiilissa
    
- Lisäile vapaasti custom tyylityksiä
    

> 💬 **Miksi:**  
> Käyttöliittymän viimeistely tekee sovelluksesta ammattimaisen ja helppokäyttöisen kaikilla laitteilla.

---

## 🟧 **Tehtävä 6 – Dockerointi (Frontend + Backend + Tietokanta)**

### 🎯 Tavoite

Käynnistä koko sovellus yhdellä komennolla Docker Compose -ympäristössä. 

---

### 🪜 Vaiheittain

## 1) Ympäristömuuttujat

### 1A Jos käytät **lokaalia Postgresia** (ei Dockerissa)

**Tiedosto:** `backend/.env`

```
# Vaihda portti/osoite oman asennuksesi mukaan:
DATABASE_URL="postgresql://postgres:postgres@localhost:5432/plantsdb?schema=public"
JWT_SECRET="supersecret"
PORT=8001
```

> 💡 Kun backend pyörii **Dockerissa**, `localhost` viittaa konttiin, ei koneeseesi.
> 
> - **macOS/Windows (Docker Desktop)**: käytä `host.docker.internal`
>     
> - **Linux**: käytä Dockerin gatewayta `172.17.0.1` (yleensä), tai määritä `extra_hosts`.
>     

**Esim. backend Dockerissa + lokaali Postgres (mac/Win):**

```
DATABASE_URL="postgresql://postgres:postgres@host.docker.internal:5432/plantsdb?schema=public"
```

**Esim. backend Dockerissa + lokaali Postgres (Linux):**

```
DATABASE_URL="postgresql://postgres:postgres@172.17.0.1:5432/plantsdb?schema=public"
```

### 1B. Jos käytät **Docker-Postgresia**

**Tiedosto:** `backend/.env`

```
# HUOM: hostname on compose-palvelun nimi "db"
DATABASE_URL="postgresql://postgres:postgres@db:5432/plantsdb?schema=public"
JWT_SECRET="supersecret"
PORT=8001
```



## 2) Dockerfilet (samat molemmissa malleissa)

**Tiedosto:** `backend/Dockerfile`

```dockerfile
FROM node:20
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npx prisma generate && npm run build
EXPOSE 8001
CMD ["node", "dist/server.js"]
```

**Tiedosto:** `frontend/Dockerfile`

```dockerfile
FROM node:20 AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=build /app/dist /usr/share/nginx/html
EXPOSE 80
```

## 3) docker-compose.yml – molemmat vaihtoehdot profiileilla

**Tiedosto:** `docker-compose.yml`

```yaml
version: "3.9"

services:
  # VAIHTOEHTO A: Docker-Postgres (käynnistyy vain profiililla: db)
  db:
    image: postgres:16
    profiles: ["db"]     # <— HUOM! tämä käynnistyy vain --profile db
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: plantsdb
    ports:
      - "5432:5432"      # valinnainen; poista jos et halua julkaista hostille
    volumes:
      - db_data:/var/lib/postgresql/data

  backend:
    build: ./backend # <-- Muokkaa vastaamaan oman backend kansion nimeä
    environment:
      # HUOM: Kun käytät Docker-Postgresia, osoita "db"-palveluun
      # Kun käytät lokaalisti, yliaja .env:llä (host.docker.internal / 172.17.0.1)
      DATABASE_URL: ${DATABASE_URL}
      JWT_SECRET: ${JWT_SECRET}
      PORT: ${PORT:-8001}
    # Jos käytät Docker-Postgresia, lisää riippuvuudeksi db-profiili:
    #depends_on:
    #  - db
    ports:
      - "8001:8001"
    # Linux: mahdollista auttaa yhteydessä hostin tietokantaan
    # extra_hosts:
    #   - "host.docker.internal:host-gateway"

  frontend:
    build: ./frontend # <-- Muokkaa vastaamaan oman frontend kansion nimeä
    environment:
      # Jos käytät Nginxiä staattiseen palv. ja backend on eri portissa,
      # määritä Vite buildissa osoite, tai käytä samaa originia reverse-proxyn kautta.
      # VITE_API_BASE: "http://localhost:8001/api"
      VITE_API_BASE: "/api"  # jos reverse-proxy käytössä
    depends_on:
      - backend
    ports:
      - "8080:80"

volumes:
  db_data:
```

> 🗒️ **Selitys profiileista:**
> 
> - Käynnistä **Docker-Postgres** profiililla: `--profile db`
>     
> - Jos käytät **lokaalia Postgresia**, ÄLÄ käytä `--profile db` ⇒ `db`-palvelu ei käynnisty.
>     

---

## 4) Käynnistyskomennot

### A) **Docker-Postgres** (kaikki kontissa)

1. Varmista, että `backend/.env` käyttää `db` hostia:
    
    ```
    DATABASE_URL="postgresql://postgres:postgres@db:5432/plantsdb?schema=public"
    ```
    
2. Käynnistä:
    
    ```bash
    docker compose --profile db up --build
    ```
    
3. Aja migraatiot:
    
    ```bash
    docker compose exec backend npx prisma migrate deploy
    ```
    
4. (Valinn.) siemen-data:
    
    ```bash
    docker compose exec backend npm run prisma:seed
    ```
    

### B) **Lokaali Postgres** (vain backend+frontend kontissa)

1. Varmista `backend/.env` osoittaa **hostin** Postgresiin:
    
    - mac/Win:
        
        ```
        DATABASE_URL="postgresql://postgres:postgres@host.docker.internal:5432/plantsdb?schema=public"
        ```
        
    - Linux:
        
        ```
        DATABASE_URL="postgresql://postgres:postgres@172.17.0.1:5432/plantsdb?schema=public"
        ```
        
2. Käynnistä **ilman profiilia**:
    
    ```bash
    docker compose up --build
    ```
    
3. Aja migraatiot hostiin:
    
    ```bash
    docker compose exec backend npx prisma migrate deploy
    ```
    

---

## 5) Vite/Frontend API-osoite

- **Sama origin** (suositeltu): laita Nginx/reverse-proxy ohjaamaan `/api` → backend.  
    Tällöin `frontend/.env`:
    
    ```
    VITE_API_BASE=/api
    ```
    
- **Eri origin**: aseta **CORS** backendissä ja osoita URL:iin:
    
    ```
    VITE_API_BASE=http://localhost:8001/api
    ```
    

> 💡 Varmista, että frontend buildaa oikealla ympäristömuuttujalla (Dockerfile käyttää `npm run build`).

---

## 6) Prisma-komennot (molemmissa malleissa)

**Ensimmäinen ajokerta / skeemamuutokset:**

```bash
# Generoi client
docker compose exec backend npx prisma generate

# Migraatiot (dev tai deploy)
docker compose exec backend npx prisma migrate deploy
# tai kehityksessä:
# docker compose exec backend npx prisma migrate dev --name init
```

---

## 7) Yleiset ongelmat & ratkaisut

- **Backend ei saa yhteyttä tietokantaan**
    
    - Jos käytät **Docker-Postgresia**, varmista `DATABASE_URL` hostiksi `db` (Compose-palvelun nimi).
        
    - Jos käytät **lokaalia Postgresia**, käytä **host.docker.internal** (mac/Win) tai **172.17.0.1** (Linux).
        
- **`ECONNREFUSED 5432`**
    
    - Postgres ei vielä valmis → odota hetki ja yritä migraatiot uudelleen.
        
    - Composition `depends_on` ei odota terveystilaa — voit lisätä healthcheckin tai ajaa migraatiot manuaalisesti hieman myöhemmin.
        
- **CORS-virheet**
    
    - Jos frontend ja backend eri originissa, salli CORS backendissä (`app.use(cors({ origin: "http://localhost:8080" }))`).
        
    - Suosi samaa originia (`/api` reverse-proxy).
        
- **SSL/sertifikaatit** (jos käytät pilvipalvelua)
    
    - Päivitä `DATABASE_URL`-parametreihin `?sslmode=require` tms. hostisi ohjeiden mukaan.
        

---

## 8) Pika-checklist 

-  Päätitkö käyttää **lokaalia** vai **Docker**-Postgresia?
    
-  Asetitko `backend/.env` oikein (HOSTNAME: `db` vs `host.docker.internal`/`172.17.0.1`)?
    
-  Käynnistitko oikealla komennolla (`--profile db` vai ilman)?
    
-  Ajoitko **Prisma migrate** -komennot onnistuneesti?
    
-  Toimiiko frontend–backend yhteys (API BASE, CORS)?
    


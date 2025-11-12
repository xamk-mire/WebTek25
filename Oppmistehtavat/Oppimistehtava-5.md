
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

#### 1️⃣ PostgreSQL-palvelu

**Tiedosto:** `docker-compose.yml` (root kansiossa)

Lisää tietokantapalvelu:

```yaml
services:
  db:
    image: postgres:16
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: plantsdb
    ports:
      - "5432:5432"
    volumes:
      - db_data:/var/lib/postgresql/data
```

#### 2️⃣ Backend Dockerfile

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

#### 3️⃣ Frontend Dockerfile

**Tiedosto:** `frontend/Dockerfile`

```dockerfile
FROM node:20 as build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=build /app/dist /usr/share/nginx/html
EXPOSE 80
```

#### 4️⃣ Käynnistä palvelut

```bash
docker compose up --build
```

Voit myös käyttää Docker Desktop sovellusta docker konttien ajoon.

> 💬 **Miksi:**  
> Docker varmistaa, että sovellus toimii samanlaisessa ympäristössä kaikilla käyttäjillä – riippumatta käyttöjärjestelmästä.

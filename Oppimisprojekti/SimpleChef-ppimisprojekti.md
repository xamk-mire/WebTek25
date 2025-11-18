
# **Johdanto**

SimpleChef on oppimisprojekti, jossa rakennat web-sovelluksen nykyaikaisilla web-teknologioilla joita on käytetty kurssilla. Projektin tavoitteena on harjoitella käytännössä Reactin, TypeScriptin, Express.js:n, PostgreSQL:n, Prisman ja Swaggerin käyttöä sekä toteuttaa moderni kirjautumisjärjestelmä JSON Web Tokeneilla (JWT) ja bcrypt-salasanatiivistyksellä.

Sovellus toimii reseptien jakamiseen tarkoitettuna alustana, jossa käyttäjät voivat rekisteröityä, kirjautua sisään, luoda omia reseptejä, selata muiden reseptejä sekä merkitä mieluisimmat suosikeikseen. Projektissa pääset harjoittelemaan sekä frontend- että backend-kehitystä, tietokantasuunnittelua, REST-rajapintojen toteuttamista ja dokumentointia sekä käyttöliittymän suunnittelua Tailwindin ja DaisyUI:n avulla.

Lopputuloksena syntyy täysin toimiva reseptisovellus, jota voit hyödyntää sekä oppimisen tukena että näyttötyönä omassa portfoliossasi.

---

>[!NOTE]
> Oppimisprojektin tekoon voit hyödyntää kurssilla toteutettua oppimistehtävä projektia sekä harjoituksia

___

# ✅ SimpleChef – Projektitehtävät 

Kaikki tehtävät tehdään **TypeScriptillä**.

---

## 🧩 Tehtävä 1 – Backend-projektin alustus (TypeScript + Express)

### 🎯 Tavoite

Käynnistää täysin toimiva TypeScript-pohjainen Express.js -palvelin, johon voidaan myöhemmin liittää Prisma, Swagger ja autentikointi.

### 📌 Vaatimukset

**Projektirakenne**

- Luo hakemisto `simplechef-backend/`.
    
- Aja `npm init -y`.
    
- Luo kansiorakenne:
    
    ```txt
    src/
      server.ts
      routes/
      middleware/
      swagger.ts
    prisma/
    ```
    

**Asennettavat paketit**

Asenna seuraavat:

```bash
npm install express cors dotenv
npm install swagger-ui-express swagger-jsdoc
npm install @prisma/client pg
npm install jsonwebtoken bcrypt

npm install -D prisma
npm install -D typescript ts-node-dev @types/node @types/express @types/cors @types/swagger-ui-express @types/jsonwebtoken
```

**TypeScript-konfiguraatio**

- Luo `tsconfig.json` (esim. `npx tsc --init`) ja varmista:
    
    - `"strict": true`
        
    - `"rootDir": "src"`
        
    - `"outDir": "dist"`
        
    - `"esModuleInterop": true`
        
    - `"moduleResolution": "node"`
        

**Peruspalvelin**

`src/server.ts`:

- Luo Express-sovellus
    
- Lisää:
    
    - `app.use(cors())`
        
    - `app.use(express.json())`
        
- Lisää reitti:
    
    - `GET /health` → palauttaa `{ status: "ok" }`
        
- Käynnistä palvelin porttiin **3000**.
    

**Käynnistyskomento**

- `package.json`:
    
    ```json
    "scripts": {
      "dev": "ts-node-dev --respawn --transpileOnly src/server.ts"
    }
    ```
    

---

## 🧩 Tehtävä 2 – PostgreSQL & Prisma käyttöönotto

### 🎯 Tavoite

Yhdistää backend Prisma ORM:ään ja PostgreSQL-tietokantaan.

### 📌 Vaatimukset

- Aja:
    
    ```bash
    npx prisma init
    ```
    
- `.env` sisältää:
    
    ```env
    DATABASE_URL="postgresql://user:password@localhost:5432/simplechef"
    JWT_SECRET="jokin_turvallinen_satunnainen_merkkijono"
    ```
    
- Luo PostgreSQL-tietokanta `simplechef` (esim. psql/pgAdmin).
    
- Aja:
    
    ```bash
    npx prisma generate
    ```
    
- Varmista, ettei komento anna virheitä.
    

---

## 🧩 Tehtävä 3 – Prisma Recipe-mallin määrittely

### 🎯 Tavoite

Luoda ensimmäinen tietomalli reseptille.

### 📌 Vaatimukset

**`prisma/schema.prisma` sisältää ainakin:**

```prisma
model Recipe {
  id           Int      @id @default(autoincrement())
  title        String
  description  String?
  ingredients  String
  instructions String
  imageUrl     String?
  createdAt    DateTime @default(now())
}
```

**Migraatio**

- Aja:
    
    ```bash
    npx prisma migrate dev --name init_recipes
    ```
    

**Tarkistus**

- `npx prisma studio` näyttää `Recipe`-taulun
    
- Taulussa on kaikki määritellyt sarakkeet
    

---

## 🧩 Tehtävä 4 – Reseptien CRUD API + Swagger-dokumentointi

### 🎯 Tavoite

Rakentaa resepteille Prisma-pohjainen CRUD API ja dokumentoida se Swaggerilla.

### 📌 Vaatimukset

**Reitit (Express + TypeScript)**

Kansiossa `src/routes/recipes.ts`:

- `GET /api/recipes`
    
    - palauttaa listan kaikista resepteistä (uusin ensin)
        
- `GET /api/recipes/:id`
    
    - palauttaa yksittäisen reseptin id:n mukaan
        
- `POST /api/recipes`
    
    - luo uuden reseptin
        
- `PUT /api/recipes/:id`
    
    - päivittää reseptin
        
- `DELETE /api/recipes/:id`
    
    - poistaa reseptin
        

**Validointi**

- `id` muunnetaan `Number(req.params.id)` ja tarkistetaan `isNaN`
    
- `POST` ja `PUT`:
    
    - vaaditaan `title`, `ingredients`, `instructions`
        
    - virheellisessä syötteessä palautetaan:
        
        ```json
        { "error": "Invalid input" }
        ```
        

**Swagger-konfiguraatio**

- `src/swagger.ts` määrittelee:
    
    - `openapi: "3.0.0"`
        
    - `info` (title, version, description)
        
    - `apis: ["./src/routes/*.ts"]`
        
- `server.ts`:
    
    ```ts
    import swaggerUi from "swagger-ui-express";
    import { swaggerSpec } from "./swagger";
    
    app.use("/api-docs", swaggerUi.serve, swaggerUi.setup(swaggerSpec));
    ```
    

**Swagger-kommentit**

- `recipes.ts` sisältää:
    
    - `components.schemas.Recipe`-määrittelyn
        
    - `@swagger`-kommentit jokaiselle CRUD-reitille
        
    - Kaikki reseptireitit käyttävät tagia: `Recipes`
        

**Tarkistus**

- Swagger UI toimii osoitteessa `http://localhost:3000/api-docs`
    
- `GET /api/recipes` näkyy dokumentoituna
    

---

## 🧩 Tehtävä 5 – Frontend-projektin alustus (React + TS + Tailwind + DaisyUI)

### 🎯 Tavoite

Perustaa moderni React-sovellus TypeScriptillä ja valmiilla UI-tyylikirjastolla.

### 📌 Vaatimukset

**Projekti**

- Luo projektikansio `simplechef-frontend/`
    
- Aja:
    
    ```bash
    npm create vite@latest simplechef-frontend --template react-ts
    ```
    
- Asenna Tailwind + DaisyUI dokumentaation ohjeiden mukaan.
    
- Varmista että yksinkertainen DaisyUI-nappi (`<button className="btn btn-primary">`) näkyy oikein.
    

**Rakenne**

Kansiot:

```txt
src/
  api/
  pages/
  components/
```

**Reititys**

- Asenna:
    
    ```bash
    npm install react-router-dom
    ```
    
- Ota `BrowserRouter` käyttöön:
    
    - alustavat reitit:
        
        - `/` (reseptilista)
            
        - `/recipes/new`
            
        - `/recipes/:id`
            
        - `/recipes/:id/edit`
            
        - `/login`
            
        - `/register`
            
        - `/favorites`
            

**Navigaatio**

- Luo `Navbar.tsx`:
    
    - linkit:
        
        - “Reseptit” → `/`
            
        - “Uusi resepti” → `/recipes/new`
            
        - “Suosikit” → `/favorites`
            
        - “Kirjaudu” / “Kirjaudu ulos” / “Rekisteröidy” (myöhemmin toiminnallisuus)
            

---

## 🧩 Tehtävä 6 – Keskitetty API-kerros (httpClient)

### 🎯 Tavoite

Luoda keskitetty API-kerros, jota kaikki frontendin sivut käyttävät.

### 📌 Vaatimukset

**`src/api/httpClient.ts`**

- Määrittele:
    
    - `const API_BASE_URL = import.meta.env.VITE_API_URL ?? "http://localhost:3000";`
        
- Toteuta funktiot:
    
    - `apiGet<T>(path: string, options?: RequestInit): Promise<T>`
        
    - `apiPost<TBody, TResponse>(path: string, body: TBody, options?: RequestInit): Promise<TResponse>`
        
    - `apiPut<TBody, TResponse>(path: string, body: TBody, options?: RequestInit): Promise<TResponse>`
        
    - `apiDelete<TResponse>(path: string, options?: RequestInit): Promise<TResponse>`
        

**Vaatimukset HTTP-kerrokselle**

- Kaikki funktiot:
    
    - lisäävät `Content-Type: application/json`, jos body
        
    - lukevat mahdollisen JWT-tokenin esim. `localStorage.getItem("authToken")`
        
    - jos token on olemassa, lisäävät headerin:
        
        ```http
        Authorization: Bearer <token>
        ```
        
- Jos vastaus ei ole `res.ok`, heitetään `Error`, jossa viesti otetaan palvelimen palauttamasta `error`-kentästä jos mahdollista.
    

**Rajoitus**

- **Yhdessäkään React-komponentissa ei saa käyttää `fetch`-funktiota suoraan**  
    – kaikki API-kutsut kulkevat `httpClient`-funktion kautta.
    

---

## 🧩 Tehtävä 7 – Resepti-API frontendissä (recipeApi) ja reseptilista-sivu

### 🎯 Tavoite

Määritellä tyypitetty resepti-API-kerros ja käyttää sitä reseptilistan näyttämiseen.

### 📌 Vaatimukset

**`src/api/recipes.ts`**

- Tyypit:
    
    ```ts
    export type Recipe = {
      id: number;
      title: string;
      description?: string;
      ingredients: string;
      instructions: string;
      imageUrl?: string;
      createdAt: string;
    };
    
    export type CreateRecipeDto = Omit<Recipe, "id" | "createdAt">;
    export type UpdateRecipeDto = Partial<CreateRecipeDto>;
    ```
    
- API-olio:
    
    ```ts
    export const recipeApi = {
      getAll: () => apiGet<Recipe[]>("/api/recipes"),
      getById: (id: number) => apiGet<Recipe>(`/api/recipes/${id}`),
      create: (data: CreateRecipeDto) => apiPost<CreateRecipeDto, Recipe>("/api/recipes", data),
      update: (id: number, data: UpdateRecipeDto) => apiPut<UpdateRecipeDto, Recipe>(`/api/recipes/${id}`, data),
      delete: (id: number) => apiDelete<void>(`/api/recipes/${id}`),
    };
    ```
    

**`RecipeListPage.tsx`**

- Käyttää `recipeApi.getAll()` reseptien hakemiseen
    
- Tilat:
    
    - `recipes: Recipe[]`
        
    - `loading: boolean`
        
    - `error: string | null`
        
- Näyttää:
    
    - lataustilan
        
    - virheilmoituksen
        
    - DaisyUI-korttilistan resepteistä (`RecipeCard`-komponentti suositeltava)
        

---

## 🧩 Tehtävä 8 – Reseptin yksityiskohtasivu

### 🎯 Tavoite

Näyttää yksittäisen reseptin tiedot ja mahdollistaa poiston.

### 📌 Vaatimukset

**`RecipeDetailPage.tsx`**

- `useParams` tyypitettynä: `{ id: string }`
    
- Hakee reseptin `recipeApi.getById(Number(id))`
    
- Näyttää:
    
    - otsikon
        
    - kuvan, jos `imageUrl`
        
    - kuvauksen
        
    - ainesosat (esim. rivinvaihdoista listaksi)
        
    - valmistusohjeet
        
- Toiminnot:
    
    - “Muokkaa” → ohjaa `/recipes/:id/edit`
        
    - “Poista”:
        
        - pyytää `confirm`-vahvistuksen
            
        - kutsuu `recipeApi.delete(id)`
            
        - onnistuneesti → ohjaa `/`
            

Käyttää edelleen vain `recipeApi`, ei suoria `fetch`-kutsuja.

---

## 🧩 Tehtävä 9 – Reseptilomake (luonti ja muokkaus)

### 🎯 Tavoite

Laadukas, uudelleenkäytettävä reseptilomake.

### 📌 Vaatimukset

**`RecipeForm.tsx`**

- Props:
    
    ```ts
    type RecipeFormProps = {
      initialData?: CreateRecipeDto;
      onSubmit: (data: CreateRecipeDto) => Promise<void> | void;
      submitLabel?: string;
    };
    ```
    
- Kentät:
    
    - title (required)
        
    - description
        
    - ingredients (textarea)
        
    - instructions (textarea)
        
    - imageUrl
        
- DaisyUI-lomakekomponentit
    
- Lomake EI tee itse API-kutsuja, vaan välittää datan `onSubmit`-funktiolle.
    

**Luontisivu**

- `RecipeCreatePage` (tai `RecipeFormPage` create-tilassa):
    
    - `onSubmit`:
        
        - kutsuu `recipeApi.create`
            
        - uudelleenohjaa detail-sivulle
            

**Muokkaussivu**

- Lataa olemassa olevan reseptin ja välittää `initialData`-propiksi
    
- `onSubmit`:
    
    - kutsuu `recipeApi.update`
        
    - uudelleenohjaa detail-sivulle
        

---

## 🧩 Tehtävä 10 – Haku ja suodatus reseptilistassa

### 🎯 Tavoite

Lisätä client-puoleinen hakutoiminto reseptilistaukseen.

### 📌 Vaatimukset

- `RecipeListPage` sisältää:
    
    - hakukentän (`input className="input input-bordered"`)
        
    - tilan `query: string`
        
- Filtteröinti:
    
    - tehdään ainoastaan client-puolella
        
    - suodatetaan `recipes`-taulukkoa:
        
        - `title`
            
        - `description`
            
    - käyttämällä `toLowerCase().includes(query.toLowerCase())`
        
- Jos suodatettu lista on tyhjä:
    
    - näytetään viesti: “Ei löytynyt reseptejä hakuehdoilla.”
        

---

## 🧩 Tehtävä 11 – Prisma User & Favorite -mallit (auth-valmiit)

### 🎯 Tavoite

Lisätä käyttäjät ja suosikit tietomalliin siten, että käyttäjämalli tukee aitoutettua kirjautumista.

### 📌 Vaatimukset

**`schema.prisma` laajennus:**

```prisma
model User {
  id           Int        @id @default(autoincrement())
  email        String     @unique
  passwordHash String
  createdAt    DateTime   @default(now())
  favorites    Favorite[]
}

model Favorite {
  id        Int      @id @default(autoincrement())
  userId    Int
  recipeId  Int
  createdAt DateTime @default(now())

  user   User   @relation(fields: [userId], references: [id])
  recipe Recipe @relation(fields: [recipeId], references: [id])

  @@unique([userId, recipeId])
}
```

**Migraatio**

- Aja:
    
    ```bash
    npx prisma migrate dev --name add_users_and_favorites
    ```
    

**Tarkistus**

- `User` ja `Favorite` näkyvät Prisma Studiossa
    
- `User.email` on `@unique`
    
- `User` sisältää `passwordHash`-kentän, ei selkokielistä salasanaa
    

---

## 🧩 Tehtävä 12 – Token-pohjainen kirjautuminen (bcrypt + jsonwebtoken)

### 🎯 Tavoite

Toteuttaa JWT-pohjainen kirjautuminen ja rekisteröinti backendissä.

### 📌 Vaatimukset

**Reitit (esim. `src/routes/auth.ts`)**

- `POST /api/auth/register`
    
    - Body: `{ email: string, password: string }`
        
    - Salasana:
        
        - väh. 6 merkkiä (tarkistetaan)
            
        - hashataan `bcrypt`illä (esim. 10 suolakierrosta)
            
    - Jos email varattu → 400 + virheviesti
        
    - Jos ok → luo käyttäjän ja palauttaa:
        
        - `{ token: string }` tai `{ token, user: {...} }`
            
- `POST /api/auth/login`
    
    - Body: `{ email: string, password: string }`
        
    - Hakee käyttäjän emaililla
        
    - Vertailee salasanaa `bcrypt.compare`
        
    - Onnistuneessa kirjautumisessa:
        
        - luo JWT-token (`jsonwebtoken.sign`)
            
        - payload vähintään: `{ userId: number }`
            
        - allekirjoitus `process.env.JWT_SECRET`
            
        - palauttaa `{ token }`
            
    - Virheellisessä tunnuksessa/salasanassa → 401, järkevä virheilmoitus
        

**Autentikaatiomiddleware**

- `src/middleware/auth.ts`:
    
    - lukee `Authorization`-headerin
        
        - muotoa `Bearer <token>`
            
    - jos header puuttuu → 401
        
    - jos token ei kelpaa → 401
        
    - jos ok → lisää `req.user = { id: userId }` (tai vastaava)
        

**Käyttö**

- Lisää `authMiddleware` suojaamaan jatkossa:
    
    - suosikkireitit
        
    - reseptien luonti/muokkaus/poisto (valinnainen, mutta suositeltava)
        

**Swagger**

- Lisää `Auth`-tagi ja dokumentoi:
    
    - `/api/auth/register`
        
    - `/api/auth/login`
        
- Määrittele `bearerAuth` securityScheme:
    
    ```yaml
    components:
      securitySchemes:
        bearerAuth:
          type: http
          scheme: bearer
          bearerFormat: JWT
    ```
    
- Lisää suojatuille reiteille:
    
    ```yaml
    security:
      - bearerAuth: []
    ```
    

---

## 🧩 Tehtävä 13 – Suosikkien backend API (JWT-suojattu) + Swagger

### 🎯 Tavoite

Toteuttaa JWT-suojattu suosikkireittien API.

### 📌 Vaatimukset

**Reitit (esim. `src/routes/favorites.ts`)**

Kaikissa käytettävä `auth`-middlewarea (käyttäjä oltava kirjautunut).

- `POST /api/recipes/:id/favorite`
    
    - lukee `req.user.id` (tyypitettynä)
        
    - tarkistaa että resepti on olemassa
        
    - luo `Favorite`-rivin (jos ei vielä ole)
        
- `DELETE /api/recipes/:id/favorite`
    
    - poistaa käyttäjän kyseisen reseptin suosikin
        
- `GET /api/users/me/favorites`
    
    - palauttaa kirjautuneen käyttäjän suosikit
        
    - sisällyttää reseptit:
        
        ```ts
        prisma.favorite.findMany({
          where: { userId: req.user.id },
          include: { recipe: true },
        })
        ```
        

**Swagger-vaatimukset**

- Tag: `Favorites`
    
- Reitit dokumentoitu Swaggerissa
    
- Kaikilla `security: [ { bearerAuth: [] } ]`
    

---

## 🧩 Tehtävä 14 – Frontend-autentikointi (kirjautuminen, rekisteröinti, tokenin hallinta)

### 🎯 Tavoite

Toteuttaa kirjautumis- ja rekisteröitymisnäkymät, tokenin tallennus sekä tokenin käyttö API-kutsuissa.

### 📌 Vaatimukset

**`src/api/auth.ts`**

- Tyypitetyt funktiot:
    
    ```ts
    type AuthResponse = { token: string };
    
    export const authApi = {
      register: (email: string, password: string) =>
        apiPost<{ email: string; password: string }, AuthResponse>("/api/auth/register", { email, password }),
      login: (email: string, password: string) =>
        apiPost<{ email: string; password: string }, AuthResponse>("/api/auth/login", { email, password }),
    };
    ```
    

**Tokenin tallennus**

- Onnistuneessa login/register -kutsussa:
    
    - tallenna token esim. `localStorage.setItem("authToken", token)`
        
- `httpClient.ts` lukee tokenin ja lisää `Authorization: Bearer ...` kaikkiin pyyntöihin.
    

**Auth-konteksti (suositeltava)**

- Luo `AuthContext`, joka:
    
    - tietää onko käyttäjä kirjautunut (token olemassa)
        
    - tarjoaa funktiot `login`, `logout`, `register`
        

**Sivut**

- `LoginPage.tsx`
    
    - lomake: email + password
        
    - kutsuu `authApi.login`
        
    - tallentaa tokenin
        
    - ohjaa esim. `/`
        
- `RegisterPage.tsx`
    
    - lomake: email + password + passwordConfirm (valinnainen)
        
    - kutsuu `authApi.register`
        
    - tallentaa tokenin
        
    - ohjaa esim. `/`
        

**Käyttöliittymä**

- Navigaatiopalkki:
    
    - jos ei kirjautunut → näytä linkit “Kirjaudu” ja “Rekisteröidy”
        
    - jos kirjautunut → näytä “Kirjaudu ulos” -painike
        

---

## 🧩 Tehtävä 15 – Suosikkien käyttöliittymä + viimeistely

_(Yhdistää aiemmat suosikki- ja viimeistelytehtävät yhdeksi kokonaisuudeksi.)_

### 🎯 Tavoite

Mahdollistaa kirjautuneelle käyttäjälle suosikkien hallinnan ja viimeistellä koko sovellus.

### 📌 Vaatimukset

**Suosikkien UI**

- Lisää reseptikortteihin ja detail-sivulle ❤️-painike:
    
    - jos resepti on käyttäjän suosikeissa → näytä “täytetty” ikoni
        
    - muussa tapauksessa → “tyhjä” ikoni
        
- Käytä `favoriteApi`–moduulia (esim. `getMyFavorites`, `add`, `remove`), joka käyttää `httpClient`-kerrosta ja Authorization-otsaketta.
    
- Luo `FavoritesPage.tsx`, joka:
    
    - kutsuu `favoriteApi.getMyFavorites()`
        
    - näyttää reseptit samalla korttinäkymällä kuin päälista
        

**Reittien suojaus frontendissä**

- Suosikit-sivu ja esim. reseptin luonti/muokkaus voivat edellyttää kirjautumista:
    
    - jos ei tokenia → uudelleenohjaus `/login`
        

**UI-viimeistely**

- Yhtenäinen DaisyUI-tyyli
    
- Perusresponsiivisuus (toimii mobiilissa ja desktopissa)
    
- Selkeät virheilmoitukset (login virhe, API-virheet jne.)
    
- Lataustilat pitkissä API-kutsuissa
    

**Backend-viimeistely**

- Swagger-dokumentaatio päivitetty vastaamaan lopullisia reittejä:
    
    - Recipes
        
    - Auth
        
    - Favorites
        
- Suojatut reitit dokumentoitu `bearerAuth`-securityllä
    

**README**

- Sisältää:
    
    - projektin kuvauksen
        
    - käytetyt teknologiat
        
    - asennusohjeet (backend + frontend + tietokanta)
        
    - ohjeet:
        
        - miten käynnistetään backend
            
        - miten käynnistetään frontend
            
    - maininnan:
        
        - API-dokumentaatio osoitteessa `/api-docs`
            
        - kirjautuminen on **JWT-token**-pohjainen
            
        - frontend käyttää keskitettyä API-kerrosta (`src/api/httpClient.ts`)
            

---

# Harjoitus: Express.js-palvelimen käyttö React-sovelluksessa (TypeScript)


Tässä tehtävässä opit, kuinka yhdistää Express.js React-sovellukseen, jotta voit luoda yksinkertaisen backend-palvelimen, joka toimii React-sovelluksesi kanssa. Käytämme Viteä nopeana kehitystyökaluna, TypeScriptiä tiukempaan tyypitykseen ja Tailwind CSS:ää käyttöliittymän tyylittelyyn.

#### Tavoitteet:

- Luoda React-sovellus Viten avulla
- Lisätä Tailwind CSS sovellukseen
- Luoda yksinkertainen Express.js-pohjainen backend TypeScriptillä
- Yhdistää frontend ja backend niin, että React-sovellus voi tehdä API-kutsuja Express.js-palvelimeen
- Lisää Swagger dokumentointi backend:n api:lle

## Teknologiat

- **React:** Front-end-sovelluskehyksenä.
- **TypeScript**: Muuttujien ja parametrien datan tyypitykseen.
- **Vite:** Nopea kehitystyökalu React-sovelluksille.
- **TypeScript:** JavaScriptin superset, joka lisää tyyppitarkistuksen.
- **Tailwind CSS:** Tyylityökirjasto.
- **Express.js:** Back-end-palvelimen luomiseen.

## Projektin rakenne

Tehtävän toteutuksessa käytetään yksinkertaista rakennetta
```
harjoitus-2/
├─ server/     # Express + TS + Swagger
└─ client/     # React + Vite + TS + Tailwind
```

## Tehtävä 1 – Projektin alustus

**Tavoite:** Saat React + TypeScript -projektin pyörimään.

1. Asenna Node.js (≥ 18) ja npm.  
    Tarkista:
    
    ```bash
    node -v
    npm -v
    ```
    
2. Luo uusi Vite-projekti:

Luo aluksi uusi kansio tehtävän tekoa varten

> [!NOTE]
> Luo client projekti juuri kansioon
> Tarvittaessa vlitse framework:si React ja variant:ksi TypeScript

```bash
npm create vite@latest client -- --template react-ts
cd client
npm install
```

3. Asenna Tailwind CSS

```bash
npm install tailwindcss @tailwindcss/vite
```

Päivitä `vite.config.ts` tiedostoa, lisäämällä sen sisälle `@tailwindcss/vite` pluginin

```ts
import { defineConfig } from 'vite';
import react from "@vitejs/plugin-react";
import tailwindcss from '@tailwindcss/vite';

export default defineConfig({
  plugins: [
    react(),
    tailwindcss(),
  ],
})
```

Lisää `@import` CSS tiedostoon joka importtaa Tailwind CSS:n, esim. `index.css`

```css
@import "tailwindcss";
```

4. Käynnistä kehityspalvelin:
    
```bash
npm run dev
```
    
Avaa selain osoitteessa, jonka Vite näyttää (yleensä `http://localhost:5173`).
    

✅ **Valmis, kun:** Näet Viten oletus “Hello React” -sivun.

---

### Vaihe 2: Express.js-palvelimen luominen

#### 1. **Alusta uusi Node.js-projekti** sovelluksen juurikansioon (eri kansioon client:n kanssa) komennoilla:
```bash
	mkdir server
	cd server
  npm init -y
  npm i express cors swagger-jsdoc swagger-ui-express
  npm i -D typescript ts-node-dev @types/node @types/express @types/cors @types/swagger-jsdoc @types/swagger-ui-express
  npx tsc --init
 ```

#### 2. **Luo TypeScript-konfiguraatio** `server`-kansioon:
```bash
 npx tsc --init
```

Muokkaa `tsconfig.json` -tiedostoa ja varmista, että seuraavat asetukset ovat käytössä:
   
```jsonc
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

#### 3 Päivitä projektin scriptejä 

Muokkaa `server/package.json` tiedostoa lisäämällä sen sisälle seuraavat scriptit

```jsonc
{
  "scripts": {
    "dev": "ts-node-dev --respawn --transpile-only src/index.ts",
    "build": "tsc",
    "start": "node dist/index.js"
  }
}
```

#### 4. Lisää tyyppi tiedostot

Luo uusi `src` kansio ja lisää sen sisälle seuraavat tiedostot

#### `src/types.ts`

```ts
export type Todo = {
  id: string;
  title: string;
  done: boolean;
};
```

#### `src/store.ts`

Ajon aikainen store jota käytetään datan tallentamiseen (simuloi tietokantaa).

```ts
import { Todo } from "./types";

const todos: Todo[] = [
  { id: "1", title: "Learn Express + TS", done: true },
  { id: "2", title: "Hook up React client", done: false }
];

export const Store = {
  list(): Todo[] { return todos; },
  add(todo: Todo) { todos.push(todo); },
  remove(id: string) {
    const idx = todos.findIndex(t => t.id === id);
    if (idx !== -1) todos.splice(idx, 1);
  }
};
```

#### `src/routes/todos.ts`

Express Router (**no Swagger annotations included**) — you will write the Swagger JSDoc yourself as part of the exercise.

```ts
import { Router, Request, Response } from "express";
import { Store } from "../store";
import { Todo } from "../types";
import crypto from "node:crypto";

const router = Router();

// GET /api/todos → list all todos
router.get("/", (_req: Request, res: Response) => {
  res.json(Store.list());
});

// POST /api/todos → create a new todo (requires { title: string })
router.post("/", (req: Request, res: Response) => {
  const title = String(req.body?.title ?? "").trim();
  if (!title) return res.status(400).json({ error: "title is required" });
  const todo: Todo = { id: crypto.randomUUID(), title, done: false };
  Store.add(todo);
  res.status(201).json(todo);
});

// DELETE /api/todos/:id → delete by id
router.delete("/:id", (req: Request, res: Response) => {
  const before = Store.list().length;
  Store.remove(req.params.id);
  const after = Store.list().length;
  if (after === before) return res.status(404).json({ error: "not found" });
  res.status(204).send();
});

export default router;
```
---
### Tehtävä 4.1: lisää Swagger JSDoc -annotaatiot

Lisää **OpenAPI 3** -annotaatiot (JSDoc) **jokaisen reitin yläpuolelle** ja **tiedoston alkuun**, jotta Swagger UI dokumentoi API:n oikein. Käytä `swagger-jsdoc`-konventioita. Kun olet valmis, käynti osoitteessa `/docs` näyttää täysin kuvatut päätepisteet. Tarkemmin:

1. **Tagi**
    

- Määrittele `Todos`-tagi tälle reitittimelle ja lisää lyhyt kuvaus.
    

2. **Skeemat** (`components.schemas`)
    

- Määrittele `Todo`-olio, jossa on kentät: `id` (string), `title` (string), `done` (boolean).
    
- Tee kaikista kolmesta **pakollisia** (`required`) ja lisää **example**-arvot joka kentälle.
    

3. **Yleiset vastaukset** (`components.responses`)
    

- Luo `NotFound`-vastaus koodilla **404** ja lyhyellä kuvauksella.
    

4. **Polut (paths)**
    

- **`GET /api/todos`** → lisää `summary`, liitä `Todos`-tagi, ja määritä **200**-vastaus, joka palauttaa **taulukon `Todo`-olioita**.
    
- **`POST /api/todos`** → lisää `summary`, liitä `Todos`-tagi, määritä `requestBody`, jossa `application/json`-skeema `{ title: string }` (**pakollinen**); määritä **201**-vastaus, joka palauttaa yhden `Todo`-olion; lisää myös **400** virheelliselle syötteelle.
    
- **`DELETE /api/todos/{id}`** → lisää `summary`, liitä `Todos`-tagi, määritä **polkuparametri** `id` (string, **pakollinen**, kuvaus); määritä **204**-vastaus onnistumiselle sekä **404** käyttäen luomaasi `NotFound`-vastausta.
    

5. **Palvelimet (servers)**
    

- Et tarvitse määritellä `servers`-osiota tässä tiedostossa, koska se tulee `src/index.ts`-tiedoston swagger-asetuksista. Varmista kuitenkin, että annotaatiot **kääntyvät `swagger-jsdoc`illa** ja **näkyvät `/docs`**-sivulla.
---

#### `src/index.ts`

Wire up Express, CORS, routes, and Swagger UI.

```ts
import express from "express";
import cors from "cors";
import swaggerJsdoc from "swagger-jsdoc";
import swaggerUi from "swagger-ui-express";
import todosRouter from "./routes/todos";

const app = express();
const PORT = process.env.PORT ?? 4000;

app.use(cors({ origin: ["http://localhost:5173"], credentials: false }));
app.use(express.json());

// Routes
app.use("/api/todos", todosRouter);

// Swagger setup
const swaggerSpec = swaggerJsdoc({
  definition: {
    openapi: "3.0.0",
    info: {
      title: "Todo API (Exercise)",
      version: "1.0.0"
    },
    servers: [{ url: `http://localhost:${PORT}` }]
  },
  apis: ["./src/routes/*.ts"]
});

app.use("/docs", swaggerUi.serve, swaggerUi.setup(swaggerSpec));

app.get("/", (_req, res) => {
  res.send("API is running. See /docs for Swagger UI.");
});

app.listen(PORT, () => {
  // eslint-disable-next-line no-console
  console.log(`Server listening on http://localhost:${PORT}`);
});
```

### 5 Käynnistä palvelin

```bash
npm run dev
```

- Vieraile osoitteessa **[http://localhost:4000/docs](http://localhost:4000/docs)** → sinun pitäisi nähdä Swagger UI käyttöliittymä
    
- Voit nyt testata pavelimellasi löytyviä API reittejä Swagger UI:n avulla
    

### Vaihe 3: Frontend:n ja Backend:n yhdistäminen

#### 1. Lisää client sovellukseesi tyyppi määritelmät `Todo` tyypille

**`src/types.ts`**

```ts
export type Todo = {
  id: string;
  title: string;
  done: boolean;
};
```

#### 2. Lisää seuraavaksi yksinkertainen palvelu (service) komponentti, jonka tehtävä on hakea dataa Express-palvelimeltasi

**`src/api.ts`**

```ts
import type { Todo } from "./types";

const BASE = "http://localhost:4000";

export async function listTodos(): Promise<Todo[]> {
  const res = await fetch(`${BASE}/api/todos`);
  if (!res.ok) throw new Error("Failed to list todos");
  return res.json();
}

export async function createTodo(title: string): Promise<Todo> {
  const res = await fetch(`${BASE}/api/todos`, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ title })
  });
  if (!res.ok) throw new Error("Failed to create todo");
  return res.json();
}

export async function deleteTodo(id: string): Promise<void> {
  const res = await fetch(`${BASE}/api/todos/${id}`, { method: "DELETE" });
  if (!res.ok && res.status !== 204) throw new Error("Failed to delete todo");
}
```

#### 3. Päivitä käyttöliittymä komponettiasi

Päivitä `src/App.tsx` ja toteuta sekä UI että tilanhallinta itse. Tavoitteena on lukea, luoda ja poistaa tehtäviä (todos) API:n kautta ja näyttää ne siistillä Tailwind-käyttöliittymällä.

#### Vaatimukset

- **Tila**: `todos: Todo[]`, `title: string`, `loading: boolean`, `error: string | null`.
    
- **Lataus sivun avauksessa**: kutsu `listTodos()` sisällä `useEffect`, käsittele onnistuminen/virhe, päivitä `loading`.
    
- **Lisääminen**: lomake, jossa tekstikenttä ja lähetä-nappi. Lähetyksessä validoi, että `title` ei ole tyhjä; kutsu `createTodo`, lisää palautettu todo listan alkuun ja tyhjennä syöte. Käsittele virheet.
    
- **Poistaminen**: jokaisella rivillä Poista-nappi; kutsu `deleteTodo(id)` ja poista rivi listasta (optimistisesti tai onnistumisen jälkeen). Käsittele virheet.
    
- **UI**: käytä Tailwindia väleihin, reunoihin, tyhjän tilan viesteihin sekä virhebanneriin. Lisää linkki osoitteeseen `http://localhost:4000/docs`.
    
- **Saavutettavuus**: nimeä syötekenttä (label), varmista että nappia voi käyttää näppäimistöllä, ja ilmoita virheet alueella, jossa on `role="alert"`.
    

```tsx
import { useEffect, useState } from "react";
import { createTodo, deleteTodo, listTodos } from "./api";
import type { Todo } from "./types";

export default function App() {
  const [todos, setTodos] = useState<Todo[]>([]);
  const [title, setTitle] = useState("");
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    listTodos()
      .then(setTodos)
      .catch((e) => setError(e.message))
      .finally(() => setLoading(false));
  }, []);

  async function onAdd(e: React.FormEvent) {
    e.preventDefault();
    if (!title.trim()) return;
    try {
      const created = await createTodo(title.trim());
      setTodos((t) => [created, ...t]);
      setTitle("");
    } catch (e: any) {
      setError(e.message);
    }
  }

  async function onDelete(id: string) {
    try {
      await deleteTodo(id);
      setTodos((t) => t.filter((x) => x.id !== id));
    } catch (e: any) {
      setError(e.message);
    }
  }

  return (
    <div className="min-h-screen bg-gray-50">
      <header className="p-6 shadow bg-white">
        <div className="max-w-3xl mx-auto">
          <h1 className="text-2xl font-bold">Todo App (Express + React + TS)</h1>
          <p className="text-sm text-gray-600">Swagger docs at <a className="underline" href="http://localhost:4000/docs" target="_blank">/docs</a></p>
        </div>
      </header>

      <main className="max-w-3xl mx-auto p-6">
        {/* 1) Lomake: kontrolloitu input + submit-nappi */}
        {/*    - label syötteelle (saavutettavuus) */}
        {/* 2) Virhebanneri (role="alert") kun error != null */}
        {/* 3) Latausindikaattori kun loading == true */}
        {/* 4) Tehtävälista; jokaisessa rivissä Poista-nappi joka kutsuu onDelete */}
        {/* 5) Tyhjän tilan viesti kun listassa ei ole tehtäviä */}
      </main>
    </div>
  );
}
```

#### 4. Käynnistä palvelin

Varmista, että backend/serverisi on myös käynnissä

```bash
npm run dev
```

- Avaa selaimessa **[http://localhost:5173](http://localhost:5173/)**
    
- Lisää ja poista Todo tehtäviä → testaa toimivuus varmuudella käyttämällä Swagger UI:ta (esim. tarkasta että tehtävät varmasti ilmestyvät store:n listaan)

**Jos saat CORS-virheen**, tarkoittaa se että frontend yritti tehdä pyynnön backend-palvelimelle, mutta palvelin ei sallinut pyyntöä toiselta verkkotunnukselta. Tämä on yleinen ongelma, kun frontend ja backend toimivat eri osoitteissa (esim. http://localhost:5173 frontendille ja http://localhost:4000 backendille).    


### Tehtävän viimeistely

Kun olet saanut yhteyden toimimaan, voit muokata sovellusta lisäämällä uusia reittejä Express.js-palvelimelle ja näyttämällä niiden tietoja Reactin käyttöliittymässä.




# Harjoitus 3: Express + Prisma (PostgreSQL) & React + Tailwind (TypeScript)

>[!NOTE]
> Tämä harjoitus on suunniteltu Harjoitus-2 tehtävän jälkeen toteutettavaksi

**Tavoite:** Rakennat täyden Todo-sovelluksen päivittämällä **Harjoitus-2:ssa** toteutettua Todo applikaatiota, jossa backend on **Express + TypeScript + Prisma (PostgreSQL)** ja dokumentointi **Swaggerilla**. Frontend on **React + Vite + TypeScript + Tailwind**. Käytät migraatioita lisätäksesi **status** (done) ja **kategoria** (enum) -ominaisuudet, ja tuot ne näkyviin käyttöliittymässä.

## 0) Tehtävän alustus

Luo harjoituksen toteutusta varten oma kansio ja kopioi toteuttamasi [Harjoitus-2:n](https://github.com/xamk-mire/WebTek25/blob/main/Harjoitukset/Harjoitus-2.md) toteutus sen sisälle, jonka jälkeen voit alkaa toteuttamaan harjoituksen tehtäviä.

## 1) Backend (Express + TS + Prisma + Swagger)

### 1.1 Asennus ja init

Luo env tiedosto: **`server/.env`**

```env
DATABASE_URL="postgresql://postgres:postgres@localhost:5432/todos_dev?schema=public"
PORT=4000
CORS_ORIGIN=http://localhost:5173
```

`DATABASE_URL` on Prismalle rajapinta DB:hen. `PORT` palvelimen portti. `CORS_ORIGIN` sallittu frontend‑osoite.

**Asenna ja initialisoi Prisma**

```bash
npm i @prisma/client
npm i -D prisma
npx prisma init
```

Asenna Prisma client (runtime) ja Prisma CLI (dev). `prisma init` luo `prisma/schema.prisma` ja `.env`.
### 1.2 Prisma & .env

```bash
npx prisma init
```


### 1.3 Migraatiot vaiheittain

Teet **kolme** migraatiota, jotta muutokset ovat läpinäkyviä.

**A) Alkumalli** – `init_todos`  
**`prisma/schema.prisma`**

```prisma
generator client { provider = "prisma-client-js" }

datasource db { provider = "postgresql"; url = env("DATABASE_URL") }

model Todo {
  id        String   @id @default(uuid())
  title     String
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}
```

Aja:

```bash
npm run prisma:migrate -- --name init_todos
npm run prisma:generate
```

`migrate dev` luo ja ajaa migraation; `generate` tuottaa TypeScript‑clientin.

**B) Status (done)** – `add_done_status`  
Lisää `done` oletuksella `false`:

```prisma
model Todo {
  id        String   @id @default(uuid())
  title     String
  done      Boolean  @default(false)
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}
```

Aja:

```bash
npm run prisma:migrate -- --name add_done_status
```


**C) Kategoria (enum)** – `add_category_enum`  
Lisää enum ja sarake oletuksella `GENERAL`:

```prisma
enum Category { GENERAL WORK STUDY HOME }

model Todo {
  id        String   @id @default(uuid())
  title     String
  done      Boolean  @default(false)
  category  Category @default(GENERAL)
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}
```

Aja:

```bash
npm run prisma:migrate -- --name add_category_enum
npm run prisma:generate
```

Uusi enum tallentuu skeemaan ja taulu päivittyy. `generate` päivittää tyypit.

> **Tarkista käyttämällä prisma studiota**: `npm run prisma:studio` → näet `Todo`-taulun.

## 2 Prisman kytkentä Expressiin

### 2.1 Prisma-client

Lisää tai päivitä db.ts tiedostoa **`src/db.ts`** – Prisma-client

```ts
import { PrismaClient } from "@prisma/client";
export const prisma = new PrismaClient({ log: ["warn", "error"] });
```

Yksi PrismaClient‑instanssi jaetaan reittien kesken.

### 2.2 Toteuta/Päivitä reitit (server/src/routes/todos.ts)

**`src/routes/todos.ts`** – Kirjoita reitit **itse** käyttäen Prisma-kutsuja. Vähintään:

- `GET /api/todos` – listaus + suodatus `?category=...&done=true|false`
```ts
router.get('/', async (req: Request, res: Response) => {

  const items = await prisma.todo.findMany({

    orderBy: { createdAt: 'desc' },

  });

  

  res.json(items);

});
```
    
- `POST /api/todos` – luonti `{ title, category? }`
```ts
router.put('/:id', async (req: Request, res: Response) => {

  const { id } = req.params;

  const { title, done, category } = req.body as {

    title?: string;

    done?: boolean;

    category?: 'GENERAL' | 'WORK' | 'STUDY' | 'HOME';

  };

  

  try {

    const updated = await prisma.todo.update({

      where: { id },

      data: { title, done, category },

    });

    res.json(updated);

  } catch (e: any) {

    res.status(404).json({ error: 'not found' });

  }

});
```
    
- `PUT /api/todos/:id` – päivitys `{ title?, done?, category? }`
```ts
router.post('/', async (req: Request, res: Response) => {

  const title = String(req.body?.title ?? '').trim();

  const category = req.body?.category as

    | 'GENERAL'

    | 'WORK'

    | 'STUDY'

    | 'HOME'

    | undefined;

  if (!title) return res.status(400).json({ error: 'title is required' });

  try {

    const created = await prisma.todo.create({

      data: {

        title,

        category, // jos undefined, Prisma käyttää oletusta GENERAL

      },

    });

    res.status(201).json(created);

  } catch (e: any) {

    res.status(400).json({ error: 'invalid payload' });

  }

});
```
    
- `DELETE /api/todos/:id` – poisto
```ts
router.delete('/:id', async (req: Request, res: Response) => {

  const { id } = req.params;

  try {

    await prisma.todo.delete({ where: { id } });

    res.status(204).send();

  } catch {

    res.status(404).json({ error: 'not found' });

  }

});
```
    

**Vinkki:** Käytä try/catch ja palauta 400/404 tilanteen mukaan.
### 2.4 API‑dokumentaatio – Swagger (kirjoitat itse)

Lisää tai päivitä **Swagger JSDoc**‑annotaatiot `src/routes/todos.ts`:

- `components.schemas`: `Category` (enum) ja `Todo` (id, title, done, category, createdAt, updatedAt)
    
- Polut: `GET /api/todos` (query parametrit), `POST /api/todos` (requestBody + 201), `PUT /api/todos/{id}` (requestBody + 200/404), `DELETE /api/todos/{id}` (204/404)
    

**Tavoite:** `/docs` näyttää kaikki endpointit, skeemat ja esimerkit; Try‑it‑out toimii.

---

## 3) Frontend (React + Vite + TS + Tailwind)

### 2.1 Tyypit & API-kutsut

**`src/types.ts`** – käytä **enumia** kategoriassa:

```ts
export enum Category {
  GENERAL = "GENERAL",
  WORK = "WORK",
  STUDY = "STUDY",
  HOME = "HOME",
}

export type Todo = {
  id: string;
  title: string;
  done: boolean;
  category: Category;
  createdAt: string;
  updatedAt: string;
};
```


### 2.2 API-apufunktiot

**`src/api.ts`** – kirjoita tai päivitä itse funktiot `listTodos`, `createTodo`, `updateTodo`, `deleteTodo` käyttäen `fetch`iä. Huomioi:

- `listTodos(): Promise<Todo[]>` 
    
- `createTodo(payload: { title: string; category?: Category }): Promise<Todo>` POST: JSON body
    
- `updateTodo(id: string, patch: Partial<Pick<Todo, "title"|"done"|"category">>): Promise<Todo>` PUT
    
- `deleteTodo(id: string): Promise<void>` DELETE
    

### 2.3 App käyttöliittymän päivitys

**`src/App.tsx`** – tee UI seuraavin vaatimuksin (ulkoasun toteutus vapaa):

- **Tila:** `todos`, `title`, `category`, `loading`, `busy`, `error`.
    
- **Lataus:** hae lista komponentin mountissa (`listTodos`).
    
- **Luonti:** ohjattu syöte + `select` kategoriasta; `createTodo` form‑submitissä; lisää uusi alkuun.
    
- **Status:** checkbox/toggle muuttaa `done`; kutsu `updateTodo` (optimistinen päivitys ok).
    
- **Kategoria:** rivikohtainen `select`; kutsu `updateTodo`.
    
- **UX:** virhebanneri (`role="alert"`), latausindikaattori, tyhjän tilan viestit, labelit.
    
- **Tyyli:** Tailwind‑kortit, spacing, borderit, rounded.


> Saat ohjenuoraksi kevyen skeletonin aiemmasta tehtävästä – varsinainen toteutus jää sinulle.

### 2.4 Käynnistys ja testaus

```bash
npm run dev
# http://localhost:5173
```

---

## ) Päättarkistukset (end‑to‑end)

- Backend pyörii `http://localhost:4000`, ja `/health` ok
    
- Migraatiot ajettu: `init_todos`, `add_done_status`, `add_category_enum`
    
- Prisma Client generoitu; reitit käyttävät Prismaa (ei in‑memory Storea)
    
- Swagger `/docs` kuvaa enumit, mallit, queryt, rungot ja vastaukset
    
- Frontend näyttää **statuksen** ja **kategorian** sekä tukee luonti/toggle/kategorian vaihto/poisto

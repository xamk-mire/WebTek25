# Oppimistehtävä: Plants Dashboard (React + TypeScript + CSS)

Rakennetaan React-sovellus, joka näyttää kasvilistan, mahdollistaa kastelun päälle/pois-kytkemisen ja yksityiskohtaisen näkymän yhdelle kasville.  

---

## Tehtävä 1 – Projektin alustus

**Tavoite:** Saat React + TypeScript -projektin pyörimään.

1. Asenna Node.js (≥ 18) ja npm.  
    Tarkista:
    
    ```bash
    node -v
    npm -v
    ```
    
2. Luo uusi Vite-projekti:

> [!NOTE]
> Tarvittaessa vlitse framework:si React ja variant:ksi TypeScript


    ```bash
    npm create vite@latest plants-dashboard -- --template react-ts
    cd plants-dashboard
    npm install
    ```
    
3. Käynnistä kehityspalvelin:
    
    ```bash
    npm run dev
    ```
    
    Avaa selain osoitteessa, jonka Vite näyttää (yleensä `http://localhost:5173`).
    

✅ **Valmis, kun:** Näet Viten oletus “Hello React” -sivun.

---

## Tehtävä 2 – Tietotyypit ja Mock API

**Tavoite:** Määrittele kasvin ja laitteen tietomallit sekä toteuta yksinkertainen mock-API, joka simuloi backendin vastauksia.

### Vaihe 1 – Tyypit

**`src/lib/types.ts`**

```ts
export type DeviceStatus = "ok" | "fault" | "offline";

export type Device = {
  id: number;
  status: DeviceStatus;
  battery: number;
  watering: boolean;
  moisture: number;
  created_at: string;
  updated_at: string;
};

export type Plant = {
  id: number;
  name: string;
  species: string;
  location?: string | null;
  notes?: string | null;
  device: Device;
};
```

### Vaihe 2 – Mock API

**`src/lib/api.mock.ts`**

```ts
import type { Plant, Device } from "./types";

const now = () => new Date().toISOString();
const rnd = (a: number, b: number) => +(a + Math.random() * (b - a)).toFixed(1);

let plants: Plant[] = [
  {
    id: 1,
    name: "Monstera - Office",
    species: "monstera",
    location: "Office",
    device: { id: 1, status: "ok", battery: 92, watering: false, moisture: rnd(45, 60), created_at: now(), updated_at: now() },
  },
  {
    id: 2,
    name: "Fiddle Leaf Fig",
    species: "ficus",
    location: "Living Room",
    device: { id: 2, status: "ok", battery: 87, watering: false, moisture: rnd(35, 50), created_at: now(), updated_at: now() },
  },
  {
    id: 3,
    name: "Cactus",
    species: "cactus",
    location: "Kitchen",
    device: { id: 3, status: "ok", battery: 96, watering: false, moisture: rnd(15, 30), created_at: now(), updated_at: now() },
  },
];

const delay = (ms=250) => new Promise(r => setTimeout(r, ms));

export async function listPlants(): Promise<Plant[]> {
  await delay();
  return JSON.parse(JSON.stringify(plants));
}

export async function getPlant(id: number): Promise<Plant> {
  await delay();
  const plant = plants.find(p => p.id === id);
  if (!plant) throw new Error("Plant not found");
  return JSON.parse(JSON.stringify(plant));
}

export async function togglePlantWatering(id: number): Promise<Plant> {
  await delay(200);
  const i = plants.findIndex(p => p.id === id);
  if (i < 0) throw new Error("Plant not found");
  const plant = plants[i];
  const device: Device = {
    ...plant.device,
    watering: !plant.device.watering,
    updated_at: now(),
  };
  plants[i] = { ...plant, device };
  return JSON.parse(JSON.stringify(plants[i]));
}
```

### Vaihe 3 – API base

**`src/lib/api.base.ts`**

```ts
import * as mock from "./api.mock";

export const API = {
  listPlants: mock.listPlants,
  getPlant: mock.getPlant,
  togglePlantWatering: mock.togglePlantWatering,
};
```

✅ **Valmis, kun:** Voit kokeilla `API.listPlants()` funktiota esim. `console.log`illa ja selaimen konsoliin tulostuu listaus kasveista.

---

## Tehtävä 3 – Kasvilista (PlantsPage)

**Tavoite:** Näytä kaikki kasvit kortteina.

### 1. Luo komponentti `src/components/PlantCard.tsx`, joka näyttää:
    
    - kasvin nimen, lajin, sijainnin,
        
    - laitteen tilan, kosteuden, akun ja kastelun,
        
    - napit “Start/Stop watering” ja “Open”.


### Malliesimerkki — Kortin rakenne (BookCard → sinulla PlantCard)

```tsx
// BookCard.tsx (MALLI — muokkaa omaan domainiin)
import type { Book } from "../lib/library.types";

export function BookCard({
  book,
  onOpen,
  onToggleFavorite,
}: {
  book: Book;
  onOpen: (id: number) => void;
  onToggleFavorite: (id: number) => void;
}) {
  const shelfBadge =
    book.shelfStatus === "available" ? "badge badge--ok" :
    book.shelfStatus === "damaged"   ? "badge badge--fault" :
                                       "badge badge--offline"; // reused colors

  return (
    <div className="card">
      <div className="card__row">
        <div>
          <h3 className="card__title">{book.title}</h3>
          <p className="card__subtitle">{book.author} · {book.location ?? "—"}</p>
        </div>
        <span className={shelfBadge}>{book.shelfStatus}</span>
      </div>

      <div className="kv-grid">
        <div>Pages: <strong>{book.pages}</strong></div>
        <div>Rating: <strong>{book.rating}/5</strong></div>
        <div>Favorite: <strong>{book.favorite ? "YES" : "NO"}</strong></div>
        <div>Updated: <span className="mono">{new Date(book.updated_at).toLocaleString()}</span></div>
      </div>

      <div className="btn-row">
        <button
          className={`btn ${book.favorite ? "btn--danger" : ""}`}
          onClick={() => onToggleFavorite(book.id)}
        >
          {book.favorite ? "Unfavorite" : "Favorite"}
        </button>
        <button className="btn btn--secondary" onClick={() => onOpen(book.id)}>
          Open
        </button>
      </div>
    </div>
  );
}
```


        
### 2. Luo sivu `src/pages/PlantsPage.tsx`:
    
    - hae kasvit `API.listPlants()`-funktiolla,
        
    - renderöi kasvikortit `.grid`-layoutilla,
        
    - kytke “toggle”-nappi `API.togglePlantWatering(id)`-funktioon.
        

### Malliesimerkki —  (Books-teema)

```tsx
// BooksPage.tsx (MALLI — muokkaa omaan domainiin)
import { useEffect, useState } from "react";
import { LibraryAPI } from "../lib/library.api";
import type { Book } from "../lib/library.types";

export function BooksPage() {
  const [books, setBooks] = useState<Book[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError]   = useState<string | null>(null);

  useEffect(() => {
    (async () => {
      try {
        setLoading(true);
        const data = await LibraryAPI.listBooks();   // <- API.listPlants()
        setBooks(data);
      } catch (e: any) {
        setError(e.message || String(e));
      } finally {
        setLoading(false);
      }
    })();
  }, []);

  async function toggleFavorite(id: number) {        // <- toggle watering
    // 1) Optimistinen UI
    setBooks(prev => prev.map(b => b.id === id ? { ...b, favorite: !b.favorite } : b));
    try {
      // 2) Serveri (mock/real)
      const updated = await LibraryAPI.toggleFavorite(id); // <- API.togglePlantWatering(id)
      // 3) Päivitä kirjojen tila
      setBooks(prev => prev.map(b => b.id === id ? updated : b));
    } catch (e: any) {
      // 4) Virhe → palauta/ilmoita
      setError(e.message || String(e));
      // helppo palautus: refetch
      const data = await LibraryAPI.listBooks();
      setBooks(data);
    }
  }

  return (
    <div className="container">
      <header className="header">
        <h1 className="header__title">Books</h1>
        <p className="header__subtitle">Mock mode – swap to real API later</p>
      </header>

      <div className="grid">
        {books.map(book => (
          <BookCard
            key={book.id}
            book={book}
            onOpen={(id) => console.log("Open", id)}   // <- Task 5: korvaa navigoinnilla
            onToggleFavorite={toggleFavorite}          // <- onToggleWatering
          />
        ))}
      </div>
    </div>
  );
}
```

### 3. Voit testata `PlantsPage` sivun näkymää, kutsumalla komponenttia `App.tsx` tiedoston sisällä


```tsx
export default function App() {
  return (
    <PlantsPage></PlantsPage>
  );
}
```


**Sovella omaan projektiin:**

- `Book` → **Plant**
    
- `favorite` → **watering**
    
- `shelfStatus` → **device.status**
    
- `LibraryAPI.listBooks()` → **API.listPlants()**
    
- `LibraryAPI.toggleFavorite(id)` → **API.togglePlantWatering(id)**
    


✅ **Valmis, kun:** Sovelluksen aloitussivulla näkyy lista kasveista ja kastelun tila vaihtuu nappia painamalla.

---

## Tehtävä 4 – Kasvin yksityiskohtasivu (PlantDetailPage)

**Tavoite:** Näytä yhden kasvin laitetiedot.

1. Luo sivu `src/pages/PlantDetailPage.tsx`:
    
    - Hae kasvi `API.getPlant(id)` (voit kovakoodata testiksi esim. `id = 1` aluksi),
        
    - Näytä kasvin nimi, laji ja sijainti,
        
    - Näytä laitteen kentät: id, status, moisture, battery, watering, updated_at.
        


### Malliesimerkki — kovakoodattu id ja tietojen näyttö

```tsx
// BookDetailPage.tsx (MALLI — muokkaa omaan domainiin)
import { useEffect, useState } from "react";
import { LibraryAPI } from "../lib/library.api";
import type { Book } from "../lib/library.types";

export function BookDetailPage() {
  const [book, setBook]     = useState<Book | null>(null);
  const [loading, setLoad]  = useState(true);
  const [error, setError]   = useState<string | null>(null);

  const id = 1; // testaa eri arvoilla; teillä myöhemmin route-param

  useEffect(() => {
    let alive = true;
    (async () => {
      try {
        setLoad(true);
        const b = await LibraryAPI.getBook(id); // <- API.getPlant(id)
        if (alive) setBook(b);
      } catch (e: any) {
        setError(e.message || String(e));
      } finally {
        setLoad(false);
      }
    })();
    return () => { alive = false; };
  }, [id]);


  return (
    <div className="container">
      <header className="header">
        <h1 className="header__title">{book.title}</h1>
        <p className="header__subtitle">{book.author} · {book.location ?? "—"}</p>
      </header>

      <div className="card">
        <h3 className="section__title">Shelf</h3>
        <div className="kv-grid">
          <div>ID: <code className="mono">{book.id}</code></div>
          <div>Status: <strong>{book.shelfStatus}</strong></div>
          <div>Pages: <strong>{book.pages}</strong></div>
          <div>Rating: <strong>{book.rating}/5</strong></div>
          <div>Favorite: <strong>{book.favorite ? "YES" : "NO"}</strong></div>
          <div>Updated: <span className="mono">{new Date(book.updated_at).toLocaleString()}</span></div>
        </div>
      </div>
    </div>
  );
}
```

**Sovella omaan projektiin:**

- `Book` → **Plant**
    
- “Shelf”-kortti → **Device**-kortti (id, status, moisture, battery, watering, updated)
    


✅ **Valmis, kun:** Näet kovakoodatun kasvin yksityiskohtaiset tiedot sivulla.

---

## Tehtävä 5 – React Router

**Tavoite:** Lisää navigointi kasvilistan ja detail-sivun välille.

1. Asenna React Router:
    
    ```bash
    npm i react-router-dom
    ```
    
2. Muokkaa `src/app/App.tsx`:
    
    - määritä reitit:
        
        - `/plants` → `PlantsPage`
            
        - `/plants/:id` → `PlantDetailPage`
            
        - `/` → uudelleenohjaus `/plants`
            
    - lisää fallback `*` → “Not found”.


**A) Reititys Appissa**

```tsx
// App.tsx (MALLI — muokkaa omaan domainiin)
<BrowserRouter>
  <Routes>
    <Route path="/" element={<Navigate to="/books" replace />} />
    <Route path="/books" element={<BooksPage />} />
    <Route path="/books/:id" element={<BookDetailPage />} />
    <Route path="*" element={<div className="container"><p>Not found</p></div>} />
  </Routes>
</BrowserRouter>
```


        
3. Muokkaa `PlantCard`-komponenttia niin, että “Open”-nappi navigoi `/plants/:id`-reitille.


**B) Navigointi kortista detailiin**

```tsx
// BooksPage.tsx
const nav = useNavigate();
function onOpen(id: number) {
  nav(`/books/${id}`);      // <- teillä: nav(`/plants/${id}`)
}
```


    
4. Muokkaa `PlantDetailPage`-komponenttia:
    
    - hae `id` `useParams`-hookilla,
        
    - muunna `id` numeroksi `parseInt`,
        
    - hae oikea kasvi API:sta.


**C) Parametrin luku detail-sivulla**

```tsx
// BookDetailPage.tsx
const { id: idParam } = useParams<{ id: string }>();
const id = idParam ? parseInt(idParam, 10) : NaN;
if (Number.isNaN(id)) {
  // näytä virhe tai ohjaa listaan
}
```



✅ **Valmis, kun:** Klikkaamalla kasvikortin “Open”-painiketta avautuu kasvin detail-sivu.

---

## Tehtävä 6 – Lataus, virheet ja viimeistely

**Tavoite:** Viimeistele käytettävyys ja tyylit.

1. Näytä “Loading…” kun dataa haetaan.

2. Näytä virheilmoitus (`.alert`) jos API heittää virheen.

**A) Lataus/virhe**

```tsx
{loading && <p>Loading…</p>}
{error && <p className="alert">{error}</p>}
```


3. Varmista, että kaikki tyylit ovat `globals.css`-tiedostossa, ei inline-tyylejä.
    
4. Lisää peruslayout- ja korttityylit (`.container`, `.grid`, `.card`, `.btn`, `.badge`).


**B) CSS-luokkia (sama ideapohja kuin Tailwind CSS:ssä)**

```css
/* globals.css — lyhyt idea, laajenna itse */
.container{max-width:960px;margin:0 auto;padding:24px}
.header{margin-bottom:16px}
.header__title{margin:0;font-size:24px;font-weight:700}
.header__subtitle{margin:0;color:#6b7280;font-size:14px}

.grid{display:grid;grid-template-columns:repeat(auto-fill,minmax(280px,1fr));gap:16px}
.card{background:#fff;border:1px solid #e5e7eb;border-radius:12px;padding:16px}
.card__row{display:flex;justify-content:space-between;align-items:flex-start;margin-bottom:8px}
.card__title{margin:0;font-weight:700}
.card__subtitle{margin:0;font-size:12px;color:#6b7280}

.kv-grid{display:grid;grid-template-columns:repeat(2,1fr);gap:8px;font-size:14px}

.btn{border:0;border-radius:8px;padding:8px 12px;background:#2563eb;color:#fff;font-weight:600;cursor:pointer}
.btn--secondary{background:#111827}
.btn--danger{background:#dc2626}
.btn-row{display:flex;gap:8px;margin-top:12px}

.badge{display:inline-block;border-radius:999px;padding:2px 8px;font-size:12px}
.badge--ok{background:#ecfdf5;color:#10b981}
.badge--fault{background:#fffbeb;color:#f59e0b}
.badge--offline{background:#fef2f2;color:#ef4444}

.alert{background:#fef2f2;color:#991b1b;padding:8px;border-radius:8px}
.mono{font-family:ui-monospace,SFMono-Regular,Menlo,Consolas,monospace}
```


✅ **Valmis, kun:**

- Listasivu näyttää kortit siististi,
    
- Yksityiskohtasivu toimii reittiparametrilla,
    
- Virhe- ja lataustilat näkyvät oikein.
    
        

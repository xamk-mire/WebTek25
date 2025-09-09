# Oppimistehtävä: Plants Dashboard (React + TypeScript + Tailwind CSS)

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
// BookCard.tsx (MALLI – käännä PlantCardiksi)
export function BookCard({ book, onOpen, onToggleFavorite }: Props) {
  const badge =
    book.shelfStatus === "available" ? "bg-emerald-50 text-emerald-700" :
    book.shelfStatus === "damaged"   ? "bg-amber-50 text-amber-700"    :
                                       "bg-rose-50 text-rose-700";

  return (
    <div className="rounded-xl border border-slate-200 bg-white p-4 shadow-sm">
      <div className="mb-3 flex items-start justify-between">
        <div>
          <h3 className="font-semibold">{book.title}</h3>
          <p className="text-xs text-slate-500">{book.author} · {book.location ?? "—"}</p>
        </div>
        <span className={`inline-flex rounded-full px-2 py-0.5 text-xs ${badge}`}>
          {book.shelfStatus}
        </span>
      </div>

      <dl className="grid grid-cols-2 gap-2 text-sm">
        <div>Pages: <span className="font-semibold">{book.pages}</span></div>
        <div>Rating: <span className="font-semibold">{book.rating}/5</span></div>
        <div>Favorite: <span className="font-semibold">{book.favorite ? "YES" : "NO"}</span></div>
        <div>Updated: <span className="font-mono">{new Date(book.updated_at).toLocaleString()}</span></div>
      </dl>

      <div className="mt-3 flex gap-2">
        <button
          className={`rounded-lg px-3 py-2 text-white ${book.favorite ? "bg-rose-600" : "bg-emerald-600"}`}
          onClick={() => onToggleFavorite(book.id)}
        >
          {book.favorite ? "Unfavorite" : "Favorite"}
        </button>
        <button
          className="rounded-lg bg-slate-900 px-3 py-2 text-white"
          onClick={() => onOpen(book.id)}
        >
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

  async function handleToggleFav(id: number) {
  setBooks(prev => prev.map(b => b.id === id ? { ...b, favorite: !b.favorite } : b));
  try {
    const updated = await LibraryAPI.toggleFavorite(id);   // API.togglePlantWatering(id)
    setBooks(prev => prev.map(b => (b.id === id ? updated : b)));
  } catch (e: any) {
    setError(e.message || String(e));
    const fresh = await LibraryAPI.listBooks();            // API.listPlants()
    setBooks(fresh);
  }
}

 // BooksPage.tsx (MALLI – käännä omalle domainille)
return (
  <div className="mx-auto max-w-5xl p-6">
    <header className="mb-6">
      <h1 className="text-2xl font-bold">Books</h1>
      <p className="text-sm text-slate-500">Mock mode – swap to real API later</p>
    </header>

    <div className="grid grid-cols-1 gap-4 sm:grid-cols-2 lg:grid-cols-3">
      {books.map((b) => (
        <BookCard
          key={b.id}
          book={b}
          onOpen={(id) => console.log("Open", id)}         // nav(`/plants/${id}`)
          onToggleFavorite={(id) => handleToggleFav(id)}   // toggle watering
        />
      ))}
    </div>
  </div>
);

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

 const id = 1;
useEffect(() => {
  let alive = true;
  (async () => {
    try {
      setLoading(true);
      const item = await LibraryAPI.getBook(id); // teillä: API.getPlant(id)
      if (alive) setBook(item);
    } catch (e: any) {
      setError(e.message || String(e));
    } finally {
      setLoading(false);
    }
  })();
  return () => { alive = false; };
}, [id]);


  return (
    <div className="container">
     <header className="mb-6">
          <h1 className="text-2xl font-bold">{book.title}</h1>
          <p className="text-sm text-slate-500">{book.author} · {book.location ?? "—"}</p>
        </header>
        
        <div className="rounded-xl border border-slate-200 bg-white p-4 shadow-sm">
          <h3 className="mb-2 text-base font-semibold">Shelf</h3>
          <dl className="grid grid-cols-2 gap-2 text-sm">
            <div>ID: <span className="font-mono">{book.id}</span></div>
            <div>Status: <span className="font-semibold">{book.shelfStatus}</span></div>
            <div>Pages: <span className="font-semibold">{book.pages}</span></div>
            <div>Rating: <span className="font-semibold">{book.rating}/5</span></div>
            <div>Favorite: <span className="font-semibold">{book.favorite ? "YES" : "NO"}</span></div>
            <div>Updated: <span className="font-mono">{new Date(book.updated_at).toLocaleString()}</span></div>
          </dl>
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


**Tavoite:** Viimeistele ominaisuudet ja UX Tailwindilla.

### Mitä sinun tulee toteuttaa (Plants)

- Selkeät **loading**- ja **error**-tilat sekä listassa että detailissä.
    
- Yhtenäinen ulkoasu Tailwind-luokilla — ei omia CSS-tiedostoja.
    
- Painikkeiden tilat (esim. “Stop watering” punainen, “Water” vihreä).
    
- Badge-tyyli laitteen tilalle (ok/fault/offline) väreillä.
    

### Mallipalapalat (Books)

**A) Loading & error**

```tsx
{loading && <p className="text-slate-500">Loading…</p>}
{error && <p className="rounded-md bg-rose-50 p-3 text-rose-700">{error}</p>}
```

**B) Tilabadge valinnan kautta**

```tsx
const badge =
  status === "available" ? "bg-emerald-50 text-emerald-700" :
  status === "damaged"   ? "bg-amber-50 text-amber-700"    :
                           "bg-rose-50 text-rose-700";
<span className={`inline-flex rounded-full px-2 py-0.5 text-xs ${badge}`}>{status}</span>
```

**C) Responsiivinen ruudukko ja kortit**

```tsx
<div className="grid grid-cols-1 gap-4 sm:grid-cols-2 lg:grid-cols-3">
  {/* cards */}
</div>

<div className="rounded-xl border border-slate-200 bg-white p-4 shadow-sm">
  {/* content */}
</div>
```

**D) Nappien kontrastit**

```tsx
<button className="rounded-lg bg-emerald-600 px-3 py-2 text-white hover:bg-emerald-700">
  Primary Action
</button>
<button className="rounded-lg bg-slate-900 px-3 py-2 text-white hover:bg-black">
  Secondary
</button>
<button className="rounded-lg bg-rose-600 px-3 py-2 text-white hover:bg-rose-700">
  Danger
</button>
```


✅ **Valmis, kun:**

- Listasivu näyttää kortit siististi,
    
- Yksityiskohtasivu toimii reittiparametrilla,
    
- Virhe- ja lataustilat näkyvät oikein. Voit testata virhe- ja lataustiloja pakottamalla, esim. asettamalla `loading` muuttujan arvon manuaalisesti `true` :ksi
    
        

# Harjoitus 4: UI‑päivitys daisyUI + shadcn/ui

>[!NOTE]
> Tämä harjoitus on suunniteltu Harjoitus-3 tehtävän jälkeen toteutettavaksi

**Tavoite:** Jatka Todo‑sovelluksen kehitystä päivittämällä käyttöliittymä käyttämään sekä **daisyUI**‑komponentteja (teemat, peruskomponentit) että **shadcn/ui**‑komponentteja (rakenteelliset, saavutettavat primitiivit).

## 0) Tehtävän alustus

Luo harjoituksen toteutusta varten oma kansio ja kopioi toteuttamasi [Harjoitus-3:n](https://github.com/xamk-mire/WebTek25/blob/main/Harjoitukset/Harjoitus-3.md) toteutus sen sisälle, jonka jälkeen voit alkaa toteuttamaan harjoituksen tehtäviä.

## 1) Tailwind:n asennus

Varmista, että Tailwind v4 on käytössä ja CSS‑tiedostossa on perusimport:

**`src/index.css`** tai **`src/app.css`**

```css
@import "tailwindcss";
```

**Tavoite:** Tailwind 4.0/4.1 tuo uuden tavan lisätä plugineita suoraan CSS:ään, ja minimoi config‑tiedostojen tarpeen.

---
## 2 Asenna daisyUI 

### 2.1 Asennus

```bash
npm i -D daisyui@latest
```

**Tavoite:** Asennetaan daisyUI kehitysriippuvuudeksi (Tailwind‑plugin). 

### 2.2 Ota plugin käyttöön CSS:ssä

Avaa **`src/index.css`** ja lisää daisyUI‑plugin Tailwindin jälkeen:

```css
@import "tailwindcss";
@plugin "daisyui";
```

**Tavoite:** Tailwind v4:ssa pluginit lisätään `@plugin`‑direktiivillä CSS:ään, ei configiin. 

### 2.3 Testaa että toimii

Käytä jotakin daisyUI‑komponenttiluokkaa (esim. `btn`, `card`) ja varmista että tyyli näkyy. (Voit aloittaa esim. nappien ja korttien päivityksestä.)

---

## 3) Asenna shadcn/ui

### 3.1 shadcn/ui CLI (Vite‑projekti)

Aja projektin juuressa (client‑kansiossa):

```bash
npx shadcn-ui@latest init
```

Valitse **Vite** / **React** ja seuraa ohjeita. (CLI osaa alustaa Tailwind v4 ‑ystävällisesti ja luo pohjan komponenttien generointiin.) 

> Jos CLI pyytää lisäämään tyylit/tokensit, hyväksy oletukset. shadcn/ui:n 

### 3.2 Lisää muutama peruskomponentti

Lisää komponentit (esim. **Button**, **Card**, **Badge**, **Checkbox**, **Select**):

```bash
npx shadcn-ui@latest add button card badge checkbox select
```

**Tavoite:** shadcn/ui generoi lähdekoodikomponentit projektiisi, joita voit muokata vapaasti. 

---

## 4) UI‑päivityksen tavoitteet (Todo‑sovellus)

### 4.1 Yleinen rakenne

- Korvaa nykyiset perus‑Tailwind‑elementit **daisyUI**‑komponenteilla **tai** **shadcn/ui**‑komponenteilla seuraavasti:
    
    - **Sivun runko ja kortit**: daisyUI `card`, `navbar`, `footer`
        
    - **Toiminnot**: shadcn/ui `Button`
        
    - **Tilan ilmaisu**: shadcn/ui `Badge` tai daisyUI `badge`
        
    - **Syötteet**: shadcn/ui `Checkbox`, `Select`, `Input` (tai daisyUI:n `select`, `input`)
        

**Miksi kaksi kirjastoa?**

- daisyUI tarjoaa nopeasti käyttövalmiita luokkakomponentteja ja teemoja.
    
- shadcn/ui tarjoaa saavutettavia, muokattavia, koodina generoituja komponentteja, jotka istuvat suoraan koodipohjaan. 
    

>[!NOTE]
> Tyylikirjastoja harvemmin sekoitetaan keskenään. Nyt molemmat tyylikirjastot on asennettuna saman projektin sisään harjoituksena, jotta vältytään tarpeelta toteuttaa 2 erillistä projektia.

### 4.2 Header ja teemat (daisyUI)

- Lisää yläreunaan **navbar** daisyUI:llä (otsikko, linkki `/docs`).
    
- Aktivoi daisyUI‑teemat (oletus riittää, mutta voit lisätä teemanvaihtajan myöhemmin).
    

### 4.3 Listan kortit

- Kääri Todo‑lista **daisyUI `card`** ‑komponentteihin (otsikko, sisältö, actions‑alue).
    
- Näytä **kategoria** `badge`‑komponentilla.
    

### 4.4 Lomakkeet

- Korvaa luontilomakkeen elementit shadcn/ui‑komponenteilla:
    
    - Title: `Input`
        
    - Category: `Select` (+ `SelectItem` arvot)
        
    - Lähetys: `Button`
        
- Disabloi `Button` kun tallennus on käynnissä.
    

### 4.5 Status (done)

- Korvaa tavallinen checkbox shadcn/ui **Checkbox**‑komponentilla.
    
- Tee **optimistinen** päivitys kuten aiemmin, mutta käytä uutta komponenttia.
    

### 4.6 Ilmoitukset ja virheet

- Käytä daisyUI:n `alert` tai shadcn/ui:n `Badge`/`Card` yhdistelmää virheilmoituksiin.
    

---

## 5) Koodimuutokset (esimerkinomaiset)

> Kirjoita varsinainen toteutus itse; alla on vain rungot, joiden avulla näet missä kohtaa kirjastot kytketään.

### 5.1 DaisyUI luokkia JSX:ään

```tsx
// Esim. kortti
<div className="card bg-base-100 shadow">
  <div className="card-body">
    <h2 className="card-title">Tehtävät</h2>
    {/* sisältö */}
  </div>
</div>
```

### 5.2 shadcn/ui komponenttien käyttö

```tsx
// esim. Button ja Checkbox
import { Button } from "@/components/ui/button";
import { Checkbox } from "@/components/ui/checkbox";

<Button onClick={onAdd}>Lisää</Button>
<Checkbox checked={todo.done} onCheckedChange={(v) => onToggleDone(todo.id, Boolean(v))} />
```

### 5.3 Select kategorioille (shadcn/ui)

```tsx
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from "@/components/ui/select";

<Select value={todo.category} onValueChange={(val) => onChangeCategory(todo.id, val)}>
  <SelectTrigger className="w-40">
    <SelectValue placeholder="Kategoria" />
  </SelectTrigger>
  <SelectContent>
    <SelectItem value="GENERAL">GENERAL</SelectItem>
    <SelectItem value="WORK">WORK</SelectItem>
    <SelectItem value="STUDY">STUDY</SelectItem>
    <SelectItem value="HOME">HOME</SelectItem>
  </SelectContent>
</Select>
```

### 5.4 Badge kategoriolle

- Vaihtoehto A (daisyUI): `badge badge-outline`
    
- Vaihtoehto B (shadcn/ui): lisää oma `Badge`‑komponentti ja tyylit.
    

---

## 6) Testaa UI

1. Käynnistä backend (`npm run dev` server‑kansiossa)
    
2. Käynnistä frontend (`npm run dev` client‑kansiossa)
    
3. Varmista, että:
    
    - Luontilomake toimii (Button, Input, Select)
        
    - Status‑checkbox toimii (optimistinen päivitys)
        
    - Kategorian vaihto toimii
        
    - Tyylit ja komponentit renderöityvät (kortit, badge, navbar)
        

>[!NOTE]
> Voit halutessasi testailla tyylikirjastojen käyttöä enemmänkin, esim. toteuttamalla samat UI komponentit kahteen kertaan sekä daisyUI että shadcn/ui:n avulla.

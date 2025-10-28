
# Oppimistehtävä 4: daisyUI + Kasvien ja laitteiden luonti ja poisto

Tässä harjoituksessa laajennetaan aiemmin toteutettua **React + TypeScript + Tailwind** -sovellusta lisäämällä  **daisyUI**-komponentit, sekä **luonti- ja poistotoiminnot** kasveille ja niiden laitteille.

Harjoitus jatkaa suoraan edellisistä tehtävistä, joissa toteutettiin backend- ja frontend-yhteys REST-rajapinnan kautta.

---

## 🟩 Tehtävä 1 — Asenna ja ota käyttöön daisyUI

### Tavoite

Asenna **daisyUI** (Tailwind CSS -komponenttikirjasto) projektiin, jotta käyttöliittymään saadaan valmiiksi tyyliteltyjä elementtejä.

### Hyväksymiskriteerit

- daisyUI:n tyylit toimivat käyttöliittymässä.
    
- Tailwind toimii normaalisti ilman virheitä.
    

---

## 🟦 Tehtävä 2 — **Muuta olemassa oleva Tailwind-tyylitys käyttämään daisyUI-komponentteja (frontend)**

### Tavoite

Refaktoroi nykyinen UI käyttämään **daisyUI-komponenttiluokkia** (btn, card, badge, alert, modal, input, navbar, jne.) pelkkien Tailwind-apuluokkien sijaan. Tavoitteena on yhtenäinen, teemoitettava ja helppo ylläpitää -tyylitys.


### Mitä tiedostoja muokataan

- `src/components/PlantCard.tsx`
    
- `src/pages/PlantsPage.tsx`
    
- `src/pages/PlantDetailPage.tsx`
    
- (tarvittaessa) `src/App.tsx`
    
- `src/index.css` (vain jos haluat siivota omaa, ei-tarvittavaa custom-CSS:ää)
    

### Vaatimukset

1. **Painikkeet (buttons)**
    
    - Korvaa pitkät Tailwind-luokitusketjut (esim. `inline-flex items-center ... focus:ring-*`) daisyUI-luokilla:
        
        - Peruspainike: `btn`
            
        - Painotukset: `btn-primary`, `btn-secondary`, `btn-ghost`, `btn-outline`, `btn-error`
            
        - Koot: `btn-sm`, `btn-md`, `btn-lg`
            
        - Lataus: `btn loading`
            
    - **Sovella:**
        
        - “Start/Stop watering” → `btn btn-primary` (tai `btn btn-outline`)
            
        - “Open” / “View” → `btn btn-ghost` tai `btn`
            
        - “Delete” → `btn btn-error`
            
2. **Kortit (cards)**
    
    - Muuta listan ja detaljisivun laatikot käyttämään daisyUI-kortteja:
        
        - Säiliö: `card`
            
        - Sisältö: `card-body`
            
        - Otsikko: `card-title`
            
        - Toiminnot: `card-actions`
            
    - **Sovella:**
        
        - `PlantCard.tsx` → ympäröi sisältö `card`-rakenteella ja siirrä nappirivi `card-actions`-alueelle.
            
        - `PlantsPage.tsx` → grid säilyy, kortit vaihtuvat `card`-pohjaan.
            
3. **Statukset ja erottimet (badges & chips)**
    
    - Korvaa manuaaliset värit (esim. `bg-emerald-50 text-emerald-600`) daisyUI-badgeilla:
        
        - `badge` + teemat: `badge-success`, `badge-warning`, `badge-error`, `badge-neutral`
            
    - **Sovella:**
        
        - Laitteen status “ok|fault|offline” → `badge-success|badge-warning|badge-error`
            
4. **Ilmoitukset (alerts)**
    
    - Virhe- ja info-tilat: `alert alert-error`, `alert alert-success`, `alert alert-info`, `alert alert-warning`
        
    - **Sovella:**
        
        - `PlantsPage.tsx` / `PlantDetailPage.tsx` lataus-/virheviestit → korvaa omat `text-slate-*`-alertit daisyUI `alert` -komponentilla.
            
5. **Teemat ja värit**
    
    - Hyödynnä daisyUI:n teemoja (esim. `light`, `dark`, `cupcake`).
        
    - **Kielletty tässä tehtävässä:** kovakoodatut värit (`bg-slate-*`, `text-slate-*`, `focus:ring-*`) **interaktiivisissa** UI-elementeissä → korvaa daisyUI-luokilla.
        

### Hyväksymiskriteerit (Tehtävä 2)

-  Yleiset painikkeet, kortit, badge-tunnisteet, modaalit, alertit ja syötekentät on muutettu käyttämään daisyUI-komponenttiluokkia.
    
-  Interaktiivisissa elementeissä ei käytetä kovakoodattuja Tailwind-värejä tai omia focus-ring -ketjuja (daisyUI huolehtii tyylistä).
    
-  `PlantCard.tsx`, `PlantsPage.tsx` ja `PlantDetailPage.tsx` näyttävät visuaalisesti yhtenäisiltä daisyUI-teeman kanssa.
    
-  Responsiivinen grid säilyy (`grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-4` tms.), mutta korttien sisäinen tyyli perustuu daisyUI:hin.
    


---

## 🟧 Tehtävä 3 — Kasvin luontilomake (frontend)

### Tavoite

Toteuta **kasvin luontilomake** daisyUI-komponenteilla niin, että se istuu Tehtävä 2:ssa käytettyihin tyyleihin.

### Vaatimukset

- Lisää käyttöliittymään elementti minkä avulla voidaan avata kasvien luontilomake.
    
- Käytä lomakkeen rakenteena `card`-komponenttia.
    
- Kentät:
    
    - `name` (pakollinen)
        
    - `species` (pakollinen)
        
    - `location` (valinnainen)
        
    - `notes` (valinnainen)
        
    - Valinnainen osio laitteen oletusarvoille.
        
- Napit:
    
    - `btn btn-primary` – Lähetä
        
    - `btn btn-ghost` – Peruuta / tyhjennä
        
- Lähetyksessä:
    
    - Käytä olemassa olevaa `addPlant` metodia API kutsun teossa backendiin.
        
    - Päivitä kasvilistaa onnistuneen lisäyksen jälkeen, jotta siinä näytetään juuri lisätty kasvi.
	    
	    - Uuden kasvin päivittyminen listaukseen ei tule vaatia sivun erillistä päivittämistä

### Hyväksymiskriteerit

- Pakolliset kentät validoidaan.
    
- Onnistunut luonti lisää kasvin listaan ilman sivun uudelleenlatausta.
    
- Virheet ja verkko-ongelmat näytetään käyttäjälle
    

---

## 🟨 Tehtävä 4 — Kasvin poisto (backend)

### Tavoite

Toteuta backend-reitti, joka poistaa kasvin sekä siihen liittyvän laitteen.

### Vaatimukset

1. **Reitti**
    
    - `DELETE /api/plants/:id`
        
    - Parametri: `id` (numero)
        
2. **Toiminta**
    
    - Poistaa kasvin ja siihen liitetyn laitteen.
        
    - Palauttaa `204 No Content` onnistuneesta poistosta.
        
3. **Virheenkäsittely**
    
    - `400 Bad Request` jos id ei ole kelvollinen.
        
    - `404 Not Found` jos kasvia ei löydy.
        
4. **Kerrokset**
    
    - Repository: `repoDeletePlant(id: number): Promise<boolean>`
        
    - Service: `deletePlant(id: number): Promise<boolean>`
        
    - Controller: validoi id:n ja palauttaa statuksen (404 / 204).
        
5. Päivitä olemassa olevaa prisman schemaa lisäämällä  **onDelete: Cascade** -käyttäytymiselle laitteiden poistossa.
	
	- Voit tarvittaessa katsoa esimerkkiä  [Referential actions - Prisma Docs](https://www.prisma.io/docs/orm/prisma-schema/data-model/relations/referential-actions)
	
6. Aja lopuksi uusi migraatio **onDelete: Cascade**:n lisäykselle

### Hyväksymiskriteerit

- Reitti poistaa kasvin ja laitteen tietokannasta.
    
- Palauttaa oikeat statuskoodit.
    
- Lisätty Swagger/OpenAPI-dokumentaatioon.
    

---

## 🟦 Tehtävä 5 — Kasvilista (frontend) korteilla ja poistolla

### Tavoite

Näytä kasvit **daisyUI-korteilla** ja mahdollista **poisto** vahvistusmodaalin kautta. Varmista, että tyyli ja käytös noudattavat Tehtävä 2:n linjaa.

### Vaatimukset

1. **Käyttöliittymä**
    
    - Komponentti: `PlantsPage`
        
    - Asettelu:  
        `grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4`
        
    - Jokainen `card` näyttää:
        
        - Kasvin nimen, lajin ja sijainnin (jos on)
            
        - Laitteen tiedot: kosteus, akku, tila (badge)
            
        - Toiminnot:
            
            - **Näytä** → vie yksityiskohtasivulle
                
            - **Kastele / Lopeta** → kutsuu `API.togglePlantWatering(id)`
                
            - **Poista** → avaa vahvistusikkunan
                
2. **Vahvistusikkuna**
    
    - daisyUI `modal`, otsikko “Delete plant?”
        
    - Kuvaus: “Action will also delete the device connected to the plant.”
        
    - Nappulat:
        
        - `btn btn-error` – Delete plant
            
        - `btn` – Cancel
            
    - Vahvistettaessa:
        
        - Kutsu `API.deletePlant(id)`
            
        - Poista kasvi tilasta hakemalla lista uudelleen
            
3. **Käyttökokemus**
    
    - Latausanimaatio (`btn loading`) toiminnoille
        
    - Poiston aikana painikkeet pois käytöstä
        
    - Virheet näkyvät `alert alert-error` -komponenttina
        

### Hyväksymiskriteerit

- Kortit käyttävät daisyUI-tyylejä.
    
- Poistaminen avaa vahvistusikkunan ja poistaa kasvin listalta.
    
- Painikkeet estävät kaksoisklikkaukset.
    

---

## 🟪 Tehtävä 6 — Toastit, lataus- ja virhetilat (UX-viimeistely)

### Tavoite

Paranna sovelluksen käyttökokemusta lisäämällä **toast-ilmoitukset**, **latausindikaattorit** ja **selkeä virheenkäsittely**.

### Vaatimukset

1. **Toastit**
    
    - Lisää globaali ilmoitusjärjestelmä (esim. daisyUI:n `toast` tai oma hook).
        
    - Näytä onnistumisviesti:
        
        - Kasvin luonti
            
        - Kasvin poisto
            
        - Laitteen lisäys tai poisto
            
    - Näytä virheviesti API-virheistä.
        
2. **Latausindikaattorit**
    
    - Käytä `btn loading` -tilaa pitkissä toiminnoissa.
        
    - Lisää tarvittaessa yleinen “Loading…”-spinneri sivuille.
        
    - Estä painikkeiden painaminen toiminnon aikana.
        
3. **Virheenkäsittely**
    
    - Näytä `alert alert-error` tai toast virhetilanteissa.
        
    - Käsittele myös odottamattomat backend-virheet (500).
        
4. **Yhtenäisyys**
    
    - Käytä samaa ilmoitustyyliä koko sovelluksessa.
        
    - Modaalit sulkeutuvat automaattisesti onnistuneen toiminnon jälkeen.
        
    - Lomakkeet tyhjennetään onnistumisen jälkeen.
        

### Hyväksymiskriteerit

-  Kaikki toiminnot (luonti, poisto, päivitys) näyttävät selkeän etenemisen ja palautteen.
    
-  Painikkeet ja lomakkeet estävät toiminnon aikana uudelleenklikkauksen.
    
-  Virheet ja onnistumiset näkyvät selkeästi toastien tai alertien kautta.
    
-  Modaalit sulkeutuvat automaattisesti onnistuneen toiminnon jälkeen.
    
-  Ei “jumiutuneita” lataustiloja tai hiljaisia/näkymättömiä virheitä.
    
- Varmista, että **kaikki uudet** (Tehtävät 3 & 5) näkymät käyttävät samaa toast/alert -tapaa.

---

## ✅ Lopulliset hyväksymiskriteerit

-  daisyUI toimii ja on käytössä.
    
-  Backend tukee `DELETE /api/plants/:id`.
    
-  Frontend pystyy luomaan ja poistamaan kasveja API:n kautta.
    
-  Kortit näyttävät kasvien ja laitteiden tiedot oikein.
    
-  Vahvistusikkunat ja toastit toimivat.
    
-  Lataus- ja virhetilat näkyvät oikein.
    
-  Kaikki API-kutsut palauttavat oikeat HTTP-statuskoodit.
    

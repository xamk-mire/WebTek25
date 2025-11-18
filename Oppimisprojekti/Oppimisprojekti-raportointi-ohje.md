
# 📄 **Raportointiohje – SimpleChef-projektin loppuraportti**

Tämä ohjeistus kertoo, mitä sinun tulee sisällyttää SimpleChef-projektin loppuraporttiin.  
Raportin tarkoituksena on osoittaa, että olet:

- ymmärtänyt toteuttamasi kokonaisuuden
    
- pystyt kuvailemaan sen rakennetta
    
- ja ennen kaikkea: pystyt **demonstroimaan järjestelmän toimivuuden kuvakaappauksin ja tekstiselityksin**
    

Raportti palautetaan projektin mukana PDF-, Word- tai Markdown-muodossa.

---

# ✔️ **1. Sovelluksen arkkitehtuuri**

Kuvaa lyhyesti projektin tekninen rakenne.

Vastaa esimerkiksi näihin kysymyksiin:

- Miten frontend ja backend keskustelevat?
    
- Mitä varten keskitetty API-kerros on olemassa?
    
- Miten Prisma toimii tietokannan kanssa?
    
- Miten token-pohjainen kirjautuminen toimii?
    

Voit myös halutessasi lisätä yksinkertaisen arkkitehtuurikaavion (ei pakollinen).

**Laajuus:** 3–6 kappaletta.

---

# ✔️ **2. Tietomallit**

Kuvaa, mitä tietomalleja teit Prismassa:

- User
    
- Recipe
    
- Favorite
    

Lisää mukaan **kuvakaappaus Prisma Studiosta**, jossa näkyy vähintään:

- tietokantataulut
    
- muutama esimerkkirivi
    

**Lisää tekstiselitys**, jossa kerrot:

- mitä kukin malli tallentaa
    
- mitä suhteita mallien välillä on
    

---

# ✔️ **3. Backendin toimivuuden esittely (kuvakaappaukset + selitykset)**

Tässä osiossa sinun tulee selkeästi demonstroida backendin toiminta.

### 📸 **Pakolliset kuvakaappaukset:**

1. **Swagger-dokumentaatio näkyvillä selaimessa**  
    – Lisää selitys, mitä reittejä sivu sisältää.
    
2. **Reseptien haku (GET /api/recipes)**  
    – Kuvakaappaus Swaggerista tai Postmanista.  
    – Selitä, mitä vastauksessa näkyy.
    
3. **Reseptin luonti POST-pyynnöllä**  
    – Näytä lähetetty body ja palvelimen vastaus.  
    – Selitä onnistunut luonti.
    
4. **Käyttäjän rekisteröinti ja kirjautuminen**  
    – Rekisteröintipyyntö  
    – Login-pyyntö  
    – JWT-token näkyvissä vastauksessa  
    – Selitä, mihin tokenia käytetään.
    
5. **Suosikin lisääminen**  
    – Kuvakaappaus POST /recipes/:id/favorite  
    – Selitä, että toiminto vaatii tokenin.
    
6. **Suosikkien haku**  
    – Kuvakaappaus GET /users/me/favorites  
    – Selitä, että suosikit liitetään kirjautuneeseen käyttäjään.
    

**Jokaisen kuvakaappauksen yhteydessä tulee olla 2–4 lauseen tekstiselitys**, jossa kerrot:

- mitä kuvassa tapahtuu
    
- miksi se on tärkeää
    
- miten se liittyy sovelluksen toimintaan
    

---

# ✔️ **4. Frontendin toimivuuden esittely (kuvakaappaukset + selitykset)**

Esittele SimpleChef-sovelluksen käyttöliittymä.

### 📸 **Pakolliset kuvakaappaukset:**

1. **Etusivu (reseptilista)**  
    – Näytä reseptit kortteina.  
    – Selitä, miten tiedot haetaan API:n kautta.
    
2. **Reseptin luontilomake**  
    – Näytä lomakkeen kentät.  
    – Selitä validoinnit tai keskeiset toiminnot.
    
3. **Reseptin yksityiskohtasivu**  
    – Näytä toimintopainikkeet (Muokkaa, Poista, Suosikki).  
    – Selitä, mitä API-kutsuja sivu käyttää.
    
4. **Kirjautumisnäkymä**  
    – Selitä, miten onnistuneen kirjautumisen jälkeen token tallennetaan ja miten se vaikuttaa UI:hin.
    
5. **Suosikkisivu**  
    – Näytä käyttäjän suosikit.  
    – Selitä, miten suosikkitiedot haetaan käyttäjäkohtaisesti.
    
6. **Kirjautunut vs. ei-kirjautunut käyttäjä**  
    – Kuvakaappaus navbaarista ennen/ jälkeen kirjautumisen.  
    – Selitä roolin vaikutus mahdollisiin toimintoihin.
    

### 📌 **Vaatimus:**

Jokaisen kuvakaappauksen yhteydessä tulee olla selittävä teksti:

- Mitä toiminnallisuutta kuva näyttää?
    
- Mitä toimintoja UI kutsuu backendistä?
    
- Miten JWT-token vaikuttaa toimintaan?
    

---

# ✔️ **5. Toteutetut vaatimukset (omaa pohdintaa)**

Listaa lyhyesti:

- mitkä kurssin vaatimat ominaisuudet toteutit
    
- mikä oli haastavinta
    
- mikä onnistui parhaiten
    
- mitä tekisit toisin seuraavalla kerralla
    

Tämä osio on tärkeä arviointia varten.

---

# ✔️ **6. Yhteenveto**

Loppuun 3–5 lauseen yhteenveto, jossa kerrot:

- mitä opit projektista
    
- mikä oli projektin lopputulos
    
- mitä haluaisit kehittää jatkossa
    

# Specifikacija Projektnog Zadatka: Distributed Collaborative Code Editor

## 1. Opis Problema

Razvoj softvera u modernom okruženju često zahteva rad više programera na istom kodu istovremeno. Postojeća rešenja (npr. Google Docs) koriste centralizovane mehanizme zaključavanja ili operacione transformacije (OT) koje su kompleksne i teško skalabilne u distribuiranim sistemima.

Cilj ovog projekta je implementacija platforme za **kolaborativno programiranje u realnom vremenu** koja omogućava korisnicima da istovremeno uređuju kod bez konflikata. Sistem će se oslanjati na **CRDT (Conflict-free Replicated Data Types)** strukture podataka kako bi se osigurala eventualna konzistentnost podataka bez potrebe za centralnim zaključavanjem.

## 2. Arhitektura Sistema (Mikroservisi)

U skladu sa zahtevima predmeta, sistem je projektovan kao mikroservisna aplikacija implementirana u potpunosti u programskom jeziku **Rust**. Sistem se sastoji od 4 nezavisna servisa koji komuniciraju asinhrono.

### Spisak servisa:

#### A. Session Gateway Service

* **Uloga:** Upravljanje WebSocket konekcijama sa klijentima (Frontend).
* **Funkcionalnost:** Održava perzistentne sesije, rutira poruke između korisnika u istoj sobi i backend sistema. Obezbeđuje real-time prenos operacija kucanja.
* **Tehnologije:** **Rust**, **Axum** (web framework), **Tokio** (async runtime), `tokio-tungstenite`.

#### B. CRDT Synchronization Service - *Core servis*

* **Uloga:** "Mozak" sistema koji rešava konflikte pri uređivanju teksta.
* **Funkcionalnost:** Implementira **RGA (Replicated Growable Array)** algoritam za sinhronizaciju teksta. Prima operacije (insert/delete), rešava konflikte na osnovu logičkih satova i osigurava konvergenciju sadržaja kod svih klijenata.
* **Baza:** **Redis** (za brzo čuvanje privremenog stanja dokumenta i RGA stabla u memoriji).

#### C. Auth & User Service

* **Uloga:** Upravljanje korisnicima i bezbednost.
* **Funkcionalnost:** Registracija, login (JWT autentifikacija), upravljanje pravima pristupa projektima (Read/Write). Ispunjava zahtev za autorizaciju i autentifikaciju.
* **Baza:** **PostgreSQL** (Relaciona baza za trajno čuvanje podataka o korisnicima i projektima).

#### D. Code Execution Service

* **Uloga:** Izvršavanje i validacija koda koji korisnici napišu.
* **Funkcionalnost:** Izolovano pokretanje koda (Sandbox) i vraćanje rezultata (stdout/stderr) nazad u editor. Umesto eksternih skripti, servis koristi Rust biblioteke za interakciju sa Docker API-jem radi bezbednog izvršavanja.
* **Tehnologije:** **Rust**, Docker SDK (za izolaciju procesa).

## 3. Komunikacija i Obrasci

* **Asinhrona komunikacija:** Servisi međusobno komuniciraju putem Message Brokera **RabbitMQ** (koristeći Rust `lapin` biblioteku) radi razdvajanja odgovornosti i skalabilnosti.
* **API Gateway:** Ulazna tačka za sve REST zahteve i usmeravanje saobraćaja ka odgovarajućim mikroservisima.

## 4. Dodatne Funkcionalnosti (dodaci ako nije dovoljna kompleksnost projekta)

* **Kontejnerizacija:** Ceo sistem (svi servisi, baze i message broker) se podižu upotrebom **Docker Compose** alata.
* **Analitika:** Generisanje izveštaja o aktivnosti (broj linija koda, frekvencija izmena po korisniku) i vizualizacija.
* **Generisanje PDF-a:** Mogućnost eksporta napisanog koda sa sintaksnim farbanjem (syntax highlighting) u PDF format.

---

# Plan za Diplomski Rad (Proširenje)

Nakon uspešne odbrane projektnog zadatka, planirano je proširenje sistema u diplomski rad pod naslovom:
**"Implementacija distribuiranog sistema za kolaboraciju u realnom vremenu zasnovanog na CRDT strukturama"**

### Planirana proširenja (Roadmap):

1. **Napredna CRDT Implementacija i Optimizacija:**
* Prelazak sa osnovnog modela na optimizovanije strukture (npr. *YATA* ili *RGA Split*) radi smanjenja memorijskog otiska ("Tombstone garbage collection").
* *Doprinos:* Analiza performansi RGA algoritma u Rust-u pod velikim opterećenjem.


2. **Offline-first podrška:**
* Omogućavanje rada bez interneta, pri čemu se lokalne promene čuvaju i automatski sinhronizuju (merge) sa serverom kada se konekcija ponovo uspostavi, iskorišćavajući komutativna svojstva CRDT-a.


3. **Event Sourcing i "Time Travel":**
* Implementacija sistema za verzionisanje gde se svaka promena čuva kao događaj, omogućavajući korisnicima povratak na bilo koje prethodno stanje dokumenta.

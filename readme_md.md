# Utjecaj tipa i dimenzionalnosti vektorske reprezentacije na konvergenciju agenta pojačanog učenja za preporuku vijesti

Ovaj projekt istražuje i uspoređuje kako izbor vektorske reprezentacije teksta i njezina dimenzionalnost utječu na brzinu konvergencije agenta dubokog potkrepljivanog učenja (**DQN** – *Deep Q-Network*) te na ukupnu kvalitetu preporuka vijesti na skupu podataka **MIND-small**.

---

## 🎯 Cilj istraživanja

* Ispitati utjecaj **četiri metode vektorskog predstavljanja teksta** (temeljenih na učestalosti riječi i na kontekstu) na proces učenja DQN agenta.
* Analizirati utjecaj **dimenzionalnosti vektora** ($100$, $200$ i $300$ dimenzija) na brzinu konvergencije, stabilnost učenja i evaluacijski povrat.
* Usporediti uspješnost preporuka na poznatim korisnicima, novim (*cold-start*) korisnicima te u odnosu na osnovni pristup temeljen na kosinusnoj sličnosti.

---

## 📊 Skup podataka (MIND-small)

Eksperimenti su provedeni na javno dostupnom skupu podataka **MIND-small** (*Microsoft News Dataset*):
* **Ukupno članaka:** $51\,282$ članka raspoređena u $17$ glavnih kategorija (prevladavaju *news* i *sports* s $\approx 59\%$ članaka; $2\,666$ članaka nema sažetak).
* **Ukupno korisnika i interakcija:** $50\,000$ korisnika i $156\,965$ impresija (od čega $892$ ili $1,78\%$ *cold-start* korisnika bez povijesti čitanja).
* **Filtriranje impresija:** Zadržane su impresije s najmanje $7$ kandidata, barem $1$ klikom i najmanje $4$ neklika.

### Vremenska podjela podataka (granica: 14. studenoga 2019.)

| Skup podataka | Broj korisnika | Broj impresija | Opis |
| :--- | :---: | :---: | :--- |
| **Trening** | $10\,000$ | $26\,029$ | Interakcije prije 14. 11. 2019. |
| **Validacija** | $3\,000$ | $3\,000$ | Odabir modela i praćenje konvergencije |
| **Evaluacija (poznati korisnici)** | $4\,052$ | $6\,543$ | Korisnici prisutni u trening skupu (nakon 14. 11.) |
| **Evaluacija (novi korisnici)** | $3\,735$ | $5\,518$ | Korisnici neviđeni tijekom treninga (nakon 14. 11.) |

---

## 🔤 Vektorske reprezentacije teksta

Korpus je formiran spajanjem naslova i sažetka članka uz pretvorbu u mala slova. Za metode temeljene na rječniku uklonjene su stop riječi i primijenjen je prag pojavljivanja $\text{min\_df} = 5$ (rječnik od $18\,646$ riječi). Svi nenulti vektori su **L2 normalizirani**.

1. **BoW (Bag of Words):** Rijetki prikaz učestalosti riječi ($18\,646 \rightarrow 100 / 200 / 300$ dimenzija).
2. **LSA (Latent Semantic Analysis):** TF-IDF matrica (zaglađeni IDF) reducirana metodom SVD na $100$, $200$ i $300$ dimenzija.
3. **Word2Vec:** Prethodno trenirani *Google News* vektori ($300\text{D}$), prosjek vektora riječi u članku; dimenzije $100$ i $200$ dobivene su SVD redukcijom.
4. **FastText:** Prethodno trenirani vektori ($300\text{D}$) temeljeni na n-gramima znakova, prosjek vektora riječi u članku; dimenzije $100$ i $200$ dobivene su SVD redukcijom.

---

## 🤖 Arhitektura i okruženje DQN agenta

* **Epizoda:** Jedna impresija predstavlja jednu epizodu u kojoj agent sekvencijalno odabire **$5$ različitih članaka**.
* **Nagrada ($r$):** $1$ za zabilježeni klik, $0$ za neklik.
* **Korisnički profil:** Eksponencijalno ponderirani prosjek kliknutih članaka iz povijesti ($\text{profil}_{\text{novi}} = 0,95 \times \text{profil}_{\text{prethodni}} + 0,05 \times \text{vektor}_{\text{članka}}$). Za korisnike bez upotrebljive povijesti koristi se prosječni trening profil.
* **Stanje i Q-mreža:** Ulazni vektor dimenzije $2D + 1$ (profil korisnika dimenzije $D$, vektor članka kandidata dimenzije $D$ i udio preostalih odabira) prolazi kroz dva skrivena sloja ($256 \rightarrow 128$, aktivacija *ReLU*) do procjene $Q$-vrijednosti.
* **Treniranje:** Svaka od $12$ kombinacija (4 reprezentacije $\times$ 3 dimenzionalnosti) trenirana je kroz **$5$ ponavljanja** (različita početna sjemena — ukupno $60$ treninga) uz $\varepsilon$-pohlepnu strategiju ($\varepsilon: 1,0 \rightarrow 0,05$), spremnik iskustava, ciljnu mrežu i rano zaustavljanje nakon $10$ krugova bez poboljšanja.

---

## 📈 Ključni rezultati

### 1. Prosječni evaluacijski povrat $R$ po reprezentaciji (preko 100D, 200D i 300D)

| Reprezentacija | Poznati korisnici | Novi korisnici |
| :--- | :---: | :---: |
| **BoW** | $0,3668$ | $0,3510$ |
| **LSA** | $0,3747$ | $0,3621$ |
| **Word2Vec** | **$0,4116$** | **$0,4020$** |
| **FastText** | $0,3897$ | $0,3803$ |

### 2. Tijek učenja i brzina konvergencije (prosjek 5 ponavljanja)

| Kombinacija | $\text{AUC}_R$ (prvih 15 krugova) | Najbolji val. povrat $R$ | Krugovi do $95\%$ | Vrijeme do $95\%$ |
| :--- | :---: | :---: | :---: | :---: |
| **BoW 300D** | $0,4373$ | $0,4630$ | $4,0$ | $48,49\text{ s}$ |
| **LSA 300D** | $0,4446$ | $0,4705$ | $4,0$ | $48,84\text{ s}$ |
| **Word2Vec 100D** | $0,4633$ | $0,5043$ | $6,8$ | $78,37\text{ s}$ |
| **Word2Vec 200D** | **$0,4663$** | $0,5060$ | $7,2$ | $83,73\text{ s}$ |
| **Word2Vec 300D** | $0,4514$ | **$0,5069$** | $9,6$ | $112,03\text{ s}$ |
| **FastText 300D** | $0,4152$ | $0,4916$ | $17,2$ | $193,35\text{ s}$ |

### 3. Metrike rangiranja za Word2Vec (poznati korisnici, prosjek 5 ponavljanja)

| Metrika | Word2Vec 200D | Word2Vec 300D |
| :--- | :---: | :---: |
| **Hit@1** | **$12,55\%$** | $11,18\%$ |
| **Hit@2** | **$21,29\%$** | $20,56\%$ |
| **Hit@5** | $40,41\%$ | **$40,92\%$** |
| **nDCG@5** | **$0,2209$** | $0,2190$ |
| **MRR** | **$0,2204$** | $0,2135$ |
| **Povrat $R$** | $0,4117$ | **$0,4153$** |

---

## 💡 Zaključci

* **Nadmoć kontekstualnih prikaza:** Word2Vec i FastText dosljedno nadmašuju BoW i LSA i kod poznatih i kod novih korisnika, potvrđujući da semantičko grupiranje tekstova olakšava generalizaciju na neviđene članke (u testnom skupu je $\approx 63\%$ novih članaka).
* **Kompromis brzine i kvalitete učenja:** Prikazi temeljeni na učestalosti riječi (BoW, LSA) konvergiraju brže ($\approx 4$ kruga do $95\%$ svog maksimuma), ali dosežu nižu gornju granicu kvalitete preporuka od Word2Vec modela.
* **Uloga dimenzionalnosti:** Najviši prosječni povrat za Word2Vec i FastText postignut je pri **$300\text{D}$**, dok su za BoW i LSA najbolji prosjeci ostvareni pri **$200\text{D}$**, odnosno **$100\text{D}$** (uz preklapanja intervala između pojedinih sjemena).
* **Usporedba s kosinusnom sličnošću:** Pri $300\text{D}$ na poznatim korisnicima, izravno rangiranje kosinusnom sličnošću nad LSA prikazom ($R = 0,4203$) blago nadmašuje DQN s Word2Vec prikazom ($R = 0,4153 \pm 0,0048$), što pokazuje da konačna uspješnost uvelike ovisi i o samom mehanizmu rangiranja.

---

## 👤 Autor

**Mario Lucić**
# Carver — Proefrit Registratie-applicatie

Desktop-applicatie voor Carver Europe BV om proefritten te plannen,
beheren en evalueren. Geschreven in **C# / .NET 8 Windows Forms** met
**Microsoft SQL Server** als database.

Auteur: **Rik Postma** — Software Developer (MBO4), Firda
Docent: **Marco Hoekstra**

---

## Vereisten

- **Visual Studio 2022** (Community is voldoende) met de workload
  *.NET desktop development*.
- **.NET 8 SDK** (komt mee met VS 2022 17.8+).
- **Microsoft SQL Server 2022** met **SQL Server Management Studio 22 (SSMS)**.

---

## Installatie — stappenplan

### 1. Database opzetten

1. Open **SSMS 22** en maak verbinding met je lokale SQL Server-instance
   (bijvoorbeeld `localhost` of `.\SQLEXPRESS`, afhankelijk van je install).
2. Open het bestand `Database\CarverDB.sql` (`File → Open → File…`).
3. Voer het hele script uit (**F5**). Dit script:
   - maakt de database `CarverDB` aan;
   - maakt de tabellen `Gebruiker`, `Carver`, `Prospect`, `Proefrit`
     en `Gebruikerservaring`;
   - vult de 5 Carver-modellen (Base, R+, Range, S+, Sport) met prijzen;
   - maakt een admin-gebruiker met inlog `admin` / `admin123`.

### 2. Connection string aanpassen

Open `App.config` in de project-root en zorg dat de waarde van
`CarverDB` klopt met je server-naam:

```xml
<connectionStrings>
  <add name="CarverDB"
       connectionString="Server=localhost;Database=CarverDB;Trusted_Connection=True;Encrypt=False;TrustServerCertificate=True;"
       providerName="Microsoft.Data.SqlClient" />
</connectionStrings>
```

Aanpassingen die soms nodig zijn:

| Situatie | Vervang `Server=localhost` door |
|---|---|
| Standaard local install | `Server=localhost` of `Server=.` |
| Named instance, bv. Express | `Server=.\SQLEXPRESS` |
| Andere machine | `Server=<machine-naam>` |

### 3. Applicatie starten

1. Open `CarverApp.sln` in Visual Studio.
2. Build (**Ctrl+Shift+B**). NuGet-pakketten worden automatisch opgehaald
   (Microsoft.Data.SqlClient, QuestPDF, ClosedXML, ConfigurationManager).
3. Start (**F5**).
4. Log in met **gebruikersnaam `admin`** en **wachtwoord `admin123`**.
   *(Wijzig dit wachtwoord direct via Gebruikersbeheer.)*

---

## OOP-concepten in de code

| Concept | Waar te vinden |
|---|---|
| **Inheritance** | `Administrator : Gebruiker` |
| **Polymorphism** | `virtual RolNaam` in `Gebruiker`, override in `Administrator` |
| **Encapsulation** | Repository-klassen verbergen SQL achter methodes |
| **Abstraction** | `DatabaseHelper` abstraheert connectie-details |
| **Composition** | `Proefrit` bevat navigation-properties `Prospect` en `Carver` |

---

## Functionaliteit

- **Inloggen** met gehasht wachtwoord (SHA-256 + random salt).
- **Nieuwe proefrit** registreren (prospect-gegevens + Carver + datum).
- **Lijst Uit te voeren** met wijzig- en afrond-knoppen.
- **Lijst Uitgevoerd** met ervaring-knop.
- **4 vragen** beantwoorden (Niet mee eens / Neutraal / Mee eens).
- **Statistieken** visueel als staafdiagram (eigen GDI+ rendering).
- **PDF-export** van klantenkaart (QuestPDF).
- **Excel-export** van klantenlijst (ClosedXML).
- **Gebruikersbeheer** — alleen zichtbaar voor administrators.

---

## Projectstructuur

```
CarverApp/
├── CarverApp.sln
├── CarverApp.csproj
├── App.config                         — Connection string
├── Program.cs                         — Entry point
├── Database/CarverDB.sql              — DB-aanmaakscript
├── Models/                            — Domeinklassen (OOP)
│   ├── Gebruiker.cs
│   ├── Administrator.cs               — erft van Gebruiker
│   ├── Prospect.cs
│   ├── Carver.cs
│   ├── Proefrit.cs
│   └── Gebruikerservaring.cs
├── Data/DatabaseHelper.cs             — Verbinding
├── Repositories/                      — Data-access laag
│   ├── GebruikerRepository.cs
│   ├── CarverRepository.cs
│   ├── ProspectRepository.cs
│   ├── ProefritRepository.cs
│   └── ErvaringRepository.cs
├── Helpers/
│   ├── PasswordHelper.cs              — SHA-256 + salt
│   ├── PdfExporter.cs                 — QuestPDF
│   └── ExcelExporter.cs               — ClosedXML
└── Forms/                             — UI-laag
    ├── FrmLogin
    ├── FrmHoofd
    ├── FrmProefritNieuw
    ├── FrmProefritWijzigen
    ├── FrmProefrittenGepland
    ├── FrmProefrittenUitgevoerd
    ├── FrmErvaring
    ├── FrmStatistieken
    ├── FrmGebruikersbeheer
    └── FrmGebruikerEdit
```

---

## Troubleshooting

**"Cannot connect to CarverDB"** bij opstarten →
1. Controleer of SQL Server draait (`services.msc` → SQL Server).
2. Test in SSMS dat je kunt verbinden met dezelfde server-naam.
3. Check de `Server=…` waarde in `App.config`.

**"Login failed for user"** → je SQL-server staat op SQL-authenticatie.
Pas in `App.config` aan naar:
`Server=…;Database=CarverDB;User Id=sa;Password=jouwwachtwoord;TrustServerCertificate=True;`

**Wachtwoord vergeten van admin** → voer in SSMS uit:
```sql
USE CarverDB;
UPDATE Gebruiker
SET WachtwoordHash = 'gOOYXqsurc9zF8hUkHElDQ==:MZ6E+03PvFjmjMC03FLJSY4H54k6hPJdAbPRPzkp5m0='
WHERE Gebruikersnaam = 'admin';
-- wachtwoord wordt weer 'admin123'
```

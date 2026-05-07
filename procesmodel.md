# Generiek Procesmodel RVO – Zaakmanagement op nieuw platform

> Dit document beschrijft een generiek, regeling-onafhankelijk procesmodel voor zaakmanagement.
> Alle diagrammen zijn in Mermaid en gebruiken gestandaardiseerde termen.

---

## Inhoudsopgave

1. [Doel en afbakening](#1-doel-en-afbakening)
2. [Kernprincipes](#2-kernprincipes)
3. [Generieke end-to-end lifecycle](#3-generieke-end-to-end-lifecycle)
4. [Hiërarchie: Hoofdzaak · Subzaak · SubZaakTaak](#4-hiërarchie-hoofdzaak--subzaak--subzaaktaak)
5. [Statusmodel](#5-statusmodel)
6. [Fase-orkestratie (sequentie)](#6-fase-orkestratie-sequentie)
7. [Architectuur (componenten)](#7-architectuur-componenten)
8. [Generiek datamodel](#8-generiek-datamodel)
9. [Governance en ontwerpafspraken](#9-governance-en-ontwerpafspraken)
10. [Uitbreidbaarheid voor nieuwe regelingen](#10-uitbreidbaarheid-voor-nieuwe-regelingen)
11. [Lead-architect validatie](#11-lead-architect-validatie)
12. [Naamgevingsconventies](#12-naamgevingsconventies)

---

## 1. Doel en afbakening

Dit model is bedoeld als herbruikbaar fundament voor het opzetten van nieuwe uitvoeringsprocessen op een nieuw RVO-platform.

**Afbakening:**
- Geen regeling-specifieke processtappen.
- Geen juridisch-specifieke stromen of terminologie.
- Geen productspecifieke of domeinspecifieke uitzonderingen.

**Doel:**
- Eén uniforme structuur voor processturing, statusafleiding en historie.
- Snelle onboarding van nieuwe regelingen via configuratie in plaats van maatwerk.

---

## 2. Kernprincipes

1. **Configuratie boven codering**  
   Procesvarianten worden geconfigureerd met fase- en taakdefinities.

2. **Eenduidige hiërarchie**  
   Elke Hoofdzaak bestaat uit Subzaken; elke Subzaak bestaat uit SubZaakTaak-items.

3. **Centrale orkestratie**  
   Een Procesmanagement-service bepaalt startvoorwaarden, volgorde en statusafleiding.

4. **Volledige auditability**  
   Elke statuswijziging en taakuitkomst is historisch herleidbaar.

5. **Idempotente verwerking**  
   Herhaalde events veroorzaken geen dubbele of inconsistente mutaties.

6. **Scheiding van verantwoordelijkheden**  
   UI, processturing, gegevensopslag en documentopslag zijn losgekoppeld.

---

## 3. Generieke end-to-end lifecycle

Onderstaand proces toont een neutrale lifecycle zonder regeling- of juridisch-specifieke paden.

```mermaid
flowchart TD
    START([Verzoek ontvangen]) --> HZ_CREATE

    subgraph HZ["Hoofdzaak"]
        HZ_CREATE[Hoofdzaak registreren] --> HZ_INIT[Initiële context vastleggen]
        HZ_INIT --> SZ_INTAKE

        subgraph SZ1["Subzaak: Intake"]
            SZ_INTAKE[Input verzamelen] --> TAAK_INTAKE_1
            subgraph SZT1["SubZaakTaak Intake"]
                TAAK_INTAKE_1[Ontvangst registreren] --> TAAK_INTAKE_2[Invoer structureren]
            end
            TAAK_INTAKE_2 --> SZ1_DONE[/Subzaak Intake afgerond/]
        end

        SZ1_DONE --> SZ_VALIDATIE

        subgraph SZ2["Subzaak: Validatie"]
            SZ_VALIDATIE[Validatie starten] --> TAAK_VAL_1
            subgraph SZT2["SubZaakTaak Validatie"]
                TAAK_VAL_1[Gegevenscontrole] --> TAAK_VAL_2[Regelcontrole]
            end
            TAAK_VAL_2 --> SZ2_DONE[/Subzaak Validatie afgerond/]
        end

        SZ2_DONE --> BESLISSING{Voldoende?}
        BESLISSING -- Nee --> SZ_AANVULLING
        BESLISSING -- Ja --> SZ_UITVOERING

        subgraph SZ3["Subzaak: Aanvulling"]
            SZ_AANVULLING[Aanvulling opvragen] --> TAAK_AANV_1
            subgraph SZT3["SubZaakTaak Aanvulling"]
                TAAK_AANV_1[Aanvulling ontvangen] --> TAAK_AANV_2[Aanvulling verwerken]
            end
            TAAK_AANV_2 --> SZ3_DONE[/Subzaak Aanvulling afgerond/]
        end

        SZ3_DONE --> SZ_VALIDATIE

        subgraph SZ4["Subzaak: Uitvoering"]
            SZ_UITVOERING[Uitvoering starten] --> TAAK_UIT_1
            subgraph SZT4["SubZaakTaak Uitvoering"]
                TAAK_UIT_1[Beslismoment uitvoeren] --> TAAK_UIT_2[Resultaat voorbereiden]
            end
            TAAK_UIT_2 --> SZ4_DONE[/Subzaak Uitvoering afgerond/]
        end

        SZ4_DONE --> SZ_CONTROLE

        subgraph SZ5["Subzaak: Controle"]
            SZ_CONTROLE[Controle starten] --> TAAK_CON_1
            subgraph SZT5["SubZaakTaak Controle"]
                TAAK_CON_1[Kwaliteitscontrole] --> TAAK_CON_2[Output valideren]
            end
            TAAK_CON_2 --> SZ5_DONE[/Subzaak Controle afgerond/]
        end

        SZ5_DONE --> SZ_AFSLUITING

        subgraph SZ6["Subzaak: Afsluiting"]
            SZ_AFSLUITING[Afsluiten starten] --> TAAK_AFS_1
            subgraph SZT6["SubZaakTaak Afsluiting"]
                TAAK_AFS_1[Uitkomst publiceren] --> TAAK_AFS_2[Historie finaliseren]
            end
            TAAK_AFS_2 --> SZ6_DONE[/Subzaak Afsluiting afgerond/]
        end

        SZ6_DONE --> HZ_DONE[/Hoofdzaak afgerond/]
    end

    HZ_DONE --> END([Zaak gesloten])
```

---

## 4. Hiërarchie: Hoofdzaak · Subzaak · SubZaakTaak

```mermaid
graph TD
    HZ["Hoofdzaak\nZaaknummer + status + context"]

    HZ --> SZ_A["Subzaak: Intake"]
    HZ --> SZ_B["Subzaak: Validatie"]
    HZ --> SZ_C["Subzaak: Aanvulling"]
    HZ --> SZ_D["Subzaak: Uitvoering"]
    HZ --> SZ_E["Subzaak: Controle"]
    HZ --> SZ_F["Subzaak: Afsluiting"]

    SZ_A --> SZT_A1["SubZaakTaak: Ontvangst registreren"]
    SZ_A --> SZT_A2["SubZaakTaak: Invoer structureren"]

    SZ_B --> SZT_B1["SubZaakTaak: Gegevenscontrole"]
    SZ_B --> SZT_B2["SubZaakTaak: Regelcontrole"]

    SZ_D --> SZT_D1["SubZaakTaak: Beslismoment uitvoeren"]
    SZ_D --> SZT_D2["SubZaakTaak: Resultaat voorbereiden"]

    SZ_F --> SZT_F1["SubZaakTaak: Uitkomst publiceren"]
    SZ_F --> SZT_F2["SubZaakTaak: Historie finaliseren"]
```

### Legenda

| Niveau | Term | Omschrijving |
|--------|------|--------------|
| 1 | **Hoofdzaak** | Centrale casus met unieke identificatie en gecombineerde status |
| 2 | **Subzaak** | Afgebakende fase met eigen lifecycle |
| 3 | **SubZaakTaak** | Kleinste uitvoerbare taak binnen een Subzaak |

---

## 5. Statusmodel

Het statusmodel ondersteunt een externe, vereenvoudigde status en een interne, gedetailleerde status. Parallelle Subzaken zijn toegestaan; de Hoofdzaakstatus is een afgeleide aggregatie.

```mermaid
stateDiagram-v2
    [*] --> Nieuw : Hoofdzaak aangemaakt
    Nieuw --> InIntake : Intake gestart
    InIntake --> InValidatie : Intake afgerond

    InValidatie --> WachtOpAanvulling : Aanvulling nodig
    WachtOpAanvulling --> InValidatie : Aanvulling verwerkt

    InValidatie --> InUitvoering : Validatie akkoord
    InUitvoering --> InControle : Uitvoering afgerond
    InControle --> InAfsluiting : Controle akkoord
    InAfsluiting --> Afgesloten : Afsluiting afgerond

    InIntake --> Gepauzeerd : Tijdelijke blokkade
    InValidatie --> Gepauzeerd : Tijdelijke blokkade
    InUitvoering --> Gepauzeerd : Tijdelijke blokkade
    Gepauzeerd --> InIntake : Hervatten
    Gepauzeerd --> InValidatie : Hervatten
    Gepauzeerd --> InUitvoering : Hervatten

    Nieuw --> Geannuleerd : Geannuleerd
    InIntake --> Geannuleerd : Geannuleerd
    InValidatie --> Geannuleerd : Geannuleerd
    InUitvoering --> Geannuleerd : Geannuleerd
    InControle --> Geannuleerd : Geannuleerd
    InAfsluiting --> Geannuleerd : Geannuleerd

    Afgesloten --> [*]
    Geannuleerd --> [*]
```

---

## 6. Fase-orkestratie (sequentie)

```mermaid
sequenceDiagram
    actor Gebruiker
    participant UI as Zaakportaal
    participant PMS as Procesmanagement-service
    participant STORE as Zaakregister
    participant ENGINE as Procesengine
    participant DOC as Documentopslag
    participant EXT as Externe koppelingen

    Gebruiker->>UI: Verzoek indienen
    UI->>PMS: Hoofdzaak starten
    PMS->>STORE: Hoofdzaak opslaan
    STORE-->>PMS: Bevestiging
    PMS-->>UI: Zaakreferentie

    ENGINE->>PMS: Subzaak startverzoek
    PMS->>STORE: Context en historie ophalen
    STORE-->>PMS: Gegevensset
    PMS->>PMS: Startvoorwaarden evalueren

    alt Start toegestaan
        PMS->>STORE: Subzaak registreren (gestart)
        PMS-->>ENGINE: Start akkoord
        ENGINE->>STORE: SubZaakTaak updates registreren
        ENGINE->>DOC: Documentreferenties opslaan
        DOC-->>ENGINE: Referentie bevestigd
    else Start niet toegestaan
        PMS-->>ENGINE: Start geweigerd
        PMS->>STORE: Incident registreren
    end

    ENGINE->>PMS: Subzaak afgerond
    PMS->>STORE: Subzaakstatus bijwerken
    PMS->>PMS: Hoofdzaakstatus opnieuw afleiden
    PMS->>STORE: Geaggregeerde status opslaan
    PMS->>ENGINE: Volgende Subzaak(en) activeren
    PMS->>EXT: Statuspublicatie

    UI->>PMS: Zaakstatus opvragen
    PMS->>STORE: Historie ophalen
    STORE-->>PMS: Historie
    PMS-->>UI: Status + historie
```

---

## 7. Architectuur (componenten)

```mermaid
graph TB
    subgraph KANAAL["Kanalen"]
        PORTAAL["Zaakportaal"]
        WERKLIJST["Werklijst"]
        BEHEER["Beheerconsole"]
    end

    subgraph PLATFORM["Nieuw Platform"]
        subgraph PMS["Procesmanagement-service"]
            ORK["Orkestratie"]
            STA["Statusafleiding"]
            PLN["Planning"]
            HIS["Historiebeheer"]
        end

        ENGINE["Procesengine"]
        STORE["Zaakregister"]
        DOC["Documentopslag"]
        EVT["Eventbus"]
    end

    subgraph KOPPELINGEN["Koppelingen"]
        API["Externe API's"]
        MSG["Berichtuitwisseling"]
    end

    PORTAAL <--> ORK
    WERKLIJST <--> STA
    BEHEER <--> ORK

    ORK <--> STA
    ORK <--> PLN
    ORK <--> HIS

    ORK <--> STORE
    STA --> STORE
    HIS --> STORE

    ENGINE <--> ORK
    ENGINE --> DOC
    ENGINE --> EVT

    ORK --> API
    EVT --> MSG
```

---

## 8. Generiek datamodel

Dit model bevat alleen generieke kernentiteiten die in vrijwel elk zaakproces toepasbaar zijn.

```mermaid
erDiagram
    HOOFDZAAK {
        string ID PK
        string NUMMER
        string STATUS_EXTERN
        string STATUS_INTERN
        string TYPE
        datetime AANGEMAAKT_OP
        datetime GEWIJZIGD_OP
        string AANGEMAAKT_DOOR
        string GEWIJZIGD_DOOR
    }

    HOOFDZAAK_PARAMETER {
        string ID PK
        string HOOFDZAAK_ID FK
        string NAAM
        string WAARDE
        datetime AANGEMAAKT_OP
    }

    SUBZAAK {
        string ID PK
        string HOOFDZAAK_ID FK
        string CODE
        string STATUS
        int VOLGORDE
        datetime START_OP
        datetime EIND_OP
        datetime AANGEMAAKT_OP
        datetime GEWIJZIGD_OP
    }

    SUBZAAKTAAK {
        string ID PK
        string SUBZAAK_ID FK
        string CODE
        string STATUS
        string RESULTAAT
        string DOCUMENT_REF
        datetime START_OP
        datetime EIND_OP
        datetime AANGEMAAKT_OP
        datetime GEWIJZIGD_OP
    }

    STATUS_OVERGANG {
        string ID PK
        string ENTITEIT_TYPE
        string ENTITEIT_ID
        string VAN_STATUS
        string NAAR_STATUS
        datetime OVERGANG_OP
        string BRON
    }

    DOCUMENT {
        string ID PK
        string REFERENTIE
        string SUBZAAKTAAK_ID FK
        string TYPE
        datetime AANGEMAAKT_OP
    }

    HOOFDZAAK ||--o{ SUBZAAK : bevat
    HOOFDZAAK ||--o{ HOOFDZAAK_PARAMETER : heeft
    SUBZAAK ||--o{ SUBZAAKTAAK : bevat
    SUBZAAKTAAK ||--o{ DOCUMENT : verwijst
    HOOFDZAAK ||--o{ STATUS_OVERGANG : logt
    SUBZAAK ||--o{ STATUS_OVERGANG : logt
    SUBZAAKTAAK ||--o{ STATUS_OVERGANG : logt
```

### Datamodel-richtlijnen

- Alle entiteiten hebben technische ID, tijdstempels en herkomstvelden.
- Statussen worden centraal beheerd en historisch gelogd.
- Extensies worden toegevoegd via parameters of aanvullende tabellen, niet door kernentiteiten te vervuilen.

---

## 9. Governance en ontwerpafspraken

### 9.1 Procesgovernance
- Elke Subzaak heeft een duidelijke entry- en exit-conditie.
- Elke SubZaakTaak levert een expliciet resultaat op.
- Herstart en compensatie verlopen via gecontroleerde statusovergangen.

### 9.2 Technische governance
- API-contracten versioneren per major/minor.
- Event schema’s worden centraal gevalideerd.
- Idempotency keys zijn verplicht op muterende opdrachten.

### 9.3 Operationele governance
- Monitoring op doorlooptijd, foutpercentages en wachtrijen.
- Incidentregistratie op taak-, fase- en hoofdzakniveau.
- Beheerconsole ondersteunt pauzeren, hervatten en handmatige correctie.

### 9.4 Security en privacy
- Least privilege voor service-accounts.
- Gegevensminimalisatie in externe publicaties.
- Volledige audittrail voor alle statusmutaties.

---

## 10. Uitbreidbaarheid voor nieuwe regelingen

Nieuwe regelingen worden toegevoegd met een vaste implementatiestrategie:

1. Definieer een nieuw **zaaktype** met configuratie.
2. Selecteer en orden standaard **Subzaken**.
3. Koppel per Subzaak de benodigde **SubZaakTaak-items**.
4. Configureer statusmapping extern/intern.
5. Koppel optionele externe integraties via gestandaardiseerde connectoren.

**Resultaat:**
- Hoge herbruikbaarheid van platformcomponenten.
- Beperkt maatwerk en voorspelbare implementatietijd.
- Eenduidige operatie over verschillende regelingen heen.

---

## 11. Lead-architect validatie

Deze validatie beoordeelt of het model geschikt is als blauwdruk voor een nieuwe regeling op een nieuw platform.

### 11.1 Validatiecriteria

- **Generiek:** geen regeling- of juridische afhankelijkheden.
- **Schaalbaar:** meerdere zaaktypen en parallelle subzaken ondersteund.
- **Bestuurbaar:** beheer, monitoring en herstelpaden expliciet.
- **Controleerbaar:** auditbare statusovergangen en historie.
- **Implementeerbaar:** duidelijke componentgrenzen en datacontracten.

### 11.2 Beoordeling

| Criterium | Resultaat | Toelichting |
|----------|-----------|-------------|
| Generiek | ✅ | Regeling-specifieke en juridische termen verwijderd |
| Schaalbaar | ✅ | Parallelle subzaken en configureerbare stappen ondersteund |
| Bestuurbaar | ✅ | Pauzeren/hervatten, incidentregistratie en beheerconsole opgenomen |
| Controleerbaar | ✅ | Statusovergangen en historie als kernmodel gedefinieerd |
| Implementeerbaar | ✅ | Duidelijke scheiding tussen portaal, orkestratie, engine en storage |

### 11.3 Iteratiecheck

De validatie toont geen blokkerende hiaten voor generieke inzet. Een extra iteratie is niet noodzakelijk voor de basisarchitectuur.

**Conclusie:** model is optimaal als generieke startarchitectuur voor nieuwe regelingen op een nieuw RVO-platform.

---

## 12. Naamgevingsconventies

| Begrip | Definitie |
|--------|-----------|
| **Hoofdzaak** | Centrale casus met unieke identificatie en geaggregeerde status |
| **Subzaak** | Fase binnen de Hoofdzaak met eigen lifecycle |
| **SubZaakTaak** | Kleinste uitvoerbare taak binnen een Subzaak |
| **Procesmanagement-service** | Component voor orkestratie, statusafleiding en historie |
| **Procesengine** | Component die SubZaakTaak-items uitvoert conform configuratie |
| **Zaakregister** | Persistente opslag voor Hoofdzaak, Subzaak, SubZaakTaak en historie |
| **Documentopslag** | Opslag van documentreferenties gekoppeld aan SubZaakTaak-items |
| **Externe status** | Vereenvoudigde status voor externe consumptie |
| **Interne status** | Gedetailleerde status voor operationele sturing |

# Architektura Systemu w Modelu C4 (Poziomy 1 i 2)

Dokumentacja architektoniczna projektu **System Gry Zręcznościowej IoT (Micro:bit + Arduino + Online Leaderboard)** opracowana zgodnie ze standardem C4 model (Simon Brown).

Plik zawiera skonsolidowane i poprawione diagramy architektury dla **Poziomu 1 (Kontekst Systemu)** oraz **Poziomu 2 (Kontenery)** wraz z kompletnymi opisami technicznymi i legendą.

---

## Poziom 1: Kontekst Systemu (System Context)

Diagram kontekstu przedstawia systemy wchodzące w skład rozwiązania na najwyższym poziomie abstrakcji, ich granice, użytkowników (aktorów) oraz relacje z zewnętrznym środowiskiem transmisyjnym.

W ramach rozwiązania funkcjonują dwa systemy wchodzące w zakres implementacyjny projektu:
1. **Konsola Gry (Micro:bit A)** – autonomiczny system wbudowany (*embedded*).
2. **Platforma Rankingu Online** – system serwerowo-webowy.

Pomiędzy nimi pośredniczy zewnętrzny węzeł transmisyjny – **Czarna Skrzynka** (bramka Wi-Fi oparta na Micro:bit B i Arduino).

### Diagram kontekstu

```mermaid
flowchart TD
    classDef person fill:#08427b,stroke:#073b6f,color:#fff;
    classDef internal fill:#1168bd,stroke:#0b4884,color:#fff;
    classDef external fill:#6c757d,stroke:#495057,color:#fff;

    PLAYER["👤 Gracz<br/><b>[Osoba]</b><br/><br/>Gra na fizycznej konsoli oraz<br/>przegląda ranking wyników w przeglądarce"]:::person

    CONSOLE["Konsola Gry (Micro:bit A)<br/><b>[System]</b><br/><br/>Autonomiczne urządzenie zręcznościowe.<br/>Realizuje logikę gry i nadaje wynik radiowo"]:::internal
    BLACKBOX["Czarna Skrzynka<br/><b>[System Zewnętrzny]</b><br/><br/>Bramka Wi-Fi (Micro:bit B + Arduino).<br/>Odbiera pakiet radiowy i wysyła HTTP POST"]:::external
    LEADERBOARD["Platforma Rankingu Online<br/><b>[System]</b><br/><br/>Odbiera wyniki przez REST API,<br/>utrwala je w bazie i serwuje stronę WWW"]:::internal

    PLAYER -->|"Interakcja fizyczna i gra<br/>[Przyciski fizyczne / LED]"| CONSOLE
    CONSOLE -->|"Wysyła wynik oraz target_url<br/>[Radio 2.4 GHz]"| BLACKBOX
    BLACKBOX -->|"Przekazuje wynik ze znacznikiem czasu<br/>[HTTP POST (JSON) / Wi-Fi]"| LEADERBOARD
    PLAYER -->|"Przegląda publiczny ranking<br/>[Przeglądarka WWW / HTTPS]"| LEADERBOARD

    subgraph LEGEND [" Legenda "]
        direction LR
        L_PERS["Osoba / Użytkownik"]:::person
        L_INT["Nasz system (w zakresie)"]:::internal
        L_EXT["System zewnętrzny"]:::external
    end
    style LEGEND fill:#ffffff00,stroke:#cccccc,stroke-dasharray: 2 2;
```

### Opis elementów poziomu 1

| Element | Typ | Opis |
| :--- | :--- | :--- |
| **Gracz** | Osoba | Użytkownik fizycznie grający na konsoli Micro:bit A oraz weryfikujący swoje pozycje w rankingu przez przeglądarkę WWW. |
| **Konsola Gry (Micro:bit A)** | System (nasz zakres) | System wbudowany realizujący pętlę gry, detekcję wejść i wyświetlanie stanu gry. Po zakończeniu rozgrywki rozgłasza wynik radiowo. |
| **Czarna Skrzynka** | System zewnętrzny | Sprzętowa bramka Wi-Fi (Micro:bit B + Arduino). Odbiera pakiet radiowy od konsoli i wykonuje żądanie sieciowe `POST` pod wskazany adres `target_url`. |
| **Platforma Rankingu Online** | System (nasz zakres) | Centralny system odbierający wyniki przez REST API, utrwalający je w relacyjnej bazie danych oraz serwujący stronę rankingu. |

---

## Poziom 2: Kontenery (Containers)

Diagram kontenerów przedstawia dekompozycję systemów na niezależnie wdrażalne i uruchamialne jednostki oprogramowania (firmware, aplikacja kliencka, usługa backendowa, baza danych) oraz protokoły komunikacyjne pomiędzy nimi.

W architekturze zdefiniowano:
- **System: Konsola Gry** – kontener z oprogramowaniem wbudowanym (firmware MicroPython).
- **System: Platforma Rankingu Online** – trzy kontenery:
  1. **Frontend WWW** – statyczna aplikacja internetowa (HTML5 / JS Fetch API) wykonywana w przeglądarce gracza.
  2. **Backend REST API** – aplikacja serwerowa (FastAPI) w kontenerze Docker.
  3. **Baza Danych** – instancja PostgreSQL w kontenerze Docker.
- **Czarna Skrzynka** – zewnętrzny węzeł transmisyjny.

### Diagram kontenerów

```mermaid
flowchart TD
    classDef person fill:#08427b,stroke:#073b6f,color:#fff;
    classDef container fill:#1168bd,stroke:#0b4884,color:#fff;
    classDef db fill:#1168bd,stroke:#0b4884,color:#fff;
    classDef external fill:#6c757d,stroke:#495057,color:#fff;

    PLAYER["👤 Gracz<br/><b>[Osoba]</b><br/><br/>Gra na fizycznej konsoli oraz<br/>przegląda ranking wyników w przeglądarce"]:::person

    subgraph SYS_CONSOLE [" System: Konsola Gry "]
        MB_CONSOLE["Konsola Gry (Micro:bit A)<br/><b>[Kontener: Firmware MicroPython]</b><br/><br/>Realizuje logikę gry, obsługę przycisków,<br/>matrycy LED 5x5 oraz nadawanie radiowe"]:::container
    end

    BLACKBOX["Czarna Skrzynka<br/><b>[System Zewnętrzny]</b><br/><br/>Bramka Wi-Fi (Micro:bit B + Arduino).<br/>Odbiera pakiet radiowy i wysyła HTTP POST"]:::external

    subgraph SYS_LEADERBOARD [" System: Platforma Rankingu Online "]
        SPA["Frontend WWW<br/><b>[Kontener: HTML5 / JavaScript Fetch]</b><br/><br/>Pobiera i prezentuje tabelę rankingu<br/>w przeglądarce gracza"]:::container

        API["Backend REST API<br/><b>[Kontener: Python / FastAPI / Docker]</b><br/><br/>Obsługuje endpointy REST API, przetwarza wyniki<br/>oraz serwuje pliki statyczne frontendu"]:::container

        DB[("Baza Danych<br/><b>[Kontener: PostgreSQL / Docker]</b><br/><br/>Trwały magazyn danych. Przechowuje wpisy<br/>rankingowe w tabeli scores")]:::db
    end

    %% Relacje
    PLAYER -->|"Gra w grę zręcznościową<br/>[Przyciski fizyczne / LED]"| MB_CONSOLE
    MB_CONSOLE -->|"Wysyła player_id, score, target_url<br/>[Radio 2.4 GHz]"| BLACKBOX
    BLACKBOX -->|"POST /api/scores z timestampem<br/>[HTTP / JSON / Wi-Fi]"| API

    API -->|"Zapisuje i odczytuje wyniki<br/>[SQL / TCP 5432]"| DB

    PLAYER -->|"Wczytuje aplikację rankingową<br/>[Przeglądarka WWW]"| SPA
    API -.->|"Dostarcza pliki statyczne (HTML/JS)<br/>[HTTP]"| SPA
    SPA -->|"Pobiera dane rankingu (GET /api/leaderboard)<br/>[HTTP REST API / JSON]"| API

    subgraph LEGEND2 [" Legenda "]
        direction LR
        L2_PERS["Osoba / Użytkownik"]:::person
        L2_CONT["Kontener aplikacji"]:::container
        L2_DB[("Baza danych")]:::db
        L2_EXT["System zewnętrzny"]:::external
    end

    style SYS_CONSOLE fill:#1168bd0a,stroke:#1168bd,stroke-width:2px,stroke-dasharray: 4 4;
    style SYS_LEADERBOARD fill:#1168bd0a,stroke:#1168bd,stroke-width:2px,stroke-dasharray: 4 4;
    style LEGEND2 fill:#ffffff00,stroke:#cccccc,stroke-dasharray: 2 2;
```

### Opis elementów poziomu 2

| Kontener / Element | Technologia | Rola i odpowiedzialność | Komunikacja |
| :--- | :--- | :--- | :--- |
| **Konsola Gry (Micro:bit A)** | Firmware MicroPython | Realizacja logiki gry zręcznościowej, odczyt przycisków, obsługa animacji LED. Po zakończeniu nadaje radiowo identyfikator gracza, wynik oraz `target_url`. | `Radio 2.4 GHz` do Czarnej Skrzynki. |
| **Czarna Skrzynka** | Micro:bit B + Arduino (C++) | Pośrednik transmisyjny: Micro:bit B odbiera radio i przesyła UART-em do Arduino, które wysyła żądanie HTTP z dołączonym znacznikiem czasu. | Odbiór `Radio 2.4 GHz`, wysyłka `HTTP POST (Wi-Fi)` do API. |
| **Frontend WWW** | HTML5, JavaScript (Fetch API) | Lekki interfejs renderowany w przeglądarce internetowej gracza, prezentujący tabelę najlepszych wyników. | `HTTP GET /api/leaderboard` do Backend API. |
| **Backend REST API** | Python 3, FastAPI, Docker | Obsługa zapytań HTTP: `POST /api/scores` (walidacja i zapis wyniku) oraz `GET /api/leaderboard` (pobranie wyników). Serwuje pliki statyczne strony WWW. | Odbiór `HTTP` (port 8000), komunikacja z bazą przez `TCP 5432`. |
| **Baza Danych** | PostgreSQL, Docker | Przechowywanie tabeli `scores` (`id`, `player_id`, `score`, `timestamp`). Zapewnia trwałość i spójność danych rankingu. | Protokół PostgreSQL (`TCP 5432`). |

---

## Kluczowe usprawnienia wprowadzone względem wersji roboczej

1. **Jawne granice systemowe (System Boundaries):** Na Poziomie 2 wydzielono `SYS_CONSOLE`, co eliminuje problem kontenera zawieszonego poza jakimkolwiek systemem.
2. **Kierunkowość relacji (Unidirectional Arrows):** Zlikwidowano dwukierunkową strzałkę `SPA <--> API`. Relacje precyzyjnie rozróżniają pobieranie danych rankingu (odpytywanie API przez SPA) od dostarczania plików aplikacji webowej.
3. **Legenda diagramów:** Każdy diagram posiada zintegrowany klucz wizualny wyjaśniający role elementów i styl obramowania.
4. **Przenośność ikon:** Zastąpiono selektory FontAwesome uniwersalnymi symbolami Unicode, dzięki czemu diagramy renderują się bezbłędnie w każdym silniku Markdown.

# Architektura Systemu w Modelu C4

Dokumentacja architektoniczna projektu **System Gry Zręcznościowej IoT (Micro:bit + Arduino + Online Leaderboard)** opracowana zgodnie ze standardem C4 model.

---

## Poziom 2: Kontenery (Containers)

Diagram kontenerów przedstawia architekturę techniczną wysokiego poziomu – dekompozycję systemów na niezależnie wdrażalne jednostki wykonawcze (aplikacje, bazy danych, firmware) oraz protokoły komunikacji między nimi.

W naszym rozwiązaniu zdefiniowano następujące kontenery:
1. **Konsola gracza (Micro:bit A)**
2. **Frontend WWW** – strona internetowa z wynikami w przeglądarce.
3. **Backend REST API** – w kontenerze Docker.
4. **Baza Danych** – PostgreSQL w kontenerze Docker.

Pomiędzy konsolą a backendem pośredniczy **Czarna Skrzynka** (Micro:bit B + Arduino).

### Diagram kontenerów

```mermaid
flowchart TD
    classDef person fill:#08427b,stroke:#073b6f,color:#fff;
    classDef container fill:#1168bd,stroke:#0b4884,color:#fff;
    classDef db fill:#1168bd,stroke:#0b4884,color:#fff;
    classDef external fill:#6c757d,stroke:#495057,color:#fff;

    PLAYER["fa:fa-user Gracz<br/><b>[Osoba]</b><br/><br/>Gra na fizycznej konsoli oraz<br/>przegląda ranking wyników w przeglądarce"]:::person

    MB_CONSOLE["Konsola Gry (Micro:bit A)<br/><b>[Kontener: Firmware MicroPython]</b><br/><br/>Realizuje logikę gry, obsługę przycisków,<br/>matrycy LED 5x5 oraz nadawanie radiowe"]:::container

    BLACKBOX["Czarna Skrzynka<br/><b>[System Zewnętrzny]</b><br/><br/>Bramka Wi-Fi (Micro:bit B + Arduino).<br/>Odbiera pakiet radiowy i wysyła HTTP POST"]:::external

    subgraph SYS_LEADERBOARD [" System: Platforma Rankingu Online "]
        SPA["Frontend WWW<br/><b>[Kontener: HTML5 / JavaScript Fetch]</b><br/><br/>Pobiera i prezentuje tabelę rankingu"]:::container

        API["Backend REST API<br/><b>[Kontener: Python / FastAPI / Docker]</b><br/><br/>Obsługuje endpointy REST API, przetwarza wyniki<br/>oraz serwuje pliki statyczne frontendu"]:::container

        DB[("Baza Danych<br/><b>[Kontener: PostgreSQL / Docker]</b><br/><br/>Trwały magazyn danych. Przechowuje wpisy<br/>rankingowe w tabeli scores")]:::db
    end

    %% Przepływ danych
    PLAYER -->|"1. Gra w grę zręcznościową<br/>[Przyciski fizyczne / LED]"| MB_CONSOLE
    MB_CONSOLE -->|"2. Wysyła player_id, score, target_url<br/>[Radio 2.4 GHz]"| BLACKBOX
    BLACKBOX -->|"3. POST /api/scores z timestampem<br/>[HTTP / JSON / Wi-Fi]"| API

    API -->|"4. Zapisuje i odczytuje wyniki<br/>[SQL / TCP 5432]"| DB

    PLAYER -->|"5. Przegląda ranking<br/>[Przeglądarka WWW]"| SPA
    SPA <-->|"6. Pobiera ranking / Serwuje pliki<br/>[HTTP REST API / JSON]"| API

    style SYS_LEADERBOARD fill:#1168bd0d,stroke:#1168bd,stroke-width:2px,stroke-dasharray: 4 4;
```

### Opis kontenerów

#### 1. Kontenery Systemu Konsoli Gry
* **Gra (Micro:bit A):**
  - **Odpowiedzialność:** Pętla gry zręcznościowej, detekcja wejść z przycisków i sensorów, wyświetlanie animacji i punktów na matrycy LED 5x5.
  - **Komunikacja:** Po zakończeniu rozgrywki nadaje bezprzewodowo pakiet radiowy (2.4 GHz) zawierający identyfikator gracza, wynik oraz `target_url`.

#### 2. Kontenery Systemu Platformy Rankingu Online
* **Frontend WWW (Fetch API):**
  - **Odpowiedzialność:** Lekki interfejs użytkownika uruchamiany w przeglądarce internetowej gracza.
  - **Komunikacja:** Wysyła żądania `GET /api/leaderboard` do Backend API i wstawia dane do tabeli rankingu.
* **Backend REST API [Python, FastAPI, Docker]:**
  - **Odpowiedzialność:**
    - Udostępnienie endpointa `POST /api/scores` do zapisu nowego wyniku nadesłanego przez bramkę sieciową.
    - Udostępnienie endpointa `GET /api/leaderboard` zwracającego posortowany ranking wyników.
    - Serwowanie statycznych plików strony WWW (`index.html`, `app.js`).
  - **Komunikacja:** Odbiera ruch HTTP od Czarnej Skrzynki i Frontendu; komunikuje się z bazą danych przez protokół PostgreSQL (TCP port 5432).
* **Baza Danych [PostgreSQL, Docker]:**
  - **Odpowiedzialność:** Trwałe przechowywanie historii wyników w pojedynczej tabeli `scores` (`id`, `player_id`, `score`, `timestamp`).
  - **Komunikacja:** Połączenie TCP na domyślnym porcie 5432 z kontenerem Backend API.

#### 3. Zewnętrzne
* **Czarna Skrzynka [Micro:bit B + Arduino]:**
  - Zewnętrzny moduł transmisyjny: odbiera radiowy komunikat z Micro:bita A, formatuje go do JSON i przesyła jako żądanie HTTP `POST` pod wskazany `target_url` serwera.

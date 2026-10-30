# Architektura Systemu w Modelu C4

Dokumentacja architektoniczna projektu **System Gry Zręcznościowej IoT (Micro:bit + Arduino + Online Leaderboard)** opracowana zgodnie ze standardem C4 model.

---

## Poziom 1: Kontekst Systemu (System Context)

Diagram kontekstu przedstawia systemy wchodzące w skład rozwiązania na najwyższym poziomie abstrakcji, ich granice, użytkowników (aktorów) oraz relacje z zewnętrznym środowiskiem transmisyjnym.

W ramach rozwiązania zdefiniowano dwa niezależne systemy:
1. **Konsolę Gry (Micro:bit A)** – autonomiczny system wbudowany (embedded).
2. **Platformę Rankingu Online** – system serwerowo-webowy.

Pomiędzy nimi pośredniczy zewnętrzna **Czarna Skrzynka** (bramka Wi-Fi).

### Diagram kontekstu

```mermaid
flowchart TD
    classDef person fill:#08427b,stroke:#073b6f,color:#fff;
    classDef internal fill:#1168bd,stroke:#0b4884,color:#fff;
    classDef external fill:#6c757d,stroke:#495057,color:#fff;

    PLAYER["fa:fa-user Gracz<br/><b>[Osoba]</b><br/><br/>Gra na fizycznej konsoli oraz<br/>przegląda ranking wyników w przeglądarce"]:::person

    CONSOLE["Konsola Gry (Micro:bit A)<br/><b>[System]</b><br/><br/>Autonomiczne urządzenie zręcznościowe.<br/>Realizuje logikę gry i nadaje wynik radiowo"]:::internal
    BLACKBOX["Czarna Skrzynka<br/><b>[System Zewnętrzny]</b><br/><br/>Bramka Wi-Fi (Micro:bit B + Arduino).<br/>Odbiera pakiet radiowy i wysyła HTTP POST"]:::external
    LEADERBOARD["Platforma Rankingu Online<br/><b>[System]</b><br/><br/>Odbiera wyniki przez REST API,<br/>utrwala je w bazie i serwuje stronę WWW"]:::internal

    PLAYER -->|"1. Gra w grę zręcznościową<br/>[Przyciski / LED]"| CONSOLE
    CONSOLE -->|"2. Wysyła wynik i target_url<br/>[Radio 2.4 GHz]"| BLACKBOX
    BLACKBOX -->|"3. Przesyła wynik z timestampem<br/>[HTTP POST (JSON) / Wi-Fi]"| LEADERBOARD
    PLAYER -->|"4. Przegląda ranking wyników<br/>[Przeglądarka WWW / HTTP]"| LEADERBOARD
```

### Opis elementów kontekstu

#### 1. Aktorzy (People)
* **Gracz (Player):**
  - Wchodzi w bezpośrednią, fizyczną interakcję z konsolą Micro:bit A (sterowanie przyciskami, obserwacja gry na matrycy LED).
  - Korzysta z przeglądarki internetowej, aby sprawdzać aktualne wyniki w rankingu.

#### 2. Systemy wchodzące w zakres projektu (Our Systems)
* **Konsola Gry (Micro:bit A):**
  - Niezależny system wbudowany (firmware MicroPython).
  - Odpowiada za pętlę gry, zliczanie punktów, a po zakończeniu rozgrywki za nadanie bezprzewodowego komunikatu radiowego zawierającego `player_id`, `score` oraz `target_url`.
* **Platforma Rankingu Online:**
  - System serwerowy (FastAPI + PostgreSQL + statyczny interfejs WWW).
  - Wystawia punkt końcowy REST API do odbioru wyników, zapisuje je trwale w relacyjnej bazie danych oraz serwuje stronę WWW z tabelą wyników.

#### 3. Systemy zewnętrzne (External Systems)
* **Czarna Skrzynka (Bramka Wi-Fi: Micro:bit B + Arduino):**
  - Zewnętrzny węzeł transmisyjny pośredniczący w komunikacji.
  - Odbiera pakiet radiowy od konsoli i wykonuje żądanie sieciowe `POST` pod wskazany w pakiecie adres `target_url`, uzupełniając dane o znacznik czasu (`timestamp`).

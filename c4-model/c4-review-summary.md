# Podsumowanie Audytu i Przeglądu Dokumentacji C4 (Poziomy 1 i 2)

> Dokument podsumowujący ocenę jakości, poprawności architektonicznej oraz wprowadzonych usprawnień dla dokumentacji C4 Model w projekcie **System Gry Zręcznościowej IoT (Micro:bit + Arduino + Online Leaderboard)**.

---

## 1. Cel i Zakres Przeglądu

Przedmiotem analizy była ocena poprawności wykonania dokumentacji architektonicznej przygotowanej w standardzie C4 Model (autorstwa Simona Browna), znajdującej się w plikach:
- `c4-model/level-1.md` – Poziom 1: Kontekst Systemu (*System Context*),
- `c4-model/level-2.md` – Poziom 2: Kontenery (*Containers*),
w odniesieniu do specyfikacji technicznej zawartej w `README.md`.

---

## 2. Ocena Ogólna (Werdykt)

Dotychczasowe wykonanie dokumentacji oceniono jako **bardzo dobre (ok. 90% zgodności ze sztuką inżynierską)**, wyraźnie przewyższające typowe projekty akademickie:
- Prawidłowo zachowano poziomy abstrakcji pomiędzy kontekstem a dekompozycją kontenerową.
- Węzły diagramów zostały sformatowane w pożądanej konwencji C4: `Nazwa`, `[Typ / Rola]` oraz `Krótki opis odpowiedzialności`.
- Relacje na strzałkach precyzyjnie definiują protokoły i nośniki (`[Radio 2.4 GHz]`, `[HTTP POST (JSON) / Wi-Fi]`, `[SQL / TCP 5432]`).
- Zastosowano oficjalną paletę barw C4:
  - Osoba / Aktor: granatowy (`#08427b`),
  - Systemy / Kontenery własne: niebieski (`#1168bd`),
  - Systemy zewnętrzne: szary (`#6c757d`).

---

## 3. Zidentyfikowane Nieścisłości i Obszary do Poprawy

W trakcie szczegółowego audytu zidentyfikowano 5 kwestii wymagających korekty zgodnie z kanonem C4 Model:

### 1. Niespójność granic systemowych na Poziomie 2 (System Boundaries)
* **Stan pierwotny:** Na Poziomie 1 zdefiniowano dwa niezależne systemy w naszym zakresie: `Konsolę Gry (Micro:bit A)` oraz `Platformę Rankingu Online`. Na Poziomie 2 obrys systemu (`subgraph SYS_LEADERBOARD`) posiadała wyłącznie platforma rankingu. Konsola `MB_CONSOLE` została oznaczona jako kontener, lecz znajdowała się poza jakimkolwiek obrysem systemowym.
* **Wymóg C4:** Każdy kontener musi przynależeć do nadrzędnego systemu. Konsola powinna być ujęta we własny obrys `subgraph SYS_CONSOLE [" System: Konsola Gry "]`.

### 2. Obustronna strzałka `SPA <--> API`
* **Stan pierwotny:** Relacja `SPA <-->|"6. Pobiera ranking / Serwuje pliki [HTTP REST API / JSON]"| API`.
* **Wymóg C4:** 
  - Simon Brown rekomenduje bezwzględne unikanie strzałek dwukierunkowych (*„Avoid bidirectional arrows”*), ponieważ zamazują one kierunek zależności i inicjatora połączenia.
  - Relacja łączyła dwa wykluczające się kierunki: SPA (kod JavaScript działający w przeglądarce klienta) nie serwuje plików do API. To serwer API (FastAPI) udostępnia pliki statyczne (`HTML/JS`) do przeglądarki, a skrypt SPA wysyła zapytania `GET /api/leaderboard` do API.

### 3. Numeracja relacji (1, 2, 3...) – Diagram Statyczny vs Dynamiczny
* **Stan pierwotny:** Wszystkie relacje zostały ponumerowane sekwencyjnie.
* **Wymóg C4:** Statyczne diagramy kontekstu i kontenerów opisują stałe zdolności i powiązania systemu (np. *„Wysyła wyniki do”*). Numerowanie kroków jest domeną tzw. **Diagramu Dynamicznego** (C4 Dynamic Diagram) modelującego pojedynczy scenariusz użycia (*Use Case*). Krok przeglądania rankingu przez gracza nie jest przecież częścią sekwencji nadawania wyniku z konsoli.

### 4. Brak Legendy (Legend / Key)
* **Stan pierwotny:** Brak sekcji objaśniającej użyte barwy i kształty.
* **Wymóg C4:** Każdy formalny diagram C4 powinien zawierać legendę wyjaśniającą znaczenie kolorów (aktor, system wewnętrzny, system zewnętrzny, kontener, magazyn danych).

### 5. Kompatybilność ikon Mermaid (`fa:fa-user`)
* **Stan pierwotny:** `fa:fa-user` w etykiecie aktora.
* **Problem:** W rendererach markdown, które nie ładują zewnętrznych arkuszy FontAwesome (w tym w terminalach czy niektórych wtyczkach podglądu), ciąg ten wyświetla się jako surowy tekst `fa:fa-user Gracz`.
* **Rozwiązanie:** Zastąpienie zapisem z uniwersalnym emoji `👤 Gracz`.

---

## 4. Wdrożone Zmiany

W odpowiedzi na powyższe wnioski oraz zgłoszenie dotyczące braku renderowania diagramów w środowisku terminalowym CLI, utworzono skonsolidowany plik:
👉 **`c4-model/c4-diagrams.md`**

W pliku wdrożono:
1. **Pełny, poprawiony Diagram Poziomu 1 (Kontekst):**
   - Dodano wizualną legendę C4 w formie podgrafu Mermaid.
   - Wprowadzono przenośne ikony Unicode.
   - Opisy relacji sformułowano w sposób deklaratywny.
   - Uzupełniono tabelę katalogową elementów systemu.
2. **Pełny, poprawiony Diagram Poziomu 2 (Kontenery):**
   - Dodano brakujący `subgraph SYS_CONSOLE [" System: Konsola Gry "]`.
   - Zlikwidowano dwukierunkową strzałkę – rozbito komunikację na `SPA --> API` (odpytywanie o dane rankingu) oraz `API -.-> SPA` (dostarczanie plików statycznych).
   - Dodano dedykowaną legendę dla kontenerów i baz danych.
   - Uzupełniono kompleksową tabelę z technologiami, rolami i protokołami.

---

## 5. Struktura Plików Dokumentacji w Repozytorium

```text
jipp-5-sem/
├── README.md                     # Główny opis założeń technicznych i architektury projektu
└── c4-model/
    ├── level-1.md                # Wersja pierwotna: Poziom 1 (Kontekst Systemu)
    ├── level-2.md                # Wersja pierwotna: Poziom 2 (Kontenery)
    ├── c4-diagrams.md            # Zaktualizowana, kompletna dokumentacja C4 (L1 + L2 + tabele + legendy)
    └── c4-review-summary.md      # Niniejsze podsumowanie audytu architektonicznego
```

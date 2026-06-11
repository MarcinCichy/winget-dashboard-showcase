# Dokumentacja Projektu

Ten plik jest **indeksem dokumentacji**, a nie drugim pełnym README projektu.

Główny opis repozytorium, architektury i podstawowego uruchomienia znajduje się w:

- [README.md](../README.md)

## Jak używać tej dokumentacji

- `README.md` w root:
  - główny opis projektu,
  - quick start,
  - build agenta i instalatora,
  - podstawowy przegląd architektury.
- `docs/README.md`:
  - mapa dokumentów,
  - wejście do materiałów roboczych, historycznych i learning docs.

## Najważniejsze dokumenty aktywne

- [CURRENT_STATE_AND_REFACTOR_PLAN.md](./CURRENT_STATE_AND_REFACTOR_PLAN.md)
  - główny dokument roboczy: stan projektu, wykonane prace, dalsza kolejność refaktoru
- [QUICK_START.md](./QUICK_START.md)
  - szybkie uruchomienie lokalne
- [DEPLOY_FROM_WINDOWS.md](./DEPLOY_FROM_WINDOWS.md)
  - deploy z Windows przez SCP/SSH bez gita na serwerze
- [UPDATE_CONFIDENCE_MODEL.md](./UPDATE_CONFIDENCE_MODEL.md)
  - model rozdzielenia statusu wykonania aktualizacji od potwierdzenia jej efektu
- [../help/](../help/)
  - dokumentacja użytkowa widoczna także z poziomu aplikacji

## Katalogi w `docs/`

- `learning/`
  - krótkie techniczne lessons learned i case studies z wdrożeń, błędów i decyzji projektowych
- `archive/`
  - starsze checkpointy, plany, UAT, sesje i materiały referencyjne

## Kiedy aktualizować który plik

- aktualizuj `README.md` w root, gdy zmienia się:
  - opis projektu,
  - sposób uruchomienia,
  - proces builda,
  - architektura wysokiego poziomu
- aktualizuj `docs/README.md`, gdy zmienia się:
  - struktura dokumentacji,
  - najważniejsze aktywne dokumenty,
  - sposób poruszania się po `docs/`

## Uwaga utrzymaniowa

Nie duplikuj pełnej treści `README.md` do `docs/README.md`.

Jeśli ten plik zaczyna powtarzać:

- opis projektu,
- pełny quick start,
- cały proces builda,

to znaczy, że dokumentacja znów się rozjeżdża i trzeba wrócić do modelu:

- `README.md` = główny opis repo
- `docs/README.md` = indeks dokumentacji

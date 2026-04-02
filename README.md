# Winget Dashboard

Centralne zarzadzanie oprogramowaniem na komputerach Windows z uzyciem `winget` i `chocolatey`.

## Dokumentacja aktywna

- `CURRENT_STATE_AND_REFACTOR_PLAN.md` - glowny dokument roboczy: stan projektu, wykonane prace i dalsza kolejnosc refaktoru
- `QUICK_START.md` - szybkie uruchomienie lokalne
- `../help/` - dokumentacja dostepna z poziomu aplikacji i pozostawiona poza `docs/`
- `archive/` - starsze checkpointy, plany, UAT i notatki sesyjne

## Spis tresci

- [Opis](#opis)
- [Najwazniejsze funkcje](#najwazniejsze-funkcje)
- [Architektura](#architektura)
- [Struktura projektu](#struktura-projektu)
- [Zrzuty ekranu](#zrzuty-ekranu)
- [Uruchomienie serwera](#uruchomienie-serwera)
- [Budowa pakietu agenta](#budowa-pakietu-agenta)
- [Instalator Inno Setup](#instalator-inno-setup)
- [Testy](#testy)
- [Technology Stack](#technology-stack)
- [English Version](#english-version)

## Opis

`Winget Dashboard` to aplikacja Flask do monitorowania komputerow, raportowania stanu aplikacji i aktualizacji oraz zdalnego zlecania akcji takich jak update, uninstall, refresh raportu, logi czy aktualizacja agenta.

Projekt sklada sie z:
- serwera Flask w `winget_dashboard/`
- komponentow agenta Windows w `builder_helper/agent_src/`
- narzedzi do budowy i dystrybucji pakietu agenta

## Najwazniejsze funkcje

- centralny dashboard komputerow
- zdalne akcje dla aplikacji i systemu
- grupy komputerow i uprawnienia uzytkownikow
- scheduled tasks
- raporty i historia raportow
- logi agenta i logi serwera
- per-agent API keys
- zdalna aktualizacja agenta

## Architektura

Po stronie serwera projekt jest zorganizowany warstwowo:
- `api/` i `views.py` obsluguja HTTP
- `services/` zawiera logike biznesowa
- `repositories/` odpowiadaja za dostep do danych
- `db.py` zawiera `DatabaseManager`, `DatabaseContext` i inicjalizacje bazy

Bootstrap aplikacji zostal rozbity do:
- `winget_dashboard/app_setup/auth_setup.py`
- `winget_dashboard/app_setup/blueprints.py`
- `winget_dashboard/app_setup/errors.py`
- `winget_dashboard/app_setup/http_setup.py`
- `winget_dashboard/app_setup/infra_setup.py`
- `winget_dashboard/app_setup/logging_setup.py`
- `winget_dashboard/app_setup/templates.py`

Dodatkowo `winget_dashboard/dependencies.py` odpowiada za per-request tworzenie i cache zaleznosci serwisow.

## Struktura projektu

```text
winget_agent_new/
|-- agent_builds/                 # Paczki .zip agenta przechowywane po stronie serwera
|-- builder_helper/
|   |-- agent_src/                # Zrodla agenta Windows
|   |-- builder_helper.py         # Lokalny helper do budowy pakietu agenta
|   |-- error_definitions.json
|   `-- dist/                     # Wyniki lokalnego buildu helpera
|-- help/                         # Dokumentacja pomocy wykorzystywana przez aplikacje
|-- docs/
|   |-- archive/                 # Archiwum starszych notatek i planow
|   |-- CURRENT_STATE_AND_REFACTOR_PLAN.md
|   |-- QUICK_START.md
|   `-- README.md
|-- instance/                     # Runtime data, baza SQLite, logi
|-- Output/                       # Wyniki instalatora Inno Setup
|-- SourceFiles/                  # Wejscie do instalatora Inno Setup
|-- scripts/                      # Skrypty pomocnicze
|-- screenshots/                  # Zrzuty ekranu
|-- tests/                        # Testy
|-- winget_dashboard/
|   |-- api/                      # Endpointy API
|   |-- app_setup/                # Rozbity bootstrap aplikacji Flask
|   |-- repositories/             # Warstwa dostepu do danych
|   |-- services/                 # Logika biznesowa
|   |-- static/                   # CSS / JS / assets
|   |-- templates/                # Szablony Jinja2
|   |-- config.py
|   |-- db.py
|   |-- dependencies.py
|   |-- scheduler.py
|   |-- schema.sql
|   `-- views.py
|-- .env.example
|-- docker-compose.yml
|-- Dockerfile
|-- requirements.txt
|-- requirements-windows.txt
|-- run.py
`-- setup_script.iss
```

## Zrzuty ekranu

| Widok | Opis |
|------|------|
| ![Strona glowna - kafelki](../screenshots/main.png) | Strona glowna - widok kafelkow |
| ![Strona glowna - lista](../screenshots/list.png) | Strona glowna - widok tabeli |
| ![Szczegoly komputera](../screenshots/computer.png) | Szczegolowy widok komputera |
| ![Ustawienia komputera](../screenshots/computer_settings.png) | Ustawienia i konfiguracja komputera |
| ![Grupy](../screenshots/groups.png) | Zarzadzanie grupami komputerow |
| ![Zaplanowane zadania](../screenshots/scheduler.png) | Widok scheduled tasks |
| ![Historia raportow](../screenshots/history.png) | Historia raportow |
| ![Ustawienia](../screenshots/settings.png) | Ustawienia serwera |
| ![Uzytkownicy](../screenshots/users.png) | Zarzadzanie uzytkownikami |
| ![Pomoc](../screenshots/help.png) | Wbudowana dokumentacja pomocy |

## Uruchomienie serwera

### Wymagania

- Python 3.8+
- Git

### Instalacja

```bash
git clone <adres-repozytorium>
cd winget_agent_new
python -m venv venv
```

Aktywacja virtualenv:

```bash
# Windows
venv\Scripts\activate

# Linux/macOS
source venv/bin/activate
```

Instalacja zaleznosci:

```bash
pip install -r requirements.txt
```

Utworzenie konfiguracji:

```bash
# Windows
copy .env.example .env

# Linux/macOS
cp .env.example .env
```

Minimalne wpisy w `.env`:

```ini
SECRET_KEY=<twoj_klucz_sesji>
API_KEY=<twoj_klucz_api>
RATELIMIT_ENABLED=true
NOTIFICATIONS_ENABLED=false
```

Inicjalizacja bazy:

```bash
flask --app run init-db
```

Uruchomienie developerskie:

```bash
python run.py
```

Uruchomienie produkcyjne:

```bash
waitress-serve --host=0.0.0.0 --port=5000 winget_dashboard:create_app
```

## Budowa pakietu agenta

Budowa wykonywana jest lokalnie na stacji administratora Windows.

Instalacja zaleznosci:

```bash
pip install -r requirements-windows.txt
```

Budowa `builder_helper.exe`:

```bash
pyinstaller --onefile --name builder_helper builder_helper/builder_helper.py
```

Po uruchomieniu `builder_helper.exe` generator w panelu WWW moze przygotowac spersonalizowany pakiet `.zip` agenta.

## Instalator Inno Setup

Repo korzysta obecnie z plikow w root:
- `SourceFiles/`
- `Output/`
- `setup_script.iss`

Typowy przeplyw:
1. skopiuj `agent.exe`, `ui_helper.exe`, `updater.exe` do `SourceFiles/`
2. otworz `setup_script.iss` w Inno Setup
3. zaktualizuj numer wersji i skompiluj instalator
4. wynik pojawi sie w `Output/`

## Testy

Uruchomienie wszystkich testow:

```bash
pytest
```

Coverage:

```bash
pytest --cov=winget_dashboard --cov-report=html
```

Uwaga: pelne `pytest` moze obecnie dawac niestabilny sygnal w tym repo z powodu problemow srodowiska z katalogami tymczasowymi `pytest_runtime_tmp`.

## Technology Stack

- Backend: Python, Flask, Waitress, SQLite
- Frontend: HTML, CSS, JavaScript
- Agent: Python, pywin32, requests
- Package managers: winget, chocolatey
- Build tools: PyInstaller, Inno Setup
- Tests: pytest, Selenium

## English Version

### Description

`Winget Dashboard` is a Flask-based application for monitoring Windows computers, reporting installed software and pending updates, and triggering remote actions such as update, uninstall, report refresh, log collection, and agent self-update.

The project consists of:
- a Flask server in `winget_dashboard/`
- Windows agent components in `builder_helper/agent_src/`
- helper tooling for building and distributing the agent package

### Key Features

- central dashboard for managed computers
- remote software and OS actions
- computer groups and user access control
- scheduled tasks
- reports and report history
- agent and server logs
- per-agent API keys
- remote agent update flow

### Architecture

On the server side, the project is organized in layers:
- `api/` and `views.py` handle HTTP
- `services/` contains business logic
- `repositories/` handle data access
- `db.py` contains `DatabaseManager`, `DatabaseContext`, and DB initialization

Flask bootstrap is split into:
- `winget_dashboard/app_setup/auth_setup.py`
- `winget_dashboard/app_setup/blueprints.py`
- `winget_dashboard/app_setup/errors.py`
- `winget_dashboard/app_setup/http_setup.py`
- `winget_dashboard/app_setup/infra_setup.py`
- `winget_dashboard/app_setup/logging_setup.py`
- `winget_dashboard/app_setup/templates.py`

`winget_dashboard/dependencies.py` is responsible for per-request service dependency wiring.

### Project Structure

```text
winget_agent_new/
|-- agent_builds/
|-- builder_helper/
|   |-- agent_src/
|   |-- builder_helper.py
|   |-- error_definitions.json
|   `-- dist/
|-- help/
|-- docs/
|   |-- archive/
|   |-- CURRENT_STATE_AND_REFACTOR_PLAN.md
|   |-- QUICK_START.md
|   `-- README.md
|-- instance/
|-- Output/
|-- SourceFiles/
|-- scripts/
|-- screenshots/
|-- tests/
|-- winget_dashboard/
|   |-- api/
|   |-- app_setup/
|   |-- repositories/
|   |-- services/
|   |-- static/
|   |-- templates/
|   |-- config.py
|   |-- db.py
|   |-- dependencies.py
|   |-- scheduler.py
|   |-- schema.sql
|   `-- views.py
|-- .env.example
|-- docker-compose.yml
|-- Dockerfile
|-- requirements.txt
|-- requirements-windows.txt
|-- run.py
`-- setup_script.iss
```

### Screenshots

| View | Description |
|------|-------------|
| ![Main - tiles](../screenshots/main.png) | Main page - tile view |
| ![Main - list](../screenshots/list.png) | Main page - table view |
| ![Computer details](../screenshots/computer.png) | Detailed computer view |
| ![Computer settings](../screenshots/computer_settings.png) | Computer settings |
| ![Groups](../screenshots/groups.png) | Group management |
| ![Scheduled tasks](../screenshots/scheduler.png) | Scheduled tasks view |
| ![Report history](../screenshots/history.png) | Report history |
| ![Settings](../screenshots/settings.png) | Server settings |
| ![Users](../screenshots/users.png) | User management |
| ![Help](../screenshots/help.png) | Built-in help |

### Server Setup

Requirements:
- Python 3.8+
- Git

Installation:

```bash
git clone <repository-address>
cd winget_agent_new
python -m venv venv
```

Activate virtualenv:

```bash
# Windows
venv\Scripts\activate

# Linux/macOS
source venv/bin/activate
```

Install dependencies:

```bash
pip install -r requirements.txt
```

Create config:

```bash
# Windows
copy .env.example .env

# Linux/macOS
cp .env.example .env
```

Minimal `.env`:

```ini
SECRET_KEY=<your-session-key>
API_KEY=<your-api-key>
RATELIMIT_ENABLED=true
NOTIFICATIONS_ENABLED=false
```

Initialize the database:

```bash
flask --app run init-db
```

Run in development:

```bash
python run.py
```

Run in production:

```bash
waitress-serve --host=0.0.0.0 --port=5000 winget_dashboard:create_app
```

### Agent Package Build

Agent package build is performed locally on a Windows admin workstation.

```bash
pip install -r requirements-windows.txt
pyinstaller --onefile --name builder_helper builder_helper/builder_helper.py
```

After starting `builder_helper.exe`, the web panel generator can create a personalized `.zip` agent package.

### Inno Setup Installer

Current installer-related files live in the repo root:
- `SourceFiles/`
- `Output/`
- `setup_script.iss`

Typical flow:
1. copy `agent.exe`, `ui_helper.exe`, `updater.exe` into `SourceFiles/`
2. open `setup_script.iss` in Inno Setup
3. update the version and compile the installer
4. the result will appear in `Output/`

### Tests

```bash
pytest
pytest --cov=winget_dashboard --cov-report=html
```

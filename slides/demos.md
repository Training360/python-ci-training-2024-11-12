# CI/CD implementálása Python projekten

## Előfeltételek

Szoftverek:

* [Parancssoros Git kliens](https://git-scm.com/downloads)
* [Python interpreter](https://www.python.org/downloads/)
* [Visual Studio Code](https://code.visualstudio.com/), extensionök:
  * Python
  * Markdown Execute
* [Docker Desktop](https://www.docker.com/products/docker-desktop/)
* Bármilyen adatbáziskezelő kliens, pl. DBeaver

Regisztráció:

* [GitHub](https://github.com/)

## Python ellenőrzése

```sh
python --version
```

## Virtuális környezet

```sh
python -m venv venv --prompt employees
venv\Scripts\activate
```

## Packaging

[Python Packaging User Guide](https://packaging.python.org/en/latest/)

> A collection of tutorials and references to help you distribute and install Python packages with modern tools.

> The authoritative resource on how to package, publish, and install Python projects using current tools.

Karbantartja: [Python Packaging Authority](https://www.pypa.io/en/latest/) munkacsoport

[Python Enhancement Proposals (PEP)](https://peps.python.org/)

Python nyelv hivatalos fejlesztési javaslatainak rendszere, egyfajta útmutatóként és döntéshozatali eszközként szolgálnak a Python jövőbeni fejlődési irányait illetően.

* Python script: önmagában futtatható megfelelő Python interpreterrel, ha csak standard library-re van függősége
* Python module: Python file
* Import package: könyvtár, mely Python fájlokat tartalmazhat
* Source Distribution, _sdist_, tömörített fájl (pl. `tar.gz`), mely több könyvtárat és fájlt tartalmazhat (import package és module), önmagában nem használható, build kell hozzá
* Build Distribution, önmagában is használható, egyszerűen oda kell másolni.
* Wheel egy standard bináris Build Distribution. Hatékonyabb, mint a Source Distribution Package
* Best practice mindkét Distribution előállítása és publikálása

## Packaging flow

* _Source tree_, verziókezelőben tárolt
* Konfigurációst fájl, pl. `pyproject.toml`
* Build egy build toollal, előáll a Source és Build Distribution
* Publikálás, pl. a [Python Package Index-re (PyPI)](https://pypi.org/)

Elkészült és publikált csomag:

* Letölthető
* Saját környezetbe telepíthető (itt történhet build is - konfigurációs fájl alapján)
  * Tipikusan a Python környezet `site-packages` könyvtárába

## Konfigurációs fájl

```toml
[build-system]
requires = ["setuptools>=75.0.0", "wheel"]
build-backend = "setuptools.build_meta"
```

(A `[ ]` karakterekkel jelölt szavak jelzik a TOML fájlban a table-öket.)

Elválik a build frontend és a build backend, köztük egy vékony API. A build backend végzi a konkrét műveleteket, ezek lehetnek a flit, hatch, pdm, poetry, Setuptools, trampolim, és whey.

Build frontend pl. a build, mely egy [PEP 517 ](https://peps.python.org/pep-0517/) build frontend.

A pip a legelterjedtebb eszköz csomagok telepítésére. Amikor telepítés közben buildelni is kell, akkor build frontendként viselkedik.

## Függőségek

PIP frissítése

```sh
python -m pip install --upgrade pip
```

Telepítendő: Visual Studio TOML extension

```toml
[build-system]
requires = ["setuptools>=75.0.0", "wheel"]
build-backend = "setuptools.build_meta"

[project]
name = "employees"
version = "1.0.0"
dependencies = [
    "Flask",
    "psycopg[binary,pool]",
    "Flask-WTF"
]

[project.optional-dependencies]
dev = [
    "build"
]
```

Különválasztva a futtatáshoz és a fejlesztéshez szükséges csomagok.

Csomagok telepítése:

```sh
python -m pip install --editable ".[dev]"
```

[Development mode](https://setuptools.pypa.io/en/latest/userguide/development_mode.html): `-e` / `--editable`

Load the code under development directly from the project folder. Az editable package Pythonban egy olyan csomag, amelyet fejlesztési környezetben közvetlenül a forráskódból lehet használni telepítés nélkül.

## Freeze

```sh
python -m pip freeze --exclude-editable > constraints.txt
```

Ha más akarja pont ugyanazt telepíteni:

```shell
python -m pip install -c constraints.txt .
```

## Build

Source distribution előállítása:

```sh
python -m build --sdist .
```

Eredmény a `dist/employees-1.0.0.tar.gz` fájl.

Wheel Build distribution előállítása:

```sh
python -m build --wheel .
```

Eredmény a `dist/employees-1.0.0-py3-none-any.whl` fájl.

## Futtatás

Adatbázis indítása Docker konténerben:

```sh
docker run -d -e POSTGRES_DB=employees -e POSTGRES_USER=employees  -e POSTGRES_PASSWORD=employees  -p 5432:5432  --name employees-postgres postgres
```

```sh
flask --app employees run --debug
```

## Tesztelés

### Unit tesztek

pytest

```sh
python -m pytest -v test/unit/
```

`-v` verbosity

Tesztek futtathatók Visual Studio Code-ból

Visual Studio Code: Testing / Configure Python Tests

### Coverage

pytest-cov

```toml
[project.optional-dependencies]
dev = [
    "pytest-cov",
]
```

```sh
pytest --cov=employees --cov-report=xml -v test/unit
```

### Integrációs tesztek

```sh
python -m pytest -v test/integration/
```

### E2E tesztek

API:

requests

```sh
python -m pytest -v test/e2e/test_employees_api.py
```

E2E:

Selenium

```sh
python -m pytest -v test/e2e/test_employees_ui.py
```

## Statikus kódellenőrzés

### Statikus kódellenőrző eszközök

Elterjedtek:

* Black: kódformázás
* Flake8: PEP8 és clean code
* Mypy: statikus típusellenőrzés
* Pylint: részletes kódminőség ellenőrzés
* isort: importok rendezése
* Bandit: biztonsági hibák és sebezhetőségek

Kevésbé elterjedtek:

* [pydocstyle](https://www.pydocstyle.org/en/stable/): Python docstring konvenciók [PEP 257](http://www.python.org/dev/peps/pep-0257/) ellenőrzésére
* [pyupgrade](https://github.com/asottile/pyupgrade): újabb nyelvi elemekre konvertál
* [vulture](https://pypi.org/project/vulture/): unused code

Újabb versenyző:

* Ruff
  * 10-100x gyorsabb, mint a Flake8 vagy Black
  * `pyproject.toml` support
  * Replace Flake8 (plus dozens of plugins), Black, isort, pydocstyle, pyupgrade, autoflake, stb.

```sh
pre-commit autoupdate --repo https://github.com/pycqa/isort
```

### Ruff

[Ruff](https://docs.astral.sh/ruff/)

VSVisual Studio Code extension: Ruff

Markdown dokumentumokban lévő kódokat is tud kezelni

### pre-commit

[pre-commit](https://pre-commit.com/)

> Multi-language package manager for pre-commit hook

[Pár előre gyártott hook](https://github.com/pre-commit/pre-commit-hooks)

```sh
pre-commit run --all-files
```

Ha csak valamelyik hookot akarjuk lefuttatni:

```sh
pre-commit run mypy --all-files
```

Alapból minden toolt saját virtual environmentben futtat. Ha azt akarjuk, hogy a
már létrehozott virtual environmentben fusson, akkor kell a `language: system`
alkalmazása.

Git hook script installálása:

```sh
pre-commit install
```

## Dokumentáció

### Eszközök

* sphinx
  * docstring alapján
  * reStructuredText vagy MyST Markdown
  * cross-reference
  * Különböző formátumok: HTML, LaTeX (for PDF), ePub, Texinfo, stb.
  * Témák
* pdoc
  * Egyszerűbb, mint a Sphinx
  * Markdown itt is behúzható
  * PlantUML illeszthető
* MkDocs
  * Markdown alapú static site generator
* [Pandoc](https://pandoc.org/)
  * Universal document converter

pdoc

```sh
pdoc -d restructuredtext -o .\build\docs employees
```

MkDocs

Alt + D

```sh
mkdocs serve
```

```sh
mkdocs build
```

### PlantUML

VSCode Extension: PlantUML

mkdocs_puml

Saját belső PlantUML szerver is használható, Dockerben indítható.

## Docker

```sh
docker build -t employees .
```

```sh
docker run -d -p 5000:5000 --name my-employees employees
```

Gunicorn

```sh
docker compose up -d
```

```sh
docker build -t employees-test -f Dockerfile.test .
```

```sh
docker compose --profile api-test up --abort-on-container-exit
```

```sh
docker compose --profile ui-test up --abort-on-container-exit
```


## GitHub push

```sh
gh repo create employees-python-2024-11-12 --public --source=. --remote=origin --push
```

## Versioning schemes

* [Semantic versioning](https://semver.org/)

## GitHub release

Generate release notes: 

* Merged pull requests
* Contributors

## GitHub Actions

* Munkafolyamatok automatizálására, de ebből csak egy a CI/CD

Javasolt Visual Studio Code extension: GitHub Actions

* VM (hosted v. self-hosted), containers
* Matrix build, különböző operációs rendszereken, interpreter verziókon, stb.
* Integráció a GitHub repository-kkal, és GitHub Packages-zel, Package Registry-vel

* Event, workflow, job, step
* Step: script vagy action (újrafelhasználható komponens) futtatása

* Jobok
  * Egy jobot ugyanaz a runner futtatja, azaz a benne lévő steppeket is
  * Jobok alapból párhuzamosan futnak
  * Lehet közöttük függőséget definiálni, akkor szekvenciálisan

Event:

* Creates a PR
* Contributor joined
* Opens an issue
* Pushes a commit to a repository
* PR merged
* Kapcsolt eszközök is tudnak eventet küldeni

[Workflow templates](https://github.com/actions/starter-workflows/tree/main/ci)

* [Actions](https://github.com/actions)
* [Marketplace](https://github.com/marketplace)

### Act

[Act](https://github.com/nektos/act)

```sh
gh extension install https://github.com/nektos/gh-act
```

GitHub Actions Visual Studio Code extension



```sh
gh act push
```

Medium

### Docker

`ubuntu` image-en van előretelepített Docker

Vannak actionök

Push:

* DockerHub
* GitHub Registry

### Release

[bump-my-version](https://callowayproject.github.io/bump-my-version/)

```sh
git push --tags
```

### GitHub Pages

Settings / Pages: Source: GitHub Actions
# <img src=".gitlab/gitlab.png" alt="gitlab" height="30"/> bootstrap

::include{file=.gitlab/badges.md}

Zestaw komponentów GitLab CI/CD ujednolicających przygotowanie i zakończenie
jobów. Komponenty dostarczają wspólny obraz wykonawczy, funkcje logowania,
inicjalizację środowiska, baner pipeline, narzędzia GitLab oraz odnośnik do
dokumentacji joba.

Repozytorium udostępnia dwa niezależne komponenty:

| Komponent | Ukryty job | Zastosowanie |
|---|---|---|
| `before_script` | `.before_script` | Przygotowanie środowiska przed `script` |
| `after_script` | `.after_script` | Wyświetlenie dokumentacji po zakończeniu `script` |

## Użycie

```yaml
include:
  - component: $CI_SERVER_FQDN/dev.rachuna/pipelines/gitlab/components/bootstrap/before_script@1.0.0
    inputs:
      image: registry.gitlab.com/dev.rachuna/artifacts/containers/python:1.2.0
      before_script:
        - echo "Dodatkowe przygotowanie joba"

  - component: $CI_SERVER_FQDN/dev.rachuna/pipelines/gitlab/components/bootstrap/after_script@1.0.0
    inputs:
      image: registry.gitlab.com/dev.rachuna/artifacts/containers/python:1.2.0
      after_script:
        - echo "Dodatkowe sprzątanie po jobie"

verify:
  extends:
    - .before_script
    - .after_script
  variables:
    PIPELINE_TYPE: CI
    PROJECT_TYPE: gitlab-pipelines
    DOCS_MD_FILE_PATH: docs/verify/README.md
    CI_JOB_BEFORE_SCRIPT: |
      - echo "Dodatkowe przygotowanie joba"
    CI_JOB_AFTER_SCRIPT: |
      - echo "Dodatkowe sprzątanie po jobie"
  script:
    - h1 "Weryfikacja"
    - echo "Uruchamiam testy"
```

W pipeline produkcyjnym należy wskazywać opublikowany tag SemVer zamiast
gałęzi lub `latest`.

## `before_script`

Komponent rozszerza job o następujące kroki, wykonywane w podanej kolejności:

1. Ładuje funkcje logowania z helpera `logger`.
2. Inicjalizuje środowisko przez `job-prepare`.
3. Wyświetla baner za pomocą helpera `logo`.
4. Ładuje funkcje helpera `gitlab-tools`.
5. Uruchamia polecenia przekazane przez input `before_script`.
6. Uruchamia zawartość zmiennej `CI_JOB_BEFORE_SCRIPT`, jeśli jest ustawiona.

### Inputs

| Input | Typ | Wartość domyślna | Opis |
|---|---|---|---|
| `image` | string | `registry.gitlab.com/dev.rachuna/artifacts/containers/python:1.2.0` | Obraz joba |
| `before_script` | array | `[]` | Dodatkowe polecenia wykonywane po helperach |

## `after_script`

Komponent rozszerza job o następujące kroki:

1. Ładuje funkcje logowania z helpera `logger`.
2. Ponownie inicjalizuje środowisko przez `job-prepare`, ponieważ GitLab
   wykonuje `after_script` w osobnym procesie.
3. Wyświetla wynik joba oraz link do jego dokumentacji przez `job-docs`.
4. Uruchamia polecenia przekazane przez input `after_script`.
5. Uruchamia zawartość zmiennej `CI_JOB_AFTER_SCRIPT`, jeśli jest ustawiona.

### Inputs

| Input | Typ | Wartość domyślna | Opis |
|---|---|---|---|
| `image` | string | `registry.gitlab.com/dev.rachuna/artifacts/containers/python:1.2.0` | Obraz joba |
| `after_script` | array | komunikat z nazwą projektu i refem | Dodatkowe polecenia wykonywane po helperach |

## Zmienne

| Zmienna | Wymagana | Opis |
|---|---|---|
| `CI_CONFIG_PATH` | tak | Źródło informacji o repozytorium konfiguracji CI; GitLab ustawia ją automatycznie |
| `PIPELINE_TYPE` | dla `before_script` | Typ pipeline wyświetlany w banerze, na przykład `CI` lub `CD` |
| `DOCS_MD_FILE_PATH` | dla `after_script` | Ścieżka do dokumentacji joba |
| `PROJECT_TYPE` | nie | Dla wartości `gitlab-pipelines` helper sprawdza, czy plik dokumentacji istnieje |
| `CI_JOB_BEFORE_SCRIPT` | nie | Dodatkowy skrypt uruchamiany na końcu `before_script` |
| `CI_JOB_AFTER_SCRIPT` | nie | Dodatkowy skrypt uruchamiany na końcu `after_script` |

Zmienne `CI_JOB_BEFORE_SCRIPT` i `CI_JOB_AFTER_SCRIPT` są wykonywane przez
`eval`. Powinny pochodzić wyłącznie z zaufanej konfiguracji CI/CD.

## Dostępne helpery

| Helper | Odpowiedzialność |
|---|---|
| `logger` | Funkcje `banner`, `h1`, `h2`, `h3`, `info`, `warn` i `critical` |
| `job-prepare` | Tryb strict mode, parsowanie `CI_CONFIG_PATH` i funkcja `check_var` |
| `logo` | Baner z typem pipeline i gałęzią repozytorium CI |
| `gitlab-tools` | Funkcje `gitlab_commit_amend`, `run_new_pipeline` i `gitlab_ssh_key` |
| `job-docs` | Link do dokumentacji oraz status joba |

Szczegółowy opis helperów i ich dodatkowych wymagań znajduje się w katalogu
[`helpers`](helpers/).

## Struktura repozytorium

```text
.
├── helpers/
│   ├── gitlab-tools/
│   ├── job-docs/
│   ├── job-prepare/
│   ├── logger/
│   └── logo/
└── templates/
    ├── after_script/template.yml
    └── before_script/template.yml
```

Każdy komponent znajduje się w `templates/<nazwa>/template.yml`, zgodnie ze
strukturą wymaganą przez GitLab CI/CD Catalog. Pliki helperów są dołączane
lokalnie przez komponenty i udostępniają ukryte fragmenty konfiguracji
wykorzystywane przez `!reference`.

## Wydanie

1. Połącz zmiany z gałęzią domyślną.
2. Utwórz tag zgodny z SemVer, na przykład `1.0.0`.
3. Pipeline powinien zweryfikować komponenty i utworzyć GitLab Release.

Aby publikować wydania w katalogu, w ustawieniach projektu GitLab musi być
włączona opcja **CI/CD Catalog project**.

---

::include{file=.gitlab/footer.md}

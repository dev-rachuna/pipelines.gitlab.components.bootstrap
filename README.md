# <img src=".gitlab/gitlab.png" alt="GitLab" height="30"/> Bootstrap CI/CD Components

::include{file=.gitlab/badges.md}

Zestaw komponentów GitLab CI/CD do przygotowania jobów, walidacji kodu i
automatycznego wersjonowania projektów.

## Komponenty

| Komponent | Zastosowanie | Domyślny job lub ukryty job |
|---|---|---|
| [`_before_script`](templates/_before_script/README.md) | Przygotowanie środowiska i helperów przed `script` | `.before_script` |
| [`_after_script`](templates/_after_script/README.md) | Wyświetlenie statusu i dokumentacji po `script` | `.after_script` |
| [`conventional-commits`](templates/conventional-commits/README.md) | Walidacja tytułów commitów | `conventional-commits` |
| [`dynamic-deployment-environment`](templates/dynamic-deployment-environment/README.md) | Generowanie jobów wdrożeniowych dla aktywnych środowisk | `dynamic-deployment-environment` |
| [`shellcheck`](templates/shellcheck/README.md) | Walidacja skryptów powłoki | `shellcheck` |
| [`yamllint`](templates/yamllint/README.md) | Walidacja plików YAML | `yamllint` |
| [`versioning`](templates/versioning/README.md) | Semantic Release, wersja i changelog | `versioning` |

## Helpery

| Helper | Zastosowanie |
|---|---|
| [`logger`](helpers/logger/README.md) | Formatowane logowanie, nagłówki, bannery i komunikaty krytyczne |
| [`job-prepare`](helpers/job-prepare/README.md) | Inicjalizacja środowiska i danych repozytorium CI |
| [`logo`](helpers/logo/README.md) | Wyświetlanie banera z typem pipeline i gałęzią |
| [`gitlab-tools`](helpers/gitlab-tools/README.md) | Funkcje GitLab API, Git i konfiguracja SSH |
| [`glab`](helpers/glab/README.md) | Uwierzytelnianie GitLab CLI i konfiguracja operacji Git przez HTTPS |
| [`job-docs`](helpers/job-docs/README.md) | Wyświetlanie statusu joba i odnośnika do dokumentacji |

Wszystkie komponenty są wersjonowane wspólnie. Dla stabilnych pipeline należy
używać pełnego taga SemVer, na przykład `1.0.0`.

## Szybki start

Komponenty jobów korzystają z ukrytych jobów `.before_script` i
`.after_script`. Należy dołączyć je razem z wybranymi jobami:

```yaml
include:
  - component: $CI_SERVER_FQDN/dev.rachuna/pipelines/gitlab/components/bootstrap/_before_script@1.0.0
  - component: $CI_SERVER_FQDN/dev.rachuna/pipelines/gitlab/components/bootstrap/_after_script@1.0.0

  - component: $CI_SERVER_FQDN/dev.rachuna/pipelines/gitlab/components/bootstrap/conventional-commits@1.0.0

  - component: $CI_SERVER_FQDN/dev.rachuna/pipelines/gitlab/components/bootstrap/dynamic-deployment-environment@1.0.0
    inputs:
      job-stage: prepare
      job-definitons-script: |
        "Deploy:$ENV_NAME":
          stage: deployment
          extends: [.deployment]
          environment:
            name: $ENV_NAME

  - component: $CI_SERVER_FQDN/dev.rachuna/pipelines/gitlab/components/bootstrap/shellcheck@1.0.0
    inputs:
      job-before-script: |
        echo "Przygotowanie plików do analizy"

  - component: $CI_SERVER_FQDN/dev.rachuna/pipelines/gitlab/components/bootstrap/yamllint@1.0.0

  - component: $CI_SERVER_FQDN/dev.rachuna/pipelines/gitlab/components/bootstrap/versioning@1.0.0
    inputs:
      job-dry-run: true
```

> [!warning]
> Komponent `_before_script` wymaga zmiennej `GITLAB_TOKEN`, używanej przez
> helper `glab` do uwierzytelnienia w GitLab.com. Komponent `versioning`
> korzysta z tego samego tokenu. Operacje publikujące changelog wymagają
> również `GITLAB_USER_NAME` i `GITLAB_USER_EMAIL`.

## Zmienne sterujące

| Zmienna | Efekt dla wartości `false` |
|---|---|
| `ENABLED_CONVENTIONAL_COMMITS` | Wyłącza job Conventional Commits |
| `ENABLED_SHELLCHECK` | Wyłącza job ShellCheck |
| `ENABLED_YAMLLINT` | Wyłącza job yamllint |
| `ENABLED_VERSIONING` | Wyłącza job versioning |

Każdy gotowy job udostępnia inputy `job-name`, `job-stage`, `job-image`,
`job-rules`, `job-needs`, `job-resource-group`, `job-before-script` i
`job-after-script`. Komponent `dynamic-deployment-environment` udostępnia
dodatkowo input `job-definitons-script`, który definiuje job generowany dla
każdego aktywnego środowiska. Szczegółowe wartości znajdują się w dokumentacji
poszczególnych komponentów poniżej.

## Struktura repozytorium

```text
.
├── helpers/
│   ├── gitlab-tools/
│   ├── glab/
│   ├── job-docs/
│   ├── job-prepare/
│   ├── logger/
│   └── logo/
├── templates/
│   ├── _after_script/
│   ├── _before_script/
│   ├── conventional-commits/
│   ├── dynamic-deployment-environment/
│   ├── shellcheck/
│   ├── versioning/
│   └── yamllint/
├── .gitlab-ci.yml
├── LICENSE
└── README.md
```

---

::include{file=.gitlab/footer.md}

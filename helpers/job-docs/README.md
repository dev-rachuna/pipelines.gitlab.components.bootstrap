# Helper: job-docs

Wyświetla po zakończeniu joba kolorowy baner z odnośnikiem do jego
dokumentacji.

## Co robi

1. Sprawdza, czy zmienna `DOCS_MD_FILE_PATH` jest zdefiniowana.
2. Dla `PROJECT_TYPE=gitlab-pipelines` sprawdza, czy wskazany plik istnieje
   w katalogu roboczym.
3. Wybiera repozytorium i ref zawierające dokumentację.
4. Buduje URL do pliku w widoku GitLab `/-/blob/`.
5. Wyświetla zielony baner dla `CI_JOB_STATUS=success`, a czerwony dla
   pozostałych statusów.

Brak wymaganej zmiennej lub pliku kończy wykonanie przez funkcję `critical`.

## Budowanie URL-a

Helper tworzy adres w formacie:

```text
$CI_SERVER_URL/<repozytorium>/-/blob/<ref>/$DOCS_MD_FILE_PATH
```

Repozytorium jest wybierane w następującej kolejności:

1. `GITLAB_CI_COMPONENTS_PATH`, jeżeli jest ustawiona i niepusta.
2. `GITLAB_CI_REPOSITORY_PATH` wyznaczona przez `job-prepare`.

Ref jest wybierany w następującej kolejności:

1. `GITLAB_CI_COMPONENTS_VERSION`, jeżeli jest ustawiona i niepusta.
2. `GITLAB_CI_REPOSITORY_BRANCH` wyznaczona przez `job-prepare`.

Dzięki temu komponent może linkować do dokumentacji z tej samej wersji,
z której został dołączony, a zwykły job do dokumentacji w repozytorium
konfiguracji CI.

## Zmienne

| Zmienna | Wymagana | Pochodzenie | Opis |
|---|---|---|---|
| `DOCS_MD_FILE_PATH` | tak | komponent lub job | Ścieżka do pliku dokumentacji względem korzenia wybranego repozytorium |
| `CI_JOB_STATUS` | tak | GitLab CI | Status bieżącego joba |
| `CI_SERVER_URL` | tak | GitLab CI | Bazowy URL instancji GitLab |
| `GITLAB_CI_COMPONENTS_PATH` | nie | komponent | Repozytorium zawierające dokumentację komponentu |
| `GITLAB_CI_COMPONENTS_VERSION` | nie | komponent | Tag, gałąź lub SHA dokumentacji komponentu |
| `GITLAB_CI_REPOSITORY_PATH` | fallback | `job-prepare` | Repozytorium konfiguracji CI |
| `GITLAB_CI_REPOSITORY_BRANCH` | fallback | `job-prepare` | Ref repozytorium konfiguracji CI |
| `PROJECT_TYPE` | nie | CI/CD Variables projektu | Dla `gitlab-pipelines` włącza kontrolę istnienia pliku |

## Użycie

Helper jest automatycznie wstrzykiwany przez komponent `_after_script`:

```yaml
after_script:
  - !reference [.helpers.logger.script.sh]
  - !reference [.helpers.job-prepare.script.sh]
  - !reference [.helpers.job-docs.script.sh]
```

Job korzystający z `_after_script` powinien ustawić co najmniej:

```yaml
verify:
  extends:
    - .after_script
  variables:
    DOCS_MD_FILE_PATH: docs/verify/README.md
  script:
    - echo "Weryfikacja"
```

Komponent przechowujący dokumentację w tym repozytorium może dodatkowo
ustawić:

```yaml
variables:
  GITLAB_CI_COMPONENTS_PATH: dev.rachuna/pipelines/gitlab/components/bootstrap
  GITLAB_CI_COMPONENTS_VERSION: "1.0.0"
  DOCS_MD_FILE_PATH: templates/shellcheck/README.md
```

## Uwagi

- `job-prepare` musi zostać wykonany przed `job-docs`, ponieważ dostarcza
  zmienne fallbackowe repozytorium i refa.
- Kontrola `-f` dotyczy wyłącznie `PROJECT_TYPE=gitlab-pipelines`.
- Dla innych typów projektów dokumentacja może znajdować się w osobnym
  repozytorium i nie musi istnieć w katalogu roboczym joba.

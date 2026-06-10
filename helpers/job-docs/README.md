# Helper: job-docs

Wyświetla link do dokumentacji joba na końcu każdego wykonania — zarówno po sukcesie, jak i po błędzie.

## Co robi

1. Weryfikuje, że zmienna `DOCS_MD_FILE_PATH` jest ustawiona — jeśli nie, kończy z `critical`.
2. Dla technologii `gitlab-pipelines` dodatkowo weryfikuje, że plik pod `DOCS_MD_FILE_PATH` istnieje na dysku.
3. Buduje URL do pliku README w repozytorium komponentów (`GITLAB_CI_COMPONENTS_PATH`) lub, gdy ta zmienna nie jest ustawiona, w repozytorium CI (`GITLAB_CI_REPOSITORY_PATH`).
4. Wyświetla banner:
   - zielony — gdy `CI_JOB_STATUS == success`
   - czerwony — gdy job zakończył się błędem

## Wymagane zmienne

| Zmienna | Skąd pochodzi | Opis |
|---|---|---|
| `DOCS_MD_FILE_PATH` | `variables/main.yml` każdego joba | Ścieżka do README joba względem korzenia repozytorium CI |
| `CI_JOB_STATUS` | GitLab CI (automatyczna) | Status bieżącego joba (`success` / `failed`) |
| `CI_SERVER_URL` | GitLab CI (automatyczna) | Bazowy URL serwera GitLab |
| `GITLAB_CI_COMPONENTS_PATH` | CI/CD Variables projektu (opcjonalna) | Ścieżka projektu z dokumentacją komponentów; ma pierwszeństwo przed `GITLAB_CI_REPOSITORY_PATH` |
| `GITLAB_CI_REPOSITORY_PATH` | `job-prepare.sh` | Ścieżka projektu (np. `dev.rachuna/flows/gitlab`) |
| `GITLAB_CI_REPOSITORY_BRANCH` | `job-prepare.sh` | Gałąź repozytorium CI |
| `PROJECT_TYPE` | CI/CD Variables projektu | Typ technologii (np. `gitlab-pipelines`, `image-builder`) |

## Użycie w jobach

Helper jest wstrzykiwany w `after_script`, żeby działał również po nieudanym `script`:

```yaml
after_script:
  - !reference [.common.job-prepare.script.sh]
  - !reference [.common.logger.script.sh]  - !reference [.common.job-docs.script.sh]   # ← wyświetla link do dokumentacji
  - !reference [.common.mój-job.after_script.sh]
```

## Uwagi

- Helper **musi** być poprzedzony `job-prepare.sh` (dostarcza `GITLAB_CI_REPOSITORY_PATH` i `GITLAB_CI_REPOSITORY_BRANCH`).
- Jeżeli `GITLAB_CI_COMPONENTS_PATH` nie jest ustawiona lub ma pustą wartość, URL dokumentacji używa `GITLAB_CI_REPOSITORY_PATH`.
- Każdy job ustawia własną wartość `DOCS_MD_FILE_PATH` w swoim `variables/main.yml`.
- Sprawdzenie istnienia pliku (`-f`) jest wykonywane tylko dla `PROJECT_TYPE=gitlab-pipelines`, bo inne technologie mogą trzymać dokumentację w osobnym repozytorium.

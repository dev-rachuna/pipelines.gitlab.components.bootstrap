# Helper: gitlab-tools

Zestaw funkcji bash do interakcji z GitLab API oraz konfiguracji środowiska git/SSH w jobach CI/CD.

## Funkcje

### `gitlab_commit_amend()`

Dodaje wszystkie zmiany do stage'a (`git add -A`) i amenduje ostatni commit, a następnie wymusza push.

Obsługuje trzy scenariusze:

| Scenariusz | Działanie | `GITLAB_AMEND_RESULT` |
|---|---|---|
| Brak zmian w stage'u | Loguje informację, nie robi nic | `no_changes` |
| Zmiany cofają stan do `HEAD^` (formatowanie przywróciło oryginał) | Cofa stage (`git restore --staged .`), pomija amend | `skipped` |
| Są rzeczywiste zmiany | `git commit --amend --no-edit` + `git push --force-with-lease` | `pushed` |

**Wymagane zmienne:**

| Zmienna | Opis |
|---|---|
| `GITLAB_USER_EMAIL` | E-mail ustawiany w `git config user.email` |
| `GITLAB_USER_NAME` | Nazwa ustawiania w `git config user.name` |
| `CI_COMMIT_REF_NAME` | Gałąź docelowa pushu |

**Zmienna wyjściowa:** `GITLAB_AMEND_RESULT` — dostępna po wywołaniu funkcji.

---

### `run_new_pipeline()`

Uruchamia nowy pipeline przez GitLab API dla bieżącej gałęzi, wypisuje URL nowego pipeline'u i kończy bieżącego joba z kodem `1`.

Używana gdy job (np. autofix) zmienił kod i chce przekazać sterowanie świeżemu pipeline'owi.

**Wymagane zmienne:**

| Zmienna | Opis |
|---|---|
| `GITLAB_TOKEN` | Personal/project access token z uprawnieniem `api` |
| `CI_PROJECT_ID` | ID projektu GitLab (dostępne automatycznie w CI) |
| `CI_COMMIT_REF_NAME` | Gałąź dla nowego pipeline'u |

---

### `gitlab_ssh_key()`

Konfiguruje klucz SSH w `~/.ssh/id_rsa` i dodaje do `known_hosts` hosty:
- `gitlab.rachuna-dev.pl`
- `gitlab.com`

Używana gdy job wykonuje operacje git przez SSH (np. push do zewnętrznego repozytorium).

**Wymagana zmienna:**

| Zmienna | Opis |
|---|---|
| `GITLAB_SSH_KEY` | Zawartość klucza prywatnego SSH (secret w CI/CD Variables) |

## Użycie w jobies

Helper jest wstrzykiwany przez `before_script` każdego joba:

```yaml
before_script:
  - !reference [.helpers.job-prepare.script.sh]
  - !reference [.helpers.gitlab-tools.script.sh]   # ← funkcje dostępne od tej chwili
```

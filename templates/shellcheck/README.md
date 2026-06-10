# Job: shellcheck

Waliduje jakość skryptów bash w plikach `*.sh` oraz plikach wykonywalnych
znalezionych w repozytorium.

## Co robi

Uruchamia `shellcheck --shell=bash` na każdym pliku `*.sh` oraz każdym pliku
z ustawionym bitem wykonywalności znalezionym rekurencyjnie w repozytorium.
Katalog `.git` jest pomijany. Błąd shellcheck kończy job jako failed
(`critical`).

## Kiedy się uruchamia

| Warunek | Zachowanie |
|---------|-----------|
| CI pipeline + branch != `main` + zmiana `**/*.sh.yml` lub `**/*.sh` względem `main` | Automatycznie (`when: on_success`) |
| CD pipeline (`PIPELINE_TYPE=CD`) | Nigdy |
| Push na `main` | Nigdy |
| Brak zmian w `*.sh` / `*.sh.yml` | Nigdy |

## Zmienne wejściowe

| Zmienna | Domyślna | Wymagana | Opis |
|---------|----------|----------|------|
| `DOCS_MD_FILE_PATH` | `common/jobs/shellcheck/README.md` | ✔️ | Ścieżka do dokumentacji joba (helper `job-docs`) |

## Logika skryptu

```bash
while IFS= read -r -d '' file_sh; do
  shellcheck --shell=bash "$file_sh" || critical "❌ shellcheck wykrył błędy"
done < <(find ./ -path './.git' -prune -o -type f \( -name "*.sh" -o -perm /111 \) -print0)
```

Iteruje po plikach `*.sh` i plikach wykonywalnych przez process substitution
(`< <(find ...)`). Przy pierwszym błędzie shellcheck job kończy się jako
failed.

## Lokalne uruchomienie

```bash
# Instalacja
apt-get install shellcheck   # lub: brew install shellcheck

# Sprawdzenie jednego pliku
shellcheck --shell=bash common/helpers/logger.sh

# Sprawdzenie wszystkich (jak robi job)
while IFS= read -r -d '' file_sh; do
  shellcheck --shell=bash "$file_sh"
done < <(find ./ -path './.git' -prune -o -type f \( -name "*.sh" -o -perm /111 \) -print0)
```

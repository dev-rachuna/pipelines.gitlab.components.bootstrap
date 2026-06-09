# Helper: logger

Zestaw funkcji bash do formatowanego logowania w jobach CI/CD — nagłówki, bannery kolorowe, komunikaty krytyczne.

## Funkcje

### `banner <kolor> <tekst>`

Wyświetla wyróżniony blok z tekstem: kolorowa linia tła na górze i dole, tekst wycentrowany pośrodku.

Szerokość bloku: `term_width` (domyślnie 130 znaków).

| Kolor | Tekst | Tło linii |
|---|---|---|
| `red` | czerwony | czerwone |
| `green` | zielony | zielone |
| `blue` | niebieski | niebieskie |
| `yellow` | żółty | żółte |
| _(inne)_ | domyślny | domyślne |

---

### `h1 <tekst>`

Nagłówek pierwszego poziomu — pogrubiony tekst wycentrowany między dwiema liniami `=` w kolorze niebieskim.

```
==============================================================
                        Tytuł sekcji
==============================================================
```

---

### `h2 <tekst>`

Nagłówek drugiego poziomu — prefix `==>` w kolorze żółtym, tekst biały/pogrubiony.

```
==> Komunikat
```

---

### `h3 <tekst>`

Nagłówek trzeciego poziomu — prefix `--->` w kolorze niebieskim.

```
---> Szczegół
```

---

### `info <tekst>`

Alias dla `banner blue`. Używany do komunikatów informacyjnych.

---

### `warn <tekst>`

Alias dla `banner yellow`. Używany do ostrzeżeń.

---

### `critical <tekst>`

Alias dla `banner red`, po którym następuje `exit 1`. Kończy wykonanie joba z błędem.

## Zmienna konfiguracyjna

| Zmienna | Domyślna wartość | Opis |
|---|---|---|
| `term_width` | `130` | Szerokość linii bannera i nagłówka `h1` w znakach |

## Użycie w jobach

Helper jest wstrzykiwany po `job-prepare` w `before_script` i `after_script`:

```yaml
before_script:
  - !reference [.common.job-prepare.sh]
  - !reference [.common.logger.script.sh]   # ← funkcje dostępne od tej chwili
  - !reference [.common.logo.script.sh]
```

Przykład użycia w skrypcie joba:

```bash
h1 "Etap budowania"
h2 "Pobieranie zależności"
h3 "npm install"

info "Budowanie zakonczone sukcesem"
warn "Brak pliku .npmrc — uzywam domyslnych ustawien"
critical "Nie znaleziono pliku package.json"   # przerywa job
```

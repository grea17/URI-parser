# URI Parser — Common Lisp

Libreria in **Common Lisp** (paradigma **funzionale**) che riconosce un URI secondo una sintassi semplificata della [RFC 3986](https://datatracker.ietf.org/doc/html/rfc3986) e lo scompone nei suoi componenti: schema, userinfo, host, port, path, query e fragment. Per una panoramica dei concetti e per il confronto con la versione Prolog vedi il [README principale](../README.md).

## Avvio

```
sbcl --load uri-parse.lisp
```

oppure, da un REPL già aperto:

```lisp
(load "uri-parse.lisp")
```

## Utilizzo

La libreria si usa tramite la funzione **`URI-PARSE`**, che riceve una stringa contenente un URI e restituisce una **`URI-STRUCT`** con i componenti scomposti.

```lisp
(defparameter *u*
  (uri-parse "http://utente@disco.unimib.it:8080/didattica/corsi?anno#orario"))

(uri-scheme *u*)    ; => "http"
(uri-host *u*)      ; => "disco.unimib.it"
(uri-port *u*)      ; => 8080
(uri-path *u*)      ; => "didattica/corsi"

(uri-display *u*)
; Schema:      "http"
; Userinfo:    "utente"
; Host:        "disco.unimib.it"
; Port:        8080
; Path:        "didattica/corsi"
; Query:       "anno"
; Fragment:    "orario"
```

I componenti assenti valgono `NIL`; la porta, se non indicata, vale `80`.

### Come funziona il parsing

1. **`EXTRACT-SCHEMA`** riconosce lo schema (tutto ciò che precede il primo `:`).
2. Se lo schema è tra quelli a sintassi speciale (`mailto`, `news`, `tel`, `fax`, `zos`) viene usato **`EXTRACT-SPECIAL-URI`**; altrimenti si applica la sintassi generica.
3. Ogni componente è estratto da una funzione dedicata: `EXTRACT-AUTHORITY` (che a sua volta usa `EXTRACT-USERINFO`, `EXTRACT-HOST`, `EXTRACT-PORT`), `EXTRACT-PATH`, `EXTRACT-QUERY`, `EXTRACT-FRAGMENT`.

### Funzioni pubbliche

| Funzione                       | Descrizione                                                         |
|--------------------------------|---------------------------------------------------------------------|
| `(uri-parse stringa)`          | Restituisce una `uri-struct`; se l'argomento non è una stringa restituisce `NIL`. |
| `(uri-scheme uri)`             | Schema                                                              |
| `(uri-userinfo uri)`           | Userinfo                                                            |
| `(uri-host uri)`               | Host                                                                |
| `(uri-port uri)`               | Port                                                                |
| `(uri-path uri)`               | Path                                                                |
| `(uri-query uri)`              | Query                                                               |
| `(uri-fragment uri)`           | Fragment                                                            |
| `(uri-display uri [stream])`   | Stampa i componenti. Senza `stream` usa lo stream corrente.         |

> **Attenzione:** se a `uri-display` si passa uno stream esplicito, la funzione lo **chiude** al termine della stampa.

## Schemi a sintassi speciale

| Schema       | Sintassi                                                      |
|--------------|---------------------------------------------------------------|
| `mailto`     | `mailto:userinfo[@host]`                                      |
| `news`       | `news:host`                                                   |
| `tel`, `fax` | `tel:userinfo` / `fax:userinfo`                               |
| `zos`        | `zos://[userinfo@]host[:port]/id44[(id8)][?query][#fragment]` |

Per `zos` l'`id44` e l'eventuale `id8` (tra parentesi tonde) sono sequenze di caratteri alfanumerici; l'`id44` può contenere anche il punto.

## Errori

Un URI non valido segnala un errore con un messaggio che indica il componente coinvolto:

```lisp
(uri-parse "http://host/a<b")
; ERROR: invalid path character
```

## Caratteri ammessi

Userinfo, host e path accettano lettere, cifre, lo spazio e `- . _ ~ [ ] ! $ & ' ( ) * + , ; =` (il path anche `/`). Query e fragment accettano in più `: / ? @`. Questo rende validi URI come `http://host/pagina?q=1&lang=it`.

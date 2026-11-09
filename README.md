# URI Parser

Libreria per il riconoscimento e la scomposizione di **URI** secondo una sintassi semplificata della [RFC 3986](https://datatracker.ietf.org/doc/html/rfc3986), realizzata due volte: in **Common Lisp** (paradigma funzionale) e in **Prolog** (paradigma logico).

Progetto di Linguaggi di Programmazione.

## Cos'è un URI

Un **URI** (*Uniform Resource Identifier*) è una stringa che identifica una risorsa in modo univoco. Il componente primario è lo **schema**, che stabilisce la sintassi da seguire: schemi diversi identificano risorse di natura diversa e usano componenti diversi.

Esempio di scomposizione:

```
http://utente@disco.unimib.it:8080/didattica/corsi?anno#orario
```

| Componente | Valore             |
|------------|--------------------|
| Schema     | `http`             |
| Userinfo   | `utente`           |
| Host       | `disco.unimib.it`  |
| Port       | `8080`             |
| Path       | `didattica/corsi`  |
| Query      | `anno`             |
| Fragment   | `orario`           |

Se la porta non è indicata viene usato il valore di default `80`. Il `/` iniziale del path non viene incluso nel componente.

## Struttura della repository

```
URI-parser/
├── README.md              <- questo file
├── Lisp/
│   ├── README.md          <- guida alla versione Common Lisp
│   └── uri-parse.lisp
└── Prolog/
    ├── README.md          <- guida alla versione Prolog
    └── urilib-parse.pl
```

## Le due implementazioni a confronto

|                              | Common Lisp                          | Prolog                                                        |
|------------------------------|--------------------------------------|---------------------------------------------------------------|
| File                         | `Lisp/uri-parse.lisp`                | `Prolog/urilib-parse.pl`                                      |
| Entry point                  | `(uri-parse stringa)`                | `urilib_parse(+Stringa, -URI)`                                |
| Risultato                    | struct `uri-struct`                  | termine `uri(Schema, Userinfo, Host, Port, Path, Query, Fragment)` |
| Componente assente           | `NIL`                                | `[]`                                                          |
| Stampa                       | `(uri-display uri [stream])`         | `urilib_display(URI)` / `urilib_display(URI, Stream)`         |
| Input non valido             | segnala un errore                    | il predicato fallisce                                         |

## Avvio rapido

### Common Lisp (SBCL)

```
sbcl --load Lisp/uri-parse.lisp
```

```lisp
* (uri-display (uri-parse "http://utente@disco.unimib.it:8080/didattica/corsi?anno#orario"))
Schema:      "http"
Userinfo:    "utente"
Host:        "disco.unimib.it"
Port:        8080
Path:        "didattica/corsi"
Query:       "anno"
Fragment:    "orario"
```

### Prolog (SWI-Prolog)

```
swipl -s Prolog/urilib-parse.pl
```

```prolog
?- urilib_parse("http://utente@disco.unimib.it:8080/didattica/corsi?anno#orario", URI).
URI = uri(http, utente, 'disco.unimib.it', 8080, 'didattica/corsi', anno, orario).
```

L'URI va passato come **stringa** (tra doppi apici).

## Schemi supportati

Oltre alla sintassi generica, sono previsti cinque schemi con sintassi speciale.

| Schema          | Sintassi                                                 |
|-----------------|----------------------------------------------------------|
| generico        | `schema:[//[userinfo@]host[:port]][/path][?query][#fragment]` |
| `mailto`        | `mailto:userinfo[@host]`                                 |
| `news`          | `news:host`                                              |
| `tel`, `fax`    | `tel:userinfo` / `fax:userinfo`                          |
| `zos`           | `zos://[userinfo@]host[:port]/id44[(id8)][?query][#fragment]` |

Uno schema non seguito da altro (`ftp:`) è un URI valido.

## Note sui caratteri ammessi

Le due versioni non accettano lo stesso insieme di caratteri. La versione Lisp è più permissiva e accetta, per esempio, `=` e `&` in query e fragment (`?q=1` viene riconosciuto); la versione Prolog accetta solo lettere, cifre e `_ . ( )` in ogni componente, quindi un URI come `http://host/a?q=1` non viene riconosciuto.

## Riferimenti

- [RFC 3986 — Uniform Resource Identifier (URI): Generic Syntax](https://datatracker.ietf.org/doc/html/rfc3986)

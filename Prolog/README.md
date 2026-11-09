# URI Parser — Prolog

Libreria in **Prolog** (paradigma **logico**) che riconosce un URI secondo una sintassi semplificata della [RFC 3986](https://datatracker.ietf.org/doc/html/rfc3986) e lo scompone nei suoi componenti: schema, userinfo, host, port, path, query e fragment. Per una panoramica dei concetti e per il confronto con la versione Lisp vedi il [README principale](../README.md).

## Avvio

```
swipl -s urilib-parse.pl
```

oppure, da un toplevel già aperto:

```prolog
?- consult('urilib-parse.pl').
```

## Utilizzo

La libreria si usa tramite il predicato **`urilib_parse/2`**, che riceve una **stringa** (tra doppi apici) contenente un URI e unifica il secondo argomento con un termine `uri/7`:

```
uri(Schema, Userinfo, Host, Port, Path, Query, Fragment)
```

```prolog
?- urilib_parse("http://utente@disco.unimib.it:8080/didattica/corsi?anno#orario", URI).
URI = uri(http, utente, 'disco.unimib.it', 8080, 'didattica/corsi', anno, orario).

?- urilib_parse("http://utente@disco.unimib.it:8080/didattica/corsi?anno#orario", URI),
   urilib_display(URI).
Schema = http
Userinfo = utente
Host = disco.unimib.it
Port = 8080
Path = didattica/corsi
Query = anno
Fragment = orario
```

I componenti assenti valgono `[]`; la porta, se non indicata, vale `80`. Se l'URI non è valido, `urilib_parse/2` **fallisce**.

### Come funziona il parsing

1. **`schema/3`** riconosce lo schema (tutto ciò che precede il primo `:`).
2. **`parse_urilib_with_schema/3`** sceglie la sintassi in base allo schema (generico oppure `mailto`, `news`, `tel`, `fax`, `zos`) e richiama un predicato per ciascun componente: `authority/5` (che usa `userinfo/3`, `ip/3`, `host/3`, `port/3`), `path/3`, `query/3`, `fragment/3`.

### Predicati pubblici

| Predicato                         | Descrizione                                                              |
|-----------------------------------|--------------------------------------------------------------------------|
| `urilib_parse(+URIString, -URI)`  | Scompone l'URI in un termine `uri/7`.                                    |
| `urilib_display(+URI)`            | Stampa i componenti sullo stream di output corrente.                     |
| `urilib_display(+URI, +Stream)`   | Stampa i componenti su `Stream`.                                         |

> **Attenzione:** `urilib_display/2` **chiude** lo stream al termine della stampa.

## Schemi a sintassi speciale

| Schema       | Sintassi                                                      | Risultato                                   |
|--------------|---------------------------------------------------------------|---------------------------------------------|
| `mailto`     | `mailto:userinfo[@host]`                                      | `uri(mailto, 'mario.rossi', 'unimib.it', [], [], [], [])` |
| `news`       | `news:host`                                                   | `uri(news, [], 'news.unimib.it', [], [], [], [])` |
| `tel`, `fax` | `tel:userinfo` / `fax:userinfo`                               | `uri(tel, '0264484111', [], [], [], [], [])` |
| `zos`        | `zos://[userinfo@]host[:port]/id44[(id8)][?query][#fragment]` | `uri(zos, utente, 'host.it', 8080, 'dati.set(membro)', [], [])` |

Per `zos`:

- l'`id44` deve iniziare con una lettera, è lungo al massimo 44 caratteri, può contenere lettere, cifre e punti e non può terminare con un punto;
- l'`id8`, se presente, è tra parentesi tonde, è lungo al massimo 8 caratteri ed è alfanumerico.

## Host

L'host può essere un nome (che deve iniziare con una lettera) oppure un **indirizzo IPv4** nel formato `a.b.c.d` con ogni ottetto tra 0 e 255:

```prolog
?- urilib_parse("http://192.168.1.1:80/x", URI).
URI = uri(http, [], '192.168.1.1', 80, x, [], []).
```

## Caratteri ammessi

In ogni componente sono accettati solo lettere, cifre e i simboli `_ . ( )`. Un URI che contiene altri caratteri (per esempio `=` o uno spazio) non viene riconosciuto e `urilib_parse/2` fallisce.

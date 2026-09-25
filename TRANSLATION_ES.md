# Spanish (Latin American) Localization for Pokemon Heart & Soul

**Status: complete — 99.6% of all translatable strings (20,391 / 20,464).**
Built entirely from the project source. No ROM is distributed with this translation.

---

## What this is

A full **Latin American Spanish (es-419)** localization of Pokemon Heart & Soul,
made on top of the project's own source tree. It is not a patch over the compiled
ROM: every string lives in the same files the game is built from, so the translation
is built exactly like the game itself.

**This is a translation contribution, nothing more.** Heart & Soul is the work of its
authors — the story, the code, the art, the systems. The Spanish text is offered back
to the project. No authorship of the hack is claimed anywhere.

```
755 files changed   (706 .inc · 30 .c · 18 .h)
46,861 insertions / 46,847 deletions
```

## Coverage

| Area | Strings | Coverage |
|---|---|---|
| Johto / Kanto dialogue and scripted events | 8,616 / 8,616 | **100%** |
| Items, moves, abilities, species, berries | 2,323 / 2,328 | 99.8% |
| Other (mail, TV, PokéNav, secret bases, ...) | 3,350 / 3,364 | 99.6% |
| Battle messages | 581 / 585 | 99.3% |
| Menus, options menu, system text | 5,521 / 5,571 | 99.1% |
| **Overall** | **20,391 / 20,464** | **99.6%** |

The remaining 73 strings are documented under *Known limitations* below, with the
reason for each.

## Terminology

Names are taken from the **officially localized games**, not translated from English:

- Pokemon, moves, abilities, items and berries use their official Spanish names
  (e.g. `Sitrus Berry` -> `Baya Citron`, `Cheri Berry` -> `Baya Zreza`).
- The official name is used whole whenever the engine's field allows it. Where it does
  not fit, only the *generic* part is abbreviated and the distinctive name is kept
  (`Max Repel` -> `Max. Repel.`, never a synonym).
- Text style follows the games: `tu` form, neutral Latin American Spanish, accents
  and inverted punctuation throughout.

## Engine adaptations (worth reading — these are latent bugs the translation exposed)

Translating into Spanish is longer than English, which surfaced several fixed-width
fields where a longer string silently loses its terminator (`0xFF`) and the engine
then reads into the next entry — causing corrupted text, names running together, or
in the worst case a crash. All of these were found, fixed and verified against the
compiled ROM:

| Location | Field | Why it broke | Resolution |
|---|---|---|---|
| `struct Item.name` (`ITEM_NAME_LENGTH` = 14) | 13 usable bytes | No room for the terminator; `shop.c` copies item names into a 16-byte cell with an unbounded `StringCopy` -> **crash when buying** | All 786 item names brought under 13 bytes (generic part abbreviated, proper name kept) |
| `struct Berry.name` (`BERRY_NAME_LENGTH` = 6) | 6 bytes | Spanish names with the `Baya ` prefix (11 bytes) were cut mid-character, leaving a control byte inside the copied text -> **the whole berry message corrupted and terminated early** | Distinctive official word (`ZREZA`, `ARANJA`, `CITRO`, ...) |
| `gTypeNames` (`TYPE_NAME_LENGTH` = 6) | 7 bytes | `VOLADOR`, `FANTASMA`, `SINIESTRO` truncated | Field widened to 10 (`global.h`), official names kept complete |
| `gTrainerClassNames[][13]` | 13 bytes | 47 compiler warnings (`excess elements in array initializer`); names copied until `EOS` read into the next entry -> class names run together in battle | Field widened to 19 (definition **and** the `extern` in `include/data.h`); names abbreviated to the 12-byte maximum English ever used |
| `sATypeMove_Table[][17]` | copied into `gBattleTextBuff2` = `TEXT_BUFF_ARRAY_COUNT` = 16 bytes | Spanish entries were 17-23 bytes -> 1-7 bytes written past the buffer | Entries shortened to `mov. <TYPE>` (<=14 bytes) |

These are documented because they are not translation trivia: the first two are
reachable in the English build too if any name is ever lengthened, and the
`shop.c` cell is a genuine unbounded copy.

## Build note

Building the current source with a recent devkitARM (GCC 15) needs `-std=gnu17` in
both `CFLAGS` and `CPPFLAGS`; the branch carries that one-line Makefile change.
It is unrelated to the translation and can be split into its own commit/PR if you
prefer to keep the translation diff purely textual.

## Known limitations

1. **21 strings deliberately left in English.** They contain `{STR_VAR_n}` or `{PKMN}`
   placeholders whose *order* matters. The translation produced reordered placeholders,
   which would print the wrong name in the sentence — worse than leaving English.
2. **~52 technical strings untranslated** (debug text, link/placeholder-only entries,
   strings with no user-visible meaning).
3. **Long lines in a few menu entries** can slightly overflow their box. Most dialogue
   was re-wrapped to fit the 38-character box; a handful of menu labels are still wider
   than the space available. Cosmetic, no crashes.

## How to build / play

Requires a **Pokemon Emerald Version (U)** ROM of your own, and the devkitARM toolchain.

```
# build from source
git clone <this branch>
make tools -j$(nproc) && make modern -j$(nproc)     # -> pokemonHnS.gba
```

Players: apply the official Heart & Soul `.ups` patch to an Emerald (U) ROM with
[Rom Patcher JS](https://www.marcrobledo.com/RomPatcher.js/legacy/), then apply this
translation's patch on top of the result. **The ROM itself has never been and will
never be distributed here.**

## Verification method

Every claim above was checked against the compiled ROM, not against the source:
symbol tables are read out of the binary with `nm -S` and every entry is verified to
carry its `0xFF` terminator inside its own field. A small audit script ships with the
project and is run after every build; its current output is `RESULTADO: limpio`
(0 compiler warnings about translated text, 6/6 text tables clean).

---

# Resumen en espanol

Localizacion completa al **espanol latinoamericano** de Heart & Soul, hecha sobre el
codigo fuente del proyecto: **20.391 de 20.464 cadenas (99,6%)**, sin distribuir nunca
un ROM. Los nombres (Pokemon, movimientos, objetos, bayas) usan los oficiales de los
juegos, no traducciones del ingles. El texto es un aporte al proyecto original: el
hack es de sus autores.

El trabajo destapo y corrigio cinco campos de ancho fijo donde el texto en espanol
perdia el terminador y el juego leia memoria de al lado (nombre de objeto, nombre de
baya, tipos, clases de entrenador y una tabla de mensajes de batalla). Todos estan
verificados contra el binario compilado, con un auditor que corre despues de cada
compilacion.

Quedan 73 cadenas: 21 dejadas en ingles a proposito (sus codigos reordenados pondrian
el nombre equivocado en la frase) y ~52 tecnicas.

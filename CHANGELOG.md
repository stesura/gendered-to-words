# Changelog

## 0.2.1 (2026-09-23)

No changes to the library's output.

- The repository moved to the Stesura organisation: `github.com/stesura/gendered-to-words`.
  `homepage`, `bugs` and `repository` in `package.json` now point there.
- Releases are published from GitHub Actions through npm trusted publishing (OIDC), with
  provenance, instead of a long-lived npm token.
- **Requires Node 22 or later** (`engines.node` was `>=20.0.0`). Node 20 is out of support;
  CI now tests Node 22 and 24.
- Copyright holder is now innov-ars SRL. The original to-words copyright is kept.

## 0.2.0 (2026-08-04)

Correctness release. **Output changes for many locales** — see "Breaking" below.

### Fixed: compound ordinals

`toOrdinal()` previously looked up a table and, when nothing matched, returned the
**cardinal unchanged**. Italian 21st came out `Ventuno`, French 42nd `Quarante-Deux`,
Spanish 42nd `Cuarenta Y Segundo`. The fallback is gone: unresolvable ordinals now
throw instead of returning silently wrong output.

Ordinals are now derived rather than enumerated, via a new `ordinalDerivation` in the
locale config with three strategies:

- `last` — inflect the final token (en, de, nl, fr, tr)
- `whole` — inflect the fully written-out numeral (it `Centounesimo`, ca `Quaranta-Dosè`)
- `components` — inflect every additive component (es `Cuadragésimo Segundo`, pt, is, ar)

Also fixed: locales storing hundreds as one atomic token (el-GR, lv-LV, tr-TR) rendered
**200th as 100th**; Chinese had no ordinal for 11 and produced `二十 第一` for 21 (now
`ordinalPrefix` gives `第十一` / `第二十一`).

### Fixed: written form

- English hyphenates compound tens — `Twenty-One`, `Forty-Second` (was `Twenty One`)
- German, Dutch and Italian numerals are written solid, with scale nouns from "million"
  up kept separate: `Einhunderteins` (was `Hundert Eins`), `Honderdzevenendertig`,
  `Duecento`, `Zwei Millionen Siebenhunderteinundvierzigtausendvierunddreißig`
- The sign and decimal-point words are no longer glued into the numeral
  (`Null Komma Null Vier`, not `Nullkommanullvier`)

### Fixed: cardinals

- German: `Eine Million` (was `Eins Million`), `Zwei Millionen` (was `Zwei Million`)
- Italian: `Milleuno` (was `Mila Uno`), `Duecento` (was `Due Cento`), and `Due Milioni` —
  a misdeclared `dual` plural form was dropping the multiplier, rendering 2,000,000 as
  bare `Milioni`
- Dutch: `Honderd`, not `Een Honderd`

### Added

- **`en-GB`** locale: `One Hundred And One`, `One Thousand Two Hundred And Thirty-Four`.
  The bare code `en` still resolves to `en-US`.
- **Feminine variants for it, es, pt-PT, pt-BR, ca** — `GENDER_VARIANTS` previously held
  French only. Covers ordinals throughout and cardinal agreement where the language has
  it (`Doscientas`, `Duas`, `Dues`, `Veintiuna`).
- `createLocaleVariant` accepts `transformOrdinalWords` / `transformOrdinalDerivation`,
  so a gender variant is one rule plus its irregulars rather than a duplicated table.
- `createLocaleVariant` accepts `agreementOverrides` for number words that agree **only
  in final position**. A numeral in front of a scale noun agrees with that noun, and
  "million" and up are masculine in every language with a variant here — so a feminine
  variant must produce "doscientas páginas" but "doscientos millones". Each entry is
  paired with the base locale's own value automatically, so the two can't drift apart.

### Fixed: locale data inconsistencies found by an all-locale output sweep

- **tr-TR** stored cardinals lowercase but ordinals Title Cased, so 123rd came out
  `yüz yirmi Üçüncü`. Now consistently Title Cased, like every other locale;
  `lowercase: true` still applies Turkish casing rules correctly (`ikinci`).
- **ca-ES** built the plural of "milió" by appending a mark, giving `Milións`; the accent
  drops, so it is now `Milions`.
- **lv-LV** had one Title Cased entry among otherwise lowercase data, so 1100 read
  `viens tūkstotis Simtu`.

### Fixed: missing conjunctions

Four locales joined their parts with nothing at all. Each language puts the linking word
in a different place, so the fix is per-locale rather than a shared default:

- **ro-RO** links tens to units only: `Douăzeci Și Unu`, and `O Sută Douăzeci Și Trei`
  with no conjunction after the hundreds (was `Douăzeci Unu`).
- **sq-AL** links every part: `Njëqind E Njëzet E Tre` (was `Njëqind Njëzet Tre`).
- **nb-NO** links the hundreds but not the larger scales: `Sju Hundre Og Åttini`, and
  `Tusen To Hundre Og Trettifire` with no "og" after "tusen".
- **sw-TZ** links every part: `Ishirini Na Moja`. Swahili also names the scale _before_
  its multiplier, so the hundreds are now atomic entries in the right order —
  `Mia Moja Na Ishirini Na Tatu`, not `Moja Mia Ishirini Tatu`. Correct through 1999;
  see the limitations below.

### Fixed: unpluralised scale nouns

- **sv-SE**: `Två Miljoner` (was `Två Miljon`). The scale nouns are also common gender,
  so 1,000,000 is `En Miljon`, not the neuter `Ett Miljon`.
- **el-GR**: `Δύο Εκατομμύρια` (was `Δύο Εκατομμύριο`).

### Fixed: numerals written with stray spaces

Four more locales write their numerals solid but were being space-joined. All four are
wrong across almost the whole practical range, not just at large values:

- **zh-CN / zh-TW**: `一百二十三` (was `百 二十 三`). Chinese numerals never contain spaces,
  and 百/千/万/亿 take their 一 — though 十 does not, so 11 stays `十一`.
- **ja-JP**: `百二十三` (was `百 二十 三`). Same spacing rule, but standard Japanese omits
  the 一 before 百/千, so its data is otherwise unchanged.
- **fi-FI**: `Satakaksikymmentäkolme` (was `Sata Kaksikymmentä Kolme`). A numeral above
  one also takes the partitive: `Kaksituhatta`, `Kaksi Miljoonaa`.
- **hu-HU**: `Százhuszonhárom` (was `Száz Huszonhárom`). Hungarian also distinguishes
  `kettő` standing alone from `két` before a noun, so 2000 is `Kétezer`, not `Kettőezer`.

### Build

Replaces `oxfmt` with `prettier` (the inherited shared config specified a style this fork
does not use, so `npm run format` had never passed) and pins line endings to LF via
`.gitattributes`, without which the check cannot pass on a Windows clone. `format` now
runs in CI.

`@mastermunj/oxc-config` is dropped and its oxlint rules inlined into `.oxlintrc.json` —
identical rule set, verified by rule count. The package listed `oxfmt` as a peer
dependency, so removing oxfmt as a direct dependency left the lockfile inconsistent with
the tree npm builds, and `npm ci` failed in CI while every local check passed.

### Fixed: reported by a downstream consumer

- **`ToWordsCore` ignored per-call gender.** Variant resolution lived only in the
  `ToWords` subclass, so the tree-shaken path — the one where gender matters most,
  since it is the path that avoids the 40-locale registry — could not reach it.
  `setLocale` now also accepts a map: `setLocale({ masculine: ItIt, feminine: ItItF })`.
  A single class still works and ignores gender.
- **`./package.json` was not exported.** The `./*` wildcard swallowed it into the locale
  subpath, so `require("gendered-to-words/package.json")` 404'd for tooling that reads it.
  (`publint` does not check this.)
- **`resolveLocaleCode` is now exported**, so callers holding free-form input can test a
  code instead of wrapping every call in a try/catch.
- **A bare `fr` resolved to `fr-BE`**, whose septante/nonante forms are a minority
  variant — the prefix fallback took the first alphabetical match. Regional defaults are
  now ordered deliberately: `en`→`en-US`, `fr`→`fr-FR`, `pt`→`pt-BR`, `zh`→`zh-CN`.

### Fixed: gender leaking into scale multipliers

`fr-FR`/`fr-BE` feminine rendered 1,000,000 as `Une Million` since 0.1.0; the same class
of error affected the new it/es/pt/ca variants during development (`Una Milione`,
`Duas Milhões`, `Vinte E Uma Milhões`). Feminine forms now apply only in final position.
A sweep asserts the invariant across every gendered locale and every scale.

### Breaking

Output changes for **en-US** (hyphenation), **de-DE**, **nl-NL**, **it-IT**
(concatenation and cardinal fixes), **fr-FR**, **fr-BE**, **es-ES**, **pt-PT**, **pt-BR**,
**ca-ES**, **is-IS**, **ar-SA**, **el-GR**, **lv-LV**, **tr-TR**, **zh-CN**, **zh-TW**
(ordinals). `toOrdinal()` now throws where it previously returned a cardinal.

The protected `ToWordsCore#convertNumber` is replaced by `convertNumberSegments`, which
returns independently-written segments (sign, integer, point, fraction) so concatenating
locales don't glue across them. Removed rather than kept as a wrapper so that a subclass
overriding it fails to compile instead of silently no longer affecting `convert()`.
Only relevant if you subclass `ToWordsCore`.

French `toOrdinal(1)` is still `Premier`, but `ordinalWordsMapping[1]` is now `Unième`
(the compound form) with `Premier` moved to `ordinalExactWordsMapping` — relevant only if
you build a locale variant on top of `fr-FR`.

### Known limitations

Ordinals are verified exhaustively for 0–1000 in every locale, plus round scale values.
Beyond that:

- Ordinals whose value spans a million or more are derived structurally and may not read
  idiomatically. Spelling out ordinals that large is vanishingly rare; cardinals in the
  same range are correct.
- Spanish/Portuguese ordinal multiples of a thousand render analytically (`Dos Milésimo`)
  rather than as the solid `dosmilésimo`.
- Feminine agreement stops at masculine scale nouns, which is right for "million" and
  above (`Dois Milhões`, never `Duas Milhões`) but also suppresses it before Portuguese
  and Catalan `mil`, where the multiplier _should_ agree: 2000 feminine renders
  `Dois Mil` where `Duas Mil` is wanted. Distinguishing the two needs per-scale
  agreement data that the locale config does not yet carry.
- Arabic and Icelandic compound ordinals are a large improvement on the previous cardinal
  fallback but deserve a native-speaker review: Arabic 123rd renders as
  `الثالث و العشرون و المائة`, where idiomatic usage would be `الثالث والعشرون بعد المائة`.
- Chinese cardinal orthography is unchanged: numerals are still space-separated and omit
  the leading 一 in 一百. Only the ordinal marking was fixed.
- Hungarian inserts a hyphen at the scale boundary above two thousand
  (`kétezer-háromszáznegyvenöt`); the converter writes it solid throughout, so values
  below 2000 are correct and larger ones lack the hyphen.
- Swahili names the scale word before its multiplier, which the converter can only
  express through atomic entries. Those now cover the hundreds, so 0–1999 is correct,
  but 2000 still reads `Mbili Elfu` rather than `Elfu Mbili`.

### Known pre-existing issues, not addressed here

Found while sweeping every locale over 0–21, 123/456/789 and 123,456,789. All predate
this release and are untouched by it — recorded so they are not mistaken for regressions:

- **zh-CN / zh-TW / ja-JP** still decompose values above 9,999 against redundant myriad
  scales, so 123,456,789 renders as `一亿二千万三百万四十万五万六千七百八十九` instead of
  `一亿二千三百四十五万六千七百八十九`. (ko-KR is correct, and is the model for a fix.)
  Everything below 10,000 is now right.
- **Slavic paucal selection** picks the plural where the last digit should decide:
  ru 123,000,000 gives `Миллионов` where `Миллиона` is right. Affects ru, uk, pl, cs, sk,
  sl, lt. Each language has its own rule; this needs a dedicated pass.
- **he-IL** prefixes a redundant multiplier once the hundreds carry a remainder: 100 and
  200 are correct (`מאה`, `מאתיים`) but 123 gives `אחת מאה ו עשרים ו שלוש` and 456 gives
  `ארבע מאה …` instead of `מאה …` / `ארבע מאות …`.
- **lv-LV** stores its words lowercase while the other 40 locales are Title Cased. It is
  internally consistent, so it is left as-is, but it is inconsistent across the package.

## 0.1.0 (2026-08-04)

Initial release, forked from [to-words](https://github.com/mastermunj/to-words) v5.3.0.

- Removed currency conversion, the CLI, and most region-duplicate locales to minimize bundle size
- Added gendered locale variants (French feminine) resolvable via the `gender` option in `toWords()` / `toOrdinal()`
- Added `toOrdinalIndicator()` — digit + ordinal indicator formatting (21st, 1er/1re, 1º/1ª, 1., 1:a, 第1) with gender support, structured parts output, and a typographic `superscript` convention flag; category selection via `Intl.PluralRules`
- Locale data now carries its canonical BCP 47 code (`LocaleConfig.localeCode`) so plural-category selection and lowercasing never depend on the ambient runtime locale
- Kept: cardinal and ordinal conversion, BigInt and string inputs, locale auto-detection, per-locale tree-shakeable entry points

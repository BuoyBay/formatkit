# FormatKit 0.1.0

`Import formatkit` (dependency: moneykit 0.1.0). `NumericCodec(New FormatProfile())`
validates and copies the profile; `Profile()` and `FormatProfile.Clone()` return
independent copies. Mutating a configuration never changes an existing codec.
`Validate()` throws InvalidArgumentException. No settings or global locale changes.

Profile fields: Language, NumberLocale, ResolvedNumberLocale, DateLocale, CurrencyCode, DecimalSeparator,
GroupingSeparator, PrimaryGroup, SecondaryGroup, Grouping, Accounting, MinFraction,
MaxFraction, Rounding (MoneyRoundingMode), CurrencyDigitsOverride (-1 = metadata),
CurrencySymbolOverride, CurrencySuffix, CurrencySpace, DatePattern, TimePattern,
TimeEditPattern, TimeZoneIdentity, Provenance.
Language/date locale/timezone/patterns are identities for parent glue, not resolved
or interpreted here. `Invariant()` and the default constructor provide explicit invariant conventions without
constructing any OS Locale. `NumberLocale` records the request;
`ResolvedNumberLocale` records the host identity or invariant identity. With explicit
conventions these identities are informational. `ResolveSystem(primary = 3, secondary = 3)`
resolves host numeric preferences; on Linux it honors LC_ALL, LC_NUMERIC, LANG in
that order and strips encoding/modifier suffixes before constructing Locale.
Unknown/uninstalled identities throw rather than silently falling back. The request
remains `system`, resolved identity and environment source remain separately visible.
This does not modify the process locale. No named-timezone resolution is performed.
`ResolveHost(locale, primary = 3, secondary = 3)` copies host number separators
only; group sizes are caller supplied because Locale exposes no grouping sizes.

| NumericCodec API | Contract |
|---|---|
| IntegerInvariant / IntegerEdit / IntegerDisplay(Integer) | Exact full signed Int64; edit has no groups. |
| MoneyInvariant / MoneyEdit / MoneyDisplay(Currency) | Invariant/edit preserve all four decimals; display uses Money.Format rounding and code metadata/override. |
| NumberInvariant / NumberEdit / NumberDisplay(Double) | Finite only; invariant/edit use a short decimal that round-trips the stored Double, expanded without exponent; display rounds that representation. |
| PercentRatioEdit / PercentRatioDisplay(Double) | Ratio 0.075 means 7.5%; decimal shift before rendering avoids intermediate multiplication overflow. |
| PercentPointsEdit / PercentPointsDisplay(Double) | Points 7.5 means 7.5%. |
| AnalyzeInteger / AnalyzeMoney / AnalyzeNumber / AnalyzePercentRatio / AnalyzePercentPoints(String) | FormatAnalysis with State (Empty/Incomplete/Valid/Invalid), Code, Position, Canonical and typed IntegerValue/MoneyValue/NumberValue. Only read typed values when Valid. |

All localized analyses consume the entire input. No trimming, exponent, alternate
symbols, non-ASCII digits, leading plus, or trailing sign. Leading minus or profile
accounting parentheses accepted; money requires the configured code/symbol affix,
prefix by default or suffix with CurrencySuffix, and one ASCII space when
CurrencySpace is true. Percent requires `%` suffix. Edit outputs
use those same affixes. Position is zero-based Unicode codepoint, including original
affixes; end failures point one past the last character. Empty means exactly empty
input. Incomplete covers sign/affix-only, terminal decimal/group separator or missing
closing accounting/percent affix; malformed content remains Invalid. Codes:
empty, incomplete, syntax, grouping, precision, overflow, underflow, length. Limit 1024
codepoints. Ungrouped input accepted; supplied groups must match primary size at
right, secondary in middle and 1..secondary at left. A grouped whole number cannot
start with zero; ungrouped leading zeros are accepted and canonicalized. Separators
match exactly, including case. Each separator is exactly one
codepoint; Unicode spaces including NBSP/narrow NBSP work only when configured.

Integer and money parsers accumulate checked negative magnitudes, including Int64
minimum, without Double or Currency.Parse. Money rejects >4 decimals (even zeros),
never quantizes input. Number/percent allow up to 340 decimals for Double edit
round-trip, reject overflow and nonzero underflow to zero. Negative zero normalizes
to positive zero throughout. Display supports 0..15 fractions for Double, 0..4 for
money, all five MoneyRoundingMode values. Money display min=max currency digits;
number min/max use profile. Integer display ignores fraction policy. Rounding a
money amount past storage bounds throws OverflowException via MoneyKit. Double
display is decimal rounding of the short round-trip text, not an arbitrary
precision representation of the binary rational. Invariant output has no affixes,
groups or accounting; percent invariant uses NumberInvariant on stored ratio/points.

Built-in bounded metadata: USD/EUR/GBP/CHF/CAD/AUD = 2, JPY = 0, KWD/BHD = 3,
CLF = 4. No worldwide catalog, symbol inference, exchange conversion or cash-rounding
rules. Unknown uppercase three-letter codes require CurrencyDigitsOverride 0..4.
Symbol override must contain no digit, sign, parentheses, percent, separator or
whitespace and is limited to 16 codepoints; empty means code. Code stays independent of number locale.

Untouched fields must retain their original typed value; do not parse display text
on focus/blur or commit a display-rounded value. Edit APIs retain storage precision.
Parent glue owns dirty-state tracking, date/time behavior and preference persistence.

Maintainers run the private validation suite before release. See the repository README for public build instructions.

The renderer searches increasing scientific-mask precision and requires exact
Double parse-back, trimming only redundant decimal zeros. This adapts to both
Buoy 0.72's miscounted exponent placeholders and corrected counting; no fixed
incorrect placeholder count is required. Ordinary edit text is `0.1` and `7.5%`,
while difficult values retain enough digits to round-trip. The independent oracle
searches 1..17 significant digits using Python's binary64 formatter. The retained
scientific probe records upstream behavior without requiring the old defect.

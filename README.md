# Parse and format localized numbers with explicit profiles

`Import formatkit` (dependency: moneykit 0.1.0). `NumericCodec(New FormatProfile())`
validates and copies the profile; `Profile()` and `FormatProfile.Clone()` return
independent copies. Mutating a configuration never changes an existing codec.
`Validate()` throws InvalidArgumentException. No settings or global locale changes.

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

## Install

From a Buoy consumer project with BPM installed:

```sh
bpm add formatkit --git https://github.com/BuoyBay/formatkit.git --rev refs/heads/main
```

BPM records the resolved commit in its lockfile. Keep that lockfile with the
consumer project for reproducible installs.
BPM also resolves `moneykit`.

## API entrypoints

- `Import formatkit`
- `NumericCodec(New FormatProfile())`
- `Profile()`
- `FormatProfile.Clone()`
- `Validate()`

## Build from source

With the native `build-packages` driver from [devtools](https://github.com/BuoyBay/devtools)
on your `PATH`, run from this checkout:

```sh
build-packages --root "$PWD" --compile-consumer formatkit
```

The driver checks source targets and compiles an import consumer. See the
[API guide](docs/API.md) for behavior, limits and any host prerequisites.

## Local Buoy setup

On your own machine, install Buoy and run:

```sh
buoy ai init --both
```


## Contributing

[Contributing](CONTRIBUTING.md) · [License](LICENSE)

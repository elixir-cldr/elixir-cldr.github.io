---
layout: post
categories: [cldr, i18n, l10n, locale, language_tag]
title: User Preferences From a Language Tag
---

Language tags define a standard way to express a users preferences but how to we extract those preferences in a cohesive fashion in `ex_cldr`?

### What a Language Tag can tell us?

Posit the language tag `en`. We know from the [previous post](https://elixir-cldr.github.io/2020-11-07-exploring-language-tags/) that the user is telling us that they prefer information presented in the english language.

It may suprise you to learn that from this simple language tag we also know a lot more!  We know, by implication, at least the following:

* The territory<sup>1</sup> preferred by the user
* The preferred currency
* The preferred calendar
* The preferred number system (system of digits)

### The users preferred territory

Certain localisation functions are dependent on understanding with which territory a user affiliates themselves. One example is units of measurement where the US tends to use units derived from the [Imperial System of Measurement](https://en.wikipedia.org/wiki/Imperial_units) whereas most of the world uses the [Metric system](https://en.wikipedia.org/wiki/Metric_system). Therefore it is helpful to know the users territory in order to present units of measure in an appropriate manner.

In [ex_cldr](https://hex.pm/packages/ex_cldr) we can identify the user's territory with `Cldr.Locale.territory_from_locale/1`.  For example:

```elixir
iex> Cldr.Locale.territory_from_locale "en"
:US

iex> Cldr.Locale.territory_from_locale "en-AU"
:AU

iex> Cldr.Locale.territory_from_locale "en-GB"
:GB

iex> Cldr.Locale.territory_from_locale "ja"
:JP
```

You may be wondering how we could have derived that the territory for the language tag `en` happens to be `:US`.  Since at current count 105 territories have english defined as one active language a decision has to be made.

`CLDR`'s policy is that when a language tag is comprised solely of a language identifier, the territory with the largest population is defined to be the default. This means that `en` is considered to the same as `en-US` since the US has the largest english-speaking population worldwide.

In a similar manner, `pt` is derived to be the same as `pt-BR` since Brazil is the largest population of Portugese speakers worldwide. Not always the expectation of developers or users!

As a result of this policy, there is no `pt-BR` or `en-US` locale data defined in CLDR although a language tag of this form is most definitely accepted and resolved correctly.

In some cases it is desirable to override the territory to be used for some kinds of localisation and in this case the [regional override](https://unicode.org/reports/tr35/#RegionOverride) can be specified as part of the [BCP 47 `U`](https://unicode.org/reports/tr35/#u_Extension) of a language tag.  We will explore the `U` extension in a later post but for now we can show some examples of its use:

```elixir
iex> MyApp.Cldr.Locale.territory_from_locale "en-u-rg-auzzzz"
:AU

iex> MyApp.Cldr.Locale.territory_from_locale "ja-u-rg-uszzzz"
:US
```

The syntax is unusual in order to comply with the overall specification of language tags however it is straight forward to understand when recognising that the `zzzz` is padding to create a 6-character region override code.

Why is overriding the territory (region) preference useful? In `CLDR`, the `region override (rg)` is used to [overide the territory to be used](https://unicode.org/reports/tr35/tr35-info.html#rgScope) when:

* Identifying the desired calendar to be used
* Identifying the which unit preferences to be used
* Identifying the currency formatting to be used

Its not a common case the overriding the territory becomes a requirement so it is not something to be used without specific use case.

### The users preferred currency

In the same way that we can infer the users preferred territory from a language tag, we can similarly infer the preferred currency or override it is desired.  Since most territories have a single authorised currency, its a simple matter to identify the preferred territory using then process defined above and the look up the currency associated with that territory. The function `Cldr.Currency.currency_from_locale/1` takes care of all of that for us. For example:

```elixir
iex> Cldr.Currency.currency_from_locale "en"
:USD

iex> Cldr.Currency.currency_from_locale "en-AU"
:AUD

iex> Cldr.Currency.currency_from_locale "en-JP"
:JPY

iex> Cldr.Currency.currency_from_locale "en-IR"
:IRR
```

However in some circumstances a user may prefer a specific currency be used by default. Perhaps you have your secret stash in a Swiss bank account and prefer to have currency reporting in Swiss Francs (`:CHF`). In that case:

```elixir
iex> Cldr.Currency.currency_from_locale "en-u-cu-chf"
:CHF
```

It's much preferred that currencies be cleary specified in all operations. However a default currency is helpful when parsing user input. `Money.parse/2` can use this default currency.  For example:

```elixir
iex> Money.parse("100")
#Money<:USD, 100>

iex> Money.parse("100", locale: "en-AU")
#Money<:AUD, 100>

iex> Money.parse("100", locale: "en-AU-u-cu-chf")
#Money<:CHF, 100>
```

In order to be especially cautious (and with money that's never a bad thing), the option `default_currency: false` can be passed. For example:

```elixir
iex> Money.parse("100", default_currency: false)
{:error, {Money.Invalid,
  "A currency code, symbol or description must be specified but was not found in \"100\""}}
```

We'll cover localised money in a separate post, but money parsing is really quite flexible and can recognise both currency symbols and textual currencies like:

```elixir
iex> Money.parse "USD 100,00", locale: "de"
#Money<:USD, 100.00>

iex> Money.parse("100", default_currency: :EUR)
#Money<:EUR, 100>

iex> Money.parse("100 eurosports", fuzzy: 0.8)
#Money<:EUR, 100>

iex> Money.parse("100 eurosports", fuzzy: 0.9)
{:error, {Money.UnknownCurrencyError, "The currency \"eurosports\" is unknown or not supported"}}

iex> Money.parse("100 afghan afghanis")
#Money<:AFN, 100>
```

### The users preferred calendar

Although for civil use the [proleptic gregorian calendar](https://en.wikipedia.org/wiki/Proleptic_Gregorian_calendar) may be the most commonly used calendar, many other calendars are in use around the world for civil or religious purposes.

`CLDR` maintains content to allow the localisation of calendar information for a number of calendars which can be returned by `Cldr.known_calendars/0`. For example:

```elixir
iex> Cldr.known_calendars
[:buddhist, :chinese, :coptic, :dangi, :ethiopic, :ethiopic_amete_alem,
 :gregorian, :hebrew, :indian, :islamic, :islamic_civil, :islamic_rgsa,
 :islamic_tbla, :islamic_umalqura, :japanese, :persian, :roc]
```

Since `ex_cldr` attempts to make localisation as simple as possible, localising dates and times requires a calendar implementation for any `CLDR` calendar that is to be localised.  As of this November 2020, the following calendars are supported:

* `:gregorian` with the `Cldr.Calendar.Gregorian` module defined by [ex_cldr_calendars](https://hex.pm/packages/ex_cldr_calendars)
* `:coptic` with the `Cldr.Calendar.Coptic` module defined by [ex_cldr_calendars_coptic](https://hex.pm/packages/ex_cldr_calendars_coptic)
* `:egyptian` with the `Cldr.Calendar.Egyptian` module defined by [ex_cldr_calendars_egyptian](https://hex.pm/packages/ex_cldr_calendars_egyptian)
* `:persian` with the `Cldr.Calendar.Persian` module defined by [ex_cldr_calendars_persian](https://hex.pm/packages/ex_cldr_calendars_persian)

Many locales support allow more than one calendar type with a preferred calendar as the default.  The supported calendars for a given locale can be returned with `Cldr.Calendar.Preference.preferences_for_territory/1`. For example:

```elixir
iex> Cldr.Calendar.Preference.preferences_for_territory(:AU)
{:ok, [:gregorian]}

iex> Cldr.Calendar.Preference.preferences_for_territory(:IR)
{:ok, [:persian, :gregorian, :islamic, :islamic_civil, :islamic_tbla]}

iex> Cldr.Calendar.Preference.preferences_for_territory(:JP)
{:ok, [:gregorian, :japanese]}
```

The first entry in the list is the preferred and default calendar and the other calendars in the list are also valid to be selected - as long as that calendar has an implementation in `ex_cldr`.

If the requested calendar is valid but has no implementation in `ex_cldr` then the first calendar in the preference list that *does* have an implementation is selected. This will most commonly be the gregorian calendar.

Now we can put all the pieces together an identify what a user's calendar preference is for a given langauge tag.

```elixir
iex> Cldr.Calendar.calendar_from_locale "en"
{:ok, Cldr.Calendar.US}

iex> Cldr.Calendar.calendar_from_locale "en-AU"
{:ok, Cldr.Calendar.AU}

iex> Cldr.Calendar.calendar_from_locale "fa-IR"
{:ok, Cldr.Calendar.Persian}
```

#### Variations on the theme of the Gregorian calendar

Although the gregorian calendar may be in common use, the application of that calendar does vary from territory to territory.  For example, in the US the first day of the week is considered to by Sunday and in the UK it is considered to be Monday.  In conjunction with the [ex_cldr_calendars](https://hex.pm/packages/ex_cldr_calendars), `ex_cldr` will return a calendar module that most closely reflects the calendar preferences of the territory appropriate for the given language tag. Some examples:

```elixir
iex> Cldr.Calendar.calendar_from_locale "en"
{:ok, Cldr.Calendar.US}

iex> Cldr.Calendar.calendar_from_locale "en-GB"
{:ok, Cldr.Calendar.GB}
```

You might reasonably ask why, since both territories use the gregorian calendar, a different calendar is returned?  [ex_cldr_calendars](https://hex.pm/packages/ex_cldr_calendars) builds on the Elixir `Calendar` protocol to provide a very flexible calendar model that encompasses week-based calendars like the [ISO Week](https://en.wikipedia.org/wiki/ISO_week_date) calendar, month-based calendars, and fiscal calendars like the [445 calendar](https://en.wikipedia.org/wiki/4–4–5_calendar). `ex_cldr_calendars` calendars can have different start days of week, different start months and differnt month formations of a quarter too.

In the examples above the difference is solely that the US calendar starts the week on a Sunday and the GB calendar starts the week on a Monday. Unless you are using the functionality of `ex_cldr_calendars` the two calendars will work iterchangeably with the standard `Calendar.ISO` calendar.

### The users preferred number system

Although the [Hindu Arabic](https://en.wikipedia.org/wiki/Hindu–Arabic_numeral_system) is the most familiar number system to developers, there are many other number systems in use around the world. Some have the same decimal structure as the Hindu-Arabic system and others use a different, often algorithmic, number system.  The [numbers systems](https://unicode.org/reports/tr35/tr35-numbers.html#Numbering_Systems) known to `CLDR` can be returned with `Cldr.Number.System.known_number_systems/0`. There are more than 80 numbers systems defined. For example:

```elixir
iex> Cldr.Number.System.known_number_systems
[:adlm, :ahom, :arab, :arabext, :armn, :armnlow, :bali, :beng, :bhks, :brah,
 :cakm, :cham, :cyrl, :deva, :diak, :ethi, :fullwide, :geor, :gong, :gonm,
 :grek, :greklow, :gujr, :guru, :hanidays, :hanidec, :hans, :hansfin, :hant,
 :hantfin, :hebr, :hmng, :hmnp, :java, :jpan, :jpanfin, :jpanyear, :kali,
 :khmr,  :knda, :lana, :lanatham, :laoo, :latn, :lepc, :limb, :mathbold,
 :mathdbl, :mathmono, :mathsanb, ...]
```

Not all number systems are application to all locales. The valid number systems for a given locale can be returned by `Cldr.Number.System.number_systems_for/1`. For example:

```elixir
iex> Cldr.Number.System.number_systems_for "en"
{:ok, %{default: :latn, native: :latn}}

iex> Cldr.Number.System.number_systems_for "fa"
{:ok, %{default: :arabext, native: :arabext}}

iex> Cldr.Number.System.number_systems_for "he"
{:ok, %{default: :latn, native: :latn, traditional: :hebr}}
```

From these examples we can see that number systems can be referenced directly by name, such as `:latn` , `:arabext` and `:hebr` or by a number system type such as `:default`, `:native` and `:traditional`.  In `ex_cldr` a number system can be referenced by either the name directly or by a number system type *except* when referenced in a language tag. In this case, only the number system name is recognised.

Putting it all together as part of a language tag, we use the [nu](https://unicode.org/reports/tr35/#UnicodeNumberSystemIdentifier) subtag of the `U` extension as the following examples show:

```elixir
iex> Cldr.Number.System.number_system_from_locale "he"
:latn

iex> Cldr.Number.System.number_system_from_locale "he-u-nu-hebr"
:hebr

iex> Cldr.Number.System.number_system_from_locale "en"
:latn

iex> Cldr.Number.System.number_system_from_locale "en-u-nu-thai"
:thai
```

Although `CLDR` states that any parameter `nu` must be valid for the give locale, `ex_cldr` does not apply this constraint and any number system known to `CLDR` may be used in a language tag. Note how ever that `Cldr.Number.to_string/2` will enforce that the requested number system is known to the given locale.

#### How is the number system applied?

A number system is typically used when formatting a number. The following examples illustrate:

```elixir
iex> Cldr.Number.to_string 123456, locale: "th"
{:ok, "123,456"}

iex> Cldr.Number.to_string 123456, locale: "th-u-nu-thai"
{:ok, "๑๒๓,๔๕๖"}

# In the "ar" locale the number system is always :arab
iex> Cldr.Number.to_string 123456, locale: "ar"
{:ok, "١٢٣٬٤٥٦"}
```

### In Summary

In this post we learned that a language tag can convey a lot more information about the users preferences that may seem obvious on the surface. Building on the this basic knowledge we can construct more complex and functionally rich expressions of the users preference a canonical way to drive content localisation and parsing.

___

#### Footnotes

<sup>1</sup>The term territory is used deliberately. Although the term country is more widely used a country is a political entity and as such there are always shifting boundary and territorial disputes. Since our purpose is only to express user preferences without endorsing any particular political claim, we use the term territory. `CLDR` uses the term region which is considered here interchangable with territory.

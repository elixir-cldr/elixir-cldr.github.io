---
layout: post
categories: [cldr, i18n, l10n, locale, language tag]
title: Exploring Language Tags
---

The primary intent of localisation is to present data to and accept input from users according to their personal preferences. In this post we explore how a language tag can be used to help fulfill that intent.

### Expressing a users preference

Often a users preferences are rooted in a particular culture (such as a preferred language), their location (geographic region and timezone) as well as other considerations that are both well known (such as currency preference) or specific to an individual or application (maybe *vegetarian* versus *non-vegetarian* or favourite colour *blue*).

The first formalisation of an approach to express a users preference was defined in [RFC1766](https://tools.ietf.org/html/rfc1766) in 1995, superceded by [RFC3066](https://tools.ietf.org/html/rfc3066) in 2001 then by [RFC4646](https://tools.ietf.org/html/rfc4646) in 2005 and [RFC5646](https://tools.ietf.org/html/rfc5646) in 2006.  The most current standard is [Best Current Practice (BCP47)](https://tools.ietf.org/html/bcp47) and it's this version to which [CLDR](https://cldr.unicode.org) and [ex_cldr](https://hex.pm/packages/ex_cldr) comply.

For CLDR, [Unicode Technical Report 35 (TR35)](http://unicode.org/reports/tr35/#Unicode_Language_and_Locale_Identifiers) governs the interpretation of `BCP47` and describes implementation details to which `ex_cldr` conforms. Where `ex_cldr` does not conform it is to be considered a bug and [opening an issue](https://github.com/elixir-cldr/issues) is always appreciated.

### Language Tag Basics

Given that one of the most defining characteristic of any culture is its language it should not surprising that the simplest language tag consists of just a language identifier.

Typically, the language idenfifier is the corresponding [ISO639-1](https://en.wikipedia.org/wiki/List_of_ISO_639-1_codes) code for the given language. The following are well known examples:

* "en" for english
* "fr" for french
* "zh" for chinese
* "ja" for japanese
* "ar" for arabic

Using a language identifier goes a long way to allowing an application to present information in a format understandable by a user but it is still a long way from expressing a users preference given the variation in language use in different cultures. Chinese as a [macrolanguage](https://en.wikipedia.org/wiki/ISO_639_macrolanguage) has at least 16 languages within this grouping. The english-speaking world divides into two broad categories of countries that use [British spelling norms versus US spelling norms](https://en.wikipedia.org/wiki/American_and_British_English_spelling_differences). Differences also appear in date formatting, currencies, calendar use and measurement systems.

To further express a users preference, a language tag can also specify a territory name that recognises a given language as spoken within in a given territory. The territory identifiers are typically those defined in [ISO3166-1](https://en.wikipedia.org/wiki/ISO_3166-1) using the 2-letter variant.  Some examples are:

* "en-AU" for English as spoken in Australia
* "fr-CA" for French as spoken in Canada
* "zh-HK" for Chinese as spoken in Hong Kong
* "ar-SA" for Arabic as spoken in Saudi Arabia

In some cases, a territory expands more broadly beyond regional boundaries and in these cases, a [UN Standard Code for Statistical Use](https://en.wikipedia.org/wiki/UN_M49) code may be used. In `CLDR` such language tags are used where certain groupings of regions share common conventions. One example of this is `"ar-001"` which means "standard Arabic" where `001` is the UN.M49 code for "The World".

By reflecting a users preference language and territory an application can go a long way to respecting a users preferences. However there are significant populations whose written language can take different forms and to recognise this preference, a language tag can include a script identifier.  Where a given language has only a single written form a script subtag is normally not expressed but for languages where there are multiple scripts in common use applying a script subtag is desirable. Some examples:

* "az-Arab"	for Azerbaijani in Arabic script
* "az-Cyrl"	for Azerbaijani in Cyrillic script
* "az-Latn"	for Azerbaijani in Latin script
* "zh-Hans"	for Chinese, in simplified script
* "zh-Hant"	for Chinese, in traditional script

Combining a language identifier, a script identifier and a territory identifier we can address a significant part of the world's population in familiar terms:

* "zh-Hant-CN" for chinese speakers in mainland China using the traditional Chinese script
* "th-Thai-TH" for Thai speakers in Thailand using the Thai script

### Uppercase or lowercase? Hyphen or Underscore?

As defined in `BCP47`, language tags are case insensitive. The separator between subtags is defined to be `-` (hyphen).

Although case distinctions do not carry meaning in language tags, consistent formatting and presentation of language tags will aid users. This format generally corresponds to the common conventions for the various ISO standards from which the subtags are derived. These conventions include:

* [ISO639-1](https://en.wikipedia.org/wiki/List_of_ISO_639-1_codes) recommends that language codes be written in lowercase ("mn" Mongolian).
* [ISO15924](https://en.wikipedia.org/wiki/ISO_15924) recommends that script codes use lowercase with the initial letter capitalized ("Cyrl" Cyrillic).
* [ISO3166-1](https://en.wikipedia.org/wiki/ISO_3166-1) recommends that territory codes be capitalized ("MN" Mongolia).

`CLDR` and `ex_cldr` apply these conventions and both allow the alternative `_` (underscore) separator between subtags. `ex_cldr` parses language tags in a cases insensitive manner and supports both `-` and `_` separators although `-` is to be preferred.

### Language Tags supported by CLDR

`ex_cldr` will parse any valid language tag using `Cldr.LanguageTag.new/2` however only language tags that are known to `CLDR` can be used to localise user content since the language tag is used to index into the `CLDR` repository.

Thankfully due to the work of a significant number of [contributors](http://cldr.unicode.org/index/acknowledgments) there are over 500 locales known to `CLDR`. For the interested or curious, these can be returned by `Cldr.Config.all_locale_names/0`. For example:

```elixir
iex> Cldr.Config.all_locale_names
["af", "af-NA", "agq", "ak", "am", "ar", "ar-AE", "ar-BH", "ar-DJ", "ar-DZ",
 "ar-EG", "ar-EH", "ar-ER", "ar-IL", "ar-IQ", "ar-JO", "ar-KM", "ar-KW",
 "ar-LB", "ar-LY", "ar-MA", "ar-MR", "ar-OM", "ar-PS", "ar-QA", "ar-SA",
 "ar-SD", "ar-SO", "ar-SS", "ar-SY", "ar-TD", "ar-TN", "ar-YE", "as", "asa",
 "ast", "az", "az-Cyrl", "az-Latn", "bas", "be", "bem", "bez", "bg", "bm",
 "bn", "bn-IN", "bo", "bo-IN", "br", ...]
```

The more studious or observant will notice some language identifiers that are three character codes instead of the expected two. There is a lot more to a language tag that a simple `language-script-territory` combination and we'll explore those in following posts.

# Architecture: string-extra

## Purpose

A Twig extra extension that exposes Symfony String component functionality as Twig filters: Unicode-aware string manipulation, slugging, truncation, word-wrapping, and inflection (pluralisation/singularisation) directly in templates.

## Directory Structure

```
StringExtension.php   - The sole Twig extension class; registers all filters
Tests/
  IntegrationTest.php - Integration tests for each registered filter
```

## Key Design Decisions

- **Thin wrapper**: `StringExtension` delegates all processing to `symfony/string` — no string logic lives here. The extension purely maps Twig filter names to Symfony String method calls.
- **Unicode-first**: Filters operate on `UnicodeString` objects (which use grapheme clusters and code points rather than bytes), ensuring correct behaviour with multibyte characters.
- **Inflector selection**: The extension resolves a language-specific `InflectorInterface` (English, French, Spanish) for `u_singularize()`/`u_pluralize()` filters based on locale.

## Extension Points

- Register additional inflectors for new languages by extending `StringExtension` and adding a new locale entry.
- Override the `SluggerInterface` binding in the Symfony DI container to change slug generation behaviour.

## Dependency Flow

```
Twig template {{ "hello world" | u_truncate(5) }}
  └─> StringExtension::getFilters()   — registered TwigFilter list
        └─> UnicodeString methods    — actual string processing
              └─> symfony/string (UnicodeString, AsciiSlugger, inflectors)
```

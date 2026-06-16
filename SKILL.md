---
name: generate-repo-docs-files
description: >-
  Maak de bestanden aan die nodig zijn voor een goed open source project, of
    controleer of ze al aanwezig zijn. Gebruik deze skill wanneer de gebruiker
    een repository open source wil publiceren, ontbrekende bestanden wil
    toevoegen, of een project wil klaarmaken voor publieke bijdragen.
    Voorbeelden: "maak dit open source", "open source checklist", "voeg README
    toe", "CODE_OF_CONDUCT.md aanmaken", "publiccode.yml", "Standard for Public
    Code", "nieuwe open source repo opzetten", "welke bestanden heb ik nodig",
  "voeg een Security.md toe","voeg een licentie toe".
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Bash(gh api *)
  - Bash(gh issue list *)
  - Bash(gh pr list *)
  - Bash(gh search *)
  - Bash(git *)
  - Bash(curl *)
  - Bash(find *)
  - Bash(npx *)
  - Bash(docker *)
  - Bash(/tmp/pcval/publiccode-parser-go *)
  - Bash(npx @stoplight/spectral-cli *)
  - WebFetch(*)
  - AskUserQuestion
  - Read
  - Bash(curl -s *)
  - Bash(npx github:developer-overheid-nl/oas-generator *)
  - Bash(npx @developer-overheid-nl/don-checker *)
  - Bash(npx @stoplight/spectral-cli *)
  - Bash(npx @redocly/cli *)
  - Bash(npx @openapitools/openapi-generator-cli *)
  - Bash(git clone *)
  - WebFetch(*)
---

# Maak alle bestanden aan die je nodig hebt voor een Open Source project

Maak alle benodigde bestanden aan die je nodig hebt om je project te open sourcen.

## Instructies

- Gebruik de `repo-docs-generator` om de files te genereren: `npx github:developer-overheid-nl/repo-docs-generator`
- Zet de files in de root van het project.
- Doe dit doormiddel van het aanmaken van een input.json op basis van de beschikbare projectinformatie. De input.json geef je vorm aan de hand van deze schema.json: https://github.com/developer-overheid-nl/repo-docs-generator/blob/main/input_json_schema.json
- Vraag zo nodig aan de user om extra input als je niet voldoende informatie hebt voor het aanmaken van de files.

## Projectinformatie verzamelen

Zoek naar de volgende bestanden en lees ze om projectinformatie te verzamelen:

- `README.md` — naam, beschrijving, doel van het project
- Doorzoek het project op een `openapi.json` bestand. Gebruik de inhoud hiervan
  bij het genereren van de publiccode.yml.
- `package.json` / `pom.xml` / `pyproject.toml` / `Cargo.toml` / `go.mod` —
  controleer of er nuttige eigenschappen in staan.
- Bestaande lokale `publiccode.yml` — als die bestaat, werk deze bij in plaats
  van een nieuwe aan te maken.
- `.git/config` of remote URL — voor de `url` van de repository.

Haal aanvullende informatie op via de remote:

- Gebruik de repository-URL om projectinformatie op te halen van de git-omgeving
  (GitHub / GitLab / Forgejo).
- Haal voor `maintenance.contacts[].name` de weergavenaam van de
  GitHub-organisatie op via de organisatiepagina (bijv.
  `https://github.com/<org>`). Gebruik de weergavenaam zoals getoond op de
  profielpagina, niet de slug uit de URL.
- Gebruik diezelfde weergavenaam ook als `legal.mainCopyrightOwner`.
- Als er geen git link te vinden is vraag dan om input aan de user.

## LandingURL bepalen

Zoek naar een live URL waarop de software draait (bijv. in README, `one.json`,
CI/CD-configuratie of andere configuratiebestanden). Als die gevonden wordt,
voeg deze toe als `landingURL` direct onder `url`. Zet de URL **niet** in de
`longDescription`.

## Licentie bepalen

De repo-docs-generator CLI zal een LICENSE file genereren, deze opslaan.

## SoftwareType bepalen

Gebruik de volgende criteria (in volgorde van prioriteit):

- Bevat het project een plugin-manifest (`plugin.json`, `.claude-plugin/`,
  `.vscode/`, etc.) of is het expliciet een extensie/addon voor een andere
  applicatie? → `addon`
- Zijn het alleen configuratiebestanden zonder uitvoerbare code? →
  `configurationFiles`
- Is het een herbruikbare bibliotheek zonder eigen UI of entrypoint? → `library`
- Heeft het een webinterface? → `standalone/web`
- Draait het als achtergrondservice of API zonder eigen UI? →
  `standalone/backend`
- Heeft het een desktopinterface? → `standalone/desktop`
- Anders → `standalone/other`

## Name

Zorg dat de sleutel `name` een normale titel bevat, geen kebab-case, camelCase,
PascalCase of snake_case.

## releaseDate en softwareVersion

`releaseDate` en `softwareVersion` sleutels **niet** toevoegen. Als ze bestaan,
verwijder ze dan.

## Lege regels na blokken met sub-sleutels

Na een sleutel met meerdere sub-sleutels moet altijd een lege regel worden
toegevoegd. Enkelvoudige sleutels (zoals `name`, `url`, `softwareType`,
`developmentStatus`) krijgen géén lege regel erna.

Correct:

```yaml
name: Mijn Project
url: https://example.com
softwareType: addon
developmentStatus: development

description:
  nl:
    shortDescription: ...

legal:
  license: EUPL-1.2
```

## longDescription en shortDescription opmaak

Schrijf `longDescription` als een YAML literal block scalar (`|`) met harde
regelafbrekingen zodat elke regel maximaal 80 tekens breed is. Voorbeeld:

```yaml
longDescription: |
  De OAS Generator is een webapplicatie waarmee ontwikkelaars eenvoudig een
  OpenAPI Specification (OAS) document kunnen genereren dat voldoet aan de
  Nederlandse API Design Rules.
```

Gebruik **geen** folded block scalar (`>`) voor `longDescription`, want die
voegt regels samen tot één lange regel.

Schrijf `shortDescription` als een YAML folded block scalar met strip-chomping
(`>-`) zodat elke regel maximaal 80 tekens breed is maar de waarde één
aaneengesloten string blijft (regelafbrekingen worden samengevoegd tot spaties).
Voorbeeld:

```yaml
shortDescription: >-
  Deze generator kan worden gebruikt om een OpenAPI-specificatie te maken dat
  voldoet aan de NL API Design Rules.
```

## Valideer met publiccode-parser-go

Valideer de publiccode.yml met https://github.com/developer-overheid-nl/don-checker:
npx @developer-overheid-nl/don-checker@latest validate --ruleset publiccode-05 --input ./publiccode.yml

## Referenties

- [publiccode.yml schemadocumentatie](https://yml.publiccode.tools/schema.core.html)
- [Categorielijst](https://yml.publiccode.tools/categories-list.html)
- [SPDX licentie-identifiers](https://spdx.org/licenses/)

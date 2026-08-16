# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A Minecraft Forge compatibility mod (`create_blue_skies_compat`) that bridges the **Create** mod and the **Blue Skies** mod: it adds crushed-ore and sheet items for Blue Skies ores (ventium, horizonite, falsite, charoite, aquite, diopside, moonstone) and Create-machine recipes (crushing, pressing, smelting, blasting, splashing) to process them.

**Branch-per-version repo**: `main` targets 1.19.2 (ForgeGradle 5, official mappings, deps as jars in `libs/`); `forge-1.20.1` targets 1.20.1 (ForgeGradle 6, Parchment mappings, templated mods.toml via `gradle.properties`). Porting further is capped by Blue Skies availability (as of Aug 2026: nothing newer than 1.20.4 NeoForge / 1.20.1 Forge exists, and Create skips 1.20.4 — so 1.20.1 is the ceiling until Modding Legacy ports Blue Skies to 1.21+).

Requires Java 17 toolchain. The Gradle host JVM must be ≤19 on both branches (Gradle 7.5–8.1 don't run on Java 21); on this machine use `JAVA_HOME=~/Library/Java/JavaVirtualMachines/corretto-19.0.2/Contents/Home`.

## Commands

```bash
./gradlew build          # Build the mod jar (output in build/libs/, reobfuscated via reobfJar)
./gradlew runClient      # Launch a Minecraft client with the mod (working dir: run/)
./gradlew runServer      # Launch a dedicated server with the mod
./gradlew runData        # Run data generators (outputs to src/generated/resources/)
./gradlew publishCurseForge   # Upload to CurseForge (project 860090; needs CURSEFORGE_TOKEN)
```

There are no unit tests or linters. Verification is done by building and running the client.

Note: `org.gradle.daemon=false` is set, so every Gradle invocation pays full startup cost.

## Publishing

CurseForge project ID is **860090** (slug `create-blue-skies-compat`). Publishing runs via CurseForgeGradle (`publishCurseForge` task) reading the token from `CURSEFORGE_TOKEN` env or `curseforge_token` gradle property. GitHub Actions workflow `.github/workflows/publish-curseforge.yml` (manual `workflow_dispatch`) publishes from CI using the repo secret; dispatch it against the release branch: `gh workflow run publish-curseforge.yml --ref forge-1.20.1`. Declared CurseForge relations: requires `create` and `blue-skies`. The 1.20.1 jar is tagged Forge + NeoForge.

CurseForgeGradle version must match the branch's Gradle: 1.1.18 on Gradle 8.1 (1.20.1 branch) — 1.1.26 needs Gradle 8.5+ (`Provider.filter` error otherwise).

## Architecture

The Java side is minimal — nearly all mod content is JSON data:

- **Java** (`src/main/java/net/celsiusqc/create_blue_skies_compat/`): `@Mod` entrypoint, `item/ModItems` (`DeferredRegister` of all items — new items must be registered here), and the creative tab class.
- **Recipes** (`src/main/resources/data/create_blue_skies_compat/recipes/`): organized by Create recipe type (`crushing/`, `pressing/`, `smelting/`, `blasting/`, `splashing/`), with crushing split per material. Recipes use Create's JSON recipe types (e.g. `create:crushing`) and reference `blue_skies:` ingredients.
- **Tags** (`src/main/resources/data/`): items are added to `create:crushed_ores` and `forge:plates/<metal>` so Create machines and other mods recognize them.
- **Assets** (`src/main/resources/assets/create_blue_skies_compat/`): item models, textures, `lang/en_us.json`. Every new item needs a model JSON, a texture, and a lang entry.

## Adding a new compat item (typical workflow)

1. Register the item in `ModItems`.
2. Add texture (`assets/.../textures/item/`), model (`assets/.../models/item/`), and lang entry (`assets/.../lang/en_us.json`).
3. Add recipe JSONs under `data/create_blue_skies_compat/recipes/<type>/`.
4. Add the item to relevant tags (`data/create/tags/...` or `data/forge/tags/...`).
5. Verify with `./gradlew runClient`.

# Minecraft 1.21.11

## Compatibility Focus

Minecraft 1.21.11 is stricter about how item models resolve textures and parent chains. A resource pack that worked in earlier 1.21.x builds can fail in 1.21.11 even when the textures themselves still exist.

The main requirement is that custom item models must resolve cleanly as item models. They cannot depend on block-model fallbacks, ambiguous parent paths, missing texture references, or placeholder texture keys.

## Required Changes

### Use item-safe fallback models

Any item definition that falls back to a model must fall back to an item model, not a block model.

This matters most for items that act as dispatch roots for large custom-model families. If the fallback stays on a block model, the entire item chain can fail to bake in 1.21.11.

### Stop using block parents for custom item models

Custom item models must not inherit from block-model parents.

If a model is rendered as an item, its parent chain should stay in the item-model space. This avoids mixed atlas resolution and the `Multiple atlases used` bake failure.

### Make custom parents explicit

Shared custom parents must use explicit namespaced parent paths.

Bare parent identifiers are no longer safe for custom models because they can resolve into the wrong namespace. A custom shared parent should resolve as a custom model, not as a vanilla model lookup.

### Define `particle` on custom item models

Custom item models that declare textures should also declare a `particle` texture.

In 1.21.11, missing `particle` references show up as load-time warnings. Even when a model still renders, these warnings indicate the model is not fully defined.

### Remove unresolved texture references

No face should point at an undefined texture key such as `#missing`.

Every texture reference used by model geometry must resolve to a real texture entry. Placeholder texture references that may have been tolerated before should be treated as invalid.

### Remove invalid texture paths

Texture paths must resolve exactly as written.

Typos such as malformed separators or doubled path delimiters can produce missing-texture warnings even when the asset exists.

## What To Check When Updating A Pack

### Parent chain

- Item models inherit from item-model parents only.
- Shared custom parents use explicit custom model paths.
- No custom item model depends on a block parent to render.

### Texture resolution

- Every model texture key used in faces is declared in the `textures` object.
- Every custom item model has a `particle` texture.
- No texture path contains malformed separators.
- No model contains `#missing` or other unresolved texture placeholders.

### Runtime validation

After loading the pack in 1.21.11, the client log should not report:

- `Unable to bake item model`
- `Multiple atlases used`
- `Missing block model`
- `Missing texture references in model`
- `Missing textures in model`

If any of those remain, the pack still has at least one broken item-model chain or unresolved texture reference.

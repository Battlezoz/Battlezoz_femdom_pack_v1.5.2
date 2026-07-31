# NeedsOfNature Customization Guide and Pack Creation Guide

This document describes the current customization and pack formats for NeedsOfNature.

The animation engine is built into NoN. Pack-facing names such as the `animationframework` resource namespace, `afw_animdefs` directory, and `afw_*` model keys remain the stable content format; they do not require a separate mod.

It covers both local player-side customization, such as ripped skin texture overrides, and NoN pack authoring for animations, models, accessory items, liquid values, recipes, and visual overlays without changing Java code.

---

## Ripped Skin & Masks

This chapter is written from the client/player point of view. It is for overriding your own ripped player texture and stage masks in your own Minecraft instance, not for creating a content pack.

### Ripped Player Textures

To override your own ripped player texture, place this file in:

```text
<minecraft instance>/config/needsofnature/destroyed_skin.png
```

Use a static, square player-skin layout PNG from 64x64 through 512x512 in 64-pixel steps. HD ripped skins retain their detail even with a normal 64x64 player skin and do not require an HD-skin mod. This file is the highest-priority player-side ripped skin override. If it exists, NoN uses it directly and ignores the generated tinted fallback and default-skin-specific ripped textures for your player.

Stage 1-3 blend this texture onto your normal skin through the ripped stage masks. Stage 4 uses the full `destroyed_skin.png` texture directly.

See `Tutorial/ripped skin examples/` for example texture files you can use as reference.

This local config file is uploaded and synced from the player who owns it. If multiple players want custom ripped textures, every player needs to provide their own files in their own `config/needsofnature/` directory.

### Male Model D Texture Area

The male player model uses the otherwise unused strip at `x=56-63`, `y=16-47` on a standard 64x64 player skin to texture its D bones. When creating `destroyed_skin.png` for a male or male+female player, texture this area as well.

The highlighted strip shows the section used by the male model's D bones.

![Male model D texture area](img/ripped_skin_d_texture_area.png)

<br>

### Ripped Stage Masks

Optional local stage mask overrides use:

```text
<minecraft instance>/config/needsofnature/destroyed_skin_mask_1.png
<minecraft instance>/config/needsofnature/destroyed_skin_mask_2.png
<minecraft instance>/config/needsofnature/destroyed_skin_mask_3.png
```

Masks are 64x64 grayscale/alpha-style PNGs.

White/opaque pixels apply the ripped texture for that stage. Black/transparent pixels keep the original skin. Stage 4 does not use a mask; it uses the full ripped texture.

### Quick Skin Compatibility

When Quick Skin is installed, NoN uses the currently selected static Quick Skin texture as the player's base skin. Ripped stages, mess and liquid overlays, and accessory overlays are applied to that skin both normally and during NoN animations. Standard-resolution and HD static skins are supported. Animated Quick Skin skins are not currently supported.

Changing the selected Quick Skin refreshes NoN's generated player textures automatically; leaving and rejoining the world is not required.

#### Per-Skin Ripped Textures and Masks

NoN can select different ripped textures and masks for each Quick Skin using the skin's display name. The name is converted to lowercase, spaces and unsafe path characters become underscores, and repeated separators are collapsed. For example, a Quick Skin named `My Skin` uses the folder `my_skin`:

```text
<minecraft instance>/config/needsofnature/my_skin/destroyed_skin.png
<minecraft instance>/config/needsofnature/my_skin/destroyed_skin_mask_1.png
<minecraft instance>/config/needsofnature/my_skin/destroyed_skin_mask_2.png
<minecraft instance>/config/needsofnature/my_skin/destroyed_skin_mask_3.png
```

These folders are local player configuration, not NoN pack content. Each file falls back independently. For an active Quick Skin, NoN uses this priority:

1. The matching file in the skin-named folder.
2. The corresponding global file directly inside `config/needsofnature/`.
3. NoN's normal generated/default ripped skin or bundled stage mask.

If Quick Skin has no skin selected, NoN skips the skin-named folder and uses the global files and normal fallback behavior.

Changing the selected Quick Skin or renaming it updates the active ripped texture and masks in game. The folder is based on the display name, not Quick Skin's internal asset id, so renaming a skin also changes the folder NoN looks for. Other players do not need Quick Skin or copies of these files; NoN synchronizes the resolved images through the server.

---

## Pack Creation

### AnimationDirector Blockbench Plugin

The AnimationDirector Blockbench plugin is called `Multi Actor Animator (Beta)` in Blockbench. It is meant for building NoN multi-actor animations in one Blockbench project without bone-name collisions.

To install it, open Blockbench and go to:

```text
File > Plugins
```

In the plugins menu, press `Load plugin from file` and select the `.js` plugin file in the zip downloadable from the modpage.

![Plugin Button](img/bb_add_plugin.png)

You also need the Blockbench plugin `GeckoLib Models & Animations` to create GeckoLib project types. Install it from Blockbench's plugin marketplace before creating NoN models or animations.

Create NoN animations in a Blockbench GeckoLib animated model Entity project. The exporter expects the GeckoLib/Bedrock model and animation format produced by that project type.

Use the default NoN pack's GeckoLib models as reference when setting up actors, bone names, and texture layout.

The plugin adds two tool menus:

```text
Tools > AnimationDirector Anims
Tools > AnimationDirector Model
```

Main animation workflow:

1. Use `AnimationDirector Anims > Import Actor` to import a GeckoLib/Bedrock `.json` or `.geo.json` model into the current project.
2. Each imported actor receives an internal `actorN_` prefix on its bones, for example `actor1_head`, `actor2_head`. This lets multiple actors share normal bone names like `head` and `body` without Blockbench renaming them unpredictably.
3. Use `Actor Preferences` to set each actor's export name. For NoN, this should usually match the animdef actor key, for example `player`, `wolf`, `wolf1`, or `actor2`.
4. Animate all actors together in one Blockbench animation.
5. Use `AnimationDirector Anims > Export Animations`, select the animation IDs you want, and export one conjoined JSON per animation.

Exported animation files strip the internal `actorN_` prefix again. If the Blockbench animations are named `wolfmplayer.p1`, `wolfmplayer.p2`, and so on, the exported file is:

```text
wolfmplayer.animation.json
```

Place it directly in the NoN animation folder:

```text
assets/<namespace>/geckolib/animations/afw/wolfmplayer.animation.json
```

The filename supplies the animation ID. Clips inside the file use only the stage and actor key, such as `p1_player`, `p1_wolf`, `p2_player`, and `p2_wolf`.

Actor preferences also support preview textures per actor. These are for easier editing in Blockbench and are saved into the project. Folder-level texture overrides can be enabled there too; those apply recursively and take priority over actor preview textures.

`AnimationDirector Model > Export GeckoLib Model` exports the current model as GeckoLib/Bedrock geometry and can include NoN-specific `afw_bone_textures` metadata when different bones/folders use different textures. This is useful for NoN models that need per-bone texture overrides.

For OptiFine/CEM models, use the separate Blockbench `CEM Template Loader` plugin. Load the vanilla entity template you want to add, copy your custom geometry onto that template, then save/export the `.jem` into `assets/minecraft/optifine/cem/`. Use the default NoN pack's CEM files as reference for naming and gender model variants. Warning: Cube-level rotations are not supported by Optifine models. If you need a rotated shape, place the cube or cube group inside its own folder/bone and rotate that folder/bone instead.

The plugin can re-import a conjoined file using `AnimationDirector Anims > Import Animation`. The animation ID is read from the filename, and the plugin lets you map each stored actor key back to an actor in the current project. It then rebuilds all stages with their `actorN_` prefixes restored. The old split-file import and export actions remain under the `Deprecated` submenu.

### GeckoLib Model Render Metadata

NoN supports a few optional metadata keys at the root of GeckoLib `.geo.json` model files. These keys are not part of vanilla GeckoLib, but NoN reads them from models under:

```text
assets/<namespace>/geckolib/models/
```

#### Bone Texture Overrides

Use `afw_bone_textures` when only specific bones should use a different texture. This is useful for extra detail parts, gender-specific feature parts, or appendages that should not share the base entity texture.
If you are looking for the bone texture override feature, this is the implemented JSON key.

The key is an object mapping GeckoLib bone names to texture ids:

```json
{
  "afw_bone_textures": {
    "left_feature": "mymod:textures/entity/example/features.png",
    "right_feature": "mymod:textures/entity/example/features.png"
  },
  "format_version": "1.12.0",
  "minecraft:geometry": [
    {
      "description": {
        "identifier": "geometry.example",
        "texture_width": 64,
        "texture_height": 64
      },
      "bones": [
        {
          "name": "left_feature",
          "pivot": [0, 0, 0]
        },
        {
          "name": "right_feature",
          "pivot": [0, 0, 0]
        }
      ]
    }
  ]
}
```

Only the listed bones are rendered again with the override texture. The bone names must match the final exported GeckoLib bone names exactly.

#### Emissive Textures

Use `afw_emissive_textures` when the model should render one or more fullbright overlay textures, for example glowing spider eyes.

The key is an array of texture ids:

```json
{
  "afw_emissive_textures": [
    "minecraft:textures/entity/spider_eyes.png"
  ],
  "format_version": "1.12.0",
  "minecraft:geometry": [
    {
      "description": {
        "identifier": "geometry.example",
        "texture_width": 64,
        "texture_height": 32
      },
      "bones": []
    }
  ]
}
```

Each emissive texture is rendered as a full-light overlay on bones that use the model's normal texture. The texture should be transparent everywhere that should not glow. Bones listed in `afw_bone_textures` are intentionally excluded so unrelated pixels in a model-wide emissive texture cannot make a custom-textured feature glow.

If a custom-textured bone should also glow, map that bone to its own emissive texture with `afw_emissive_bone_textures`:

```json
{
  "afw_bone_textures": {
    "glowing_feature": "mymod:textures/entity/example/features.png"
  },
  "afw_emissive_textures": [
    "mymod:textures/entity/example/eyes_e.png"
  ],
  "afw_emissive_bone_textures": {
    "glowing_feature": "mymod:textures/entity/example/features_e.png"
  }
}
```

Only the named bone is rendered with the corresponding fullbright texture. A bone override is non-emissive unless it is explicitly included in this map.

#### Render Settings

Use `afw_render` for model render settings. Set `translucent` to `true` when the entire GeckoLib model should use the translucent pipeline:

```json
{
  "afw_render": {
    "translucent": true
  },
  "format_version": "1.12.0",
  "minecraft:geometry": [
    {
      "description": {
        "identifier": "geometry.example",
        "texture_width": 64,
        "texture_height": 32
      },
      "bones": []
    }
  ]
}
```

This applies the translucent render layer to the whole model.

For models containing both opaque and translucent geometry, use `translucent_bones` instead. The listed bones are omitted from the normal model pass and rendered afterward through an ordered translucent overlay pass:

```json
{
  "afw_render": {
    "translucent_bones": ["outer_shell"]
  },
  "format_version": "1.12.0",
  "minecraft:geometry": [
    {
      "description": {
        "identifier": "geometry.example",
        "texture_width": 64,
        "texture_height": 32
      },
      "bones": [
        {
          "name": "inner",
          "pivot": [0, 0, 0]
        },
        {
          "name": "outer_shell",
          "pivot": [0, 0, 0]
        }
      ]
    }
  ]
}
```

Only the listed bone's own cubes use this pass; child bones remain in the normal pass unless they are listed separately. Keep opaque features such as eyes in normal bones and place only transparent shell geometry in translucent bones. Bone texture overrides continue to apply to translucent bones.

### Pack Location

NoN loads external packs from the game directory:

```text
<minecraft instance>/needsofnature/
```

Both folders and `.zip` files are supported. Each pack uses normal Minecraft resource/data pack structure:

```text
needsofnature/
  MyPack/
    pack.mcmeta
    pack.png
    data/
      animationframework/
        afw_animdefs/
          my_animation.json
      needsofnature/
        non_liquid_gains/
          my_liquids.json
        non_entity_profiles/
          my_entity_profiles.json
    assets/
      animationframework/
        geckolib/
          animations/
            afw/
              my_animation.animation.json
      needsofnature/
        non_accessory_items/
          my_accessory.json
```

### Empty Pack Structure Examples

Two copyable empty pack templates are included in the guide:

- [`example_packs/general_pack`](example_packs/general_pack/README.txt) uses the normal shared NoN GeckoLib model paths. Choose it when the pack should reuse the default/shared model set or intentionally provide shared model overrides.
- [`example_packs/isolated_pack`](example_packs/isolated_pack/README.txt) enables `"model_scope": "pack"` and contains the matching ID-derived model path. Choose it when the pack must keep its GeckoLib models and bone structures separate from other animation packs.

Both templates are intentionally minimal. They contain only a valid `pack.mcmeta` and explanatory placeholders for animdefs, conjoined animation files, and GeckoLib models. Replace the `example` namespace and metadata before use, then delete each placeholder as the corresponding real files are added.

Use `/reload` for server-data changes such as animdefs and liquid gains. Restart the game for data-driven accessory item registration, because new item registry entries must exist during startup.

### Minimal `pack.mcmeta`

```json
{
  "pack": {
    "pack_format": 64,
    "supported_formats": [64, 81],
    "min_format": 64,
    "max_format": 81,
    "description": "My NoN pack"
  },
  "animationframework": {
    "id": "creator:my_pack",
    "name": "My Animation Pack",
    "author": "Creator Name",
    "version": "1.0.0",
    "description": "Adds custom NoN animations."
  }
}
```

The exact `pack_format` can change with Minecraft versions. Use the format accepted by the Minecraft version you target.

The optional `animationframework` section is used by NoN's Loaded Animations screen. It groups animations by pack, shows the pack name/author, and allows the server/host to disable a whole pack or individual animations. Use a stable namespaced `id`; changing it will make existing disabled-pack settings point at the old id.

### Pack-scoped GeckoLib Models

Models normally use the shared NoN model paths under:

```text
assets/animationframework/geckolib/models/entity/
```

Normal Minecraft resource-pack priority applies to those paths, so the highest-priority version of a model is used by every animation pack. This is useful for animation-only packs that intentionally reuse the default models.

If a pack supplies its own models with a different bone structure, it can isolate them by adding `model_scope: "pack"` to its `animationframework` metadata:

```json
{
  "animationframework": {
    "id": "creator:my_pack",
    "name": "My Animation Pack",
    "author": "Creator Name",
    "model_scope": "pack"
  }
}
```

Pack-scoped models use the pack's namespaced `id` as their model root. For the id `creator:my_pack`, place models under:

```text
assets/creator/geckolib/models/my_pack/entity/wolf.m.geo.json
assets/creator/geckolib/models/my_pack/entity/wolf.f.geo.json
assets/creator/geckolib/models/my_pack/entity/wolf.mf.geo.json
assets/creator/geckolib/models/my_pack/entity/player.m.geo.json
assets/creator/geckolib/models/my_pack/entity/player.f.geo.json
assets/creator/geckolib/models/my_pack/entity/player.mf.geo.json
```

The `.m` model is used for male entities, `.f` for female entities, and `.mf` for entities with both genders. Lookup order is `.m`, `.f`, then the unsuffixed model for male entities; `.f`, `.m`, then unsuffixed for female entities; and `.mf`, `.m`, `.f`, then unsuffixed for combined-gender entities. An unsuffixed model such as `wolf.geo.json` therefore remains the final shared fallback for every gender.

Only animations whose animdefs come from this pack use that model root. These models do not replace the shared models or models belonging to packs with other ids. A valid explicit `animationframework.id` is required, and two scoped packs must not use the same id.

Scoped lookup is strict. If the pack does not provide a required generic, gendered, or entity-variant model inside its own model root, NoN shows a missing model and a setup warning instead of borrowing a shared model. Omit `model_scope` when the pack is intended to reuse the shared/default model set.

Place a `pack.png` next to `pack.mcmeta` to show a pack image in the Loaded Animations screen:

```text
MyPack/
  pack.mcmeta
  pack.png
  assets/
  data/
```

### Debug Staff

The Nature Debug Staff is not shown in the creative inventory. Give it to yourself with:

```mcfunction
/give @s needsofnature:debug_staff
```

Right-click an entity that uses NoN energy to print a debug report in chat. The report includes the entity id, UUID, gender, energy, energy gain values, active/pending animation state, aura information, cooldowns, liquid state, and pregnancy state when those systems apply.

Sneak-right-click an energy-capable entity to add `25` energy, up to that entity's max energy. This is useful for testing whether a mob reaches attack/join thresholds and whether pack animations are eligible.

### Animation Definition Files

Animation definitions are loaded from:

```text
data/<namespace>/afw_animdefs/<animation_id>.json
```

The file path becomes the animation id. For example:

```text
data/creator/afw_animdefs/wolfmplayer.json
```

becomes:

```text
creator:wolfmplayer
```

Subfolders are also part of the id path.

### Full Animdef Skeleton

```json
{
  "display_name": "Wolf Encounter",
  "actors": [
    {
      "label": "wolf",
      "entity_types": ["minecraft:wolf"],
      "actor_tags": ["optional.required.tag"],
      "actor_tags_any": ["optional.one", "optional.two"],
      "activity": "active",
      "injector": "V",
      "receiver": false
    },
    {
      "label": "player",
      "entity_types": ["minecraft:player"],
      "activity": "passive",
      "receiver": true,
      "prop_right": "minecraft:carrot",
      "prop_floor": "minecraft:lantern"
    }
  ],
  "weight": 1.0,
  "animation_tags": ["normal_match"],
  "content_tags": ["doggy"],
  "join_after": ["creator:wolfmplayer"],
  "required_union_tags": ["some.tag"],
  "speed": 1.0,
  "position_anchor_actor": "player",
  "liquid_gain_multiplier": 1.0,
  "escapable": true,
  "stage_seconds": 10,
  "manual_peak": {
    "held_items": ["minecraft:carrot", "minecraft:golden_carrot"],
    "priority": 10,
    "prop_from_held_item": true,
    "held_item_prop_slot": "right"
  },
  "block_requirements": {
    "type": "center_support",
    "support": "slab",
    "clearance": {
      "width": 1,
      "height": 2,
      "depth": 1
    },
    "surface_radius": 1
  },
  "water": "none",
  "stages": [
    {
      "stage": 1,
      "loop": true,
      "cycle_seconds": 1.0,
      "allow_join": true,
      "speed": 1.0,
      "escapable": true,
      "stage_seconds": 8,
      "stage_duration_multiplier": 1.0,
      "props": {
        "player": {
          "prop_right": "minecraft:carrot",
          "prop_floor": "minecraft:lantern"
        }
      }
    },
    {
      "stage": 2,
      "use_stage": 1,
      "loop": true,
      "cycle_seconds": 1.0,
      "cycle_midpoint_offset_seconds": -0.2,
      "allow_join": false,
      "non_peak": true,
      "escapable": false,
      "stage_seconds": 6,
      "stage_duration_multiplier": 0.5
    }
  ]
}
```

Not every key is needed. The required top-level keys are `actors` and `stages`.

### Top-Level Animdef Keys

| Key                      | Type            | Default        | Meaning                                                                                                           |
| ------------------------ | --------------- | -------------- | ----------------------------------------------------------------------------------------------------------------- |
| `display_name`           | string          | animation id   | Optional player-facing name used by NoN's animation selection and request screens.                                |
| `actors`                 | array           | required       | Actor constraints. Must contain at least one actor.                                                               |
| `weight`                 | number          | `1.0`          | Random selection weight within the highest-specificity candidate pool. Must be positive.                          |
| `animation_tags`         | string array    | empty          | Functional tags used by matching, manual peak, defeated poses, and special starts. Tags are normalized lowercase. |
| `content_tags`           | string array    | empty          | General content labels shown in NoN's loaded-animation and request screens, and used to match follow-up defeated animations. Tags are normalized lowercase. |
| `join_after`             | id array        | empty          | Restricts this definition to normal joins that replace one of the listed predecessor animations.                  |
| `required_union_tags`    | string array    | empty          | All tags must exist in the combined command tags of all selected actors.                                          |
| `speed`                  | positive number | `1.0`          | Base animation speed. Stage speed overrides it.                                                                   |
| `block_requirements`     | object          | none           | Requires a wall or center support before the animation can start.                                                 |
| `water`                  | string          | `none`         | Water placement requirement: `none`, `surface`, or `underwater`.                                                  |
| `position_anchor_actor`  | string          | none           | Actor key used as preferred anchor source. Usually a `label`.                                                     |
| `stages`                 | array           | required       | Multi-stage animation settings.                                                                                   |
| `escapable`              | boolean         | `true`         | NoN attack escape UI/logic default. Can be overridden per stage.                                                  |
| `stage_seconds`          | integer         | config default | NoN duration override for a stage. `-1` means indefinite.                                                         |
| `liquid_gain_multiplier` | number          | `1.0`          | NoN multiplier for liquid gained from this animation. Negative values are ignored.                                |
| `manual_peak`            | object          | none           | NoN held-item manual peak selection rule.                                                                         |

Do not combine `block_requirements` and `water` for the same animation. The current placement systems are separate.

### Actor Keys

NoN treats actor constraints as an unordered multiset. JSON actor order does not force which entity becomes which actor slot.

Use `label` when you care about deterministic actor clip keys. For example, this actor:

```json
{
  "label": "wolf1",
  "entity_types": ["minecraft:wolf"]
}
```

uses actor key `wolf1`, so stage 1 uses this key inside `wolfm2player.animation.json`:

```text
p1_wolf1
```

If no label is provided, NoN derives an actor key from the actor constraints. Duplicate keys get `_2`, `_3`, etc.

### Actor Constraint Keys

| Key              | Type           | Default  | Meaning                                                                                                                         |
| ---------------- | -------------- | -------- | ------------------------------------------------------------------------------------------------------------------------------- |
| `label`          | string         | derived  | Explicit actor key for resource lookup.                                                                                         |
| `entity_types`   | string array   | wildcard | Valid entity ids. Use `minecraft:player` for all player models. Do not use `minecraft:player_slim`.                             |
| `entity_variant` | string         | none     | Exact entity-state variant. Currently supports slime and magma cube sizes as `size_0`, `size_1`, etc.                            |
| `actor_tags`     | string array   | empty    | AND command-tag requirement. The entity must have every tag.                                                                    |
| `actor_tags_any` | string array   | empty    | OR command-tag requirement. The entity must have at least one listed tag.                                                       |
| `activity`       | string         | `active` | `active` or `passive`. NoN attack starts cannot use a passive attacker role.                                                    |
| `prop_left`      | item id string | none     | Default item prop for this actor's left prop bone. Primarily intended for player actors.                                        |
| `prop_right`     | item id string | none     | Default item prop for this actor's right prop bone. Primarily intended for player actors.                                       |
| `prop_floor`     | item id string | none     | Default item prop for this actor's floor prop bone. Primarily intended for player actors.                                       |
| `injector`       | boolean/string | none     | NoN injector role. Boolean `true` means generic `V`; string values are `V`, `A`, or `M`.                                        |
| `receiver`       | boolean        | inferred | NoN receiver role. If no receivers are explicit, non-injectors become receivers; if still none, player actors become receivers. |

For slime and magma cube actors, `entity_variant` uses the vanilla NBT `Size` value. The smallest slime is `size_0`. A size-specific model should use the same suffix, for example:

```json
{
  "entity_types": ["minecraft:slime"],
  "entity_variant": "size_0"
}
```

```text
assets/animationframework/geckolib/models/entity/slime_size_0.geo.json
assets/animationframework/geckolib/models/entity/magma_cube_size_3.geo.json
```

The included `model_debug_pack` has separate debug animdefs for the normal vanilla slime and magma cube sizes: `size_0`, `size_1`, and `size_3`.

### Injector Roles

NoN supports three injector roles:

| Value | Meaning                                                                                  |
| ----- | ---------------------------------------------------------------------------------------- |
| `V`   | V-slot injection. Adds to V-type liquid tanks.                                           |
| `A`   | A-slot injection. Adds to A-type liquid tanks.                                           |
| `M`   | M-type peak. Does not add liquid; currently triggers the delayed hunger reward behavior. |

Boolean `injector: true` is treated as `V`. Boolean `false` means no injector role.

Do not mark the same actor as both an injector and an explicit receiver. The current parser does not reject this, but it places that actor into both role sets, which is ambiguous and should be treated as unsupported pack syntax. Use separate actor roles instead.

Player tank rules:

| Player gender setting | Liquid tank type |
| --------------------- | ---------------- |
| Female                | `V`              |
| Female + male         | `V`              |
| Male only             | `A`              |

Male-only V-to-A conversion is a NoN debug/compat setting. When enabled, suitable V injections can be treated as A for male-only players if the A slot is not already occupied by another injector.

### Actor Activity

`activity` is used by NoN matching logic.

| Value     | Behavior                                                                         |
| --------- | -------------------------------------------------------------------------------- |
| `active`  | The role can be used by a mob initiating an attack.                              |
| `passive` | The role is valid for non-attack/voluntary joins, but not as the attacking role. |

If omitted, actors are `active`.

### Animation Tags

`animation_tags` are functional matching tags.

Common tags:

| Tag               | Meaning                                                                                                                                                                             |
| ----------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `normal_match`    | Allows a tagged animation to still be considered by generic matching. Without this, tagged definitions are ignored by generic NoN matching unless that tag is explicitly requested. |
| `manualpeak`      | Marks an animation as a NoN manual peak candidate.                                                                                                                                  |
| `defeat.on_belly` | Defeated animation pose with the player on belly.                                                                                                                                   |
| `defeat.on_back`  | Defeated animation pose with the player on back.                                                                                                                                    |
| `defeat.side`     | Side-facing defeated animation pose.                                                                                                                                                |

### Content Tags

`content_tags` describe the content or position type. For ordinary animation matching they are descriptive rather than actor requirements.

They also act as general player-facing labels for an animation. NoN displays them in the Loaded Animations list and on animation cards in the animation request screen, so keep them short and useful to players.

They also provide data-driven defeated-animation transitions. A defeated animation lists the source content tags that fit its pose:

```json
{
  "animation_tags": ["defeat.on_belly"],
  "content_tags": ["doggy", "from_behind", "on_belly"]
}
```

After an animation ends in defeat, NoN prefers defeated animations sharing at least one content tag with that source animation. Multiple entries use OR semantics: the example accepts `doggy`, `from_behind`, or `on_belly` sources. Tag matching is exact after lowercase normalization.

A side-facing defeated pose can similarly use `animation_tags: ["defeat.side"]` with `content_tags: ["spooning", "side"]`.

If source and defeated animations both omit `content_tags`, they match each other. If no defeated animation shares the source tags, NoN falls back to a weighted random choice among every otherwise valid defeated animation. The normal animdef `weight` controls choices whenever multiple candidates remain.

### Player Request Previews

Animations that can be selected for player-to-player requests may provide an optional square preview image. For animation id `<namespace>:<animation_id>`, place the PNG at:

```text
assets/<namespace>/textures/afw/previews/<animation_id>.png
```

For example, `creator:playermplayer-cowgirl` uses:

```text
assets/needsofnature/textures/afw/previews/playermplayer-cowgirl.png
```

Use a square PNG; 64x64 is recommended. NoN displays a generic placeholder when the image is missing. The selection card also displays the animdef's `content_tags`, so keep those tags short and useful to players.

Set the optional top-level `display_name` animdef field to give the animation a player-facing name in the selection, role, and consent screens. If it is omitted, NoN displays the animation id instead. The preview filename and animation identity always continue to use the real animation id.

Only ordinary, currently eligible normal-match animations appear in this selector. Manual peak, birth, fill-bottle, and defeated animations are excluded from direct player requests.

### Matching and Weights

NoN first finds all definitions that match the selected actor set and required animation tags.

Then it sorts by specificity. Specificity increases with:

| Source                            | Effect              |
| --------------------------------- | ------------------- |
| More specific `entity_types`      | Higher specificity. |
| More required `actor_tags`        | Higher specificity. |
| More `actor_tags_any` constraints | Higher specificity. |
| More `required_union_tags`        | Higher specificity. |

Only the highest-specificity group is used. `weight` is applied inside that group. A low-specificity animation with very high weight will not beat a higher-specificity animation.

### Join Chains

Use the optional top-level `join_after` array when an expanded animation should only follow specific active animations. The destination owns the relationship, so a newly added follow-up does not require editing the predecessor.

For example, the two-actor follow-up declares the one-actor animation as its predecessor:

```json
{
  "join_after": ["creator:silverfishmplayer"],
  "actors": [
    { "label": "player", "entity_types": ["minecraft:player"] },
    { "label": "silverfish1", "entity_types": ["minecraft:silverfish"] },
    { "label": "silverfish2", "entity_types": ["minecraft:silverfish"] }
  ],
  "stages": [
    { "stage": 1 }
  ]
}
```

The next expansion can continue the chain independently:

```json
{
  "join_after": ["creator:silverfishm2player"],
  "actors": [
    { "label": "player", "entity_types": ["minecraft:player"] },
    { "label": "silverfish1", "entity_types": ["minecraft:silverfish"] },
    { "label": "silverfish2", "entity_types": ["minecraft:silverfish"] },
    { "label": "silverfish3", "entity_types": ["minecraft:silverfish"] }
  ],
  "stages": [
    { "stage": 1 }
  ]
}
```

Every entry must be an exact namespaced animation id. One destination may list multiple accepted predecessors. NoN reports malformed entries, self references, and missing predecessor ids as setup warnings.

The rule applies to normal mob joins, player joins, and joins onto defeated animations. It does not restrict direct starts, queued animations, or NoN debug joins.

Chains are strict while at least one declared follower of the current animation is enabled. Only matching declared followers are considered; if they fail actor, block, protector, or other eligibility checks, the join fails instead of selecting an unrelated animation. Weights still choose among the eligible followers using the normal specificity rules.

Disabled followers do not activate a chain. If every declared follower is disabled in the loaded-animation settings, ordinary unrestricted join matching resumes. A definition with `join_after` still cannot follow an unlisted predecessor.

### Stages

Stages are required. Each stage has its own loop, cycle, join, and NoN timing settings.

| Key                             | Type            | Default           | Meaning                                                                                                                                                                                                                                            |
| ------------------------------- | --------------- | ----------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `stage`                         | number          | required          | Stage number, for example `1`.                                                                                                                                                                                                                     |
| `loop`                          | boolean         | `true`            | Whether this stage loops.                                                                                                                                                                                                                          |
| `cycle_seconds`                 | number          | `0`               | GeckoLib cycle length, converted to ticks with `ceil(seconds * 20)`.                                                                                                                                                                               |
| `cycle_midpoint_offset_seconds` | number          | `0.0`             | Loop-only timing warp. Negative moves the cycle midpoint/impact earlier; positive moves it later. Visual playback, sound cues, reactive impacts, and cue-derived NoN effects use the shifted timing. Ignored with a warning on non-looping stages. |
| `allow_join`                    | boolean         | `true`            | Whether NoN joins can expand this stage.                                                                                                                                                                                                       |
| `speed`                         | positive number | top-level `speed` | Stage-specific animation speed.                                                                                                                                                                                                                    |
| `use_stage`                     | number          | self              | Play another stage's GeckoLib animation file while retaining this stage's metadata.                                                                                                                                                                |
| `props`                         | object/null     | inherited         | Stage item prop overrides. `null` clears all props.                                                                                                                                                                                                |
| `non_peak`                      | boolean         | false             | NoN peak marker. The first marked stage is treated as the peak.                                                                                                                                                                                    |
| `escapable`                     | boolean         | top-level/default | Whether the attack escape prompt/bar is active on this stage.                                                                                                                                                                                      |
| `stage_seconds`                 | integer         | top-level/config  | NoN duration override. `-1` is indefinite; normal positive values are clamped to `1..300`.                                                                                                                                                         |
| `stage_duration_multiplier`     | number          | `1.0`             | NoN stage duration multiplier for looping stages. It scales the resolved duration after `stage_seconds`, top-level `stage_seconds`, or config defaults, then cycle alignment is applied. Valid range is `0.05..20.0`.                              |

Example:

```json
{
  "stages": [
    {
      "stage": 1,
      "loop": true,
      "cycle_seconds": 1.2,
      "allow_join": true,
      "stage_seconds": 8,
      "stage_duration_multiplier": 1.0
    },
    {
      "stage": 2,
      "loop": true,
      "cycle_seconds": 0.8,
      "allow_join": false,
      "non_peak": true,
      "escapable": false,
      "stage_seconds": 6,
      "stage_duration_multiplier": 0.5
    }
  ]
}
```

`stage_duration_multiplier` changes how long a looping stage lasts without changing GeckoLib playback speed. Use `speed` when the clip itself should play faster or slower.

Example: if the normal stage setting is `15` seconds and three non-peak stages each use `"stage_duration_multiplier": 0.3333`, each of those stages lasts about `5` seconds before cycle alignment. A peak stage still uses the peak stage setting unless it has its own `stage_seconds` or multiplier.

### Reactive Impact Cues

`reactiveimpact` is the most important sound/effect cue for normal NoN animations. It should be placed on the actual repeated impact/contact moment of a GeckoLib animation cycle.

Even though it lives inside GeckoLib `sound_effects`, it does more than play audio. NoN uses `reactiveimpact` timing for:

- impact sound selection and playback
- player FOV impact pulse
- Buttplug.io reactive impact pulses
- peak-stage impact behavior when the cue happens during the configured `non_peak` stage
- timing liquid gain pulses across the peak stage instead of granting the full amount instantly
- timing affected by `cycle_midpoint_offset_seconds`, so shifted cycles also shift cue-derived NoN behavior

NoN merges all actor sound keyframes for an animation stage into one anchor track. If multiple actor files define a sound at the same second, the first cue is kept and later duplicates at that timestamp are ignored. Because of that, put the important `reactiveimpact` cue on the actor file where it is easiest to maintain, and avoid duplicate cue timestamps across actors.

Example:

```json
{
  "animations": {
    "p1_player": {
      "animation_length": 1.0,
      "sound_effects": {
        "0.25": {
          "effect": "reactiveimpact"
        }
      }
    }
  }
}
```

Use `effect: "reactiveimpact"` for NoN impact behavior. NoN then picks an appropriate impact sound automatically based on the current context.

Do not replace `reactiveimpact` with a direct sound id unless you intentionally want audio only. A cue like `"effect": "needsofnature:impactwet01"` plays sound, but it will not trigger reactive impact mechanics.

For looping stages, cue times are offsets inside one animation cycle. If the stage loops several times, the cue repeats every cycle. For non-looping stages, the cue fires once when that timestamp is reached.

### Other Special Sound Cues

NoN also recognizes these specialized cue effects:

| Cue effect             | Used for                                                                                                   |
| ---------------------- | ---------------------------------------------------------------------------------------------------------- |
| `birth`                | Birth animations. Spawns one offspring/egg at the cue and controls multi-birth stage routing.              |
| `fillbottle_prop_fill` | Fill-bottle animation. Swaps the visual bottle prop to the filled bottle and plays the vanilla fill sound. |

Birth animations are one-actor player animdefs with `animation_tags` containing `birth`. Entity-specific birth animations can use `birth_<entity_id>` where the entity id is lowercased and non-alphanumeric characters are replaced by `_`, for example `birth_minecraft_wolf`.

### Normal NoN Sound Cues

These cues are normal audio cues. They only play sound and do not trigger any special behavior.

Use them as the cue `effect` value, for example:

```json
{
  "effect": "impactdry01"
}
```

You can also use the full sound id form, for example `needsofnature:impactdry01`.

| Cue                             | Meaning                                                                  |
| ------------------------------- | ------------------------------------------------------------------------ |
| `impactdry`                     | Random dry impact selector. Picks one of `impactdry01` to `impactdry11`. |
| `impactwet`                     | Random wet impact selector. Picks one of `impactwet01` to `impactwet16`. |
| `impactdry01` ... `impactdry11` | Specific dry impact sounds.                                              |
| `impactwet01` ... `impactwet16` | Specific wet impact sounds.                                              |
| `wet01` ... `wet12`             | Wet/slosh-style sounds.                                                  |
| `shot_in01` ... `shot_in03`     | Shot-in sounds.                                                          |
| `shot_out01` ... `shot_out06`   | Shot-out sounds.                                                         |
| `motion01` ... `motion10`       | Motion sounds.                                                           |
| `retract01` ... `retract03`     | Retract sounds.                                                          |

NoN also registers `pop`, `rip01`, `rip02` and`rip03` but these are mainly used by built-in NoN systems or debug spin mode rather than normal animation authoring.

### Inline Random Sound Pools

For one-off sound variation, put a comma-separated `random(...)` pool in the sound keyframe's `effect` field. NoN chooses one entry with equal probability each time the cue fires.

```json
{
  "sound_effects": {
    "0.25": {
      "effect": "random(needsofnature:wet02, needsofnature:wet04, needsofnature:impactwet05)",
      "volume": 1.0,
      "pitch": 1.0
    }
  }
}
```

Every entry must be a fully qualified sound event id containing its namespace. The syntax can be entered directly into Blockbench's normal sound-effect field and survives normal GeckoLib and AnimationDirector exports. Inline pools are audio-only and do not trigger special mechanics such as `reactiveimpact`, `birth`, or `fillbottle_prop_fill`.

Random selection happens independently on each client. Use a normal Minecraft `sounds.json` event instead when a pool should be reusable, weighted, or provide subtitles.

### Particle Keyframes

NoN supports GeckoLib `particle_effects` keyframes in actor animation files.

Example:

```json
{
  "animations": {
    "playerf_manual_peak.p1": {
      "animation_length": 1.0,
      "particle_effects": {
        "0.25": {
          "effect": "needsofnature:liquid_particle_falling",
          "locator": "v_locator",
          "pre_effect_script": "color_source=actor"
        }
      }
    }
  }
}
```

| Key                 | Type        | Meaning                                                                                   |
| ------------------- | ----------- | ----------------------------------------------------------------------------------------- |
| `effect`            | particle id | Registered particle type to spawn. NoN supports simple types directly and data-bearing types through integration hooks such as NoN's liquid particles. |
| `locator`           | string      | Optional GeckoLib model locator name. If found, the particle spawns at that posed locator. |
| `pre_effect_script` | string      | Optional particle options interpreted by NoN integrations.                                |

For NoN liquid particles, `pre_effect_script` accepts `color_source=tank` or `color_source=actor`. Omitting it is the same as `tank` and keeps using the player's current liquid-tank color. `actor` uses the configured liquid color of the actor that owns the particle cue; a player actor therefore uses the `minecraft:player` liquid color.

Locators are defined in the GeckoLib model on the bone that should carry the particle position:

```json
{
  "name": "body",
  "pivot": [0, 24, 0],
  "locators": {
    "v_locator": [0, 12, -1]
  }
}
```

If the locator is missing or cannot be resolved, NoN falls back to spawning the particle around the actor body instead of failing the animation.

### Stage Props

Actor-level `prop_left`, `prop_right`, and `prop_floor` define defaults. Stage-level `props` can override or clear them.

Props are primarily meant for player actors and player prop bones. They are technically parsed for any actor, but entity models usually do not expose compatible prop bones unless the model was authored for that.

The three fields render on the `propleft`, `propright`, and `propfloor` GeckoLib bones respectively. Floor props use Minecraft's ground item-display transform; animate or rotate the `propfloor` bone when a different authored orientation is needed.

```json
{
  "actors": [
    {
      "label": "player",
      "entity_types": ["minecraft:player"],
      "prop_right": "minecraft:stick",
      "prop_floor": "minecraft:lantern"
    }
  ],
  "stages": [
    {
      "stage": 1,
      "props": {
        "player": {
          "prop_right": "minecraft:carrot",
          "prop_floor": null
        }
      }
    },
    {
      "stage": 2,
      "props": {
        "player": null
      }
    },
    {
      "stage": 3,
      "props": null
    }
  ]
}
```

`"player": null` clears that actor's props. `"props": null` clears all props for that stage.

### Manual Peak Held-Item Rules

Manual peak candidates are normal animdefs with `animation_tags` containing `manualpeak`.

Add `manual_peak` for held-item-specific variants:

```json
{
  "animation_tags": ["manualpeak"],
  "manual_peak": {
    "held_items": ["minecraft:carrot", "minecraft:golden_carrot"],
    "priority": 10,
    "prop_from_held_item": true,
    "held_item_prop_slot": "right"
  }
}
```

| Key                      | Type           | Default | Meaning                                                                                 |
| ------------------------ | -------------- | ------- | --------------------------------------------------------------------------------------- |
| `held_item`              | item id string | none    | Single required held item.                                                              |
| `held_items`             | item id array  | none    | Any listed held item can match.                                                         |
| `priority`               | integer        | `0`     | Higher-priority manual peak rules are selected first.                                   |
| `prop_from_held_item`    | boolean        | `false` | If true, the exact matched held item is rendered as an item prop.                       |
| `held_item_prop_slot`    | string         | `right` | Prop destination: `left` (`propleft`), `right` (`propright`), or `floor` (`propfloor`). |

`held_item_prop_slot` is independent of the hand containing the matching item. For example, a carrot matched in the player's offhand still renders on `propfloor` when the rule selects `"floor"`. An invalid slot produces a setup warning and disables the dynamic held-item prop for that definition without disabling the animation itself.

If no held-item rule matches, NoN falls back to a normal manual peak animation.

### Wall Block Requirements

Wall requirements look for a wall of a given height and a directional clearance area in front of it.

```json
{
  "block_requirements": {
    "type": "wall",
    "height": {
      "min": 2,
      "max": 3
    },
    "clearance": {
      "width": 1,
      "height": 2,
      "depth": 2
    }
  }
}
```

`height` can also be an exact integer:

```json
{
  "block_requirements": {
    "type": "wall",
    "height": 2
  }
}
```

`clearance.width` means one block to the left and right of the directional clearance line. All clearance values are clamped to at least `1`.

#### Example: Low Wall Requirement

Example animdef block for an animation that needs a low wall:

```json
{
  "block_requirements": {
    "type": "wall",
    "height": {
      "min": 1
    },
    "clearance": {
      "width": 1,
      "height": 2,
      "depth": 2
    }
  }
}
```

This means NoN searches for a full block wall at least one block high. A valid start position must also have a free directional clearance area in front of that wall. With `width: 1`, the clearance checks the forward line plus one block to the left and one block to the right. With `depth: 2` and `height: 2`, that clear area must be two blocks deep and two blocks tall.

This shows a valid block alignment. Red glass indicates the spaces that must stay free for the directional clearance check. Orange glass indicates the animation anchor position.

![Valid low wall block alignment](img/block_requirements/wall_block.png)

<br>

This shows the same wall requirement using a fence instead of a full block. For wall requirements both fences and walls are supported and will move the animations slightly for better alignment. Red glass indicates the spaces that must stay free for the directional clearance check. Orange glass indicates the animation anchor position.

![Valid low wall fence alignment](img/block_requirements/wall_fence.png)

<br>

### Center Support Block Requirements

Center support requirements anchor an animation inside a selected support block. This is useful for animations that should happen on top of, inside of, or against a specific surface such as a block, slab, bed, or custom block.

There are two placement modes:

| Placement               | Meaning                                                                            |
| ----------------------- | ---------------------------------------------------------------------------------- |
| `directional_clearance` | Default. Requires free space next to the support block, similar to wall clearance. |
| `surface_footprint`     | Requires a same-height support surface with a literal width/depth footprint.       |

Supported `support` values:

| Support       | Meaning                                                                                                      |
| ------------- | ------------------------------------------------------------------------------------------------------------ |
| `full_block`  | Full-height block support. Top slabs and double slabs count as full blocks.                                  |
| `slab`        | Bottom slabs only. Top slabs and double slabs do not count.                                                  |
| `half_height` | Bottom slabs and beds. Beds and bottom slabs are treated as compatible for same-height footprint checks.     |
| `surface`     | Custom support based on a block id or block tag in `blocks`. Required when targeting beds only, chests, etc. |

Use `support: "surface"` with `blocks` when the animation should only run on specific blocks:

```json
{
  "block_requirements": {
    "type": "center_support",
    "support": "surface",
    "blocks": "#minecraft:beds",
    "placement": "surface_footprint",
    "surface_footprint": {
      "width": 2,
      "depth": 2,
      "height": 2
    }
  }
}
```

`blocks` accepts namespaced block ids or block tags:

```json
"blocks": "minecraft:chest"
```

```json
"blocks": ["minecraft:chest", "minecraft:trapped_chest"]
```

```json
"blocks": "#minecraft:beds"
```

#### Directional Clearance Center Support

This is the older/default center-support mode. It finds a support block and then checks for free directional space next to it.

```json
{
  "block_requirements": {
    "type": "center_support",
    "support": "slab",
    "placement": "directional_clearance",
    "clearance": {
      "width": 0,
      "height": 2,
      "depth": 2
    },
    "surface_radius": 1
  }
}
```

`placement` can be omitted here because `directional_clearance` is the default.

| Key              | Type    | Meaning                                                                                                |
| ---------------- | ------- | ------------------------------------------------------------------------------------------------------ |
| `clearance`      | object  | Directional free-space check next to the support, similar to wall clearance.                           |
| `surface_radius` | integer | Checks surrounding blocks around the support for compatible surface height and vertical air clearance. |

This means NoN searches for a bottom slab and anchors the animation inside that slab instead of above it. The directional clearance still needs a free space next to the slab; with `width: 0`, it only checks the selected forward line, and with `depth: 2`, that line must be two blocks deep. `surface_radius: 1` checks the 3x3 area around the support so nearby blocks are compatible with the slab surface height and the required air above it is clear.

This shows a valid slab surface where the surrounding blocks are also bottom slabs. Red glass indicates the spaces that must stay free for the directional clearance and vertical air checks. Orange glass indicates the animation anchor position.

![Valid bottom-slab alignment with surrounding slabs](img/block_requirements/slab_surrounded.png)

<br>
This shows a valid slab support where the surrounding blocks are air. Red glass indicates the spaces that must stay free for the directional clearance and vertical air checks. Orange glass indicates the animation anchor position.

![Valid bottom-slab alignment with surrounding air](img/block_requirements/slab_air.png)

<br>

#### Surface Footprint Center Support

Surface footprint mode does not use directional clearance. Instead, it requires a same-height support surface under the animation.

```json
{
  "block_requirements": {
    "type": "center_support",
    "support": "half_height",
    "placement": "surface_footprint",
    "surface_footprint": {
      "width": 2,
      "depth": 2,
      "height": 2,
      "margin": 1
    }
  }
}
```

| Key                        | Type    | Default | Meaning                                                    |
| -------------------------- | ------- | ------- | ---------------------------------------------------------- |
| `surface_footprint.width`  | integer | `1`     | Literal footprint width in blocks.                         |
| `surface_footprint.depth`  | integer | `1`     | Literal footprint depth in blocks.                         |
| `surface_footprint.height` | integer | `2`     | Required empty vertical space above every footprint block. |
| `surface_footprint.margin` | integer | `0`     | Extra free-space border around the footprint.              |

The selected support block is one corner of the footprint. NoN tries all four forward directions and both side directions, so even footprints such as `2x2` can be found from any corner.

Footprint cells must be valid support blocks of the same support category and same normalized support height. Margin cells do not need support blocks, but they must be empty and fluid-free from the support surface height upward.

##### Example: 2x2 Half-Height Footprint

This example accepts a 2x2 area made from bottom slabs, beds, or a compatible mixture of both.

```json
{
  "block_requirements": {
    "type": "center_support",
    "support": "half_height",
    "placement": "surface_footprint",
    "surface_footprint": {
      "width": 2,
      "depth": 2,
      "height": 2
    }
  }
}
```

This shows a valid 2x2 half-height footprint. Bottom slabs and beds may be used. Orange glass indicates the animation anchor position.

![Valid 2x2 half-height footprint](img/block_requirements/surface_footprint_half_height_2x2.png)

<br>

This shows a valid mixed 2x2 half-height footprint using two bottom slabs and one bed. This shows that `half_height` allows bottom slabs and beds to share the same required surface. Orange glass indicates the animation anchor position.

![Valid mixed 2x2 half-height footprint](img/block_requirements/surface_footprint_half_height_2x2_mixed.png)

<br>

##### Example: Bed-Only Footprint

```json
{
  "block_requirements": {
    "type": "center_support",
    "support": "surface",
    "blocks": "#minecraft:beds",
    "placement": "surface_footprint",
    "surface_footprint": {
      "width": 2,
      "depth": 2,
      "height": 2,
      "margin": 0
    }
  }
}
```

This shows a valid bed-only 2x2 footprint. Orange glass indicates the animation anchor position.

![Valid bed-only 2x2 footprint](img/block_requirements/surface_footprint_beds_2x2.png)

<br>

##### Example: 2x2 Half-Height Footprint With Margin

This example requires the same 2x2 surface, but also needs one block of free space around the footprint.

```json
{
  "block_requirements": {
    "type": "center_support",
    "support": "half_height",
    "placement": "surface_footprint",
    "surface_footprint": {
      "width": 2,
      "depth": 2,
      "height": 2,
      "margin": 1
    }
  }
}
```

This shows a valid 2x2 half-height footprint with a one-block margin. Red glass indicates the spaces that must stay free around the whole footprint. Orange glass indicates the animation anchor position.

![Valid 2x2 half-height footprint with margin](img/block_requirements/surface_footprint_half_height_2x2_margin.png)

<br>

### Water Requirements

```json
{
  "water": "surface"
}
```

Valid values:

| Value        | Meaning                                    |
| ------------ | ------------------------------------------ |
| `none`       | No water requirement.                      |
| `surface`    | Requires suitable water surface placement. |
| `underwater` | Requires suitable underwater placement.    |

### GeckoLib Animation Assets

NoN's primary animation format stores every stage and actor clip for one animation in a single conjoined file:

```text
assets/<namespace>/geckolib/animations/afw/<animation_id>.animation.json
```

For animation ID `mymod:wolfmplayer`, the file is:

```text
assets/mymod/geckolib/animations/afw/wolfmplayer.animation.json
```

The filename is the only place that stores `wolfmplayer`. Keys inside the `animations` object use the strict `p<stage>_<actorKey>` format:

```json
{
  "format_version": "1.8.0",
  "animations": {
    "p1_wolf": {
      "loop": true,
      "animation_length": 1.0,
      "bones": {}
    },
    "p1_player": {
      "loop": true,
      "animation_length": 1.0,
      "bones": {}
    },
    "p2_wolf": {
      "animation_length": 2.0,
      "bones": {}
    },
    "p2_player": {
      "animation_length": 2.0,
      "bones": {}
    }
  }
}
```

Do not repeat the animation ID in these keys. For example, `wolfmplayer.p1_wolf` is invalid in a conjoined file. The actor key should match the corresponding actor key in the animdef whenever possible. NoN may also resolve an actor clip by its entity type path when no direct actor-key clip exists.

### CEM/JEM Model Assets

NoN packs can include OptiFine/EMF-style CEM model files under:

```text
assets/minecraft/optifine/cem/<entity>.jem
```

NoN generates the matching CEM `.properties` rules for gender selection when a pack provides `.jem` files without its own explicit `.properties` file.

Gender allocation convention:

| File            | Used for                                                       |
| --------------- | -------------------------------------------------------------- |
| `<entity>.jem`  | Primary model. Used for male and male+female entities.         |
| `<entity>2.jem` | Female model. Used only for female entities.                   |

Examples:

```text
assets/minecraft/optifine/cem/wolf.jem   -> male/primary wolf model
assets/minecraft/optifine/cem/wolf2.jem  -> female wolf model
assets/minecraft/optifine/cem/horse.jem  -> male/primary horse model
assets/minecraft/optifine/cem/horse2.jem -> female horse model
```

If you provide your own `<entity>.properties`, NoN does not generate rules for that entity and your properties file controls selection. 

If only a primary model is provided, for example `wolf.jem`, it only targets male and male+female entities. Female entities need a matching `wolf2.jem` if they should use a custom model.

### Defeated Animations

Defeated animations are one-actor player animdefs tagged by pose:

```json
{
  "actors": [
    {
      "label": "player",
      "entity_types": ["minecraft:player"]
    }
  ],
  "animation_tags": ["defeat.on_belly"],
  "content_tags": ["doggy", "from_behind", "on_belly"],
  "stages": [
    {
      "stage": 1,
      "loop": true,
      "cycle_seconds": 1.0,
      "allow_join": true,
      "stage_seconds": -1,
      "escapable": true
    }
  ]
}
```

Use `defeat.on_back` for back-facing defeated animations. For example, the default back pose accepts `content_tags` of `missionary` and `on_back`.

NoN compares the source and defeated definitions using the content-tag rules described under [Content Tags](#content-tags). This relationship is owned by the defeated animation, so adding a new defeated pose does not require editing every source animation.

### Liquid Gain Overrides

Liquid gain overrides are server-data JSON files:

```text
data/<namespace>/non_liquid_gains/<file>.json
```

Example:

```json
{
  "mixed_color": "#f2ebbf",
  "entries": [
    {
      "entity": "minecraft:wolf",
      "gain_ml": 25,
      "color": "#d6c3a0"
    },
    {
      "entity": "minecraft:horse",
      "gain_ml": 40,
      "color": "#c0a070"
    }
  ]
}
```

| Key           | Type             | Meaning                                      |
| ------------- | ---------------- | -------------------------------------------- |
| `mixed_color` | `#RRGGBB` string | Optional global mixed-liquid color override. |
| `entries`     | array            | Entity-specific liquid gain/color entries.   |
| `entity`      | entity id        | Entity type id.                              |
| `gain_ml`     | integer          | Liquid gain amount, clamped to `0..1000`.    |
| `color`       | `#RRGGBB` string | Entity liquid color.                         |

Files are processed in id order. Later entries can overwrite earlier entries for the same entity.

### Entity Profiles

Entity profiles are server-data JSON files for per-entity gameplay defaults:

```text
data/<namespace>/non_entity_profiles/<file>.json
```

Example:

```json
{
  "entries": [
    {
      "entity": "minecraft:spider",
      "pregnancy_chance_percent": 5,
      "offspring_min": 1,
      "offspring_max": 3,
      "birth_entity": "minecraft:spider",
      "birth_mode": "egg",
      "energy_gain_multiplier": 1.5,
      "multi_actor_join_chance_percent": 100,
      "egg": {
        "start_size": 0.5,
        "end_size": 1.0,
        "texture": "needsofnature:textures/entity/pregnancy_egg/minecraft_spider.png",
        "health": 2.0
      },
      "gender_spawn": {
        "male_chance": 100,
        "female_chance": 0,
        "both_chance": 0
      }
    },
    {
      "entity": "minecraft:enderman",
      "pregnancy_chance_percent": 8,
      "offspring_min": 1,
      "offspring_max": 1,
      "birth_entity": "minecraft:endermite",
      "birth_mode": "direct"
    }
  ]
}
```

| Key                        | Type           | Meaning                                                                 |
| -------------------------- | -------------- | ----------------------------------------------------------------------- |
| `entries`                  | array          | Entity-specific profile entries.                                        |
| `entity`                   | entity id      | Entity type that caused the pregnancy or receives gender spawn chances. |
| `pregnancy_chance_percent` | integer        | Optional per-entity pregnancy chance override, clamped to `0..100`.     |
| `offspring_min`            | integer        | Minimum offspring count, clamped to `1..16`.                            |
| `offspring_max`            | integer        | Maximum offspring count, raised to at least `offspring_min`.            |
| `birth_entity`             | entity id      | Entity type spawned by the birth. Omit to use `entity`.                 |
| `birth_mode`               | `egg`/`direct` | `egg` spawns a NoN egg; `direct` spawns the entity immediately.         |
| `energy_gain_multiplier`   | number         | Passive energy buildup multiplier. Valid range: `0.1..10.0`.            |
| `multi_actor_join_chance_percent` | integer | Optional override for this mob's 3+ actor join roll, clamped to `0..100`. |
| `egg`                      | object         | Optional egg settings used when `birth_mode` is `egg`.                  |
| `egg.start_size`           | number         | Egg render and hitbox scale at spawn, clamped to `0.05..4.0`.           |
| `egg.end_size`             | number         | Egg render and hitbox scale when fully grown, clamped to `0.05..4.0`.   |
| `egg.texture`              | texture id     | Egg texture. Omit or leave empty to use the default NoN egg texture.    |
| `egg.health`               | number         | Egg health, clamped to `0.5..100.0`.                                    |
| `gender_spawn`             | object         | Optional gender spawn chances for this entity.                          |
| `male_chance`              | integer        | Male-only spawn percentage.                                             |
| `female_chance`            | integer        | Female-only spawn percentage.                                           |
| `both_chance`              | integer        | Both-gender spawn percentage.                                           |

If `pregnancy_chance_percent` is omitted, NoN uses the default pregnancy chance from the normal NoN settings. If it is present, it overrides that default for this entity.

If `energy_gain_multiplier` is omitted, it defaults to `1.0`. It multiplies passive energy buildup, including nearby-animation and player-aura acceleration. It does not modify energy granted directly by commands, items, or gameplay events.

If `multi_actor_join_chance_percent` is omitted, NoN uses the global **Join chance (3+ actors)** setting. Set it to `100` to make this entity always pass that probability roll. Matching, block requirements, join permissions, and gathering still apply normally.

`male_chance`, `female_chance`, and `both_chance` must total exactly `100`.

Files are processed in id order. Later entries can overwrite earlier entries for the same entity.

---

## Accessories

### Data-Driven Accessories

Accessory item JSON files are loaded from:

```text
assets/needsofnature/non_accessory_items/<item>.json
```

The namespace must be `needsofnature`. These files create item registry entries at startup, so a game restart is required after adding or removing them.

Accessories require Trinkets to be installed. If Trinkets is missing, NoN skips accessory item registration and continues without the accessory system.

Example:

```json
{
  "id": "iron_plug",
  "max_count": 1,
  "show_in_item_group": true,
  "trinkets_slots": ["legs/v", "legs/a"],
  "skin_overlays": {
    "v": "needsofnature:item/iron_plug_v",
    "a": "needsofnature:item/iron_plug_a"
  },
  "effects": {
    "liquid_decay_multiplier": {
      "value": 0.25,
      "tooltip": "positive"
    },
    "equalize_liquid_decay_context": {
      "value": true,
      "tooltip": "positive"
    },
    "player_energy_gain_multiplier": {
      "value": 3.0,
      "tooltip": "negative"
    }
  },
  "item_texture": "needsofnature:item/iron_plug"
}
```

### Accessory Item Keys

| Key                                       | Type         | Default               | Meaning                                                                                        |
| ----------------------------------------- | ------------ | --------------------- | ---------------------------------------------------------------------------------------------- |
| `id`                                      | item id/path | required              | Item id. Namespace defaults to `needsofnature`; other namespaces are rejected.                 |
| `max_count`                               | integer      | `1`                   | Stack size, clamped to `1..64`. Durability forces this to `1`.                                 |
| `max_durability`                          | integer      | `0`                   | Max item durability, clamped to `0..100000`.                                                   |
| `show_in_item_group`                      | boolean      | `true`                | Whether the item appears in the NoN creative tab.                                              |
| `trinkets_slots`                          | string/array | `["legs/v","legs/a"]` | Slots this item can be equipped in.                                                            |
| `occupies_slots`                          | string/array | empty                 | Extra slots blocked while this item is equipped.                                               |
| `skin_overlays`                           | object       | empty                 | Optional player skin overlay textures for V/A/D slots.                                         |
| `effects`                                 | object       | neutral               | Accessory stat modifiers.                                                                      |
| `blocks_injector_types`                   | string/array | empty                 | `V`, `A`, or `M` injector types blocked by this item.                                          |
| `exclusive_group`                         | string       | none                  | Prevents equipping multiple items in the same group.                                           |
| `protection_durability_cost`              | integer      | `1`                   | Durability loss when protection blocks an animation.                                           |
| `ignore_injector_slot_visual_shedding`    | boolean      | `false`               | Keeps overlay visible during matching injector animations. Pregnancy can still force shedding. |
| `ignore_injector_slot_effect_suppression` | boolean      | `false`               | Keeps effects active during matching injector animations.                                      |
| `v_injection_durability_cost`             | integer      | `0`                   | Durability loss on V injection.                                                                |
| `a_injection_durability_cost`             | integer      | `0`                   | Durability loss on A injection.                                                                |
| `blocks_pregnancy`                        | boolean      | `false`               | Blocks pregnancy behavior while active.                                                        |
| `item_texture`                            | asset id     | none                  | Simple generated item model texture.                                                           |
| `item_model`                              | asset id     | generated             | Optional custom item model id.                                                                 |

Valid trinket slot names:

```text
legs/v
legs/a
legs/d
v
a
d
```

Short names normalize to `legs/<slot>`.

Slot availability:

| Slot | Available for                   |
| ---- | ------------------------------- |
| `V`  | Female and female+male players. |
| `A`  | All player genders.             |
| `D`  | Male and female+male players.   |

### Accessory Effects

Each effect can be written as a raw value:

```json
{
  "effects": {
    "liquid_capacity_add": 100
  }
}
```

or as an object with an explicit tooltip color:

```json
{
  "effects": {
    "liquid_capacity_add": {
      "value": 100,
      "tooltip": "positive"
    }
  }
}
```

Tooltip color values:

| Value      | Color      |
| ---------- | ---------- |
| `positive` | green      |
| `negative` | red        |
| `neutral`  | white      |
| `#RRGGBB`  | custom RGB |

Supported effect keys:

| Effect                                 | Type    | Clamp           | Meaning                                                  |
| -------------------------------------- | ------- | --------------- | -------------------------------------------------------- |
| `liquid_decay_multiplier`              | number  | `0..20`         | Multiplies liquid decay.                                 |
| `equalize_liquid_decay_context`        | boolean | n/a             | Removes sneaking/water decay speed changes while active. |
| `player_energy_gain_multiplier`        | number  | `0..20`         | Multiplies player energy buildup.                        |
| `liquid_capacity_add`                  | integer | `-10000..10000` | Adds liquid tank capacity.                               |
| `liquid_gain_multiplier`               | number  | `0..20`         | Multiplies liquid gained from animations.                |
| `filled_effect_multiplier`             | number  | `0..20`         | Multiplies filled penalty/effect behavior.               |
| `pregnancy_chance_multiplier`          | number  | `0..20`         | Multiplies pregnancy chance.                             |
| `pregnancy_duration_multiplier`        | number  | `0.01..20`      | Multiplies pregnancy duration.                           |
| `mess_gain_multiplier`                 | number  | `0..20`         | Multiplies mess gain.                                    |
| `destroyed_skin_damage_multiplier`     | number  | `0..20`         | Multiplies destroyed-skin damage.                        |
| `attack_escape_hits_add`               | integer | `-49..49`       | Adds required attack escape hits.                        |
| `attack_escape_damage_multiplier`      | number  | `0..20`         | Multiplies escape damage.                                |
| `player_energy_aura_multiplier`        | number  | `0..20`         | Multiplies player energy aura.                           |
| `near_animation_mob_energy_multiplier` | number  | `0..20`         | Multiplies near-animation mob energy gain.               |

Liquid tank and pregnancy effects are only active from the correct active tank slot. Non-tank effects can still apply from other valid slots.

### Accessory Skin Overlays

`skin_overlays` maps slot letters to texture ids:

```json
{
  "skin_overlays": {
    "v": "needsofnature:item/my_accessory_v",
    "a": "needsofnature:item/my_accessory_a",
    "d": "needsofnature:item/my_accessory_d"
  }
}
```

The value is converted to a texture path. This example resolves to:

```text
assets/needsofnature/textures/item/my_accessory_v.png
```

For an A+V item using one shared overlay, point both entries at the same texture:

```json
{
  "skin_overlays": {
    "v": "needsofnature:item/my_dual_slot_accessory_overlay",
    "a": "needsofnature:item/my_dual_slot_accessory_overlay"
  }
}
```

### Animation-Blocking Accessory Example

```json
{
  "id": "my_dual_slot_accessory",
  "max_count": 1,
  "max_durability": 8,
  "show_in_item_group": true,
  "trinkets_slots": ["legs/v", "legs/a"],
  "occupies_slots": ["legs/v", "legs/a"],
  "skin_overlays": {
    "v": "needsofnature:item/my_dual_slot_accessory_overlay",
    "a": "needsofnature:item/my_dual_slot_accessory_overlay"
  },
  "blocks_injector_types": ["V", "A"],
  "exclusive_group": "slot_blocker",
  "protection_durability_cost": 1,
  "item_texture": "needsofnature:item/my_dual_slot_accessory"
}
```

### Recipes and Advancements

Recipes use normal Minecraft data-pack paths:

```text
data/<namespace>/recipe/<recipe_id>.json
```

Recipe unlock advancements use:

```text
data/<namespace>/advancement/recipes/<category>/<recipe_id>.json
```

Example shaped accessory recipes based on the default pack's slot-blocking accessory patterns:

Single-slot accessory variant with iron ingots and a gold nugget:

```json
{
  "fabric:load_conditions": [
    {
      "condition": "fabric:all_mods_loaded",
      "values": [
        "trinkets"
      ]
    }
  ],
  "type": "minecraft:crafting_shaped",
  "category": "misc",
  "pattern": [
    "IGI",
    "III",
    " I "
  ],
  "key": {
    "I": "minecraft:iron_ingot",
    "G": "minecraft:gold_nugget"
  },
  "result": {
    "id": "needsofnature:my_slot_accessory"
  }
}
```

Single-slot accessory variant with iron ingots and a copper nugget:

```json
{
  "fabric:load_conditions": [
    {
      "condition": "fabric:all_mods_loaded",
      "values": [
        "trinkets"
      ]
    }
  ],
  "type": "minecraft:crafting_shaped",
  "category": "misc",
  "pattern": [
    "ICI",
    "III",
    " I "
  ],
  "key": {
    "I": "minecraft:iron_ingot",
    "C": "minecraft:copper_nugget"
  },
  "result": {
    "id": "needsofnature:my_other_slot_accessory"
  }
}
```

Dual-slot accessory variant with gold ingots and an emerald:

```json
{
  "fabric:load_conditions": [
    {
      "condition": "fabric:all_mods_loaded",
      "values": [
        "trinkets"
      ]
    }
  ],
  "type": "minecraft:crafting_shaped",
  "category": "misc",
  "pattern": [
    "GEG",
    "GGG",
    " G "
  ],
  "key": {
    "G": "minecraft:gold_ingot",
    "E": "minecraft:emerald"
  },
  "result": {
    "id": "needsofnature:my_dual_slot_accessory"
  }
}
```

---

## Addon API

Addon mods compile against NeedsOfNature directly. There is no separate Animation Framework API dependency and no `com.afwid.api` compatibility layer. Public API types live under `com.nonid.api`; classes under `com.nonid.internal` are implementation details and may change.

Server-side mutation and animation-control methods must run on the Minecraft server thread. Client event classes under `com.nonid.api.client` must be registered from a client initializer.

### Player State

`NonPlayerApi` exposes immutable snapshots and controlled mutations for NoN player systems:

```java
NonPlayerStateSnapshot state = NonPlayerApi.getState(player);
NonGender gender = NonPlayerApi.getGender(player);
NonLiquidTankSnapshot tank = NonPlayerApi.getLiquidTank(player);
NonMessSnapshot mess = NonPlayerApi.getMess(player);
NonDestroyedSkinSnapshot rippedSkin = NonPlayerApi.getDestroyedSkin(player);
NonPregnancySnapshot pregnancy = NonPlayerApi.getPregnancy(player);
NonAccessoryEffectsSnapshot accessories = NonPlayerApi.getAccessoryEffects(player);
```

Server-side methods include `setGender`, `addLiquid`, `drainLiquid`, `addMess`, `cleanMess`, `clearMess`, `damageDestroyedSkin`, `repairDestroyedSkin`, `beginPregnancy`, and `clearPregnancy`. These methods apply NoN's normal clamping, feature toggles, synchronization, and public events.

`NonEntityApi` provides energy access for any supported living entity:

```java
if (NonEntityApi.hasEnergy(entity)) {
    NonEnergySnapshot energy = NonEntityApi.getEnergy(entity);
    NonEntityApi.addEnergy(entity, 10);
}
```

`NonEvents` contains server-side hooks for gender, liquid tank, mess, ripped skin, and pregnancy changes. It also exposes `MODIFY_LIQUID_GAIN` and `ALLOW_PREGNANCY` decision hooks.

### Animation Queries and Control

`NonAnimationApi` is the public server-side entry point for the bundled animation engine. It exposes immutable definition, pack, stage, actor, block-requirement, candidate, and active-session views.

Common queries:

```java
List<NonAnimationDefinition> definitions = NonAnimationApi.getLoadedDefinitions();
NonAnimationDefinition definition = NonAnimationApi.getDefinition(animationId);
boolean enabled = NonAnimationApi.isEnabled(animationId);

NonAnimationCandidate match = NonAnimationApi.findMatch(
        world,
        List.of(firstActor, secondActor),
        NonAnimationApi.MatchOptions.startEligible(firstActor)
);
```

`findMatch` and `findCandidates` use the same actor constraints, weights, enabled-state checks, tags, block requirements, water requirements, and placement rules as NoN. `MatchOptions` can require animation tags, an active actor, a predecessor animation, exclusions, or an addon-provided candidate filter.

Starting a matched animation:

```java
UUID instanceId = NonAnimationApi.startMatched(
        world,
        List.of(firstActor, secondActor),
        NonAnimationApi.MatchOptions.startEligible(firstActor),
        NonAnimationApi.StartOptions.defaults()
);
```

For explicit control, use `start`, `canStart`, `getSession`, `getCurrentStage`, `advanceStage`, `advanceToStage`, `multiplySpeed`, `stop`, `stopAll`, `enqueue`, and `clearQueue`. Start options control damage behavior, attacker ignoring, placement anchor, requester, metadata, debug chat, and whether multiple player actors are allowed.

`NonAnimationEvents` provides `BEFORE_START`, `STARTED`, `STAGE_CHANGED`, and `STOPPED`. `BEFORE_START` is a veto hook; returning `false` prevents the instance from starting before actors enter their animation state.

```java
NonAnimationEvents.BEFORE_START.register(context -> {
    return !context.metadata().containsKey("my_addon:block_start");
});

NonAnimationEvents.STARTED.register(context -> {
    UUID instanceId = context.session().instanceId();
});
```

`NonActorEvents.PROVIDE` lets addons contribute temporary actor tags without mutating command tags:

```java
NonActorEvents.PROVIDE.register(entity -> {
    return entity.hasStatusEffect(myEffect) ? Set.of("my_addon:affected") : Set.of();
});
```

### Player Interaction Profiles

`NonPlayerInteractionApi` resolves and updates per-world player request/attack profiles. It honors profiles disabled by the host and synchronizes mutations back to the owning player's profile screen.

```java
boolean mayRequest = NonPlayerInteractionApi.canRequest(requester, target);
boolean mayAttack = NonPlayerInteractionApi.canAttack(attacker, target);

NonPlayerInteractionApi.setOverride(
        target,
        requester,
        NonPlayerInteractionApi.Mode.REQUESTS_ONLY
);
```

Available modes are `DENY_ALL`, `REQUESTS_ONLY`, `REQUESTS_AND_ATTACKS`, and `AUTO_ACCEPT`.

### Client Animation Hooks

Client-only extension points are:

| Class | Hook |
| --- | --- |
| `NonAnimationRenderEvents` | Override the GeckoLib model, base texture, layers, per-bone textures/items/visibility, or hidden cube indices. |
| `NonAnimationSoundEvents` | Replace, modify, or suppress resolved animation sound cues. |
| `NonAnimationParticleEvents` | Replace or suppress animation particle keyframes after locator position resolution. |

Register these only from client code. Do not reference client API classes from a dedicated-server initializer.

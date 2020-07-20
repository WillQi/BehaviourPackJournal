# Behaviour Pack Journal

## Projectiles - July 19 2020

Recently, I wanted to create an arrow that does less damage than the vanilla arrow for my project. It wasn't too bad to figure out how to add a new projectile. Hopefully this will help you or the future version of myself when I come back to understand it.

In terms of the resource pack content, it's essentially the same as any normal entity implementation. Nothing really changes here.
But if we're talking about the behaviour pack, our projectile needs a `minecraft:projectile` component. Below, I pasted my modified arrow entity definition file. (For the most part, it's basically the original arrow, the key difference is that this arrow will set you on fire and everyone loves being set on fire... right?)

```json
{
  "format_version": "1.16.0",
  "minecraft:entity": {
      "description": {
          "identifier": "gamewq:modified_arrow",
          "is_summonable": true,
          "is_spawnable": false,
          "is_experimental": true,
          "runtime_identifier": "minecraft:arrow"
      },
      "components": {
          "minecraft:projectile": {
              "catch_fire": true,
              "destroy_on_hurt": true,
              "knockback": true,
              "hit_sound": "bow.hit",
              "on_fire_time": 3,
              "power": 1.6,
              "gravity": 0.05,
              "uncertainty_base": 40,
              "on_hit": {
                  "impact_damage": {
                      "filter": "player",
                      "damage": 1.25,
                      "knockback": true
                  },
                  "remove_on_hit": {},
                  "particle_on_hit": {}
              }
          }
      }
  }
}
```

I found that the `on_hit` property isn't documented for 1.16, I'm not sure why, but afaik, you do need it for enemies to attack. I'd like to research a bit more about that as I don't really want to use anything that isn't supported anymore unless I have to.

In terms of the accuracy of the shooter, `uncertainty_base` and `uncertainty_multiplier` are the properties you need to modify. (For the longest time, I kept looking around for a component that modified accuracy in the shooter's entity definition file LOL) The accuracy is calculated as `uncertainity_base - uncertainty_multiplier * difficulty_level`. The higher it is, the better the accuracy.

Something that really stumped me and got me upset was this bug where my projectile would incorrectly push the player when it hit them. (along with the projectile not pointing in the right direction) This is where `runtime_identifier` is crucial. This single line fixed that problem. I'm not sure as to why it did, but I'm assuming there's something hardcoded somewhere in the game's engine? I found that even if I had the exact same behaviour and resource pack data as the original arrow (minus the identifier being different) the game would still have this bug. So it's very likely something is hardcoded. (Either that or I'm missing something)

To make an entity actually shoot the projectile, you just need to add the `minecraft:shooter` component, your ranged attack behaviour, and a behaviour to target a entity.

```json
{
    "format_version": "1.16.0",
    "minecraft:entity": {
        "description": {
            "identifier": "gamewq:basic_skeleton",
            "is_summonable": true,
            "is_spawnable": true,
            "is_experimental": true
        },
        "components": {

            "minecraft:shooter": {
                "def": "gamewq:modified_arrow"
            },

            "minecraft:behavior.ranged_attack": {
                "priority": 0,
                "attack_interval_min": 2,
                "attack_interval_max": 3,
                "attack_radius": 15
            },
            "minecraft:behavior.nearest_attackable_target": {
                "entity_types": [
                    {
                        "filters": {
                            "all_of": [
                                {
                                    "test": "is_family",
                                    "subject": "other",
                                    "value": "player"
                                }
                            ]
                        },
                        "max_dist": 16
                    }
                ],
                "must_see": true,
                "priority": 1
            },

            "minecraft:physics": {},
            "minecraft:movement": {
                "value": 0.25
            },
            "minecraft:movement.generic": {},
            "minecraft:navigation.walk": {},
            "minecraft:can_climb": {},
            "minecraft:jump.static": {},
        }
    }
}
```

\- William Qi

[[Home]](../index.md) [[Prev]](./Experimenting-with-Behaviour-Animations-July-8-2020.md)
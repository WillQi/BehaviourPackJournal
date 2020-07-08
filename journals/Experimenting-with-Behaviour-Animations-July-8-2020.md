# Behaviour Pack Journal

## Experimenting with Behaviour Animations - July 8 2020

For one of the projects I'm working on, I wanted to find a way to have an entity slowly fall when they spawn and then stay in midair after a few seconds. Fortunately, this is possible with behaviour packs using behaviour animation controllers and animations.
For the most part they work just like resource packs, because animation controllers go in the `animation_controllers` directory and animations go in the `animations` directory. The only differences are the content of the files and how to implement it in the behavioural entity definition file.

What's neat about behaviour animation controllers and animations is how you can assign entities to execute commands or events when they enter the animation or somewhere during the animations. This is great news for me because I can use `/effect @s slow_falling 5 1 true` to create the slow falling effect, and I can trigger a entity event after a few seconds of falling to remove the component group with `minecraft:physics` which should stop the entity from falling.

### Behavioural Animation and Animation Controllers

The animation controller is pretty the same for both resource pack and behaviour packs. The only difference would be that you can trigger commands and events within the state object using the `on_entry` and `on_exit` properties.

```json
{
    "format_version": "1.10.0",
    "animation_controllers": {
        "controller.animation.controller_name": {
            "initial_state": "default",
            "states": {
                "default": {
                    "animations": [
                        // any animations to trigger
                    ],
                    "on_entry": [
                        "/say this is triggering a command", //
                        "@s some_mob_event:event_name"
                    ],
                    "on_exit": [
                        // Same thing as on_entry
                    ]
                }
            }
        }
    }
}
```

Animations are also very similar to their resource pack counterparts. The main difference would be the `timeline` property which allows commands and events to run at certain points in the animation.
```json
{
    "format_version": "1.8.0",
    "animations": {
        "animation.animation_name": {
            "animation_length": 3,
            "loop": false,
            "timeline": {
                "0.0": [
                    // Commands and events to trigger at 0.0s during the animation.
                ]
            }
        }
    }
}
```

We will need to modify the entity definition file in order to register the animation and animation controller files to the entity.
```json
{
    "format_version": "1.12.0",
    "minecraft:entity": {
        "description": {
            "identifier": "entity_id",
            "is_spawnable": true,
            "is_summonable": true,
            "is_experimental": true,
            "animations": {
                // These work the same as the resource pack's entity animation property.
            },
            "scripts": {
                "animate": [
                    // Any animation/controller name you specify here will be automatically run.
                ]
            }
        }
    }
}
```

### Application

Now that we have this information, we can now apply that to our original problem.

> I wanted to find a way to have an entity slowly fall when they spawn and then stay in midair after a few seconds.

I went about doing this by adding a component group and registering some events in the entity definition file.
```json
"component_groups": {
    "falling": {
        "minecraft:physics": {
            "has_gravity": true
        }
    },
    "still": {
        "minecraft:physics": {
            "has_gravity": false
        }
    }
},

"events": {
    "minecraft:entity_spawned": {
        "add": {
            "component_groups": [
                "falling"
            ]
        }
    },
    "wq:still": {
        "remove": {
            "component_groups": [
                "falling"
            ]
        },
        "add": {
            "component_groups": [
                "still"
            ]
        }
    }
}
```

I went about doing this by creating a animation lasting 3.0 seconds which...
- Applies the `slow_falling` effect to the entity at 0.0 seconds.
- Triggers the `wq:still` mob event on itself at 2.0 seconds. (Removing the `falling` component group and adding the `still` component group)
- Sets the block under itself as a barrier block at 2.0 seconds.
- Removes the block under itself at 2.5 seconds.

**Why we need to set the barrier block when we're removing it moments later?**
I found that even though we removed the `minecraft:physics` component, the entity was still falling. My belief is that there's some sort of velocity still active due to the gravitational pull `minecraft:physics` applies which exists even after we remove the component. I found that you could remove it by having the entity fall on a block.

In this case, we did not need to use any animation controller, however maybe in the future we will!
\- William Qi

[Home](../index.md)
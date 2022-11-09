---
title: Hayden Badger - Seldom
---

*The code in this page is dual-licensed under Apache 2.0 and MIT at your option.*

# Hayden Badger - Seldom

Hello! I'm Hayden, a game developer and Bevy plugin author. This portfolio showcases
my more notable projects. First, here's a brief introduction.

I've used various tools to make games since 2008, but I didn't learn Rust until 2019,
when I was in college. It quickly became my favorite language, so I started many projects
in it. I used Amethyst on and off, until I discovered Bevy 0.4 in December 2020. I used it
on and off, until September 2021, when I became less busy with school and work, and I started
working on my Bevy projects very actively. This active work continues to today.

In May 2022, I earned my Software Engineering B.S. from Arizona State University with a 4.00 GPA.
Since then, I've been living off the money I saved up and working on various projects in Bevy.
After adding Rhai scripting to [one of my games](#mystery-castle-may-2022---july-2022), I realized
some of my work could be useful to other developers. Since then, I've been writing my games
in a decoupled way, so I can pull out a piece of it and release it as a plugin. I've released
5 Bevy-related crates so far, and I currently have 3 more in the works.

I ordered these projects roughly by some combination of reverse-chronology and relevance.
This portfolio is split into four main sections. Here's some links for navigation.

- [Bevy Plugins](#bevy-plugins) (5 items)
- [Games](#games) (13 items)
- [Work](#work) (3 items)
- [Other Projects](#other-projects) (4 items)

## Bevy Plugins

[Back to top](#hayden-badger---seldom)

### `seldom_pixel` (July 2022 - Present)

[GitHub](https://github.com/Seldom-SE/seldom_pixel)

<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/pmTPdGxYVYw" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

`seldom_pixel` is my largest plugin and the plugin that I'm proudest of. It handles filters,
animations, typefaces, particle emitters,
[`bevy_ecs_tilemap`](https://github.com/StarArawn/bevy_ecs_tilemap) integration, and much more,
for limited color palette pixel art games. It leverages the color palette restriction
to make it easy to create effects, and relatively easy for me to add effect-related features.
I originally made it for [Bloodcurse Island](#bloodcurse-island), and I've been using it
for all of my projects since then.

The two main pieces of `seldom_pixel` are the asset processing and the rendering. Bevy `Image`s
are processed into my own image representation that varies based on the type of asset.
These assets each have custom rendering code, and are rendered based on the entities and components
in the world, onto an `Image` that represents the screen.

Here's a sample of code that uses `seldom_pixel`.

<!-- TODO fix this style -->

```rust
*cursor = PxCursor::Filter {
    idle: assets.invert.clone(),
    left_click: assets.invert.clone(),
    right_click: assets.invert.clone(),
};

commands
    .spawn_bundle(PxSpriteBundle {
        sprite: assets.background.clone(),
        anchor: PxAnchor::BottomLeft,
        layer: Layer::Background,
        ..default()
    })
    .insert(assets.background_filter.clone())
    .insert(Name::new("Background"));

commands
    .spawn_bundle(PxSpriteBundle::<Layer> {
        sprite: assets.player_idle.clone(),
        ..default()
    })
    .insert(PxSubPosition::default())
    .insert(Player)
    .insert(Name::new("Player"));
```

### `seldom_state` (September 2022 - Present)

[GitHub](https://github.com/Seldom-SE/seldom_state)

`seldom_state` is my most popular crate. It adds a component-based state machine that you can add
to your entities. You can define your own states and triggers and automatically add and remove
bundles based on the current state. Most of the implementation involved Rust type system magic
and Bevy reflection, which was fun to work with and resulted in a pretty clean API.
Once components-as-bundles lands in a Bevy release, I'll be able to remove all these tuples
from the API.

Here's a sample of code that uses `seldom_state`.

```rust
.insert(
    StateMachine::new((Move,))
        .trans::<(Move,)>(JustPressedTrigger(Action::Melee), (Melee,))
        .trans::<(Melee,)>(DoneTrigger::success(), (Move,))
        .remove_on_exit::<(Melee,), PxAnimationBundle>()
        .trans::<(Move,)>(JustPressedTrigger(Action::Shoot), (Shoot,))
        .trans::<(Shoot,)>(DoneTrigger::success(), (Move,))
        .remove_on_exit::<(Shoot,), PxAnimationBundle>(),
)
```

### `seldom_map_nav` (September 2022 - Present)

[GitHub](https://github.com/Seldom-SE/seldom_map_nav)

<video src="https://user-images.githubusercontent.com/38388947/200685525-9fbf9826-178e-4a71-9dd5-35431853f4ad.mp4" data-canonical-src="https://user-images.githubusercontent.com/38388947/200685525-9fbf9826-178e-4a71-9dd5-35431853f4ad.mp4" controls="controls" muted="muted" class="d-block rounded-bottom-2 border-top width-fit" style="max-height:640px;">
</video>

`seldom_map_nav` adds navmesh generation, pathfinding, and navigation for tilemaps to Bevy. It uses
the [`navmesh`](https://github.com/PsichiX/navmesh) crate to generate navmeshes from a list
of triangles, and to do pathfinding. It uses
the [`cdt`](https://github.com/Formlabs/foxtrot/tree/master/cdt) crate to generate a list
of triangles from a list of vertices and constraints. The part that `seldom_map_nav` handles
is generating a list of vertices and constraints from a tilemap, with varying levels of clearance.
It also handles navigation and provides an API for developers to interface with the crate's
features. The generated paths are occasionally poor, even on the greatest quality settings,
so I'm keeping an eye out for other navmesh crates that I can depend on.

Here's a sample of code that uses `seldom_map_nav`. It also uses `seldom_state`.

```rust
let pathfind = Pathfind::new(
    map,
    0.,
    Some(REPATH_FREQUENCY),
    PathTarget::Dynamic(player),
    NAV_QUERY,
    NAV_PATH_MODE,
);

// ...

.insert(
    StateMachine::new((Idle,))
        .trans::<(Idle,)>(
            Near {
                target: player,
                range: TRACK_PLAYER_RANGE,
            },
            NavBundle {
                nav: Nav::new(GHOST_SPEED),
                pathfind: Pathfind {
                    radius: GHOST_RADIUS,
                    ..pathfind.clone()
                },
            },
        )
        .trans::<NavBundle>(
            NotTrigger(Near {
                target: player,
                range: LOSE_PLAYER_RANGE,
            }),
            (Idle,),
        ),
)
```

### `seldom_fn_plugin` (August 2022 - Present)

[GitHub](https://github.com/Seldom-SE/seldom_fn_plugin)

`seldom_fn_plugin` is a small crate that improves the ergonomics of Bevy plugins by avoiding
them entirely. The example below shows a basic usage, but it can also help avoid using
`PhantomData` and making unnecessary clones.

```rust
// Before:

pub struct ControlsPlugin;

impl Plugin for ControlsPlugin {
    fn build(&self, app: &mut App) {
        app.init_resource::<Controls>();
    }
}

// After:

pub fn controls_plugin(app: &mut App) {
    app.init_resource::<Controls>();
}
```

### `seldom_interop` (September 2022 - Present)

[GitHub](https://github.com/Seldom-SE/seldom_interop)

`seldom_interop` is a small crate that adds traits for Bevy position components. I made it
so that I could use [`seldom_map_nav`](#seldom_map_nav-september-2022---present)
with [`seldom_pixel`](#seldom_pixel-july-2022---present)'s position components, while maintaining
support for `Transform`.

## Games

[Back to top](#hayden-badger---seldom)

### Star Machine (December 2021 - April 2022)

[Play](star_machine.html)

*I recommend playing the game before you watch the video. Use the video as a fallback in case*
*the game doesn't work or you get stuck on a level. This section's text contains instructions.*

<video src="https://user-images.githubusercontent.com/38388947/200714099-6aafe661-c5da-4ba3-80e4-375447296ae2.mp4" data-canonical-src="https://user-images.githubusercontent.com/38388947/200685525-9fbf9826-178e-4a71-9dd5-35431853f4ad.mp4" controls="controls" muted="muted" class="d-block rounded-bottom-2 border-top width-fit" style="max-height:640px;">
</video>

Click [here](#hayden-badger---seldom) to try the game! I've compiled it to WebAssembly,
so you can play in your browser. The link leads to a webpage with a dark gray background. A black
rectangle will appear when the game has downloaded, and some UI will show up once it's loaded.
The game starts in the editor, which also has menus to change the controls and start the game.
A quirk about deploying Bevy games to WebAssembly is that it uses scan codes instead of key codes.
This means that even if you use an alternative keyboard layout, you will want to set your controls
to QWERTY. The game starts in Dvorak by default, so there will be a button that says,
"Controls: Dvorak". Click that button so it changes to "Controls: QWERTY". After that, click
the "Play" button to start the game. If you need to restart a level, press Escape, and then click
"Test".

Use WASD, or your keyboard's equivalent, to move your character, and press space to place
or pick up a wire. You can hold the WASD keys, or spam them, whichever you prefer. You can also
hold space while moving to place or pick up the wire on each tile you move to. The last two levels
in the game (the second and third ones with lasers) are really hard, so you may want to peek
at the video for hints. There are supposed to be more levels before those ones,
but this is where I left off before I put the project on hold.

You can use the level editor as well, but there are some things you will need to keep in mind.
An empty level is loaded as soon as you start the game, so you may start placing tiles immediately.
The bottom-left corner of the tilemap is at the center of the screen, so try placing tiles closer
to the top-right of the screen to get your bearings. Use the WASD keys to move the camera
(remember to set your controls to QWERTY) to a better position. Experiment with the "Palette",
"Layer", "Tile Properties", and "Tools" windows to learn the editor. You must place at least one
Spawn tile, else the game will crash when you test the level. To test your level, click "Test",
and to return to the editor, either complete your level or press Escape. The game will crash if you
place a tile at an edge of the tilemap, or make it possible for a laser to emit out of the tilemap.

Star Machine was never built for WebAssembly, but I made this build specifically
for this portfolio. Unfortunately, WebAssembly requires using `bevy_ecs_tilemap`'s `atlas` feature,
which leaves visual artifacts at the edges of tiles. Also, saving and loading levels from disk
doesn't work on WebAssembly. Anyway, I hope you enjoy!

Star Machine is my proudest work, and my most complex game. I originally came up with the idea
in middle school, and a friend and I would spend class time designing puzzles on graph paper,
which we would have each other solve at lunch. It was inspired by Minecraft's redstone,
and we built a couple prototypes within Minecraft, until a teacher told me I should just make
a standalone game. In senior year of high school, two friends and I made a prototype in JavaScript,
but it lacked most of the features. I worked on an earlier version in Bevy for my honors thesis
from February 2021 to June 2021, but I dropped out of my school's honors program and stopped work
on the game. Then, in December 2021, I restarted and made this version of the game
and it's much more robust than all previous versions. I have many more ideas for this game,
and I want to extend it with hundreds of levels in the future.

The most complex part of the game, and the most difficult piece of software I've ever written,
is the solver. It's the part of the game that analyzes how the components are connected via
wires and lasers, and determines what should be on and what should be off. What makes the solver
so complex is that it is possible to create cycles that either enforce that the cycle should
stay in the same power state, or contradict itself, making it impossible for it to be in either
state. I would have made a much simpler algorithm that allows the game to make inconsistencies
and just design around it, but I wanted to incorporate these contradictions into a game mechanic,
flavored as alternating current. Having to update the solver every time I wanted
to add a new mechanic is what drove me away from this project, but I plan to return
to it eventually, with a more robust and abstract solver.

### Mystery Castle (May 2022 - July 2022)

<video src="https://user-images.githubusercontent.com/38388947/200753144-97823c41-cd99-4ba9-99a6-4d9d1a929ca2.mp4" data-canonical-src="https://user-images.githubusercontent.com/38388947/200685525-9fbf9826-178e-4a71-9dd5-35431853f4ad.mp4" controls="controls" muted="muted" class="d-block rounded-bottom-2 border-top width-fit" style="max-height:640px;">
</video>

After I graduated college, I started working on a game with some friends. I pitched this idea
for a mystery game to them, and they liked it, so we got started. I did the programming
and scripting, and we also had an artist, writer, and another scripter. I got the game
to a state where the team could start rapidly adding content, while I work on making the game
look and feel better, but the artist dropped from the team due to being busy, and the rest
of the team didn't have as much time as I did, so we put the project on indefinite hiatus.

The game was inspired by Outer Wilds and Return of the Obra Dinn. It's a mystery game
with time mechanics. This portfolio is public, though, so I don't want to spoil the central
mechanic. I'm willing to spoil this temporary dialogue though.

I focused on making the game very quickly extendable though LDtk, Rhai scripting, and .ron config
files, so none of the game's content is hard-coded. The LDtk levels could specify script names
to run on collision or interact, and the scripts could pull information from the levels,
game state, data exposed from other scripts, and config. I think the scripting turned out well.

```rhai
switch time() {
    "3:00" => {
        dialog("The door is locked").wait();
        add_journal("ohio_bedroom_door_locked");
    }
    "3:10" => {
        dialog("Jarvis hears the sound of glass shattering").wait();
        set_time("3:20");
    }
    "3:20" => dialog("The door is locked").wait(),
    "3:30" => {
        if flag("hall_door_unlocked") {
            teleport("jarvis", [0, 4]);
        } else {
            investigate_start("jarvis", #{
                background: "hall_door_background",
            });
            show_inventory();

            loop {
                switch prompt().wait() {
                    "hall" => {
                        dialog("This room is connected to the hall...", "jarvis").wait();
                        dialog("I'm not sure where I was going with that", "jarvis").wait();
                    }
                    "ohio_bedroom_door_locked" => dialog("Yep, this thing sure is locked!", "jarvis").wait(),
                    "gabriel_key" => dialog("Gabriel gave me the key for this door", "jarvis").wait(),
                    "ohio_key" => {
                        dialog("Jarvis unlocks the door").wait();
                        set("hall_door_unlocked");
                        break;
                    }
                    _ => dialog("I'm not sure what to do with this", "jarvis").wait(),
                }
            }

            investigate_end();
        }
    }
    _ => {
        teleport("jarvis", [0, 4]);
    }
}
```

### Voxmod (April 2022 - May 2022)

*The following repository is not currently licensed, in case you're someone who is concerned*
*about liability.*

[GitHub](https://github.com/Seldom-SE/voxmod)

<video src="https://user-images.githubusercontent.com/38388947/200760158-d14d366e-717e-40b7-a3c7-926f1d78b515.mp4" data-canonical-src="https://user-images.githubusercontent.com/38388947/200685525-9fbf9826-178e-4a71-9dd5-35431853f4ad.mp4" controls="controls" muted="muted" class="d-block rounded-bottom-2 border-top width-fit" style="max-height:640px;">
</video>

I started this game because I thought Minecraft wasn't doing enough to foster their mapping
and modding communities, so only the more technically-skilled players could play and make maps
and mods. So, the idea was to make an easily extensible voxel game framework, like Garry's Mod
but for voxel games. Using Rob Swain's
[`bevy-vertex-pulling`](https://github.com/superdump/bevy-vertex-pulling) as a guide,
I got chunk-based voxel rendering working, and I made a very reusable menu UI. I stopped working
on this shortly after graduating college, so I could work
on [Mystery Castle](#mystery-castle-may-2022---july-2022).

### Dark Realms (August 2022 - November 2022)

### Spindarella's Monsters (August 2022)

### Bloodcurse Island (July 2022 - August 2022)

### Cotton (July 2022)

### Super Dodge Mania! (November 2018 - June 2020)

### CPI 311 Game Engine (August 2020 - December 2020)

### Robbin' and Rollin' (October 2019 - December 2019)

### Bevy Cursed Tomb (February 2022 - March 2022)

### Perlin Island Generator (July 2020 - August 2020)

### Cthulhu (October 2019 - December 2019)

## Work

[Back to top](#hayden-badger---seldom)

### Sparklight (June 2021 - August 2021)

### StreamWork (September 2018 - January 2021)

### True Fans (September 2021 - April 2022)

## Other Projects

[Back to top](#hayden-badger---seldom)

### Tower (February 2022)

### Lighthouse (October 2018 - April 2019)

### ChromAR (September 2019)

### GitHub Grader (January 2021 - May 2021)

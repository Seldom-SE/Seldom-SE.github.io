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
After adding Rhai scripting to one of my games, I realized some of my work could be useful
to other developers. Since then, I've been writing my games in a decoupled way,
so I can pull out a piece of it and release it as a plugin. I've released 5 Bevy-related
crates so far, and I currently have 3 more in the works.

This portfolio is split into four main sections. Here's some links for navigation.

- [Bevy Plugins](#bevy-plugins) (5 items)
- [Games](#games) (13 items)
- [Work](#work) (3 items)
- [Other Projects](#other-projects) (4 items)

## Bevy Plugins

[Back to top](#hayden-badger---seldom)

### `seldom_pixel`

[GitHub](https://github.com/Seldom-SE/seldom_pixel)

<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/pmTPdGxYVYw" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

`seldom_pixel` is my largest plugin and the plugin that I'm proudest of. It handles filters,
animations, typefaces, particle emitters, `bevy_ecs_tilemap` integration, and much more,
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

### `seldom_state`

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

### `seldom_map_nav`

[GitHub](https://github.com/Seldom-SE/seldom_map_nav)

<video src="https://user-images.githubusercontent.com/38388947/200685525-9fbf9826-178e-4a71-9dd5-35431853f4ad.mp4" data-canonical-src="https://user-images.githubusercontent.com/38388947/200685525-9fbf9826-178e-4a71-9dd5-35431853f4ad.mp4" controls="controls" muted="muted" class="d-block rounded-bottom-2 border-top width-fit" style="max-height:640px;">
</video>

`seldom_map_nav` adds navmesh generation, pathfinding, and navigation for tilemaps to Bevy. It uses
the `navmesh` crate to generate navmeshes from a list of triangles, and to do pathfinding. It uses
the `cdt` crate to generate a list of triangles from a list of vertices and constraints. The part
that `seldom_map_nav` handles is generating a list of vertices and constraints from a tilemap,
with varying levels of clearance. It also handles navigation and provides an API for developers
to interface with the crate's features. The generated paths are occasionally poor,
even on the greatest quality settings, so I'm keeping an eye out for other navmesh crates
that I can depend on.

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

### `seldom_fn_plugin`

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

### `seldom_interop`

[GitHub](https://github.com/Seldom-SE/seldom_interop)

`seldom_interop` is a small crate that adds traits for Bevy position components. I made it
so that I could use `seldom_map_nav` with `seldom_pixel`'s position components, while maintaining
support for `Transform`.

## Games

[Back to top](#hayden-badger---seldom)

### Star Machine

[Play](star_machine.html)

*I recommend playing the game before you watch the video. Use the video as a fallback in case*
*the game doesn't work or you get stuck on a level. This section's text contains instructions.*

<video src="https://user-images.githubusercontent.com/38388947/200714099-6aafe661-c5da-4ba3-80e4-375447296ae2.mp4" data-canonical-src="https://user-images.githubusercontent.com/38388947/200685525-9fbf9826-178e-4a71-9dd5-35431853f4ad.mp4" controls="controls" muted="muted" class="d-block rounded-bottom-2 border-top width-fit" style="max-height:640px;">
</video>

Click the play button at the top of this section to try the game! I've compiled it to WebAssembly,
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
and we built a couple prototypes within Minecraft, before a teacher told me I should just make
a standalone game. In senior year of high school, two friends and I made a prototype in JavaScript,
but it lacked most of the features. I made this version of the game on my own in Bevy,
and it's much more robust. I have many more ideas for this game, and I want to extend it
with hundreds of levels in the future.

The most complex part of the game, and the most difficult piece of software I've ever written,
is the solver. It's the part of the game that analyzes how the components are connected via
wires and lasers, and determines what should be on and what should be off. What makes the solver
so complex is that it is possible to create cycles that either enforce that the cycle should
stay in the same power state, or contradict itself, making it impossible for it to be in either
state. I would have made a much simpler algorithm that allows the game to make contradictions,
but I wanted to use these contradictions as a game mechanic, flavored as alternating current.
Having to update the solver every time I wanted to add a new mechanic is what drove me away
from this project, but I plan to return to it eventually, with a more robust and abstract solver.

### Mystery Castle

### Voxmod

### Dark Realms

### Spindarella's Monsters

### Bloodcurse Island

### Cotton

### Super Dodge Mania!

### CPI 311 Game Engine

### Robbin' and Rollin'

### Bevy Cursed Tomb

### Perlin Island Generator

### Cthulhu

## Work

[Back to top](#hayden-badger---seldom)

### Sparklight

### StreamWork

### True Fans

## Other Projects

[Back to top](#hayden-badger---seldom)

### Tower

### Lighthouse

### ChromAR

### GitHub Grader

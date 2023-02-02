---
title: Hayden Badger - Seldom
---

*The code in this page is dual-licensed under Apache 2.0 and MIT at your option.*

# Hayden Badger - Seldom

Hello! I'm Hayden, a game developer and Bevy plugin author. This portfolio showcases
my more notable projects. First, here's a brief introduction.

I've used various tools to make games since 2008, but I didn't learn Rust until 2019,
when I was in college. It quickly became my favorite language, so I started many projects
in it. I used Amethyst on and off, until I got into Bevy 0.4 in December 2020. I used it
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
- [Games](#games) (15 items)
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
I originally made it for [Bloodcurse Island](#bloodcurse-island-july-2022---august-2022),
and I've been using it for all of my Bevy projects since then.

The two main pieces of `seldom_pixel` are the asset processing and the rendering. Bevy `Image`s
are processed into my own image representation that varies based on the type of asset.
These assets each have custom rendering code, and are rendered based on the entities and components
in the world, onto an `Image` that represents the screen.

Here's a sample of code that uses `seldom_pixel`.

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
to your entities. You can define your own states and triggers, and you can automatically add
and remove bundles based on the current state. Most of the implementation involved Rust type system
magic and Bevy reflection, which was fun to work with and resulted in a pretty clean API.

Here's a sample of code that uses `seldom_state`.

```rust
.insert(
    StateMachine::new(Move)
        .trans::<Move>(JustPressedTrigger(Action::Melee), Melee)
        .trans::<Melee>(DoneTrigger::success(), Move)
        .remove_on_exit::<Melee, PxAnimationBundle>()
        .trans::<Move>(JustPressedTrigger(Action::Shoot), Shoot)
        .trans::<Shoot>(DoneTrigger::success(), Move)
        .remove_on_exit::<Shoot, PxAnimationBundle>(),
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
    StateMachine::new(Idle)
        .trans::<Idle>(
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
            Idle,
        ),
)
```

### `seldom_fn_plugin` (August 2022 - Present)

[GitHub](https://github.com/Seldom-SE/seldom_fn_plugin)

`seldom_fn_plugin` is a small crate that improves the ergonomics of Bevy plugins by avoiding
them entirely. The example below shows a basic usage, but it can also help to avoid using
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

<video src="https://user-images.githubusercontent.com/38388947/200714099-6aafe661-c5da-4ba3-80e4-375447296ae2.mp4" data-canonical-src="https://user-images.githubusercontent.com/38388947/200714099-6aafe661-c5da-4ba3-80e4-375447296ae2.mp4" controls="controls" muted="muted" class="d-block rounded-bottom-2 border-top width-fit" style="max-height:640px;">
</video>

#### Instructions

Click [here](star_machine.html) to try the game! I've compiled it to WebAssembly,
so you can play in your browser. The link leads to a webpage with a dark gray background. A black
rectangle will appear when the game has downloaded and some UI will show up once it's loaded.
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

#### Description

Star Machine is my proudest work, and my most complex game. I have many more ideas for this game,
and I want to extend it with hundreds of levels in the future.

The most complex part of the game, and the most difficult piece of software I've ever written,
is the solver. It's the part of the game that analyzes how the components are connected via
wires and lasers, and determines what should be on and what should be off. What makes the solver
so complex is that it is possible to create cycles that either enforce that the cycle should
stay in the same power state, or contradict itself, making it impossible for it to be in either
state. I would have made a much simpler algorithm that allows the game to make inconsistencies
and just design around it, but I wanted to incorporate these contradictions into a game mechanic,
flavored as alternating current. Having to update the solver every time I wanted to add a new
mechanic is what drove me away from this project, but I plan to return to it eventually,
with a more robust and abstract solver.

### Heist Havoc (December 2022 - Present)

<video src="https://user-images.githubusercontent.com/38388947/216214284-c3e629de-c812-4d0b-8afe-3b4844f96b6c.mp4" data-canonical-src="https://user-images.githubusercontent.com/38388947/200797425-0fb3d389-28c3-46f6-bdb7-cc424ec9edd0.mp4" controls="controls" muted="muted" class="d-block rounded-bottom-2 border-top width-fit" style="max-height:640px;">
</video>

This is a networked game based on [Robbin' and Rollin'](#robbin-and-rollin-october-2019---december-2019). I used `naia`
for the networking. It handles syncing the ECS between client and server, but it doesn't handle
rollback, so I implemented that myself. My rollback code doesn't work well on `naia`'s layers
of abstraction, so I'm looking into using `naia_socket` directly instead.

### Mystery Castle (May 2022 - July 2022)

<video src="https://user-images.githubusercontent.com/38388947/200753144-97823c41-cd99-4ba9-99a6-4d9d1a929ca2.mp4" data-canonical-src="https://user-images.githubusercontent.com/38388947/200753144-97823c41-cd99-4ba9-99a6-4d9d1a929ca2.mp4" controls="controls" muted="muted" class="d-block rounded-bottom-2 border-top width-fit" style="max-height:640px;">
</video>

After I graduated from college, I started working on a game with some friends. I did the programming
and scripting, and we also had an artist, writer, and another scripter. I got the game
to a state where the team could start rapidly adding content, while I work on making the game
look and feel better, but we had to put the project on indefinite hiatus after multiple team members
became busy.

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

### Terraria Archipelago (December 2022 - Present)

[GitHub](https://github.com/Seldom-SE/archipelago_terraria_client) - [Steam](https://steamcommunity.com/sharedfiles/filedetails/?id=2922217554)

A randomizer is a mod for a game that shuffles around items in the game without making it
unsolvable. So, in Hollow Knight, for example, when you pick up the item that lets you dash,
it might give you the item that lets you double jump instead. [Archipelago](https://archipelago.gg/)
is a multiworld randomizer, which means that it shuffles items *between* games. So,
when you defeat Skeletron in your Terraria game, your friend might get the dash in Hollow Knight,
but your game isn't in a Post-Skeletron state (so you can't enter the Dungeon) until your friend
picks up some other item in their game.

I made a Terraria integration for Archipelago. The server-side integration is in Python. It handles
the complex game logic and interfaces with Archipelago's core. The mod is written in C# and handles
communicating with Archipelago's server, processing player commands, and directly editing Terraria's
bytecode (CIL) to modify gameplay features. I am currently working on tweaking Terraria's
progression to be a better fit for randomization and adding support for Calamity, a popular mod
for Terraria.

### Voxmod (April 2022 - May 2022)

*The following repository is not currently licensed, in case you're someone who is concerned*
*about liability.*

[GitHub](https://github.com/Seldom-SE/voxmod)

<video src="https://user-images.githubusercontent.com/38388947/200760158-d14d366e-717e-40b7-a3c7-926f1d78b515.mp4" data-canonical-src="https://user-images.githubusercontent.com/38388947/200760158-d14d366e-717e-40b7-a3c7-926f1d78b515.mp4" controls="controls" muted="muted" class="d-block rounded-bottom-2 border-top width-fit" style="max-height:640px;">
</video>

I started this game because I thought Minecraft wasn't doing enough to foster its mapping
and modding communities. So only the more technically-skilled players could play and make maps
and mods. The idea was to make an easily extensible voxel game framework, like Garry's Mod
but for voxel games. Using Rob Swain's
[`bevy-vertex-pulling`](https://github.com/superdump/bevy-vertex-pulling) as a guide,
I got chunk-based voxel rendering working, and I made a very reusable menu UI. I stopped working
on this shortly after graduating college, so I could work
on [Mystery Castle](#mystery-castle-may-2022---july-2022).

### Dark Realms (August 2022 - November 2022)

Dark Realms is a roguelike that's like a zoomed-in Pacman. You collect loot from chests
while avoiding ghosts. It's the black-and-white game featured at the end of `seldom_pixel`'s
[demo video](https://youtu.be/pmTPdGxYVYw?t=90). I spent most of the development time getting
my crates to a releasable state and releasing them.

### Spindarella's Monsters (August 2022)

[Play](https://iyes.itch.io/spindarellas-monsters) - [GitHub](https://github.com/BroovyJammy/gaem)

This game was a blast to create. It was for Bevy's second game jam, whose theme was "COMBINE".
I worked with some very talented gamedevs in the Bevy community, so we got a lot done.
You can see the credits on [the Itch page](https://iyes.itch.io/spindarellas-monsters). We made a
turn-based tactics game where you build and send monstrosities into battle to impress a spider
lady. I focused more on the in-game interface and effects, unit movement and attacking, AI,
and design.

### Bloodcurse Island (July 2022 - August 2022)

Bloodcurse Island was a game that I was inspired to start, influenced by my work
on [Cotton](#cotton-july-2022). It's actually only a menu, and I spent the entire time working
on [`seldom_pixel`](#seldom_pixel-july-2022---present). It's a pretty menu, though,
and it's a great showcase for `seldom_pixel`. It's the second to last scene in `seldom_pixel`'s
[demo video](https://youtu.be/pmTPdGxYVYw?t=76).

### Cotton (July 2022)

[GitHub](https://github.com/Seldom-SE/cotton)

![A game like Catan, with hexagonal tiles of terrain surrounded by water and game pieces](https://raw.githubusercontent.com/Seldom-SE/cotton/main/screenshot.png)

Cotton, inspired by Catan, was a game that I made because I wanted to add more variation to Catan.
I enjoy how the variation in the board setup can make for very different games. If all of the
brick tiles are assigned unfavorable chits, for example, the gameplay will be more focused
on development cards, which don't need bricks. I expanded this variation by randomly assigning
each tile, port, and chit independently of each other, instead of shuffling and dealing. Now,
you might end up with a game with no mountains or three deserts. This one was more of a side project
after [Mystery Castle](#mystery-castle-may-2022---july-2022) was put on hiatus, but I finished
the core of the game, except you can't trade or win. But then I came up with new ideas for it,
which became [Bloodcurse Island](#bloodcurse-island-july-2022---august-2022).

### Super Dodge Mania! (November 2018 - June 2020)

*CW: flashing lights*

<video src="https://user-images.githubusercontent.com/38388947/200797425-0fb3d389-28c3-46f6-bdb7-cc424ec9edd0.mp4" data-canonical-src="https://user-images.githubusercontent.com/38388947/200797425-0fb3d389-28c3-46f6-bdb7-cc424ec9edd0.mp4" controls="controls" muted="muted" class="d-block rounded-bottom-2 border-top width-fit" style="max-height:640px;">
</video>

In high school, I made a game called `dodger.py` in Python and pygame. Later, to learn Unity,
I decided to reimplement `dodger.py`, which became Super Dodge Mania!. I kept working on it
on and off, keeping the simple gameplay, but adding more and more polish. I taught myself CG
to write the shaders for this game. I also made the background music (unmute the video player to
hear it), which came out well for a first song. The song gets gradually more bitcrushed
as the player loses health. I only tested it on my old MacBook, which can't record video
while playing the game, so I recorded this video on my main machine, which is much faster. So,
I just discovered that some of the effects, such as the hitstun and screenshake, don't work
as well on faster machines. I never ended up releasing Super Dodge Mania! because the codebase
is a disaster, and it would be easier to start over than to fix all the bugs. After all,
it's my first Unity project.

### CPI 311 Game Engine (August 2020 - December 2020)

In college, I took a Game Engine Development class, which had one project over the entire semester:
build a game engine on top of MonoGame. We had to implement rendering, shaders, a Unity-like
GameObject, physics, a hierarchy system, and a simple game that proves the functionality
of the engine. I went above and beyond in this class, implementing an ECS (Just the front-end
for one. I didn't know about the storage pattern at the time) and a scheduler, inspired by Bevy
(which I discovered at some point during this class) and Amethyst.

### Robbin' and Rollin' (October 2019 - December 2019)

<video src="https://user-images.githubusercontent.com/38388947/200965224-82621938-ddc3-49aa-8022-7f938639494f.mp4" data-canonical-src="https://user-images.githubusercontent.com/38388947/200965224-82621938-ddc3-49aa-8022-7f938639494f.mp4" controls="controls" muted="muted" class="d-block rounded-bottom-2 border-top width-fit" style="max-height:640px;">
</video>

This is the final project for my Game Development I class. It was in a team of 6, but I was
the only experienced game developer, so I ended up doing all of the sounds, almost all
of the programming, most of the design, and some of the art. It's very fun! The goal is to gather
the most money before the time runs out. There's a constant sense of risk and reward,
because the player has to decide which items to go for based on the type, distance, proximity
to the opponent, items they and the opponent have, etc. Picking up money bags leaves less space
in your inventory and decreases your walking speed, but bringing them back to spawn
is the only way to get money. It would be a great multiplayer game, which I implemented
in [Heist Havoc](#heist-havoc-december-2022---present).

### Bevy Cursed Tomb (February 2022 - March 2022)

[Play](https://mdenchev.itch.io/bevy-cursed-tomb) - [GitHub](https://github.com/mdenchev/bevy_jam_1)

I teamed up with 4 other people for the first Bevy Game Jam. We made a game where you
have to kill 50 enemies as quickly as possible, but you can go back in time to the beginning
of the game as many times as you want to improve your time. I made the art, and the item
and inventory system.

### Perlin Island Generator (July 2020 - August 2020)

![An abstract, pixelated, top-down view of an island](https://user-images.githubusercontent.com/38388947/200968846-00ff741b-9e31-4e4c-af29-0a7bd92192f5.png)

I made this island generator in Amethyst.

### Cthulhu (October 2019 - December 2019)

<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/tHMylnw0zy8" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

I made this game as a project for my Game Development I class, under my school's honors college.
It was inspired by Factorio, but the goal is to survive as long as possible. The normal enemies
can't destroy your buildings, but they do distract units, making it difficult to build if you
don't get rid of them. The final boss cannot be defeated and leaves a trail of tiles
that cannot be built on, so the endgame involves building what you need in confined spaces.
Unfortunately, the endgame drags on, because the enemies eventually create lag, slowing down
the game.

## Work

[Back to top](#hayden-badger---seldom)

### Sparklight (June 2021 - August 2021)

I was brought on as an intern to a team that creates user-facing code. In my time here, I developed
APIs in ASP.NET Core to replace some legacy code and expand a product's capabilities.
I was involved in the process of Scrum and participated in requirements-gathering meetings.
I learned a lot about software development in larger companies and workplace communication.

### StreamWork (September 2018 - January 2021)

[StreamWork](https://www.streamwork.live/)

StreamWork was a startup that I co-founded! We, a small team with only two developers, created
an educational live-streaming website that raised $27,000 in grants. It was built in ASP.NET Core
on Microsoft ASURE, using C#, HTML/CSS, and JavaScript, with a SQL database. The website
was fully-functioning, with OBS-compatible live-streaming, video archiving, live chat,
a comments section, view counting, video search, customizable profiles and profile pictures,
an email notification system, and a payment system. I learned how to work on a team under time
and budget restraints. I'm proud of what my team and I built, and it's satisfying to see what we
can build over a long period of time.

### True Fans (September 2021 - April 2022)

[Client](https://roguemedialive.com/)

Four peers and I developed an iOS/Android social medium
targeting sports fans for an external client as our capstone project. It was written
in React Native with Typescript, using Firebase/Firestore for the database. I wrote
most of the backend and database code. We used Agile Scrum to organize, and personally met
with the client to gather requirements. The app has not been released yet, but we finished
the work that we were responsible for, and the client was very pleased with the end result.

## Other Projects

[Back to top](#hayden-badger---seldom)

### Tower (February 2022)

[GitHub](https://github.com/Seldom-SE/tower)

I wrote an esoteric programming language with painfully vertical data storage. You have to store
all of your data in 3 registers, but you can compress the data as much as you want. The challenge
is avoiding overwriting data that you still need. Here's a program that makes sure your brackets
are balanced:

```
a:-1
b:-1
b[ab]
?c[
    a,
    ?!||=a;)=a;]=a;}[
        ca
        #b
        ?!=acb:-1
        c:0
    ?c]
    ?!=a;([
        a;)
        b[ab]
    ?c]
    ?!=a;[[
        a;]
        b[ab]
    ?c]
    ?!=a;{[
        a;}
        b[ab]
    ?c]
?!|=a;\n=b:-1]
a:0
#b
.=a:-1
```

### Lighthouse (October 2018 - April 2019)

In my freshman year of college, three of my peers and I got a research grant for this entry
to the ASURE VR Innovation Challenge. We made an app that creates safe walking routes between
locations based on crime data that we got access to from police departments. We didn't win
the competition, but we were recognized for being a team of freshmen amidst teams of seniors
and graduate students. It was written in Unity, and I did mostly frontend work, including UI,
displaying maps, and rendering the AR camera.

### ChromAR (September 2019)

This is a hackathon project that I worked on with one other. It's an iOS app for the colorblind
that identifies colors, using AR. It was written in Swift. I worked on the backend,
algorithms, and image processing.

### GitHub Grader (January 2021 - May 2021)

This is a Node.js/JavaScript project that a peer and I worked on for an instructor,
under our school's honors program, to help her with grading students' projects. It pulls students'
commit information from GitHub and analyzes it to determine their frequency of work
and other information, using lines of code count and user story points. It uses the GitHub
REST API.

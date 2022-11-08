---
title: Hayden Badger - Seldom
---

*The code featured on this page is dual-licensed under Apache 2.0 and MIT at your option.*

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

[Github](https://github.com/Seldom-SE/seldom_pixel)

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

Here's a bit of code that uses `seldom_pixel`.

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

### `seldom_map_nav`

### `seldom_fn_plugin`

### `seldom_interop`

## Games

[Back to top](#hayden-badger---seldom)

### Star Machine

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

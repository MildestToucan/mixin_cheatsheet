# GauntRecluse's Mixin Cheatsheet (HEAVILY WIP)

## Table of Contents

The content is greatly incomplete. Please take anything you see in this repo with a grain of salt.

### [Injectors](injectors/README.md)

- [`@At`](injectors/At.md)
- [Injection Points](injectors/InjectionPoints.md)
- [`@Inject`](injectors/Inject.md)

### [Sugar](sugar/README.md)

- [`@Local`](sugar/Local.md)
- [`@Share`](sugar/Share.md)
- [`@Cancellable`](sugar/Cancellable.md)

## Target Audience

***THIS IS NOT FOR BEGINNERS WHO ARE STILL LEARNING HOW TO USE MIXINS AND WHAT THEY ARE.***

***DO NOT USE THIS REPOSITORY TO LEARN ENTIRELY NEW CONCEPTS. IT IS NOT INTENDED FOR THAT PURPOSE.***

This is for people who already know how to use mixins but need quick refreshers on syntax or some other aspect.

## Goals

Provide generally reliable examples of injectors and other things people may need a quick refresher on based on best-practices,
experience, and some amount of opinion.

This also will serve as my personal place to infodump niche info I find and things I want to keep a note of somewhere that wouldn't fit in proper documentation or tutorials.

This repo will not hold the user's hand through the basic syntax and functionality of Mixin as a whole. It assumes the user is already able to read and write mixins for the most part, but was interested in double-checking something.

## Other Resources

### Official Documentation

For MixinExtras features, always prioritize reading the [official MixinExtras wiki](github.com/LlamaLad7/MixinExtras/wiki/).

For base Mixin features, read the [javadocs](https://jenkins.liteloader.com/job/Mixin/javadoc/) (does not include Fabric Mixin features!)

### Fabric Wiki

The Fabric Wiki is generally an inconsistent and incomplete resource for Mixin, but here are some pages I have contributed to myself that I would generally trust the information of:

- [Glossary](https://wiki.fabricmc.net/tutorial:mixin_glossary)
- [Accessors](https://wiki.fabricmc.net/tutorial:mixin_accessors)
- [Exporting Targets](https://wiki.fabricmc.net/tutorial:mixin_export)
- [Overriding a Mixin Target's Parent](https://wiki.fabricmc.net/tutorial:mixin_override)

### Fabric Docs

The Fabric Docs will over time hopefully contain solid resources on mixins, but it's currently very limited:

- [JVM Bytecode](https://docs.fabricmc.net/develop/mixins/bytecode)

### Help Channels

To find help channels the following pages link to the official Discord servers of modding platforms which provide Mixin support:

- [Fabric](https://fabricmc.net/discuss/)
- [NeoForge](https://neoforged.net)
- [SpongePowered](https://spongepowered.org/%5DSponge)

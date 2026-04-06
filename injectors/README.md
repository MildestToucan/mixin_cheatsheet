# Injectors

Injectors are the primary tool used in Mixin and MixinExtras to make changes to target methods. They add new information
to the target method, or modify existing instructions in ways that conserve the original, such that they can chain on one another.

This means that to ensure the best functionality with injectors, you should try to avoid operations that can cancel the target method, or ignore
the original instructions entirely by replacement or otherwise, as much as possible.

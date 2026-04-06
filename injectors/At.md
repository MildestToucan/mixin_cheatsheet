# @At

<- [Back to Main](/README.md)

<- [Back to Injectors](/injectors/README.md)

`@At` is not an injector, but it is used by most injectors to specify what instruction(s) to target in the target method.

## External Resources

[JavaDocs](https://jenkins.liteloader.com/job/Mixin/javadoc/org/spongepowered/asm/mixin/injection/At.html)

## Syntax

More-or-less irrelevant attributes of the annotation will not be documented here.

```java
@At(
    // An injection point is essentially a search algorithm to use as the base for,
    // finding targets in the targeted method. One must always be specified.
    value = "INJECTION_POINT",
    /*
    A "specifier" can be added to an injection point to tweak its targeting:
    I_POINT:FIRST (default when slicing) only targets the first match. Equivalent to ordinal = 0
    I_POINT:LAST targets the last match
    I_POINT:ALL (default when not slicing) targets all matches
    I_POINT:ONE throws an error if more than one match is found
    */


    // For FIELD, the path of the field being accessed. For INVOKE, the path method being invoked.
    // The path includes the path name and the member's signature.
    // For NEW, the class name and optionally the constructor's signature.
    target = "targetPath",

    // Used by FIELD to know whether targeting a field PUT or GET
    // Use the ASM Opcodes interface to get the int constants by mnemonic
    opcode = Opcodes./* PUTFIELD/GETFIELD or PUTSTATIC/GETSTATIC for static fields*/,

    // Used by CONSTANT to specify the constant values to look for.
    // Takes Strings formatted as "key=value". Could also be used by a custom point for custom args.
    args = /* String or array of strings, unused by default */

    // Used to "shift" the targeted instruction.
    // Only really useful to target the instruction after the returned target, as
    // most injection points already insert injects before the matching instruction.
    shift = At.Shift.AFTER,

    // Used specify the ordinal of the instruction to target amidst the matches.
    // 0 would be the first match, 5 the sixth match, etc.
    // Ordinals above 0 are brittle. Use slices and expressions instead if possible.
    ordinal = /* Some int value, unused by default */,

)
```

## Usage

The broad strokes is that the `@At`'s `value`, the injection point, is the search algorithm used
on the target method's instructions. Some injection points use `target` or `opcode` to add additional
context, and others may use an extra annotation on the injector's handler method for such context.

The algorithm is applied in the target method's instructions, which returns a list of matching instructions.
Using a `@Slice`, an injector can make the search only apply to a subsection of the target method.

Using `shift` with `AFTER` will return the instructions immediately *after* the otherwise matching instructions.

An `ordinal` can be used to pick a specific match, and the injection point's specifier can further
tweak the returned list.

## Effect

The injector will apply its modifications based on the instructions returned by the `@At`'s search.
Specific effects depend entirely on a given injector.

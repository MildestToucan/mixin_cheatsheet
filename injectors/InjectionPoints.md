# Injection Points

<- [Back to Main](/README.md)

<- [Back to Injectors](/injectors/README.md)

As noted in the [@At](At.md) page, an injection point is used as the search algorithm to find targets
to modify using an injector. This page will omit injection points that aren't relevant to writing modern mixins.

Should be noted that injection points when used by injectors will either be used to *modify* the returned instructions,
such as with `@WrapOperation`, or inject *before* them, such as with `@Inject` or `@ModifyVariable`, with some exceptions like `STORE`.

To clarify, the "candidate list" in this page refers to the list of instructions to search in. Typically, this means
the entire method or the instructions contained in the relevant slices.

When an injection point returns all matching a criteria, it is implicit that other tools like ordinals and specifiers
can be used to pick a specific instruction. Ordinals and specifiers are always applied *after* the point-specific filters like
`INVOKE`, `NEW` or `FIELD`'s `target`.

## External Resources

[Injection point reference](https://github.com/SpongePowered/Mixin/wiki/Injection-Point-Reference)

[MixinExtras Expression wiki page](https://github.com/LlamaLad7/MixinExtras/wiki/Expressions)

## HEAD

[Javadocs](https://jenkins.liteloader.com/job/Mixin/javadoc/org/spongepowered/asm/mixin/injection/points/MethodHead.html)

Used to inject as early as possible in the method body, typically.

Returns the first instruction in the candidate list. If using a slice this means the first instruction in the slice.
Although if you're injecting at the `HEAD` of a slice, you should likely just directly target the slice's `from` instead of `HEAD`.

## CTOR_HEAD

[Javadocs](https://jenkins.liteloader.com/job/Mixin/javadoc/org/spongepowered/asm/mixin/injection/points/ConstructorHead.html)

Used to inject as early as possible in a constructor's body non-statically.

Used on constructors, this returns the first instruction in the candidate list *after* the `super` call. When performing
non-static injects, this should be used rather than `HEAD`, as injecting at any point prior to `CTOR_HEAD`'s point will
be disallowed on certain Mixin versions, and otherwise will be allowed if the handler is `static`.

## RETURN

[Javadocs](https://jenkins.liteloader.com/job/Mixin/javadoc/org/spongepowered/asm/mixin/injection/points/BeforeReturn.html)

Used to inject just before a method's return.

Returns all `RETURN` opcode instructions in the candidate list.

## TAIL

[Javadocs](https://jenkins.liteloader.com/job/Mixin/javadoc/org/spongepowered/asm/mixin/injection/points/BeforeFinalReturn.html)

Used to inject at the latest possible point in the method.

Returns the final `RETURN` opcode instruction in the target *method*. Slices do not apply.
The final return is not always what appears as the latest return in the source code, due to jumps.

## INVOKE

[Javadocs](https://jenkins.liteloader.com/job/Mixin/javadoc/org/spongepowered/asm/mixin/injection/points/BeforeInvoke.html)

Used to modify a method call or inject relative to one.

Returns all instructions in the candidate list invoking a method matching the `target` attribute's member path.

## FIELD

[Javadocs](https://jenkins.liteloader.com/job/Mixin/javadoc/org/spongepowered/asm/mixin/injection/points/BeforeFieldAccess.html)

Used to modify a field access or inject relative to one.

Returns all instructions in the candidate list accessing a field matching the `target` attribute's member path
and the `opcode` attribute.

## NEW

[Javadocs](https://jenkins.liteloader.com/job/Mixin/javadoc/org/spongepowered/asm/mixin/injection/points/BeforeNew.html)

Used to modify an object instantiation or inject relative to one.

Returns all `NEW` instructions in the candidate list matching the `target` attribute's signature.

## CONSTANT

[Javadocs](https://jenkins.liteloader.com/job/Mixin/javadoc/org/spongepowered/asm/mixin/injection/points/BeforeConstant.html)

Used to modify a constant value.

Returns all instructions matching the specified constant value in `args`. Values are specified in strings following a `"key=value"` format.
The args for `CONSTANT` are:

- `nullValue`: takes a boolean, `true` to look for `null` constants.
- `intValue`, `longValue`, `doubleValue`, `stringValue`: take their respective types, set to the value to look for.
- `class`: takes a string containing the fully qualified class name to match `Class` literals in the instruction list.

## STORE & LOAD

[STORE Javadocs](https://jenkins.liteloader.com/job/Mixin/javadoc/org/spongepowered/asm/mixin/injection/modify/AfterStoreLocal.html)
[LOAD Javadocs](https://jenkins.liteloader.com/job/Mixin/javadoc/org/spongepowered/asm/mixin/injection/modify/BeforeLoadLocal.html)

Exclusively used by `@ModifyVariable` to return `xLOAD` and `xSTORE` instructions in the list which load or store the matching variable respectively.

Notable here is that `STORE` will inject the variable modifier *after* the store instruction as doing before would cause issues.
This is unlike most injection points which will inject new instructions *before* the targeted instruction.

## MIXINEXTRAS:EXPRESSION

Requires MixinExtras 0.5.0+

See [the official MixinExtras tutorial](https://github.com/LlamaLad7/MixinExtras/wiki/Expressions) for an in-depth explanation.

Used to target a wide variety of instructions using a Java-like expression. This can be used to define context around a target,
or to target sets of instructions that would normally not be targetable directly.

Expression-exclusive targets include:

- Comparisons
- Array operations
- Array literals
- Binary expressions
- Method references
- Unary expressions
- String concatenation
- `xLOAD` and `xSTORE` instructions outside of the context of `@ModifyVariable`.

Expressions can be paired with injection point specifiers, slices and ordinals as needed.

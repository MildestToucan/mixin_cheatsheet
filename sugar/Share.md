# @Share

<- [Back to Main](/README.md)

<- [Back to Sugar](/sugar/README.md)

`@Share` is a way for injectors with the same target method to create new local variables in the target that
can be shared between them as an extra parameter. Unlike with a field, `@Share`d values are thread-safe and will
not consume long-term memory.

## External Resources

[Official Wiki Page](https://github.com/LlamaLad7/MixinExtras/wiki/Share)

## Syntax

```java
@Share(
    // The id defining which @Share'd local this parameter is referencing.
    // All handlers that want to share the same reference must have matching ids.
    value = /* String value. Must be specified */

    // Used to ensure uniqueness of IDs across mixin classes.
    // If you need to share across mixin classes, set this manually to something like a mod id.
    namespace = /* String value. Defaults to fully qualified name of the mixin class */
)
```

## Types

Shared parameters should be typed as the appropriate [LocalRef](/sugar/Local#mutable-reference-typing) type.

### Default Values

Since your injectors have to manually set the value of the `LocalRef`, if it is read from before it's written to, it will return the type's default
value for primitives, and `null` for object types.

Injectors reading from a shared `LocalRef` should always have dedicated handling for the default value,
as it is possible that another mixin or modification could cause the injector that was meant to initialize the shared value to be skipped.

## Example

Say you had the following target:

```java
public int foo(int i, int j) {
    double ding = anotherComplexOperation(someComplexOperation(i, j));
    /* ... */
    if (canPetToucans(i, j)) {
        return i;
    }
    return j;
}
```

And you needed to modify `canPetToucans` using the result of `someComplexOperation`. Instead of recalculating, you could
share the value by grabbing it with another injector:

```java
@ModifyArg(method = "foo", at = @At(value = "INVOKE", target = "<member path for anotherComplexOperation>"))
private double grabComplexValue(double ding, @Share("complexValue") LocalDoubleRef complexValueRef) {
    complexValueRef.set(ding);
    return ding;
}

@ModifyExpressionValue(method = "foo", at = @At(value = "INVOKE", target = "<member path for canPetToucans>"))
private boolean expandToucanPettingPrivileges(boolean original, @Share("complexValue") LocalDoubleRef complexValueRef) {
    // Handle the default value in case grabComplexValue is skipped.
    if (complexValueRef.get() == 0.0F) {
        return original;
    }
    return original || complexValueRef.get() == 5.0F;
}
```

Which would result in:

```diff
public int foo(int i, int j) {
+   LocalDoubleRefImpl sharedRef7 = new LocalDoubleRefImpl();
-   double ding = anotherComplexOperation(someComplexOperation(i, j));
+   double injectorAllocatedLocal5 = someComplexOperation(i, j);
+   double ding = anotherComplexOperation(this.modify$zzb000$test_env$grabComplexValue(injectorAllocatedLocal5, sharedRef7);
    /* ... */
-   if (canPetToucans(i, j)) {
+   if (this.modifyExpressionValue$zzb000$test_env$expandToucanPettingPrivileges(canPetToucans(i, j), sharedRef7)) {
        return i;
    }
    return j;
}
```

# @Local

This does not cover the usage of `@Local` in the context of Expression `@Definition`s, but as a sugar parameter.

## External Resources

[Official Wiki Page](https://github.com/LlamaLad7/MixinExtras/wiki/Local) (Read that if you're not already familiar!!!!!!!!!)

## Syntax

### Annotation

```java
@Local(
    // If true, aborts the injection, and then prints the available locals in the terminal.
    // Primarily useful to debug when figuring out which locals are available where,
    // and/or how to specify them becomes confusing.
    print = /* boolean value, false by default */,

    // Whether to restrict the search for the target local to the target method arguments.
    // Should always be true when relevant.
    argsOnly = /* boolean value, false by default */,

    // The LVT index for the targeted local. Zero-indexed.
    // This doesn't take the type of the local into account, so it is very brittle.
    index = /* int value, -1 (unused) by default */,

    // The ordinal for the targeted local. Zero-indexed.
    // If there are multiple locals of the same type available,
    // this picks a specific one as if using an array or list index.
    ordinal = /* int value, -1 (unused) by default */,

    // NOT AVAILABLE ON OBFUSCATED TARGETS.
    // Specify the names for the desired local.
    // When available and local usage is needed, specifying by name is,
    // the least brittle of the options, generally speaking.
    name = /* String[] value, empty (unused) by default */
)
```

## Local Types

The type of the annotated parameter should match the target local's type unless a mutable reference is needed.

### Mutable References

If the targeted local needs to be mutated, using the `LocalRef` equivalent is necessary:

The appropriate `LocalRef` is as follows:

| Type        | Specialized `LocalRef` |
| ----------- | ---------------------- |
| Object Type | `LocalRef<ObjectType>` |
| `byte`      | `LocalByteRef`         |
| `char`      | `localCharRef`         |
| `double`    | `localDoubleRef`       |
| `int`       | `localIntRef`          |
| `long`      | `localLongRef`         |
| `short`     | `localShortRef`        |
| `boolean`   | `LocalBooleanRef`      |

The `LocalRef` object will wrap its value and expose it with a getter and setter methods.

After the handler method, the targeted local variable will have its value set to the `LocalRef`'s wrapped value.

## Example

```java
public void foo(int i, int j) {
    double ding = someComplexOperation(i, j);
    bar(i);
}
```

You may need to target `bar`, while needing to reference `ding`, you might want to avoid recalculating `someComplexOperation`, and be able to also
have the value conserve any of the modifications it might've received from other mixins. To do so, you would use a `@Local` like so:

```java
@WrapOperation(method = "foo", at = @At(value = "INVOKE", target = /* [bar member path here] */))
private void wrapBar(int i, Operation<Void> original, @Local double ding) {
    original.call(i);
    someCustomOperation(ding);
}
```

Which would produce the result:

```diff
public void foo(int i, int j) {
    double ding = someComplexOperation(i, j);
-   bar(i);
+   Operation var10002 = var0 -> {
+       WrapOperationRuntime.checkArgumentCount(var0, 1, "[int]");
+       bar((Integer)var0[0]);
+       return (Void)null;
+   };
+   LocalDoubleRefImpl ref6 = new LocalDoubleRefImpl();
+   ref6.init(ding);
+   this.wrapOperation$zza000$test_env$wrapBar$mixinextras$bridge$5(i, var10002, (double)ref6);
+   ding = ref6.dispose();
}
```

The local variable is passed to a generated `LocalDoubleRefImpl` which is then used by our handler.

Once our handler's callback is over, the local's value is passed back to the original variable.

### Mutable Reference

```java
@WrapOperation(method = "foo", at = @At(value = "INVOKE", target = /* [bar member path here] */))
private void wrapBar(int i, Operation<Void> original, @Local LocalDoubleRef dingRef) {
    double ding = dingRef.get();
    original.call(i);
    someCustomOperation(ding);
    if (someCondition) {
        dingRef.set(ding + customValue());
    }
}
```

The only difference in how it affects the target method is that the generated `LocalDoubleRefImpl` doesn't need to be casted
to `double` to be passed to our handler.

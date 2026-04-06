# @Inject

<- [Back to Main](/README.md)

<- [Back to Injectors](/injectors/README.md)

## External Resources

[JavaDocs](https://jenkins.liteloader.com/job/Mixin/javadoc/org/spongepowered/asm/mixin/injection/Inject.html)

## Syntax

```java
@Inject(
    method = "targetMethod",
    at = @At(/* See the dedicated @At page. */),
    cancellable = /* Set this to true if you need to cancel. */
)
// The handler method should return void.
private void handlerMethod(
    /* Target method params go here, optional. */ 
    /* If the target returns a value: */
    CallbackInfoReturnable<TargetMethodReturnType> cir
    /* Else, if the target returns void: */
    CallbackInfo ci
    /* As with other injectors, you can then add MixinExtras Sugar params. */
) {
    /* == Cancelling == */
    // If you want to cancel the target method, call:
    // If targeting a void method:
    ci.cancel();
    // Else:
    cir.setReturnValue(/* The value to return early */);
    
    /* == Getting the return value == */
    // When injecting at RETURN or TAIL you can:
    TargetMethodReturnType returnValue = cir.getReturnValue();
    // If the return value is a primitive, you can also use the appropriate specialized variant.
    // The variant is of the same name, but with the descriptor character of the primitive appended:
    byte returnB = cir.getReturnValueB();
    char returnC = cir.getReturnValueC();
    double returnD = cir.getReturnValueD();
    int returnI = cir.getReturnValueI();
    long returnJ = cir.getReturnValueJ();
    short getValueS = cir.getReturnValueS();
    boolean getValueZ = cir.getReturnValueZ();
}
```

## Effects

Injects a call to the handler method, and optionally an early return if cancelled.
With most injection points, the call will be injected before the specified instruction unless shifting `AFTER` it explicitly.

The handler's injected call is also accompanied by the instantiation of the `CallbackInfo` or `CallbackInfoReturnable` object
that it will be passed as a parameter. If cancellable, the inject's call is followed by a conditional early return.

### Usage Concerns

`@Inject` is often overused when it is not fitting, leading to more brittle or less compatible injectors. Keep in mind
that cancelling in an inject does not chain, and should almost never be done unconditionally.

For modifying an existing return value, prefer `@ModifyReturnValue` or `@WrapMethod`, or an injector that modifies the
value in the return statement more directly.

`@Inject` is best-suited for adding new operations to a target method, and should only be used if it specifically fits your
purposes, rather than just because it fits your goals.

### Example

To inject a call at the top of the following method, and optionally return early:

```java
public class Target {
    public int foo(int x, int y, int z) {
        x += (y * z);
        return z + x;
    }
}
```

You may do an inject like:

```java
@Inject(method = "foo", at = @At("HEAD"), cancellable = true)
private void cancelFooIfZGreater(int x, int y, int z, CallbackInfoReturnable<Integer> cir) {
    if (z > y && z > x) {
        cir.setReturnValue(30);
    }
}
```

The resulting decompiled bytecode in the target method might look something like:

```diff
public class Target {
    public int foo(int x, int y, int z) {
+       CallbackInfoReturnable callbackInfo4 = new CallbackInfoReturnable("foo", true);
+       this.handler$zza000$modid$cancelIfZGreater(x, y, z, callbackInfo4);
+       if (callbackInfo4.isCancelled()) {
+           return callbackInfo4.getReturnValueI();
+       }
        x += y * z;
        return z + x;
    }

+   @MixinMerged(
+   mixin = "io.github.mildesttoucan.mixin.ExampleMixin",
+   priority = 1000,
+   sessionId = "0826c6c7-f4b2-4979-8192-f6913af13e3f"
+   )
+   private void handler$zza000$modid$cancelIfZGreater(int x, int y, int z, CallbackInfoReturnable cir) {
+       if (z > y && z > x) {
+           cir.setReturnValue(30);
+       }
+   }

}
```

The main thing to keep in mind as a user of Mixin is that the callbackInfo, if cancellable, will add an early return when the handler cancels it.

Another thing to note on the technical side is that since the target method returns a primitive `int`, `CallbackInfoReturnable` calls `getReturnValueI()` to avoid
dealing with `Integer` boxing. If the method returned `Integer`, it'd instead return something closer to `return (Integer) callbackInfo4.getReturnValue();`

## Niche Knowledge & Edge Cases

### Non-void Inject Handlers

For all intents and purporses, `@Inject` handlers shouldn't return a value, as that is the documented, expected, and standard behavior.
However, it's technically possible to use it to modify the return value in a way that is compatible, somewhat similarly to `@ModifyReturnValue`.

This is absolutely not a recommended thing to do, it is not convenient, not well-documented, and it is uncertain how well it works/will keep working across
Mixin versions.

To do so, instead of a cancellable inject, we target `RETURN` or `TAIL`, and return a value matching the target method's type. For example:

For the target method:

```java
public boolean foo(int x, int y, int z) {
    if ((x + y) == z) {
        return true;
    }
    return y == z;
}
```

And the inject:

```java
@Inject(method = "foo", at = @At("RETURN"))
private boolean changeFooReturns(int x, int y, int z, CallbackInfoReturnable<Boolean> cir) {
    return cir.getReturnValueZ() || x == y;
}
```

In the decompiled source, we'd get:

```java
public boolean foo(int x, int y, int z) {
    if (x + y == z) {
        CallbackInfoReturnable callbackInfo4 = true;
        CallbackInfoReturnable var6 = new CallbackInfoReturnable("foo", false, callbackInfo4);
        return this.handler$zza000$test_env$changeFooReturns(x, y, z, var6);
    } else {
        CallbackInfoReturnable callbackInfo5 = y == z;
        CallbackInfoReturnable var7 = new CallbackInfoReturnable("foo", false, callbackInfo5);
        return this.handler$zza000$test_env$changeFooReturns(x, y, z, var7);
    }
}
```

You'll notice the very odd local variables of the type `CallbackInfoReturnable` being assigned to boolean values. How jank and fascinating this is.
Why this is happening in that way, I'm not sure.

### Legacy local capture

Local capture using this feature is not recommended, use [`@Local`](/sugar/Local.md) instead.

`@Inject` also has a `localCapture` attribute. It allows you to append parameters matching the target method's LVT
to the handler's signature after the callbackinfo parameter, excluding the target method parameters. It's possible to
omit any locals that are later in the LVT than the ones you need.

The feature makes the assumption the target method parameters are prior to the callbackinfo.

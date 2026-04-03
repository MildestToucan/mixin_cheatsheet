# @Inject

## Syntax

```java
@Inject(
    method = "targetMethod",
    at = @At(/* See the dedicated @At page. */),
    cancellable = /* whether you cancel the target method. Default false. */
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
    TargetMethodReturnType return = cir.getReturnValue();
}
```

## Effects

Injects a call to the handler method, and optionally an early return if cancelled.
With most injection points, the call will be injected before the specified instruction unless shifting `AFTER` it explicitly.

The handler's injected call is also accompanied by the instantiation of the `CallbackInfo` or `CallbackInfoReturnable` object
that it will be passed as a parameter. If cancellable, the inject's call is followed by a conditional early return.

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

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

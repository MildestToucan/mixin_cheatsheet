# @Cancellable

<- [Back to Main](/README.md)

<- [Back to Sugar](/sugar/README.md)

`@Cancellable` is a sugar parameter annotation that can be used to get a `CallbackInfo` or `CallbackInfoReturnable` on injectors other
than `@Inject`, allowing any injector to cancel a target method.

## External Resources

[MixinExtras Wiki Page](https://github.com/LlamaLad7/MixinExtras/wiki/Cancellable)

## Syntax

`@Cancellable` has no annotation attributes. It is used on a `CallbackInfo` or `CallbackInfoReturnable<TargetMethodReturnType>`
parameter appended to the signature of a non-`@Inject` injector handler. Cancelling syntax is the same as with [`@Inject`](/injectors/Inject.md).

## Niche Interactions

With `@WrapOperation` specifically, the same `@Cancellable` callbackinfo is passed to all handlers in a chain of `@WrapOperation`s.
You can therefore use `CallbackInfo#isCancelled()` or `CallbackInfoReturnable#getReturnValue()` to check if another `@WrapOperation`
in the chain applied before you has cancelled, and react accordingly.

---
layout: sip
title: SIP 25 - @static fields and methods in Scala objects(SI-4581)
disqus: true
---

__Dmitry Petrashko__

__first submitted TODO 2016__

## Motivation ##

We would like to allow methods and fields to be compiled as static. This is usable for interop with Java and other JVM languages and is convenient for optimizations.

## Use Cases

Some JVM and JavaScript frameworks require classes to have specific static fields.

For example, classes extending `android.os.Parcelable` are required to have a static field named `CREATOR` of type `android.os.Parcelable$Creator`.

Another example is using an [`AtomicReferenceFieldUpdater`](http://docs.oracle.com/javase/7/docs/api/java/util/concurrent/atomic/AtomicReferenceFieldUpdater.html).

@sjrd could you please illustrate JS exaples?

## Syntax ##
In order for method or field to be considered static it needs to be defined in an `object` and annotated `@static`.
There is no special syntax proposed to access these members, they are accessed as if they were a member of defining objects with all appropriate access requirements for accessing them.

## Restrictions ##

The following rules ensure that method can be correctly compiled into static member on JVM:

0. Only objects can have fields annotated as `@static`

1. The fields annotated with `@static` shoudld preceed any non-`@static` fields. This ensures that we do not introduce surprises for users in initialization order.

2. The right hand side of method or field annotated as `@static` can only refer to members of globally accessable objects and `@static` members.

3. If member `foo` of `object C` is annotated `@static`, companion class `C` is not allowed define term members with name `foo`. 

4. If member `foo` of `object C` is annotated `@static`, companion class `C` is not allowed to inherit classes that define a term member with name `foo`..

5. If companion `object P` defines an `@static` method or field `foo`, classes inheriting from companion `class P` are not allowed to define term members with name `foo`.

## Compilation scheme ##
The current proposed scheme piggybacks on already existing scoping restrictions in typer, thus requiring `@static` methods to be defined in `objects`.
If implemented in dotty code base, such modifications would be needed:
 - extend RefChecks to check restrictions 2 and 3. This can be done in a separate mini-phase;
 - extend LamdaLift.CollectDependencies to be aware that accessing member annotated `@static` should not trigger capturing object that contains this member;
 - extend LambdaLift to trigger error if `@static` annotated method cannot be lifted to top level scope;
 - extend GenBCode to emmit static fields and methods in classes.

## Comparison with @lrytz [proposal](https://gist.github.com/lrytz/80f3141de8240f9629da) ##
Lucas Rytz has proposed a similar SIP, but his SIP requires changes to typer to ensure that `@static` fields do not capture `this`. 
It also does not address the question of `@static` members in inner objects.

## See Also ##
[Si
[Another proposal by @lrytz](https://gist.github.com/lrytz/80f3141de8240f9629da)
[Old discussion on scala-internals mailing list](https://groups.google.com/forum/#!searchin/scala-internals/static/scala-internals/vOps4k8CADY/Dq1I3Ysvao0J)

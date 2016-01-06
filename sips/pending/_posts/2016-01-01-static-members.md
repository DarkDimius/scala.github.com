---
layout: sip
title: SIP 25 - Trait Parameters
disqus: true
---

__Dmitry Petrashko__

__first submitted 18 June 2015__

## Motivation ##

We would like to allow methods and fields to be compiled as static. This is usable for interop with Java and other JVM languages and is convenient for optimizations.

## Syntax ##
In order for method or field to be considered static it needs to be defined in an `object` and annotated `@static`.

## Restrictions ##

The following rules ensure that method can be correctly compiled into static member on JVM:

1. The right hand side of method or field annotated as `@static` can only refer to members of globally accessable objects and `@static` members.

2. If member `foo` of `object C` is annotated `@static`, companion class `C` is not allowed define term members with name `foo`. 

3. If member `foo` of `object C` is annotated `@static`, companion class `C` is not allowed to inherit classes that define a member with name `foo`. 

## Compilation scheme ##
The current proposed scheme piggybacks on already existing scoping restrictions in typer, thus requiring `@static` methods to be defined in `objects`.
If implemented in dotty code base, such modifications would be needed:
 - extend RefChecks to check restrictions 2 and 3. This can be done in a separate mini-phase;
 - extend LamdaLift.CollectDependencies to be aware that accessing member annotated `@static` should not trigger capturing object that contains this member;
 - extend LambdaLift to trigger error if `@static` annotated method cannot be lifted to top level scope;
 - extend GenBCode to emmit static fields and methods in classes.

## See Also ##

[Dotty Issue #640](https://github.com/lampepfl/dotty/issues/640)

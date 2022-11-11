---
title: "11 Micro-optimisation Tips for Skyrim's Scripting Language"
date: 2022-11-11 11:00:00 +0100
toc: true
toc_label: "Contents"
toc_sticky: false
categories:
  - "Skyrim" 
  - "Papyrus" 
  - "Programming"
  - "Optimisations"
excerpt: "The *Papyrus* scripting language of *The Elder Scrolls V: Skyrim* is somewhat notorious in the game's
  large community of third-party mod developers and users for performance-related issues (though there are
  also [plenty of misconceptions](https://www.reddit.com/r/skyrimmods/comments/45cdup/long_ramble_about_scripts_and_how_they_affect/),
  and it's not necessarily as bad as many people think). There are already many relatively well-known tips out there
  for useful optimisations, but in this post I will also include a few that I have not yet seen elsewhere.
  All of the tips in this post are based on real code that I've seen looking at scripts from various popular
  Skyrim mods."
---

The *Papyrus* scripting language of *The Elder Scrolls V: Skyrim* is somewhat notorious in the game's
large community of third-party mod developers and users for performance-related issues (though there are
also [plenty of misconceptions](https://www.reddit.com/r/skyrimmods/comments/45cdup/long_ramble_about_scripts_and_how_they_affect/),
and it's not necessarily as bad as many people think). There are already many relatively well-known tips out there
for useful optimisations, but in this post I will also include a few that I have not yet seen elsewhere.
All of the tips in this post are based on real code that I've seen looking at scripts from various popular
Skyrim mods.

## The Papyrus Engine

Before diving into some useful optimisations that can be used in Papyrus scripts, it's good to take a moment to
get a better understanding of how Papyrus scripts are run within the game's engine, and what implications this has
on the performance of scripts.

### Fixed Papyrus Budget per Frame

One of the main things to know about the engine is that 
**the game engine has a fixed time budget that it allocates for running active Papyrus scripts on the main thread per frame**. By
default this budget is 1.2ms per frame, but this can (probably) be edited by tweaking a setting in the game's .ini
file (which I don't think I would recommend doing). That is 1.2 milliseconds per frame for **all** synced native calls from actively running
scripts in that frame combined, it's not *per* script. The game engine will distribute this time a bit over all active
scripts, but not let them use any more time. It prefers for scripts to take longer and get spread out over multiple frames,
[rather than allowing heavy scripts to spend more time and cause the game's framerate to be lowered](https://www.creationkit.com/index.php?title=Differences_from_Previous_Scripting#Performance).

This has two implications:
1. An expensive or poorly-written script, or simply a large number of actively running scripts, will never cause the game's framerate to slow down.
2. Expensive or poorly-written scripts, or simply a large number of actively running scripts, can cause other simultaneously-running scripts to 
become slower / less responsive. If the same time budget (per frame) needs to be spread out over more scripts, that's less time available per 
script per frame (and hence fewer instructions executed).

The second point is sometimes no problem at all, but can also sometimes snowball into a large problem. It's fine if there are occasionally
a few frames where the "demand" for script-execution time exceeds the available budget, as long as there are also frames around it in which
the demand is lower and some scripts can get finished. But if there is a relatively consistent high load, where new scripts with new instructions
that want to get executed keep getting added faster than the game can execute them, we have a self-reinforcing problem that can snowball out of control.
The execution of scripts can get delayed by very noticeable, substantial amounts. 

With this in mind, I would argue that it is not just important to optimise scripts for which you care that they actually run quickly, but
also optimise any other scripts (even if delays in them would be fine) to minimise how much your non-time-sensitive scripts are affecting
other scripts---which themselves may be highly time-sensitive.

### Fixed Budget per Frame per Script

In SSE (Skyrim's Special Edition, the 64bit version of the game, including the later Anniversary Edition), 
[it appears that the game engine also enforces a fixed budget per frame per script](https://www.reddit.com/r/skyrimmods/comments/wupg1j/papyrus_engine_is_400_times_slower_on_ae_than_le/),[^1]
in addition to the fixed budget per frame for all active scripts together as discussed above. In comparison to the original 32bit version of Skyrim, this would:
1. Reduce the likelihood that a single big script slows down others.
2. Not change the likelihood of a large number of simultaneously-active scripts (regardless of size) together slowing down others.
3. Increase the likelihood that you'll find your own script or function to be extremely slow if it is big (has many instructions to be executed).

Interestingly, this budget does not seem to be a time budget, but rather an instructions budget (judging by various experiments reported in the page
I linked above). Hence, any simple little instruction you can shave off a script can make a difference here, regardless of whether it is an expensive
instruction (like function calls, which are notoriously expensive in Papyrus), or something cheap like casting a variable to a different type.

### Note on Profiling-guided Optimising & Papyrus

Normally in most programming languages, and most software, we would rather not dive into micro-optimisations without profiling our code.
[Profiling is indeed also possible in Papyrus](https://www.nexusmods.com/skyrim/articles/52764/), but in my opinion there are some peculiarities
about working with Papyrus in Skyrim that make profiling less interesting, and just diving into the code and applying micro-optimisations more
interesting:
1. Real-world code (i.e., scripts in mods or in the base game) are generally very short and fast (in terms of wall time), making the accuracy
of timings a bit questionable. If you artificially make them slower (e.g., by taking the same code you would realistically only want to run
once, but putting repeating it many times in a loop) such that you actually have something to measure, you start hitting a bunch of limitations
caused by the different budgets described above that you wouldn't have hit otherwise.
2. Even if something is already very fast, micro-optimisations may still be worth it because they can stop your scripts from negatively affecting
other scripts. Due to the various budgets that the engine uses, there can be "sudden" effects, where even a single instruction more or less can 
make a full frame's difference in terms of whether or not your script is still running.
3. Different users (modded Skyrim players) have wildly different setups. Some use no mods, some use a few mods, and others use hundreds or thousands
of mods. Even if a little optimisation has no noticeable impact on your setup, it may have a noticeable effect on other people's installations.
4. Papyrus scripts are usually dozens of lines, not tens of thousands. So investing a little bit of time into some micro-optimisations is usually
not actually a lot of work at all.

## Papyrus Micro-optimisations

### Highest-impact Micro-optimisations

First, I will discuss some of the highest-impact micro-optimisations that script authors should generally look towards first. I will still only
be focusing on micro-optimisations though, so not, for example, major algorithm overhauls. I will also not get into detail on some of the more
well-known concerns around [RegisterForUpdate()](https://www.creationkit.com/index.php?title=RegisterForUpdate_-_Form) 
*(hint: you should probably use [RegisterForSingleUpdate()](https://www.creationkit.com/index.php?title=RegisterForSingleUpdate_-_Form) instead)*, 
[OnHit](https://www.creationkit.com/index.php?title=OnHit_-_ObjectReference) 
*(hint: consider using [PAPER's OnImpact event if possible](https://github.com/DennisSoemers/PAPER/wiki/New-Papyrus-Events#onhit-events))*,
or [OnMagicEffectApply](https://www.creationkit.com/index.php?title=OnMagicEffectApply_-_ObjectReference) 
*(hint: consider using various events from [Po3's Papyrus Extender](https://github.com/powerof3/PapyrusExtenderSSE) instead, as in
the [Vanilla Scripting Enhancements mod](https://www.nexusmods.com/skyrimspecialedition/mods/68139))* either.

#### 1. Avoiding Game.GetPlayer()

Let's start with an easy one. [Game.GetPlayer()](https://www.creationkit.com/index.php?title=GetPlayer_-_Game) is a function that returns a reference to the Player. However,
function calls are slow in Papyrus (especially when they aren't in [the list of non-delayed native functions](https://www.creationkit.com/index.php?title=Category:Non-delayed_Native_Function)). 
Instead, we can simply get a reference to the Player through [a property](https://www.creationkit.com/index.php?title=Variables_and_Properties_(Papyrus)#Declaring_Properties).
This will always be substantially faster. 

#### 2. Avoiding abbreviated versions of functions

For many native functions, there are also abbreviated versions available that do exactly the same thing, but just have a slightly shorter function name.
Unfortunately, none of these were implemented to be native functions (with different names, but mapping to the same native code) themselves. Instead, they
were all implemented as single-line Papyrus functions that just call the corresponding native function with the longer name. This means that they are
strictly slower (because they have additional overhead from an extra Papyrus function call), and should really just never be used. Some examples include:

- [GetAV()](https://www.creationkit.com/index.php?title=GetActorValue_-_Actor): should call [GetActorValue()](https://www.creationkit.com/index.php?title=GetActorValue_-_Actor) instead.
- [GetAVPercentage()](https://www.creationkit.com/index.php?title=GetActorValuePercentage_-_Actor): should call [GetActorValuePercentage()](https://www.creationkit.com/index.php?title=GetActorValuePercentage_-_Actor) instead.
- [GetBaseAV()](https://www.creationkit.com/index.php?title=GetBaseActorValue_-_Actor): should call [GetBaseActorValue()](https://www.creationkit.com/index.php?title=GetBaseActorValue_-_Actor) instead.
- [DamageAV()](https://www.creationkit.com/index.php?title=DamageActorValue_-_Actor): should call [DamageActorValue()](https://www.creationkit.com/index.php?title=DamageActorValue_-_Actor) instead.
- [ModAV()](https://www.creationkit.com/index.php?title=ModActorValue_-_Actor): should call [ModActorValue()](https://www.creationkit.com/index.php?title=ModActorValue_-_Actor) instead.
- [SetAV()](https://www.creationkit.com/index.php?title=SetActorValue_-_Actor): should call [SetActorValue()](https://www.creationkit.com/index.php?title=SetActorValue_-_Actor) instead.
- [ForceAV()](https://www.creationkit.com/index.php?title=ForceActorValue_-_Actor): should call [ForceActorValue()](https://www.creationkit.com/index.php?title=ForceActorValue_-_Actor) instead.
- [RestoreAV()](https://www.creationkit.com/index.php?title=RestoreActorValue_-_Actor): should call [RestoreActorValue()](https://www.creationkit.com/index.php?title=RestoreActorValue_-_Actor) instead.
- [GetRef()](https://www.creationkit.com/index.php?title=GetReference_-_ReferenceAlias): should call [GetReference](https://www.creationkit.com/index.php?title=GetReference_-_ReferenceAlias) instead.

Closely related to the above examples, it is also good to avoid short convenience functions that only wrap what would really be a short, single line of code anyway, even
if it abstracts multiple instructions. Of course, having functions with meaningful names wrapping functionality is good from the points of view of code readability and maintenance,
but can be detrimental to performance due to function call overhead. For example:

- Should use `GlobalVariable.GetValue() as int` (manually adding the cast to an int) instead of the 
[GetValueInt() convenience function](https://www.creationkit.com/index.php?title=GetValueInt_-_GlobalVariable).
- Should use `GetReference() as Actor` (manually adding the cast to an Actor) instead of the 
[GetActorReference() or GetActorRef() convenience functions](https://www.creationkit.com/index.php?title=GetActorReference_-_ReferenceAlias).

### Lower-impact Micro-optimisations

Next, I will look at some even lower-level, and probably lower-impact, micro-optimisations. Many of these no longer really involve avoiding
Papyrus function call overhead or things that are slow due to concurrency / syncing with framerate, but they rather involve optimising away
some low-level operations.

#### 3. Do not write `b == true`

I often see code like this:

```
Function TestEqualToTrue(bool b)
	If (b == true)		; Do NOT do this!
		Debug.Notification("test")
	EndIf
EndFunction
```

My issue with this code is that the comparison to `true` is completely unnecessary. An `if`-statement just
requires a single expression that evaluates to a `bool`. The variable `b` is *already* a `bool`, so we can use
that directly: there is no need to compare it to `true` (which would be a more complex expression that ultimately
just gives us the value of `b` back again)! The more efficient implementation would be as follows:

```
Function TestTrue(bool b)
	If (b)			; This is better!
		Debug.Notification("test")
	EndIf
EndFunction
```

To show that this actually makes a difference, we can use the [Champollion decompiler](https://www.nexusmods.com/skyrim/mods/35307)
to decompile compiled versions of scripts with those functions, and annotate the decompiled source code with assembly:

```
function TestEqualToTrue(Bool b)

	;Bool ::temp3
	;None ::NoneVar
	
	; 000 : cmp_eq ::temp3 b true 
	; 001 : jmpf ::temp3 004
	if b == true
		; 002 : callstatic debug Notification ::NoneVar "test" 
		debug.Notification("test")
	; 003 : jmp 004
	endIf
endFunction

function TestTrue(Bool b)

	;None ::NoneVar
	
	; 000 : jmpf b 003
	if b
		; 001 : callstatic debug Notification ::NoneVar "test" 
		debug.Notification("test")
	; 002 : jmp 003
	endIf
endFunction
```

In the first version, we have to (1) run `cmp_eq` to compare `b` to `true`, (2) put the outcome in a temporary variable 
`::temp3`, and (3) jump to `004` (past the `debug.Notification()` call) if the comparison was false. In the second version,
we just directly jump based on the value of `b`: no comparison, and no temporary variable!  

#### 4. Do not write `b != true`

Similar to the previous idea, I find the following code (including assembly in comments) problematic:

```
function TestNotEqualToTrue(Bool b)

	;Bool ::temp0
	;None ::NoneVar
	
	; 000 : cmp_eq ::temp0 b true 
	; 001 : not ::temp0 ::temp0 
	; 002 : jmpf ::temp0 005
	if b != true
		; 003 : callstatic debug Notification ::NoneVar "test" 
		debug.Notification("test")
	; 004 : jmp 005
	endIf
endFunction
```

The `if b != true` line compiles into three instructions: (1) A `cmp_eq` operation to compare `b` and `true` for 
equality, putting the result in a `::temp0` variable. (2) A `not` operation to negate `::temp0`. (3) A `jmpf` that
jumps past the `debug.Notification()` call if `::temp0` is false.

Better solutions are to write either (1) `if !b`, or (2) `if b == false`. The first solution avoids the `cmp_eq`
(it only needs to negate and jump), whereas the second solution avoids the `not` (it only needs to compare and jump).

#### 5. Use the expected type for function arguments if possible

If a function in Papyrus expects parameters of a certain type, it will *only* take constants or variables of 
*exactly* that type. The compiler will not complain if you give it something that can, with 100% certainty, be
cast into the expected type... but it will insert an extra casting operation into the compiled code.
Even for constants, **the compiler is not smart enough to perform these casts at compile-time**.

For example, the [Utility.Wait](https://www.creationkit.com/index.php?title=Wait_-_Utility) function expects
a single argument of the `float` type. So, what should we write if we want our script to wait for one second?
Quite often, I see this: `Utility.Wait(1)`. But if we decompile a function with this line of code in it, and
include assembly in the comments again, we get the following:

```
function WaitInt()

	;Float ::temp4
	;None ::NoneVar
	
	; 000 : cast ::temp4 1 
	; 001 : callstatic utility Wait ::NoneVar ::temp4 
	utility.Wait(1 as Float)
endFunction
```

We don't even have to look at the assembly, but can just look at the decompiled source code in the last line,
to see something interesting: the `1` constant was changed into `1 as Float`. An explicit casting operation
has been inserted, which gets executed (at runtime!) every time this line of code runs. The assembly shows
that, in practice, this does not only mean that we have the extra `cast` operation, but also an extra 
`::temp4` variable to hold the result of the cast.

In contrast, if we simply write `1.0` instead of `1`, such that we have a constant of the proper type
(`float` instead of `int`), we get the following decompiled code (annotated with assembly):

```
function WaitFloat()

	;None ::NoneVar
	
	; 000 : callstatic utility Wait ::NoneVar 1.00000 
	utility.Wait(1.00000)
endFunction
```

In the example of `Utility.Wait()`, it's just a single parameter. But there are also many worse cases out
there, such as [`PlayImpactEffect()`](https://www.creationkit.com/index.php?title=PlayImpactEffect_-_ObjectReference).
This function has four different `float` parameters which, in many scripts
I've seen out there, are all often set to `int` constants. In fact, even the example line of code on the official
documentation page gets it wrong (using `0, 0, -1, 512` as arguments instead of `0.0, 0.0, -1.0, 512.0`)!

While this idea most often applies to `int` and `float` constants, it can also apply to other types. For example,
I've run into some scripts that store a reference in a variable of type `Actor`, but exclusively use it as an
argument for functions that except parameters of type `ObjectReference` (of which `Actor` is a subclass). This,
unnecessarily, produces a runtime cast from `Actor` to `ObjectReference` every time it is passed into a function. 
This is a rare case though, since usually, when we have an object of type `Actor`, we have a mix of calls where
some really require it to be of that type, and only some expect just the `ObjectReference`. In these cases, I would
simply recommend only keeping the `Actor` variable around, and not worry about how it sometimes gets cast.

#### 6. Compare variables to constants of matching type

The previous point focused on arguments passed into functions, but the same idea applies when comparing
variables to constants with (in)equality operators (i.e., `<`, `<=`, `==`, `>=`, `>`, or `!=`). If you have a
`float` variable that you want to compare to some constant, you should make sure to also type the constant
as a `float`. 

```
function SomeFunction(float f)
	if (f > 1)		; Do NOT do this! The 1 will be cast at runtime to 1.0!
		; some code here
	endIf
	
	if (f > 1.0)	; This is better!
		; some code here
	endIf
endFunction
```

#### 7. Directly assign boolean expression to variable

Consider the following code:

```
bool a = False

if (some expression that resolves to a boolean result)
	a = True
else
	a = False
endIf
```

In this code, the variable `a` actually always gets set to whatever result comes out of the (maybe simple, or maybe complex)
boolean expression used for the `if`-condition. This means that it is wasteful to actually have the `if` there at all, and
the following would be more efficient:

```
a = (some expression that resolves to a boolean result)
```

A variant of the same issue that I also sometimes see is the following:

```
; Some bool variable named "a" is guaranteed to be False when this code runs

if (some expression that resolves to a boolean result)
	a = True
endIf
```

Again, the same solution as above would be better in this case. However, in this case it is really important to make sure
that the `a` variable indeed is guaranteed to be `False` at the time when you enter this piece of code! If there is any
chance that it might already be `True`, omitting the `if`-block would change what the code does.

#### 8. Assigning negation of variable used in condition

I have seen the following construction used in some mods' scripts:

```
Function AssignNegation(bool b)
	If (b)
		b = !b
		; Some other code here too
	Else
		b = !b
		; Some other code here too
	EndIf
EndFunction
```

Note that it's actually important that there's also additional code in at least one of either the
`If`- or the `Else`-block, because otherwise we just have a variant of the issue previously described
issue and we shouldn't be using `If`- and `Else`-blocks at all. So, let's pretend that the blocks
are actually meaningful, and look at the decompiled code with assembly annotations in comments:

```
function AssignNegation(Bool b)

	;Bool ::temp5
	
	; 000 : jmpf b 004
	if b
		; 001 : not ::temp5 b 
		; 002 : assign b ::temp5 
		b = !b
	; 003 : jmp 006
	else
		; 004 : not ::temp5 b 
		; 005 : assign b ::temp5 
		b = !b
	endIf
endFunction
```

We can see that `b` is not directly set to its own negation. A temporary variable `::temp5` is used
to hold the negation of `b`, and then the value of that variable is assigned to `b` again. But, in
each of the lines of code that perform this assignment, we actually already know (at compile-time)
what value we wish to assign! Either we are inside the `If`-block (where we know `b` must
be `True`, and we want it to be set to `False`), or we are inside the `Else`-block (where we know
`b` must be `False`, and we want it to be set to `True`). If we directly use those constants, we
get the following code, annotated with assembly that shows us how it is more efficient:

```
function AssignNegatedConstant(Bool b)

	
	; 000 : jmpf b 003
	if b
		; 001 : assign b false 
		b = false
	; 002 : jmp 004
	else
		; 003 : assign b true 
		b = true
	endIf
endFunction
```

#### 9. Avoid checking something that you already know

Consider the following code:

```
if b
	; some code here
elseIf b == false	; Do not do this!
	; some other code here
endIf
```

As soon as we go past the `if`-block (i.e., into a subsequent `elseIf` or `else`), we already know
that `b` must evaluate to `false`. So, there is no need to verify that it is `false`, and we should
rather write the following:

```
if b
	; some code here
else
	; some other code here
endIf
```

This is especially important in cases where `b` is a more complex boolean expression (including one
or more function calls or just a longer chain of multiple boolean variables), but technically also
still relevant in the most simple case where `b` is just a single variable.

#### 10. Nest conditions instead of repeating them

Consider this code:

```
if a && b
	; some code here
elseIf a && c
	; some other code here
endIf
```

In cases where `a` or `b` are false, and we drop into checking the condition for the `elseIf` line,
we will waste time testing the value of "`a`" a second time, even though we just already did that for
the first `if` line. It is more efficient (again, especially if `a` is a more complex expression) to
rewrite this as follows:

```
if a
	if b
		; some code here
	elseIf c
		; some other code here
	endIf
endIf
```

#### 11. Factoring out value added to lower and upper bounds for `Utility.RandomInt`

For the final tip of this post, I have a rather specific case that I ran into looking
at various mods' scripts. The original script included this expression (where `x` was
some `int` variable):

```
Utility.RandomInt(x + 5, x + 20)
```

The [`Utility.RandomInt()`](https://www.creationkit.com/index.php?title=RandomInt_-_Utility) function
takes two arguments. The first is a lower bound, and the second an upper bound. It returns a random
integer between those bounds (in contrast to most programming languages, both inclusive!). So, the
above expression simply picks a random integer between `x + 5` and `x + 20`. There is a slightly more
efficient way to write the same thing though:

```
x + Utility.RandomInt(5, 20)
```

If we write the code like this, we first generate a random integer between `5` and `20`, and afterwards
add `x` to whichever random value was returned. This ultimately leads to a random value being drawn from
the same distribution, but it is slightly more efficient because the operation of adding `x` to another
value is only executed once instead of twice.

[^1]: Nightfallstorm's recently-released [Papyrus Tweaks NG](https://www.nexusmods.com/skyrimspecialedition/mods/77779) mod can increase the number of operations that can be executed per frame, per stack. This probably makes my entire post a lot less useful (but faster is still always better!).
# microlens

[![Build status](https://secure.travis-ci.org/aelve/microlens.svg)](http://travis-ci.org/aelve/microlens)

*A tiny part of the lens library which you can depend upon.*

## What you get

  * Essential lenses and traversals, as well as ones which are simply nice to have (like `each`, `at`, and `ix`).

  * More beginner-friendly documentation.

  * Compatibility with lens. If you want to define a `Lens` or a `Traversal` in your package, you can depend on this package without fear.

  * **No** dependencies. And the whole package builds in ~4s on my laptop. There are separate packages (only [microlens-ghc][] for now) which provide additional instances and let you use `each` and friends with various container types.

  * No awkward renamed functions or any of such nonsense. You can at any moment replace `Lens.Micro` with `Control.Lens` and get the full power of lens. There are also no unique to microlens functions which you would have to rewrite when switching to lens (even tho I was tempted to add some).

  * No Template Haskell dependency. There is a separate package for generating (lens-compatible) record lenses, which is called [microlens-th][]. It takes ~6s to build and it doesn't require any special dependencies either.

  * All `INLINE` pragmas sprinkled thru lens were preserved, as well as flags from the `.cabal` file. Performance shouldn't suffer; if it does, it's a bug.

[microlens]: http://hackage.haskell.org/package/microlens
[microlens-mtl]: http://hackage.haskell.org/package/microlens-mtl
[microlens-th]: http://hackage.haskell.org/package/microlens-th
[microlens-ghc]: http://hackage.haskell.org/package/microlens-ghc

## All packages in the family

  * [microlens][] – all basic functionality, plus `each`/`at`/`ix`
  * [microlens-mtl][] – `+=` and friends, `use`, `zoom`/`magnify`
  * [microlens-th][] – `makeLenses` and `makeFields`
  * [microlens-ghc][] – instances to make `each`/`at`/`ix` usable with arrays, `ByteString`, and containers
  * microlens-platform – haven't written it yet, but it'll have instances for `Text`, `Vector`, and `HashMap`

## Competitors

  * [basic-lens][] – the smallest library ever, containing only `Lens`, `view`, `set`, and `over` (and no lenses whatsoever). Uses only 1 extension – `RankNTypes` – and thus can be used with e.g. JHC and really old GHCs. Otherwise, you probably should still use a bigger lens library instead unless you're sure you won't actually need any standard lenses (e.g. `_1`) in your code, or if you're fine with copying definitions of those lenses by hand when you need them (i.e. replicating parts of some other lens library in your project).

  * [reasonable-lens][] – a bigger library which has `Lens`, some utilities (like `view`, `use`, `+=`), `makeLenses` even, but little else – no lenses (except for `_1` ... `_4`), no `Traversal`, no documentation. Overall it looks like something slapped together in a hurry by someone who simply needed to get rid of a lens dependency in one of nir projects.

  * [lens-simple][] – a single module reexporting parts of [lens-family][]. It's the most feature-complete library on this list, with both `Lens` and `Traversal` available, as well as a number of lenses, traversals, and utilities. However, it has some annoyances – no `each`, `_1` and `_2` work only on pairs, `ix` doesn't work on lists or arrays and is thus useless, `at` only works on `Map`, etc. I don't think these will ever be fixed, as they require defining some ad-hoc typeclasses, and the current absence of any such typeclasses in lens-family seems to suggest that the authors consider it a bad idea.

  * [data-lens-light][] – a library which uses a different formulation of lenses and is thus incompatible with lens (it uses different names, too). Doesn't actually provide any lenses.

[basic-lens]: http://hackage.haskell.org/package/basic-lens
[reasonable-lens]: http://hackage.haskell.org/package/reasonable-lens
[lens-simple]: http://hackage.haskell.org/package/lens-simple
[lens-family]: http://hackage.haskell.org/package/lens-family
[data-lens-light]: http://hackage.haskell.org/package/data-lens-light

For *most* usecases microlens is a better library than any of those libraries, but... well, like, c'mon, if you somehow accidentally happen to have used only the functions that reasonable-lens provides, and you are sure won't need any other functions, then reasonable-lens might win over microlens simply because with microlens you'd have to import 2–3 packages (depending on whether you need e.g. Template Haskell), and with reasonable-lens – only 1, which means that I can't claim with a straight face that microlens is *strictly* better.

## What's bad about this package

I hate it when people advertise things without also describing their disadvantages, so I'll list the ones I can think of here.

  * No prisms, no isomorphisms, no indexed traversals, and probably never will be.

  * Having to depend on 2–4 packages (if you're using a bigger chunk of lens's functionality) can be annoying. (But I hope that build times are worth it!)

  * This package doesn't actually let you write everything full lens-style (for instance, there's no `to`, no myriads of functions generalised for lenses by adding the `Of` suffix, etc). On the other hand, I guess some would actually consider it an advantage. Anyway, if you want to use lens as a *language* instead of as a tool, you can most likely afford depending on the full package.

  * If Edward et al. continue using incomprehensible dark magic to make lens faster, one day I *might* give up and stop backporting all their changes to microlens.

  * Scary orphan instances (in the [microlens-ghc][] package) are scary. (I don't think they're as scary as they seem, but some people disagree.)

  * There are `set` and `over` in the basic package, but no `view`. (There's `view` in [microlens-mtl][], but I can't have `view` in both packages at once, and having `view` in the basic package would add an mtl dependency.)

## Design decisions

microlens doesn't include anything lens doesn't include, even tho sometimes I'm very tempted to improve something in microlens just because I have control over it.

-----------------------------------------------------------------------------

All the `*Of` functions aren't included. If you don't know, those are `sumOf`, `lengthOf`, `setOf`, etc., and they are roughly equivalent to following:

~~~ haskell
sumOf    l s = sum          (s ^.. l)
lengthOf l s = length       (s ^.. l)
setOf    l s = Set.fromList (s ^.. l)
~~~

(Where `^..` takes something which extracts several targets, and returns a list of those targets. E.g. `(1, 2) ^.. both` is `[1, 2]`).

I guess the reason for including them all into `lens` (and there's an awful lot of them) is somewhere between

  * “they are faster than going thru intermediate lists”
  * “there are some rare cases when you can use a SomeSpecialisedMonoid but can't use `Endo [a]`”
  * “it's nice to be able to say `sumOf (each._1) [(1,"x"),(2,"y")]` instead of clumsy `sum . (^.. each._1) $ [(1,"x"),(2,"y")]`”

I suspect that the last reason is the most important one. The last reason is also the one I dislike most.

There are lots of functions which work on lists; lists are something like “the basic collection/stream type” in Haskell. GHC tries a lot to optimise code which produces and consumes lists; admittedly, it doesn't always succeed. `lens` seems to be trying to sidestep this whole list machinery.

  * With lists: one function traverses something and extracts a list of results, another function does something to those results.

  * With lenses: one function traverses something and takes another function as a parameter (to know what to do with results). Note that here `each._1` is the traversing function; it seems like `sumOf` takes it as a parameter, but in reality `sumOf` merely gives “summation” as the parameter to the traversing function.

The latter way is theoretically nicer, but *not* when you've got the rest of huge ecosystem using lists as the preferred way of information flow, otherwise you're bound to keep rewriting all functions and adding `Of` to them. `lens` is good for creating functions which extract data, and for creating functions which update structures (nested records, etc.), but it's not good enough to make the whole world want to switch to writing lens-compatible *consumers* of data.

So, don't write `Of` functions yet, it's not their time. Also, this suffix is uglyuglyugly.

-----------------------------------------------------------------------------

All those `<<&&=` operators aren't included. I don't know how useful they are – I never used them – and I don't like them because it feels like they do more harm than good, but they're not included merely because there's too many of them and I don't know how to pick which ones to include. Operators like `+=` or `.=` are available from [microlens-mtl][]. The only operators available from `Lens.Micro` are `%~`, `.~` and `&`.

-----------------------------------------------------------------------------

`Prism` and `Iso` aren't included. Long story short, I can't afford depending on [profunctors][]. This is explained better in the [Motivation](#motivation) section. It'd be cool if I could include them, but for now isos are nonexistent and those prisms/isos which are included are actually just traversals.

[profunctors]: http://hackage.haskell.org/package/profunctors

-----------------------------------------------------------------------------

Nothing indexed is included.

~~~
class ( Choice p
      , Corepresentable p
      , Comonad (Corep p)
      , Traversable (Corep p)
      , Strong p
      , Representable p
      , Monad (Rep p)
      , MonadFix (Rep p)
      , Distributive (Rep p)
      , ArrowLoop p
      , ArrowApply p
      , ArrowChoice p
      ) 
      => Conjoined p

class Conjoined p => Indexable i p
~~~

Perhaps it is for the best.

-----------------------------------------------------------------------------

Instances of `Ixed`, `Each`, `At`, etc are all split off into separate packages, which is understandable, because otherwise we'd have to have [vector][] as a dependency (the alternative is having orphan instances, which I'm not afraid of). However, even instances for libraries shipped with GHC (such as [array][]) are in [their own package][microlens-ghc]. There are 2 reasons for this:

* I *really* want to be able to say “this library has no dependencies”.
* All those instances actually take quite some time to build (for the same reason not all instances for tuples are included in the main package).

[array]: http://hackage.haskell.org/package/array

## Motivation

[lens][] is awesome. It's also huge, and requires lots of dependencies; not only [vector][] and [text][] (which you probably have anyway), but:

  * bifunctors
  * comonad
  * contravariant
  * distributive
  * exceptions
  * free
  * hashable
  * primitive
  * profunctors
  * reflection
  * semigroupoids
  * semigroups
  * split
  * tagged
  * transformers-compat
  * unordered-containers
  * void

[lens]: http://hackage.haskell.org/package/lens
[vector]: http://hackage.haskell.org/package/vector
[text]: http://hackage.haskell.org/package/text

Each of these packages is a nice one, solving a well-defined task, or providing a useful abstraction. Most of category-theoretical ones ([bifunctors][], etc.) only take seconds to install. However, having to depend on them all is a death by a thousand papercuts. It's alright for an application, but it's absolutely not alright for a library... until lens becomes a part of the Haskell Prelude, or subsumes it entirely. (If this was Edward Kmett's plan all along: sorry for spoiling it!)

[bifunctors]: http://hackage.haskell.org/package/bifunctors

What Edward wants is a huge, huge, huge Swiss knife. And ne is smart and strong enough to carry such a knife with nem and benefit from it. I imagine that there are lots of applications in the world which depend on `lens` and use more than 20% of its possibilities; even if not, creating such a Swiss knife is an entirely legitimate goal in itself.

What I want is a small knife to carry in my pocket. I actually have one in mine right now; it's just as sharp as other knives, but it's 5cm long and it contains scissors, a toothpick, tweezers, and a nail file. It's not enough for all purposes, or for even *most* purposes; but the cool thing is that I never have to consider taking it out of the pocket. Choosing to carry it isn't ever a tradeoff.

If you don't like knives (I don't either, I actually only ever use small scissors), consider an alternative analogy: lens is Emacs or Microsoft Word, microlens is nano (or Notepad) (possibly with Emacs's bindings, tho Emacs's default bindings actually suck, so it's not such a good analogy after all).

### But hasn't Edward said splitting `lens` into several packages is impossible?

Yes, but it's not what I'm trying to do. What Edward said (...or I think ne said), in a nutshell, is:

> You can't make a nice library with prisms and isos and everything without tying yourself to GHC.

Yes, but it doesn't matter because lots of people don't mind being tied to GHC. It's like saying that you shouldn't play World Of Warcraft because then you'll be tied to Windows. So what? People play World Of Warcraft anyway.

Moreover, I can't include `Prism` and `Iso` anyway without depending on `profunctors`, which depends on all other Edward's small packages, because they all provide instances for each other's types, because there's a useful type called `Tambara`, and it's important that there are instances like `Profunctor p => Strong (Tambara p)`. Like a big, happy family, Edward's packages all hold to each other.

I don't know why Edward can't just make a separate package called “classes”, where `Profunctor` and `Contravariant` would reside along with their *only* instances people ever use, namely `Profunctor (->)` and `Contravariant (Const a)`, and all nir other packages would just depend on classes and provide useful `Tambara` and `Cotambara` instances, and then microlens could also depend on classes and provide prisms and isos. There's probably a good reason. Or maybe Edward just doesn't want to maintain yet another package, ne's got too many of them already. Or maybe Edward has more interesting things to think about (imagine having to continue maintaining something as huge as lens when you're no longer really passionate about it or just want to be doing something else).

> Splitting `lens` would either lead to orphaned instances or loss of functionality.

Agreed. Splitting anything reasonably complex into several parts would lead to loss of functionality.

However, orphaned instances aren't the end of the world, and neither is loss of functionality. There are other important things to consider.

> Template Haskell code can't be moved out either because it doesn't make sense to tie yourself to GHC and implement so many useful things but leave automatic lens generation out.

Agreed.

It definitely doesn't make much sense to create lens and leave TH bits out, just as it doesn't make sense to produce a 2000$ tv set where you can't switch between sound tracks when watching a film from a USB stick. Fuck you, Philips, so much.

It makes sense, however, to leave TH bits out when they account for a 200% increase in build time, and they do in my case. So I moved them into a separate package.

> I used to think I wanted a simple lens library. Then I threw the doors open to everyone to tug on it in different directions and I found that the simple library I wanted wasn't so simple any more and I let it reshape itself to meet everyone's contradictory demands.

Well, yep, if you've made an amazing knife for youself you might as well make it nicer for other people to use it, and let them add features you personally don't need but don't mind having in your knife either, because what does it matter when the knife weighs 10kg already?

### What about lens-family?

[lens-family][] is mostly compatible with lens (unless I decide to nitpick and say that its `makeLensesBy` and `intAt` aren't present in lens at all), which has few dependencies, and which provides Template Haskell in a separate package as well.

[lens-family]: http://hackage.haskell.org/packages/lens-family

It looks like lens-family values cleanness and simplicity, which unfortunately means that it might've been hard for me (if possible at all) to convince its maintainer to make changes which would bring it closer to lens (`INLINE` pragmas, using unsafe `#.` operator, adding `each`, etc). I actually like cleanness and dislike excessive optimisation (especially of the kind that is used in lens) too, but making a library *I* would like wasn't my goal. The goal was to push people who aren't using a lens library towards using one.

Yep, in a way it's [NIH](https://en.wikipedia.org/wiki/Not_invented_here). However, I think that in this case in this case NIH is justified, if only because most reasons NIH is bad (time spent rewriting the library could've been spent improving another library, different libraries are incompatible with each other and so the community is fractured, etc) don't really apply here.

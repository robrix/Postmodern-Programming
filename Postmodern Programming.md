## Foreword: What’s In a Name?

The schedule calls this talk “Building a Better Mousetrap: Declarative Programming and You.” I’ve tried on a few other names for it in the meantime, and am now thinking of it as Postmodern Programming. I hope you’ll forgive the change, but I think this new title more accurately describes its subject.

I’ve gone through many iterations of this material, including a half-written GitHub client which was to be used to inspect its own source code and commits during the talk. I call it Navel-Gazing. We won’t be looking at it today (although it’s on my github page if you’re interested), but I think you’ll find its name appropriate to the talk, too.

As to _why_ “Postmodern Programming” or “Navel-Gazing” or any of this might fit, I will tell you a fact about me that has never been and will never be listed on my résumé: I can type at sustained speeds of around 140wpm. I spend all day in front of a computer and much of it hammering away on a keyboard with a sound like rain falling steadily on a corrugated iron roof, but I don’t list that fact on my résumé because it’s completely irrelevant: nobody pays a programmer for their typing skills.

The important parts of the job are done with our minds. It’s how we think about things, and what things we think up, and how we arrange these things. This is a talk about programming in the abstract; it’s a talk about _abstraction_ in the abstract. But we shouldn’t take that too seriously.

Really, this is a talk about language and languages, but while it features Objective-C but isn’t particularly _about_ Objective-C. Nothing I talk about today is going to make you a better _Cocoa_ programmer, but it might make you a better programmer—that is to say, _you_ will very likely make you a better programmer, and I will be very glad indeed if I have helped in some small way.

I tell a (small) lie: it is a _little_ bit about Cocoa, in that it is about declarative programming and how to do it, and if we’re using Cocoa, which is rather the point of this conference, then obviously it’s going to intersect with that somewhat. But on the whole these things apply rather more widely, and it can be a good thing to think of oneself as a _programmer_, rather than as a _Cocoa_ or _Cocoa Touch programmer_.

Of course, being that this is a talk about declarative programming, it might be discuss what that even means. Wikipedia (the authoritative repository of all human knowledge) gives three definitions:

1. A program that describes _what_ computation should be performed and not _how_ to compute it
2. Any programming language that lacks side effects (or more specifically, is referentially transparent)
3. A language with a clear correspondence to mathematical logic

Well, that certainly clarifies things. And as an aside: you have to love any concept whose wikipedia page includes the heading “Subparadigms.”

For the moment let’s think of it as “not imperative programming,” where “imperative programming” is you, the Grand Imperator, telling the program what to do. Declarative programming would instead make you the Grand Declarator: you instead tell the program what it _is_. And, with that, the program no doubt walks away enlightened. (Even if we don’t.)

## Prologue: Everything I Know Is Wrong

My name is Rob Rix. I’ve been a programmer for as long as I can remember, I’ve got twelve years of professional experience behind me, and I’ve spent the last eighteen months doing consulting and product development with Black Pixel. Before that, I spent eighteen months implementing sync, and so I know a thing or two about making mistakes.

All of which is to say: I know what I’m talking about, or at least, I think I do. But you shouldn’t take this as fact. Nothing in this talk is a stroke of genius or a flash of inspiration. Nothing in this talk is particularly new. If it’s new to _you_, there’s nothing wrong with that, either—it just means it’s your lucky day, and for that matter, mine too. So please don’t take my word on anything: if there’s anything in here that strikes you as interesting or dull or likely or implausible, try it out for yourself. See where it takes you. More than anything else, I would love to see you make things.

In the meantime, some things in this talk are bad ideas. Some of this is bad advice! Certainly if Monday morning you resolve to do nothing but what I talk about today then you are going to be very sorry when you get fired for it.

Nevertheless, that which I think I know, follows: this is a talk about making mistakes.

## Part the First: Don’t Repeat Don’t Repeating Yourself

Every day, we write code we’ve written before. We all do this, all the time. This isn’t practice, either; it’s boilerplate. “Boilerplate” ought to be a swear word.

Apple has done a lot to diminish this which we often take advantage of because we’re programmers: we hate doing things twice when we could have done them once. So we use properties (no matter our feelings on dot notation), since that means we can write less boilerplate accessor code. And we’re a little happier for doing so. This is good for us!

Because any time we write the same code twice, we are wasting ourselves. By definition, it could have been abstracted. We could have written it once. Perhaps the language makes it unclear, or difficult. Often we’re writing what is only _mostly_ the same code, but there are tangled-up bits of differences swimming about in the middle of it. Perhaps it’s so straightforward that pulling the code we’re repeating out into another method and using that instead is going to result in a net increase in the amount of code we write. Why make things more complex than you have to?

There is no silver bullet. A professional must be prepared for the fact that she will have to make compromises, to do things the “wrong” way, in the course of her duties, and that every action she takes is a trade-off. But neither does she forget that she has repeated herself, or what the trade-off she made was, or why she made it.

There is, therefore, a time to write the code twice. But if you always do that, you’re never going to get anything done. We cannot rebuild the universe every time we want a piece of pie!

And unfortunately, even if we were to simply avoid repeating ourselves, it wouldn’t be enough.

## Part the Second: Abstraction, Abstracted

It’s famously said that there are only two hard problems in computer science: cache invalidation, naming things, and off-by-one errors. This is a talk about the median value, about naming things.

Until recently, I thought that joke was saying that picking the name is the hard part: “Is this a ManagerFactoryObjectController or a ControllerFactoryManagerObject?”

That’s _a_ joke, but perhaps not _the_ joke. In truth, it isn’t easy selecting a name, but that’s not the hard part by half. The hard part is selecting something which is deserving of a name.

How many of us have implemented view controller containment in a UIViewController subclass? Perhaps this is a view controller which decorates some child view controller with some common widget or branding that we want to use in various parts of our app?

You have to keep track of a lot of things: calling the correct containment methods on the old one in the right order, likely assigning it to a property, removing the old subview, calling the correct containment methods on the new one in the right order, adding the new subview, ensuring it has the right constraints or frame…

If you do all of this ad hoc any time you need to set the child view controller, it quickly becomes an intractable mess strewn across the `-viewDidLoad`, `-viewWillAppear:`, `-viewDidAppear:` and other methods of every view controller we might want to apply this treatment to. Instead, we abstract that into a method:

    -(void)setChild:(UIViewController *)child {
      [_child willMoveToParentViewController:nil];
      [_child.view removeFromSuperview];
    	[_child removeFromParentViewController];
    	_child = child;
    	[self addChildViewController:child];
    	[self.view addSubview:child];
    	child.frame = self.view.bounds;
    	[child didMoveToParentViewController:self];
    }

Is this an abstraction? Sure. It abstracts the act of setting the child. Does what it says on the tin!

Even if we’re only writing it once, that’s a lot of code to have to write in order to do a single thing. Why didn’t Apple make this more convenient? Why do you have to call willMove/remove and add/didMove paired like that? Why do you have to add and remove the subview yourself?

What if you want to animate the transition? They can’t just add and remove the subview on your behalf. They also can’t necessarily just call remove or didMove on your behalf, since those should be called when the animation is complete. You can use `-transitionFromViewController:…` to handle much of this, but it takes two view controllers which are already children, so some of this is still going to have to be handled externally.

This is unwieldy, but it seems that there’s no good way around it being unwieldy. That’s ok; it’s not like it’s costing us anything, unless we need to write something _slightly_ different, in which case we can copy and paste this and tweak it to do what it needs to do in the new context _and get over it_. Move on.

We know about this sort of thing. Sure, we might repeat ourselves _a little_, but we have more important things to worry about. And code is never going to be perfect, anyway; that’s idealistic nonsense bordering on sociopathy.

But…

How many of us have written a `-firstObject` method in a category on `NSArray`? What did it look like?

    -(id)firstObject {
    	return self.count? self[0] : nil;
    }

This is abstraction too, isn’t it? We’ve named something.

Perhaps we wrote this method after getting fed up of writing this out by hand every time we wanted to reason about the first object in an array. We might have been doing this directly, explicitly using the logic long-form:

    -(NSString *)firstName {
    	NSString *nameComponents = [self.fullName componentsSeparatedByString:@" "];
    	return nameComponents.count? nameComponents[0] : nil;
    }

We might also have been doing it indirectly:

    -(void)cycleSubviews {
    	if (self.view.subviews.count) {
    		[self.view sendSubviewToBack:self.view.subviews[0]];
    		[self.view setNeedsLayout];
    	}
    }

In both cases we’re employing the concept of `-firstObject` even if we haven’t yet named it. Is `-firstObject` an abstraction? Absolutely. And it’s satisfying: it does exactly what you want. You’re not going to have to change it; there’s nothing to tweak. It is as close to perfect as anything we write can be.

What makes this different from `-setChild:`? Is it the parameter? Is it just the length? Is there a shorter example that shows the same contrast with `-firstObject`? Maybe you have a flower shop app and are writing a view controller to enter the destination details:

    -(void)setUpNavigationItem {
    	self.navigationItem.title = @"Recipient";
    	self.navigationItem.prompt = @"Where would you like the flowers sent?";
    }

This isn’t unwieldy at all: there’s only two lines of code in this method. It’s trivial. In fact, why would you even write this method? Why would anyone ever write this method, except to make `-viewDidLoad` maybe a couple of lines shorter? It’s ridiculous: we are going to call this all of once, and you’ve actually increased the program’s length by a net three lines of code: one for the declaration of the method, one for the closing brace, and one to call it in `-viewDidLoad`.

Furthermore, if you subclass it, and you don’t know that this method exists, you’re going to have to make sure you’re calling `[super viewDidLoad]` _before_ you set the subclass’ title and prompt or else they’ll be reset on your behalf, infuriatingly.

And you’re not repeating yourself anyway, right? It’s not like you’d ever use these values anywhere else; you’d use different values. Not repetition at all.

Of course, while the content varies, the actions do not. You’re duplicating the assignment of these two related properties. It _is_ repetition; but it appears to be almost unavoidable repetition. Oh, you could parameterize the method with two strings, but what has that got you, really? At best a convenience that you will, again, use at most once or twice and which saves you, ultimately, nothing.

The commonality this shares with `-setChild:` is also the contrast it has with `-firstObject`: `-setChild:` and `-setUpNavigationItem` both tell the computer _how_ to do something. `-firstObject`, on the other hand, defines _what_ the first object _is_.

Which is to say: of course `-firstObject` _also_ tells the computer _how_ to accomplish its task: send the count message to self, and if its return value was nonzero then return the result of sending `-objectAtIndexedSubscript:` to self with a parameter of zero; otherwise, return nil. But this recipe for _how_ the computer should perform it coincides nicely with its declaration of _what it is_: “the first object of arrays is, when nonempty the one at object 0, and otherwise nil.”

With `-setUpNavigationItem`, defining the _what_ is almost a contradiction in terms: it’s not a noun, not a concept. It has no value (by which I mean that it declares its return type to be `void`, i.e. no type, no value). It was _abstracted_, but in a sense it is not _an abstraction_, but merely an _extraction_ of specific instructions.

Declarative programming is nebulous, ill-defined, _abstract_. For the moment let’s characterize it as the `-firstObject` approach, whereas the `setUpNavigationItem` and `-setChild:` approaches are _imperative_. Or, more succinctly: the difference between what and how.

But _what_ `-firstObject` is boils down to semantics, surely. We don’t have access to the syntax tree at runtime; we don’t have Lisp-style macros that can operate on the messages at or before compile-time, either. Nothing about these messages is available at any point to us, so _all that they can do is tell the computer how it should behave_! Right?

In fact, yes. The specific messages or structure of messages _are_ (basically) irrelevant. It really _is_ semantics. And of course “semantics” means “meaning.” So where an abstraction is a concept named and made available to us for use across varying specific details (in `-firstObject`’s case, which array we are getting the first object of), the fact that it is _declarative_ tells us that its _name_ has _meaning_. In this sense, too, `-setChild:` and `-setUpNavigationItem` are not abstractions, despite having been abstracted: they have no meaning, only instructions. They are not nouns. _They are not concepts._

They do not exist. They are unreal, unthings.


While `-firstObject` is an abstraction, it can be abstracted further. For example, we could generalize it to return the object at any in-bounds ordinal or nil:

    -(id)nthObject:(NSUInteger)n {
    	return (self.count > n)? self[n] : nil;
    }

Now we can define `-firstObject` in terms of `-nthObject:`:

    -(id)firstObject {
    	return [self nthObject:0];
    }

Clearly, `-nthObject:` is more abstract than `-firstObject`: it includes one fewer concrete concept, but can still implement the same behaviour. Can we abstract it further?

One way to identify a potential abstraction is simply to consider some decision that the code in question is making and simply take the result of that decision as a parameter. `-nthObject:` is making several decisions: it decides whether to return a member or nil; it decides to return a specific value in the in-bounds case; and it decides to return a specific value (nil) in the out-of-bounds case. Let’s consider this last one. Extracting it into a parameter, we get:

    -(id)nthObject:(NSUInteger)n default:(id)marker {
    	return (self.count > n)? self[n] : marker;
    }

    -(id)nthObject:(NSUInteger)n {
    	return [self nthObject:n default:nil];
    }

This is, again, clearly more abstract: we generalized from the return of nil to the return of some marker value.

In each of these instances, a decision was abstracted by replacing a constant with a parameter (and adjusting the logic accordingly where necessary). Each resulting method is more abstract than the previous. Are there other ways of abstracting?

I’m not going to answer this, or at least not right now. But take it as an experiment: Can you abstract a concept without replacing a constant with a variable (parameter, instance variable, or otherwise)? If so, what does that look like? And by all means if you _do_ try this, let me know where it takes you. I know what my answer is, but it might not be the same as yours!

## Part the Third: Complexity Complex

It might seem like a non sequitur to jump from talking about abstraction to talking about complexity, but I promise the segue makes sense.

Has anyone here been programming for the Mac since before 2011 or for iOS since before 2012? Or has anyone written apps without autolayout and then tried writing one with autolayout?

How often did you swear?

Maybe you said to yourself: Why does autolayout have to make it so hard to just set the view’s frame? This was so easy with autoresizing masks! Why does autolayout have to make everything so complicated?

To be clear: this feeling is totally justified. It _was_ easy with autoresizing masks. It really _is_ less easy with autolayout. But what does that mean, _precisely_?

There’s an interesting contrast to be drawn between _simplicity_ and _ease_. I am hardly the first to make this distinction, but I’d like to talk about it in terms of counting.

How many concrete concepts does `-firstObject` contain? When I say “concrete” I mean more or less “invariant”—concepts which are part of the contract of the method. Remember, this is, by definition, semantics, so we shouldn’t be afraid of this question just because it’s fuzzy. Let’s count them:

    -(id)firstObject {
    	return self.count? self[**0**] : **nil**;
    }

(Notice that I’m using the original definition, before we abstracted it.)

I count (at least) two:

1. The index of the object that we want to extract.

2. The object that we want to return if the array is empty.

(As an aside, we could also consider the condition as being a concrete, or at least _relatively_ concrete concept. For the purposes of this exercise, however, it is not hugely informative to do so.)

How about `-nthObject:`?

    -(id)nthObject:(NSUInteger)n {
    	return (self.count > n)? self[n] : nil;
    }

Using the same metrics, it looks like only one:

1. The object that we want to return if the desired index is out of bounds.

And `-nthObject:default:`?

    -(id)nthObject:(NSUInteger)n default:(id)marker {
    	return (self.count > n)? self[n] : marker;
    }

By my count, 0. So it would seem that the more abstract the code, the fewer concrete concepts are involved. And indeed it makes some intuitive sense that abstraction and concreteness would be inversely linked.

Now let’s look at the definitions of `-firstObject` and `-nthObject:` in terms of their more concrete counterparts:

    -(id)firstObject {
    	return [self nthObject:0];
    }

    -(id)nthObject:(NSUInteger)n {
    	return [self nthObject:n default:nil];
    }

Looked at through the same lens, each of these definitions has captured only one concrete concept! And _this_ is more than simply semantics: did you notice that when we defined `-nthObject:default:` and then redefined `-nthObject:` in terms of it, we didn’t need to change `-firstObject` at all? It stayed exactly the same.

This is getting near the heart of the difference between simplicity and ease. When you want the first object in an array (defaulting to nil if empty), is it easier to write the code out by hand or to use `-nthObject:` and pass 0? Obviously, `-nthObject:` is easier. Is it easier to use `-nthObject:`, again passing 0, or to use `-firstObject`? Again, obviously `-firstObject` is easier. But is it _simpler_?

When we looked at the original implementation of `-firstObject`, we counted two concrete concepts. When we looked at the redefinition, we counted only one. Where did the other one go? And _why_ didn’t we have to change `-firstObject` when we changed `-nthObject:`?

Remember that the two concrete concepts in the original `-firstObject` were `0` and `nil`. The redefinition only included `0`, however, because `nil` was moved inside `-nthObject:`! `-firstObject` is in fact using _exactly_ as many concrete concepts as it was before, it’s just hidden the second one behind the call to `nthObject:`—behind the veil of abstraction. Where it belongs!

This, then, is the very core of simplicity: a simpler program has fewer concepts, a complex one has more. `-firstObject` is more complex than `-nthObject:` precisely because it is less general; the increase in abstraction mirrors an increase in simplicity.

_Ease_ is in fact _convenience_. What makes it so hard to just set the view’s frame with autolayout? It’s not complexity; it’s inconvenience.

Let me qualify that a little further: If `-firstObject` is exactly as complex before and after the existence of `-nthObject:`, then isn’t it more complex to use autolayout constraints to define the frame of the view on the screen than to assign that frame?

If that’s all you ever want to do, then yes. But what’s involved in setting the frame when you want to ensure that it behaves according to a system of rules, e.g. that it is never larger than its parent, that it is never smaller than a certain size, that it is the same width as some other thing, that it has a certain aspect ratio?

The complexity of implementing all of this quickly exceeds that of employing autolayout. The key is that what I will call the _total_ complexity of `-firstObject`—that is, the number of concrete concepts used by either it or the abstractions it uses—remained the same, the _local_ complexity—the number of direct references to concrete concepts—_was_ reduced.

The _apparent_ simplicity of `-setFrame:`—really its ease—doesn’t scale. Without an abstraction of the rules for how the frame of that view should behave, you have to implement them all by hand. You _also_ have to ensure that they’re all enforced _whenever their dependencies change_. Whereas autolayout encompasses not only the rules embodied in the constraints, but also the necessary logic to keep them consistent and current—`-layout[Subviews]`, `-updateConstraints`, and so on.

This sheds new light on `-setUpNavigationItem`. How many concrete concepts does it involve?

    -(void)setUpNavigationItem {
    	self.navigationItem.title = @"Recipient";
    	self.navigationItem.prompt = @"Where would you like the flowers sent?";
    }

Multiple choice:

1. 2.
2. 3.
3. Wait, are we talking about total or local complexity here?
4. At least 4.
5. You’re clearly just going to tell us anyway so get on with it.

Good answer.

The values assigned to `self.navigationItem.title` and `self.navigationItem.prompt` are clearly both concrete concepts. But, crucially, so are `self.navigationItem`’s `title` and `prompt`!

Wait a minute. We didn’t count anything involving `self` in `-firstObject` or `-nthObject:`; what gives?

Masked by the setters is another concrete concept: the change to the navigation item inherent in assigning a variable to one of its properties. This may seem like a bit of a leap from counting constants, but remember that one of the difficulties with the usefulness of `-setUpNavigationItem` was that if you subclassed the class implementing it, you would have to think about the order in which methods _which you might not even know about_ would be called in, in order to ensure that you did not encounter bugs. It’s not explicit in the syntax, but changes to state are real concepts which you have to think about in order to program in Objective-C.

This is because Objective-C, like many languages, is _imperative_ in nature: you, the Grand Imperator, tell the program _how to behave_. In a declarative programming language, for example SQL, Prolog, or Haskell, you are instead the Grand Declarator, telling each abstraction _what it is_. Again: how vs. what.

In order to describe changes in state in a declarative programming language, you therefore have to make them explicit: tell the program what it is in terms of these changes. Any part of the program using them is therefore referring to at least one concrete concept, making it more complex than an equivalent part of the program not also using state changes.

In an imperative language, state changes are the _modus operandi_, or even the _mode de vie_. You live and breathe them, and thus all of your code is made more complex.

At the same time, abstracting-without-abstractions denies you the ability to deal with these extra concepts behind the veil of local complexity; that is, any code _using_ an imperative abstraction necessarily incurs the complexity of any changes it performs in a total sense (as with any other abstraction), but _also_ incurs it locally—because any other changes to the same state need to be carefully sequenced. The details leak from callee to caller, again and again, and can never be contained.

## Part the Somethingth: Flow Control

While Objective-C is an imperative language, we can view this as meaning that it can be programmed in an imperative style. Fortunately for us, we have seen that it can also be programmed in a declarative style.




## Abstraction and Composition

The Structure and Interpretation of Computer Programs, in my opinion the most important work on our field yet produced, tells us early on that _the most important features_ of every programming language are its facilities for abstraction and composition, and that we should therefore judge every language by these features.

## Abstraction in the Abstract

There is a proverb about a man who points at the moon, and says “There is the moon.” You do not think he is talking about his finger, but about the thing he points at. Extending his finger is shorthand; it is a symbol, a stand-in, to avoid his having to take you to the moon and smack you over the head with it before you get the point. The moon is a concrete thing; his finger, and indeed the word “moon,” are abstractions. We can be even more abstract: other worlds have moons, too. _The_ moon is really just _a_ moon. This is my moon; there are many like it, but this one is mine.

To abstract is to identify an idea, a concept, as a unique thing which can be reasoned and acted upon in isolation. It is to give it a name. It is to define that name with that concept. To add a definition to the dictionary.

It’s hard to think about something which you do not have a word for. Sometimes we don’t have good enough words for things that are important to us: the feeling of growing up and taking responsibility for your life. The bittersweet experience of a trusted colleague and friend leaving to work on something important to them: that simultaneous grief at your loss and joy at their gain. Or how the joy outweighs the grief.

We know these things, but we don’t talk about them all that often. Perhaps we would do so more if we did have words for them. Perhaps it’s for the best that we don’t; but then, Hawking coining “spaghettification” for the experience of being sucked past the event horizon of a black hole has not had much effect on my conversations, for better or for worse.

Therefore, we make words for things we need to think about. We name not just unique objects, or categories of objects, but also concepts. Sometimes these are new words, like zeitgeist or monad. These are _never_ new ideas, however: we cannot name an idea we have not had. Even if we are the first and only to think of an idea, the selection of the word must follow, never lead.

No word is an island, of course; we use them together in sentences according to the grammar of our language. The concept of spaghettification in the abstract is much less useful if we can’t speak of the inevitable and redundant spaghettification of a delicious pasta dinner ejected into the galactic core.

Naming things and phrasing are fairly innate to human beings. Of course, they’re not always easy.

Some people are particularly good at the selection and combination of words. Writers and poets, lawyers and politicians, comedians and liars. Teachers, hopefully. And someday maybe presenters at conferences will be, too.

But naming things? Abstraction, is that a skill? Absolutely it is. If you hurt your hand because you touch the flame, you do not learn only not to touch _this particular_ flame, but all flames; and from there, to _things that are hot_, and you might further abstract from this experience the important lesson to be more careful during chemistry class. Each abstraction is a leap from symbol to definition. Making these leaps is a skill. It can be _practiced_. You can make these leaps more readily, reflexively, and see where it takes you.

Another part of this skill is developing your intuition for which leaps _should_ be made. You learned that when you got burnt, your hand became a bit more red where you were burnt, as your body works to heal itself. You probably don’t make the leap to assuming that everything which is burnt turns red and heals itself. Certainly this does not seem to be true of the match!

## Composition









The risk is falling behind. Apple is not afraid to strand you without any customers. The market is not afraid to ignore you if you don’t work with the latest & greatest.

At the same time you have an incredible amount to gain: abstraction is one of the two core skills of programming on any platform. (The other one is composition.) You should not try to be a better Mac and iOS programmer! You should try to be a better programmer. Focusing on the fundamental skills will make you more effective at programming, period. It will change how you think about problem-solving, and will help you tackle problems that you felt were beyond your reach.

It will also help you write more reliable programs, more maintainable programs, faster programs. None of these is a determinant of success by itself, but if an unreliable, unmaintainable, dog slow mess of a program is successful now, it very soon will not be—because change is inevitable.

Necessity demands pragmatism. Pragmatism demands flexibility. Flexibility demands simplicity. Simplicity _is_ abstraction.

- why?
	- don’t want to/can’t take the time that’s going to be required to learn the thing
		- you have to ship
		- you have to support old releases, not the latest & greatest beta which could very well brick your phone
		- by the time you can use the new API, WWDC is a distant memory (if you even got to attend!) and you’re forced to rely on the docs, StackOverflow, the dev forums, twitter, mailing lists, colleagues, talks at conferences
		- this is a treadmill: we have every reason to believe that Apple will continue releasing new API every summer until the sun explodes
		- is this really going to put food on my plate?
	- concerns about performance (abstraction generally implies indirection, i.e. at least one extra step)
		- optimization is hard
		- every little bit of friction could hurt your framerate
		- you put lots of work into your existing stuff to make it fast enough
	- concerns about “magic”—opaque
		- can’t just set a breakpoint
			- 
		- can’t just step through it in the debugger
		- some things that were trivial are much less obvious now
	- often something that was easy the old way is comparatively more complex now
	- what we know already is good enough to have got us this far

- but: you _need_ to be better at making and using abstractions
	- your customers like shiny, pretty things


## Q?&A!
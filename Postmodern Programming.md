## Foreword: What’s In a Name?

The schedule calls this talk “Building a Better Mousetrap: Declarative Programming and You.” Mark Dalrymple calls it the same, but with “Moose” instead of “Mouse.” I’ve tried on a few other names for it in the meantime, and am now thinking of it as Postmodern Programming. I hope you’ll forgive the change, but I think this new title more accurately describes its subject.

I’ve gone through many iterations of this material, including a half-written GitHub client which was to be used to inspect its own source code and commits during the talk. I call it Navel-Gazing. We won’t be looking at it today (although it’s on my github page if you’re interested), but I think you’ll find its name appropriate to the talk, too.

As to _why_ “Postmodern Programming” or “Navel-Gazing” or any of this might fit, I will tell you a fact about me that has never been and will never be listed on my résumé: I can type at sustained speeds of around 140wpm. I spend all day in front of a computer and much of it hammering away on a keyboard with a sound like rain falling steadily on a corrugated iron roof, but I don’t list that fact on my résumé because it’s completely irrelevant: nobody pays a programmer for their typing skills.

The important parts of the job are done with our minds. It’s how we think about things, and what things we think up, and how we arrange these things. This is a talk about programming in the abstract; it’s a talk about _abstraction_ in the abstract. But we shouldn’t take that too seriously.

Really, this is a talk about language and languages, but while it features Objective-C but isn’t particularly _about_ Objective-C. Nothing I talk about today is going to make you a better _Cocoa_ programmer, but it might make you a better programmer—that is to say, _you_ will very likely make you a better programmer, and I will be very glad indeed if I have helped in some small way.

I tell a (small) lie: it is a _little_ bit about Cocoa, in that it is about declarative programming and how to do it, and if we’re using Cocoa, which is rather the point of this conference, then obviously it’s going to intersect with that somewhat. But on the whole these things apply rather more widely, and it can be a good thing to think of oneself as a _programmer_, rather than as a _Cocoa_ or _Cocoa Touch programmer_.

Of course, being that this is a talk about declarative programming, it might be useful to discuss what that even means. Wikipedia (the authoritative repository of all human knowledge) gives us three definitions:

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

Every day, we write code we’ve written before. We all do this, all the time. This isn’t us practicing, either; it’s boilerplate. “Boilerplate” ought to be a swear word.

Apple has done a lot to reduce this which we often take advantage of because we’re programmers and therefore we hate doing things twice when we could have done them once. So we use properties (no matter what our feelings on dot notation are), since that means we can write less boilerplate accessor code. And we’re a little happier for doing so. This is good for us!

Because any time we write the same code twice, we are wasting ourselves. By definition, it could have been abstracted. We could have written it once. Perhaps the language makes it unclear, or difficult. Often we’re writing what is only _mostly_ the same code, but there are tangled-up bits of differences swimming about in the middle of it. Perhaps it’s so straightforward that pulling the code we’re repeating out into another method and using that instead is going to result in a net increase in the amount of code we write. Why make things more complex than you have to?

There is no silver bullet. A professional must be prepared for the fact that she will have to make compromises, to do things the “wrong” way, in the course of her duties, and that every action she takes is a trade-off. But neither does she forget that she has repeated herself, or what the trade-off she made was, or why she made it.

There is, therefore, a time to write the code twice. But if you always do that, you’re never going to get anything done. We cannot rebuild the universe every time we want a piece of pie!

And unfortunately, even if we were to simply avoid repeating ourselves, it wouldn’t be enough.

## Part the Second: Abstraction, Abstracted

It’s famously said that there are only two hard problems in computer science: cache invalidation, naming things, and off-by-one errors. (This is a talk about the median value: about naming things.)

Until recently, I thought that joke was saying that picking the name is the hard part: “Is this a ManagerFactoryObjectController or a ControllerFactoryManagerObject?”

That’s _a_ joke, but perhaps not _the_ joke. In truth, it isn’t easy selecting a name, but that’s not the hard part by half. The hard part is selecting something which is deserving of a name: abstracting.

How many of us have implemented view controller containment in a UIViewController subclass? Perhaps this is a view controller which decorates some child view controller with some common widget or branding that we want to use in various parts of our app?

You have to keep track of a lot of things: calling the correct containment methods on the old one in the right order, likely assigning it to a property, removing the old subview, calling the correct containment methods on the new one in the right order, adding the new subview, ensuring it has the right constraints or frame…

If you do all of this ad hoc any time you need to set the child view controller, it quickly becomes an intractable mess strewn across the `-viewDidLoad`, `-viewWillAppear:`, `-viewDidAppear:` and other methods of every view controller we might want to apply this treatment to. Instead, we abstract that into a method:

```objc
-(void)setChild:(UIViewController *)child {
    [_child willMoveToParentViewController:nil];
    [_child.view removeFromSuperview];
    [_child removeFromParentViewController];
    _child = child;
    [self addChildViewController:child];
    [self.view addSubview:child.view];
    child.view.frame = self.view.bounds;
    [child didMoveToParentViewController:self];
}
```

Is this an abstraction? Sure. It abstracts the act of setting the child. Does what it says on the tin!

Even if we’re only writing it once, that’s a lot of code to have to write in order to do a single thing. Why didn’t Apple make this more convenient? Why do you have to call willMove/remove and add/didMove paired like that? Why do you have to add and remove the subview yourself?

What if you want to animate the transition? They can’t just add and remove the subview on your behalf. They also can’t necessarily just call remove or didMove on your behalf, since those should be called when the animation is complete. You can use `-transitionFromViewController:…` to handle much of this, but it takes two view controllers which are already children, so some of this is still going to have to be handled externally.

This is unwieldy, but it seems that there’s no good way around it being unwieldy. That’s ok; it’s not like it’s costing us anything, unless we need to write something _slightly_ different, in which case we can copy and paste this and tweak it to do what it needs to do in the new context _and get over it_. Move on.

We know about this sort of thing. Sure, we might repeat ourselves _a little_, but we have more important things to worry about. And code is never going to be perfect, anyway; that’s idealistic nonsense bordering on sociopathy.

But…

Before iOS 7/OS X 10.9, how many of wrote a `-firstObject` method in a category on `NSArray`? What did it look like?

```objc
-(id)firstObject {
	return self.count? self[0] : nil;
}
```

This is abstraction too, isn’t it? We’ve named something.

Perhaps we wrote this method after getting fed up of writing this out by hand every time we wanted to reason about the first object in an array. We might have been doing this directly, explicitly using the logic long-form:

```objc
-(NSString *)firstName {
	NSArray *nameComponents = [self.fullName componentsSeparatedByString:@" "];
	return nameComponents.count? nameComponents[0] : nil;
}
```

We might also have been doing it indirectly:

```objc
-(void)cycleSubviews {
	if (self.view.subviews.count) {
		[self.view sendSubviewToBack:self.view.subviews[0]];
		[self.view setNeedsLayout];
	}
}
```

In both cases we’re employing the concept of `-firstObject` even if we haven’t yet named it. Is `-firstObject` an abstraction? Absolutely. And it’s satisfying: it does exactly what you want. You’re not going to have to change it; there’s nothing to tweak. It is as close to perfect as anything we write can be.

What makes this different from `-setChild:`? Is it the parameter? Is it just the length? Is there a shorter example that shows the same contrast with `-firstObject`? Maybe you have a flower shop app and are writing a view controller to enter the destination details:

```objc
-(void)setUpNavigationItem {
	self.navigationItem.title = @"Recipient";
	self.navigationItem.prompt = @"Where would you like the flowers sent?";
}
```

This isn’t unwieldy at all: there’s only two lines of code in this method. It’s trivial. In fact, why would you even write this method? Why would anyone ever write this method, except to make `-viewDidLoad` maybe a couple of lines shorter? It’s ridiculous: we are going to call this all of once, and you’ve actually increased the program’s length by a net three lines of code: one for the declaration of the method, one for the closing brace, and one to call it in `-viewDidLoad`.

Furthermore, if you subclass it, and you don’t know that this method exists, you’re going to have to make sure you’re calling `[super viewDidLoad]` _before_ you set the subclass’ title and prompt or else they’ll be reset on your behalf, infuriatingly.

And you’re not repeating yourself anyway, right? It’s not like you’d ever use these values anywhere else; you’d use different values. Not repetition at all.

Of course, while the content varies, the actions do not. You’re duplicating the assignment of these two related properties. It _is_ repetition; but it appears to be almost unavoidable repetition. Oh, you could parameterize the method with two strings, but what has that got you, really? At best a convenience that you will, again, use at most once or twice and which saves you, ultimately, nothing. As with `-setChild:`, we’re left with discomfort: users have to know _how it works_ to be able to use it or the class containing it safely.

The commonality this shares with `-setChild:` is also the contrast it has with `-firstObject`: `-setChild:` and `-setUpNavigationItem` both tell the computer _how_ to do something. `-firstObject`, on the other hand, defines _what_ the first object _is_.

Which is to say: of course `-firstObject` _also_ tells the computer _how_ to accomplish its task: send the count message to self, and if its return value was nonzero then return the result of sending `-objectAtIndexedSubscript:` to self with a parameter of zero; otherwise, return nil. But this recipe for _how_ the computer should perform it coincides nicely with its declaration of _what it is_: “the first object of arrays is, when nonempty the one at object 0, and otherwise nil.”

With `-setUpNavigationItem`, defining the _what_ is a contradiction in terms: it’s not a noun, not a concept. It has no value (by which I mean that it declares its return type to be `void`, i.e. no type, no value). It was _abstracted_, but in a sense it is not _an abstraction_, but merely an _extraction_ of specific instructions.

Declarative programming is nebulous, ill-defined, _abstract_. For the moment let’s characterize it as the `-firstObject` approach, whereas the `setUpNavigationItem` and `-setChild:` approaches are _imperative_. Or, more succinctly: the difference between what and how.

But _what_ `-firstObject` is boils down to semantics, surely. We don’t have access to the syntax tree at runtime; we don’t have Lisp-style macros that can operate on the messages at or before compile-time, either. Nothing about these messages is available at any point to us, so _all that they can do is tell the computer how it should behave_! Right?

In fact, yes. The specific messages or structure of messages _are_ (basically) irrelevant. It really _is_ semantics. And of course “semantics” means “meaning.” So where an abstraction is a concept named and made available to us for use across varying specific details (in `-firstObject`’s case, which array we are getting the first object of), the fact that it is _declarative_ tells us that its _name_ has _meaning_. In this sense, too, `-setChild:` and `-setUpNavigationItem` are not abstractions, despite having been abstracted: they have no meaning, only instructions. They are not nouns. _They are not concepts._

They do not exist. They are unreal, unthings.

This is what happens to you when you stare too long into the `void`.


While `-firstObject` is an abstraction, it can be abstracted further. For example, we could generalize it to return the object at any in-bounds ordinal or nil:

```objc
-(id)nthObject:(NSUInteger)n {
	return (self.count > n)? self[n] : nil;
}
```

Now we can define `-firstObject` in terms of `-nthObject:`:

```objc
-(id)firstObject {
   	return [self nthObject:0];
}
```

Clearly, `-nthObject:` is more abstract than `-firstObject`: it includes one fewer concrete concept, but can still implement the same behaviour. Can we abstract it further?

One way to identify a potential abstraction is simply to consider some decision that the code in question is making and simply take the result of that decision as a parameter. `-nthObject:` is making several decisions: it decides whether to return a member or nil; it decides to return a specific value in the in-bounds case; and it decides to return a specific value (nil) in the out-of-bounds case. Let’s consider this last one. Extracting it into a parameter, we get:

```objc
-(id)nthObject:(NSUInteger)n default:(id)marker {
	return (self.count > n)? self[n] : marker;
}
-(id)nthObject:(NSUInteger)n {
	return [self nthObject:n default:nil];
}
```

This is, again, clearly more abstract: we generalized from the return of nil to the return of some marker value.

In each of these instances, a decision was abstracted by replacing a constant with a parameter (and adjusting the logic accordingly where necessary). Each resulting method is more abstract than the previous. Are there other ways of abstracting?

I’m not going to answer this, or at least not right now. But take it as an experiment: Can you abstract a concept without replacing a constant with a variable (parameter, instance variable, or otherwise)? If so, what does that look like? And by all means if you _do_ try this, let me know where it takes you. I know what my answer is, but it might not be the same as yours!

## Part the Third: Complexity Complex

It might seem like a non sequitur to jump from talking about abstraction to talking about complexity, but I promise the segue makes sense if you’re me.

Has anyone here been programming for the Mac since before 2011 or for iOS since before 2012? Or has anyone written apps without autolayout and then tried writing one with autolayout?

How often did you swear?

Maybe you said to yourself: Why does autolayout have to make it so hard to just set the view’s frame? This was so easy with autoresizing masks! Why does autolayout have to make everything so complicated?

To be clear: this feeling is totally justified. It really _was_ easy with frames and autoresizing masks. It really _is_ less easy with autolayout. But what does that mean, _precisely_?

There’s an interesting contrast to be drawn between _simplicity_ and _ease_. I am hardly the first to make this distinction, but I’d like to talk about it in terms of counting.

How many concrete concepts does `-firstObject` contain? When I say “concrete” I mean more or less “invariant”—concepts which are part of the contract of the method. Remember, this is, by definition, semantics, so we shouldn’t be afraid of this question just because it’s fuzzy. Let’s count them:

```objc
-(id)firstObject {
	return self.count? self[**0**] : **nil**;
}
```

(Notice that I’m using the original definition, before we abstracted it.)

I count (at least) two:

1. The index of the object that we want to extract.

2. The object that we want to return if the array is empty.

(As an aside, we could also consider the condition as being a concrete, or at least _relatively_ concrete concept. For the purposes of this exercise, however, it is not hugely informative to do so.)

How about `-nthObject:`?

```objc
-(id)nthObject:(NSUInteger)n {
	return (self.count > n)? self[n] : nil;
}
```

Using the same metrics, it looks like only one:

1. The object that we want to return if the desired index is out of bounds.

And `-nthObject:default:`?

```objc
-(id)nthObject:(NSUInteger)n default:(id)marker {
	return (self.count > n)? self[n] : marker;
}
```

By my count, 0. So it would seem that the more abstract the code, the fewer concrete concepts are involved. And indeed it makes some intuitive sense that abstraction and concreteness would be inversely linked.

Now let’s look at the definitions of `-firstObject` and `-nthObject:` in terms of their more concrete counterparts:

```objc
-(id)firstObject {
	return [self nthObject:0];
}

-(id)nthObject:(NSUInteger)n {
	return [self nthObject:n default:nil];
}
```

Looked at through the same lens, each of these definitions has captured only one concrete concept! And _this_ is more than simply semantics: perhaps you noticed that when we defined `-nthObject:default:` and then redefined `-nthObject:` in terms of it, we didn’t need to change `-firstObject` at all! It stayed exactly the same.

This is getting near the heart of the difference between simplicity and ease. When you want the first object in an array (defaulting to nil if empty), is it easier to write the code out by hand or to use `-nthObject:` and pass 0? Obviously, `-nthObject:` is easier. Is it easier to use `-nthObject:`, again passing 0, or to use `-firstObject`? Again, obviously `-firstObject` is easier. But is it _simpler_?

When we looked at the original implementation of `-firstObject`, we counted two concrete concepts. When we looked at the redefinition, we counted only one. Where did the other one go? And _why_ didn’t we have to change `-firstObject` when we changed `-nthObject:`?

Remember that the two concrete concepts in the original `-firstObject` were `0` and `nil`. The redefinition only included `0`, however, because `nil` was moved inside `-nthObject:`! `-firstObject` is in fact using _exactly_ as many concrete concepts as it was before, it’s just hidden the second one behind the call to `nthObject:`—behind the veil of abstraction. Where it belongs!

This, then, is the very core of simplicity: a simpler program has fewer concepts, a complex one has more. `-firstObject` is more complex than `-nthObject:` precisely because it is less general; the increase in abstraction mirrors an increase in simplicity.

_Ease_ is in fact _convenience_. What makes it so hard to just set the view’s frame with autolayout? It’s not complexity; it’s inconvenience.

Let me qualify that a little further: If `-firstObject` is exactly as complex before and after the existence of `-nthObject:`, then isn’t it more complex to use autolayout constraints to define the frame of the view on the screen than to assign that frame?

If that’s all you ever want to do, then yes. But what’s involved in setting the frame when you want to ensure that it behaves according to a system of rules, e.g. that it is never larger than its parent, that it is never smaller than a certain size, that it is the same width as some other thing, that it has a certain aspect ratio?

The complexity of implementing all of this quickly exceeds that of using autolayout. The key is that what I will call the _total_ complexity of `-firstObject`—that is, the number of concrete concepts used by either it or the abstractions it uses—remained the same, the _local_ complexity—the number of direct references to concrete concepts—_was_ reduced.

The _apparent_ simplicity of `-setFrame:`—really its ease—doesn’t scale. Without an abstraction of the rules for how the frame of that view should behave, you have to implement them all by hand. You _also_ have to ensure that they’re all enforced _whenever their dependencies change_. Whereas autolayout encompasses not only the rules embodied in the constraints, but also the necessary logic to keep them consistent and current—`-layout[Subviews]`, `-updateConstraints`, and so on.

This sheds new light on `-setUpNavigationItem`. How many concrete concepts does it involve?

```objc
-(void)setUpNavigationItem {
	self.navigationItem.title = @"Recipient";
	self.navigationItem.prompt = @"Where would you like the flowers sent?";
}
```

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

In an imperative language, state changes are the _status quo_, the _modus operandi_, or even the _mode de vie_. You live and breathe them, and thus all of your code is made more complex.

At the same time, abstracting imperatively, abstracting-without-abstractions, denies you the ability to deal with these extra concepts behind the veil of local complexity; that is, any code _using_ an imperative abstraction necessarily incurs the complexity of any changes it performs in a total sense (as with any other abstraction), but _also_ incurs it locally—because any other changes to the same state need to be carefully sequenced. The details leak from callee to caller, again and again, and can never be contained.

## Part the Somethingth: Flow Control

While Objective-C is an imperative language, we can view this as meaning that it can be programmed in an imperative style. Fortunately for us, we have seen that it can also be programmed in a declarative style. What would a declarative counterpart to `-setUpNavigationItem` look like?

First off, it would have to have a return value; it’s not an abstraction if it doesn’t exist. Since this is just a second approximation, let’s just return a dictionary:

```objc
-(NSDictionary *)
```

Wait a minute! Immediately we hit a wall. What do we call it? We’re basically describing how a `UINavigationItem` should be configured, but we can’t just call it `-navigationItem` and return a `UINavigationItem *` instead of a dictionary, can we? And even if we did, aren’t we just going to be calling setters on that, thus incurring more complexity?

To the first question: that sounds like a fun experiment! Let me know how it goes for you.

To the second question: If state is mutated in the forest and no one is around to observe its side effects, does it incur a penalty? Less cheekily: We’d definitely have to call setters on it. But how would anything know? If we allocate a new object, and assign some values to it, nothing outside of that method can see those individual changes. Only changes to an object that _both_ methods use will cause problems. And of course nothing can use the returned object until it has been returned, by which point the method creating it will not be changing it again!

In the meantime, we’ll still just return a dictionary from this method; let’s call it `-navigationItemState`:

```objc
-(NSDictionary *)navigationItemState {
	return @{ @"title": @"Recipient",
	          @"prompt": @"Where would you like the flowers sent?" };
}
```

Now we have only the complexity of the values, not of the changes as well. But surely this is hand-waving: this doesn’t assign anything to the `navigationItem`; it’s just moved the problem elsewhere!

Indeed it has. The sad truth is that `UIViewController`’s API is not a particularly declarative one; we’re on our own for this one. Alright then, what does it look like to assign this, perhaps in `-viewDidLoad`?

```objc
NSDictionary *state = self.navigationItemState;
self.navigationItem.title = state[@"title"];
self.navigationItem.prompt = state[@"prompt"];
```

Now I _know_ it’s just hand-waving: that’s exactly the kind of assignment we were trying to avoid!

Or is it? Since the change has been decoupled from the values, we can subclass this view controller with impunity, implementing `-navigationItemState` and returning our own values. We’re still out of luck if we don’t know it exists, of course; we would still be changing our own `navigationItem`’s properties in competition with `[super viewDidLoad]`, and thus dependent on ordering. But this is a non-issue; it’s trivially obvious that a declarative API can’t help you if you don’t use it!

But what if we only want to show the navigation item’s prompt when in portrait, and never when in landscape? Should we implement `-navigationItemStateForUserInterfaceOrientation:`?

That might be a start; but we’d still have to make sure that we’re assigning the state manually every time the device’s orientation changes. Then if we decide we also want it to change when some _other_ factor is changed, we’re forced to update it then, too, and carefully ensure that we apply all the relevant rules _every time_. Oh, and that it all happens on the main thread, if that’s a concern for any of our state transitions! It’s a lot like `-setFrame:` vs. autolayout.

Strike that: it’s _exactly_ like `-setFrame:` vs. autolayout.

We’ve missed an opportunity for abstraction. We care about more than just the state, we also care about how it depends on the state around it. Maybe we encode these rules as lines of imperative code, or maybe we encode them in some declarative abstraction—something which we construct, and whose structure implies the desired behaviour.

We further need to encode _when_ the state changes, i.e. in response to changes in what properties should the navigation item’s state change? This is an orthogonal concern, but clearly closely related: each of the navigation item state’s rules is based on some piece of state in some other object, for example the view controller’s orientation; let’s call these _dependencies_. Therefore, when the value of any dependency changes, so should the navigation item’s state.

When you put this all together, you have something like KVO. No wait, ReactiveCocoa! Or maybe Cocoa Bindings? Or maybe just a `Memo` object:

```objc
@interface Memo : NSObject
    
-(instancetype)initWithBlock:(id(^)())block;
    
-(void)addDependencyWithObject:(id)dependency keyPath:(NSString *)keyPath;
    
@property (readonly) id value;
    
@end
```

When you create a `Memo`, you give it a block that it should call when any of its dependencies is changed.

`-addDependencyWithObject:keyPath:` adds a dependency which it watches for changes, presumably with KVO.

When you call `-value`, it checks to see if it has been invalidated by a change to a dependency, and if so, evaluates the block. It returns the object that the block produced most recently.

Every Memo is effectively a cache of a calculated value that is automatically invalidated whenever one or more of its dependencies changes; it is updated lazily, when its value is requested.

This is not sufficient to cover the gamut of responses to state changes! It is perhaps instead the 90% solution: enough to cover the majority of cases. I’d encourage you to try implementing an object with this interface, and to use it and see when and why it falls short; if you’d like to see what I did, it’s in the aforementioned Navel-Gazing repository on my github account.

While it doesn’t cover everything, this minimal representation _is_ sufficient to handle changes to the notification item’s state in response to changes in orientation:

```objc
-(NSDictionary *)navigationItemState {
   	return UIInterfaceOrientationIsPortrait([UIApplication sharedApplication].statusBarOrientation)?
   		@{ @"title": @"Recipient",
   		   @"prompt": @"Where would you like the flowers sent?" }
   	:	@{ @"title": @"Recipient" };
}

 -(void)viewDidLoad {
    Memo *memo = [[Memo alloc] initWithBlock:^{
    	NSDictionary *state = self.navigationItemState;
    	self.navigationItem.title = state[@"title"];
    	self.navigationItem.prompt = state[@"prompt"];
    	return state;
    }];
    [memo addDependencyWithObject:[UIApplication sharedApplication] keyPath:@"statusBarOrientation"];
}
```

But if you try this, it doesn’t work. What’s wrong? Memos as described above are lazy, but since we’re never using the memo’s value, it’s never being calculated. Once again, the impedance mismatch between imperative and declarative styles rears its ugly head.

One way to bridge the gap would be to implement _strict_, or _greedy_ evaluation, forcing the value to be updated as soon as it is invalidated. (I’ll leave that as an exercise for the audience.) But taking a step back, it’s useful to realize that this is, again, a problem where imperative code forces us to care about ordering. Pushing this imperative code to the margins will help the rest of your code be cleaner, easier to reason about; but you can never root it out completely. It taints anything which calls it.

On the other hand, if we were always using the value of the memo at all the right times, for example if internally `UIViewController` was using some mechanism to observe changes to our memo for updates—because of course every `Memo`’s `value` is KVO-compliant!—we would see exactly the desired behaviour. The problems with ordering and timing would disappear.

In fact, the way we use `Memo` above is acting in a role that a reactive programming glossary might refer to as a sink. While `Memo` is _capable_ of acting as what’s called a signal, which responds to changes in its dependencies by computing new values on demand, it’s being used here solely to update the navigation item imperatively in response to changes. Again, we don’t use its value, so we don’t run its block. Conflating the imperative _how_ this memo computes with the declarative _what_ this memo represents has caused this bug. Worse, it’s caused us to consider complicating the abstraction with a different evaluation scheme in order to fix it. Perhaps we don’t need to change it at all.

Where a signal is declarative, producing new values when necessary and demanded of it, a sink is imperative, performing some imperative side effects as soon as it’s invalidated. Can we push these imperative effects further into the margins? `Memo` takes us from change to value; maybe we need an abstraction to take us from change to effect. Let’s call it a `Sink`.

```objc
@interface Sink : NSObject
    
-(instancetype)initWithBlock:(void(^)())block;
    
-(void)addDependencyWithObject:(id)dependency keyPath:(NSString *)keyPath;
    
@end
```

There are strong similarities with `Memo`, and maybe we should think about abstracting the dependency handling out; but for the moment let’s assume that this is someone else’s problem. Instead of adding greedy evaluation to `Memo`, we’ll add a `Sink` which just calls a block when its dependencies invalidate it; no value will be returned, since we don’t need one to call the imperative code.

```objc
-(void)viewDidLoad {
	UIApplication *app = [UIApplication sharedApplication];
	Memo *memo = [[Memo alloc] initWithBlock:^{
		UIInterfaceOrientation orientation = app.statusBarOrientation;
		return [self navigationItemStateForOrientation:orientation];
	}];
	[memo addDependencyWithObject:app keyPath:@"statusBarOrientation"];
    	
	Sink *sink = [[Sink alloc] initWithBlock:^{
		NSDictionary *state = memo.value;
		self.navigationItem.title = state[@"title"];
		self.navigationItem.prompt = state[@"prompt"];
    }];
    [sink addDependencyWithObject:memo keyPath:@"value"];
}
```

Now, when the status bar orientation changes, `Memo` is invalidated, which in turn immediately invalidates its value and notifies any observers via KVO. This triggers the sink to run its block and ask `memo` for its value, and when it does so, `memo` is evaluated.

Turning to the evaluation of the memo, we see that we’ve also now abstracted the orientation into a parameter, such that the `-navigationItemStateForOrientation:` method no longer needs to know about the `UIApplication`; removing the reference to the singleton means that there’s one fewer tightly coupled symbol, and ergo less complication. This method isn’t shown, but adding the parameter is straightforward.

We could go a step further, abstracting the dependency tracking, perhaps even providing the dependencies’ latest values to the memo such that it no longer has to duplicate the dependency key path and fetching the state. If the dependency tracking memoizes these values, then it looks more and more like a `Memo` itself, and we might think about making it a protocol instead of just a class interface. But for now, we will move along to the sink.

Having retrieved `memo`’s value, `sink` applies it to the navigation item; and, as expected, the problems of timing and ordering have gone away.

With those problems goes some knowledge and control. We no longer have concrete knowledge or control of _precisely_ when and how the state is calculated or used. It’s memoized, so it may not be calculated every time it’s used. If it’s lazy, so it may not be calculated when it’s _un_used. We don’t know how often the memoized value will be requested of us, because that’s coming from some other part of the code—or of some other person’s code.

It’s no coincidence that this loss of control and gain in precision go hand in hand; they are, in fact, one and the same. As abstraction increases, complexity decreases; and control flow is inherently complex. Specific control flow is the opposite of high abstraction. Per Kowalski, [ALGORITHM = LOGIC + CONTROL](http://www.doc.ic.ac.uk/~rak/papers/algorithm%20=%20logic%20+%20control.pdf); and in particular, _declarative_ abstractions gradually abstract the control away, replacing it with structure.

## Part the Nextth: Why Declarative?

All declarative systems are abstract, but perhaps not all abstractions are declarative. What makes an abstraction declarative?

As we saw, a return value goes a long ways: it gives us something to reason on. If that is declaring what the concept of the method _is_, and if some abstractions are more abstract than others, then maybe we could extrapolate that some such declarations are more declarative than others.

For example, `-firstObject` is clearly declarative code, but it doesn’t give us as much useful to reason on as `Memo` or even partly-imperative `Sink` does—it has meaning, but it has _less_ meaning than they do; after all, you can only refer to it in the abstract to a certain point—you must always be speaking of the array whose first object this is.

Instances of `Memo` and `Sink`, on the other hand, are values by nature, where methods are less so (not being truly first-class concepts in the language). As we have seen, they can be combined with other objects quite flexibly. It does not seem too much of a stretch to describe them as being _more declarative_ than, say, the average app delegate or those view controllers you sometimes see where it’s clear the developers wasn’t going to add one single other controller class to the project without a gun to their head.

This is a bit off in the weeds; I’m not sure you’ll agree with me, and I’m not sure anyone else will either. (Take that as an imperative, if you will, to test this declaration.) But part of what I think about when I think of “declarative programming” is the notion that you are constructing a system of objects at runtime out of which the desired behaviour falls naturally: a necessary consequence of the structure.

To give an example of this, the appearance of a view hierarchy is a consequence of the structure of the views, and their arrangement and visual properties. All of these are declared properties of the view controllers; in this sense, we can think of a view hierarchy as being a declarative system, whose value is the contents of the layer, or perhaps the screen buffer.

View controllers have a similar hierarchy which provides the semantic structure of the app; this, and the paths through which the user will be taken at runtime, is the structure made explicit in storyboards.

Views and view controllers are, basically, a solved problem—Apple has provided these things, and by and large we use them. It’s important to realize, however, that they have not cornered the market. We solve problems all day every day, generally more than we cause; that’s why anyone bothers to pay us. Some of these problems are likely to have declarative solutions (after all, if you can’t talk about what these things are, then what are they, really?); some of those declarative solutions may well be qualitatively better than how we might be solving them otherwise—in fact, I think that’s extremely likely, given what we’ve seen about their behaviour with regard to time and ordering relative to imperative solutions.

How can we find such solutions?

By pursuing simplicity.

## Part the nth: The Composition of Abstraction

The Structure and Interpretation of Computer Programs, in my opinion the most important work on our field yet produced, tells us early on that _the most important features_ of every programming language are its facilities for abstraction and composition, and that we should therefore judge every language by these features.

This is hyperbole, but I think it’s justified hyperbole. Composition is abstraction’s dual; where abstraction is breaking a problem into simpler components, composition is reassembling those into the solution. Constructing an abstraction is generally itself composition of other abstractions; any time you use an abstraction, you are composing.

Where abstraction reduces complexity, composition increases it. This is warranted, of course; if you have a complex problem, then the solution—the negative space around the problem, if you will, sharing its surface with all its fractal knots and whorls—must likewise be complex!

Fortunately, imperative composition and declarative composition model the complexity differently. Imperative composition models the complexity in the structure of the code, with every piece of the solution expressed explicitly in the control flow. Declarative composition models the complexity in the structure of the object graph at runtime, with every piece of the solution implied in the code and how the structure is built—and in effect, how it is automated, since we rarely build an object graph by hand.

Imperative composition is the approach we’re all familiar with; we write code to do one thing, and then to do another thing. Perhaps we write a loop. This is `-setUpNavigationItem` or `-setChild:`—imperative code, ordered, normal, and regrettably fragile.

Declarative composition, however, is the assembly of declarative abstractions by appending or alternating declarative abstractions with other declarative abstractions. An array is a declarative abstraction; so is a tree node; or a view (with subviews); or a view controller (with child view controllers); or an autolayout constraint with its items. So is a `Memo` with dependencies that turn out to themselves be other `Memo`s, if we take that route. And for much the same reason, so is a `Sink`, despite its imperative process.

Even though a declarative solution will model the complexity in the structure of the object graph and not in the structure of the code, managing this complexity is important—this is, after all, a key responsibility of every programmer who wishes to continue solving problems tomorrow without having to rewrite all of their code from scratch. We need to be able to debug, we need to have acceptable performance, and we need, fundamentally, to understand what’s going on—to have a mental model.

It is therefore in our best interests to ensure that the abstractions we build are as simple as possible: simple abstractions are more flexible, meaning more easily composed together, because they do not introduce factors not necessary to their operation.

Each instance variable you add to a class increases its complexity. Each reference to a singleton increases its complexity. Each reference to a global variable increases its complexity. Each line of code increases its complexity.

There is no analogous cost to adding classes to the project. We may feel like we incur some terrible penalty to have to add a new class, but as with integers, these things aren’t rationed; we aren’t in danger of exhausting a 32-bit address space with classes any time soon.

Ergo, more, simpler classes are “better”—in the sense of allowing “more declarative” solutions—than are fewer, more complex classes. And at this scale, we can use lines of code as a rule of thumb: longer classes are probably more complex, and are less likely to be adequately declarative.

`Memo` is fairly minimalistic. We could factor out the observation of dependencies—that’s orthogonal to the core invalidate/evaluate/value concept, and could perhaps be used by Sink. We could have the dependencies themselves be `Memo`s, giving this operation closure. However, we see diminishing returns at some point; while I might seem to sneer at convenience, it’s ultimately what pays Apple’s bills, and mine, and likely yours. As a declarative programmer, you are designing APIs; as an API designer, developers are your users. And users are people too!

In the end, you have to ship. Experimenting with these ideas will not help you ship on Monday. Experimenting with these ideas will not help you ship next week. Probably not next month. But three to six months from now, having this experimentation behind you, having experience with these ideas in your repertoire, it absolutely will.

Declarative code is simpler. It’s usually easier to read, in my experience. It’s smaller, if more spread out; easy to get around. It’s more reliable simply because each component has fewer moving pieces which could fail, and fewer joints between them that could break. On top of all of that, the lack of ordering means that declarative code is immune to race conditions, the single largest issue with concurrency. And it is clearly the direction that the market and the industry is heading in: there is a lot of ongoing, exciting research being done in declarative languages and systems and how to survive in an imperative world.

Further, Apple is increasingly implementing declarative APIs, and being able to use them isn’t enough; if we are to be able to tackle harder and harder problems, if we are to learn and grow, and if we are to stay relevant in the market, we need to be able to write them.

But again, it’s important to remember that there is no silver bullet. You _will_ run into problems along the way:

Declarative code can be slower than the imperative code you might have written—but it can also be faster, since having the structure available at runtime might give you hints about how the objects are used which could help you to automatically organize them in memory for better locality of reference. Compilers do this with the syntax trees they build; that’s approximately what optimizers are. We can do so too.

Declarative code can be harder to debug than the imperative code you might have written—but it can also be easier, since you have the structure at runtime to look at; you can analyze the state of the graph as a whole instead of just stepping through it. It’s necessarily _different_, of course; if control flow is abstracted, it becomes increasingly difficult for your the API’s client to simply use lldb to debug it.

Declarative code can use more memory than the imperative code you might have written—but it can also use less. Once again, the increase in context could be leveraged to allow you to optimize for space as well as time.

Declarative code can be less familiar than the imperative code you might have written. Only time and practice will fix this one.

No declarative programming techniques will solve these problems for you; they may in fact require you to solve them for your API’s clients, where before they might have been your clients’ problem. This is a responsibility, but it’s also an opportunity, as is so often the case: API design is not a very common expertise in the market.

There may also be problems that don’t lend themselves to a declarative solution. Even if a particular problem does, your particular solution may not allow for some particular functionality or flexibility that some particular client requires. Every abstraction reduces the potential computation that can be done with it by some small amount; unless your abstraction is a Turing-complete language, it will necessarily reduce the flexibility of its clients to some degree. This is fine; this is, in fact, the point. When you can do everything, doing anything at all requires a lot of time and effort—and really, that’s what we’re here for.

## Epilogue: Abstraction in the Abstract

There is a proverb about a man who points at the moon, and says “There is the moon.” We do not think he is referring to his finger, and so the lesson is: “Do not mistake the pointing finger for the moon.”

Really, his finger is a symbol, a stand-in, to avoid his having to take you to the moon and smack you over the head with it before you get the point. The moon is a concrete thing; his finger, and indeed the word “moon,” are abstractions. We can be even more abstract: other worlds have moons, too. _The_ moon is really just _a_ moon. This is my moon; there are many like it, but this one is mine.

(Why are we uncomfortable using articles and pronouns in method names?)

To abstract is to identify an idea, a concept, as a unique thing which can be reasoned and acted upon in isolation. It is to give it a name; to define that name with that concept. This is equally true in language and in code.

We abstract because it’s difficult to reason about something which we do not have a word for. Sometimes we don’t have good enough words for things that are important to us: the feeling of summer’s warmth passing as fall’s crispness enters the air; or of the bittersweet experience of a trusted colleague and friend leaving to work on something important to them: that simultaneous grief at your loss and joy at their gain. Or how the joy outweighs the grief.

We have these concepts, but it’s hard to think or talk about them without the symbols for them, the words. Abstracting gives us what we lack. We name unique objects, and categories of objects, and also concepts.

Sometimes these are new words, like _zeitgeist_ or _monad_. These are _never_ new ideas, however: we cannot name an idea we have not had. Even if we are the first and only to think of an idea, the selection of the word must follow, never lead. So it is with our software: we must always abstract from something we already know, and this is done most easily with working code.

If abstraction is vocabulary, then composition is grammar: phrasing our abstractions together such that we can apply them to connote meaning (and behaviour!). Using this most effectively is going to require at least as much practice, hand-in-hand with abstraction: they are yin and yang, simultaneously candlestick and faces, negative space and figure.

So if it ain’t broke, you aren’t trying hard enough yet. It’s going to be hard and uncomfortable and it might make you unpopular if you do it at work right before a deadline, but it’s the only way to master the skills, and the only way to apply these skills once mastered to concepts you were previously unfamiliar with.

By all means, therefore, make mistakes. Break things.

Just do so on a branch.

## Q?&A!

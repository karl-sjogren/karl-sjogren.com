---
title:      "Switching from Moq to FakeItEasy"
date:       2023-08-19 12:00:00
tags:       c# moq fakeiteasy regex
---

[Moq](https://github.com/moq/moq) is a great library for mocking your
dependencies in your C# unit tests. I've used it for almost 10 years after
I switched from Rhino Mocks. However, after the last weeks controversy
surrounding Moq and its creator I don't trust it in my codebases anymore.
I won't reiterate what happened, you can check out the issues in the repo or
through [Google](https://www.google.com/search?q=moq%20controversy) if you
don't know what I'm talking about.

While Moq is still a great library, there are alternatives out there. I've
checked out [FakeItEasy](https://fakeiteasy.github.io/) and it seems to be a
good alternative to me (there is also [NSubstitute](https://nsubstitute.github.io/)
which also looks good). I won't go into detail here why I selected one over
the other, but looked quite equal to me.

So last friday I started by converting my current project at work which is a
medium sized project which had somewhere between 500 and 600 tests. Since I
love writing regular expressions and don't get to do it nearly enough, I set
to work! With some basic expressions I managed to convert all the tests in
about 3 hours of which maybe 30 minutes was spent handling special cases.

We'll get to the actual regular expressions later, but lets first look at
how to convert the most standard cases from Moq to FakeItEasy.

## Converting a simple Moq setup

Creating a mocked object is pretty similar in both projects. The main
difference is that FakeItEasy uses a static factory to create the mocks while
Moq creates a new instance of the `Mock` class.

So for Moq you need to call `.Object` on the mock object to get the mocked
instance while FakeItEasy returns the mocked instance directly. Moq has a
simple method of creating a simple mock, if you don't actually want to set up
anything on it. If so then you can use `Mock.Of<T>` instead.

```csharp
// Moq
var animalMock = new Mock<IAnimal>();
var sut = new AnimalFeeder(animalMock.Object);

// or
var animalMock = Mock.Of<IAnimal>();
var sut = new AnimalFeeder(animalMock);

// FakeItEasy
var animalFake = A.Fake<IAnimal>();
var sut = new AnimalFeeder(animalFake);
```

Easy right? This could easily be automated using a simple regular expression!

> The regular expressions in this article was executed using the global search and
replace in VSCode. They aren't perfect, for example they don't handle line breaks
in all places that they can occur, but more of a best effort to speed up the
conversion process.
{: .prompt-info }

First well need to find all the `new Mock<` and `Mock.Of<` and replace it with
`A.Fake<`.

```sh
Search: (new\sMock|Mock\.of)<
Replace: A.Fake<
```

After this we also need to fix the `.Object` calls. This can be a bit tricky
depending on the codebase. If you have other obejcts with a `.Object` property
then this will also replace those, so you might need to do some manual work here.

But in my case this worked just fine.

```sh
Search: ([a-z]*?)\.Object([,);\s])
Replace: $1$2
```

Next up we'll look at how you set up a simple method call to return a value. In
Moq this is done by calling `Setup` on the mock object. In FakeItEasy you call
the static method `A.CallTo` and pass in the mocked object.

```csharp
// Moq
var animalMock = new Mock<IAnimal>();
animalMock.Setup(x => x.Eat()).Returns(true);
var sut = new AnimalFeeder(animalMock.Object);

// FakeItEasy
var animalFake = A.Fake<IAnimal>();
A.CallTo(() => animalFake.Eat()).Returns(true);
var sut = new AnimalFeeder(animalFake);
```

This basic example looks easy enough but there are some differences. Moq has
`.Returns()`and `.ReturnsAsync()` while FakeItEasy handles both cases using
just `Returns()`. The Moq methods can also take `Func<T>` to return a computed
value while FakeItEasy need a call to `ReturnsLazily()` for that.

```sh
Search: ([a-z]+?)(?:[\s\n]*?).Setup(?:Get)?\((?:[a-z]*?)\s=>\s(?:[a-z\s\n]*?)\.([a-z0-9._="'<>,\s\n[\]()]*)\)([\s\n]*?).Returns(?:Async)?(<?[a-z0-9]*?>?)\(((?:.|[\n])*?)\);
Replace: A.CallTo(() => $1.$2)$3.Returns$4($5);
```

And as noted above, we also need to change any `.Returns(Func<T>)` with
`ReturnsLazily(Func<T>)`.

```sh
Search: \.Returns(<?[a-z0-9]*?>?)\((\(.*?\)|[a-z0-9]*?)\s=>
Replace: .ReturnsLazily$1($2 =>
```

It will be a similar expression for `Throws()` and `ThrowsAsync()`.

```sh
Search: ([a-z]*?)(?:[\s\n]*?).Setup\((?:[a-z]*?) => (?:[a-z]*?)\.([a-z0-9.="'_<>,\s[\]()]*)\)([\s\n]*?).Throws(Async)?\(((?:.|[\n])*?)\);
Replace: A.CallTo(() => $1.$2)$3.Throws$4($5);
```

So what about methods with arguments? Well, that's pretty similar as well. Moq
uses the static `It` class with `Ìt.IsAny<T>()` to match any argument while
FakeItEasy uses `A<T>.Ignored` to do the same. `A<T>.Ignored` can also be shortened
to `A<T>._` for brevity (which I prefer).

```csharp
// Moq
var animalMock = new Mock<IAnimal>();
animalMock.Setup(x => x.Eat(It.IsAny<string>())).Returns(true);
var sut = new AnimalFeeder(animalMock.Object);

// FakeItEasy
var animalFake = A.Fake<IAnimal>();
A.CallTo(() => animalFake.Eat(A<string>._)).Returns(true);
var sut = new AnimalFeeder(animalFake);
```

To match a specific argument Moq uses `It.Is<T>(x => x == value)` while
FakeItEasy uses `A<T>.That.Matches(x => x == value)`.

```csharp
// Moq
var animalMock = new Mock<IAnimal>();
animalMock.Setup(x => x.Eat(It.Is<string>(x => x == "Banana"))).Returns(true);
var sut = new AnimalFeeder(animalMock.Object);

// FakeItEasy
var animalFake = A.Fake<IAnimal>();
A.CallTo(() => animalFake.Eat(A<string>.That.Matches(x => x == "Banana"))).Returns(true);
var sut = new AnimalFeeder(animalFake);
```

These can be quite easily be matched and replaced like this.

```sh
Search: It\.IsAny<([a-z<>[\]]*?)>\(\)
Replace: A<$1>._
```

```sh
Search: It\.Is<([a-z<>[\]]*?)>\((.*?)\)
Replace: A<$1>.That.Matches($2)
```

## Converting callbacks

Moq supports callbacks for when a method is called by calling `Callback<T>()`.
FakeItEasy supports this as well using `.Invokes`.

```csharp
// Moq
var animalMock = new Mock<IAnimal>();
animalMock.Setup(x => x.Eat(It.IsAny<string>())).Callback<string>(x => Console.WriteLine(x));
var sut = new AnimalFeeder(animalMock.Object);

// FakeItEasy
var animalFake = A.Fake<IAnimal>();
A.CallTo(() => animalFake.Eat(A<string>._)).Invokes<string>(x => Console.WriteLine(x));
var sut = new AnimalFeeder(animalFake);
```

The following expressions sets this up in most cases, I did however have to
manually fix some case though.

```sh
Search: ([a-z]+?)(?:[\s\n]*?).Setup(?:Get)?\((?:[a-z]*?)\s=>\s(?:[a-z\s\n]*?)\.([a-z0-9._="'<>,\s\n[\]()]*)\)([\s\n]*?).Callback(?:Async)?(<?[a-z0-9,\s]*?>?)\(((?:.|[\n])*?)\);
Replace: A.CallTo(() => $1.$2)$3.Invokes($5);
```

## Converting verifications

Verifying that a call has happened is done by calling `.Verify()` within Moq.
For FakeItEasy this is also done using `A.CallTo()` and then `MustHaveHappened()`
or `MustNotHaveHappened()`.

I don't use this kind of verification that often but these expressions fixed
my scenarios. It won't fix calls using `Times.AtLeastOnce()` and other convenience
methods though. It could be easy to add that if needed though.

```sh
Search: ([a-z]*?)(?:\s*?).Verify\((?:[a-z]*?)\s=>\s(?:[a-z\s\n]*?)\.([a-z0-9._="'<>,\s[\]()]*),[\s\n]*?Times.Once\((.*?)\)\);
Replace: A.CallTo(() => $1.$2)$3.MustHaveHappened();

Search: ([a-z]*?)(?:\s*?).Verify\((?:[a-z]*?)\s=>\s(?:[a-z\s\n]*?)\.([a-z0-9._="'<>,\s[\]()]*),[\s\n]*?Times.Never\((.*?)\)\);
Replace: A.CallTo(() => $1.$2)$3.MustNotHaveHappened();

Search: ([a-z]*?)(?:\s*?).Verify\((?:[a-z]*?)\s=>\s(?:[a-z\s\n]*?)\.([a-z0-9._="'<>,\s[\]()]*),[\s\n]*?Times.Exactly\((.*?)\)\);
Replace: A.CallTo(() => $1.$2).MustHaveHappened($3, Times.Exactly);
```

## Other things to change

### Capture.In

Moq supports an easy way to save arguments passed to a method in a list using
`Capture.In`. FakeItEasy currently can do this as well, but requires using
`Invokes` and adding it to a list manually.

There are probably a lot of other advanced use cases to migrate, but I haven't
run in to them in any of the projects migrated so far.

```csharp
// Moq
var eatenFoods = new List<string>();

var animalMock = new Mock<IAnimal>();
animalMock.Setup(x => x.Eat(Capture.In(eatenFoods)));
var sut = new AnimalFeeder(animalMock.Object);

// FakeItEasy
var eatenFoods = new List<string>();

var animalFake = A.Fake<IAnimal>();
A.CallTo(() => animalFake.Eat(A<string>._)).Invokes<string>(x => eatenFoods.Add(x));
var sut = new AnimalFeeder(animalFake);
```

## Source

The regular expressions shown above are collected in a gist which might get
updated more frequently then this article.

[karl-sjogren/af0f6db69f4c0e473f66241ec10e8eff](https://gist.github.com/karl-sjogren/af0f6db69f4c0e473f66241ec10e8eff)

## Closing words

I've so far updated one project at work as well as some of my stuff on GitHub.
I will most likely keep migrating stuff to FakeItEasy when I get back to older
stuff still using Moq, or maybe I'll try out NSubstitute. All three libraries
seem to be more or less comparable in terms of features, it is mostly how things
are accomplished that changes.

Would I go back to Moq if the controversy clears up? Maybe, or at least maybe I
wouldn't migrate all my projects. But right now it seems like that won't happen
and if it is forked I would like to see that the fork is actually active and not
just a source copy without any future activity.

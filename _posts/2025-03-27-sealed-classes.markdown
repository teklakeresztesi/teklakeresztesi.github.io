---
layout: post
title:  "Sealed classes in Flutter"
date:   2025-03-27 19:06:13 +0200
categories: flutter
---
I recently interviewed for a Senior Flutter Developer position and at the live code review session I was asked about sealed classes and I gave a wrong answer. So I looked into the topic in more detail.
 
<br/>
# What Are Sealed Classes?

A sealed class in Dart is a class that restricts which classes can extend it. Unlike normal classes, sealed classes allow only a predefined set of subclasses within the same file. This makes them particularly useful for handling state management and complex app logic. - This means that once I create a sealed class, I have to declare all subclasses in the same file and it gives me the ability to use a switch to check for the different subclasses.


***The syntax is the following:***
{% highlight dart %}
sealed class AuthState {}

class Authenticated extends AuthState {
  final String userId;
  Authenticated(this.userId);
}

class Unauthenticated extends AuthState {}

class Loading extends AuthState {}
{% endhighlight %}

<br/>
I can use the sealed class in a switch, and will get a compiler error if I forget to handle one of the cases.

***Usage:***
{% highlight dart %}
BlocBuilder<AuthCubit, AuthState>(
  builder: (context, state) {
    switch (state) {
      case Authenticated(:var userId):
        return Text("Welcome, $userId!");
      case Unauthenticated():
        return Text("Please log in.");
      case Loading():
        return CircularProgressIndicator();
    }
  },
)
{% endhighlight %}

<br/>
# The trick question

But the question wasn't about the usage of sealed classes, all of the authState classes extended AuthState, except for one, which implemented it. In the heat of the moment I said that the code won't compile, since only abstract classes can be implemented, but then I tried it in VSCode and I didn't get a compile error, instead the state looked like any of the other states.
***So what IS the difference between extends and implements in this case?***

{% highlight dart %}
sealed class AuthState {}

class Authenticated extends AuthState {
  final String userId;
  Authenticated(this.userId);
}

class Unauthenticated extends AuthState {}

class Loading extends AuthState {}

class AuthError implements AuthState {
  final String message;
  AuthError(this.message);
}
{% endhighlight %}

<br/>
# Extends vs. implements

When a class extends another class, it inherits all the properties and methods of the base class. The subclass is considered a specialized version of the base class. It can override methods or add new functionality, but it is still tightly coupled to the base class.

When a class implements another class (or interface), it does not inherit any properties or methods from the base class. Instead, it is required to provide its own implementation of all the methods and properties defined in the base class (or interface). This is more like a contract that the implementing class must fulfill.

<br/>
# The correct answer

The catch of the question was actually that sealed classes are implicitly abstract, so it is syntactically correct to implement a sealed class, but when using bloc states, we generally want to extend the base state, because our states will be specialized ones of the sealed class, carrying extra information, not only adhering to a contract.

<br/>
# So let's break the code

I create my bloc states with the [freezed] package, and add factory constructors to my different states.

{% highlight dart %}
part 'auth_state.freezed.dart';

@freezed
sealed class AuthState with _$AuthState {
  const factory AuthState.loading() = Loading;
  const factory AuthState.authenticated(String userId) = Authenticated;
  const factory AuthState.unauthenticated() = Unauthenticated;
}

class AuthError implements AuthState {
  final String message;
  AuthError(this.message);
}
{% endhighlight %}

So this does give me an error for the AuthError class:
'Missing concrete implementations of '_$AuthState.map', '_$AuthState.mapOrNull', '_$AuthState.maybeMap', '_$AuthState.maybeWhen', and 2 more.
Try implementing the missing methods, or make the class abstract.'

This means that AuthError is missing the implementation for the contract that the generated class _$AuthState mixin requires.

<br/>
# Conclusion

Technically you can implement the sealed class, since it is implicitly abstract, I personally prefer to use the freezed version with factory constructors, this also means there's no extends or implements in the code.
I'm definitely glad that this question made me think of sealed classes a bit more.

<br/>

# Native

If you're interested how this works in native code, check out [this article][medium] to see how to use sealed classes in kotlin or enums in swift.

<br/>

[freezed]: https://pub.dev/packages/freezed
[medium]: https://medium.com/@da_pacheco/using-kotlins-sealed-class-to-approximate-swift-s-enum-with-associated-data-7e0abac88bbf

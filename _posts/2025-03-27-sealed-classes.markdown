---
layout: post
title:  "Sealed classes in Flutter"
date:   2025-03-27 19:06:13 +0200
categories: flutter
---
I recently interviewed for a Senior Flutter Developer position and at the live code review session I was asked about sealed classes and I gave a wrong answer. So I looked into the topic in more detail.
 
<br/><br/>
# What Are Sealed Classes?

A sealed class in Dart is a class that restricts which classes can extend it. Unlike normal classes, sealed classes allow only a predefined set of subclasses within the same file. This makes them particularly useful for handling state management and complex app logic.


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

<br/><br/>
This way you can use the sealed class in a switch, and you will get a compiler error if you forget to handle one of the cases.

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

<br/><br/>
# The trick question

But the question wasn't about the usage of sealed classes, all of the authState classes extended AuthState, except for one, which implemented it. In the heat of the moment I said that the code won't compile, since only abstract classes can be implemented, but then I tried it in VSCode and I didn't get a compile error, instead the state looked like any of the other states. So what IS the difference between extends and implements in this case?

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

<br/><br/>
# Extends vs. implements

When a class extends another class, it inherits all the properties and methods of the base class. The subclass is considered a specialized version of the base class. It can override methods or add new functionality, but it is still tightly coupled to the base class.

When a class implements another class (or interface), it does not inherit any properties or methods from the base class. Instead, it is required to provide its own implementation of all the methods and properties defined in the base class (or interface). This is more like a contract that the implementing class must fulfill.

<br/><br/>
# The correct answer

The catch of the question was actually that sealed classes are implicitly abstract, so it is syntactically correct to implement a sealed class, but when using bloc states, we generally want to extend the base state, because our states will be specialized ones of the sealed class, carrying extra information, not only adhering to a contract.

<br/><br/>

I started the [The Bare Minimum][the-bare-minimum] series to present the bare minimum knowledge needed in Flutter to make your life easier.

Check out my [Linkedin profile][linkedin] for more info on how to get the most out of Flutter. If you have questions, feel free to reach out.

[linkedin]: https://www.linkedin.com/in/tekla-keresztesi-02887298/
[the-bare-minimum]: https://teklakeresztesi.github.io

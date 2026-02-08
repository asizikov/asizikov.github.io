---
date: "2016-09-24T00:00:00Z"
image: /images/code-for-reviewer/matrix.png
title: Write code for the reviewer, not for the compiler
tags:
  - programming
slug: code-for-reviewer
---

![Code](/images/code-for-reviewer/matrix.png)

I've been doing code reviews on a daily basis for several years for now. This activity is very different from what I do as a developer.

Most of the time I try to understand, debug, fix or update existing code. Less often developers write new code from scratch.  I think it's safe to say that we all know how to work with a code written by another human.

One might want to say that if we know how to work with other people's code, we can be good at reading and understanding it during the code review process.

Unfortunately, there is a difference. When we work with a code we have it opened in our editor or IDE. It doesn't matter if it's a modern heavy IDE or a lightweight editor. What's important is that this is a tool which we are familiar with. 

We know how to get extra information about any line code on the screen. In my case, it's a Visual Studio with ReSharper and several more extensions. I have color coded info, I have tooltips and I can navigate through the code. 

However, when reviewing a Pull Request on GitHub I have just a basic syntax highlighting in the editor.    

![Pull Request sample](/images/code-for-reviewer/nancy-pr.png)

## Problem

Modern programming languages allow a developer to skip some symbols and take some shortcuts. It's different for every language, I'll use C# as an example in this post.

Let's look at this snippet: 

```csharp
public void ProcessSomething(Guid accountId)
{
  AccountBalance accountBalance = 
    this.balanceCalculator
        .GetAccountBalanceAsAtDate<Guid>(accountId: accountId, date: this.DateUtils.Yesterday);
  this.CalculateFees(accountBalance);
}
``` 

Here we have a short method implementation. C# keywords are in blue-ish color, method names are in magenta, the rest is gray. 

Even if you don't know C# you can read and understand what's going on there. 

If you do know C# you'd notice that this code is rather redundant. However, this is pretty much how compiler sees your code. Ok, almost :) 

Let me read this code. I'll try to read all the information which I see here.

> This method takes an account id (GUID). First it calls the **GetAccountBalanceAsAtDate** generic method with **Guid** generic parameter. This method is an instance method of field **balanceCalculator**. 
This is a current class instance field.  
> There are two parameters passed to that method. The first one is an account id, the parameter name is **accountId**. The second parameter name is **date**, and we pass the value of the property called **Yesterday** of the current class instance property called **DateUtils**.
> 
>The result is assigned to the local variable **accountBalance** of **AccountBalance** type. 
> After that the result is passed to the instance method **CalculateFees**. 

That's a lot of information, right? We know types, parameter names, we know that methods and properties are not static and so on. 

We still don't know what type `Yesterday` is, but the rest is clear. We have plenty of information about the logic of the method as well as about types and even some details about `GetAccountBalanceAsAtDate` method signature.

## Getting simpler

When I look at this code in my IDE I see that some code is marked as redundant (grayed out): 

![Same Code in IDE](/images/code-for-reviewer/redundant-code.png)

What if I apply suggested "Remove redundant code" refactorings? We'd loose some information about this method. I'm going to be careful here.

### Argument names

Explicit argument names here are probably the least important information. Even more, it's a bit misleading. Are the arguments optional or not? We don't know that. (Well, I do know, and they are not optional).

Let's remove them: 

![Code in IDE, argument names omitted](/images/code-for-reviewer/redundant-code-no-args-names.png)

I'm going to repeat our exercise one more time: 

> This method takes an account id (GUID). First it calls the **GetAccountBalanceAsAtDate** generic method with **Guid** generic parameter. This method is an instance method of field **balanceCalculator**. 
This is a current class instance field.  
> There are two parameters passed to that method. The first one is an account id, and as a second parameter  we pass the value of the property called **Yesterday** of the current class instance property called **DateUtils**.
> 
>The result is assigned to the local variable **accountBalance** of **AccountBalance** type. 
> After that the result is passed to the instance method **CalculateFees**. 

Hm, looks like we didn't lose much information, right? 

### Explicit variable type

When reviewing this snippet we don't know much about `AccountBalance` type. Is it a `struct` or a `class`? What kind of data does it contain? 

In fact, we can just omit the type declaration here. 

![Code in IDE, use var instead of Explicit type](/images/code-for-reviewer/redundant-code-use-var.png)

So far so good. We don't know the type, but we do know that the type is correct. We were able to pass it to `CalculateFees` method, and our code compiles. 

### Generic parameter type

What kind of information could the reviewer gather from the explicit generic parameter type specification? 

Honestly, I don't know. Perhaps we can figure out that the method **is** generic. Well, good to know, but as long as the compiler can build the code, it doesn't matter for the reviewer. 

After all the reviewer is not trying to catch compilation errors, eh?

![Code in IDE, implicit generic parameter type](/images/code-for-reviewer/redundant-code-no-generic-type.png)

### this qualifier

This is a very painful moment. 

There is a StyleCop rule [SA1101: PrefixLocalCallsWithThis](http://stylecop.soyuz5.com/SA1101.html) which originally [aimed against underscore prefixes](https://blogs.msdn.microsoft.com/sourceanalysis/2008/05/25/a-brief-history-of-c-style/) for instance variables.

First of, all we should understand, that the compiler does know that `CalculateFees()` is an instance method of the current class. It does know where `DateUtils` comes from. 

Honestly, how often do you need to know if `balanceCalculator` is an instance variable? That might be important when you editing the code. Not very often, but I can buy this argument. 

But is that important during the code review? I'd say that it's not. Id doesn't matter for this method. The method is short enough so we can see that `balanceCalculator` is not declared inside the method.
If the method was longer, it would not be important anyway. What's important is that the `balanceCalculator` was used to calculate the balance. 

Nobody would argue that the code outside of the editor contains less information about its structure. The absence of this knowledge is not that bad, though. 

For example, When reviewing the code outside of the editor no one can tell what's going on here:

```csharp
Foo(Bar.Car);
Foo(Barr.Car);
Foo(Barrr.Car);
``` 

Each `*.Car` could be either a property of current class instance property or a static property of another class or an enum value. `this.` qualifier will help us with the first option, but that's it.

In your IDE you'll spend a second to find out, right? I'll use color codes: 

![Code in IDE, no redundant code](/images/code-for-reviewer/color-codes.png)

You can hover or navigate to it. Doesn't really matter.

When you read this code as a plain text you can only tell that the method `Foo` is doing something with the car. You have lost some information, and you only have a semantics of the code here. 

And this is just fine. You can focus on what the code does, not on how it does. The "how" part is checked by the compiler.

It requires some courage, but I'm going to remove all `this.` prefixes.

## Final look 

![Code in IDE, no redundant code](/images/code-for-reviewer/no-redundant.png)

So, what do we have here? As we learned, as a reviewer we need to focus on the logic, and we shouldn't try to chase any errors which can be caught by the compiler.

Let's read it one more time: 

> The method takes an account Id as an argument, calls **GetAccountBalanceAsAtDate** method of the **balanceCalculator**. **accounId** is passed as a first parameter, the second parameter is a value **Yesterday** of **DateUtils**.
> The result of that method call is used as a parameter for  **CalculateFees** method.

Now scroll up and compare with the first attempt. Much shorter, right?

Ok, the reviewer has no information about types, about the nature of methods and properties. They do not know what `DayUtils` is. It might be an `enum`, or a static class or anything else. 

The only information that reviewer has is the logic of the method. And now they can focus on it because that's what is important here after all. The logic, not the AST structure.

![Review ](/images/code-for-reviewer/review.png)


Please, let the compiler to its job, and help the reviewer focus on what **does** matter here.

Feel free to comment on this if you agree or disagree. 

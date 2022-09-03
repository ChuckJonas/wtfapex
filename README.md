# What the f\*ck, Apex?

> A list of funny and tricky Apex examples

**_Inspired by [wtfjs](https://github.com/denysdovhan/wtfjs)_**

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
## 📖Table of Contents

- [✍🏻 Notation](#-notation)
- [👀Examples](#examples)
  - [When a boolean is not a boolean](#when-a-boolean-is-not-a-boolean)
  - [String compare is case-insensitive (except when it's not)](#string-compare-is-case-insensitive-except-when-its-not)
  - [Object `equals` override](#object-equals-override)
  - [Shadowing System (global) classes](#shadowing-system-global-classes)
  - ["Phantom" Inner Class Type Equivalency](#phantom-inner-class-type-equivalency)
  - [List<Id> `contains` & `indexOf` is broken](#listid-contains--indexof-is-broken)
  - [`final` parameters "exist", but can be reassigned](#final-parameters-exist-but-can-be-reassigned)
  - [Fulfilling Interface Contracts with Static Methods](#fulfilling-interface-contracts-with-static-methods)
  - [Exceptions are "exceptional"](#exceptions-are-exceptional)
  - [`System` can have ambiguous return types](#system-can-have-ambiguous-return-types)
  - [Odd List Initialization bug](#odd-list-initialization-bug)
  - [Local Scope Leak](#local-scope-leak)
  - [Broken type inference for `Set<>`](#broken-type-inference-for-set)
  - [String.Format with single quotes](#stringformat-with-single-quotes)
  - [Line continuation breaks for static method](#line-continuation-breaks-for-static-method)
  - [Fun with Hashcodes](#fun-with-hashcodes)
  - [JSON Serialization](#json-serialization)
  - [Generics (parameterized interfaces) exist, but you can't use them](#generics-parameterized-interfaces-exist-but-you-cant-use-them)
  - [Polymorphic Primitives](#polymorphic-primitives)
  - [Invalid HTTP method: PATCH](#invalid-http-method-patch)
- [🔧 Since Fixed](#-since-fixed)
  - [Mutating Datetimes](#mutating-datetimes)
  - [More hashcode fun](#more-hashcode-fun)
  - [Initializing Abstract Classes](#initializing-abstract-classes)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## ✍🏻 Notation

**`// ->`** is used to show the result of an expression. For example:

```java
1 + 1; // -> 2
```

**`// >`** means the result of `System.debug` or another output. For example:

```java
System.debug('hello, world!'); // > hello, world!
```

**`// !`** Means a runtime exception was thrown:

```java
Decimal d = 1 / 0; // ! Divide by 0
```

**`// ~`** Code fails to compile:

```java
Decimal d = 'foo'; // ~ Illegal assignment from String to Decimal
```

**`//`** is just a comment used for explanations. Example:

```java
// Assigning a function to foo constant
Foo foo = new Foo();
```

## 👀Examples

### When a boolean is not a boolean

```apex
Boolean b;
if(b!=true) system.debug('b is not true');   // > b is not true
if(b!=false) system.debug('b is not false'); // > b is not false
if(b){}                                      // ! Attempt to de-reference a null object
```

See [Advanced Apex Programming in Salesforce](http://advancedapex.com/2012/08/23/funwithbooleans/) for explanation.

### String compare is case-insensitive (except when it's not)

```java
String x = 'Abc';
String y = 'abc';
System.assert(x == y);
System.assertEquals(x, y); // ! Expected: Abc, Actual: abc
```

See [explanation on StackExchange](https://salesforce.stackexchange.com/questions/80456/is-there-any-difference-in-equals-and-for-string-variables)

### Object `equals` override

Try to `override` the `equals` method on any class and you'll be greeted with a very unexpected compile error: `@Override specified for non-overriding method`.

However, just remove the `override` keyword and it compiles!

At first glance it might seem like this works, but it is in fact very broken :(

```java
public class MyClass {
    public Boolean equals(Object other) {
        System.debug('Called my equals');
        return false;
    }
}

MyClass m = new MyClass();
Object o = new MyClass();

m.equals('a'); // > 'Called my equals'
o.equals('a'); // System.debug is never called :(
```

Source: [Aidan Harding](https://salesforce.stackexchange.com/questions/339160/why-doesnt-equals-behave-like-a-virtual-method)

### Shadowing System (global) classes

Nothing prevents you from recreating a class with the same name of one that exists in the `System` (default) namespace.

```java
public class Database {
  public static List<sObject> query(String qry) {
    System.debug(qry);
    return null;
  }
}
```

Running `Database.query('foo')` will call our new class (essentially overriding the [Database methods](https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_dynamic_soql.htm))!?

The same principle also applies to standard SObjects:

```java
public class Account { }

Account acc = new Account();
acc.AccountNumber = '123'; // ! Variable does not exist: AccountNumber
```

Source: [Daniel Ballinger](https://twitter.com/FishOfPrey/status/1013530412121915392)

### "Phantom" Inner Class Type Equivalency

```java
public class IExist{}
System.assertEquals(IExist.class, IExist.IDont.Class); // -> passes
```

Source: [Kevin Jones](https://twitter.com/nawforce/status/1154135982280597504)

### List<Id> `contains` & `indexOf` is broken

#### "Apex Log level" influences behavior

The easiest way to test this is by writing the output to an object field.  

1. Replace `accId` with a dummy account
2. Open a dev console and set the log levels to `Apex=Finest`
3. Run the following code

```java
Id accId = '0012F00000YIc48QAD';
Account acc = new Account(Id = accId);
List<Id> haystack = new List<Id>();
haystack.add('0012F00000YIc46QAD');
haystack.add(accId);
haystack.add('0012F00000YIc49QAD');
String debug = 'Index: ' + haystack.indexOf(needle) + ' Contains: '   + haystack.contains(needle);
acc.AccountNumber = debug;
update acc;
```

4. Open the account. You should see the `AccountNumber` equals `Index: 1 Contains: true`
5. Set log levels to `Apex=None`
6. Run the code again
7. Refresh the account. `AccountNumber` will now equal `Index: -1 Contains: false`

Apparently this is a [known issue and it has been fixed](https://success.salesforce.com/issues_view?id=a1p3A000000AT9cQAG).  This test shows otherwise...

#### 15 Char Id's don't work

Behind the scenes Salesforce seems to always convert 15 character Id's to 18.  

Equivalency works as expected in most cases:

```java
Id a15 = '0012F00000YIc48';
Id a18 = '0012F00000YIc48QAD';
System.assert(a15 == a18);
```

However, for the List `contains` & `indexOf` methods, it doesn't:

```java
List<Id> idList = new List<Id>{
   '0012F00000YIc46',
   '0012F00000YIc48',
   '0012F00000YIc49'
};
System.debug(idList); //-> (0012F00000YIc46QAD, 0012F00000YIc48QAD, 0012F00000YIc49QAD)
System.debug(idList.indexOf('0012F00000YIc48')); // > -1
System.debug(idList.contains('0012F00000YIc48')); // > false
```
You can avoid this by first assigning the value you are checking to an `Id` type.

### `final` parameters "exist", but can be reassigned

You won't find a reference to it in the docs, but the compiler does apparently allow [`final` parameters](https://stackoverflow.com/questions/2236599/final-keyword-in-method-parameters). However, it doesn't actually to prevent reassignment of such parameters:

```java
public void withFinalParam(final String iAmFinal){
  iAmFinal = 'just kidding'; //compiles
}
```

Source: [Kevin Jones](https://twitter.com/nawforce/status/1251425747320934400)

### Fulfilling Interface Contracts with Static Methods

This shouldn't work but it does. Apparently also works with batch.

```java
public class StaticsAreCool implements Schedulable{
   public static void execute(SchedulableContext sc){
   }
}
```

Source: [Kevin Jones](https://twitter.com/nawforce/status/1195419601737265152)

### Exceptions are "exceptional"

In their naming conventions:

```java
public class OhGodBees extends Exception{}
// ~ Classes extending Exception must have a name ending in 'Exception'
```

and their Constructors:

```java
public class BeesException extends Exception{
    public BeesException(){}
}
// ~ System exception constructor already defined: <Constructor>()
```

For explanation and further interesting observations, [see Chris Peterson's blog post.](https://www.ca-peterson.com/2015/01/23/leaky_abstractions_apex_exception_types/)

### `System` can have ambiguous return types

`Database.query` is one of many cases where the Salesforce `System` namespace doesn't play by its own rules. It can return either a `List<SObject>` or a single `SObject`. No casting required.

```java
Foo__c foo = Database.Query('SELECT Id FROM Foo__c');
List<Foo__c> foos = Database.Query('SELECT Id FROM Foo__c');
```

Try writing your own method to do this and you'll get an error:

> Method already defined: query SObject Database.query(String) from the type Database (7:27)

You can overload arguments, but not `return` type.

### Odd List Initialization bug

The initialization syntax for `List<T>` expects `T ... args`.

So obviously, if you passed a `List<T>` into it, you will get compile error:

```java
List<Task>{new List<Task>()}; // ~ Initial expression is of incorrect type, expected: Task but was: List<Task>
```

Except, if List comes from `new Map<Id,T>().values()`...

The following code compiles without issue!

```java
new List<Task>{new Map<Id,Task>().values()};
```

To add to the perplexity, when executed you will receive the following runtime error:

> System.QueryException: List has no rows for assignment to SObject

Source: Matt Bingham

### Local Scope Leak

If you write an If/Else without braces, symbols scoped in the "if" seem to leak into the "else":

```java
if(false)
   String a = 'Never going to happen.';
else
   a = 'I should not compile';
```

Worth noting that Java won't even allow you to declare a block scoped variable inside a "braceless IF" as it can never be referenced elsewhere.

Source: [Kevin Jones](https://twitter.com/nawforce/status/1180936132491657224)

### Broken type inference for `Set<>`

Let's take a look at the standard `Set` class...

It can be iterated in a foreach loop:

```java
Set<String> mySet = new Set<String>{'a', 'b'};
for(String s : mySet){}
```

But, according to Salesforce (compiler & runtime), it does not actually implement the `Iterable` interface:

```java
String.join(mySet, ','); // ~ "Method does not exist or incorrect signature: void join(Set<String>, String)..."

// Just to make sure, lets check at runtime..
System.debug(mySet instanceof Iterable<String>);  // > false
```

Except... It actually does:

```java
String.join((Iterable<String>) mySet, ','); // this works!?
```

[Vote to fix this](https://success.salesforce.com/ideaView?id=08730000000kxLyAAI)

### String.Format with single quotes

```java
String who = 'World';
String msg = String.format(
     'Hello, \'{0}\'',
     new List<String>{who}
);
System.assert(msg.contains(who)); // ! assertion failed
```

Unexpectedly, `msg` is set to `Hello, {0}`

🤔

To get this to work properly you must escape _two_ single quotes:

```java
String who = 'World';
String msg = String.format(
     'Hello, \'\'{0}\'\'',
     new List<String>{who}
);
System.assert(msg.contains(who)); // -> passes
```

[Explanation by Daniel Ballinger](https://developer.salesforce.com/forums/?id=906F00000008yzsIAA)

### Line continuation breaks for static method

In apex, all statements must be terminated by a `;`. This allows statements to span multiple lines:

```java
Order o = new OrderBuilder()
  .addLineItem('foo', 5)
  .addLineItem('bar', 10)
  .setDiscount(0.5)
  .toOrder();
```

However, for some reason if the method is static, apex doesn't let it span a newline:

```java
Order o = OrderBuilder
  .newInstance() // ~ Variable does not exist: OrderBuilder
  .addLineItem('foo', 5)
  .addLineItem('bar', 10)
  .setDiscount(0.5)
  .toOrder();
```

Source: [Leo Alves](https://twitter.com/leofilipealves)

### Fun with Hashcodes

[Enums in batch](https://salesforce.stackexchange.com/questions/158557/enums-as-map-keys-dont-work-in-batchable)

[Objects in hashcodes](https://salesforce.stackexchange.com/questions/41741/map-set-size-when-sobjects-are-duplicated/41743#41743)

### JSON Serialization

1. There's no way to control automatic serialization of object properties (like `[JsonProperty(PropertyName = "FooBar")]` in C#)
2. There are reserved keywords that you can't use as property names.

Meaning the following cannot be parsed or generated using `JSON.deserialize` or `JSON.serialize`:

```json
{
  "msg": "hello dingus",
  "from": "Dr. Dingus"
}
```

[Work-around](https://salesforce.stackexchange.com/questions/2276/how-do-you-deserialize-json-properties-that-are-reserved-words-in-apex)

### Generics (parameterized interfaces) exist, but you can't use them

Once upon a time, generics were actually part of Apex. However, they have since been removed (with the exception of system classes (`List<>`, `Batchable<>`, etc).

Why would you want generics when your OS has perfectly good Copy & Paste functionality built right into it?

[Vote for Generics](https://success.salesforce.com/ideaView?id=08730000000aDnYAAU)

### Polymorphic Primitives

```apex
Object x = 42;
System.debug(x instanceOf Integer); // > true
System.debug(x instanceOf Long);    // > true
System.debug(x instanceOf Double);  // > true
System.debug(x instanceOf Decimal); // > true
```

Source: [Daniel Ballinger](https://twitter.com/FishOfPrey/status/1051965154454265856)

### Invalid HTTP method: PATCH

When you try this:

```java
Http h = new Http();
HttpRequest req = new HttpRequest();
req.setEndpoint('hooli.com');
req.setMethod('PATCH');
HttpResponse res = h.send(req); // ! Invalid HTTP method: PATCH
```

[There is a workaround](https://salesforce.stackexchange.com/questions/57215/how-can-i-make-a-patch-http-callout-from-apex), but only supported by some servers.

### Integers cannot be assigned the minimum Integer value

Integers have a minimum possible value of -2147483648, but this value cannot be directly assigned to an Integer.

```Integer totallyLegalNumber = -2147483648; // ! Illegal integer
```

The miscarriage of justice above can be avoided with the following workaround that clearly demonstrates the legality of the number:

```Integer totallyLegalNumber = -2147483647;
totallyLegalNumber--;
System.debug(totallyLegalNumber); // > -2147483648
```

Source: [Mehdi Maujood](https://www.apexsandbox.io/problem/68)

## 🔧 Since Fixed

Thankfully, these WTF's have since been fixed by Salesforce. We'll keep them documented for historical purposes (and entertainment).

### Mutating Datetimes

https://twitter.com/FishOfPrey/status/869381316105588736

### More hashcode fun

https://twitter.com/FishOfPrey/status/1016821563675459585

https://salesforce.stackexchange.com/questions/224490/bug-in-list-contains-for-id-data-type

### Initializing Abstract Classes

Resolved in Spring '20 by the "_Restrict Reflective Access to Non-Global Controller Constructors in Packages_" Critical Update

```java
public abstract class ImAbstract {
    public String foo;
}

ImAbstract orAmI = (ImAbstract) JSON.deserialize('{"foo":"bar"}', ImAbstract.class);
System.debug(orAmI.foo); // > bar
```

See [Stack Exchange Post](https://salesforce.stackexchange.com/questions/250184/can-create-an-instance-of-abstract-class-salesforce-bug?atw=1)

# What the f*ck, Apex?

> A list of funny and tricky Apex examples

***Inspired by [wtfjs](https://github.com/denysdovhan/wtfjs)***

### When a boolean is not a boolean

``` apex
Boolean b;
if(b!=true) system.debug('b is not true');
if(b!=false) system.debug('b is not false');
if(b){} //throws NPE exception
```

See [Advanced Apex Programming in Salesforce](http://advancedapex.com/2012/08/23/funwithbooleans/) for explanation.


### String compare is case-insensitive (except when it's not) 

``` apex
String x = 'Abc';
String y = 'abc';
System.assert(x == y); // passes
System.assertEquals(x, y); // fails
```
See [explanation on StackExchange](https://salesforce.stackexchange.com/questions/80456/is-there-any-difference-in-equals-and-for-string-variables)

### Shadowing System (global) classes

Nothing prevents you from recreating a class with the same name of one that exists in the `System` (default) namespace.

``` apex
public class Database {
    public static List<sObject> query(String qry){
        System.debug(qry);
        return null;
    }
}
```

Running `Database.query('foo')` will call our new class (essentially overriding the [Database methods](https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_dynamic_soql.htm))!?

The same principle also applies to standard SObjects: 

``` java
public class Account { }

Account acc = new Account();
acc.AccountNumber = '123'; // Variable does not exist: AccountNumber
```


Source: [Daniel Ballinger](https://twitter.com/FishOfPrey/status/1013530412121915392)

### "Phantom" Inner Class Type Equivalency 

```java
public class IExist{}
System.assertEquals(IExist.class, IExist.IDont.Class); //passes
```

Source: [Kevin Jones](https://twitter.com/nawforce/status/1154135982280597504)

### Fulfilling Interface Contracts with Static Methods

This shouldn't work but it does.  Apperently also works with batch.

``` java
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
//fails: OhGodBees: Classes extending Exception must have a name ending in 'Exception'
```

and their Constructors:

```java
public class BeesException extends Exception{
    public BeesException(){}
}
//fails: System exception constructor already defined: <Constructor>()
```

For explanation and further interesting observations, [see Chris Peterson's blog post.](https://www.ca-peterson.com/2015/01/23/leaky_abstractions_apex_exception_types/)

### `System` can have ambiguous return types

`Database.query` is one of many cases where the Salesforce `System` namespace doesn't play by its own rules. It can return either a `List<SObject>` or a single `SObject`.  No casting required.

```java
Foo__c foo = Database.Query('SELECT Id FROM Foo__c');
List<Foo__c> foos = Database.Query('SELECT Id FROM Foo__c');
```

Try writing your own method to do this and you'll get an error: 

> Method already defined: query SObject Database.query(String) from the type Database (7:27)

You can overload arguments, but not `return` type.

### Odd List Initialization bug

The initialization sytax for `List<T>` expects `T ... args`.

So obviously, if you passed a `List<T>` into it, you will get compile error:

``` java
List<Task>{new List<Task>()}; // Initial expression is of incorrect type, expected: Task but was: List<Task>
``` 

Except, if List comes from `new Map<Id,T>().values()`... 

The following code compiles without issue!

``` java
new List<Task>{new Map<Id,Task>().values()};
```

To add to the perplexity, when executed you will recieve the following runtime error:

> System.QueryException: List has no rows for assignment to SObject

Source: Matt Bingham 

### Local Scope Leak

If you write an If/Else without braces, symbols scoped in the "if" seem to leak into the "else":

``` java
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

``` java
Set<String> mySet = new Set<String>{'a', 'b'};
for(String s : mySet){}
```

But, according to Salesforce (compiler & runtime), it does not actually implement the `Iterable` interface:

``` java
String.join(mySet, ','); // Doesn't compile! "Method does not exist or incorrect signature: void join(Set<String>, String)..."

// Just to make sure, lets check at runtime..
System.assert(mySet instanceof Iterable<String>);  // Yup, this fails! I guess Set really isn't an Iterable...
```

Except... It actually does:

``` java
String.join((Iterable<String>) mySet, ','); // this works!?
```

[Vote to fix this](https://success.salesforce.com/ideaView?id=08730000000kxLyAAI)

### String.Format with single quotes


``` java
String who = 'World';
String msg = String.format(
     'Hello, \'{0}\'',
     new List<String>{who}
);
System.assert(msg.contains(who)); //fails
```

This unexpectedly outputs `Hello, {0}`

🤔

To get this to work properly you must escape *two* single quotes:

``` java
String who = 'World';
String msg = String.format(
     'Hello, \'\'{0}\'\'',
     new List<String>{who}
);
System.assert(msg.contains(who)); //passes
```

[Explanation by Daniel Ballinger](https://developer.salesforce.com/forums/?id=906F00000008yzsIAA)

### Line continuation breaks for static method

In apex, all statements must be terminated by a `;`.  This allows statements to span multiple lines:

``` java
Order o = new OrderBuilder()
  .addLineItem('foo', 5)
  .addLineItem('bar', 10)
  .setDiscount(0.5)
  .toOrder();
```

However, for some reason if the method is static, apex doesn't let it span a newline:

``` java
Order o = OrderBuilder
  .newInstance() //static
  .addLineItem('foo', 5)
  .addLineItem('bar', 10)
  .setDiscount(0.5)
  .toOrder();
```
 
This code will error out with the following message:

> Variable does not exist: OrderBuilder

Source: [Leo Alves](https://twitter.com/leofilipealves)

### Fun with Hashcodes

[Enums in batch](https://salesforce.stackexchange.com/questions/158557/enums-as-map-keys-dont-work-in-batchable)

[Objects in hashcodes](https://salesforce.stackexchange.com/questions/41741/map-set-size-when-sobjects-are-duplicated/41743#41743)

### JSON Serialization

1. There's no way to control automatic serialization of object properties (like `[JsonProperty(PropertyName = "FooBar")]` in C#)
2. There are reserved keywords that you can't use as property names.

Meaning the following cannot be parsed or generated using `JSON.deserialize` or `JSON.serialize`:

``` json
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

### Polymorphic Primatives

``` apex
Object x = 42;
System.debug(x instanceOf Integer); // true
System.debug(x instanceOf Long);  // true
System.debug(x instanceOf Double);  // true
System.debug(x instanceOf Decimal); // true
```

Source: [Daniel Ballinger](https://twitter.com/FishOfPrey/status/1051965154454265856)

### Invalid HTTP method: PATCH

When you try this:

``` java
Http h = new Http();
HttpRequest req = new HttpRequest();
req.setEndpoint('hooli.com');
req.setMethod('PATCH');
HttpResponse res = h.send(req);
```

[There is a workaround](https://salesforce.stackexchange.com/questions/57215/how-can-i-make-a-patch-http-callout-from-apex), but only supported by some servers.

# Since Fixed

Thankfully, these WTFs have since been fixed by Salesforce.  We'll keep them documented for historical purposes (and entertainment).

### Mutating Datetimes

https://twitter.com/FishOfPrey/status/869381316105588736

### More hashcode fun

https://twitter.com/FishOfPrey/status/1016821563675459585

https://salesforce.stackexchange.com/questions/224490/bug-in-list-contains-for-id-data-type

### Initializing Abstract Classes

Resolved in Spring '20 by the "_Restrict Reflective Access to Non-Global Controller Constructors in Packages_" Critical Update

``` apex
public abstract class ImAbstract {
    public String foo;
}

ImAbstract orAmI = (ImAbstract) JSON.deserialize('{"foo":"bar"}', ImAbstract.class);
System.debug(orAmI.foo);
```

See [Stack Exchange Post](https://salesforce.stackexchange.com/questions/250184/can-create-an-instance-of-abstract-class-salesforce-bug?atw=1)

# What the f*ck, Apex?

> A list of funny and tricky Apex examples

Inspired by [wtfjs](https://github.com/denysdovhan/wtfjs)

### When a boolean is not a boolean

``` apex
Boolean b;
if(b!=true) system.debug('b is not true');
if(b!=false) system.debug('b is not false');
```

See [Advanced Apex Programming in Salesforce](http://advancedapex.com/2012/08/23/funwithbooleans/) for explanation.


### String compare is case-insensitive (except when it's not) 

``` apex
String x = 'Abc';
String y = 'abc';
System.assert(x == y); // passes
System.assertEquals(x, y); // fails
```
[Explanation](https://salesforce.stackexchange.com/questions/80456/is-there-any-difference-in-equals-and-for-string-variables)

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

Running `Database.query('foo')` will call our new class (essentially override the [Database methods](https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_dynamic_soql.htm))!?


[source](https://twitter.com/FishOfPrey/status/1013530412121915392)

### "Phantom" Inner Class Type Equivalency 

```java
public class IExist{}
System.assertEquals(IExist.class, IExist.IDont.Class); //passes
```

Source: [Kevin Jones](https://twitter.com/nawforce/status/1154135982280597504)

### Exceptions are "exceptional"

In their naming conventions:

```java
public class OhGodBees extends Exception{}
//fails: OhGodBees: Classes extending Exception must have a name ending in 'Exception'
```

Constructors:

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

Apparently, once upon a time, generics were part of Apex. However, they have since been removed (with the exception of system classes (`List<>`, `Batchable<>`, etc).  

Why would you want generics when your OS has perfectly good Copy & Paste functionality built right into it?

[Vote for Generics](https://success.salesforce.com/ideaView?id=08730000000aDnYAAU)

### Initializing Abstract Classes

``` apex
public abstract class ImAbstract {
    public String foo;
}

ImAbstract orAmI = (ImAbstract) JSON.deserialize('{"foo":"bar"}', ImAbstract.class);
System.debug(orAmI.foo);
```

[source](https://salesforce.stackexchange.com/questions/250184/can-create-an-instance-of-abstract-class-salesforce-bug?atw=1)

### Polymorphic Primatives

``` apex
Object x = 42;
System.debug(x instanceOf Integer); // true
System.debug(x instanceOf Long);  // true
System.debug(x instanceOf Double);  // true
System.debug(x instanceOf Decimal); // true
```

Source [Daniel Ballinger](https://twitter.com/FishOfPrey/status/1051965154454265856)


## Since Fixed

Thankfully these WTFs have since been fixed by Salesforce.  We'll keep them documented for historical purposes (and entertainment).

### Mutating Datetimes

https://twitter.com/FishOfPrey/status/869381316105588736

### More hashcode fun
https://twitter.com/FishOfPrey/status/1016821563675459585

https://salesforce.stackexchange.com/questions/224490/bug-in-list-contains-for-id-data-type

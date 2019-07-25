# What the f*ck Apex?

> A list of funny and tricky Apex examples

Inspired by [wtfjs](https://github.com/denysdovhan/wtfjs)

### When a boolean is not a boolean

``` apex
Boolean b;
if(b!=true) system.debug(‘b is not true’);
if(b!=false) system.debug(‘b is not false’);
```

See [Advanced Apex Programming in Salesforce](http://advancedapex.com/2012/08/23/funwithbooleans/) for explination.


### String compare is Case-insenstive (except when it's not) 

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

Running `Database.query('foo')` will call our new class (essentially override the [Database methods](https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_dynamic_soql.htm)!?)


[source](https://twitter.com/FishOfPrey/status/1013530412121915392)

### "Phantom" Inner Class Type Equivalency 

```java
public class IExist{}
System.assertEquals(IExist.class, IExist.IDont.Class); //passes
```

Source: [Kevin Jones](https://twitter.com/nawforce/status/1154135982280597504)

### Exception are "exceptional"

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

And Overrides:

```java
public class BeesException extends Exception{
    public void setMessage(String message){
        super.setMessage(message);
    }
}
//fails: BeesException: Method must use the override keyword: void setMessage(String)


public class BeesException extends Exception{
    public override void setMessage(String message){
        super.setMessage(message);
    }
}
//fails: Object has no superclass for super invocation
```

[For Explination see](https://www.ca-peterson.com/2015/01/23/leaky_abstractions_apex_exception_types/)

### System can have ambiguous returns types

`Database.query` is one of many cases where the salesforce "System" namespace doesn't play by it's own rules.   It can either return a `List<SObject>` or a single `SObject`.  No Casting required.

```java
Foo__c foo = Database.Query('SELECT Id FROM Foo__c');
List<Foo__c> foos = Database.Query('SELECT Id FROM Foo__c');
```

Try writing your own method to do this and you'll get a error: 

> Method already defined: query SObject Database.query(String) from the type Database (7:27)

You can overload arguments, but not `return` type.

### Fun with Hashcodes

[Enums in batch](https://salesforce.stackexchange.com/questions/158557/enums-as-map-keys-dont-work-in-batchable)

[Objects in hascodes](https://salesforce.stackexchange.com/questions/41741/map-set-size-when-sobjects-are-duplicated/41743#41743)

### JSON Serialization

1. There way to control automatic serialization of object properties (like `[JsonProperty(PropertyName = "FooBar")]` in C#)
2. There are reserved keywords that you can't use as property names.

Meaning the following cannot be parsed or generated using `JSON.deserialize` or `JSON.serialize`:

``` json
{
   "msg": "hello dingus",
   "from": "Dr. Dingus"
}
```

[Work Around](https://salesforce.stackexchange.com/questions/2276/how-do-you-deserialize-json-properties-that-are-reserved-words-in-apex)

### Generics (parametrized interfaces) exist but you can't use them

Apperently once upon a time, generics were part of apex.  However, they have since been removed (with the exception of system classes (`List<>`, `Batchable<>`, etc).  

Why would you want generics when you're OS has perfectly good Copy & Paste functionality built right into it?

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

Source [Dainel Barrileger](https://twitter.com/FishOfPrey/status/1051965154454265856)


## Since Fixed

Thankfully these WTF's have since been fixed by Salesforce.  We'll keep them documented for historical purposes (and entertainment).

### Mutating Dates

https://twitter.com/FishOfPrey/status/869381316105588736

### More hashcode fun
https://twitter.com/FishOfPrey/status/1016821563675459585
https://salesforce.stackexchange.com/questions/224490/bug-in-list-contains-for-id-data-type

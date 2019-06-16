# What the f*ck Apex?
A list of things that make Salesforce Apex developers hate their lives.

Inspired by (wtfjs)[https://github.com/denysdovhan/wtfjs]

## When a boolean is not a boolean

``` apex
Boolean b;
if(b!=true) system.debug(‘b is not true’);
if(b!=false) system.debug(‘b is not false’);
```

See (Advanced Apex Programming in Salesforce)[http://advancedapex.com/2012/08/23/funwithbooleans/] for explination.


## Shadowing System (global) classes

Nothing prevents you from recreating a class with the same name of one that exists in the `System` (default) namespace.

(source)[https://twitter.com/FishOfPrey/status/1013530412121915392]

## Fun with Hashcodes

https://twitter.com/FishOfPrey/status/1016821563675459585 (bug that has since been fixed)[https://salesforce.stackexchange.com/questions/224490/bug-in-list-contains-for-id-data-type]

[Enums in batch](https://salesforce.stackexchange.com/questions/158557/enums-as-map-keys-dont-work-in-batchable)

(Objects in hascodes)[https://salesforce.stackexchange.com/questions/41741/map-set-size-when-sobjects-are-duplicated/41743#41743]

## JSON Serialization

1. There way to control automatic serialization of object properties (like `[JsonProperty(PropertyName = "FooBar")]` in C#)
2. There are reserved keywords that you can't use as property names.

Meaning the following cannot be parsed or generated using `JSON.deserialize` or `JSON.serialize`.

### Work Arounds
- https://salesforce.stackexchange.com/questions/2276/how-do-you-deserialize-json-properties-that-are-reserved-words-in-apex


## Generics (parametrized interfaces) existe but you can't use them

Apperently once upon a time, generics were part of apex.  However, they have since been removed (with the exception of system classes (`List<>`, `Batchable<>`, etc).  

Why would you want generics when you're OS has perfectly good Copy & Paste functionality built right into it?

(Vote for Generics)[https://success.salesforce.com/ideaView?id=08730000000aDnYAAU]

## Initializing Abstract Classes

https://salesforce.stackexchange.com/questions/250184/can-create-an-instance-of-abstract-class-salesforce-bug?atw=1

## Polymorphic Primatives

``` apex
Object x = 42;
System.debug(x instanceOf Integer); // true
System.debug(x instanceOf Long);  // true
System.debug(x instanceOf Double);  // true
System.debug(x instanceOf Decimal); // true
```

Source (Dainel Barrileger)[https://twitter.com/FishOfPrey/status/1051965154454265856]

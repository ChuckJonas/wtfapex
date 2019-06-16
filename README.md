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

## Initlizing Abstract Classes

https://salesforce.stackexchange.com/questions/250184/can-create-an-instance-of-abstract-class-salesforce-bug?atw=1


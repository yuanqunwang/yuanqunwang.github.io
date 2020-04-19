# Lambda Expression

## Lambda

## Functional Interface

* a hint for lambda expression
* ensure only one effectively abstract method

## Target Typing

### Context for target typing

* method call
* assignment
* constructor

### rule for target typing

between the context and the lambda expression (method reference)

* method parameters type should same
* method parameters number should same
* method return type should be compatible
* exception throw by method should be compatible

## Lexical scoping

* `this` refers to enclosing instance (unlike inner class refers to inner class itself)
* any method call refers to enclosing instance method

## Method reference

method reference is a effectively lambda expression.

* static method reference
* instance method reference
* instance method reference of an object of particular type
* constructor method reference: `Myclass::new`
* array constructor method reference: `Myclass[]::new`

## Stream

## create stream

## intermediate operation & terminal operation

* intermediate operations are always lazy: `map`, `filter`
* terminal operations are always eager: `forEach` `sum`, `max`, `reduce`, `collect`
* intermediate operations chained with terminal operations forms stream pipeline.
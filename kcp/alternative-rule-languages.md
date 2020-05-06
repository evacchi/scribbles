# Example: Annontated Rule Definitions

We propose special annotations to decorate methods in a traditional JVM language. 
These annotations will contain rule constraints; the body of the methods
will contain the consequence. For instance, in Java:

```java
public class AnnotatedUnit implements RuleUnitMemory {
	private final DataStore<Person> persons = DataSource.createStore();

	public void adultRule(
	        @When("/persons[ age >= 18 ]") Person adult,
	        @When("/persons[ age != 0 ]") Person anotherBinding) {

	    System.out.printf("%s is an adult\n", adult.getName());
	}
	...
```

The same technique can be applied to different programming languages
(e.g. Kotlin, Scala, etc.) thereby allowing for _polyglot_ rule definitions.

## Motivation

Today, Kogito only supports DRL. It is not possible to use a different
programming language apart from the Java dialect in the consequences, 
nor is it possible to use plain Java to express rules, but only DRL files.

## Goal

The goal is to realize a mechanism to plug different language implementations,
into the rule engine, while, at the same time, keep the burden low for maintainers
of the platform.

Annotated methods relieve developers from the burden of supporting yet-another
language implementation, and let the users pick a language of their choosing
to write rules.

## Implementation

The initial goal of this proposal will be to realize a throwaway PoC for the feature.

1. An annotation processor pre-processes a Java class and generates a DRL file

	```java
	public class AnnotatedUnit implements RuleUnitMemory {
		private final DataStore<Person> persons = DataSource.createStore();

		public void adultRule(
		        @When("/persons[ age >= 18 ]") Person adult,
		        @When("/persons[ age != 0 ]") Person anotherBinding) {

		    System.out.printf("%s is an adult\n", adult.getName());
		}
		...
	```

2. The DRL file contains one rule per annotated method;

	- the method name is the rule name
	- each annotated parameter is a condition
	- the parameter is the binding for that condition  
	- the RHS of the rule **delegates** to the method,
      passing the matched variables

	e.g., consider the generated code for the rule above:

	```java
	package org.kie.kogito.queries;
	unit org.kie.kogito.queries.AnnotatedUnit;
	rule adultRule when
	  adult : /persons[ age >= 18 ]
	  anotherBinding : /persons[ age != 1 ]
	then
	  unit.adultRule(adult, anotherBinding);
	end
	```

3. DRL is generated in a directory that is picked up by the usual
  code generation pipeline, which resumes work from there.


## Impact

### Pros

- it is always possible to compile this class file. 
- It is always possible to add a break point inside of the rule body!
- it is always possible to instantiate the class as a regular class 
  for testing purposes (just invoke them as plain methods)
- Polyglot rules: we only need to parse and validate the pattern, the rest is
  delegated to the host language compiler


### Cons

- we may need to think of a way to break up large rule bases 
in a language-meaningful way (many interfaces with default methods, 
mixing-in in a class? a delegate system? disallow it because we favor smaller 
rule bases, and usage of rule unit dispatch?)





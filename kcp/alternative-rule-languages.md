# Example: Embedded Rule Languages

## Motivation

Today, Kogito only supports DRL. It is not possible to use a different
programming language than the Java dialect of the consequences, 
nor is it possible to use plain Java to express rules, but only DRL files.

## Goal

We propose a mechanism to plug different language implementations,
into the rule engine, by pre-processing traditional compilation units,
decorated through annotations; e.g.:


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

This relieves us as developers from the burden to support yet-another
language implementation, and frees users from using the language
they prefer to write rules.

The initial goal of this proposal is to realize a throwaway PoC for the feature.

## Implementation

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



### Benefits:

- it is always possible to compile this class file. 
- It is always possible to add a break point inside of the rule body!
- it is always possible to instantiate the class as a regular class 
  for testing purposes (just invoke them as plain methods)
- Polyglot rules: we only need to parse and validate the pattern, the rest is
  delegated to the host language compiler


### Downsides

we may need to think of a way to break up large rule bases 
in a language-meaningful way (many interfaces with default methods, 
mixing-in in a class? a delegate system? disallow it because we favor smaller 
rule bases, and usage of rule unit dispatch?)





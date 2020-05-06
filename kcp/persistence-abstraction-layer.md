# Example: Persistence Abstraction Layer

## Motivation

Today, only IncredibleDB is allowed as a persistence backend. 
However, IncredibleDB has several limitations:

- it does not support non-volatile storage
- it is a document store that does not support SQL queries
- licensing is unclear

## Goal

We want to be able to plug different storage mechanisms
depending on varying requirements.

## Description

We propose a plug-in mechanism for the storage backend so that
different persistence implementations can be supported.

## Implementation

IncredibleDB is hard-coded in multiple places. We propose to:

- Create an **Abstraction Layer** to support separate storage backends
- **Refactor** the IncredibleDB implementation into one plug-in implementation
- Add a dummy implementation for testing

### Design of the Abstraction Layer

Because our current architecture is based around the IncredibleDB document store,
we will model the abstraction around a generic document store. 
New implementations should mimic the behavior of such a document store.

### Design of the Generic Document Store

....

## Impact

## Pros

- more storage backends can be supported

## Cons

- additional maintainance cost
- we need to get the abstraction right or we end up with least common denominator

# Writing Procedural Specs
### ...using Bikeshed
### ...for the Web Platform

Author: garykac@ (relying heavily on the expertise of domenic@, jakearchibald@, jsbell@, ...)  
Status: Draft

## About

A simple overview of common structures in algorithmic web standard specifications.
The examples are all presented in Bikeshed.

NOTE: This document is mostly a collection of notes that I started making while wading into the procedural
spec writing process. It does not currently cover everything, but rather only those things that I've bumped into so far.
It's written as a set of recommendations, even though this is not official documentation.

## Sequential algorithm steps

All algorithm steps are implicitly sequential and each step must begin with `1.` so that they are auto-numbered.

```
	1. step-1
	1. step-2
	1. step-3
```

Note that Bikeshed doesn't require that the lines begin with `1.` -- any number followed by a `.` will work.
Some spec editors prefer to explicitly number the items 1, 2, 3 to make the Bikeshed source document easier
to read, but the recommendation here is to use `1.` everywhere to make it easier to insert new steps at the
beginning or middle of the algorithm.

## Concept Types and Objects

Types and objects that are required by the algorithm should be described outside the algorithm and should be
presented as a definition, for example:

```
	The user agent has a <dfn>really useful object</dfn> that ...
```

Note that the definition is a short, readable English phrase rather than a variable name (so no camelCase
et al.). These lines are not numbered because they are not part of an algorithm.

References to this type or object should be wrapped in `[=` and `=]` so that they are auto-linked by Bikeshed.

```
	1. If the [=really useful object=] is null, then …
```

_Note: You could also wrap types/objects with a simple `<a>` tag and Bikeshed will auto-link that as well,
but the `[=`square bracket`=]` notation is easier to read and allows concise use of `[=for/term=]` for
scoped definitions._

### Scoping Definitions

If you are defining a generic term, you may need to scope it to disambiguate it from other uses of the same
term (which may occur if another specification exports that term). Consider the following 2 definitions for
"resolution":

```
	A [=television=] has a <dfn>resolution</dfn> that …
	Each completed [=action=] has a <dfn>resolution</dfn> that …
```

These definitions can be scoped using the "for" attribute:

```
	A [=television=] has a <dfn for=television>resolution</dfn> that …
	Each completed [=action=] has a <dfn for=action>resolution</dfn> that …
```

This allows the definitions to be explicitly referenced using the scope and the term:

```
	1. If the [=television/resolution=] is "1080p", then …
	1. If the [=action/resolution=] is "fail", then …
```

Note that if you are adding multiple definitions for the same object, you can set a default scope as follows:

```
	<div dfn-for=television>
		A [=television=] has a <dfn>current channel</dfn> …
		A [=television=] has a <dfn>resolution</dfn> …
	</div>
```

## Internal Slots

In IDL you define attributes, but in a procedural spec you should refer to "internal slot"s that hold
the raw values associated with those attributes.

As an example, given the following IDL:

```
	Interface Television {
		attribute unsigned short channel;
		readonly attribute boolean supportsHighDefinition;
	};
```

A procedural spec would describe this by (1) **defining an "internal slot"** for the channel value:

```
A [=television=] has a <dfn for=television>current channel</dfn> which is a positive integer, initially 3.
```

(2) specifying that the IDL attribute's **getter** should return the value in the internal slot:

```
The <dfn attribute for=Television>channel</dfn> IDL attribute's getter must return the television object’s [=television/current channel=].
```

(3) specifying that the IDL attribute's **setter** should update the value in the internal slot (not for readonly attributes!):

```
The <dfn attribute for=Television>channel</dfn> IDL attribute's setter, given |value|, must set the television object’s [=television/current channel=] to |value|.
```

(4) making sure that all operations in the spec **refer to the internal slot** rather than the IDL attribute:

```
When the + button on the remote is pressed, increase [=television/current channel=] of the associated [=television=] by 1.
```

## Local variables

When a variable is introduced (usually at the start of the algorithm, but may occur in the middle), use "let".

```
1. Let |y| be false.
```

Update already created variables with "set".

```
1. Set |x| to null.
```

## Variable Types

Boolean:
* To create booleans: "Let |x| be a boolean initially set to true." (or false)
* To update booleans: "set" to "true" or "false"
* To check booleans: use "is true" or "is false"

_Note: Many specs use "flags" which you "set" and "clear" and check for being "set" or "not set". Newer specs are starting to use "boolean". Existing specs that use flags may be converted at some point._

## Returning a value

To return a value from the algorithm, use "return".

```
	1. Return |x|.
```

## If - Then - Else

Simple "if x then y" constructions can be placed on a single line.

```
1. If |x| is "some-value", set |y| to true.
```

The "else" clause (if present) must be a numbered sibling to the matching "if" clause.

```
1. If |x| is "some-value", set |y| to true.
1. Else set |y| to false.
```

TODO: Resolve "else" vs. "otherwise" (which is used in many specs).  Options include: (1) Require "else" and "else if"; (2) Require "otherwise" and "otherwise if"; (3) Allow both (but require that the spec must be consistent). Also, see Ternary Operator for another use of "otherwise".

Any number of "else-if" clauses can be added, each one numbered at the same level as the matching "if" (see Switch, below, for another option if you have many "else-if"s).

```
1. If |x| is "some-value", set |some-var| to true.
1. Else if |x| is "other-value", set |other-var| to true.
1. Else set |other-other-var| to false.
```

If multiple actions need to occur, then "substeps" can be specified.

```
1. If |x| is "some-value", run the following substeps:
    1. Set |some-var| to true.
    1. Set |some-other-var| to false.
1. Else if |x| is "other-value", run the following substeps:
    1. Set |some-var| to false.
    1. Set |some-other-var| to true.
```

Or simply:

```
1. If |x| is "some-value", then
    1. Set |some-var| to true.
```

## Switch

In some cases it may be more readable to list a set of choices rather than using if/else steps. 

```
1. Switch on |style| and run the associated substeps:
    <dl class=switch>
        <dt>"prose"</dt>
        <dd>
            1. Append the string "old and busted" to the output.
            1. Return false 
        </dd>

        <dt>"algorithmic"</dt>
        <dd>
            1. Append the string "new hotness" to the output.
            1. Return true
        </dd>

        <dt>Otherwise</dt>
        <dd>
            1. Throw TypeError.
        </dd>
    </dl>
```

The cases can be more complicated than simple values e.g. ranges, referencing lists, etc. In the extreme, conditions can be used as cases where this improves readability over long if/else if/else if steps, e.g.:

```
1. Jump to the appropriate steps below:
    <dl class=switch>
        <dt>If |var| is a list and length is greater than 1</dt>
        <dd>
            ... 
        </dd>

        <dt>If |var| is a list and length is 1</dt>
        <dd>
            …
        </dd>

        <dt>If |var| is a string</dt>
        <dd>
            …
        </dd>

        <dt>Otherwise</dt>
        <dd>
            …
        </dd>
    </dl>
```

_Note: In various spec stylesheets, the "switch" class generates ↪ (U+21AA RIGHTWARDS ARROW WITH HOOK) using CSS ::before content._

## Ternary Operator

The compact ternary operator:

```
	1. Set |x| to 1 if some-condition, or 2 otherwise.
```

## Conditions

Boolean conditions

```
1. If |var| is true, …
```

For conditions with complex data types, use "equals" and make sure it is linked to the text that defines equality for that data type. For example, for URLs:

```
	1. If |thisURL| [=url/equals=] |requestURL|, …
```

Note:` [=url/equals=]` is a Bikeshed shortcut for `<a for="url">equals</a>`.

## For Each

Simple "for each" constructions can be placed on a single line.

```
1. [=For each=] {{Entry}} |e| in |entry-list|, set |e|'s {{Entry/start}} to 0.
```

For more complex actions, use a colon and add indented substeps.

```
1. [=For each=] {{Entry}} |e| in |entry-list|:
    1. Set |e|'s {{Entry/start}} to 0.
    1. Set |e|'s {{Entry/end}} to 0.
```

To exit the loop prematurely, use "break"/"break out of this loop" (to stop the loop entirely by jumping to the end) or "continue"/"continue to the next item" (to stop this iteration and continue with the next item in the collection).

As a shortcut for:

```
1. [=For each=] |e| in |entry-list|:
    1. If |e|'s {{type}} is "t", set |e|'s {{start}} to 0.
```
you can use "for each … whose …":
```
1. [=For each=] |e| in |entry-list| whose {{type}} is "t", set |e|'s {{start}} to 0.
```

## Exiting

To exit the entire algorithm:

```
1. Abort these steps
```
or
```
	1. Return
```

To exit by throwing an error:

```
	1. If Type(|v|) is not an Object, [=throw=] a TypeError
```

The exit action can be appended to another action.

```
	1.  Set |x| to false and abort these steps.
```

## Asynchronous tasks

By default, all algorithms run on the main thread. To run part of the algorithm on a background thread, use the "in parallel" construction:

```
	1. Run these substeps [=in parallel=]:
	    1. step-1
	    1. step-2
```

This indicates that a separate task should be created on the background thread to run the substeps (step-1 & step-2) in order. This new background task is run in parallel with the steps on the main thread.

Note: In English, the construction "run these substeps in parallel: A, B, C" means "run A, B and C in parallel with each other". However, in specifications, it has the specific meaning of running A, B and C sequentially on a background thread (that runs in parallel with the main thread).

### Running on the Background Thread

When specifying steps that run on a background thread, there are some restrictions to be aware of:

* You cannot update values that are accessible from JS. Any action that needs to be observable from JS must be made on the main thread.

Background tasks can create other background tasks (using the "in parallel" construction), but to get back onto the main thread, you need to "queue a task".

```
1. [=Queue a task=] that runs the following substeps:
    1. step-1
    1. step-2
```

If you need to refer to this task elsewhere, it can be introduced as a local variable.

```
[=Queue a task=] |task| that runs the following substeps:
    1. step-1
    1. step-2
```

Elsewhere in the algorithm, to block on this task, use "wait".

```
Wait for |task| to have executed.
```

## Promises

The proper way to use Promises in specifications is covered in
[Writing Promise-Using Specifications](https://www.w3.org/2001/tag/doc/promises-guide).
This sections provides only a brief high-level summary.

To create a new promise:

```
1. Let |promise| be a new [=promise=].
```

To resolve the promise with a value.

```
1. Resolve |promise| with |some-value|. 

1. Resolve |promise| with undefined. 
```

To reject a promise, use "reject" with the error.

```
	1. Reject |promise| with an "AnError" DOMException.
```

## Defining Functions

Functions with explicit input and output can be written as follows:

```
	<dfn>is it really Santa</dfn>
	: Input
	::    |name|, a string
	::    |isJolly|, a boolean
	: Output
	::   |isSanta|, a boolean
	1. Let |isSanta| be false.
	1. If name is "Nick" and |isJolly| is true, set |isSanta| to true.
	1. Return |isSanta|.
```

Sometimes, the function name is written more formally using camelCase or InitialCaps. In this example, that could be something like <dfn>isReallySanta</dfn>.

If there are no input parameters to the function, the entire Input/Output block at the beginning is typically omitted.

If there is input, but no output, then use "None" for output.

```
	<dfn>do something</dfn>
	: Input
	::    |name|, a string
	: Output
	::   None
	1. Fire an event named "name" at the xyz element.
```


## Calling Functions
Functions without parameters are called as follows:

```
	1. Let |result| be the result of running [=FunctionName=].
```

Functions with parameters:

```
 	1. Let |result| be the result of running [=FunctionName=] with "Kringle" and true as the arguments.
```
or
```
	1. Let |result| be the result of running [=FunctionName=] given "Kringle" and true.
```
or (naming each argument):
```
  1. Let |result| be the result of running [=FunctionName=] with |name| as "Kringle" and |isJolly| as true.
```
  
## Global Objects

For some global Concept Objects, it is important to make sure that they are defined in the appropriate location to ensure that feature works as intended (and so that it is implemented in a compatible way in different implementations).

Here are some common locations considered for global objects:

## User Agent

This makes the object the same for the entire browser (across all tabs and web pages). Usually, this is not what you want.

## Browsing Context (aka, a "tab" in the browser)

Placing an object here makes it available to every page that is navigated to within that browsing context, even to cross-origin sites. This is also probably not where you want to place web platform objects.

## Window/Navigator

Window and Navigator have the same observable behavior, so the choice between them is arbitrary. However, it is often easier to write specs that refer to "the navigator's object xxx" (for Navigator) rather than "the navigator's object relevant global object's xxx" (for Window).

Consider the following scenarios:
* Inject into about:blank and then navigate: Objects **are** preserved.
* document.open(): Objects **are not** preserved (because the Window is re-created)

## Document

This gives the object to every document, including those created by document.implementation.createDocument() and the XHR's [response document](https://xhr.spec.whatwg.org/#response-document-object).

If your feature is user-facing, then you probably want to use "Document with a Browsing Context" so that the object is not present in those situations.

## Document with a Browsing Context

This gives the object to every document that is associated with a tab in the browser.

Consider the following scenarios:
* Inject into about:blank and then navigate: Objects **are not** preserved.
* document.open(): Objects **are** preserved

_TODO: What about global objects for Workers? (for APIs exposed to both Windows and Workers)_

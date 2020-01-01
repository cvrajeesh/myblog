title: Build Parsers using PEG.js
date: 2015-10-13 14:41:03
tags:
- JavaScript
- Misc
---

Writing parser from scratch is a tedious task. Recently I came across a simple parser generator for JavaScript called [PEG.js]

{% blockquote from official website http://pegjs.org/ %}
PEG.js is a simple parser generator for JavaScript that produces fast parsers with excellent error reporting. You can use it to process complex data or computer languages and build transformers, interpreters, compilers and other tools easily.
{% endblockquote %}

In order to create a parser, provide a grammar as the input and [PEG.js] will build a parser for that grammar. Then that generated parser can be used to parse the content.

[PEG.js] grammar has a particular syntax, you can learn more about it from http://pegjs.org/documentation#grammar-syntax-and-semantics. If you are familiar with regular expression it will easy understand.

Take a simple example for writing grammar which will parse texts like "$100", "$150" etc...

Example text follow a pattern where currency symbol i.e. "$" is followed by a number. Translating this into [pEG.js] grammar looks like this

```
Text
 = "$" Number

Number
 = Digit+

Digit
 = [0-9]
```
Every grammar has set of *rules*, in this case we have three rules. Every rule has a *name* that identify the rule and a parsing expression. Rule name is followed by "=" then parsing expression.

1. Rule **Text**
 	Parsing expression for this rule is *"$" Number*. This parsing expression says "Match text that has literal "$" followed by a *Number*". Note *Number* is name of a rule.
2. Rule **Number**
	Parsing expression "Digit+" means - match text that has one or more *Digit*.
3. Rule **Digit**
	Last rule says match any character that is between 0 to 9

Hope you got basic idea how a grammar is created. [PEG.js] provides playground where you can try your grammar and see how it is working. To try this go to http://pegjs.org/online and copy paste the grammar we have created on text box 1 and enter "$100" on text box 2. Parsed result will be shown on the *output* section and it looks like this

```json
[
   "$",
   [
      "1",
      "0",
      "0"
   ]
]
```
[PEG.js] was able to parse successfully, if you provide an invalid input like "$100$" it will show a parser error.

Note if you want to just get the value from that text ("100" from "$100"), the output that we got is not that useful. In order extract only the required values, parsing expression allows to write custom JavaScript expressions inside the curly braces ("{" and "}"). Updating *rule* **Text** like below will return just the value "100"

```
Text
 = "$" n:Number
 { return n.join(""); }
```
Two changes we have made to the previous *rule*
1. **n:**number - added a prefix ***n*** to *rule* **Number**
	Here we are assigning the match result of *rule* **Number** to a variable called ***n***
2. `{ return n.join(""); }`
	JavaScript statement inside the curly brace call `join` method on the variable ***n***. In this case variable ***n*** contains an array of characters, so calling `join` on that will concatenate and produce a string which returned as the result of *rule* **Text**

##Parsing Grocery List
Take some more complex example where you want to parse grocery list and return the result in a JSON format like below where quantity should be in grams

```json
[
 { "name": "Onion", "quantity": 1000 },
 { "name": "Tomatoes", "quantity": 500 }
]
```
Grocery list text will be in the following format - where the quantity can be in *kg* or *gm*

```
Onion 1kg
Tomatoes 500gm
Beans 1kg
```
Please try for yourself and share your grammar.

Here is the grammar I have created to do this; Below JsFiddle is interactive you can change grammar and grocery list, clicking on **Parse** button will show the result.

{% jsfiddle 1u0ug5wd result light 100% 675 %}

Hope this article was helpful. Please share your thoughts and comments.

[PEG.js]: http://pegjs.org/

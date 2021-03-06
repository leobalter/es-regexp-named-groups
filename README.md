# RegExp Named Capture Groups

## Introduction

Numbered capture groups allow one to refer to certain portions of a string that a regular expression matches. Each capture group is assigned a unique number and can be referenced using that number, but this can make a regular expression hard to grasp and refactor.

For example, given `/(\d{4})-(\d{2})-(\d{2})/` that matches a date, one cannot be sure which group corresponds to the month and which one is the day without examining the surrounding code. Also, if one wants to swap the order of the month and the day, the group references should also be updated.

Named capture groups provide a nice solution for these issues.

## High Level API

A capture group can be given a name using the `(?<name>...)` syntax, for any identifer `name`. The regular expression for a date then can be written as `/(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/`. Each name should be unique and follow the grammar for ECMAScript identifiers.

Named groups can be accessed from properties of the regular expression result. Numbered references to the groups are also created, just as for non-named groups. For example:

```js
let re = /(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/;
let result = re.exec('2015-01-02');
// result.year === '2015';
// result.month === '01';
// result.day === '02';

// result[0] === '2015-01-02';
// result[1] === '2015';
// result[2] === '01';
// result[3] === '02';
```

The interface interacts nicely with destructuring, as in the following example:

```js
let {one, two} = /^(?<one>.*):(?<two>.*)$/.exec('foo:bar');
console.log(`one: ${one}, two: ${two}`);  // prints one: foo, two: bar
```

### Backreferences

A named group can be accessed within a regular expression via the `\k<name>` construct. For example,

```js
let duplicate = /^(?<half>.*).\k<half>$/u;
duplicate.test('a*b'); // false
duplicate.test('a*a'); // true
```

Named references can also be used simultaneously with numbered references.

```js
let triplicate = /^(?<part>.*).\k<part>.\1$/u;
duplicate.test('a*a*a'); // true
duplicate.test('a*a*b'); // false
```

### Replacement targets

Named groups can be referenced from the replacement value passed to `String.prototype.replace` too. If the value is a string, named groups can be accessed using the `$<name>` syntax. For example:

 ```js
let re = /(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/;
let result = '2015-01-02'.replace(re, '$<day>/$<month>/$<year>');
// result === '02/01/2015'
```

Note that an ordinary string literal, not a template literal, is passed into `replace`, as that method will resolve the values of `day` etc rather than having them as local variables. An alternative would be to use `${day}` syntax (while remaining not a template string); this proposal uses `$<day>` to draw a parallel to the definition of the group and a distinction from template literals.

If the second argument to `String.prototype.replace` is a function, then the named groups can be accessed via a new parameter called `groups`. The new signature would be `function (matched, capture1, ..., captureN, position, S, groups)`. Named captures would still participate in numbering, as usual.

## Details

### Overlapping group names

RegExp result objects have some non-numerical properties already, which named capture groups may overlap with, namely `length`, `index` and `input`. In this proposal, to avoid ambiguity and edge cases around overlapping names, RegExps with such properties would be an Early Error, rejected when the RegExp is parsed.

An alternative possibility would be to provide named matches in a new `groups` object which is a property of the match result object.

### Backwards compatibility of new syntax

The syntax for creating a new named group, `/(?<name>)/`, is currently a syntax error in ECMAScript RegExps, so it can be added to all RegExps without ambiguity. However, the named backreference syntax, `/\k<foo>/`, is currently permitted in non-Unicode RegExps and matches the literal string `"k<foo>"`. In Unicode RegExps, such escapes are banned, however. Therefore, in this proposal, named backreferences are only permitted in Unicode RegExps.

An alternative possibility would be to only allow named capture groups in Unicode RegExps, to avoid the hazard of a user thinking they had support but not realizing that backreferences were not permitted.

## Precedent in other programming languages

This proposal is analogous to what many other programming languages have done for named capture groups. It seems to be what the consensus syntax is moving towards, though the Python syntax is an interesting and compelling outlier which would address the non-Unicode backreference issue.

### Perl [ref](http://perldoc.perl.org/perlre.html#Regular-Expressions)

Perl uses the same syntax as this proposal for named capture groups `/(?<name>)/` and backreferences `/\k<name>/`.

### Python [ref](https://docs.python.org/2/library/re.html#regular-expression-syntax)

Named captures have the syntax `"(?P<name>)"` and have backrereferences with `(?P=name)`.

### Java [ref](https://blogs.oracle.com/xuemingshen/entry/named_capturing_group_in_jdk7)

JDK7+ supports named capture groups with syntax like Perl and this proposal.

### .NET [ref](https://msdn.microsoft.com/en-us/library/bs2twtah(v=vs.110).aspx#Anchor_1)

C# and VB.NET support named capture groups with the syntax `"(?<name>)"` as well as `"(?'name')"` and backreferences with `"\k<name>"`.

### PHP

According to a [Stack Overflow post](http://stackoverflow.com/questions/6971287/named-capture-in-php-using-regex) and a [comment on php.net docs](http://php.net/manual/en/function.preg-match.php#89418), PHP has long supported named groups with the syntax `"(?P<foo>)"`, which is available as a property in the resulting match object.

### Ruby [ref](https://ruby-doc.org/core-2.2.0/Regexp.html#class-Regexp-label-Capturing)

Ruby's sytax is identical to .NET, with named capture groups with the syntax `"(?<name>)"` as well as `"(?'name')"` and backreferences with `"\k<name>"`.

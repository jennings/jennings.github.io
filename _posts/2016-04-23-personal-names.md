---
title: Personal Names
---

I'm lucky because most software is designed for me. I never have a decision to
make when I run across a form:

> <table>
>     <tr>
>         <td>First name</td>
>         <td><input type="text" value="Stephen"/></td>
>     </tr>
>     <tr>
>         <td>Last name</td>
>         <td><input type="text" value="Jennings"/></td>
>     </tr>
> </table>

But not every name follows the "FirstName LastName" convention. Some names are
written family-name-first (China), some people have multiple family names
(Hispanic cultures), and others have more complicated rules.

[This article from the W3C is an excellent primer on how personal names differ
across cultures][w3].

The gist is that, if you've written this code:

```c#
Console.WriteLine($"Hello {firstName} {lastName}!");
print( "Hello " + firstName + " " + lastName + "!" )
```

then you might have called some people by the wrong name. If you can, it could
be better to use "full name" and "nickname" fields:

> <table>
>     <tr>
>         <td>Full name</td>
>         <td><input type="text" value="Stephen Jennings"/></td>
>     </tr>
>     <tr>
>         <td>What should we call you?</td>
>         <td><input type="text" value="Stephen"/></td>
>     </tr>
> </table>

For a more humorous description of cultural differences in names: [Falsehoods
Programmers Believe About Names][falsehoods].

[w3]: https://www.w3.org/International/questions/qa-personal-names
[falsehoods]: http://www.kalzumeus.com/2010/06/17/falsehoods-programmers-believe-about-names/ 

# 019 — Mixin Arguments, Defaults, and Keyword Arguments: Q&A

## Parameters and positional arguments

### 1. What is a mixin parameter?

A parameter is a named value declared in a mixin definition. The mixin body can use that parameter as a Sass variable.

### 2. How do I declare parameters?

Write their dollar-prefixed names in parentheses after the mixin name, separated by commas.

### 3. What is a positional argument?

A positional argument is assigned to a parameter based on its position in the include call. The first value maps to the first parameter, the second to the second, and so on.

### 4. What is the difference between a parameter and an argument?

The parameter is the name in the mixin definition. The argument is the SassScript value supplied when the mixin is included.

### 5. What types of values can an argument contain?

Arguments can be SassScript expressions such as numbers, colors, strings, variables, calculations, or function results that make sense for the mixin.

### 6. Must every parameter be supplied?

Every parameter without a default must receive a value. A parameter with a default can be omitted.

### 7. What happens if a required argument is missing?

Sass reports an argument error because the mixin does not have a value for that required parameter.

### 8. Does a mixin automatically validate whether an argument is appropriate CSS?

No. The mixin author is responsible for designing appropriate inputs. Sass can catch syntax and type-related errors in some expressions, but a mixin can still emit an ineffective property or value.

## Defaults

### 9. How do I give a parameter a default value?

Add a colon and a SassScript expression after the parameter name in the mixin definition.

### 10. What happens when a caller omits an optional argument?

The default expression is used for that include.

### 11. Can a caller override a default?

Yes. Supply a positional value in the correct position or pass a keyword argument for that parameter.

### 12. Can a default refer to another parameter?

Yes. A default expression can refer to a parameter that appears earlier in the parameter list.

### 13. Why put required parameters before optional parameters?

It makes the call structure predictable: callers provide the required values first, and can omit the trailing values that have defaults.

### 14. Is !default how mixin parameter defaults are declared?

No. Mixin parameters use a colon and default expression in the parameter list. The !default flag is a separate Sass variable feature.

## Keyword arguments

### 15. What is a keyword argument?

A keyword argument names the parameter as well as supplying its value, for example $radius: 0.5rem.

### 16. When are keyword arguments helpful?

They are useful when a mixin has several parameters, especially when values have similar types or when callers need to override a later optional parameter.

### 17. Can keyword arguments be supplied in a different order from the definition?

Yes. Sass matches a keyword argument to its parameter name rather than its position.

### 18. Can positional and keyword arguments be combined?

Yes. A call can begin with positional arguments and then specify other values by name.

### 19. Can I put a positional argument after a keyword argument?

No. Put positional arguments before keyword arguments. After naming an argument, use names for any subsequent supplied values.

### 20. Can a keyword argument skip an optional parameter?

Yes. Name the later parameter you want to override. Any omitted optional parameter continues to use its default.

### 21. Can I provide a parameter both positionally and by keyword?

No. That supplies the same argument twice and Sass reports an error.

### 22. What happens if the keyword name is misspelled?

Sass reports an unknown argument name. Check that the keyword matches the parameter name in the mixin definition.

### 23. Can I name every argument?

Yes. Required and optional parameters can both be supplied by keyword. This can make a complex include easier to read.

## API design and debugging

### 24. Why do parameter names matter to callers?

Callers can use keyword arguments, so the names are part of the mixin's public API. Renaming one can break callers even if the mixin body still compiles.

### 25. Why are defaults useful?

They make common calls shorter while still allowing less common use cases to customize the mixin.

### 26. Should every parameter have a default?

No. Required parameters should remain required when the mixin cannot provide a meaningful default. Avoid defaults that hide a missing design decision.

### 27. What error occurs if too many positional values are supplied?

Sass reports an argument error unless the mixin is explicitly designed to accept arbitrary arguments. Variadic argument lists are an advanced topic.

### 28. How do I debug a wrong generated value?

Check the parameter order, argument values, keyword names, defaults, and the generated CSS. A positional call can compile while assigning values to the wrong parameters if the types are compatible.

### 29. Should I prefer positional or keyword arguments?

Use positional arguments when the order and meaning are obvious. Use keywords when they make the call easier to understand or when you need to skip an optional parameter.

### 30. What should I do before renaming a mixin parameter?

Search for all include sites, including keyword calls and other projects that consume the mixin. Update callers or provide a deliberate compatibility transition.

### 31. Can an argument be a Boolean value?

Yes. A mixin can accept true or false. How that Boolean changes emitted CSS is a separate conditional-logic topic.

### 32. Are mixin content blocks the same as arguments?

No. Arguments pass values into parameters. A content block passes a block of Sass statements to a mixin and is covered in the next chapter.

## Official references

- [Sass @mixin and @include: arguments and defaults](https://sass-lang.com/documentation/at-rules/mixin/)
- [Sass variables and default expressions](https://sass-lang.com/documentation/variables/)
- [Sass stylesheet structure](https://sass-lang.com/documentation/syntax/structure/)

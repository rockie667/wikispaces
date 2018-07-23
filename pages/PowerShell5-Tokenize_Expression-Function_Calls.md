---
title: PowerShell5-Tokenize_Expression-Function_Calls
---
[[http://schuchert.wikispaces.com/PowerShell5.TokenizeExpression.FirstStabAtParentheses|<— Back]]  [[PowerShell5.TokenizeExpression|^^ Up ^^]]  [[http://schuchert.wikispaces.com/PowerShell5.TokenizeExpression.ConvertTokenizerToAnEnumerator|Next—>]][
Finally, function calls. This might already work. Let's write an experiment and see what the results are:
```powershell
        @{expression = 'f(g(3))'; expected = @('f', '(', 'g', '(', '3', ')', ')')}
```
This passes. The question I'm wondering is whether we want those results, or something more like this:
```powershell
        @{expression = 'f(g(3))'; expected = @('f(', 'g(', '3', ')', ')')}
```
As it turns out, the first version passes as is. The second version requires a change to one of the regular expressions:
```powershell
    static [Array]$regex = @( '^([()])', '^([\d\w]+\({0,1})', '^([^\d\w\s]+)' )
```
The change is in the middle regex, allowing for zero or 1 ( at the end of a series of digits and letters. Given the change is easy, I'll leave this as is and consider the tokenizer done for now. Next, we'll move on to the [[PowerShell5.ShuntingYardAlgorithm|Shunting Yard Algorithm in PowerShell 5]].

[[PowerShell5.TokenizeExpression.FinalishVersion|Here's my final-ish version.]].
[[http://schuchert.wikispaces.com/PowerShell5.TokenizeExpression.FirstStabAtParentheses|<— Back]]  [[PowerShell5.TokenizeExpression|^^ Up ^^]]  [[http://schuchert.wikispaces.com/PowerShell5.TokenizeExpression.ConvertTokenizerToAnEnumerator|Next—>]][
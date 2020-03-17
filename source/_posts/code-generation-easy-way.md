title: "Code Generation - Easy Way"
date: 2009-06-05 01:44:41
tags:
- Visual Studio
---

Recently I read about Text [Template Transformation Toolkit](http://msdn2.microsoft.com/en-us/library/bb126445.aspx)(T4), I was really amazed by this because I thought only CodeDOM was the only way to generate code.

> T4 is a template based code generation engine which comes with Visual Studio 2008 (It is not a framework feature).

This simple example shows how powerful it is -

if you have a string array and you want to generate a strongly typed `enum` from this.

###Steps

1) Open up VS 2008 and Create a project.
2) Add a file called "EnumGenerator.tt" to the project. Accept the warning when add this file. Paste these code into the file you have created right now.

```
<#@ template language="C#" hostspecific="True" debug="True" #>
<#@ output extension="cs" #>
<#
  string[] values = new string[]{"Red","Green","Blue"};
#>

namespace TextTemplate {
<#
  PushIndent("\t");
#>
enum Color {
  <#
    for(int i = 0; i < values.Length; i++)
    {
      PushIndent("\t");
      WriteLine(values[i] + ((i != values.Length -1) ? "," : ""));
      PopIndent();
    }
  #>
}
<#
  PopIndent();
#>
```
3) Done! Click the plus sign near the `EnumGenerator.tt`, you can see the source code.

The beauty of this technique is, it is not runtime or compile time. The code is generated at design time which allow you to use this enum(or whatever code you have generated) at the same time you write your code.

To learn more about this, here is the great link - http://www.olegsych.com/2007/12/text-template-transformation-toolkit/

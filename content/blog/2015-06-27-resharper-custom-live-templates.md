---
date: 2015-06-27T00:00:00Z
tags: 
    - programming
    - resharper
title: Custom live templates for ReSharper
slug: resharper-custom-live-templates
---

Hi!

As a .NET developer I'm enjoying using an intelligent plug-in for Visual Studio which is called [ReSharper](https://www.jetbrains.com/resharper/).
It saves me time, provides me with static analysis and generates code for me.

In fact, code completion and code generation are a very crucial part of this product. By default, ReSharper comes with more than 20 [Live Templates](https://www.jetbrains.com/resharper/features/code_templates.html). The good fact is that it's easy to customize, and you can introduce your own templates.

## Creating the Live Template

Go to RESHARPER -> Templates Explorer.

![Template Explorer](/images/2015/06-resharper-custom-live-templates/templates-explorer.PNG)

Select the C# scope and press the New Template button. 

We are going to create a template for the async method which returns a Task\<T\>.

Let's type a template body:

{{< highlight php >}}
public async System.Threading.Tasks.Task<$RETURN_TYPE$> $METHOD_NAME$Async(){$END$}
{{< / highlight >}}

`$RETURN_TYPE` and `$METHOD_NAME` are placeholders for the return type and for the method name.

$END$ - marks the point where the text caret is placed when the template is inserted. 


## Availability

Live templates allow us to define the scope where the template will be available. In our case, the async keyword is valid in the context of C# 5.0 and higher versions.

To do so, click the Availability link on the right side and select the option "In C# where type member declaration is allowed". This will prevent the template from being activated in unexpected places (like inside the method). After that, set the minimum C# version to 5.0.


![availability](/images/2015/06-resharper-custom-live-templates/availability.PNG)

Now the template should look like the following:

![Result](/images/2015/06-resharper-custom-live-templates/overview.png)

I've declared the shortcut 'am' for the template.

## In action

Let's type 'am' in the code, and here we go: our brand new template is activated.

![Template in action](/images/2015/06-resharper-custom-live-templates/in-action.PNG)

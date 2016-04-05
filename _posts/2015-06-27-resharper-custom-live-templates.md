---
layout: post
title: Custom live templates for ReSharper
tags: templates, resharper
---

Hi!

As a .NET developer I'm enjoying to use an intelligent plug-in for Visual Studio which is called [ReSharper](https://www.jetbrains.com/resharper/).
It saves me time, provides me a static analysis and generates code for me.

In fact code completion and code generation is a very crucial part of this product. By default ReSharper goes with more than 20 [Live Templates](https://www.jetbrains.com/resharper/features/code_templates.html). The good fact is that it's easy to customize, and you can introduce your own templates.

## Creating the Live Template

Go to RESHARPER -> Templates Explorer 

![Template Explorer](/images/resharper-custom-live-templates/templates-explorer.PNG)

Select the C# scope and press the New Template button. 

We are going to create a template for the async method which returns a Task\<T\>.

Let's type a template body:

{% highlight php %}
public async System.Threading.Tasks.Task<$RETURN_TYPE$> $METHOD_NAME$Async(){$END$}
{% endhighlight %}

RETURN\_TYPE and METHOD\_NAME are placeholders for the return type and for the method name.

$END$ - marks the point where the text caret is placed when the template is inserted. 


## Availability

Live templates allows us to define the scope where the template will be available. In our case async keyword is valid in the context of C# 5.0 and higher versions.

To do so click the Availability link on the right side and select the option "In C# where type member declaration is allowed". This will prevent the template to be activated in unexpected places (like inside the method). After that set the minimum C# version to 5.0.


![availability](/images/resharper-custom-live-templates/availability.PNG)

Now the template should look like the following:

![Result](/images/resharper-custom-live-templates/overview.png)

I've declared the shortcat 'am' for the template.

## In action

Let's type 'am' in the code and here we go: our brand new template is activated.

![Template in action](/images/resharper-custom-live-templates/in-action.PNG)

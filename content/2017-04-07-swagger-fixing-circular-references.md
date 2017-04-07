---
title: "Swagger: ASP.NET Core Fixing Circular Self References"
author: "Nicklas Møller Jepsen"
date: "2017-04-07"
url: "/2017/04/07/swagger-asp-net-core-fixing-circular-self-references/"
comments: "true"
categories:
  - programming
tags:
  - c-sharp
  - api
  - webapi
  - entityframework
  - swagger
  - swashbuckle
  - ef
  - aspnet
  - aspnetcore 
---
## The Problem
When creating a ASP.NET/ASP.NET Core Web API, providing good documentation is critical. Swagger greatly helps with that by autogenerating the doc for you.

Now if you use the fully integrated setup with Entity Framework and scaffolding controllers by providing your model and db context you get Visual Studio to do all the hard work for you. However, if you have an EF model where there's a relation to an entity that has a relation back to the original entity you will end up with a circular loop of references when generators kick in.

## The Solution
Use Swaggers [ISchemaFilter](https://github.com/domaindrivendev/Swashbuckle/blob/master/Swashbuckle.Core/Swagger/ISchemaFilter.cs) interface!
I wrote an implementation that basically ignores all related objects in the same namespace as the one the requsted.

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Reflection;
using Swashbuckle.AspNetCore.Swagger;
using Swashbuckle.AspNetCore.SwaggerGen;

namespace SystemOut.Swagger.EfHelpers
{
    /// <summary>
    /// Apply this filter to ignore all related objects that could cause  a self referencing loop
    /// </summary>
    /// <typeparam name="TNsType"></typeparam>
    public class ApplyIgnoreRelationshipsInNamespace<TNsType> : ISchemaFilter
    {
        /// <summary>
        /// Required by interface
        /// </summary>
        /// <param name="model"></param>
        /// <param name="context"></param>
        public void Apply(Schema model, SchemaFilterContext context)
        {
            if (model.Properties == null)
                return;
            var excludeList = new List<string>();

            if (context.SystemType.Namespace == typeof(TNsType).Namespace)
            {
                excludeList.AddRange(
                    from prop in context.SystemType.GetProperties()
                    where prop.PropertyType.Namespace == typeof(TNsType).Namespace
                    select prop.Name.ToCamelCase());
            }

            foreach (var prop in excludeList)
            {
                if (model.Properties.ContainsKey(prop))
                    model.Properties.Remove(prop);
            }
        }
    }
}
```

You apply the filter in the Startup class in `ConfigureServices`, like so:

```csharp
services.AddSwaggerGen(c =>
{
	c.SchemaFilter<ApplyIgnoreRelationshipsInNamespace<YourEfModelNameSpace>>();

	...
```

Oh and in case you wonder, the `ToCamelCase` extension is implemented like so:

```csharp
public static class StringExtensions
{
    public static string ToCamelCase(this string str)
    {
        if (string.IsNullOrEmpty(str))
            return str;
        return str[0].ToString().ToLower() + str.Substring(1, str.Length-1);
    }
}
```


You can find the source on Github: [Swagger EF Model Helpers](https://github.com/nicklasjepsen/swagger-ef-model-helpers)

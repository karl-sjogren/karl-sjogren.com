---
title:      "Switching from Moq to FakeItEasy"
date:       2023-09-16 21:00:00
tags:       c# dotnet dotnet-template
---

Every time I start a new C# project I tend to do the same things every time.
I create the folder structure I want, with a `src` and `test` folder at the
root, I add the basic files such as `nuget.config`, `Directory.Build.props`,
`.editorconfig` etc. I usually copy these from some other project I had checked
out just because it was easier.

This quickly becomes a hassle and maybe I get some custom settings from a 
specific project that I won't want. So I started looking into making a
template with my standard setup sometime at the end of last year.

It turns out that creating a template for use with `dotnet new` is really,
really simple. All you need is a nuget package with a `template.json` file
that sets up some metadata.

I won't go through how to setup a simple template here, if you don't want to
read about **my** template, you can read more over at [Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/core/tools/custom-templates).

## Karls Opinionated Solution

Thats the name of my template. It conveys two things, that it is mine and
that it is how **I** want things setup. This isn't a collaboration (though
I've incorporated some things suggested by friends and colleagues) or
something that everyone will want to use. But it has a lot of good things
that you might want to use or modify for your own project.

If you want to check it out yourself you can create a new project by
executing the below commands.

```sh
dotnet new install Karls.Templates
dotnet new karls-solution -n TemplateSample
```

Or you can just go directly to the source at <https://github.com/karl-sjogren/dotnet-template>.

## Basic template structure

If you look at the repository the basic structure is **not** as I prefer
to setup my project, but this is a special one.

The template repository has a loot of stuff in it, currently sitting at
18 directories and 46 files. I'm hoping to cover most of it in a few
blog posts over the coming weeks but I'll start with the basics.

Below is the minimal amount of files needed for (an empty) template.

    .
    ├── Karls.Templates.csproj
    └── templates
        └── opinionated-solution
            └── .template.config
                └── template.json

As I said at the start, the template is just a nuget package so at the
root we have a  `.csproj` file that targets `netstandard2.0` and a 
`PackageType` that is set to  `Template`. It also needs a version and
a `PackageId` set. You can read more at the Microsoft Learn link at the
start of the article.

Currently there is only one template in this package but I've thought
about adding some more smaller templates into it as well.

Here is a slimmed down version of the project file at the root of my
repository.

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <PackageType>Template</PackageType>
    <PackageVersion>1.1.7</PackageVersion>
    <PackageId>Karls.Templates</PackageId>
    <PackageTags>dotnet-new;templates</PackageTags>

    <TargetFramework>netstandard2.0</TargetFramework>

    <IncludeContentInPack>true</IncludeContentInPack>
    <IncludeBuildOutput>false</IncludeBuildOutput>
    <ContentTargetFolders>content</ContentTargetFolders>
  </PropertyGroup>

  <ItemGroup>
    <Content Include="templates\**\*" Exclude="templates\**\bin\**;templates\**\obj\**" />
    <Compile Remove="**\*" />
  </ItemGroup>

</Project>
```

The next required file is the `.template.config/template.json` file.
Every template in the package needs one of these, but the package can
contain several templates.

My `template.json` contains a lot of metadata for the template, it also
adds optional arguments to pass to `dotnet new`.

This is a stripped down version of my `template.json`, it has my name,
the template name, some tags and a `shortName` tag so that we don't
have to type out the whole name when we want to start a new project.

```json
{
  "$schema": "http://json.schemastore.org/template",
  "identity": "Karls.Templates.OpinionatedSolution",
  "author": "Karl-Johan Sjögren",
  "classifications": [
    "Opinionated",
    "Solution"
  ],
  "name": "Karls Opinionated Solution",
  "shortName": "karls-solution"
}
```

In my next post I'll look more into the `template.json` file and
all the things that I have in it.

---
layout: post
title: "Localizing Entity Framework POCO Properties with JSON - Part 1"
date: 2013-09-07
post_id: 7
---

<div class="update">
    <p><strong>UPDATE</strong> (February 2019)</p>
    <p>A new and re-branded version of OrangeJetpack.Localization is now available as <a href="https://xaki.io">Xaki</a>!</p>
    <p>Xaki is a re-envisioned and rewritten package for simple localization in ASP.NET Core and .NET Standard with full compatibility with ASP.NET MVC and classic Entity Framework.</p>
</div>

It’s day #1 on a new project demanding my favorite requirement: localization yay! The yay emphasize is mine because localization presents some thorny technical challenges that often pull along a trailer load of complexity. Framework tools make many of the localization tasks a breeze but there’s always that one piece just beyond .NET’s reach: storing localized text in a database. Since I’ve spent far too many hours (days?) wrestling with lookup table and lookup tables for lookup table solutions it’s time to start this project off with a simple and elegant solution – one that plays nice with Entity Framework and maybe even brings its own UI to the party.

My requirements are simple: I’m using EF code first and will need to store entities having multilanguage properties... and I want to work with a super-simple, elegant, unobtrusive API that reads like poetry. Spoiler alert — we’re bringing JSON to the party along with my new favorite NuGet package (ok dislaimer I wrote it).

![](https://andy.azureedge.net/blog/localized-property-636217948871506770.jpg)

Let’s jump in — this tutorial will ditch the too-popular generic lookup table approach and store localized text for entities as key/value serialized JSON. Our JSON is simple, readable, and searchable. We’ll then use some simple extension methods from the NuGet package [OrangeJetpack.Localization](https://www.nuget.org/packages/OrangeJetpack.Localization) to make getting and setting our localized text a breeze.

To peak ahead just a bit we’ll be using a regular EF POCO model and code-first migrations. We’ll work with an entity **Planet** having a **PlanetId** and a **Name** property that will eventually be localized to English, Russian, and Japanese. Ultimately it will look like this in our database:

![](https://andy.azureedge.net/blog/8-28-2013-8-54-38-pm-636217948862861091.png)

## Setup

To start things off new-up a fresh ASP.NET MVC web application. You can play along at home or download a working demo at: [https://github.com/andy-mehalick/OrangeJetpack.LocalizationDemo](https://github.com/andy-mehalick/OrangeJetpack.LocalizationDemo). Next we’ll add our simple EF POCO and DbContext classes:

```csharp
public class Planet
{
    [Key, DatabaseGenerated(DatabaseGeneratedOption.Identity)]
    public int PlanetId { get; set; }

    [Required]
    public string Name { get; set; }
}

public class PlanetsContext : DbContext
{
    public DbSet<planet> Planets { get; set; }
}
```

We’ll then let code first migrations create our local db with some Powershell in Package Manager Console:

```shell
> Enable-Migrations
```

```shell
> Add-Migrations AddPlanet
```

```shell
> Update-Database
```

Finally, let’s add a HomeController with Index() and scaffold a list view.

```csharp
public class HomeController : Controller
{
    private readonly PlanetsContext _db = new PlanetsContext();

    public ActionResult Index()
    {
        var planets = _db.Planets.ToList();

        return View(planets);
    }
}
```

Snap your fingers to add some sample data and you should see:

![](https://andy.azureedge.net/blog/8-28-2013-7-36-35-pm-636217948851575515.png)

Eesh, close but let’s now show the localized content only.

## NuGet and OrangeJetpack.Localization.Mvc

Start by added the NuGet package [OrangeJetpack.Localization.Mvc](https://www.nuget.org/packages/OrangeJetpack.Localization.Mvc) (it will drag its dependency [OrangeJetpack.Localization](https://www.nuget.org/packages/OrangeJetpack.Localization) along with it):

```shell
> Install-Package OrangeJetpack.Localization.Mvc
```

This package will add a LocalizationLanguages property to app settings, let’s add the language codes for English, Russian, and Japanese:

```xml
<configuration>
    <appSettings>
        <add key="LocalizationLanguages" value="en,ru,ja" />
    </appSettings>
</configuration>
```

Finally, let’s update Planet to indicate it is localizable and that Planet.Name is localized – we can do this by implementing ILocalizable and decorating Name with LocalizedAttribute:

```csharp
public class Planet : ILocalizable
{
    [Key, DatabaseGenerated(DatabaseGeneratedOption.Identity)]
    public int PlanetId { get; set; }

    [Required, Localized]
    public string Name { get; set; }
}
```

This now generously grants us access to an IEnumerable<ILocalizeable>.Localize() extension method. This method accepts a language code and a list of properties to localize.

Back to HomeController.Index(), add an optional langCode input parameter and change _db.Planets.ToList() to _db.Planets.Localize(langCode, I => i.Name):

```csharp
public class HomeController : Controller
{
    private readonly PlanetsContext _db = new PlanetsContext();

    public ActionResult Index(string langCode = "en")
    {
        var planets = _db.Planets.Localize(langCode, i => i.Name);

        return View(planets);
    }
}
```

Of course we could get language code from the user’s web browser or a profile setting but let’s keep it simple for now. If we omit a language code or use an unsupported one it will default to the first language in our app settings, “en” by default. Let’s run it again passing in the language code for Russian:

![](https://andy.azureedge.net/blog/8-28-2013-7-59-35-pm-636217948858550811.png)

Here’s what it would look like with no language, Japanese, or an unknown language:

![](https://andy.azureedge.net/blog/8-28-2013-8-54-38-pm-636217948862861091.png)

## Multiple Localized Properties

Finally, one bonus of our extension method Localize() is its support for multiple properties; if Planet had additional localized properties we could pass them in as a params[] list:

```csharp
_db.Planets.Localize(langCode, i => i.Name, i => i.Description, i => i.Atmosphere);
```

## Conclusion

<p><strike>That’s it for now, in Part 2 we’ll look at adding and updating our localizable entities with the included editor template for MVC and eventually we’ll dig deeper in OrangeJetpack.Localization implementation. If you want to jump ahead you can find the project at [https://github.com/andy-mehalick/OrangeJetpack.Localization](https://github.com/andy-mehalick/OrangeJetpack.Localization).</strike></p>

For more information and a better approach to localization be sure to check out [Xaki](https://xaki.io).
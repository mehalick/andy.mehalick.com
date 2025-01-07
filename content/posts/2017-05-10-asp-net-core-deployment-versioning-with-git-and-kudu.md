---
layout: post
post_id: 10
date: 2017-05-10
title: ASP.NET Core Deployment Versioning with Git and Kudu
excerpt: How to add deployment version metadata to ASP.NET Core websites using Git, Microsoft Azure, and Kudu.
cover: https://andy-bhbtdzffahctcwh8.z01.azurefd.net/blog/crab-636301801571668026.jpg
---

While there's no shortage of fancy and elaborate ways to add version and deployment metadata into ASP.NET apps you already have an easy option if you're using Git and Azure's Kudu deployment. With a simple script and a few extra bits added to your ASP.NET Core project you can easily add version and deployment data directly into your pages:

```html
<meta name="application-name" content="MyApp" data-version="8e879f1" data-deployment="2017-05-10 10:21:14Z" /> 
```

We're assuming you're already set up with Git automated deployment in Azure using [Kudu](https://github.com/projectkudu/kudu/wiki). If not perhaps you should be, it's a quick and easy way to get automated deployment working with ASP.NET, ASP.NET Core, Node, PHP, and basic websites using Git repositories hosted in GitHub, Bitbucket, and Azure.

The first thing we'll need to do it extend our out-of-box Kudu deployment to add a version file to our site each time a deployment completes. We can use a simple JSON file with some commit info and our deployment date:

```json
{
    "DeployUtc":  "2017-05-10 10:21:14Z",
    "ShortHash":  "8e879f1",
    "FullHash":  "8e879f1c8a791a99a5ae40210fd5240a478f2c6c",
    "Subject":  "some commit message here...",
    "Name":  "Andy Mehalick",
    "Email":  "mehalick@gmail.com"
}
```

We really only need the first two properties but the rest might be useful in other scenarios.

## Post Deployment Action Hook

To create our **version.json** file we'll add a [post deployment action hook](https://github.com/projectkudu/kudu/wiki/Post-Deployment-Action-Hooks) into our Kudu deployment. This is just a simple script that Kudu will know to run with each deployment. Kudu can use a variety of script types and in this case we'll use PowerShell:

```powershell
$o = @{}
$o.DeployUtc = Get-Date -format u;
$o.ShortHash = git.exe log -1 --pretty=format:%h;
$o.FullHash = git.exe log -1 --pretty=format:%H;
$o.Subject = git.exe log -1 --pretty=format:%s;
$o.Name = git.exe log -1 --pretty=format:%cn;
$o.Email = git.exe log -1 --pretty=format:%ce;

$o | ConvertTo-Json | Out-File "D:\home\site\wwwroot\version.json";
```

Our script executes git.exe in Azure and writes the details from the last commit to our JSON file. If you'd like to write more commit info to your JSON you can see what's available via Git's [pretty formats doc](https://git-scm.com/docs/pretty-formats).

We'll save the above script as **Set-GitAppVersion.ps1** and while we could simply upload it to Kudu it's better to save it into our existing Git repository and then just tell Kudu where to find it.

First, save this file as ~\deploy\PostDeploymentActions\Set-GitAppVersion.ps1 and then create a ~\\.deployment file with the following setting:

```conf
[config]
SCM_POST_DEPLOYMENT_ACTIONS_PATH = deploy\PostDeploymentActions\
```

This sets us up nicely for future post deployment scripts as well, anything added to our deploy\PostDeploymentActions\ folder will get executed automatically by Kudu after each deployment.

To see everything in action you'll need to set up Git automated deployment using GitHub, Bitbucket, or even Azure directly if you want. Once you're set up and a deployment has been triggered you'll be able to see a log of you post deployment scripts. 

If you're pushing directly to Azure:

![](https://andy-bhbtdzffahctcwh8.z01.azurefd.net/blog/2017-04-29_13-28-30-636290585727143991.png)

Or via the Azure portal, App > Deployment Options > Deployment Details:

![](https://andy-bhbtdzffahctcwh8.z01.azurefd.net/blog/2017-04-29_13-24-11-636290582870290991.png)

Finally, to confirm version.json was created successfully you can connect to your site via FTP or via your Kudu site available at https://{siteName}.scm.azurewebsites.net/DebugConsole:

![](https://andy-bhbtdzffahctcwh8.z01.azurefd.net/blog/2017-04-29_13-14-40-636290577131373014.png)

## Reading our Version Deployment File

Our ASP.NET app will read version.json and write some or all of the data into our pages. In this example we'll use the Git short hash as our version along with our deployment date/time, so let's create a simple model that we can use:

```cs
public class AppVersion
{
    public string ShortHash { get; set; } = "000000";
    public DateTime DeployUtc { get; set; } = DateTime.UtcNow;
}
```

We can read version.json with a service that uses ASP.NET Core's IHostingEnvironment to find our file at the root of our app:

```cs
public class AppVersionService : IAppVersionService
{
    private readonly IHostingEnvironment _hostingEnvironment;

    public AppVersionService(IHostingEnvironment hostingEnvironment)
    {
        _hostingEnvironment = hostingEnvironment;
    }

    private AppVersion _appVersion;
    public AppVersion AppVersion
    {
        get
        {
            if (_appVersion != null)
            {
                return _appVersion;
            }

            var path = Path.Combine(_hostingEnvironment.WebRootPath, "version.json");
            if (path == null || !File.Exists(path))
            {
                _appVersion = new AppVersion();
            }
            else
            {
                using (var sr = File.OpenText(path))
                {
                    _appVersion = JsonConvert.DeserializeObject<AppVersion>(sr.ReadToEnd());
                }
            }

            return _appVersion;
        }
    }
}
```

So that we only have to read and deserialize version.json once we can set AppVersionService up as a singleton in Startup.cs:

```cs
public void ConfigureServices(IServiceCollection services)
{
    // ...

    services.AddSingleton<IAppVersionService, AppVersionService>();
}
```

## Embedding or Displaying our Version

And finally,.. you can access AppVersionService directly in _Layout.cshtml or in any view using ASP.NET Core's dependency injection:

```cs
@inject IAppVersionService AppVersionService
@{
    var shortHash = AppVersionService.AppVersion.ShortHash;
    var deployUtc = AppVersionService.AppVersion.DeployUtc.ToString("u");
}
```

How you embed or display your version metadata is entirely up to you, here's a few ideas:

### HTML5 Data-* Attibutes

My favorite option is appending our data as HTML5 data-* attributes on the application-name meta tag. This feels semantically legit and makes it easy to find and read:

```html
<meta name="application-name" content="MyApp" data-version="8e879f1" data-deployment="2017-05-10 10:21:14Z" />
```

### JSON Data Blocks

This option is clean and readable as well, just use a JSON data block:

```html
<script id="version" type="application/json">
    { 
        "ShortHash": "@shortHash",
        "DeployUtc": "@deployUtc"
    }
</script>
```

### HTML Meta Tags

Or perhaps some custom meta tags:

```html
<meta name="version:ShortHash" content="@shortHash" />
<meta name="version:DeployUtc" content="@deployUtc" />
```

### Microdata

If you plan to show version data in the footer of your site perhaps use microdata:

```html
<footer itemscope itemtype="http://schema.org/SoftwareApplication">
    &copy; Git App Version Demo | 
    Version: 
        <span itemprop="softwareVersion">@shortHash</span> 
        <time itemprop="startDate" datetime="@deployUtc">@deployUtc</time>
</footer>
```

## Sample Code and Demo

You can find a sample project on GitHub at [https://github.com/mehalick/GitAppVersion](https://github.com/mehalick/GitAppVersion) and a demo on Azure at [https://gitappversion.azurewebsites.net](https://gitappversion.azurewebsites.net/).

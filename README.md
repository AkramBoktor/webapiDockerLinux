# webapiDockerLinux
Docker with asp.net core &amp; users secret

# Managing options and secrets in .NET Core and Docker

.NET Core comes with tens of good patterns and helpers which are supposed to help developers to create high quality code which is easy to maintain and fast to develop.

Numerous tools help us to be more productive but also encourage us to be responsible for our code and developer assets.

I’m going to show you how do I solve the problem of managing configuration in .NET Core, including multiple environments, secrets and Docker.

# App Settings
First of all: where do we use configuration? Of course there is no simple answer for that and it always depends, but my strategy for this is following. Each time when you use some constant (regardless of having them in the constant field or just used value) ask yourself: “Might this value ever change?”. If the answer is “yes” or “maybe” you should put it into a configuration value.

Example:

String.Empty – constant
Number of minutes to expire cache – configuration value
In old .NET Framework we had a class called ConfigurationManager which had two main drawbacks: it used XML (the part of App.config file) and it was a static class, so you couldn’t easily mock it or inject. What we have now in .NET Core are Options and an interface IConfigurationRoot which is a root of all configuration nodes. It means that now we don’t have a flat configuration file with tens or hundreds of floating configuration keys, but a structured and grouped application settings.

To start working with this, you need to use ConfigurationBuilder. If you use ASP.NET Core it’s worth to build it in the Statup constructor, where you have access to IHostingEnvironment and you can choose used configuration file is going to be used.

The construction can look like this:
![image](https://user-images.githubusercontent.com/35446384/180922996-0633089a-e9b3-43fe-9fc0-f63e6b870bd4.png)

SetBasePath tells where should all files are placed. In the typical scenario it’s the current root path.
AddJsonFile adds a particular file to our configuration. It’s important to use the one that corresponds to the current environment. We should treat
appsettings.json as a common configuration file, while the one with environment name suffix is a environment specific one. Basically, one uses two basic environments: Development and Production (files: appsettings.Development.json, appsettings.Production.json), but for real business projects you should include Staging and Testing environments with setting values defined according to your needs. There are also two optional parameters: when you set optional to true, it means that you will not get any exception when the file does not exist and reloadOnChange will just update the configuration tree in memory every time when you edit a file.
AddEnvironmentVariables() adds a support for locally configured environment variables, so when you set up a variable you can use it in the app. auth:token crates a token value in auth section.
There is a bunch of providers available on NuGet which supports other files or sources. There is no problem to use XML or ini files. Just search for Microsoft.Extensions.Configuration and find something for you. There is something else worth to mention which is Microsoft.Extensions.Configuration.AzureKeyVault. It provides a service for storing secret data in secure storage on Azure – I’ll go deeper into this concept on another post.

Let’s add just two simple files with some service configuration files: **appsettings.json:**

![image](https://user-images.githubusercontent.com/35446384/180923074-395b084b-9d3f-4d1a-a0c1-8fd5fd77b709.png)


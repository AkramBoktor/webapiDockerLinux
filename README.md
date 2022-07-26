# webapiDockerLinux
1 - asp.net core v5
2 - Docker , Docker Desktop
# Docker with asp.net core &amp; users secret

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

![image](https://user-images.githubusercontent.com/35446384/180923196-3feaf78c-d577-4abc-986c-095768f1ec1e.png)

So far so good! We are ready to use our configuration in the app. The ASP.NET Core gives an ability to register it in the container so we can inject it into other classes. When we use default ASP.NET Core Dependency Container we should configure services in ConfigureServices method in Startup class:

public void ConfigureServices(IServiceCollection services)

![image](https://user-images.githubusercontent.com/35446384/180923263-371d714c-3b5a-4c70-8419-876f230ac268.png)

Then you can inject the confiruration into your classes wrapped in a IOptions<T> class where T is the config class defined a while ago:

![image](https://user-images.githubusercontent.com/35446384/180923302-ae7ed712-f7a5-4ac9-b7b5-e8749cf4b32c.png)

![image](https://user-images.githubusercontent.com/35446384/180923340-c766af9e-065d-42ca-8dd9-169b2b7c0dd6.png)

# Secrets
There is a temptation to use app settings to all kinds of variable config values, but there are some which should not be saved in the appsettings.json.

DO NOT SAVE IN appsettings.json FOLLOWING SETTINGS:

connection strings
passwords
password hashes
API keys
tokens
“Peppers”
cryptographic keys
… and other values which are confidential
Remember that those files are checked in your source code control system and you probably don’t want to share your production passwords with all repository users. What’s more, it might happen that some day your repository will be compromised or you simply go open source. I can bet, that there are tens of confidential data on GitHub or BitBucket public repositories. You don’t want to make this mistake.

.NET Core comes to you with a solution. They developed sort of provider which apart of looking into appsettings.json uses values from some secret.json file which is saved on each developer’s machine locally in the special folder assigned to the project (the folder name is a GUID specific for a project).

So to use them we will use VisualStudio 2017. We will need to install another **NuGet package: Microsoft.Extensions.Configuration.UserSecrets.** Next, let’s use IDE to generate secrets.json in a proper directory. Right-click on the project and choosing “Manage User Secrets” will do the thing.

  ![image](https://user-images.githubusercontent.com/35446384/180923480-f579a0ce-11ea-47b3-b5d8-65d60e0b87bc.png)

  A JSON file will open, but nothing will be added into your project. It’s been created in one of the following folders (depending on your OS):

Windows: %APPDATA%\microsoft\UserSecrets\<userSecretsId>\secrets.json
Linux: ~/.microsoft/usersecrets/<userSecretsId>/secrets.json
Mac: ~/.microsoft/usersecrets/<userSecretsId>/secrets.json The <userSecretsId> is basically a GUID which can be checked in the .csproj file.
Let’s setup some configuration here:
  
  ![image](https://user-images.githubusercontent.com/35446384/180923567-30cfd7e2-682b-4ca4-bba3-bc84b09ee46f.png)

  The last thing to use it is adding secrets support into our configuration. We will do it only in Development environment, though, since it’s not recommended to use user’s secrets on the production.

![image](https://user-images.githubusercontent.com/35446384/180923724-04289a1f-bc0e-4bc6-a1e9-902003b188b3.png)

  

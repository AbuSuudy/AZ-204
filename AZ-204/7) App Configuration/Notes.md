# App Configuration


Provides a central configuration that allows to be used multiple resources that aren't secrets.

```c#
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.Configuration.AzureAppConfiguration;
using Microsoft.FeatureManagement;
using Azure.Identity;

var builder = new ConfigurationBuilder();
builder.AddAzureAppConfiguration(options =>
{
    string endpoint = Environment.GetEnvironmentVariable("Endpoint");
    options.Connect(new Uri(endpoint), new DefaultAzureCredential());
});

Console.WriteLine(config["TestApp:Settings:Message"]);
```
## Access Nested value  
You can access nested values in your  `appsettings.json`  by using 

```json 
{
  "UserSettings": {
    "Features": {
      "EnableBetaMode": true,
      "UI": {
        "Theme": "Dark",
        "FontSize": 14
      }
    },
    "Security": {
      "TwoFactorEnabled": false,
      "AllowedIPs": [ "10.0.0.1", "10.0.0.2" ]
    }
  }
}
```

```c#
Console.WriteLine(config["UserSettings:Features:UI:Theme"]);
//Dark
```

### Extract section from configuration
You could extract a section of the config you're interested in and cast to an object if you want that ergonomics.

```c#
string featureSection = _config.GetSection("UserSettings:Features");

//If you want to use Strongly Typed Class
Features featureSection = _config.GetSection("UserSettings:Features")
.Get<Features>();
```

```json
{
  "EnableBetaMode": true,
  "UI": {
    "Theme": "Dark"
  }
}

```
## Specify a label
Allows labels for each environment and feature flags to turn on a set of configurations. 

![](Images/Pasted%20image%2020251212170344.png)

```c#
var builder = WebApplication.CreateBuilder(args);

builder.Configuration.AddAzureAppConfiguration(options =>
    {
        string endpoint = Environment.GetEnvironmentVariable("Endpoint");
        options.Connect(new Uri(endpoint), new DefaultAzureCredential())
               // Load configuration values with no label
               .Select(KeyFilter.Any, LabelFilter.Null)
               // Override with any configuration values specific to current hosting env
               .Select(KeyFilter.Any, builder.Environment.EnvironmentName);
    });
```

## Key Vault Integration
You could integrate key vault and app configuration together and access values using the same method.

```c#
var builder = WebApplication.CreateBuilder(args);

// Retrieve the App Configuration endpoint.
string endpoint = builder.Configuration.GetValue<string>("Endpoints:AppConfiguration")

// Load the configuration from App Configuration.
builder.Configuration.AddAzureAppConfiguration(options =>
{
    options.Connect(new Uri(endpoint), new DefaultAzureCredential());

    options.ConfigureKeyVault(keyVaultOptions =>
    {
        keyVaultOptions.SetCredential(new DefaultAzureCredential());
    });
});
```

## Feature Flags
Azure App Configuration provides feature management through feature flags, enabling teams to dynamically control application functionality without redeploying code. You could create flag instead of overloading you `appsettings.json`

![](Images/Pasted%20image%2020251212174254.png)

![](Images/Pasted%20image%2020251212174417.png)
## Sample .NET Generic Host as Windows Service

Visual Studio 2019 has a project template called a Worker Service. The Worker Service project template creates a simple .NET Core Generic Host application that starts a `BackgroundService` implementation of the `IHostedService` interface.

The project follows the boilerplate template for setting up logging, configuration and dependency injection. This setup is described in detail in the [`ConsoleHostSample` project](https://github.com/JeffEmery/ConsoleHostSample). [There is detailed information about the .NET Generic Host setup here.](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/host/generic-host?view=aspnetcore-5.0)

The `Worker` `BackgroundService` in the template uses the familiar `while (!stoppingToken.IsCancellationRequested)` loop with a timed `Task.Delay` to loop.

> The loop means this service is not signaled by external events to do work. This leads to the age old problem of a service polling every so often to look for work.

### Running as a Windows Service

[Glenn Condron's blog describes this Visual Studio Worker Service template.](https://devblogs.microsoft.com/aspnet/net-core-workers-as-windows-services/)

The template will run the application as a console app, starting automatically and terminating upon `CTRL+C`.

The `HostBuilder` requires more setup to respond to Windows Service events like Start and Stop.

1. Install the [Microsoft.Extensions.Hosting.WindowsServices](https://www.nuget.org/packages/Microsoft.Extensions.Hosting.WindowsServices/) nuget package.

```
Install-Package Microsoft.Extensions.Hosting.WindowsServices
```

2. Add ~~`UseServiceBaseLifetime()`~~ `UseWindowsService()` to the builder.

```csharp
Host.CreateDefaultBuilder(args)
    .UseWindowsService()
    .ConfigureServices((hostContext, services) =>
    {
        services.AddHostedService<Worker>();
    });
```

`UseWindowsService()` detects if it's running as a Windows Service and if it isn't, then it is a *noop*, allowing the application to run from the command line.

3. Install the program as a Windows Service with the [sc utility](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/sc-create)

> [See here for more detailed information on running .NET Core applications as a Windows Service](https://docs.microsoft.com/en-us/aspnet/core/host-and-deploy/windows-service?view=aspnetcore-2.2&tabs=visual-studio)

```
sc create WindowsServiceSample binPath=c:\code\WindowsServiceSample\WindowsServiceSample.exe
```

It may be helpful to [`dotnet publish`](https://docs.microsoft.com/en-us/dotnet/core/tools/dotnet-publish) the project to get the program files ready in a single folder.

```
dotnet publish -o c:\code\WindowsServiceSample
```

### Logging

[Logging is part of the default `CreateDefaultBuilder(args)` configuration for the .NET Generic Host.](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/logging/?view=aspnetcore-5.0#logging-providers) which includes the Windows Event Log.


 










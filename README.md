# eShopOnWeb

Sample ASP.NET Core reference application, powered by Microsoft, demonstrating a single-process (monolithic) application architecture and deployment model. 

This reference application is meant to support the free .PDF download ebook: [Architecting Modern Web Applications with ASP.NET Core and Azure](https://aka.ms/webappebook), updated to **ASP.NET Core 2.1**.

You can also read the book in online pages at the .NET docs here: 
https://docs.microsoft.com/en-us/dotnet/standard/modern-web-apps-azure-architecture/

![image](https://user-images.githubusercontent.com/1712635/42467632-449688c2-8367-11e8-9323-81ab50a66006.png)

The **eShopOnWeb** sample is related to the [eShopOnContainers](https://github.com/dotnet/eShopOnContainers) sample application which, in that case, focuses on a microservices/containers-based application architecture. However, **eShopOnWeb** is much simpler in regards to its current functionality and focuses on traditional Web Application Development with a single deployment.

The goal for this sample is to demonstrate some of the principles and patterns described in the [eBook](https://aka.ms/webappebook). It is not meant to be an eCommerce reference application, and as such it does not implement many features that would be obvious and/or essential to a real eCommerce application.

> ### VERSIONS
> #### The `master` branch is currently running ASP.NET Core 2.1.
> #### Older versions are tagged.


## Topics (eBook TOC)

- Introduction
- Characteristics of Modern Web Applications
- Choosing Between Traditional Web Apps and SPAs
- Architectural Principles
- Common Web Application Architectures
- Common Client Side Technologies
- Developing ASP.NET Core MVC Apps
- Working with Data in ASP.NET Core Apps
- Testing ASP.NET Core MVC Apps
- Development Process for Azure-Hosted ASP.NET Core Apps
- Azure Hosting Recommendations for ASP.NET Core Web Apps

## Running the sample

After cloning or downloading the sample you should be able to run it using an In Memory database immediately.

If you wish to use the sample with a persistent database, you will need to run its Entity Framework Core migrations before you will be able to run the app, and update the `ConfigureServices` method in `Startup.cs` (see below).

You can also run the samples in Docker (see below).

### Configuring the sample to use SQL Server

1. Update `Startup.cs`'s `ConfigureDevelopmentServices` method as follows:

```
        public void ConfigureDevelopmentServices(IServiceCollection services)
        {
            // use in-memory database
            //ConfigureTestingServices(services);

            // use real database
            ConfigureProductionServices(services);

        }
```

1. Ensure your connection strings in `appsettings.json` point to a local SQL Server instance.

2. Open a command prompt in the Web folder and execute the following commands:

```
dotnet restore
dotnet ef database update -c catalogcontext -p ../Infrastructure/Infrastructure.csproj -s Web.csproj
dotnet ef database update -c appidentitydbcontext -p ../Infrastructure/Infrastructure.csproj -s Web.csproj
```

These commands will create two separate databases, one for the store's catalog data and shopping cart information, and one for the app's user credentials and identity data.

3. Run the application.
The first time you run the application, it will seed both databases with data such that you should see products in the store, and you should be able to log in using the demouser@microsoft.com account.

Note: If you need to create migrations, you can use these commands:
```
-- create migration (from Web folder CLI)
dotnet ef migrations add InitialModel --context catalogcontext -p ../Infrastructure/Infrastructure.csproj -s Web.csproj -o Data/Migrations

dotnet ef migrations add InitialIdentityModel --context appidentitydbcontext -p ../Infrastructure/Infrastructure.csproj -s Web.csproj -o Identity/Migrations
```

## Running the sample using Docker

You can run both the Web and WebRazorPages samples at the same time by running these commands from the root folder (where the .sln file is located):

```
    docker-compose build
    docker-compose up
```

You should be able to make requests to localhost:5106 and localhost:5107 once these commands complete.

You can run just the Web or WebRazorPages application by using the instructions located in their respective `Dockerfile` files in the root of the projects. Again, run these commands from the root of the solution (where the .sln file is located).

## Running the sample on Kubernetes using Draft

1. Modify `draft.toml`, and set a unique name and namespace (optional) for your Kubernetes deployment
2. If you want to use a Kubernetes ingress controller, set the `basedomain` in the last line. Otherwise, you can delete it

```toml
[environments]
  [environments.development]
    name = "eshoponweb"
    namespace = "default"
    wait = true
    watch = false
    watch-delay = 2
    auto-connect = false
    dockerfile = "src/WebRazorPages/Dockerfile"
    chart = ""
    set = ["ingress.enabled=true", "basedomain=kkna.ogfg.link"]
```

3. Run `draft up`
4. Now, use `helm list` and `helm status` to check on the deployment. `helm status` will include a link to access your site.

```
helm list
NAME            REVISION        UPDATED                         STATUS          CHART                   APP VERSION     NAMESPACE
aspnetapp       4               Thu Dec  6 00:15:55 2018        DEPLOYED        aspnetapp-v0.0.1                        default
eshoponweb      3               Fri Dec  7 06:57:07 2018        DEPLOYED        eshoponweb-v0.0.1                       default
nginx-ingress   1               Thu Dec  6 22:49:06 2018        DEPLOYED        nginx-ingress-1.0.1     0.21.0          default
PS C:\repos\eShopOnWeb> helm status eshoponweb
LAST DEPLOYED: Fri Dec  7 06:57:07 2018
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/Service
NAME                   AGE
eshoponweb-eshoponweb  45m

==> v1/Deployment
eshoponweb-eshoponweb  45m

==> v1beta1/Ingress
eshoponweb-eshoponweb  45m

==> v1/Pod(related)

NAME                                    READY  STATUS   RESTARTS  AGE
eshoponweb-eshoponweb-547775c567-zpt7h  1/1    Running  0         4m


NOTES:

  http://eshoponweb.kkna.ogfg.link to access your application
```
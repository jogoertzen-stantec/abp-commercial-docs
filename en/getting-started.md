# Getting Started

````json
//[doc-params]
{
    "UI": ["MVC", "Blazor", "NG"],
    "DB": ["EF", "Mongo"],
    "Tiered": ["Yes", "No"]
}
````

> This tutorial assumes that you've already purchased an [ABP Commercial license](https://commercial.abp.io/pricing) and have an active ABP Commercial account.

This tutorial explains how to create a new web application using the startup template.

> This document assumes that you prefer to use **{{ UI_Value }}** as the UI framework and **{{ DB_Value }}** as the database provider. For other options, please change the preference on top of this document.

## Setup your development environment

First things first! Let's setup your development environment before creating the first project.

### Pre-requirements

The following tools should be installed on your development machine:

* [Visual Studio 2019](https://visualstudio.microsoft.com/vs/) for Windows / [Visual Studio for Mac](https://visualstudio.microsoft.com/vs/mac/). *
* [.NET Core 3.0+](https://www.microsoft.com/net/download/dotnet-core/)
* [Node v12+](https://nodejs.org)
* [Yarn v1.19+](https://yarnpkg.com/)
{{ if Tiered == "Yes" }}

* [Redis](https://redis.io/): The applications use Redis as as [distributed cache](../Caching.md). So, you need to have Redis installed & running.

{{ end }}

> You can use another editor instead of Visual Studio as long as it supports .NET Core and ASP.NET Core.

### Install the ABP CLI

[ABP CLI](https://docs.abp.io/en/abp/latest/CLI) is a command line interface that is used to automate some common tasks for ABP based solutions.

> ABP CLI is a free & open source tool for [the ABP framework](https://abp.io/). It is also used for ABP Commercial application development.

First, you need to install the ABP CLI using the following command:

````shell
dotnet tool install -g Volo.Abp.Cli
````
If you've already installed, you can update it using the following command:
````shell
dotnet tool update -g Volo.Abp.Cli
````
#### Login to your account

In order to use ABP Commercial features, you need to login your account using the ABP CLI:

````shell
abp login <username>
````

It will ask a password, so you must enter the password of your account.

> You can freely create a new account from the [ABP Account web site](https://account.abp.io/Account/Register). However, your account should be registered to an organization as a developer to be able to use the ABP Commercial. If your company has an ABP Commercial license, ask your manager to add your account to the developer list of the organization.

### Install the ABP Suite

[ABP Suite](abp-suite/index.md) is an application aims to assist you on your development.

First, you need to install the ABP Suite:

````shell
abp suite install
````

If you've already installed, you can update it:

````shell
abp suite update
````
## Create a new project

> This document assumes that you prefer to use **{{ UI_Value }}** as the UI framework and **{{ DB_Value }}** as the database provider. For other options, please change the preference on top of this document.

There are two ways of creating a new project: ABP Suite and ABP CLI.

### Using the ABP Suite to create a new project

Run the ABP Suite with the following command:

````bash
abp suite
````

This command starts the ABP Suite and opens in your default browser:

![suite-ui](images/suite-entrance-2.png)

Click to the **Create solution** button and fill the modal form:

![suite-new-project-modal](images/suite-new-project-modal.png)

Select the UI framework, Database provider and other option based on your requirements then click to the OK button.

> You can use different level of namespaces; e.g. BookStore, Acme.BookStore or Acme.Retail.BookStore. 

### Using the ABP CLI to create a new project

Use the `new` command of the ABP CLI to create a new project:

````shell
abp new Acme.BookStore -t app-pro{{if UI == "NG"}} -u angular{{else if UI == "Blazor"}} -u blazor{{end}}{{if DB == "Mongo"}} -d mongodb{{end}}{{if Tiered == "Yes" && UI != "NG"}} --tiered {{else if Tiered == "Yes" && UI == "NG"}}--separate-identity-server{{end}}
````

* `-t` argument specifies the [startup template](startup-templates/application/index.md) name. `app-pro` is the startup template that contains the essential [ABP Commercial Modules](https://commercial.abp.io/modules) pre-installed and configured for you.

{{ if UI == "NG" }}

* `-u` argument specifies the UI framework, `angular` in this case.

{{ if Tiered == "Yes" }}

{{ if UI == "MVC" }}
* `--tiered` argument is used to create N-tiered solution where authentication server, UI and API layers are physically separated.
{{ else }}
* `--separate-identity-server` argument is used to separate the identity server application from the API host application. If not specified, you will have a single endpoint on the server.
{{ end }}

{{ end }}

{{ end }}

{{ if DB == "Mongo" }}

* `-d` argument specifies the database provider, `mongodb` in this case.

{{ end }}

{{ if Tiered == "Yes" && UI != "NG" }}

* `--tiered` argument is used to create N-tiered solution where authentication server, UI and API layers are physically separated.

{{ end }}

> You can use different level of namespaces; e.g. BookStore, Acme.BookStore or Acme.Retail.BookStore. 

#### ABP CLI commands & options

[ABP CLI document](https://docs.abp.io/en/abp/latest/CLI) covers all of the available commands and options for the ABP CLI. The main difference for the ABP Commercial is the template names. See the [ABP Commercial Startup Templates](startup-templates/application/index.md) document for other commercial templates.

## The solution structure

The solution has a layered structure (based on the [Domain Driven Design](https://docs.abp.io/en/abp/Domain-Driven-Design)) and contains unit & integration test projects. See the [solution structure document](startup-templates/application/solution-structure.md) to understand the solution structure in details. 

{{ if DB == "Mongo" }}

#### MongoDB Transactions

The [startup template](Startup-templates/Index.md) **disables** transactions in the `.MongoDB` project by default. If your MongoDB server supports transactions, you can enable the it in the *YourProjectMongoDbModule* class:

  ```csharp
  Configure<AbpUnitOfWorkDefaultOptions>(options =>
  {
      options.TransactionBehavior = UnitOfWorkTransactionBehavior.Auto;
  });
  ```

> Or you can delete this code since this is already the default behavior.

{{ end }}

## Create the database

### Database connection string

Check the **connection string** in the `appsettings.json` file under the {{if Tiered == "Yes"}}`.IdentityServer` and `.HttpApi.Host` projects{{else}}{{if UI=="MVC"}}`.Web` project{{else}}`.HttpApi.Host` project{{end}}{{end}}

{{ if DB == "EF" }}

````json
"ConnectionStrings": {
  "Default": "Server=localhost;Database=BookStore;Trusted_Connection=True"
}
````

The solution is configured to use **Entity Framework Core** with **MS SQL Server** by default. EF Core supports [various](https://docs.microsoft.com/en-us/ef/core/providers/) database providers, so you can use any supported DBMS. See [the Entity Framework integration document](https://docs.abp.io/en/abp/latest/Entity-Framework-Core) to learn how to [switch to another DBMS](https://docs.abp.io/en/abp/latest/Entity-Framework-Core-Other-DBMS).

### Apply the migrations

The solution uses the [Entity Framework Core Code First Migrations](https://docs.microsoft.com/en-us/ef/core/managing-schemas/migrations/?tabs=dotnet-core-cli). So, you need to apply migrations to create the database. There are two ways of applying the database migrations.

#### Apply migrations using the DbMigrator

The solution comes with a `.DbMigrator` console application which applies migrations and also **seeds the initial data**. It is useful on **development** as well as on **production** environment.

> `.DbMigrator` project has its own `appsettings.json`. So, if you have changed the connection string above, you should also change this one. 

Right click to the `.DbMigrator` project and select **Set as StartUp Project**

![set-as-startup-project](images/set-as-startup-project.png)

 Hit F5 (or Ctrl+F5) to run the application. It will have an output like shown below:

 ![db-migrator-output](images/db-migrator-output.png)

> Initial [seed data](https://docs.abp.io/en/abp/latest/Data-Seeding) creates the `admin` user in the database (with the password is `1q2w3E*`) which is then used to login to the application. So, you need to use `.DbMigrator` at least once for a new database.

#### Using EF Core Update-Database command

Ef Core has `Update-Database` command which creates database if necessary and applies pending migrations.

{{ if UI == "MVC" }}

Right click to the {{if Tiered == "Yes"}}`.IdentityServer`{{else}}`.Web`{{end}} project and select **Set as StartUp project**: 

{{ else if UI != "MVC" }}

Right click to the `.HttpApi.Host` project and select **Set as StartUp Project**: 

{{ end }}

![set-as-startup-project](images/set-as-startup-project.png)

Open the **Package Manager Console**, select `.EntityFrameworkCore.DbMigrations` project as the **Default Project** and run the `Update-Database` command:

![package-manager-console-update-database](images/package-manager-console-update-database.png)

This will create a new database based on the configured connection string.

> **Using the `.DbMigrator` tool is the suggested way**, because it also seeds the initial data to be able to properly run the web application.
>
> If you just use the `Update-Database` command, you will have an empty database, so you can not login to the application since there is no initial admin user in the database. You can use the `Update-Database` command in development time when you don't need to seed the database. However, using the `.DbMigrator` application is easier and you can always use it to migrate the schema and seed the database.

{{ else if DB == "Mongo" }}

````json
"ConnectionStrings": {
  "Default": "mongodb://localhost:27017/BookStore"
}
````

The solution is configured to use **MongoDB** in your local computer, so you need to have a MongoDB server instance up and running or change the connection string to another MongoDB server.

### Seed initial data

The solution comes with a `.DbMigrator` console application which **seeds the initial data**. It is useful on **development** as well as on **production** environment.

> `.DbMigrator` project has its own `appsettings.json`. So, if you have changed the connection string above, you should also change this one. 

Right click to the `.DbMigrator` project and select **Set as StartUp Project**

![set-as-startup-project](images/set-as-startup-project.png)

 Hit F5 (or Ctrl+F5) to run the application. It will have an output like shown below:

 ![db-migrator-output](images/db-migrator-output.png)

> Initial [seed data](https://docs.abp.io/en/abp/latest/Data-Seeding) creates the `admin` user in the database (with the password is `1q2w3E*`) which is then used to login to the application. So, you need to use `.DbMigrator` at least once for a new database.

{{ end }}

## Run the application

{{ if UI == "MVC" }}

{{ if Tiered == "Yes" }}

1. Ensure that the `.IdentityServer` project is the startup project. Run this application that will open a **login** page in your browser.

> Use Ctrl+F5 in Visual Studio (instead of F5) to run the application without debugging. If you don't have a debug purpose, this will be faster.

You can login, but you cannot enter to the main application here. This is **just the authentication server**.

2. Ensure that the `.HttpApi.Host` project is the startup project and run the application which will open a **Swagger UI** in your browser.

![swagger-ui](images/swagger-ui.png)

This is the HTTP API that is used by the web application.

3. Lastly, ensure that the `.Web` project is the startup project and run the application which will open a **welcome** page in your browser

![mvc-tiered-app-home](images/mvc-tiered-app-home.png)

Click to the **login** button which will redirect you to the *authentication server* to login to the application:

![bookstore-login](images/bookstore-login-2.png)

{{ else # Tiered != "Yes" }}

Ensure that the `.Web` project is the startup project. Run the application which will open the **login** page in your browser:

> Use Ctrl+F5 in Visual Studio (instead of F5) to run the application without debugging. If you don't have a debug purpose, this will be faster.

![bookstore-login](images/bookstore-login-2.png)

{{ end # Tiered }}

{{ else # UI != "MVC" }}

#### Running the HTTP API Host (Server Side)

{{ if Tiered == "Yes" }}

Ensure that the `.IdentityServer` project is the startup project. Run the application which will open a **login** page in your browser.

> Use Ctrl+F5 in Visual Studio (instead of F5) to run the application without debugging. If you don't have a debug purpose, this will be faster.

You can login, but you cannot enter to the main application here. This is just the authentication server.

Ensure that the `.HttpApi.Host` project is the startup project and run the application which will open a Swagger UI:

{{ else # Tiered == "No" }}

Ensure that the `.HttpApi.Host` project is the startup project and run the application which will open a Swagger UI:

> Use Ctrl+F5 in Visual Studio (instead of F5) to run the application without debugging. If you don't have a debug purpose, this will be faster.

{{ end # Tiered }}

![swagger-ui](images/swagger-ui.png)

You can see the application APIs and test them here. Get [more info](https://swagger.io/tools/swagger-ui/) about the Swagger UI.

> ##### Authorization for the Swagger UI
>
> Most of the HTTP APIs require authentication & authorization. If you want to test authorized APIs, manually go to the `/Account/Login` page, enter `admin` as the username and `1q2w3E*` as the password to login to the application. Then you will be able to execute authorized APIs too.

{{ end # UI }}

{{ if UI == "Blazor" }}

### Running the Blazor Application (Client Side)

Ensure that the `.Blazor` project is the startup project and run the application.

> Use Ctrl+F5 in Visual Studio (instead of F5) to run the application without debugging. If you don't have a debug purpose, this will be faster.

Once the application starts, click to the **Login** link on to header, which redirects you to the authentication server to enter a username and password:

![bookstore-login](images/bookstore-login-2.png)

{{ else if UI == "NG" }}

### Running the Angular Application (Client Side)

Go to the `angular` folder, open a command line terminal, type the `yarn` command (we suggest to the [yarn](https://yarnpkg.com/) package manager while `npm install` will also work)

```bash
yarn
```

Once all node modules are loaded, execute `yarn start` (or `npm start`) command:

```bash
yarn start
```

It may take a longer time for the first build. Once it finishes, it opens the Angular UI in your default browser with the [localhost:4200](http://localhost:4200/) address.



![bookstore-login](images/bookstore-login-2.png)

{{ end }}

Enter **admin** as the username and **1q2w3E*** as the password to login to the application. 

{{ if UI == "Blazor" }}
![bookstore-home](images/bookstore-blazor-home.png)
{{else}}
![bookstore-home](images/bookstore-home-2.png)
{{end}}

The application is up and running. You can start developing your application based on this startup template.

## Mobile Development

When you create a new application, the solution includes `react-native` folder by default. This is a basic [React Native](https://reactnative.dev/) startup template to develop mobile applications integrated to your ABP based backends.

If you don't plan to develop a mobile application with React Native, you can safely delete the `react-native` folder.

> You can specifying the `-m none` option to the ABP CLI to not create the `react-native` folder in the beginning.

See the [Getting Started with the React Native](getting-started-react-native.md) document to learn how to configure and run the React Native application.

## Next

[Web Application development tutorial](tutorials/book-store/part-1.md)
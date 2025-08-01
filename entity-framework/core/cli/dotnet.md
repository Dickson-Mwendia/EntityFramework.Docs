---
title: EF Core tools reference (.NET CLI) - EF Core
description: Reference guide for the Entity Framework Core .NET CLI tools
author: SamMonoRT
ms.date: 11/08/2024
uid: core/cli/dotnet
---

# Entity Framework Core tools reference - .NET CLI

The command-line interface (CLI) tools for Entity Framework Core perform design-time development tasks. For example, they create [migrations](/aspnet/core/data/ef-mvc/migrations), apply migrations, and generate code for a model based on an existing database. The commands are an extension to the cross-platform [dotnet](/dotnet/core/tools) command, which is part of the [.NET SDK](https://www.microsoft.com/net/core). These tools work with .NET projects.

When using Visual Studio, consider using the [Package Manager Console tools](xref:core/cli/powershell) instead of the CLI tools. Package Manager Console tools automatically:

* Works with the current project selected in the **Package Manager Console** without requiring that you manually switch directories.
* Opens files generated by a command after the command is completed.
* Provides tab completion of commands, parameters, project names, context types, and migration names.

## Installing the tools

`dotnet ef` can be installed as either a global or local tool. Most developers prefer installing `dotnet ef` as a global tool using the following command:

```dotnetcli
dotnet tool install --global dotnet-ef
```

To use it as a local tool, restore the dependencies of a project that declares it as a tooling dependency using a [tool manifest file](/dotnet/core/tools/global-tools#install-a-local-tool).

Update the tool using the following command:

```dotnetcli
dotnet tool update --global dotnet-ef
```

Before you can use the tools on a specific project, you'll need to add the `Microsoft.EntityFrameworkCore.Design` package to it.

```dotnetcli
dotnet add package Microsoft.EntityFrameworkCore.Design
```

### Verify installation

Run the following commands to verify that EF Core CLI tools are correctly installed:

  ```dotnetcli
  dotnet ef
  ```

The output from the command identifies the version of the tools in use:

```output

                     _/\__
               ---==/    \\
         ___  ___   |.    \|\
        | __|| __|  |  )   \\\
        | _| | _|   \_/ |  //|\\
        |___||_|       /   \\\/\\

Entity Framework Core .NET Command-line Tools 2.1.3-rtm-32065

<Usage documentation follows, not shown.>
```

## Update the tools

Use `dotnet tool update --global dotnet-ef` to update the global tools to the latest available version. If you have the tools installed locally in your project use `dotnet tool update dotnet-ef`. Install a specific version by appending `--version <VERSION>` to your command. See the [Update](/dotnet/core/tools/dotnet-tool-update) section of the dotnet tool documentation for more details.

## Using the tools

Before using the tools, you might have to create a startup project or set the environment.

### Target project and startup project

The commands refer to a *project* and a *startup project*.

* The *project* is also known as the *target project* because it's where the commands add or remove files. By default, the project in the current directory is the target project. You can specify a different project as target project by using the <nobr>`--project`</nobr> option.

* The *startup project* is the one that the tools build and run. The tools have to execute application code at design time to get information about the project, such as the database connection string and the configuration of the model. By default, the project in the current directory is the startup project. You can specify a different project as startup project by using the <nobr>`--startup-project`</nobr> option.

The startup project and target project are often the same project. A typical scenario where they are separate projects is when:

* The EF Core context and entity classes are in a .NET class library.
* A .NET console app or web app references the class library.

It's also possible to [put migrations code in a class library separate from the EF Core context](xref:core/managing-schemas/migrations/projects).

### Other target frameworks

The CLI tools work with .NET projects and .NET Framework projects. Apps that have the EF Core model in a .NET Standard class library might not have a .NET or .NET Framework project. For example, this is true of Xamarin and Universal Windows Platform apps. In such cases, you can create a .NET console app project whose only purpose is to act as startup project for the tools. The project can be a dummy project with no real code &mdash; it is only needed to provide a target for the tooling.

> [!IMPORTANT]
> Xamarin.Android, Xamarin.iOS, Xamarin.Mac are now integrated directly into .NET (starting with .NET 6) as .NET for Android, .NET for iOS, and .NET for macOS. If you're building with these project types today, they should be upgraded to .NET SDK-style projects for continued support. For more information about upgrading Xamarin projects to .NET, see the [Upgrade from Xamarin to .NET & .NET MAUI](/dotnet/maui/migration) documentation.

Why is a dummy project required? As mentioned earlier, the tools have to execute application code at design time. To do that, they need to use the .NET runtime. When the EF Core model is in a project that targets .NET or .NET Framework, the EF Core tools borrow the runtime from the project. They can't do that if the EF Core model is in a .NET Standard class library. The .NET Standard is not an actual .NET implementation; it's a specification of a set of APIs that .NET implementations must support. Therefore .NET Standard is not sufficient for the EF Core tools to execute application code. The dummy project you create to use as startup project provides a concrete target platform into which the tools can load the .NET Standard class library.

### ASP.NET Core environment

You can specify [the environment](/aspnet/core/fundamentals/environments) for ASP.NET Core projects on the command-line. This and any additional arguments are passed into Program.CreateHostBuilder.

```dotnetcli
dotnet ef database update -- --environment Production
```

> [!TIP]
> The `--` token directs `dotnet ef` to treat everything that follows as an argument and not try to parse them as options. Any extra arguments not used by `dotnet ef` are forwarded to the app.

## Common options

| Option                                         | Short             | Description                                                                                                                                                                                                                                                   |
|:-----------------------------------------------|:------------------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| <nobr>`--json`</nobr>                          |                   | Show JSON output.                                                                                                                                                                                                                                             |
| <nobr>`--context <DBCONTEXT>`</nobr>           | <nobr>`-c`</nobr> | The `DbContext` class to use. Class name only or fully qualified with namespaces.  If this option is omitted, EF Core will find the context class. If there are multiple context classes, this option is required.                                            |
| <nobr>`--project <PROJECT>`</nobr>             | `-p`              | Relative path to the project folder of the target project.  Default value is the current folder.                                                                                                                                                              |
| <nobr>`--startup-project <PROJECT>`</nobr>     | `-s`              | Relative path to the project folder of the startup project. Default value is the current folder.                                                                                                                                                              |
| <nobr>`--framework <FRAMEWORK>`</nobr>         |                   | The [Target Framework Moniker](/dotnet/standard/frameworks#supported-target-framework-versions) for the [target framework](/dotnet/standard/frameworks).  Use when the project file specifies multiple target frameworks, and you want to select one of them. |
| <nobr>`--configuration <CONFIGURATION>`</nobr> |                   | The build configuration, for example: `Debug` or `Release`.                                                                                                                                                                                                   |
| <nobr>`--runtime <IDENTIFIER>`</nobr>          |                   | The identifier of the target runtime to restore packages for. For a list of Runtime Identifiers (RIDs), see the [RID catalog](/dotnet/core/rid-catalog).                                                                                                      |
| <nobr>`--no-build`</nobr>                      |                   | Don't build the project. Intended to be used when the build is up-to-date.                                                                                                                                                                                    |
| <nobr>`--help`</nobr>                          | `-h`              | Show help information.                                                                                                                                                                                                                                        |
| <nobr>`--verbose`</nobr>                       | `-v`              | Show verbose output.                                                                                                                                                                                                                                          |
| <nobr>`--no-color`</nobr>                      |                   | Don't colorize output.                                                                                                                                                                                                                                        |
| <nobr>`--prefix-output`</nobr>                 |                   | Prefix output with level.                                                                                                                                                                                                                                     |

Any additional arguments are passed to the application.

## `dotnet ef database drop`

Deletes the database.

Options:

| Option                   | Short             | Description                                              |
|:-------------------------|:------------------|:---------------------------------------------------------|
| <nobr>`--force`</nobr>   | <nobr>`-f`</nobr> | Don't confirm.                                           |
| <nobr>`--dry-run`</nobr> |                   | Show which database would be dropped, but don't drop it. |

The [common options](#common-options) are listed above.

## `dotnet ef database update`

Updates the database to the last migration or to a specified migration.

Arguments:

| Argument                   | Description                                                                                                                                                                                                                                                     |
|:---------------------------|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| <nobr>`<MIGRATION>`</nobr> | The target migration. Migrations may be identified by name or by ID. The number 0 is a special case that means *before the first migration* and causes all migrations to be reverted. If no migration is specified, the command defaults to the last migration. |

Options:

| Option                                    | Description                                                                                                                      |
|:------------------------------------------|:---------------------------------------------------------------------------------------------------------------------------------|
|  <nobr>`--connection <CONNECTION>`</nobr> | The connection string to the database. Defaults to the one specified in `AddDbContext` or `OnConfiguring`. |

The [common options](#common-options) are listed above.

The following examples update the database to a specified migration. The first uses the migration name and the second uses the migration ID and a specified connection:

```dotnetcli
dotnet ef database update InitialCreate
dotnet ef database update 20180904195021_InitialCreate --connection your_connection_string
```

## `dotnet ef dbcontext info`

Gets information about a `DbContext` type.

The [common options](#common-options) are listed above.

## `dotnet ef dbcontext list`

Lists available `DbContext` types.

The [common options](#common-options) are listed above.

<a name="optimize"></a>

## `dotnet ef dbcontext optimize`

Generates a compiled version of the model used by the `DbContext` and precompiles queries.

See [Compiled models](xref:core/performance/advanced-performance-topics#compiled-models) for more information.

Options:

| Option                                               | Short | Description                                                                                                                                                                    |
|:-----------------------------------------------------|:------|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| <nobr>`--output-dir <PATH>`</nobr>       | `-o`              | The directory to put files in. Paths are relative to the project directory.                                                                                       |
| <nobr>`--namespace <NAMESPACE>`</nobr>   | `-n`              | The namespace to use for all generated classes. Defaults to generated from the root namespace and the output directory plus `CompiledModels`.                                  |
| <nobr>`--suffix <SUFFIX>`</nobr>         |                   | The suffix to attach to the name of all the generated files. E.g. `.g` could be used to indicate that these files contain generated code                                            |
| <nobr>`--no-scaffold`</nobr>             |                   | Don't generate a compiled model. This is used when the compiled model has already been generated.                                                                       |
| <nobr>`--precompile-queries`</nobr>      |                   | Generate precompiled queries. This is required for NativeAOT compilation if the target project contains any queries                                                           |
| <nobr>`--nativeaot`</nobr>               |                   | Generate additional code in the compiled model required for NativeAOT compilation and precompiled queries                                                           |

> [!NOTE]
> NativeAOT support and precompiled queries are considered experimental in EF 9 and could change dramatically in the next release.

The [common options](#common-options) are listed above.

The following example uses the default settings and works if there is only one `DbContext` in the project:

```dotnetcli
dotnet ef dbcontext optimize
```

The following example optimizes the model for the context with the specified name and places it in a separate folder and namespace:

```dotnetcli
dotnet ef dbcontext optimize -o Models -n BlogModels -c BlogContext
```

## `dotnet ef dbcontext scaffold`

Generates code for a `DbContext` and entity types for a database. In order for this command to generate an entity type, the database table must have a primary key.

Arguments:

| Argument                    | Description                                                                                                                                                                                                             |
|:----------------------------|:------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| <nobr>`<CONNECTION>`</nobr> | The connection string to the database. For ASP.NET Core 2.x projects, the value can be *name=\<name of connection string>*. In that case the name comes from the configuration sources that are set up for the project. |
| `<PROVIDER>`                | The provider to use. Typically this is the name of the NuGet package, for example: `Microsoft.EntityFrameworkCore.SqlServer`.                                                                                           |

Options:

| Option                                         | Short             | Description                                                                                                                                                                                                                                                                                                                             |
|:-----------------------------------------------|:------------------|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| <nobr>`--data-annotations`</nobr>              | <nobr>`-d`</nobr> | Use attributes to configure the model (where possible). If this option is omitted, only the fluent API is used.                                                                                                                                                                                                                         |
| <nobr>`--context <NAME>`</nobr>                | `-c`              | The name of the `DbContext` class to generate.                                                                                                                                                                                                                                                                                          |
| <nobr>`--context-dir <PATH>`</nobr>            |                   | The directory to put the `DbContext` class file in. Paths are relative to the project directory. Namespaces are derived from the folder names.                                                                                                                                                                                          |
| <nobr>`--context-namespace <NAMESPACE>`</nobr> |                   | The namespace to use for the generated `DbContext` class. Note: overrides `--namespace`.                                                                                                                                                                                                                                                |
| <nobr>`--force`</nobr>                         | `-f`              | Overwrite existing files.                                                                                                                                                                                                                                                                                                               |
| <nobr>`--output-dir <PATH>`</nobr>             | `-o`              | The directory to put entity class files in. Paths are relative to the project directory.                                                                                                                                                                                                                                                |
| <nobr>`--namespace <NAMESPACE>`</nobr>         | `-n`              | The namespace to use for all generated classes. Defaults to generated from the root namespace and the output directory.                                                                                                                                                                                                                 |
| <nobr>`--schema <SCHEMA_NAME>...`</nobr>       |                   | The schemas of tables and views to generate entity types for. To specify multiple schemas, repeat `--schema` for each one. If this option is omitted, all schemas are included. If this option is used, then all tables and views in the schemas will be included in the model, even if they are not explicitly included using --table. |
| <nobr>`--table <TABLE_NAME>...`</nobr>         | `-t`              | The tables and views to generate entity types for. To specify multiple tables, repeat `-t` or `--table` for each one. Tables or views in a specific schema can be included using the 'schema.table' or 'schema.view' format. If this option is omitted, all tables and views are included.                                              |
| <nobr>`--use-database-names`</nobr>            |                   | Use table, view, sequence, and column names exactly as they appear in the database. If this option is omitted, database names are changed to more closely conform to C# name style conventions.                                                                                                                                         |
| <nobr>`--no-onconfiguring`</nobr>              |                   | Suppresses generation of the `OnConfiguring` method in the generated `DbContext` class.                                                                                                                                                                                                                                                 |
| <nobr>`--no-pluralize`</nobr>                  |                   | Don't use the pluralizer.                                                                                                                                                                                                                                                                                                               |

The [common options](#common-options) are listed above.

The following example scaffolds all schemas and tables and puts the new files in the *Models* folder.

```dotnetcli
dotnet ef dbcontext scaffold "Server=(localdb)\mssqllocaldb;Database=Blogging;Trusted_Connection=True;" Microsoft.EntityFrameworkCore.SqlServer -o Models
```

The following example scaffolds only selected tables and creates the context in a separate folder with a specified name and namespace:

```dotnetcli
dotnet ef dbcontext scaffold "Server=(localdb)\mssqllocaldb;Database=Blogging;Trusted_Connection=True;" Microsoft.EntityFrameworkCore.SqlServer -o Models -t Blog -t Post --context-dir Context -c BlogContext --context-namespace New.Namespace
```

The following example reads the connection string from the project's configuration set using the [Secret Manager tool](/aspnet/core/security/app-secrets#secret-manager).

```dotnetcli
dotnet user-secrets set ConnectionStrings:Blogging "Data Source=(localdb)\MSSQLLocalDB;Initial Catalog=Blogging"
dotnet ef dbcontext scaffold Name=ConnectionStrings:Blogging Microsoft.EntityFrameworkCore.SqlServer
```

The following example skips scaffolding an `OnConfiguring` method. This can be useful when you want to configure the DbContext outside of the class. For example, ASP.NET Core apps typically configure it in Startup.ConfigureServices.

```dotnetcli
dotnet ef dbcontext scaffold "Server=(localdb)\mssqllocaldb;Database=Blogging;Integrated Security=true;" Microsoft.EntityFrameworkCore.SqlServer --no-onconfiguring
```

## `dotnet ef dbcontext script`

Generates a SQL script from the DbContext. Bypasses any migrations.

Options:

| Option                         | Short             | Description                      |
| ------------------------------ | ----------------- | -------------------------------- |
| <nobr>`--output <FILE>`</nobr> | <nobr>`-o`</nobr> | The file to write the result to. |

The [common options](#common-options) are listed above.

## `dotnet ef migrations add`

Adds a new migration.

Arguments:

| Argument              | Description                |
|:----------------------|:---------------------------|
| <nobr>`<NAME>`</nobr> | The name of the migration. |

Options:

| Option                                 | Short             | Description                                                                                                            |
|:---------------------------------------|:------------------|:-----------------------------------------------------------------------------------------------------------------------|
| <nobr>`--output-dir <PATH>`</nobr>     | <nobr>`-o`</nobr> | The directory use to output the files. Paths are relative to the target project directory. Defaults to "Migrations".   |
| <nobr>`--namespace <NAMESPACE>`</nobr> | `-n`              | The namespace to use for the generated classes. Defaults to generated from the output directory.  |

The [common options](#common-options) are listed above.

## `dotnet ef migrations bundle`

Creates an executable to update the database.

Options:

Option                                               | Short             | Description
---------------------------------------------------- | ----------------- | -----------
`--output <FILE>`                                    | <nobr>`-o`</nobr> | The path of executable file to create.
`--force`                                            | `-f`              | Overwrite existing files.
`--self-contained`                                   |                   | Also bundle the .NET runtime so it doesn't need to be installed on the machine.
<nobr>`--target-runtime <RUNTIME_IDENTIFIER>`</nobr> | `-r`              | The target runtime to bundle for.

The [common options](#common-options) are listed above.

## `dotnet ef migrations has-pending-model-changes`

> [!NOTE]
> This command was added in EF Core 8.0.

Checks if any changes have been made to the model since the last migration.

Options:

The [common options](#common-options) are listed above.

## `dotnet ef migrations list`

Lists available migrations.

Options:

| Option                                   | Description                                                                                                                  |
| ---------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| <nobr>`--connection <CONNECTION>`</nobr> | The connection string to the database. Defaults to the one specified in AddDbContext or OnConfiguring. |
| <nobr>`--no-connect`</nobr>              | Don't connect to the database.                                                                          |

The [common options](#common-options) are listed above.

## `dotnet ef migrations remove`

Removes the last migration, rolling back the code changes that were done for the latest migration.

Options:

| Option                 | Short             | Description                                                                     |
|:-----------------------|:------------------|:--------------------------------------------------------------------------------|
| <nobr>`--force`</nobr> | <nobr>`-f`</nobr> | Revert the latest migration, rolling back both code and database changes that were done for the latest migration. Continues to roll back only the code changes if an error occurs while connecting to the database. |

The [common options](#common-options) are listed above.

## `dotnet ef migrations script`

Generates a SQL script from migrations.

Arguments:

| Argument              | Description                                                                                                                                                   |
|:----------------------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------|
| <nobr>`<FROM>`</nobr> | The starting migration. Migrations may be identified by name or by ID. The number 0 is a special case that means *before the first migration*. Defaults to 0. |
| <nobr>`<TO>`</nobr>   | The ending migration. Defaults to the last migration.                                                                                                         |

Options:

| Option                           | Short             | Description                                                        |
|:---------------------------------|:------------------|:-------------------------------------------------------------------|
| <nobr>`--output <FILE>`</nobr>   | <nobr>`-o`</nobr> | The file to write the script to.                                   |
| <nobr>`--idempotent`</nobr>      | `-i`              | Generate a script that can be used on a database at any migration. |
| <nobr>`--no-transactions`</nobr> |                   | Don't generate SQL transaction statements.   |

The [common options](#common-options) are listed above.

The following example creates a script for the InitialCreate migration:

```dotnetcli
dotnet ef migrations script 0 InitialCreate
```

The following example creates a script for all migrations after the InitialCreate migration.

```dotnetcli
dotnet ef migrations script 20180904195021_InitialCreate
```

## Additional resources

* [Migrations](xref:core/managing-schemas/migrations/index)
* [Reverse Engineering](xref:core/managing-schemas/scaffolding)
* [Compiled models](xref:core/performance/advanced-performance-topics#compiled-models)

.. _refEntityFrameworkQuickstart:
使用 EntityFramework Core 获取配置和操作数据
=================================================================

在之前的快速入门中，我们在代码中创建了客户端和作用域数据。
在启动时，IdentityServer 将此配置数据加载到内存中。
如果我们想修改这个配置数据，我们必须停止和启动 IdentityServer。

IdentityServer 还会生成临时数据，例如授权码、同意选择和刷新令牌。
默认情况下，这些也存储在内存中。

要将这些数据移动到在重新启动和跨多个 IdentityServer 实例之间持久的数据库中，我们可以使用 IdentityServer4 Entity Framework 库。

.. Note:: 除了手动配置 EF 支持外，还有一个 IdentityServer 模板可以使用 ``dotnet new is4ef`` 创建一个支持 EF 的新项目。

IdentityServer4.EntityFramework
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
``IdentityServer4.EntityFramework`` 使用以下 DbContext 实现所需的存储和服务：

    * ConfigurationDbContext —— 用于配置数据，例如客户端、资源和作用域
    * PersistedGrantDbContext —— 用于临时操作数据，例如授权码和刷新令牌

这些 context 适用于任何 Entity Framework Core 兼容的关系数据库。

您可以在 ``IdentityServer4.EntityFramework.Storage`` nuget 包中找到这些 context、它们的实体以及使用它们的 IdentityServer4 存储。

您可以在 ``IdentityServer4.EntityFramework`` 中找到在您的 IdentityServer 中注册它们的扩展方法，我们现在将这样做::

    dotnet add package IdentityServer4.EntityFramework

使用 SqlServer
^^^^^^^^^^^^^^^

对于本快速入门，我们将使用 Visual Studio 附带的 SQLServer 的 LocalDb 版本。
要将 SQL Server 支持添加到我们的 IdentityServer 项目，您需要以下 nuget 包::

    dotnet add package Microsoft.EntityFrameworkCore.SqlServer

数据库架构更改和使用 EF 迁移
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``IdentityServer4.EntityFramework.Storage`` 包包含从 IdentityServer 模型映射的实体类。
随着 IdentityServer 的模型发生变化， ``IdentityServer4.EntityFramework.Storage`` 中的实体类也会发生变化。
当您使用 ``IdentityServer4.EntityFramework.Storage`` 并随着时间的推移升级时，您需要负责数据库架构以及随着实体类的更改而对该架构进行必要的更改。
管理这些更改的一种方法是使用 `EF迁移 <https://docs.microsoft.com/en-us/ef/core/managing-schemas/migrations/index>`_，我们将在本快速入门中使用它。
如果迁移不是您的偏好，那么您可以用任何您认为合适的方式管理架构更改。

.. Note:: 您可以在 IdentityServer4.EntityFramework.Storage 存储库中找到 SqlServer 的 `最新 SQL 脚本 <https://github.com/IdentityServer/IdentityServer4/tree/main/src/EntityFramework.Storage/migrations/SqlServer/Migrations>`_。

配置存储
^^^^^^^^^^^^^^^^^^^^^^

要开始使用这些存储，您需要在 `Startup.cs` 的 ``ConfigureServices`` 方法中使用 ``AddConfigurationStore`` 和 ``AddOperationalStore`` 来替换对 ``AddInMemoryClients``、``AddInMemoryIdentityResources``、``AddInMemoryApiScopes``、``AddInMemoryApiResources`` 和 ``AddInMemoryPersistedGrants`` 的任何现有调用。

这些方法每个都需要一个 ``DbContextOptionsBuilder``，这意味着你的代码看起来像这样::

    var migrationsAssembly = typeof(Startup).GetTypeInfo().Assembly.GetName().Name;
    const string connectionString = @"Data Source=(LocalDb)\MSSQLLocalDB;database=IdentityServer4.Quickstart.EntityFramework-4.0.0;trusted_connection=yes;";

    services.AddIdentityServer()
        .AddTestUsers(TestUsers.Users)
        .AddConfigurationStore(options =>
        {
            options.ConfigureDbContext = b => b.UseSqlServer(connectionString,
                sql => sql.MigrationsAssembly(migrationsAssembly));
        })
        .AddOperationalStore(options =>
        {
            options.ConfigureDbContext = b => b.UseSqlServer(connectionString,
                sql => sql.MigrationsAssembly(migrationsAssembly));
        });

您可能需要将这些命名空间添加到文件中::

    using Microsoft.EntityFrameworkCore;
    using System.Reflection;


由于我们在本快速入门中使用 EF 迁移，因此调用 ``MigrationsAssembly`` 用于通知 Entity Framework 宿主项目将包含迁移代码。
这是必要的，因为宿主项目位于与包含 ``DbContext`` 类的程序集中不同的程序集中。

添加迁移
^^^^^^^^^^^^^^^^^

一旦 IdentityServer 被配置为使用 Entity Framework，我们需要生成一些迁移。

要创建迁移，您需要在您的机器上安装 Entity Framework Core CLI，和在 IdentityServer 中安装 ``Microsoft.EntityFrameworkCore.Design`` nuget 包::

    dotnet tool install --global dotnet-ef
    dotnet add package Microsoft.EntityFrameworkCore.Design

要创建迁移，请在 IdentityServer 项目目录中打开命令提示符并运行以下两个命令::

    dotnet ef migrations add InitialIdentityServerPersistedGrantDbMigration -c PersistedGrantDbContext -o Data/Migrations/IdentityServer/PersistedGrantDb
    dotnet ef migrations add InitialIdentityServerConfigurationDbMigration -c ConfigurationDbContext -o Data/Migrations/IdentityServer/ConfigurationDb

您现在应该在项目中看到一个 ``~/Data/Migrations/IdentityServer`` 文件夹，其中包含新创建的迁移的代码。

初始化数据库
^^^^^^^^^^^^^^^^^^^^^^^^^

现在我们有了迁移，我们可以编写代码来从迁移创建数据库。
我们还可以使用我们在之前的快速入门中已经定义的内存中配置数据为数据库种子。

.. Note:: 本快速入门中使用的方法用于轻松启动和运行 IdentityServer。 您应该设计适合您的架构的自己的数据库创建和维护策略。

在 `Startup.cs` 中添加这个方法来帮助初始化数据库::

    private void InitializeDatabase(IApplicationBuilder app)
    {
        using (var serviceScope = app.ApplicationServices.GetService<IServiceScopeFactory>().CreateScope())
        {
            serviceScope.ServiceProvider.GetRequiredService<PersistedGrantDbContext>().Database.Migrate();

            var context = serviceScope.ServiceProvider.GetRequiredService<ConfigurationDbContext>();
            context.Database.Migrate();
            if (!context.Clients.Any())
            {
                foreach (var client in Config.Clients)
                {
                    context.Clients.Add(client.ToEntity());
                }
                context.SaveChanges();
            }

            if (!context.IdentityResources.Any())
            {
                foreach (var resource in Config.IdentityResources)
                {
                    context.IdentityResources.Add(resource.ToEntity());
                }
                context.SaveChanges();
            }

            if (!context.ApiScopes.Any())
            {
                foreach (var resource in Config.ApiScopes)
                {
                    context.ApiScopes.Add(resource.ToEntity());
                }
                context.SaveChanges();
            }
        }
    }

上面的代码可能需要您将以下命名空间添加到您的文件中::

    using System.Linq;
    using IdentityServer4.EntityFramework.DbContexts;
    using IdentityServer4.EntityFramework.Mappers;

然后我们可以从 ``Configure`` 方法中调用它::

    public void Configure(IApplicationBuilder app)
    {
        // 这将进行初始数据库填充
        InitializeDatabase(app);

        // 已经在这里的其余代码
        // ...
    }

现在，如果您运行 IdentityServer 项目，则应创建数据库，并使用快速入门配置数据作为种子。
您应该能够使用 SQL Server Management Studio 或 Visual Studio 来连接和检查数据。

.. image:: images/ef_database.png

.. Note:: 上面的 ``InitializeDatabase`` 辅助 API 可以方便地为数据库设置种子，但是这种方法不适合每次应用程序运行时都执行。 填充数据库后，请考虑删除对 API 的调用。

运行客户端应用程序
^^^^^^^^^^^^^^^^^^^^^^^^^^^

您现在应该能够运行任何现有的客户端应用程序，并进行登录、获取令牌和调用 API —— 所有这些都基于数据库配置。

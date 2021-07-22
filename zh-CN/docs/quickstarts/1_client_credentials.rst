.. _refClientCredentialsQuickstart:
使用客户端凭据保护 API
==========================================
以下 Identity Server 4 快速入门提供了各种常见 IdentityServer 方案的分步说明。 
这些从绝对的基础开始，随着它们的进展变得更加复杂。 我们建议您按顺序进行操作。  

要查看完整列表，请转到 `IdentityServer4 快速入门概述 <https://identityserver4.readthedocs.io/en/latest/quickstarts/0_overview.html>`_ 。

第一个快速入门是使用 IdentityServer 保护 API 的最基本场景。 
在本快速入门中，您将定义一个 API 和一个用于访问它的客户端。 
客户端将使用其客户端 ID 和密钥从身份服务器请求访问令牌，然后使用该令牌获得对 API 的访问权限。

源代码
^^^^^^^^^^^
与所有这些快速入门一样，您可以在 `IdentityServer4 <https://github.com/IdentityServer/IdentityServer4/blob/main/samples>`_ 存储库中找到它的源代码。 本快速入门的项目是 `快速入门 #1: 使用客户端凭据保护 API <https://github.com/IdentityServer/IdentityServer4/tree/main/samples/Quickstarts/1_ClientCredentials>`_ 。

准备工作
^^^^^^^^^^^
dotnet CLI 的 IdentityServer 模板是快速入门的良好起点。
要安装模板，请打开控制台窗口并键入以下命令::

    dotnet new -i IdentityServer4.Templates

它们将用作各种教程的起点。

设置 ASP.NET Core 应用程序
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
首先为应用程序创建一个目录 —— 然后使用我们的模板创建一个包含基本 IdentityServer 设置的 ASP.NET Core 应用程序，例如::

    md quickstart
    cd quickstart

    md src
    cd src

    dotnet new is4empty -n IdentityServer

这将创建以下文件:

* ``IdentityServer.csproj`` —— 项目文件和 ``Properties\launchSettings.json`` 文件
* ``Program.cs`` 和 ``Startup.cs`` —— 主应用程序入口点
* ``Config.cs`` —— IdentityServer 资源和客户端配置文件

您现在可以使用您喜欢的文本编辑器来编辑或查看文件。 如果你想有 Visual Studio 支持，你可以添加这样的解决方案文件::

    cd ..
    dotnet new sln -n Quickstart

并让它添加您的 IdentityServer 项目（记住这个命令，因为我们将在下面创建其他项目）::

    dotnet sln add .\src\IdentityServer\IdentityServer.csproj

.. note:: 此模板中使用的协议是 ``https``，在 Kestrel 上运行时端口设置为 5001 或在 IISExpress 上运行时设置为随机端口。 您可以在 ``Properties\launchSettings.json`` 文件中更改它。 对于生产场景，您应该始终使用 ``https``。

定义 API Scope
^^^^^^^^^^^^^^^^^^^^^
API 是系统中要保护的资源。 
资源定义可以通过多种方式加载，您用于创建上述项目的模板显示了如何使用“代码即配置”方法。

已经为您创建了 Config.cs。 打开它，更新代码看起来像这样::

    public static class Config
    {
        public static IEnumerable<ApiScope> ApiScopes =>
            new List<ApiScope>
            {
                new ApiScope("api1", "My API")
            };
    }

(在 `这里 <https://github.com/IdentityServer/IdentityServer4/blob/main/samples/Quickstarts/1_ClientCredentials/src/IdentityServer/Config.cs>`_ 查看完整的文件).
	
.. note:: 如果您将在生产中使用它，那么为您的 API 提供一个逻辑名称很重要。 开发人员将使用它通过您的身份服务器连接到您的 api。 它应该用简单的术语向开发人员和用户描述您的 api。

定义客户端
^^^^^^^^^^^^^^^^^^^
下一步是定义一个客户端应用程序，我们将使用它来访问我们的新 API。

对于这种情况，客户端将没有交互式用户，而是使用 IdentityServer 的所谓的客户端密钥进行身份验证。

为此，添加客户端定义:: 

    public static IEnumerable<Client> Clients =>
        new List<Client>
        {
            new Client
            {
                ClientId = "client",

                // 没有交互式用户，使用 clientid/secret 进行身份验证
                AllowedGrantTypes = GrantTypes.ClientCredentials,

                // 用于身份验证的密钥
                ClientSecrets =
                {
                    new Secret("secret".Sha256())
                },

                // 客户端有权访问的范围
                AllowedScopes = { "api1" }
            }
        };

您可以将 ClientId 和 ClientSecret 视为应用程序本身的登录名和密码。  
向身份服务器标识您的应用程序，以便它知道哪个应用程序正在尝试连接到它。	

	
配置 IdentityServer
^^^^^^^^^^^^^^^^^^^^^^^^^^
加载资源和客户端定义发生在 `Startup.cs <https://github.com/IdentityServer/IdentityServer4/blob/main/samples/Quickstarts/1_ClientCredentials/src/IdentityServer/Startup.cs>`_ 中 —— 将代码更新为如下所示::

    public void ConfigureServices(IServiceCollection services)
    {
        var builder = services.AddIdentityServer()
            .AddDeveloperSigningCredential()        //这仅适用于没有证书可以使用的开发场景。
            .AddInMemoryApiScopes(Config.ApiScopes)
            .AddInMemoryClients(Config.Clients);

        // 为简洁起见省略
    }

就是这样 —— 现在应该配置您的身份服务器。 如果您运行服务器并将浏览器导航到 ``https://localhost:5001/.well-known/openid-configuration`` ，您应该会看到所谓的发现文档。 
发现文档是身份服务器中的标准端点。  您的客户端和 API 将使用发现文档来下载必要的配置数据。

.. image:: images/1_discovery.png

在第一次启动时，IdentityServer 会为你创建一个开发者签名密钥，它是一个名为 ``tempkey.jwk`` 的文件。
您不必将该文件签入您的源代码管理，如果它不存在，它将被重新创建。

添加 API
^^^^^^^^^^^^^
接下来，向您的解决方案添加 API。 

您可以使用 Visual Studio 中的 ASP.NET Core Web API 模板，也可以使用 .NET CLI 来创建 API 项目，就像我们在此处所做的那样。
从 ``src`` 文件夹中运行以下命令::

    dotnet new webapi -n Api

然后通过运行以下命令将其添加到解决方案中::

    cd ..
    dotnet sln add .\src\Api\Api.csproj

将 API 应用程序配置为仅在 ``https://localhost:6001`` 上运行。 您可以通过编辑 Properties 文件夹中的 `launchSettings.json <https://github.com/IdentityServer/IdentityServer4/blob/main/samples/Quickstarts/1_ClientCredentials/src/Api/Properties/launchSettings.json>`_ 文件来完成此操作。 将应用程序 URL 设置更改为::

    "applicationUrl": "https://localhost:6001"

控制器
--------------
添加一个名为 ``IdentityController`` 的新类::

    [Route("identity")]
    [Authorize]
    public class IdentityController : ControllerBase
    {
        [HttpGet]
        public IActionResult Get()
        {
            return new JsonResult(from c in User.Claims select new { c.Type, c.Value });
        }
    }

此控制器稍后将用于测试授权需求，以及通过 API 的眼睛可视化声明身份。

添加 Nuget 依赖项
-------------------------
为了使配置步骤工作，必须添加 nuget 包依赖项，在根目录中运行此命令::

    dotnet add .\\src\\api\\Api.csproj package Microsoft.AspNetCore.Authentication.JwtBearer

配置
-------------
最后一步是将身份验证服务添加到 DI（依赖注入）并将身份验证中间件添加到管道中。
这些将:

* 验证传入的令牌以确保它来自受信任的发行者
* 验证令牌是否有效与此 API 一起使用（又名 audience 受众）

将 `Startup` 更新为如下所示::

    public class Startup
    {
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddControllers();

            services.AddAuthentication("Bearer")
                .AddJwtBearer("Bearer", options =>
                {
                    options.Authority = "https://localhost:5001";

                    options.TokenValidationParameters = new TokenValidationParameters
                    {
                        ValidateAudience = false
                    };
                });
        }

        public void Configure(IApplicationBuilder app)
        {
            app.UseRouting();

            app.UseAuthentication();
            app.UseAuthorization();

            app.UseEndpoints(endpoints =>
            {
                endpoints.MapControllers();
            });
        }
    }

* ``AddAuthentication`` 将身份验证服务添加到 DI 并将 ``Bearer`` 配置为默认方案。 
* ``UseAuthentication`` 将身份验证中间件添加到管道中，因此每次调用主机时都会自动执行身份验证。
* ``UseAuthorization`` 添加授权中间件以确保匿名客户端无法访问我们的 API 端点。

在浏览器上导航到控制器 ``https://localhost:6001/identity`` 应该返回 401 状态代码。 
这意味着您的 API 需要凭证并且现在受 IdentityServer 保护。

.. note:: 如果您想知道为什么上面的代码禁用了受众验证，请查看 :ref:`这里 <refResources>` 以获得更深入的讨论。

创建客户端
^^^^^^^^^^^^^^^^^^^
最后一步是编写一个请求访问令牌的客户端，然后使用此令牌访问 API。 为此，在您的解决方案中添加一个控制台项目，记住在 ``src`` 中创建它::

    dotnet new console -n Client
    
然后和以前一样，使用::

    cd ..
    dotnet sln add .\src\Client\Client.csproj

IdentityServer 上的令牌端点实现了 OAuth 2.0 协议，您可以使用原始 HTTP 来访问它。 
但是，我们有一个名为 IdentityModel 的客户端库，它将协议交互封装在一个易于使用的 API 中。

将 ``IdentityModel`` NuGet 包添加到您的客户端。 
这可以通过 Visual Studio 的 Nuget 包管理器或 dotnet CLI 完成::

    cd src
    cd client
    dotnet add package IdentityModel

IdentityModel 包括一个与发现端点一起使用的客户端库。 这样你只需要知道 IdentityServer 的基地址 - 可以从元数据中读取实际的端点地址::

    // 从元数据中发现端点
    var client = new HttpClient();
    var disco = await client.GetDiscoveryDocumentAsync("https://localhost:5001");
    if (disco.IsError)
    {
        Console.WriteLine(disco.Error);
        return;
    }
.. note:: 如果您在连接时遇到错误，则可能是您正在运行 `https` 并且 ``localhost`` 的开发证书不受信任。 您可以运行 ``dotnet dev-certs https --trust`` 以信任开发证书。 这只需要做一次。

接下来，您可以使用发现文档中的信息向 IdentityServer 请求令牌以访问 ``api1``::

    // 请求令牌
    var tokenResponse = await client.RequestClientCredentialsTokenAsync(new ClientCredentialsTokenRequest
    {
        Address = disco.TokenEndpoint,

        ClientId = "client",
        ClientSecret = "secret",
        Scope = "api1"
    });
    
    if (tokenResponse.IsError)
    {
        Console.WriteLine(tokenResponse.Error);
        return;
    }

    Console.WriteLine(tokenResponse.Json);

（完整文件可以在 `这里 <https://github.com/IdentityServer/IdentityServer4/blob/main/samples/Quickstarts/1_ClientCredentials/src/Client/Program.cs>`_ 找到）

.. note:: 将访问令牌从控制台复制并粘贴到 `jwt.ms <https://jwt.ms>`_ 以检查原始令牌。

调用 API
^^^^^^^^^^^^^^^
要将访问令牌发送到 API，您通常使用 HTTP 授权标头。 这是使用 ``SetBearerToken`` 扩展方法完成的::

    // 调用api
    var apiClient = new HttpClient();
    apiClient.SetBearerToken(tokenResponse.AccessToken);

    var response = await apiClient.GetAsync("https://localhost:6001/identity");
    if (!response.IsSuccessStatusCode)
    {
        Console.WriteLine(response.StatusCode);
    }
    else
    {
        var content = await response.Content.ReadAsStringAsync();
        Console.WriteLine(JArray.Parse(content));
    }

（如果您在 Visual Studio 中，您可以右键单击解决方案并选择“多个启动项目”，并确保 Api 和 IdentityServer 将启动；然后运行解决方案；之后，要逐步执行客户端代码，您可以右键单击“客户端”项目并选择 调试... 启动新实例）。
输出应该是这样的:

.. image:: images/1_client_screenshot.png

.. note:: 默认情况下，访问令牌将包含有关范围、生命周期（nbf 和 exp）、客户端 ID (client_id) 和颁发者名称 (iss) 的声明。

API 授权
^^^^^^^^^^^^^^^^^^^^^^^^
现在，API 接受您的身份服务器颁发的任何访问令牌。

在下文中，我们将添加允许检查范围是否存在于客户端请求（并被授权）的访问令牌中的代码。
为此，我们将使用 ASP.NET Core 授权策略系统。 将以下内容添加到 ``Startup`` 中的 ``ConfigureServices`` 方法中::

    services.AddAuthorization(options =>
    {
        options.AddPolicy("ApiScope", policy =>
        {
            policy.RequireAuthenticatedUser();
            policy.RequireClaim("scope", "api1");
        });
    });

您现在可以在各个级别执行此策略，例如

* 全局
* 对于所有 API 端点
* 对特定的 controllers/actions

通常，您为路由系统中的所有 API 端点设置策略::

    app.UseEndpoints(endpoints =>
    {
        endpoints.MapControllers()
            .RequireAuthorization("ApiScope");
    });


进一步的实验
^^^^^^^^^^^^^^^^^^^
本演练重点介绍了迄今为止的成功路径

* 客户端能够请求令牌
* 客户端可以使用令牌访问 API

您现在可以尝试引发错误以了解系统的行为方式，例如

* 尝试在 IdentityServer 未运行时连接到它（不可用）
* 尝试使用无效的客户端 ID 或机密来请求令牌
* 尝试在令牌请求期间请求无效范围
* 尝试在未运行时调用 API（不可用）
* 不要将令牌发送到 API
* 将 API 配置为需要与令牌中的范围不同的范围

.. _refAspNetCoreAndApis:
ASP.NET Core 和 API 访问
===========================
在之前的快速入门中，我们探讨了 API 访问和用户身份验证。 
现在我们想把这两个部分放在一起。

OpenID Connect 和 OAuth 2.0 结合的美妙之处在于，您可以通过单个协议和与令牌服务的单个交换来实现。

到目前为止，我们只在令牌请求期间请求身份资源，一旦我们开始包括 API 资源，IdentityServer 将返回两个令牌：
包含身份验证和会话信息的身份令牌，以及代表登录用户访问 API 的访问令牌。

修改客户端配置
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
在 IdentityServer 中更新客户端配置很简单 —— 我们只需要将 ``api1`` 资源添加到允许的范围列表中。
此外，我们通过 ``AllowOfflineAccess`` 属性启用对刷新令牌的支持::

    new Client
    {
        ClientId = "mvc",
        ClientSecrets = { new Secret("secret".Sha256()) },

        AllowedGrantTypes = GrantTypes.Code,
                
        // 登录后重定向到哪里
        RedirectUris = { "https://localhost:5002/signin-oidc" },

        // 注销后重定向到哪里
        PostLogoutRedirectUris = { "https://localhost:5002/signout-callback-oidc" },
        
        AllowOfflineAccess = true,

        AllowedScopes = new List<string>
        {
            IdentityServerConstants.StandardScopes.OpenId,
            IdentityServerConstants.StandardScopes.Profile,
            "api1"
        }
    }

修改MVC客户端
^^^^^^^^^^^^^^^^^^^^^^^^
现在客户端要做的就是通过 scope 参数请求额外的资源。 这是在 OpenID Connect 处理程序配置中完成的::

    services.AddAuthentication(options =>
    {
        options.DefaultScheme = "Cookies";
        options.DefaultChallengeScheme = "oidc";
    })
        .AddCookie("Cookies")
        .AddOpenIdConnect("oidc", options =>
        {
            options.Authority = "https://localhost:5001";

            options.ClientId = "mvc";
            options.ClientSecret = "secret";
            options.ResponseType = "code";

            options.SaveTokens = true;

            options.Scope.Add("api1");
            options.Scope.Add("offline_access");
        });

由于启用了 ``SaveTokens``，ASP.NET Core 将自动将结果访问和刷新令牌存储在身份验证会话中。
您应该能够检查打印出您之前创建的会话内容的页面上的数据。

使用访问令牌
^^^^^^^^^^^^^^^^^^^^^^
您可以使用可以在 ``Microsoft.AspNetCore.Authentication`` 命名空间中找到的标准 ASP.NET Core 扩展方法 来访问会话中的令牌::

    var accessToken = await HttpContext.GetTokenAsync("access_token");

要使用访问令牌访问 API，您需要做的就是检索令牌，并将其设置在您的 HttpClient 上::

    public async Task<IActionResult> CallApi()
    {
        var accessToken = await HttpContext.GetTokenAsync("access_token");

        var client = new HttpClient();
        client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", accessToken);
        var content = await client.GetStringAsync("https://localhost:6001/identity");

        ViewBag.Json = JArray.Parse(content).ToString();
        return View("json");
    }

创建一个名为 json.cshtml 的视图，输出如下所示的 json::

    <pre>@ViewBag.Json</pre>

确保 API 正在运行，启动 MVC 客户端并在身份验证后调用 ``/home/CallApi`` 。

管理访问令牌
^^^^^^^^^^^^^^^^^^^^^^^^^
到目前为止，典型客户端最复杂的任务是管理访问令牌。 你通常想要 

* 在登录时请求访问和刷新令牌
* 缓存这些令牌
* 使用访问令牌调用 API，直到它过期
* 使用刷新令牌获取新的访问令牌
* 重新开始

ASP.NET Core 有许多内置工具可以帮助您完成这些任务（如缓存或 sessions），但仍有很多工作要做。 
随意看看 `这个 <https://github.com/IdentityModel/IdentityModel.AspNetCore>`_ 库，它可以自动执行许多样板任务。

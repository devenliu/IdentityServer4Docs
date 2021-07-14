.. _refExternalIdentityProviders:
使用外部身份提供商登录
========================================

ASP.NET Core 有一种灵活的方式来处理外部身份验证。 这涉及几个步骤。

.. Note:: 如果您使用的是 ASP.NET Identity，许多底层技术细节对您来说是隐藏的。 建议您还阅读 Microsoft `文档 <https://docs.microsoft.com/zh-cn/aspnet/core/security/authentication/social/>`_ 并执行 ASP.NET Identity :ref:`快速入门 <refAspNetIdentityQuickstart>`。

为外部提供商添加身份验证处理程序
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
与外部提供者对话所需的协议实现封装在 *authentication handler* 中。
一些提供商使用专有协议（例如 Facebook 等社交提供商），一些提供商使用标准协议，例如 OpenID Connect、WS-Federation 或 SAML2p。

有关添加外部身份验证和配置它的分步说明，请参阅此 :ref:`快速入门 <refExternalAuthenticationQuickstart>`。

Cookie 的作用
^^^^^^^^^^^^^^^^^^^
外部身份验证处理程序的一个选项称为 ``SignInScheme``，例如::

    services.AddAuthentication()
        .AddGoogle("Google", options =>
        {
            options.SignInScheme = "要使用的 cookie 处理程序方案";

            options.ClientId = "...";
            options.ClientSecret = "...";
        })

登录方案指定 将临时存储外部身份验证结果的 cookie 处理程序 的名称，
例如，由外部提供商发送的声明。 
这是必要的，因为在您完成外部身份验证过程之前，通常会涉及几个重定向。

鉴于这是一种常见的做法，IdentityServer 专门为此外部提供程序工作流注册了一个 cookie 处理程序。
该方案通过 ``IdentityServerConstants.ExternalCookieAuthenticationScheme`` 常量表示。
如果您要使用我们的外部 cookie 处理程序，那么对于上面的 ``SignInScheme``，您需要将值分配为 ``IdentityServerConstants.ExternalCookieAuthenticationScheme`` 常量::

    services.AddAuthentication()
        .AddGoogle("Google", options =>
        {
            options.SignInScheme = IdentityServerConstants.ExternalCookieAuthenticationScheme;

            options.ClientId = "...";
            options.ClientSecret = "...";
        })

您也可以注册自己的自定义 cookie 处理程序，如下所示::

    services.AddAuthentication()
        .AddCookie("你的自定义方案")
        .AddGoogle("Google", options =>
        {
            options.SignInScheme = "你的自定义方案";

            options.ClientId = "...";
            options.ClientSecret = "...";
        })

.. Note:: 对于特殊场景，您还可以短路外部 cookie 机制，将外部用户直接转发到主 cookie 处理程序。 这通常涉及处理外部处理程序上的事件，以确保您从外部身份源执行正确的声明转换。

触发身份验证处理程序
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
您可以通过 HttpContext 上的 ``ChallengeAsync`` 扩展方法（或使用 MVC ``ChallengeResult``）调用外部身份验证处理程序。

您通常希望将一些选项传递给 challenge 操作，例如 回调页面的路径和簿记提供者的名称，例如::

    var callbackUrl = Url.Action("ExternalLoginCallback");
    
    var props = new AuthenticationProperties
    {
        RedirectUri = callbackUrl,
        Items = 
        { 
            { "scheme", provider },
            { "returnUrl", returnUrl }
        }
    };
    
    return Challenge(provider, props);

处理回调并登录用户
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
在回调页面上，您的典型任务是：

* 检查外部提供者返回的身份。
* 决定如何与该用户打交道。 根据这是新用户还是回访用户的事实，这可能会有所不同。
* 新用户在被允许进入之前可能需要额外的步骤和 UI。
* 可能会创建一个链接到外部提供商的新内部用户帐户。
* 存储您想要保留的外部声明。
* 删除临时cookie
* 登录用户

**检查外部身份**::

    // 从临时 cookie 中读取外部身份
    var result = await HttpContext.AuthenticateAsync(IdentityServerConstants.ExternalCookieAuthenticationScheme);
    if (result?.Succeeded != true)
    {
        throw new Exception("外部认证错误");
    }

    // 检索外部用户的声明
    var externalUser = result.Principal;
    if (externalUser == null)
    {
        throw new Exception("外部认证错误");
    }

    // 检索外部用户的声明
    var claims = externalUser.Claims.ToList();

    // 尝试确定外部用户的唯一 ID —— 最常见的声明类型是 sub 声明和 NameIdentifier，
    // 具体取决于外部提供者，可能会使用其他一些声明类型
    var userIdClaim = claims.FirstOrDefault(x => x.Type == JwtClaimTypes.Subject);
    if (userIdClaim == null)
    {
        userIdClaim = claims.FirstOrDefault(x => x.Type == ClaimTypes.NameIdentifier);
    }
    if (userIdClaim == null)
    {
        throw new Exception("未知 userid");
    }
    
    var externalUserId = userIdClaim.Value;
    var externalProvider = userIdClaim.Issuer;

    // 使用 externalProvider 和 externalUserId 查找您的用户，或配置新用户

**清理和登录**::

    // 为用户颁发身份验证 cookie
    await HttpContext.SignInAsync(new IdentityServerUser(user.SubjectId) {
        DisplayName = user.Username,
        IdentityProvider = provider,
        AdditionalClaims = additionalClaims,
        AuthenticationTime = DateTime.Now
    });

    // 删除外部身份验证期间使用的临时 cookie
    await HttpContext.SignOutAsync(IdentityServerConstants.ExternalCookieAuthenticationScheme);

    // 验证返回 URL 并重定向回授权端点或本地页面
    if (_interaction.IsValidReturnUrl(returnUrl) || Url.IsLocalUrl(returnUrl))
    {
        return Redirect(returnUrl);
    }

    return Redirect("~/");

状态、URL 长度和 ISecureDataFormat
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
当重定向到外部提供程序进行登录时，来自客户端应用程序的状态必须经常往返。
这意味着在离开客户端之前捕获状态并保留直到用户返回到客户端应用程序。
许多协议，包括 OpenID Connect，允许将某种状态作为参数作为请求的一部分传递，身份提供者将在响应中返回该状态。
ASP.NET Core 提供的 OpenID Connect 身份验证处理程序利用了协议的这一特性，这就是它实现上述 ``returnUrl`` 特性的方式。

在请求参数中存储状态的问题是请求 URL 可能变得太大（超过 2000 个字符的常见限制）。
OpenID Connect 身份验证处理程序确实提供了一个可扩展点来将状态存储在您的服务器中，而不是存储在请求 URL 中。 
您可以通过实现 ``ISecureDataFormat<AuthenticationProperties>`` 并在 `OpenIdConnectOptions <https://github.com/aspnet/AspNetCore/blob/main/src/Security/Authentication/OpenIdConnect/src/OpenIdConnectOptions.cs#L249>`_ 上配置它来自己实现。

幸运的是，IdentityServer 为您提供了一个实现，由在 DI 容器中注册的 ``IDistributedCache`` 实现支持（例如标准的 ``MemoryDistributedCache``）。
要使用 IdentityServer 提供的安全数据格式实现，只需在配置 DI 时调用 ``IServiceCollection`` 上的 ``AddOidcStateDataFormatterCache`` 扩展方法。
如果没有传递参数，那么所有配置的 OpenID Connect 处理程序将使用 IdentityServer 提供的安全数据格式实现::

    public void ConfigureServices(IServiceCollection services)
    {
        // 配置 OpenIdConnect 处理程序以将状态参数持久化到服务器端 IDistributedCache。
        services.AddOidcStateDataFormatterCache();

        services.AddAuthentication()
            .AddOpenIdConnect("demoidsrv", "IdentityServer", options =>
            {
                // ...
            })
            .AddOpenIdConnect("aad", "Azure AD", options =>
            {
                // ...
            })
            .AddOpenIdConnect("adfs", "ADFS", options =>
            {
                // ...
            });
    }


如果只配置特定的方案，则将这些方案作为参数传递::

    public void ConfigureServices(IServiceCollection services)
    {
        // 配置 OpenIdConnect 处理程序以将状态参数持久化到服务器端 IDistributedCache。
        services.AddOidcStateDataFormatterCache("aad", "demoidsrv");

        services.AddAuthentication()
            .AddOpenIdConnect("demoidsrv", "IdentityServer", options =>
            {
                // ...
            })
            .AddOpenIdConnect("aad", "Azure AD", options =>
            {
                // ...
            })
            .AddOpenIdConnect("adfs", "ADFS", options =>
            {
                // ...
            });
    }


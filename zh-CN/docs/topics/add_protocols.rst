添加新协议
====================

除了对 OpenID Connect 和 OAuth 2.0 的内置支持之外，IdentityServer4 还允许添加对其他协议的支持。

您可以添加这些额外的协议端点作为中间件或使用例如 MVC 控制器。
在这两种情况下，您都可以访问 ASP.NET Core DI 系统，该系统允许重用我们的内部服务，例如访问客户端定义或密钥材料。

添加 WS-Federation 支持的示例可以在 `这里 <https://github.com/IdentityServer/IdentityServer4.WsFederation>`_ 找到。

典型的身份验证工作流程
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
身份验证请求通常是这样工作的：

* 身份验证请求到达协议端点
* 协议端点进行输入验证
* 重定向到登录页面，返回 URL 设置回协议端点（如果用户是匿名的）
    * 通过 ``IIdentityServerInteractionService`` 访问当前请求的详细信息
    * 用户身份验证（本地或通过外部身份验证中间件）
    * 用户登录
    * 重定向回协议端点
* 创建协议响应（令牌创建并重定向回客户端）

有用的 IdentityServer 服务
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
为了实现上述工作流程，需要与 IdentityServer 的一些交互点。

**访问配置并重定向到登录页面**

您可以通过注入 ``IdentityServerOptions`` 来访问 IdentityServer 配置
类到您的代码中。 这，例如 具有登录页面的配置路径::

    var returnUrl = Url.Action("Index");
    returnUrl = returnUrl.AddQueryString(Request.QueryString.Value);

    var loginUrl = _options.UserInteraction.LoginUrl;
    var url = loginUrl.AddQueryString(_options.UserInteraction.LoginReturnUrlParameter, returnUrl);

    return Redirect(url);

**登录页面与当前协议请求的交互**

``IIdentityServerInteractionService`` 支持将协议返回 URL 转换为经过解析和验证的上下文对象::

    var context = await _interaction.GetAuthorizationContextAsync(returnUrl);

默认情况下，交互服务仅理解 OpenID Connect 协议消息。
要扩展支持，您可以编写自己的 ``IReturnUrlParser``::

    public interface IReturnUrlParser
    {
        bool IsValidReturnUrl(string returnUrl);
        Task<AuthorizationRequest> ParseAsync(string returnUrl);
    }

..然后在DI中注册解析器::

    builder.Services.AddTransient<IReturnUrlParser, WsFederationReturnUrlParser>();

这允许登录页面获取诸如客户端配置和其他协议参数之类的信息。

**访问用于创建协议响应的配置和密钥材料**

通过将 ``IKeyMaterialService`` 注入您的代码，您可以访问配置的签名凭证和验证密钥::

    var credential = await _keys.GetSigningCredentialsAsync();
    var key = credential.Key as Microsoft.IdentityModel.Tokens.X509SecurityKey; 
        
    var descriptor = new SecurityTokenDescriptor
    {
        AppliesToAddress = result.Client.ClientId,
        Lifetime = new Lifetime(DateTime.UtcNow, DateTime.UtcNow.AddSeconds(result.Client.IdentityTokenLifetime)),
        ReplyToAddress = result.Client.RedirectUris.First(),
        SigningCredentials = new X509SigningCredentials(key.Certificate, result.RelyingParty.SignatureAlgorithm, result.RelyingParty.DigestAlgorithm),
        Subject = outgoingSubject,
        TokenIssuerName = _contextAccessor.HttpContext.GetIdentityServerIssuerUri(),
        TokenType = result.RelyingParty.TokenType
    };
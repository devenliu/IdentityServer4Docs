.. _refProtectingApis:
保护 API
===============
IdentityServer 默认以 `JWT <https://tools.ietf.org/html/rfc7519>`_ （JSON Web 令牌）格式发布访问令牌。

现在每个相关平台都支持验证 JWT 令牌，可以在 `此处 <https://jwt.io>`_ 找到一个很好的 JWT 库列表。
流行的类库如:

* 用于 ASP.NET Core 的 `JWT bearer 身份验证处理程序 <https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.JwtBearer/>`_
* 用于 Katana 的 `JWT bearer 身份验证中间件 <https://www.nuget.org/packages/Microsoft.Owin.Security.Jwt>`_

保护基于 ASP.NET Core 的 API 只是添加 JWT bearer 身份验证处理程序的问题::

    public class Startup
    {
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
                .AddJwtBearer(options =>
                {
                    // 您的 IdentityServer 的基地址
                    options.Authority = "https://demo.identityserver.io";

                    // 如果您使用的是 API 资源，可以在此处指定名称
                    options.Audience = "resource1";

                    // 默认情况下 IdentityServer 发出一个 typ 标头，建议额外检查
                    options.TokenValidationParameters.ValidTypes = new[] { "at+jwt" };
                });
        }
    }

.. note:: 如果您没有使用受众声明，您可以通过 ``options.TokenValidationParameters.ValidateAudience = false;`` 关闭受众检查。 有关资源、范围、受众和授权的更多信息，请参见 :ref:`此处 <refApiResources>`。

验证引用令牌
^^^^^^^^^^^^^^^^^^^^^^^^^^^
如果您使用引用令牌，则需要一个实现 `OAuth 2.0 令牌自省 <https://tools.ietf.org/html/rfc7662>`_ 的身份验证处理程序，例如 `这样 <https://github.com/IdentityModel/IdentityModel.AspNetCore.OAuth2Introspection>`_:: 

    services.AddAuthentication("token")
        .AddOAuth2Introspection("token", options =>
        {
            options.Authority = Constants.Authority;

            // 这映射到 API 资源名称和密钥
            options.ClientId = "resource1";
            options.ClientSecret = "secret";
        });

支持 JWT 和引用令牌
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
您可以设置 ASP.NET Core 以根据传入的令牌分派到正确的处理程序，有关更多信息，请参阅 `这篇 <https://leastprivilege.com/2020/07/06/flexible-access-token-validation-in-asp-net-core/>`_ 博客文章。
在这种情况下，您设置一个默认处理程序和一些转发逻辑，例如::

    services.AddAuthentication("token")

        // JWT 令牌
        .AddJwtBearer("token", options =>
        {
            options.Authority = Constants.Authority;
            options.Audience = "resource1";

            options.TokenValidationParameters.ValidTypes = new[] { "at+jwt" };

            // 如果令牌不包含点，则为引用令牌
            options.ForwardDefaultSelector = Selector.ForwardReferenceToken("introspection");
        })

        // 引用令牌
        .AddOAuth2Introspection("introspection", options =>
        {
            options.Authority = Constants.Authority;

            options.ClientId = "resource1";
            options.ClientSecret = "secret";
        });
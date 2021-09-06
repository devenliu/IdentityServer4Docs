添加更多 API 端点
=========================
向托管 IdentityServer 的应用程序添加额外的 API 端点是一种常见的情况。
这些端点通常由 IdentityServer 本身保护。

对于简单的场景，我们给你一些帮助。 请参阅高级部分以了解更多内部管道。

.. note:: 您可以通过使用我们的 ``IdentityServerAuthentication`` 处理程序或微软的 ``JwtBearer`` 处理程序来实现相同的目的。 但不建议这样做，因为它需要更多配置并创建对可能导致未来更新冲突的外部库的依赖。

首先将您的 API 注册为 ``ApiResource``，例如::

    public static IEnumerable<ApiResource> Apis = new List<ApiResource>
    {
        // 本地API
        new ApiResource(IdentityServerConstants.LocalApi.ScopeName),
    };

..并让您的客户访问此 API，例如::

    new Client
    {
        // 省略其余部分
        AllowedScopes = { IdentityServerConstants.LocalApi.ScopeName },   
    }

.. note:: ``IdentityServerConstants.LocalApi.ScopeName`` 的值为 ``IdentityServerApi``。

要为本地 API 启用令牌验证，请将以下内容添加到您的 IdentityServer 启动中::

    services.AddLocalApiAuthentication();

为了保护 API 控制器，使用 ``LocalApi.PolicyName`` 策略用 ``Authorize`` 属性修饰它::

    [Route("localApi")]
    [Authorize(LocalApi.PolicyName)]
    public class LocalApiController : ControllerBase
    {
        public IActionResult Get()
        {
            // 省略
        }
    }

然后，授权客户端可以为 ``IdentityServerApi`` 范围请求令牌并使用它来调用 API。

发现
^^^^^^^^^
如果需要，您还可以将端点添加到发现文档中，例如像这样::

    services.AddIdentityServer(options =>
    {
        options.Discovery.CustomEntries.Add("local_api", "~/localapi");
    })

高级
^^^^^^^^
在幕后， ``AddLocalApiAuthentication`` 助手做了几件事：

* 添加一个身份验证处理程序，它使用 IdentityServer 的内置令牌验证引擎来验证传入的令牌（此处理程序的名称是 ``IdentityServerAccessToken`` 或 ``IdentityServerConstants.LocalApi.AuthenticationScheme``）
* 将身份验证处理程序配置为在值为 ``IdentityServerApi`` 的访问令牌中要求范围声明
* 设置一个授权策略来检查值 ``IdentityServerApi`` 的范围声明

这涵盖了最常见的场景。 您可以通过以下方式自定义此行为：

* 通过调用 ``services.AddAuthentication().AddLocalApi(...)`` 自己添加身份验证处理程序
    * 通过这种方式，您可以自己指定所需的范围名称，或者（完全不指定范围）接受来自当前 IdentityServer 实例的任何令牌
* 使用自定义策略或代码在您的控制器中进行您自己的范围验证/授权，例如::

    services.AddAuthorization(options =>
    {
        options.AddPolicy(IdentityServerConstants.LocalApi.PolicyName, policy =>
        {
            policy.AddAuthenticationSchemes(IdentityServerConstants.LocalApi.AuthenticationScheme);
            policy.RequireAuthenticatedUser();
            // 定制需求
        });
    });

声明转换
^^^^^^^^^^^^^^^^^^^^^
您可以提供回调以在验证后转换传入令牌的声明。
要么使用辅助方法，例如::

    services.AddLocalApiAuthentication(principal =>
    {
        principal.Identities.First().AddClaim(new Claim("additional_claim", "additional_value"));

        return Task.FromResult(principal);
    });
    
...或者如果您手动添加身份验证处理程序，则在选项上实现事件。

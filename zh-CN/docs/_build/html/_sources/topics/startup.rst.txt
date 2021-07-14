.. _refStartup:
Startup
=======

IdentityServer 是中间件和服务的组合。
所有配置都在您的启动类中完成。

配置服务
^^^^^^^^^^^^^^^^^^^^
您可以通过调用将 IdentityServer 服务添加到 DI 系统::

    public void ConfigureServices(IServiceCollection services)
    {
        var builder = services.AddIdentityServer();
    }

您可以选择将选项传递到此调用中。 有关选项的详细信息，请参阅 :ref:`这里 <refOptions>`。

这将返回一个 builder 对象，该对象又具有许多连接附加服务的便捷方法。

.. _refStartupKeyMaterial:
密钥材料
^^^^^^^^^^^^
IdentityServer 支持 X.509 证书（原始文件和对 Windows 证书存储的引用）、RSA 密钥和用于令牌签名和验证的 EC 密钥。 
每个密钥都可以配置一个（兼容的）签名算法，例如 RS256、RS384、RS512、PS256、PS384、PS512、ES256、ES384 或 ES512。

您可以使用以下方法配置密钥材料：

* ``AddSigningCredential``
    添加一个签名密钥，为各种令牌创建/验证服务提供指定的密钥材料。
* ``AddDeveloperSigningCredential``
    在启动时创建临时密钥材料。 这是针对开发场景的。 默认情况下，生成的密钥将持久保存在本地目录中。
* ``AddValidationKey``
    添加用于验证令牌的密钥。 它们将由内部令牌验证器使用，并将显示在发现文档中。

内存配置存储
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

各种 “in-memory” 配置 API 允许从内存中的配置对象列表配置 IdentityServer。
这些 “in-memory” 集合可以在托管应用程序中进行硬编码，也可以从配置文件或数据库中动态加载。
但是，按照设计，这些集合仅在托管应用程序启动时创建。

这些配置 API 的使用旨在用于原型设计、开发和/或测试，其中不需要在运行时动态查询数据库以获取配置数据。
如果配置很少更改，或者在必须更改值时要求重新启动应用程序并不不方便，则这种配置样式也可能适用于生产场景。

* ``AddInMemoryClients``
    基于 ``Client`` 配置对象的内存集合注册 ``IClientStore`` 和 ``ICorsPolicyService`` 实现。
* ``AddInMemoryIdentityResources``
    基于 ``IdentityResource`` 配置对象的内存集合注册 ``IResourceStore`` 实现。
* ``AddInMemoryApiScopes``
    基于 ``ApiScope`` 配置对象的内存中集合注册 ``IResourceStore`` 实现。
* ``AddInMemoryApiResources``
    基于 ``ApiResource`` 配置对象的内存集合注册 ``IResourceStore`` 实现。

测试存储
^^^^^^^^^^^

``TestUser`` 类在 IdentityServer 中对用户、他们的凭据和声明进行建模。
``TestUser`` 的使用类似于 “in-memory” 存储的使用，因为它用于原型设计、开发和/或测试。
不建议在生产中使用 ``TestUser``。

* ``AddTestUsers``
    基于 ``TestUser`` 对象的集合注册 ``TestUserStore``。
    ``TestUserStore`` 由默认的快速入门 UI 使用。
    还注册了 ``IProfileService`` 和 ``IResourceOwnerPasswordValidator`` 的实现。

额外服务
^^^^^^^^^^^^^^^^^^^

* ``AddExtensionGrantValidator``
    添加用于扩展授权的 ``IExtensionGrantValidator`` 实现。

* ``AddSecretParser``
    添加用于解析客户端或 API 资源凭据的 ``ISecretParser`` 实现。

* ``AddSecretValidator``
    添加 ``ISecretValidator`` 实现，用于根据凭证存储验证客户端或 API 资源凭证。

* ``AddResourceOwnerValidator``
    添加 ``IResourceOwnerPasswordValidator`` 实现，用于验证资源所有者密码凭据授予类型的用户凭据。

* ``AddProfileService``
    添加 ``IProfileService`` 实现以连接到您的 :ref:`自定义用户配置文件存储 <refProfileService>`。
    ``DefaultProfileService`` 类提供了默认实现，该实现依赖于身份验证 cookie 作为在令牌中发布的唯一声明来源。

* ``AddAuthorizeInteractionResponseGenerator``
    添加 ``IAuthorizeInteractionResponseGenerator`` 实现，以自定义授权端点的逻辑，以便在必须向用户显示错误、登录、同意或任何其他自定义页面的 UI时使用。
    ``AuthorizeInteractionResponseGenerator`` 类提供了默认实现，因此如果您需要增强现有行为，请考虑从该现有类派生。

* ``AddCustomAuthorizeRequestValidator``
    添加 ``ICustomAuthorizeRequestValidator`` 实现，以在授权端点自定义请求参数验证。

* ``AddCustomTokenRequestValidator``
    添加 ``ICustomTokenRequestValidator`` 实现，以自定义令牌端点处的请求参数验证。

* ``AddRedirectUriValidator``
    添加 ``IRedirectUriValidator`` 实现，以自定义重定向 URI 验证。

* ``AddAppAuthRedirectUriValidator``
    添加符合 “AppAuth”（原生应用程序的 OAuth 2.0）的重定向 URI 验证器（进行严格验证，但也允许使用随机端口的 http://127.0.0.1）。

* ``AddJwtBearerClientAuthentication``
    添加对使用 JWT bearer 断言的客户端身份验证的支持。

* ``AddMutualTlsSecretValidators``
    为 mutual TLS 添加 X509 密钥验证器。

缓存
^^^^^^^

IdentityServer 经常使用客户端和资源配置数据。
如果这些数据是从数据库或其他外部存储加载的，那么频繁地重新加载相同的数据可能会很昂贵。

* ``AddInMemoryCaching``
    要使用下面描述的任何缓存，必须在 DI 中注册 ``ICache<T>`` 的实现。
    此 API 注册了一个基于 ASP.NET Core 的 ``MemoryCache`` 的默认内存中实现 ``ICache<T>`` 。

* ``AddClientStoreCache``
    注册一个 ``IClientStore`` 装饰器实现，它将维护一个 ``Client`` 配置对象的内存缓存。
    缓存持续时间可在 ``IdentityServerOptions``的 ``Caching`` 配置选项中配置。

* ``AddResourceStoreCache``
    注册一个 ``IResourceStore`` 装饰器实现，它将维护一个 ``IdentityResource`` 和 ``ApiResource`` 配置对象的内存缓存。
    缓存持续时间可在 ``IdentityServerOptions``的 ``Caching`` 配置选项中配置。

* ``AddCorsPolicyCache``
    注册一个 ``ICorsPolicyService`` 装饰器实现，它将维护 CORS 策略服务评估结果的内存缓存。
    缓存持续时间可在 ``IdentityServerOptions``的 ``Caching`` 配置选项中配置。

可以进一步自定义缓存：

默认缓存依赖于 ``ICache<T>`` 实现。
如果您希望为特定配置对象自定义缓存行为，您可以在依赖注入系统中替换此实现。

``ICache<T>`` 本身的默认实现依赖于.NET 提供的 ``IMemoryCache`` 接口（和 ``MemoryCache`` 实现）。
如果您希望自定义内存缓存行为，您可以替换依赖注入系统中的 ``IMemoryCache`` 实现。

配置管道
^^^^^^^^^^^^^^^^^^^^^^^^
您需要通过调用将 IdentityServer 添加到管道中::

    public void Configure(IApplicationBuilder app)
    {
        app.UseIdentityServer();
    }

.. note:: ``UseIdentityServer`` 包括对 ``UseAuthentication`` 的调用，因此不必同时具有这两个调用。

中间件没有额外的配置。

请注意，顺序在管道中很重要。
例如，您需要在实现登录屏幕的 UI 框架之前添加 IdentitySever。

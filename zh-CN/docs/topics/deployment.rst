部署
==========
您的身份服务器 `只是` 一个包含 IdentityServer 中间件的标准 ASP.NET Core 应用程序。
请先阅读有关发布和部署的 Microsoft 官方 `文档 <https://docs.microsoft.com/en-us/aspnet/core/publishing>`_
（尤其是有关负载平衡器和代理的 `部分 <https://docs.microsoft.com/en-us/aspnet/core/host-and-deploy/proxy-load-balancer?view=aspnetcore-2.2#scenarios-and-use-cases>`_）。

典型架构
^^^^^^^^^^^^^^^^^^^^
通常，您将设计 IdentityServer 部署以实现高可用性:

.. image:: images/deployment.png

IdentityServer 本身是无状态的，不需要服务器关联 —— 但需要在实例之间共享数据。

配置数据
^^^^^^^^^^^^^^^^^^
这通常包括:

* 资源
* 客户端
* 启动配置，例如 密钥材料、外部供应商设置等...

您存储该数据的方式取决于您的环境。 在配置数据很少更改的情况下，我们建议使用内存存储和代码或配置文件。

在高度动态的环境（例如 Saas）中，我们建议使用数据库或配置服务来动态加载配置。

IdentityServer 支持开箱即用的代码配置和配置文件（参见 `此处 <https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration>`_）。
对于数据库，我们提供对基于 `Entity Framework Core <https://github.com/IdentityServer/IdentityServer4.EntityFramework>`_ 的数据库的支持。

您还可以通过实现 ``IResourceStore`` 和 ``IClientStore`` 来构建自己的配置存储。

密钥材料
^^^^^^^^^^^^

启动配置的另一个重要部分是您的密钥材料，有关密钥材料和密码学的更多详细信息，请参见 :ref:`此处 <refCrypto>`。

操作数据
^^^^^^^^^^^^^^^^
对于某些操作，IdentityServer 需要一个持久化存储来保持状态，这包括:

* 颁发授权码
* 颁发引用令牌和刷新令牌
* 存储同意

您可以使用传统数据库来存储操作数据，也可以使用具有持久性功能（如 Redis）的缓存。
上面提到的 EF Core 实现也支持操作数据。

您还可以通过实现 ``IPersistedGrantStore`` 来实现对您自己的自定义存储机制的支持 —— 默认情况下，IdentityServer 注入内存版本。

ASP.NET Core 数据保护
^^^^^^^^^^^^^^^^^^^^^^^^^^^^
ASP.NET Core 本身需要共享密钥材料来保护敏感数据，如 cookie、状态字符串等。
请参阅官方文档 `这里 <https://docs.microsoft.com/en-us/aspnet/core/security/data-protection/>`_。

如果可能的话，您可以重用上述持久性存储之一，也可以使用一些简单的东西，例如共享文件。

ASP.NET Core 分布式缓存
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
某些组件依赖于 ASP.NET Core 分布式缓存。 为了在多服务器环境中工作，需要正确设置。
`官方文档 <https://docs.microsoft.com/en-us/aspnet/core/performance/caching/distributed/>`_ 描述了几个选项。

以下组件依赖于 ``IDistributedCache``:

* ``services.AddOidcStateDataFormatterCache()`` 配置 OpenIdConnect 处理程序以将状态参数持久化到服务器端 ``IDistributedCache``。
* ``DefaultReplayCache``
* ``DistributedDeviceFlowThrottlingService``
* ``DistributedCacheAuthorizationParametersMessageStore``

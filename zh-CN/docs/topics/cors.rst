.. _refCors:
CORS
====

IdentityServer 中的许多端点将通过基于 JavaScript 的客户端的 Ajax 调用进行访问。
鉴于 IdentityServer 很可能托管在与这些客户端不同的源上，这意味着 `跨源资源共享 <http://www.html5rocks.com/en/tutorials/cors/>`_ (CORS) 将需要进行配置。

基于客户端的 CORS 配置
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

配置 CORS 的一种方法是在 :ref:`客户端配置 <refClient>` 上使用 ``AllowedCorsOrigins`` 集合。
只需将客户端的源添加到集合中，IdentityServer 中的默认配置将参考这些值以允许来自源的跨源调用。

.. Note:: 确保在配置 CORS 时使用源（而不是 URL）。 例如： ``https://foo:123/`` 是一个 URL，而 ``https://foo:123`` 是一个来源。

如果您使用我们提供的“内存中”或基于 EF 的客户端配置，则将使用此默认 CORS 实现。
如果您定义自己的 ``IClientStore``，那么您将需要实现您自己的自定义 CORS 策略服务（见下文）。

自定义 Cors 策略服务
^^^^^^^^^^^^^^^^^^^^^^^^^^

IdentityServer 允许托管应用程序实现 ``ICorsPolicyService`` 以完全控制 CORS 策略。

要实现的单一方法是： ``Task<bool> IsOriginAllowedAsync(string origin)``。
如果允许 `origin`，则返回 `true`，否则返回 `false`。

实现后，只需在 DI 中注册实现，IdentityServer 就会使用您的自定义实现。

**DefaultCorsPolicyService**

如果您只是希望对一组允许的来源进行硬编码，那么您可以使用一个名为 ``DefaultCorsPolicyService`` 的预构建 ``ICorsPolicyService`` 实现。
这将在 DI 中配置为单例，并使用其 ``AllowedOrigins`` 集合进行硬编码，或将标志 ``AllowAll`` 设置为 ``true`` 以允许所有来源。
例如，在 ``ConfigureServices`` 中::

    services.AddSingleton<ICorsPolicyService>((container) => {
        var logger = container.GetRequiredService<ILogger<DefaultCorsPolicyService>>();
        return new DefaultCorsPolicyService(logger) {
            AllowedOrigins = { "https://foo", "https://bar" }
        };
    });

.. Note:: 谨慎使用 ``AllowAll``。


将 IdentityServer CORS 策略与 ASP.NET Cores CORS 策略混合使用
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

IdentityServer 使用来自 ASP.NET Core 的 CORS 中间件来提供其 CORS 实现。
托管 IdentityServer 的应用程序可能也需要 CORS 用于其自己的自定义端点。
一般来说，两者应该在同一个应用程序中一起工作。

您的代码应使用 ASP.NET Core 中记录的 CORS 功能，而不考虑 IdentityServer。
这意味着您应该像往常一样定义策略并注册中间件。
如果您的应用程序在 ``ConfigureServices`` 中定义了策略，那么这些策略应该在您使用它们的相同位置继续工作（在您配置 CORS 中间件的位置或在您的控制器代码中使用 MVC ``EnableCors`` 属性的位置） ）。
相反，如果您在使用 CORS 中间件时定义内联策略（通过策略构建器回调），那么它也应该继续正常工作。

使用 ASP.NET Core CORS 服务和 IdentityServer 之间可能存在冲突的一种情况是，如果您决定创建自定义 ``ICorsPolicyProvider``。
鉴于 ASP.NET Core 的 CORS 服务和中间件的设计，IdentityServer 实现了自己的自定义 ``ICorsPolicyProvider`` 并将其注册到 DI 系统中。
幸运的是，IdentityServer 实现旨在使用装饰器模式来包装任何已在 DI 中注册的现有 ``ICorsPolicyProvider``。
这意味着您还可以实现 ``ICorsPolicyProvider``，但它只需要在 DI 中的 IdentityServer 之前注册（例如在 ``ConfigureServices`` 中）。
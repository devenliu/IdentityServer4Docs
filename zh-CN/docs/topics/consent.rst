.. _refConsent:
同意
=======

在授权请求期间，如果 IdentityServer 需要用户同意，浏览器将被重定向到同意页面。

同意用于允许最终用户授予客户端对资源（:ref:`identity <refIdentityResource>` 或 :ref:`API <refApiResource>`）的访问权限。
这通常只对第三方客户端是必需的，并且可以在 :ref:`客户端设置 <refClient>` 上为每个客户端启用/禁用。

同意页面
^^^^^^^^^^^^
为了让用户同意，托管应用程序必须提供同意页面。
`快速启动 <https://github.com/IdentityServer/IdentityServer4.Quickstart.UI>`_ 有一个同意页面的基本实现。

同意页面通常会呈现当前用户的显示名称，
请求访问的客户端的显示名称，
客户端的 Logo，
有关客户端的更多信息的链接
以及客户端请求访问的资源列表。
允许用户表明他们的同意应该被“记住”也是很常见的，这样以后就不会为同一客户端再次提示他们了。

一旦用户提供了同意，同意页面必须通知 IdentityServer 同意，然后浏览器必须重定向回授权端点。

授权上下文
^^^^^^^^^^^^^^^^^^^^^

IdentityServer 将一个 `returnUrl` 参数（可在 :ref:`用户交互选项 <refOptions>` 上配置）传递到包含授权请求参数的同意页面。
这些参数提供同意页面的上下文，可以在 :ref:`交互服务 <refInteractionService>` 的帮助下读取。
``GetAuthorizationContextAsync`` API 将返回一个 ``AuthorizationRequest`` 的实例。

可以使用 ``IClientStore`` 和 ``IResourceStore`` 接口获取有关客户端或资源的其他详细信息。

将同意结果通知 IdentityServer
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

:ref:`交互服务 <refInteractionService>` 上的 ``GrantConsentAsync`` API 允许同意页面通知 IdentityServer 同意的结果（也可能是拒绝客户端访问）。

IdentityServer 将暂时保留同意的结果。
默认情况下，此持久性使用 cookie，因为它只需要持续足够长的时间即可将结果传送回授权端点。
这种临时持久性不同于用于“记住我的同意”功能的持久性（授权端点为用户维持“记住我的同意”）。
如果您希望在同意页面和授权重定向之间使用其他持久性，那么您可以实现 ``IMessageStore<ConsentResponse>`` 并在 DI 中注册实现。

将用户返回到授权端点
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
一旦同意页面将结果通知 IdentityServer，用户就可以被重定向回 `returnUrl`。
您的同意页面应通过验证 `returnUrl` 是否有效来防止打开重定向。
这可以通过在 :ref:`交互服务 <refInteractionService>` 上调用 ``IsValidReturnUrl`` 来完成。
此外，如果 ``GetAuthorizationContextAsync`` 返回非空结果，那么您也可以相信 `returnUrl` 是有效的。




.. _refSignOut:
注销
========

注销 IdentityServer 就像删除身份验证 cookie 一样简单，但是要进行完整的联合注销，我们还必须考虑将用户从客户端应用程序（甚至可能是上游身份提供商）中注销。

删除身份验证 cookie
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

要删除身份验证 cookie，只需在 ``HttpContext`` 上使用 ``SignOutAsync`` 扩展方法。
您将需要传递使用的方案（由 ``IdentityServerConstants.DefaultCookieAuthenticationScheme`` 提供，除非您已更改它）::

    await HttpContext.SignOutAsync(IdentityServerConstants.DefaultCookieAuthenticationScheme);

或者您可以使用 IdentityServer 提供的便捷扩展方法::

    await HttpContext.SignOutAsync();

.. Note:: 通常，您应该提示用户注销（意味着需要 POST），否则攻击者可能会热链接到您的注销页面，从而导致用户自动注销。

通知客户端用户已注销
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

作为注销过程的一部分，您需要确保通知客户端应用程序用户已注销。
IdentityServer 支持服务器测客户端（例如 MVC）的 `前端通道 <https://openid.net/specs/openid-connect-frontchannel-1_0.html>`_ 规范，
服务器端客户端（例如 MVC）的 `后端通道 <https:/ /openid.net/specs/openid-connect-backchannel-1_0.html>`_ 规范，
以及基于浏览器的 JavaScript 客户端（例如 SPA、React、Angular 等）的 `会话管理 <https://openid.net/specs/openid-connect-session-1_0.html>`_ 规范。

**前端通道 服务器端客户端**

要通过前端通道规范从服务器端客户端应用程序中注销用户，IdentityServer 中的 ``注销`` 页面必须呈现一个 ``<iframe>`` 以通知客户端用户已注销。
希望收到通知的客户端必须设置 ``FrontChannelLogoutUri`` 配置值。
IdentityServer 跟踪用户登录了哪些客户端，并在 ``IIdentityServerInteractionService`` （:ref:`详情 <refInteractionService>`）上提供了一个名为 ``GetLogoutContextAsync`` 的 API。
此 API 返回一个带有 ``SignOutIFrameUrl`` 属性的 ``LogoutRequest`` 对象，您的注销页面必须将其呈现到 ``<iframe>`` 中。

**后端通道 服务器端客户端**

要通过后端通道规范从服务器端客户端应用程序注销用户，可以使用 ``IBackChannelLogoutService`` 服务。 
当您的注销页面通过调用 ``HttpContext.SignOutAsync`` 删除用户的身份验证 cookie 时，IdentityServer 将自动使用此服务。
希望收到通知的客户端必须设置 ``BackChannelLogoutUri`` 配置值。

**基于浏览器的 JavaScript 客户端**

鉴于 `会话管理 <https://openid.net/specs/openid-connect-session-1_0.html>`_  规范是如何设计的，在 IdentityServer 中，您不需要做什么特殊的事情来通知这些客户端用户已注销。
但是，客户端必须对 `check_session_iframe` 执行监控，这是由 `oidc-client JavaScript 库 <https://github.com/IdentityModel/oidc-client-js/>`_ 实现的。

由客户端应用程序发起的注销
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

如果注销是由客户端应用程序发起的，那么客户端首先将用户重定向到 :ref:`结束会话端点  <refEndSession>`。
结束会话端点的处理可能需要在重定向到注销页面的过程中维护一些临时状态（例如，客户端的注销后重定向 uri）。
此状态可能对注销页面有用，并且该状态的标识符通过 `logoutId` 参数传递到注销页面。

:ref:`交互服务 <refInteractionService>` 上的 ``GetLogoutContextAsync`` API 可用于加载状态。
对 ``LogoutRequest`` 模型上下文类感兴趣的是 ``ShowSignoutPrompt``，它指示注销请求是否已通过身份验证，因此不提示用户注销是安全的。

默认情况下，此状态作为通过 ``logoutId`` 值传递的受保护数据结构进行管理。
如果您希望在结束会话端点和注销页面之间使用其他持久性，那么您可以实现 ``IMessageStore<LogoutMessage>`` 并在 DI 中注册实现。

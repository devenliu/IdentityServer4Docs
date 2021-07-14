.. _refSignIn:
登录
=======

为了让 IdentityServer 代表用户颁发令牌，该用户必须登录到 IdentityServer。

Cookie 认证
^^^^^^^^^^^^^^^^^^^^^
使用由 ASP.NET Core 的 `cookie 身份验证 <https://docs.microsoft.com/en-us/aspnet/core/security/authentication/cookie>`_ 处理程序管理的 cookie 跟踪身份验证。

IdentityServer 注册了两个 cookie 处理程序（一个用于身份验证会话，另一个用于临时外部 cookie）。
 默认情况下使用它们，如果您想手动引用它们，您可以从 ``IdentityServerConstants`` 类（ ``DefaultCookieAuthenticationScheme`` 和 ``ExternalCookieAuthenticationScheme`` ）中获取它们的名称。

仅公开这些 cookie 的基本设置（过期和滑动），但如果您需要更多控制，您可以注册自己的 cookie 处理程序。
当使用来自 ASP.NET Core 的 ``AddAuthentication`` 时，IdentityServer 使用与 ``AuthenticationOptions`` 上配置的 ``DefaultAuthenticateScheme`` 相匹配的任何 cookie 处理程序。

.. note:: 除了身份验证 cookie 之外，IdentityServer 还将发布一个额外的 cookie，默认名称为“idsrv.session”。 此 cookie 派生自 主身份验证 cookie，它用于 :ref:`注销时基于浏览器的 JavaScript 客户端 <refSignOut>` 的检查会话端点。 它与身份验证 cookie 保持同步，并在用户注销时删除。

覆盖 cookie 处理程序配置
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
如果您希望使用自己的 cookie 身份验证处理程序，则必须自己配置它。
这必须在 DI 中注册 IdentityServer（使用 ``AddIdentityServer``）之后在 ``ConfigureServices`` 中完成。
例如::

    services.AddIdentityServer()
        .AddInMemoryClients(Clients.Get())
        .AddInMemoryIdentityResources(Resources.GetIdentityResources())
        .AddInMemoryApiResources(Resources.GetApiResources())
        .AddDeveloperSigningCredential()
        .AddTestUsers(TestUsers.Users);

    services.AddAuthentication("MyCookie")
        .AddCookie("MyCookie", options =>
        {
            options.ExpireTimeSpan = ...;
        });

.. note:: IdentityServer 使用自定义方案（通过常量 ``IdentityServerConstants.DefaultCookieAuthenticationScheme``）在内部调用 ``AddAuthentication`` 和 ``AddCookie``，因此要覆盖它们，您必须在 ``AddIdentityServer`` 之后进行相同的调用。

登录用户界面和身份管理系统
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
IdentityServer 不提供任何用于用户身份验证的用户界面或用户数据库。
这些是您自己应该提供或开发的东西。

如果您需要一个基本 UI（登录、注销、同意和管理授权）的起点，
您可以使用我们的 `快速入门 UI <https://github.com/IdentityServer/IdentityServer4.Quickstart.UI>`_。

快速入门 UI 根据内存数据库对用户进行身份验证。 您可以使用对真实用户存储的访问权限来替换这些位。
我们有使用 :ref:`ASP.NET Identity <refAspNetIdentityQuickstart>` 的示例。

登录工作流程
^^^^^^^^^^^^^^
当 IdentityServer 在授权端点收到请求并且用户未通过身份验证时，用户将被重定向到配置的登录页面。
您必须通过 :ref:`选项 <refOptions>` 上的 ``UserInteraction`` 设置（默认为 ``/account/login``）将登录页面的路径告知 IdentityServer。
将传递一个 ``returnUrl`` 参数，通知您的登录页面，一旦登录完成，用户应该被重定向到该页面。

.. image:: images/signin_flow.png

.. Note:: 通过 ``returnUrl`` 参数请注意 `开放式重定向攻击 <https://en.wikipedia.org/wiki/URL_redirection#Security_issues>`_。 您应该验证 ``returnUrl`` 指的是众所周知的位置。 请参阅 API 的 :ref:`interaction service <refInteractionService>` 以验证 ``returnUrl`` 参数。

登录上下文
^^^^^^^^^^^^^
在您的登录页面上，您可能需要有关请求上下文的信息以自定义登录体验
（例如客户端、提示参数、IdP 提示或其他内容）。
这是通过 :ref:`interaction service <refInteractionService>` 上的 ``GetAuthorizationContextAsync`` API 提供的。

颁发 cookie 和声明
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
ASP.NET Core 的 ``HttpContext`` 上有与身份验证相关的扩展方法，用于发出身份验证 cookie 并让用户登录。
使用的身份验证方案必须与您使用的 cookie 处理程序匹配（见上文）。

当您登录用户时，您必须至少发布一个 ``sub`` 声明和一个 ``name`` 声明。
IdentityServer 还在 HttpContext 上提供了一些 SignInAsync 扩展方法，以使其更方便。

您还可以选择发出 ``idp`` 声明（用于身份提供者名称）、``amr`` 声明（用于使用的身份验证方法）和/或 ``auth_time`` 声明（用于纪元时间 用户通过身份验证）。
如果您不提供这些，则 IdentityServer 将提供默认值。

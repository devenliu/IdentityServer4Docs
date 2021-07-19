.. _refSignOutFederated:
联合注销
==================

联合注销是指用户使用外部身份提供商登录 IdentityServer，然后用户通过 IdentityServer 未知的工作流注销该外部身份提供商的情景。
当用户退出时，IdentityServer 会收到通知，以便它可以让用户注销 IdentityServer 和所有使用 IdentityServer 的应用程序。

并非所有外部身份提供商都支持联合注销，但支持联合注销的提供商将提供一种机制来通知客户端用户已注销。
此通知通常以来请求的形式出现在外部身份提供商的 ``注销`` 页面的 ``<iframe>`` 中。
IdentityServer 随后必须通知它的所有客户端（如 :ref:`此处 <refSignOut>` 所述），通常也是以来请求的形式从外部身份提供商的 ``<iframe>`` 中的发出通知。

使联合注销成为一种特殊情况（与正常的 :ref:`注销 <refSignOut>` 相比）的原因是，联合注销请求不属于 IdentityServer 中的正常注销端点。
事实上，每个外部 IdentityProvider 在您的 IdentityServer 主机中都有一个不同的端点。 
这是因为每个外部身份提供商可能使用不同的协议，并且每个中间件侦听不同的端点。

所有这些因素的最终效果是，没有像我们在正常注销工作流中那样呈现 ``注销`` 页面，
这意味着我们错过了 IdentityServer 客户端的注销通知。
我们必须为每个联合注销端点添加代码，以呈现实现联合注销所需的通知。

幸运的是 IdentityServer 已经包含了此代码。
当请求进入 IdentityServer 并调用外部身份验证提供商的处理程序时，IdentityServer 会检测这些是否是联合注销请求，如果是，它将自动呈现与 :ref:`此处描述的注销 <refSignOut>` 相同的 ``<iframe>``。
简而言之，自动支持联合注销。

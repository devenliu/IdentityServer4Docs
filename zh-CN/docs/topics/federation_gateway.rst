联合网关
==================

一种常见的体系结构是所谓的联合网关。 在这种方法中，IdentityServer 充当一个或多个外部身份提供商的网关。

.. image:: images/federation_gateway.png

这种体系结构有以下优点

* 您的应用程序只需要知道一个令牌服务（网关），并且不需要了解有关连接到外部提供商的所有详细信息。 这也意味着您可以添加或更改这些外部提供商，而无需更新您的应用程序。
* 您可以控制网关（与某些外部服务提供商相反）—— 这意味着您可以对其进行任何更改，并可以保护您的应用程序免受这些外部提供商可能对其自己的服务所做的更改。
* 大多数外部提供商只支持一组固定的声明和声明类型 —— 中间有一个网关允许对来自提供商的响应进行后处理以转换/添加/修改特定于域的身份信息。
* 一些提供商不支持访问令牌（例如社交提供者）—— 因为网关知道你的 API，它可以根据外部身份发布访问令牌。
* 一些提供商根据您连接到他们的应用程序数量收费。 网关充当外部提供商的单个应用程序。 在内部，您可以根据需要连接任意数量的应用程序。
* 一些提供商使用专有协议或对标准协议进行专有修改 —— 使用网关，您只需要处理一个地方。
* 强制每个身份验证（内部或外部）都通过一个地方，这在身份映射方面为您提供了极大的灵活性，为您的所有应用程序提供了稳定的身份，并可以处理新的需求

换句话说 —— 拥有联合网关可以让您对身份基础设施进行大量控制。 由于您的用户身份是您最重要的资产之一，我们建议您控制网关。

执行
^^^^^^^^^^^^^^
我们的 `快速启动 UI <https://github.com/IdentityServer/IdentityServer4.Quickstart.UI>`_ 利用了以下一些功能。 
另请查看 :ref:`外部身份验证快速入门 <refExternalAuthenticationQuickstart>` 和有关 :ref:`外部提供商 <refExternalIdentityProviders>` 的文档。

* 您可以通过向 IdentityServer 应用程序添加身份验证处理程序来添加对外部身份提供商的支持。
* 您可以通过调用 ``IAuthenticationSchemeProvider`` 以编程方式查询这些外部提供商。 这允许根据注册的外部提供商动态呈现您的登录页面。
* 我们的客户端配置模型允许在每个客户端的基础上限制可用的提供商（使用 ``IdentityProviderRestrictions`` 属性）。
* 您还可以使用客户端上的 ``EnableLocalLogin`` 属性来告诉您的 UI 是否应呈现 用户名/密码 输入。
* 我们的快速入门 UI 通过一次回调（请参阅“AccountController”类中的“ExternalLoginCallback”）来传递所有外部身份验证调用。 这允许单点进行后处理。

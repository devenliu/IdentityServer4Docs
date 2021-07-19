欢迎使用 IdentityServer4（最新）
=============================================

.. image:: images/logo.png
   :align: center

IdentityServer4 是用于 ASP.NET Core 的 OpenID Connect 和 OAuth 2.0 框架。

.. warning:: 
   从2020年10月1日起，我们成立了一家新的 `公司 <https://duendesoftware.com/>`_ 。
   所有新的主要功能工作都将在我们新的 `组织 <https://github.com/duendesoftware>`_ 中进行。
   新的 Duende IdentityServer 可在 FOSS (RPL) 和商业许可下使用。
   开发和测试始终免费。
   `联系 <https://duendesoftware.com/contact>`_ 了解更多信息。
   
   IdentityServer4将在2022年11月之前进行错误修复和安全更新。 


.. note:: 本文档涵盖了主分支上的最新版本。这可能尚未发布。使用左下角的版本选择器选择特定版本的文档。

它在您的应用程序中启用以下功能：


| **身份验证即服务** 
| 所有应用程序（Web、原生、移动、服务）的集中登录逻辑和工作流。 
| IdentityServer 是 OpenID Connect 的官方 `认证 <https://openid.net/certification/>`_ 实现。

| **单点登录/注销** 
| 通过多种应用程序类型进行单点登录（和注销）。

| **API访问控制** 
| 为各种类型的客户端发布 API 的访问令牌，例如 服务器到服务器、Web 应用程序、SPA 和原生/移动应用程序。

| **联合网关**
| 支持外部身份提供商，如 Azure Active Directory、Google、Facebook 等。
| 这使您的应用程序免受如何连接到这些外部提供商的详细信息的影响。

| **专注于定制**
| 最重要的部分 - IdentityServer 的许多方面都可以定制以满足 **您** 的需求。由于 IdentityServer 是一个框架，而不是盒装产品或 SaaS，您可以编写代码来调整系统，使其适合您的场景。

| **成熟的开源**
| IdentityServer 使用许可的 `Apache 2 <https://www.apache.org/licenses/LICENSE-2.0>`_ 许可证，允许在其上构建商业产品。
| 它也是 `.NET 基金会 <https://dotnetfoundation.org/>`_ 的一部分，提供治理和法律支持。

| **免费和商业支持**
| 如果您需要帮助构建或运行您的身份平台，:ref:`请告诉我们 <refSupport>` 。我们可以通过多种方式帮助您。

.. toctree::
   :maxdepth: 3
   :hidden:
   :caption: 介绍

   intro/big_picture
   intro/architecture
   intro/terminology
   intro/specs
   intro/packaging
   intro/support
   intro/test
   intro/contributing

.. toctree::
   :maxdepth: 3
   :hidden:
   :caption: 快速入门

   quickstarts/0_overview
   quickstarts/1_client_credentials
   quickstarts/2_interactive_aspnetcore
   quickstarts/3_aspnetcore_and_apis
   quickstarts/4_javascript_client
   quickstarts/5_entityframework
   quickstarts/6_aspnet_identity
   
.. toctree::
   :maxdepth: 3
   :hidden:
   :caption: 配置

   configuration/startup
   configuration/resources
   configuration/clients
   configuration/mvc
   configuration/apis

.. toctree::
   :maxdepth: 3
   :hidden:
   :caption: 主题

   topics/startup
   topics/resources
   topics/clients
   topics/signin
   topics/signin_external_providers
   topics/windows
   topics/signout
   topics/signout_external_providers
   topics/signout_federated
   topics/federation_gateway
   topics/consent
   topics/apis
   topics/deployment
   topics/logging
   topics/events
   topics/crypto
   topics/grant_types
   topics/client_authentication
   topics/extension_grants
   topics/resource_owner
   topics/refresh_tokens
   topics/reference_tokens
   topics/persisted_grants
   topics/pop
   topics/mtls
   topics/request_object
   topics/custom_token_request_validation
   topics/cors
   topics/discovery
   topics/add_apis
   topics/add_protocols
   topics/tools

.. toctree::
   :maxdepth: 3
   :hidden:
   :caption: 端点

   endpoints/discovery
   endpoints/authorize
   endpoints/token
   endpoints/userinfo
   endpoints/device_authorization
   endpoints/introspection
   endpoints/revocation
   endpoints/endsession

.. toctree::
   :maxdepth: 3
   :hidden:
   :caption: 参考

   reference/options
   reference/identity_resource
   reference/api_scope
   reference/api_resource
   reference/client
   reference/grant_validation_result
   reference/profileservice
   reference/interactionservice
   reference/deviceflow_interactionservice
   reference/ef
   reference/aspnet_identity

.. toctree::
   :maxdepth: 3
   :hidden:
   :caption: 杂项

   misc/training
   misc/blogs
   misc/videos

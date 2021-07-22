.. _refGrantTypes:
授权类型
^^^^^^^^^^^
OpenID Connect 和 OAuth 2.0 规范定义了所谓的授权类型（通常也称为流程 —— 或协议流程）。
授权类型指定客户端如何与令牌服务交互。

您需要通过 ``Client`` 配置上的 ``AllowedGrantTypes`` 属性指定客户端可以使用哪些授权类型。
这允许锁定 给定客户端所允许的 协议交互。

客户端可以配置为使用多种授权类型（例如，用于以用户为中心的操作的授权码流程 和 用于服务器到服务器通信的客户端凭据）。
``GrantTypes`` 类可用于从典型的授权类型组合中进行选择::

    Client.AllowedGrantTypes = GrantTypes.CodeAndClientCredentials;

您还可以手动指定授权类型列表::

    Client.AllowedGrantTypes = 
    {
        GrantType.Code, 
        GrantType.ClientCredentials,
        "my_custom_grant_type" 
    };

虽然 IdentityServer 支持所有标准授权类型，但对于常见的应用场景，您实际上只需要了解其中的两种。

机器到机器通信
================================
这是最简单的通信类型。 始终代表客户端请求令牌，不存在交互式用户。

在这种情况下，您使用 ``client credentials`` 授权类型向令牌端点发送令牌请求。
客户端通常必须使用其客户端 ID 和密钥对令牌端点进行身份验证。

有关如何使用它的示例，请参阅 :ref:`客户端凭据快速入门 <refClientCredentialsQuickstart>`。 

交互式客户端
===================
这是最常见的客户端场景类型：具有交互式用户的 Web 应用程序、SPA 或原生/移动应用程序。

.. Note:: 如果您不关心所有技术细节，请随意跳至摘要。

针对此类客户端，设计了 ``authorization code`` 流程。 该流程包含两个物理操作：

* 通过浏览器的前端渠道步骤，所有”交互式“事情都会发生，例如 登录页面、同意等。此步骤产生一个代表前端渠道操作结果的授权码。
* 一个后端渠道步骤，其中步骤 1 中的授权码与请求的令牌交换。 机密客户端此时需要进行身份验证。

此流程具有以下安全属性：

* 没有数据（除了基本上是随机字符串的授权码）通过浏览器渠道泄露
* 授权码只能使用一次
* 只有在（对于机密客户端 —— 稍后会详细介绍）客户端密钥已知时，授权码才能转换为令牌

这听起来很不错 —— 仍然有一个叫做 `代码替换攻击 <https://nat.sakimura.org/2016/01/25/cut-and-pasted-code-attack-in-oauth-2-0-rfc6749/>`_ 的问题。
对此，有两种现代缓解技术：

**OpenID Connect 混合流程**

此流程使用 ``code id_token`` 的响应类型来向响应添加额外的身份标识。 此令牌已签名并受到保护以防止替换。
此外，它还通过 ``c_hash`` 声明包含代码的哈希值。 这允许检查您是否确实获得了正确的代码（专家称之为分离签名）。

这解决了问题，但有以下缺点：

* ``id_token`` 通过前端渠道传输，可能会泄露额外的（个人身份）数据
* 所有缓解步骤（例如加密）都需要由客户端实施。 这导致更复杂的客户端库实现。

**RFC 7636 - Proof Key for Code Exchange (PKCE)**

这实质上为授权码流程引入了每个请求的密钥（请阅读 `此处 <https://tools.ietf.org/html/rfc7636>`_ 的详细信息）。
为此，客户端所必须要实现的就是创建一个随机字符串并使用 SHA256 对其进行哈希。

这也解决了替换问题，因为客户端可以证明它是前后渠道上的同一个客户端，并且具有以下附加优势：

* 与混合流程相比，客户端实现非常简单
* 它还解决了公共客户端没有静态密钥的问题
* 不需要额外的前端渠道响应伪像

**总结**

交互式客户端应使用基于授权码的流程。 为了防止代码替换，应该使用混合流程或 PKCE。
如果 PKCE 可用，这是解决问题的更简单的方法。

PKCE 已经是对 `原生 <https://tools.ietf.org/html/rfc8252#section-6>`_ 应用程序
和 `SPAs <https://tools.ietf.org/html/draft-ietf-oauth-browser-based-apps-03#section-4>`_ 的官方推荐 
—— 随着 ASP.NET Core 3 的发布，OpenID Connect 处理程序也默认支持。

以下是如何配置交互式客户端::

    var client = new Client
    {
        ClientId = "...",

        // 为机密客户端设置客户端密钥
        ClientSecret = { ... },

        // ...或为公共客户端关闭
        RequireClientSecret = false,

        AllowedGrantTypes = GrantTypes.Code,
        RequirePkce = true
    };


没有浏览器或输入设备受限的交互式客户端
======================================================================
此授权类型在 `RFC 8628 <https://tools.ietf.org/html/rfc8628>`_ 中有详细说明。

此流程将用户身份验证和同意外包给外部设备（例如智能手机）。
它通常由没有真正的键盘（如电视、游戏机...）并且可以请求身份和 API 资源的设备使用。

自定义场景
================
扩展授权允许使用新的授权类型扩展令牌端点。 有关更多详细信息，请参阅 :ref:`此内容 <refExtensionGrants>`。 

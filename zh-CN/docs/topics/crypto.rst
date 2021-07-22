.. _refCrypto:
密码学、密钥和 HTTPS
============================

IdentityServer 依靠几种加密机制来完成其工作。

令牌签名和验证
^^^^^^^^^^^^^^^^^^^^^^^^^^^^
IdentityServer 需要一个非对称密钥对来签名和验证 JWT。
此密钥材料可以打包为证书，也可以打包为原始密钥。
支持 RSA 和 ECDSA 密钥，支持的签名算法有：RS256、RS384、RS512、PS256、PS384、PS512、ES256、ES384 和 ES512。

您可以同时使用多个签名密钥，但每种算法仅支持一个签名密钥。
您注册的第一个签名密钥被视为默认签名密钥。

:ref:`客户端 <refClient>` 和 :ref:`API 资源 <refApiResource>` 都可以表达对签名算法的偏好。
如果您为多个 API 资源请求一个令牌，则所有资源都需要就至少一种允许的签名算法达成一致。

签名密钥和相应的验证部分的加载是通过 ``ISigningCredentialStore`` 和 ``IValidationKeysStore`` 的实现来完成的。
如果您想自定义密钥的加载，您可以实现这些接口并将它们注册到 DI。

DI 构建器扩展有几个方便的方法来设置签名和验证密钥 —— 请参阅 :ref:`此处 <refStartupKeyMaterial>`。

签名密钥滚动更新
^^^^^^^^^^^^^^^^^^^^
虽然您一次只能使用一个签名密钥，但您可以向发现文档发布多个验证密钥。
这对于密钥滚动更新很有用。

简而言之，滚动更新通常是这样工作的：

1. 您请求/创建新的密钥材料
2. 除了当前的验证密钥之外，您还可以发布新的验证密钥。 您可以使用 ``AddValidationKey`` 构建器扩展方法进行此操作。
3. 所有客户端和 API 现在都有机会在下次更新发现文档的本地副本时了解新密钥
4. 经过一段时间（例如 24 小时）后，所有客户端和 API 现在都应该同时接受旧的和新的密钥材料
5. 旧的密钥材料您想保留多久就保留多久，也许您有需要验证的长期令牌
6. 当旧密钥材料不再使用时将其报废
7. 所有客户端和 API 将在下次更新发现文档的本地副本时“忘记”旧密钥

这要求客户端和 API 使用发现文档，并且还具有定期刷新其配置的功能。

Brock 写了一篇关于密钥轮换的更详细的 `博客文章 <https://brockallen.com/2019/08/09/identityserver-and-signing-key-rotation/>`_，
还创建了一个 `商业组件 <https://www.identityserver.com/products/keymanagement>`_，可以自动处理所有这些细节。

数据保护
^^^^^^^^^^^^^^^
ASP.NET Core 中的 Cookie 身份验证（或 MVC 中的 anti-forgery）使用 ASP.NET Core 数据保护功能。
根据您的部署方案，这可能需要额外的配置。 有关更多信息，请参阅 Microsoft `文档 <https://docs.microsoft.com/en-us/aspnet/core/security/data-protection/configuration/overview>`_。

HTTPS
^^^^^
我们不强制使用 HTTPS，但对于生产环境，与 IdentityServer 的每次交互都是强制性的。

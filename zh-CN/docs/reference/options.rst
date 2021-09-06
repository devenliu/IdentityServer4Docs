.. _refOptions:
IdentityServer Options
======================

* ``IssuerUri``
    设置将出现在发现文档中的颁发者名称和颁发的 JWT 令牌。
    建议不要设置此属性，它会根据客户端使用的主机名推断颁发者名称。

* ``LowerCaseIssuerUri``
    设置为 ``false`` 以保留 IssuerUri 的原始大小写。 默认为 ``true``。

* ``AccessTokenJwtType``
    指定用于访问令牌的 JWT 类型标头的值（默认为 ``at+jwt``）。

* ``EmitScopesAsSpaceDelimitedStringInJwt``
    指定 JWT 中的作用域是作为数组还是字符串发出

* ``EmitStaticAudienceClaim``
    发出带有发行者/资源格式的 ``aud`` 声明。 默认为 false。

Endpoints
^^^^^^^^^
允许启用/禁用单个端点，例如 令牌，授权，用户信息等。

默认情况下，所有端点都已启用，但您可以通过禁用不需要的端点来锁定服务器。

* ``EnableJwtRequestUri``
    在授权端点上启用 JWT request_uri 处理。 默认为 ``false``。

Discovery
^^^^^^^^^
允许启用/禁用发现文档的各个部分，例如 端点、范围、声明、授权类型等。

``CustomEntries`` 字典允许向发现文档添加自定义元素。

Authentication
^^^^^^^^^^^^^^
* ``CookieAuthenticationScheme``
    设置主机配置的用于交互用户的cookie认证方案。 如果未设置，将从主机的默认身份验证方案中推断出该方案。 当 AddPolicyScheme 在主机中用作默认方案时，通常使用此设置。

* ``CookieLifetime``
    身份验证 cookie 生存期（仅在使用 IdentityServer 提供的 cookie 处理程序时才有效）。

* ``CookieSlidingExpiration``
    指定 cookie 是否应该滑动（仅在使用 IdentityServer 提供的 cookie 处理程序时有效）。

* ``CookieSameSiteMode``
    为内部 cookie 指定 SameSite 模式。

* ``RequireAuthenticatedUserForSignOutMessage``
    指示用户是否必须经过身份验证才能接受参数以结束会话端点。 默认为 false。

* ``CheckSessionCookieName``
    用于检查会话端点的 cookie 的名称。

* ``CheckSessionCookieDomain``
    用于检查会话端点的 cookie 的域。

* ``CheckSessionCookieSameSiteMode``
    用于检查会话端点的 cookie 的 SameSite 模式。

* ``RequireCspFrameSrcForSignout``
    如果设置，将需要在结束会话回调端点上发出 frame-src CSP 标头，该端点将 iframe 呈现给客户端以进行前端通道注销通知。 默认为 true。

Events
^^^^^^
允许配置是否以及哪些事件应该提交到注册的事件接收器。 有关事件的更多信息，请参见 :ref:`此处 <refEvents>`。

InputLengthRestrictions
^^^^^^^^^^^^^^^^^^^^^^^
允许对各种协议参数（如客户端 ID、范围、重定向 URI 等）设置长度限制。

UserInteraction
^^^^^^^^^^^^^^^

* ``LoginUrl``, ``LogoutUrl``, ``ConsentUrl``, ``ErrorUrl``, ``DeviceVerificationUrl``
    设置登录、注销、同意、错误和设备验证页面的 URL。
* ``LoginReturnUrlParameter``
    设置传递给登录页面的返回 URL 参数的名称。 默认为 *returnUrl*。
* ``LogoutIdParameter``
    设置传递给注销页面的注销消息 id 参数的名称。 默认为 *logoutId*。
* ``ConsentReturnUrlParameter``
    设置传递给同意页面的返回 URL 参数的名称。 默认为 *returnUrl*。
* ``ErrorIdParameter``
    设置传递给错误页面的错误消息 id 参数的名称。 默认为 *errorId*。
* ``CustomRedirectReturnUrlParameter``
    设置从授权端点传递给自定义重定向的返回 URL 参数的名称。 默认为 *returnUrl*。
* ``DeviceVerificationUserCodeParameter``
    设置传递给设备验证页面的用户代码参数的名称。 默认为 *userCode*。
* ``CookieMessageThreshold``
    IdentityServer 和一些 UI 页面之间的某些交互需要 cookie 来传递状态和上下文（上面的任何页面都具有可配置的 "message id" 参数）。
    由于浏览器对 cookie 的数量和大小有限制，此设置用于防止创建过多的 cookie。
    该值设置将创建的任何类型的消息 cookie 的最大数量。
    一旦达到限制，最旧的消息 cookie 将被清除。
    这有效地指示了用户在使用 IdentityServer 时可以打开多少个选项卡。

Caching
^^^^^^^
只有在启动时在服务配置中启用了相应的缓存时，这些设置才适用。

* ``ClientStoreExpiration``
    从客户端存储加载的客户端配置的缓存持续时间。

* ``ResourceStoreExpiration``
    从资源存储加载的身份和 API 资源配置的缓存持续时间。

CORS
^^^^
IdentityServer 为其某些端点支持 CORS。
底层 CORS 实现由 ASP.NET Core 提供，因此它会自动注册到依赖注入系统中。

* ``CorsPolicyName``
    将评估 CORS 请求到 IdentityServer 的 CORS 策略的名称（默认为 ``IdentityServer4``）。
    处理这个的策略提供者是根据在依赖注入系统中注册的 ``ICorsPolicyService`` 实现的。
    如果您希望自定义允许连接的 CORS 源集，则建议您提供 ``ICorsPolicyService`` 的自定义实现。

* ``CorsPaths``
    IdentityServer 中支持 CORS 的端点。
    默认为发现、用户信息、令牌和撤销端点。

* ``PreflightCacheDuration``
    `Nullable<TimeSpan>` 指示要在预检 `Access-Control-Max-Age` 响应标头中使用的值。
    默认为 `null` 表示没有在响应中设置缓存头。

CSP (内容安全政策)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
IdentityServer 在适当的情况下为某些响应发出 CSP 标头。

* ``Level``
    要使用的 CSP 级别。 默认情况下使用 CSP 级别 2，但如果必须支持旧浏览器，则将其更改为 ``CspLevel.One`` 以适应它们。

* ``AddDeprecatedHeader``
    指示是否还应发出旧的 ``X-Content-Security-Policy`` CSP 标头（除了基于标准的标头值）。 默认为 true。

Device Flow
^^^^^^^^^^^

* ``DefaultUserCodeType``
    要使用的用户代码类型，除非在客户端级别设置。 默认为 *Numeric*，一个 9 位代码。
* ``Interval``
    定义令牌端点上允许的最小轮询间隔。 默认为 *5*。

Mutual TLS
^^^^^^^^^^

* ``Enabled``
    指定是否应启用 MTLS 支持。 默认为 ``false``。
* ``ClientCertificateAuthenticationScheme``
    指定 X.509 客户端证书的身份验证处理程序的名称。 默认为 ``"Certificate"``。
* ``DomainName``
    指定用于运行 MTLS 端点的子域或完整域的名称（如果未设置，将使用基于路径的端点）。
    使用简单字符串（例如 ``mtls``）设置子域，使用完整域名（例如 ``identityserver-mtls.io``）设置完整域名。
    使用完整域名时，还需要将 ``IssuerName`` 设置为固定值。
* ``AlwaysEmitConfirmationClaim``
    指定如果存在客户端证书，是否为访问令牌发出 cnf 声明。
    通常，只有当客户端使用客户端证书进行身份验证时，才会发出 cnf 声明，
    将此设置为 true，无论身份验证方法如何，都将设置声明。 （默认为 false）。

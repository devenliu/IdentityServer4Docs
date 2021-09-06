授权端点
==================

授权端点可用于通过浏览器请求令牌或授权码。
此过程通常涉及最终用户的身份验证和可选的同意。

.. Note:: IdentityServer 支持 OpenID Connect 和 OAuth 2.0 授权请求参数的子集。 有关完整列表，请参阅 `此处<https://openid.net/specs/openid-connect-core-1_0.html#AuthRequest>`_。

``client_id``
    客户端的标识符（必需）。
``request``
    您可以提供一个子集或全部作为 JWT，而不是将所有参数作为单独的查询字符串参数提供
``request_uri``
    包含请求参数的预打包 JWT 的 URL
``scope``
    一个或多个已注册的范围（必需）
``redirect_uri`` 
    必须与该客户端允许的重定向 URI 之一完全匹配（必需）
``response_type`` 
    ``id_token`` 请求身份令牌（仅允许身份范围）

    ``token`` 请求访问令牌（仅允许资源范围）

    ``id_token token`` 请求身份令牌和访问令牌

    ``code`` 请求授权码

    ``code id_token`` 请求授权码和身份令牌

    ``code id_token token`` 请求授权码、身份令牌和访问令牌
    
``response_mode``
    ``form_post`` 将令牌响应作为表单发布而不是片段编码重定向发送（可选）
``state`` 
    identityserver 将在令牌响应上回显状态值，这是用于客户端和提供者之间的往返状态，关联请求和响应以及 CSRF/重放保护。 （推荐的）
``nonce`` 
    identityserver 将回显身份令牌中的 nonce 值，这是为了重放保护）

    *对于通过隐式授权的身份令牌是必需的。*
``prompt``
    ``none`` 请求期间不会显示 UI。 如果这是不可能的（例如，因为用户必须登录或同意），则返回错误
    
    ``login`` 即使用户已经登录并且有一个有效的会话，也会显示登录 UI
``code_challenge``
    发送 PKCE 的代码质询
``code_challenge_method``
    ``plain`` 表示 challenge 使用纯文本（不推荐）
    ``S256`` 表示 challenge 是用 SHA256 散列的
``login_hint``
    可用于在登录页面预填用户名字段
``ui_locales``
    提供有关登录 UI 所需显示语言的提示
``max_age``
    如果用户的登录会话超过最大期限（以秒为单位），将显示登录 UI
``acr_values``
    允许传入额外的身份验证相关信息 - identityserver 特殊情况下以下专有 acr_values：
        
        ``idp:name_of_idp`` 绕过登录/主域屏幕并将用户直接转发到选定的身份提供者（如果每个客户端配置允许）
        
        ``tenant:name_of_tenant`` 可用于将租户名称传递给登录 UI

**例子**

::

    GET /connect/authorize?
        client_id=client1&
        scope=openid email api1&
        response_type=id_token token&
        redirect_uri=https://myapp/callback&
        state=abc&
        nonce=xyz 

（删除了 URL 编码，并添加了换行符以提高可读性）

.. Note:: 您可以使用 `IdentityModel <https://github.com/IdentityModel/IdentityModel2>`_ 客户端库以编程方式创建授权请求 .NET 代码。 有关更多信息，请查看 IdentityModel `文档 <https://identitymodel.readthedocs.io/en/latest/client/authorize.html>`_。
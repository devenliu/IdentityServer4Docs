令牌端点
==============

令牌端点可用于以编程方式请求令牌。
它支持 ``password``、``authorization_code``、``client_credentials``、``refresh_token`` 和 ``urn:ietf:params:oauth:grant-type:device_code`` 授权类型。
此外，可以扩展令牌端点以支持扩展授权类型。

.. Note:: IdentityServer 支持 OpenID Connect 和 OAuth 2.0 令牌请求参数的子集。 有关完整列表，请参阅 `此处<http://openid.net/specs/openid-connect-core-1_0.html#TokenRequest>`_。

``client_id``
    客户端标识符（必需 - 在正文中或作为授权标头的一部分。）
``client_secret``
    客户端机密在帖子正文中，或作为基本身份验证标头。 可选的。
``grant_type``
    ``authorization_code``, ``client_credentials``, ``password``, ``refresh_token``, ``urn:ietf:params:oauth:grant-type:device_code`` 或自定义
``scope``
    一个或多个注册范围。 如果未指定，则将发布所有明确允许的范围的令牌。
``redirect_uri`` 
    ``authorization_code`` 授权类型所需
``code``
    授权码（“authorization_code” 授权类型需要）
``code_verifier``
    PKCE 证明密钥
``username`` 
    资源所有者用户名（ ``password`` 授权类型需要）
``password``
    资源所有者密码（ ``password`` 授权类型需要）
``acr_values``
   允许为 ``password`` 授权类型传递额外的身份验证相关信息 - identityserver 特殊情况，以下专有 acr_values：
        
        ``idp:name_of_idp`` 绕过登录/主域屏幕并将用户直接转发到选定的身份提供者（如果每个客户端配置允许）
        
        ``tenant:name_of_tenant`` 可用于将租户名称传递给令牌端点
``refresh_token``
    刷新令牌（ ``refresh_token`` 授予类型所需）
``device_code``
    设备代码（ ``urn:ietf:params:oauth:grant-type:device_code`` 授权类型需要）

例子
^^^^^^^

::

    POST /connect/token
    CONTENT-TYPE application/x-www-form-urlencoded

        client_id=client1&
        client_secret=secret&
        grant_type=authorization_code&
        code=hdh922&
        redirect_uri=https://myapp.com/callback

（删除了表单编码并添加了换行符以提高可读性）

.. Note:: 您可以使用 `IdentityModel <https://github.com/IdentityModel/IdentityModel>`_ 客户端库从 .NET 代码以编程方式访问令牌端点。 有关更多信息，请查看 IdentityModel `文档 <https://identitymodel.readthedocs.io/en/latest/client/token.html>`_。

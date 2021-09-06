用户信息端点
=================

用户信息端点可用于检索有关用户的身份信息（请参阅 `规范 <http://openid.net/specs/openid-connect-core-1_0.html#UserInfo>`_）。

调用者需要发送代表用户的有效访问令牌。
根据授予的范围，用户信息端点将返回映射的声明（至少需要 ``openid`` 范围）。

例子
^^^^^^^

::

    GET /connect/userinfo
    Authorization: Bearer <access_token>

::

    HTTP/1.1 200 OK
    Content-Type: application/json

    {
        "sub": "248289761001",
        "name": "Bob Smith",
        "given_name": "Bob",
        "family_name": "Smith",
        "role": [
            "user",
            "admin"
        ]
    }

.. Note:: 您可以使用 `IdentityModel <https://github.com/IdentityModel/IdentityModel2>`_ 客户端库从 .NET 代码以编程方式访问 userinfo 端点。 有关更多信息，请查看 IdentityModel `文档 <https://identitymodel.readthedocs.io/en/latest/client/userinfo.html>`_。
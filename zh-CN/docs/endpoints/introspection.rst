自省端点
======================

自省端点是 `RFC 7662 <https://tools.ietf.org/html/rfc7662>`_ 的实现。

它可用于验证参考令牌（或 JWT，如果消费者不支持适当的 JWT 或加密库）。
自省端点需要身份验证 - 由于自省端点的客户端是一个 API，您可以在 ``ApiResource`` 上配置密钥。

例子
^^^^^^^

::


    POST /connect/introspect
    Authorization: Basic xxxyyy

    token=<token>


成功的响应将返回状态代码 200 以及活动或非活动令牌::


    {
        "active": true,
        "sub": "123"
    }


未知或过期的令牌将被标记为不活动::


    {
        "active": false,
    }


无效请求将返回 400，未经授权的请求将返回 401。

.. Note:: 您可以使用 `IdentityModel <https://github.com/IdentityModel/IdentityModel2>`_ 客户端库从 .NET 代码以编程方式访问自省端点。 有关更多信息，请查看 IdentityModel `文档 <https://identitymodel.readthedocs.io/en/latest/client/introspection.html>`_。
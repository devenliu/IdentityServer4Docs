撤销端点
===================

此端点允许撤销访问令牌（仅限参考令牌）和刷新令牌。
它实现了令牌撤销规范 `（RFC 7009）<https://tools.ietf.org/html/rfc7009>`_。

``token``
    要撤销的令牌（必需）
``token_type_hint``
    ``access_token`` 或 ``refresh_token`` （可选）

例子
^^^^^^^

::

    POST /connect/revocation HTTP/1.1
    Host: server.example.com
    Content-Type: application/x-www-form-urlencoded
    Authorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JW

    token=45ghiukldjahdnhzdauz&token_type_hint=refresh_token

.. Note:: 您可以使用 `IdentityModel <https://github.com/IdentityModel/IdentityModel2>`_ 客户端库从 .NET 代码以编程方式访问吊销端点。 有关更多信息，请查看 IdentityModel `文档 <https://identitymodel.readthedocs.io/en/latest/client/revocation.html>`_。
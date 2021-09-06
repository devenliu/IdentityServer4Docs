设备授权端点
=============================

设备授权端点可用于请求设备和用户代码。
该端点用于启动设备流授权过程。

.. Note:: 结束会话端点的 URL 可通过 :ref:`发现文档 <refDiscovery>` 获得。

``client_id``
    客户端标识符（必需）
``client_secret``
    客户端密钥在 POST body 中，或作为基本身份验证标头。 可选的。
``scope``
    一个或多个已注册的范围。 如果未指定，则将发布所有明确允许的范围的令牌。

例子
^^^^^^^

::

    POST /connect/deviceauthorization

        client_id=client1&
        client_secret=secret&
        scope=openid api1

（删除了表单编码并添加了换行符以提高可读性）

.. Note:: 您可以使用 `IdentityModel <https://github.com/IdentityModel/IdentityModel2>`_ 客户端库从 .NET 代码以编程方式访问设备授权端点。 有关更多信息，请查看 IdentityModel `文档 <https://identitymodel.readthedocs.io/en/latest/client/device_authorize.html>`_。
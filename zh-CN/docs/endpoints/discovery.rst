.. _refDiscovery:
发现端点
==================

发现端点可用于检索有关您的 IdentityServer 的元数据 -
它返回诸如发行者名称、密钥材料、支持的范围等信息。有关更多详细信息，请参阅 `规范 <https://openid.net/specs/openid-connect-discovery-1_0.html>`_。
发现端点可通过相对于基地址的 ``/.well-known/openid-configuration`` 获得，例如::

    https://demo.identityserver.io/.well-known/openid-configuration

.. Note:: 您可以使用 `IdentityModel <https://github.com/IdentityModel/IdentityModel2>`_ 客户端库从 .NET 代码以编程方式访问发现端点。 有关更多信息，请查看 IdentityModel `文档 <https://identitymodel.readthedocs.io/en/latest/client/discovery.html>`_。
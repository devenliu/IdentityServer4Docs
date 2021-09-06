.. _refEndSession:
结束会话端点
====================

结束会话端点可用于触发单点注销（请参阅 `规范 <https://openid.net/specs/openid-connect-rpinitiated-1_0.html>`_）。

要使用结束会话端点，客户端应用程序会将用户的浏览器重定向到结束会话 URL。
用户在会话期间通过浏览器登录的所有应用程序都可以参与注销。

.. Note:: 结束会话端点的 URL 可通过 :ref:`发现文档 <refDiscovery>` 获得。

参数
^^^^^^^^^^

**id_token_hint**

当用户被重定向到端点时，会提示他们是否真的要注销。
客户端发送从身份验证收到的原始 *id_token* 可以绕过此提示。
这作为名为 ``id_token_hint`` 的查询字符串参数传递。

**post_logout_redirect_uri**

如果传递了有效的 ``id_token_hint``，则客户端还可以发送 ``post_logout_redirect_uri`` 参数。
这可用于允许用户在注销后重定向回客户端。
该值必须与客户端预配置的 ``PostLogoutRedirectUris`` 之一（ :ref:`client docs <refClient>`）匹配。

**state**

如果传递了有效的 ``post_logout_redirect_uri``，则客户端还可以发送 ``state`` 参数。
在用户重定向回客户端后，这将作为查询字符串参数返回给客户端。
这通常由客户端用于跨重定向来回状态。

例子
^^^^^^^

::

    GET /connect/endsession?id_token_hint=eyJhbGciOiJSUzI1NiIsImtpZCI6IjdlOGFkZmMzMjU1OTEyNzI0ZDY4NWZmYmIwOThjNDEyIiwidHlwIjoiSldUIn0.eyJuYmYiOjE0OTE3NjUzMjEsImV4cCI6MTQ5MTc2NTYyMSwiaXNzIjoiaHR0cDovL2xvY2FsaG9zdDo1MDAwIiwiYXVkIjoianNfb2lkYyIsIm5vbmNlIjoiYTQwNGFjN2NjYWEwNGFmNzkzNmJjYTkyNTJkYTRhODUiLCJpYXQiOjE0OTE3NjUzMjEsInNpZCI6IjI2YTYzNWVmOTQ2ZjRiZGU3ZWUzMzQ2ZjFmMWY1NTZjIiwic3ViIjoiODg0MjExMTMiLCJhdXRoX3RpbWUiOjE0OTE3NjUzMTksImlkcCI6ImxvY2FsIiwiYW1yIjpbInB3ZCJdfQ.STzOWoeVYMtZdRAeRT95cMYEmClixWkmGwVH2Yyiks9BETotbSZiSfgE5kRh72kghN78N3-RgCTUmM2edB3bZx4H5ut3wWsBnZtQ2JLfhTwJAjaLE9Ykt68ovNJySbm8hjZhHzPWKh55jzshivQvTX0GdtlbcDoEA1oNONxHkpDIcr3pRoGi6YveEAFsGOeSQwzT76aId-rAALhFPkyKnVc-uB8IHtGNSyRWLFhwVqAdS3fRNO7iIs5hYRxeFSU7a5ZuUqZ6RRi-bcDhI-djKO5uAwiyhfpbpYcaY_TxXWoCmq8N8uAw9zqFsQUwcXymfOAi2UF3eFZt02hBu-shKA&post_logout_redirect_uri=http%3A%2F%2Flocalhost%3A7017%2Findex.html

.. Note:: 您可以使用 `IdentityModel <https://github.com/IdentityModel/IdentityModel2>`_ 客户端库以编程方式创建 end_session 请求 .NET 代码。 有关更多信息，请查看 IdentityModel `文档 <https://identitymodel.readthedocs.io/en/latest/client/end_session.html>`_。

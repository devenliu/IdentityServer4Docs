刷新令牌
==============
由于访问令牌的生命周期有限，刷新令牌允许在没有用户交互的情况下请求新的访问令牌。

以下流支持刷新令牌：授权代码、混合和资源所有者密码凭证流。
客户端需要通过将 ``AllowOfflineAccess`` 设置为 ``true`` ，明确授权请求刷新令牌。

其他客户端设置
^^^^^^^^^^^^^^^^^^^^^^^^^^
``AbsoluteRefreshTokenLifetime``
    刷新令牌的最长生命周期（以秒为单位）。 默认为 2592000 秒 / 30 天。 0 允许刷新令牌，当与 ``RefreshTokenExpiration = Sliding`` 一起使用时，仅在经过 SlidingRefreshTokenLifetime 后过期。
``SlidingRefreshTokenLifetime``
    刷新令牌的滑动生命周期（以秒为单位）。 默认为 1296000 秒 / 15 
``RefreshTokenUsage``
    ``ReUse`` 刷新令牌时刷新令牌句柄将保持不变
    
    ``OneTimeOnly`` 刷新令牌时将更新刷新令牌句柄
``RefreshTokenExpiration``
    ``Absolute`` 刷新令牌将在固定时间点（由 AbsoluteRefreshTokenLifetime 指定）到期。 这是默认设置。
    
    ``Sliding`` 刷新令牌时，刷新令牌的生命周期将更新（按 SlidingRefreshTokenLifetime 中指定的数量）。 生命周期不会超过 `AbsoluteRefreshTokenLifetime`。
``UpdateAccessTokenClaimsOnRefresh``
    获取或设置一个值，该值指示是否应在刷新令牌请求时更新访问令牌（及其声明）。

.. note:: 公共客户端（没有客户端密钥的客户端）应该轮换他们的刷新令牌。 将 ``RefreshTokenUsage`` 设置为 ``OneTimeOnly``。

请求刷新令牌
^^^^^^^^^^^^^^^^^^^^^^^^^^
您可以通过向 scope 参数添加一个名为 ``offline_access`` 的范围来请求刷新令牌。

使用刷新令牌请求访问令牌
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
要获取新的访问令牌，请将刷新令牌发送到令牌端点。
这将产生一个新的令牌响应，其中包含一个新的访问令牌及其过期时间，并且可能还会产生一个新的刷新令牌，具体取决于客户端配置（见上文）。

::

    POST /connect/token

        client_id=client&
        client_secret=secret&
        grant_type=refresh_token&
        refresh_token=hdh922
        
(删除了表单编码并添加了换行符以提高可读性)

.. Note:: 您可以使用 `IdentityModel <https://github.com/IdentityModel/IdentityModel>`_ 客户端库从 .NET 代码以编程方式访问令牌端点。 有关更多信息，请查看 IdentityModel `文档 <https://identitymodel.readthedocs.io/en/latest/client/token.html>`_。

.. Note:: 刷新令牌必须有效，否则会返回 invalid_grant 错误。 默认情况下，refresh_token 只能使用一次。 使用已使用的 refresh_token 将导致 invalid_grant 错误。

自定义刷新令牌行为
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
所有刷新令牌处理都在 ``DefaultRefreshTokenService``（这是 ``IRefreshTokenService`` 接口的默认实现）中实现::

    public interface IRefreshTokenService
    {
        /// <summary>
        /// 验证刷新令牌。
        /// </summary>
        Task<TokenValidationResult> ValidateRefreshTokenAsync(string token, Client client);
        
        /// <summary>
        /// 创建刷新令牌。
        /// </summary>
        Task<string> CreateRefreshTokenAsync(ClaimsPrincipal subject, Token accessToken, Client client);

        /// <summary>
        /// 更新刷新令牌。
        /// </summary>
        Task<string> UpdateRefreshTokenAsync(string handle, RefreshToken refreshToken, Client client);
    }

刷新令牌处理的逻辑相当复杂，我们不建议从头开始实现接口，
除非你完全知道自己在做什么。
如果要自定义某些行为，更建议从默认实现派生并首先调用基本检查。

您可能想要做的最常见的定制是如何处理刷新令牌重放。
这适用于令牌使用已设置为仅一次的情况，但同一令牌被多次发送。
这可能指向刷新令牌的重放攻击，或者指向错误的客户端代码，如逻辑错误或竞争条件。

需要注意的是，刷新令牌永远不会在数据库中删除。
一旦被使用，``ConsumedTime`` 属性将被设置。
如果收到一个已经被消费的令牌，默认服务将调用一个名为 ``AcceptConsumedTokenAsync`` 的虚拟方法。

默认实现将拒绝请求，但在这里您可以实现自定义逻辑，如宽限期，
或撤销额外的刷新或访问令牌。

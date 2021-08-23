.. _refProfileService:
Profile Service
===============

在创建令牌或处理对 userinfo 或自省端点的请求时，IdentityServer 通常需要有关用户的身份信息。
默认情况下，IdentityServer 只有身份验证 cookie 中的声明可用于此身份数据。

将用户所需的所有可能声明都放入 cookie 是不切实际的，因此 IdentityServer 定义了一个扩展点，允许根据用户需要动态加载声明。
此扩展点是 ``IProfileService``，开发人员通常会实现此接口以访问包含用户身份数据的自定义数据库或 API。

IProfileService APIs
^^^^^^^^^^^^^^^^^^^^

``GetProfileDataAsync``
    预期为用户加载声明的 API。 它传递了一个 ``ProfileDataRequestContext`` 的实例。

``IsActiveAsync``
    预期指示当前是否允许用户获取令牌的 API。 它传递了一个 ``IsActiveContext`` 的实例。

ProfileDataRequestContext
^^^^^^^^^^^^^^^^^^^^^^^^^

为用户声明的请求建模，并且是返回这些声明的工具。 它包含以下属性：

``Subject``
    对用户建模的 ``ClaimsPrincipal``。
``Client``
    正在为其请求声明的 ``客户端``。
``RequestedClaimTypes``
    正在请求的声明类型的集合。
``Caller``
    请求声明的上下文的标识符（例如身份令牌、访问令牌或用户信息端点）。 常量 ``IdentityServerConstants.ProfileDataCallers`` 包含不同的常量值。
``IssuedClaims``
    将返回的 ``Claim`` 列表。 这预计将由自定义 ``IProfileService`` 实现填充。
``AddRequestedClaims``
    ``ProfileDataRequestContext`` 上的扩展方法来填充 ``IssuedClaims``，但首先根据 ``RequestedClaimTypes`` 过滤声明。

请求的范围和声明映射
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

客户端请求的范围控制用户声明在令牌中返回给客户端的内容。 
``GetProfileDataAsync`` 方法负责根据 ``ProfileDataRequestContext`` 上的 ``RequestedClaimTypes`` 集合动态获取这些声明。

``RequestedClaimTypes`` 集合是根据 :ref:`资源 <refResources>` 上定义的用户声明填充的，该用户声明对范围进行建模。
如果请求身份令牌并且请求的范围是 :ref:`身份资源 <refIdentityResource>`，则 ``RequestedClaimTypes`` 中的声明将根据 ``IdentityResource`` 中定义的用户声明类型填充。
如果请求访问令牌并且请求的范围是 :ref:`API 资源 <refApiResource>`，则 ``RequestedClaimTypes`` 中的声明将根据 ``ApiResource`` 和（或） ``Scope``。

IsActiveContext
^^^^^^^^^^^^^^^

对请求建模以确定当前是否允许用户获取令牌。 它包含以下属性：

``Subject``
    对用户建模的 ``ClaimsPrincipal``。
``Client``
    正在为其请求声明的 ``客户端``。
``Caller``
   请求声明的上下文的标识符（例如身份令牌、访问令牌或用户信息端点）。 常量 ``IdentityServerConstants.ProfileDataCallers`` 包含不同的常量值。
``IsActive``
    指示是否允许用户获取令牌的标志。 这预计由自定义 ``IProfileService`` 实现分配。

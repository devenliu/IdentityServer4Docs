.. _refResourceOwnerPasswordValidator:
资源所有者密码验证
===================================

如果要使用 OAuth 2.0 资源所有者密码凭据授予（又名 ``password``），则需要实现并注册 ``IResourceOwnerPasswordValidator`` 接口::

    public interface IResourceOwnerPasswordValidator
    {
        /// <summary>
        /// 验证资源所有者密码凭据
        /// </summary>
        /// <param name="context">上下文。</param>
        Task ValidateAsync(ResourceOwnerPasswordValidationContext context);
    }

在上下文中，您会发现已经解析的协议参数，如 ``UserName`` 和 ``Password``，如果您想查看其他输入数据，还会找到原始请求。

然后你的工作是实现密码验证并相应地在上下文中设置 ``Result``。 请参阅 :ref:`GrantValidationResult <refGrantValidationResult>` 文档。
.. _refCustomTokenRequestValidation:
自定义令牌请求验证和颁发
============================================

您可以在令牌端点处将自定义代码作为令牌颁发管道的一部分运行。
这允许，例如 

* 添加额外的验证逻辑
* 动态更改某些参数（如令牌寿命）

为此，实现（并注册）``ICustomTokenRequestValidator`` 接口::

    /// <summary>
    /// 允许在令牌请求中插入自定义验证逻辑
    /// </summary>
    public interface ICustomTokenRequestValidator
    {
        /// <summary>
        /// 令牌请求的自定义验证逻辑。
        /// </summary>
        /// <param name="context">上下文。</param>
        /// <returns>
        /// 验证结果
        /// </returns>
        Task ValidateAsync(CustomTokenRequestValidationContext context);
    }

上下文对象使您可以访问：

* 添加自定义响应参数
* 返回错误和错误描述
* 修改请求参数，例如 访问令牌生命周期和类型、客户端声明和确认方法

您可以使用配置构建器上的 ``AddCustomTokenRequestValidator`` 扩展方法注册验证器的实现。

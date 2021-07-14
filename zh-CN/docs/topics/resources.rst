.. _refResources:
定义资源
==================
OpenID Connect/OAuth 令牌服务的最终工作是控制对资源的访问。

IdentityServer 中的两种基本资源类型是：

* **identity resources:** 代表关于用户的声明，如用户 ID、显示名称、电子邮件地址等......
* **API resources:** 代表客户想要访问的功能。 通常，它们是基于 HTTP 的端点（又名 API），但也可以是消息队列端点或类似端点。

.. note:: 您可以使用 C# 对象模型定义资源 —— 或从数据存储加载它们。 ``IResourceStore`` 的实现处理这些低级细节。 对于本文档，我们使用内存中实现。

身份资源
------------------
身份资源是一组命名的声明，可以使用 *scope* 参数进行请求。

OpenID Connect 规范 `建议 <https://openid.net/specs/openid-connect-core-1_0.html#ScopeClaims>`_ 了几个标准范围名称来声明类型映射，它们可能对您的灵感有用，但您可以自由地自己设计它们。

其中之一实际上是强制性的，*openid* 范围，它告诉提供者在身份令牌中返回 *sub*（subject id）声明。

这是您在代码中定义 openid 范围的方式::

    public static IEnumerable<IdentityResource> GetIdentityResources()
    {
        return new List<IdentityResource>
        {
            new IdentityResource(
                name: "openid",
                userClaims: new[] { "sub" },
                displayName: "您的用户标识符")
        };
    }

但由于这是规范中的标准范围之一，您可以将其缩短为::

    public static IEnumerable<IdentityResource> GetIdentityResources()
    {
        return new List<IdentityResource>
        {
            new IdentityResources.OpenId()
        };
    }

.. note:: 有关 ``IdentityResource`` 的更多信息，请参阅参考部分。

以下示例显示了一个名为 *profile* 的自定义身份资源，它表示显示名称、电子邮件地址和网站声明::

    public static IEnumerable<IdentityResource> GetIdentityResources()
    {
        return new List<IdentityResource>
        {
            new IdentityResource(
                name: "profile",
                userClaims: new[] { "name", "email", "website" },
                displayName: "您的个人资料数据")
        };
    }

定义资源后，您可以通过 ``AllowedScopes`` 选项（省略其他属性）将其访问权限授予客户端::

    var client = new Client
    {
        ClientId = "client",
        
        AllowedScopes = { "openid", "profile" }
    };


然后客户端可以使用 scope 参数请求资源（其他参数省略）::

    https://demo.identityserver.io/connect/authorize?client_id=client&scope=openid profile

IdentityServer 然后将使用 scope 名称创建请求的声明类型列表，
并将其呈现给您的 :ref:`profile service <refProfileService>` 的实现。

.. _refApiResources:
APIs
----
设计您的 API 表面可能是一项复杂的任务。 IdentityServer 提供了一些原语来帮助您解决这个问题。

原始的 OAuth 2.0 规范中有 scope 的概念，它只是定义为客户端请求的 *访问范围*。
从技术上讲，*scope* 参数是一个空格分隔值列表 —— 您需要提供它的结构和语义。

在更复杂的系统中，通常会引入 *resource* 的概念。 这可能是例如 物理或逻辑 API。
反过来，每个 API 也可能具有 scope。 某些 scope 可能是该资源独有的，而某些 scope 可能是共享的。

让我们首先从简单的 scopes 开始，然后我们将看看 resources 如何帮助构建 scopes。

Scopes
^^^^^^
让我们对一些非常简单的东西进行建模 —— 个具有三个逻辑操作 *read*、*write* 和 *delete* 的系统。

您可以使用 ``ApiScope`` 类定义它们::

    public static IEnumerable<ApiScope> GetApiScopes()
    {
        return new List<ApiScope>
        {
            new ApiScope(name: "read",   displayName: "读取您的数据。"),
            new ApiScope(name: "write",  displayName: "写入您的数据。"),
            new ApiScope(name: "delete", displayName: "删除您的数据。")
        };
    }

然后，您可以将 scopes 分配给各种客户端，例如::

    var webViewer = new Client
    {
        ClientId = "web_viewer",
        
        AllowedScopes = { "openid", "profile", "read" }
    };

    var mobileApp = new Client
    {
        ClientId = "mobile_app",
        
        AllowedScopes = { "openid", "profile", "read", "write", "delete" }
    }

基于 Scopes 的授权
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
当客户端请求 scope（并且该 scope 是通过配置允许的，而不是通过同意拒绝）时，
该 scope 的值将作为 *scope* 类型的声明包含在结果访问令牌中（对于 JWT 和自省） ，例如::

    {
        "typ": "at+jwt"
    }.
    {
        "client_id": "mobile_app",
        "sub": "123",

        "scope": "read write delete"
    }

访问令牌的使用者可以使用该数据来确保实际上允许客户端调用相应的功能。

.. note:: 请注意，scope 仅用于授权客户端 —— 而不是用户。 IOW —— *write* scope 允许客户端调用与之相关的功能。 但是，该客户端很可能只能写入属于当前用户的数据。 这个额外的以用户为中心的授权是应用程序逻辑，不包含在 OAuth 中。

您可以通过从 scope 请求派生其他声明来添加有关用户的更多身份信息。 以下 scope 定义告诉配置系统，当授予 *write* scope 时，应将 *user_level* 声明添加到访问令牌::

    var writeScope = new ApiScope(
        name: "write",
        displayName: "写入您的数据。",
        userClaims: new[] { "user_level" });

这会将 *user_level* 声明作为请求的声明类型传递给 profile service，以便访问令牌的使用者可以使用此数据作为授权决策或业务逻辑的输入。

.. note:: 使用仅 scope 模型时，不会向令牌添加任何 aud（受众）声明，因为此概念不适用。 如果您需要 aud 声明，您可以在选项上启用 ``EmitStaticAudience`` 设置。 这将以 ``issuer_name/resources`` 格式发出 aud 声明。如果您需要更多地控制 aud 声明，请使用 API resources。

参数化 Scopes
^^^^^^^^^^^^^^^^^^^^
有时 scopes 具有特定的结构，例如，scope 名称带有附加参数： *transaction:id* 或 *read_patient:patientid*。

在这种情况下，您将创建一个没有参数部分的 scope 并将该名称分配给客户端，
但另外提供一些逻辑来在运行时使用 ``IScopeParser`` 接口或通过从我们的默认实现派生来解析 scope 的结构 ，例如::

    public class ParameterizedScopeParser : DefaultScopeParser
    {
        public ParameterizedScopeParser(ILogger<DefaultScopeParser> logger) : base(logger)
        {
        }

        public override void ParseScopeValue(ParseScopeContext scopeContext)
        {
            const string transactionScopeName = "transaction";
            const string separator = ":";
            const string transactionScopePrefix = transactionScopeName + separator;

            var scopeValue = scopeContext.RawValue;

            if (scopeValue.StartsWith(transactionScopePrefix))
            {
                // 进入这里的 scope 类似于“transaction:something”
                var parts = scopeValue.Split(separator, StringSplitOptions.RemoveEmptyEntries);
                if (parts.Length == 2)
                {
                    scopeContext.SetParsedValues(transactionScopeName, parts[1]);
                }
                else
                {
                    scopeContext.SetError("transaction scope 缺少 transaction 参数值");
                }
            }
            else if (scopeValue != transactionScopeName)
            {
                // 进入这里的 scope 不像“transaction”
                base.ParseScopeValue(scopeContext);
            }
            else
            {
                // 进入这里的 scope 恰好是“transaction”，也就是说我们忽略了它并且不将它包含在结果中
                scopeContext.SetIgnore();
            }
        }
    }

然后，您可以访问整个管道中的解析值，例如，在配置文件服务中::

    public class HostProfileService : IProfileService
    {
        public override async Task GetProfileDataAsync(ProfileDataRequestContext context)
        {
            var transaction = context.RequestedResources.ParsedScopes.FirstOrDefault(x => x.ParsedName == "transaction");
            if (transaction?.ParsedParameter != null)
            {
                context.IssuedClaims.Add(new Claim("transaction_id", transaction.ParsedParameter));
            }
        }
    }

API Resources
^^^^^^^^^^^^^
当 API 表面变得更大时，像上面使用的那样一个简单的 scope 列表可能是不可行的。

您通常需要引入某种命名空间来组织 scope 名称，也许您还想将它们组合在一起并获得一些更高级别的结构，例如访问令牌中的 *audience* 声明。
您可能还会遇到多个资源应支持相同 scope 名称的情况，而有时您明确希望将 scope 与特定资源隔离。

在 IdentityServer 中，``ApiResource`` 类允许一些额外的组织。 让我们使用以下 scope 定义::

    public static IEnumerable<ApiScope> GetApiScopes()
    {
        return new List<ApiScope>
        {
            // 发票 API 特有的 scopes
            new ApiScope(name: "invoice.read",   displayName: "读取您的发票。"),
            new ApiScope(name: "invoice.pay",    displayName: "支付您的发票。"),

            // 客户 API 特有的 scopes
            new ApiScope(name: "customer.read",    displayName: "读取您的客户信息。"),
            new ApiScope(name: "customer.contact", displayName: "允许联系您的客户之一。"),

            // 共享 scope
            new ApiScope(name: "manage", displayName: "提供对发票和客户数据的管理访问。")
        };
    }

使用 ``ApiResource`` ，您现在可以创建两个逻辑 API 及其对应的 scope::

    public static readonly IEnumerable<ApiResource> GetApiResources()
    { 
        return new List<ApiResource>
        {
            new ApiResource("invoice", "发票 API")
            {
                Scopes = { "invoice.read", "invoice.pay", "manage" }
            },
            
            new ApiResource("customer", "客户 API")
            {
                Scopes = { "customer.read", "customer.contact", "manage" }
            }
        };
    }

使用 API 资源分组可以为您提供以下附加功能

* 支持 JWT *aud* 声明。 audience 声明的值将是 API 资源的名称
* 支持在所有包含的 scopes 内添加通用用户声明
* 通过为资源分配 API 密钥来支持自省
* 支持为资源配置访问令牌签名算法

让我们看一下上述资源配置的一些示例访问令牌。

**客户端请求** invoice.read 和 invoice.pay::

    {
        "typ": "at+jwt"
    }.
    {
        "client_id": "client",
        "sub": "123",

        "aud": "invoice",
        "scope": "invoice.read invoice.pay"
    }

**客户端请求** invoice.read 和 customer.read::

    {
        "typ": "at+jwt"
    }.
    {
        "client_id": "client",
        "sub": "123",

        "aud": [ "invoice", "customer" ]
        "scope": "invoice.read customer.read"
    }

**客户端请求** manage::

    {
        "typ": "at+jwt"
    }.
    {
        "client_id": "client",
        "sub": "123",

        "aud": [ "invoice", "customer" ]
        "scope": "manage"
    }

迁移到 v4 的步骤
^^^^^^^^^^^^^^^^^^^^^
如上所述，从 v4 开始，scope 有自己的定义，并且可以选择被 resource 引用。 
在 v4 之前，scopes 始终包含在 resource 中。

要迁移到 v4，您需要拆分 scope 和 resource 注册，通常首先注册所有 scope
（例如，使用 ``AddInMemoryApiScopes`` 方法），然后注册 API resources（如果有）。
然后 API  resources 将按名称引用先前注册的 scope。

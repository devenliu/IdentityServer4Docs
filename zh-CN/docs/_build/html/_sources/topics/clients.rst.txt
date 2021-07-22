定义客户端
================
客户端表示可以从您的 IdentityServer 请求令牌的应用程序。

详细信息各不相同，但您通常会为客户端定义以下通用设置：

* 唯一的客户端 ID
* 密钥（如果需要的话）
* 允许与令牌服务的交互（称为授权类型）
* 身份和/或访问令牌被发送到的网络位置（称为重定向 URI）
* 允许客户端访问的 scopes（又名 resources）列表

.. Note:: 在运行时，通过 ``IClientStore`` 的实现来检索客户端。 这允许从任意数据源（如配置文件或数据库）加载它们。 对于本文档，我们将使用客户端存储的内存版本。 您可以通过 ``AddInMemoryClients`` 扩展方法在 ``ConfigureServices`` 中连接内存存储。

定义服务器到服务器通信的客户端
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
在这种情况下，没有交互式用户存在 —— 服务（又名客户端）想要与 API（又名 scope）通信::

    public class Clients
    {
        public static IEnumerable<Client> Get()
        {
            return new List<Client>
            {
                new Client
                {
                    ClientId = "service.client",                    
                    ClientSecrets = { new Secret("secret".Sha256()) },

                    AllowedGrantTypes = GrantTypes.ClientCredentials,
                    AllowedScopes = { "api1", "api2.read_only" }
                }
            };
        }
    }

.. _startClientsMVC:
定义用于使用身份验证和委托 API 访问的交互式应用程序
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
交互式应用程序（例如 Web 应用程序或原生桌面/移动）应用程序使用授权码流程。
此流程为您提供最佳安全性，因为访问令牌仅通过后端渠道调用传输（并允许您访问刷新令牌）::

    var interactiveClient = new Client
    {
        ClientId = "interactive",

        AllowedGrantTypes = GrantTypes.Code,
        AllowOfflineAccess = true,
        ClientSecrets = { new Secret("secret".Sha256()) },
        
        RedirectUris =           { "http://localhost:21402/signin-oidc" },
        PostLogoutRedirectUris = { "http://localhost:21402/" },
        FrontChannelLogoutUri =    "http://localhost:21402/signout-oidc",

        AllowedScopes = 
        {
            IdentityServerConstants.StandardScopes.OpenId,
            IdentityServerConstants.StandardScopes.Profile,
            IdentityServerConstants.StandardScopes.Email,

            "api1", "api2.read_only"
        },
    };

.. Note:: 请参阅 :ref:`grant types <refGrantTypes>` 主题，了解有关为您的客户端选择正确的授权类型的更多信息。

在 appsettings.json 中定义客户端
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``AddInMemoryClients`` 扩展方法还支持从 ASP.NET Core 配置文件添加客户端。 这允许您直接从 appsettings.json 文件定义静态客户端::

    "IdentityServer": {
      "IssuerUri": "urn:sso.company.com",
      "Clients": [
        {
          "Enabled": true,
          "ClientId": "local-dev",
          "ClientName": "Local Development",
          "ClientSecrets": [ { "Value": "<插入编码为 Base64 字符串的密钥的 Sha256 哈希>" } ],
          "AllowedGrantTypes": [ "client_credentials" ],
          "AllowedScopes": [ "api1" ],
        }
      ]
    }
    
然后将配置部分传递给 ``AddInMemoryClients`` 方法::

    AddInMemoryClients(configuration.GetSection("IdentityServer:Clients"))

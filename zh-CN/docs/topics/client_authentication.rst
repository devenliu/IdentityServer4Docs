客户端认证
=====================
在某些情况下，客户端需要使用 IdentityServer 进行身份验证，例如

* 机密应用程序（又名客户端）在令牌端点请求令牌
* 在内省端点验证参考令牌的 API

为此，您可以将密钥列表分配给客户端或 API 资源。

密钥解析和验证是身份服务器中的一个扩展点，开箱即用，
它支持共享秘密以及通过基本身份验证标头或 POST 正文传输共享秘密。

创建共享密钥
^^^^^^^^^^^^^^^^^^^^^^^^
以下代码设置哈希共享密钥::

    var secret = new Secret("secret".Sha256());

这个密钥现在可以分配给 ``客户端`` 或 ``ApiResource``。
Notice that both do not only support a single secret, but multiple. This is useful for secret rollover and rotation::

    var client = new Client
    {
        ClientId = "client",
        ClientSecrets = new List<Secret> { secret },

        AllowedGrantTypes = GrantTypes.ClientCredentials,
        AllowedScopes = 
        {
            "api1", "api2"
        }
    };

事实上，您还可以为密钥分配描述和到期日期。 
描述将用于日志记录，以及强制执行密钥生命周期的到期日期::

    var secret = new Secret(
        "secret".Sha256(), 
        "2016 secret", 
        new DateTime(2016, 12, 31));  

使用共享密钥进行身份验证
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
您可以将客户端 ID/密钥组合作为 POST 正文的一部分发送::

    POST /connect/token
    
    client_id=client1&
    client_secret=secret&
    ...

..或者作为一个基本的认证头::

    POST /connect/token
    
    Authorization: Basic xxxxx

    ...

您可以使用以下 C# 代码手动创建基本身份验证标头::

    var credentials = string.Format("{0}:{1}", clientId, clientSecret);
    var headerValue = Convert.ToBase64String(Encoding.UTF8.GetBytes(credentials));

    var client = new HttpClient();
    client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Basic", headerValue);

`IdentityModel <https://github.com/IdentityModel/IdentityModel>`_ 库有名为 ``TokenClient`` 和 ``IntrospectionClient`` 的辅助类，它们封装了身份验证和协议消息。

使用非对称密钥进行身份验证
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
There are other techniques to authenticate clients, e.g. based on public/private key cryptography.
IdentityServer includes support for private key JWT client secrets (see `RFC 7523 <https://tools.ietf.org/html/rfc7523>`_
and `here <https://openid.net/specs/openid-connect-core-1_0.html#ClientAuthentication>`_).

Secret extensibility typically consists of three things:

* a secret definition
* a secret parser that knows how to extract the secret from the incoming request
* a secret validator that knows how to validate the parsed secret based on the definition

Secret parsers and validators are implementations of the ``ISecretParser`` and ``ISecretValidator`` interfaces. 
To make them available to IdentityServer, you need to register them with the DI container, e.g.::

    builder.AddSecretParser<JwtBearerClientAssertionSecretParser>()
    builder.AddSecretValidator<PrivateKeyJwtSecretValidator>()

Our default private key JWT secret validator expects the full (leaf) certificate as base64 on the secret definition 
or an ESA/EC JSON web key::

    var client = new Client
    {
        ClientId = "client.jwt",
        ClientSecrets =
        {
            new Secret
            {
                Type = IdentityServerConstants.SecretTypes.X509CertificateBase64,
                Value = "MIIDATCCAe2gAwIBAgIQoHUYAquk9rBJcq8W+F0FAzAJBgUrDgMCHQUAMBIxEDAOBgNVBAMTB0RldlJvb3QwHhcNMTAwMTIwMjMwMDAwWhcNMjAwMTIwMjMwMDAwWjARMQ8wDQYDVQQDEwZDbGllbnQwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQDSaY4x1eXqjHF1iXQcF3pbFrIbmNw19w/IdOQxbavmuPbhY7jX0IORu/GQiHjmhqWt8F4G7KGLhXLC1j7rXdDmxXRyVJBZBTEaSYukuX7zGeUXscdpgODLQVay/0hUGz54aDZPAhtBHaYbog+yH10sCXgV1Mxtzx3dGelA6pPwiAmXwFxjJ1HGsS/hdbt+vgXhdlzud3ZSfyI/TJAnFeKxsmbJUyqMfoBl1zFKG4MOvgHhBjekp+r8gYNGknMYu9JDFr1ue0wylaw9UwG8ZXAkYmYbn2wN/CpJl3gJgX42/9g87uLvtVAmz5L+rZQTlS1ibv54ScR2lcRpGQiQav/LAgMBAAGjXDBaMBMGA1UdJQQMMAoGCCsGAQUFBwMCMEMGA1UdAQQ8MDqAENIWANpX5DZ3bX3WvoDfy0GhFDASMRAwDgYDVQQDEwdEZXZSb290ghAsWTt7E82DjU1E1p427Qj2MAkGBSsOAwIdBQADggEBADLje0qbqGVPaZHINLn+WSM2czZk0b5NG80btp7arjgDYoWBIe2TSOkkApTRhLPfmZTsaiI3Ro/64q+Dk3z3Kt7w+grHqu5nYhsn7xQFAQUf3y2KcJnRdIEk0jrLM4vgIzYdXsoC6YO+9QnlkNqcN36Y8IpSVSTda6gRKvGXiAhu42e2Qey/WNMFOL+YzMXGt/nDHL/qRKsuXBOarIb++43DV3YnxGTx22llhOnPpuZ9/gnNY7KLjODaiEciKhaKqt/b57mTEz4jTF4kIg6BP03MUfDXeVlM1Qf1jB43G2QQ19n5lUiqTpmQkcfLfyci2uBZ8BkOhXr3Vk9HIk/xBXQ="
            }
            new Secret
            {
                Type = IdentityServerConstants.SecretTypes.JsonWebKey,
                Value = "{'e':'AQAB','kid':'ZzAjSnraU3bkWGnnAqLapYGpTyNfLbjbzgAPbbW2GEA','kty':'RSA','n':'wWwQFtSzeRjjerpEM5Rmqz_DsNaZ9S1Bw6UbZkDLowuuTCjBWUax0vBMMxdy6XjEEK4Oq9lKMvx9JzjmeJf1knoqSNrox3Ka0rnxXpNAz6sATvme8p9mTXyp0cX4lF4U2J54xa2_S9NF5QWvpXvBeC4GAJx7QaSw4zrUkrc6XyaAiFnLhQEwKJCwUw4NOqIuYvYp_IXhw-5Ti_icDlZS-282PcccnBeOcX7vc21pozibIdmZJKqXNsL1Ibx5Nkx1F1jLnekJAmdaACDjYRLL_6n3W4wUp19UvzB1lGtXcJKLLkqB6YDiZNu16OSiSprfmrRXvYmvD8m6Fnl5aetgKw'}"
            }
        },

        AllowedGrantTypes = GrantTypes.ClientCredentials,
        AllowedScopes = { "api1", "api2" }
    };

授权请求对象
=========================
您可以将授权请求的参数作为单独的查询字符串 键/值 对提供，而不是将它们打包在签名的 JWT 中。
这使得参数可以防篡改，并且您可以对已在前端通道上的客户端进行身份验证。

您可以通过值或通过引用授权端点来传输它们 - 有关更多详细信息，请参阅 `规范 <https://openid.net/specs/openid-connect-core-1_0.html#JWTRequests>`_。

IdentityServer 要求对请求 JWT 进行签名。 我们支持 X509 证书和 JSON Web 密钥，例如::

    var client = new Client
    {
        ClientId = "foo",
        
        // 将此设置为 true 以仅接受签名请求
        RequireRequestObject = true,

        ClientSecrets = 
        {
            new Secret
            {
                // X509 证书 base64 编码
                Type = IdentityServerConstants.SecretTypes.X509CertificateBase64,
                Value = Convert.ToBase64String(cert.Export(X509ContentType.Cert))
            },
            new Secret
            {
                // RSA 密钥作为 JWK
                Type = IdentityServerConstants.SecretTypes.JsonWebKey,
                Value =
                    "{'e':'AQAB','kid':'ZzAjSnraU3bkWGnnAqLapYGpTyNfLbjbzgAPbbW2GEA','kty':'RSA','n':'wWwQFtSzeRjjerpEM5Rmqz_DsNaZ9S1Bw6UbZkDLowuuTCjBWUax0vBMMxdy6XjEEK4Oq9lKMvx9JzjmeJf1knoqSNrox3Ka0rnxXpNAz6sATvme8p9mTXyp0cX4lF4U2J54xa2_S9NF5QWvpXvBeC4GAJx7QaSw4zrUkrc6XyaAiFnLhQEwKJCwUw4NOqIuYvYp_IXhw-5Ti_icDlZS-282PcccnBeOcX7vc21pozibIdmZJKqXNsL1Ibx5Nkx1F1jLnekJAmdaACDjYRLL_6n3W4wUp19UvzB1lGtXcJKLLkqB6YDiZNu16OSiSprfmrRXvYmvD8m6Fnl5aetgKw'}"
            }
        }
    }

.. note:: Microsoft.IdentityModel.Tokens.JsonWebKeyConverter 有各种帮助程序将密钥转换为 JWK

通过引用传递请求 JWTs
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
如果使用 ``request_uri`` 参数，IdentityServer 将发出一个传出的 HTTP 调用以从指定的 URL 获取 JWT。

您可以自定义用于此传出连接的 HTTP 客户端，例如 添加缓存或重试逻辑（例如通过 Polly 库）::

    builder.AddJwtRequestUriHttpClient(client =>
    {
        client.Timeout = TimeSpan.FromSeconds(30);
    })
        .AddTransientHttpErrorPolicy(policy => policy.WaitAndRetryAsync(new[]
        {
            TimeSpan.FromSeconds(1),
            TimeSpan.FromSeconds(2),
            TimeSpan.FromSeconds(3)
        }));

.. note:: 默认情况下禁用请求 URI 处理。 在 :ref:`IdentityServer Options <refOptions>`` 下的 ``Endpoints` 上启用。 另请参阅 JAR `规范 <https://tools.ietf.org/html/draft-ietf-oauth-jwsreq-23#section-10.4>`_ 中的安全注意事项。

访问请求对象数据
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
您可以通过两种方式从请求对象中访问经过验证的数据

* 无论您在哪里可以访问 ``ValidatedAuthorizeRequest``，``RequestObjectValues`` 字典都会保存值
* 在 UI 代码中，您可以调用 ``IIdentityServerInteractionService.GetAuthorizationContextAsync``，生成的 ``AuthorizationRequest`` 对象也包含 ``RequestObjectValues`` 字典

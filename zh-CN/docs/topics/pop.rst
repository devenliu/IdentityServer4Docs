访问令牌所有权证明
=================================
默认情况下，OAuth 访问令牌称为 *bearer* 令牌。 这意味着它们不受客户的约束，任何拥有令牌的人都可以使用它（与现金相比）。

*所有权证明 Proof-of-Possession (short PoP)* 令牌绑定到请求令牌的客户端。 如果该令牌泄漏，则其他任何人都无法使用它（与信用卡相比 - 至少在理想世界中如此）。

有关更多历史和动机，请参阅 `此 <https://leastprivilege.com/2020/01/15/oauth-2-0-the-long-road-to-proof-of-possession-access-tokens/>`_ 博客文章。

IdentityServer 通过使用 :ref:`Mutual TLS 机制 <refMutualTLS>` 支持 PoP 令牌。
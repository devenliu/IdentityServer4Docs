工具
=====

``IdentityServerTools`` 类是为 IdentityServer 编写可扩展性代码时可能需要的有用内部工具的集合。 要使用它，请将其注入您的代码中，例如 控制器::

    public MyController(IdentityServerTools tools)
    {
        _tools = tools;
    }

``IssueJwtAsync`` 方法允许使用 IdentityServer 令牌创建引擎创建 JWT 令牌。 ``IssueClientJwtAsync`` 是一个更简单的版本，
用于为服务器到服务器的通信创建令牌（例如，当您必须从代码中调用受 IdentityServer 保护的 API 时）::

    public async Task<IActionResult> MyAction()
    {
        var token = await _tools.IssueClientJwtAsync(
            clientId: "client_id",
            lifetime: 3600,
            audiences: new[] { "backend.api" });

        // 更多的代码
    }
Windows 身份验证
======================
您可以通过多种方式在 ASP.NET Core（以及 IdentityServer）中启用 Windows 身份验证。

* 在 Windows 上使用 IIS 托管（进程内和进程外）
* 在 Windows 上使用 HTTP.SYS 托管
* 在任何使用协商身份验证处理程序的平台上（在 ASP.NET Core 3.0 中添加）

.. Note:: 我们只有 IIS 托管的文档。 如果您想为文档做出贡献，请打开一个 PR。 谢谢！

在使用 IIS 托管的 Windows 上
^^^^^^^^^^^^^^^^^^^^^^^^^^^^
当托管在 IIS 中时，典型的 ``CreateDefaultBuilder`` 主机设置支持基于 IIS 的 Windows 身份验证。
确保在 ``launchSettings.json`` 或您的 IIS 配置中启用了 Windows 身份验证。

IIS 集成层会将 Windows 身份验证处理程序配置到可通过身份验证服务调用的 DI 中。
通常在 IdentityServer 中，建议禁用自动行为。

这是在 ``ConfigureServices`` 中完成的（详细信息取决于进程内与进程外托管）::

    // 配置 IIS 进程外设置（参见 https://github.com/aspnet/AspNetCore/issues/14882）
    services.Configure<IISOptions>(iis =>
    {
        iis.AuthenticationDisplayName = "Windows";
        iis.AutomaticAuthentication = false;
    });

    // ..或配置 IIS 进程内设置
    services.Configure<IISServerOptions>(iis =>
    {
        iis.AuthenticationDisplayName = "Windows";
        iis.AutomaticAuthentication = false;
    });

您可以通过在 ``Windows`` 方案上调用 ``ChallengeAsync`` 来触发 Windows 身份验证（或者如果您想使用常量：``Microsoft.AspNetCore.Server.IISIntegration.IISDefaults.AuthenticationScheme``）。

这会将 ``Www-Authenticate`` 标头发送回浏览器，然后浏览器将重新加载当前的 URL，包括 Windows 身份。
当您在 ``Windows`` 方案上调用 ``AuthenticateAsync`` 并且返回的主体类型为 ``WindowsPrincipal`` 时，您可以判断 Windows 身份验证成功。

主体将具有用户和组 SID 以及 Windows 帐户名称等信息。 
以下代码段显示了如何触发身份验证，如果成功，则将信息转换为 temp-Cookie 方法的标准 ``ClaimsPrincipal``::

    private async Task<IActionResult> ChallengeWindowsAsync(string returnUrl)
    {
        // 查看是否已请求 Windows 身份验证并成功
        var result = await HttpContext.AuthenticateAsync("Windows");
        if (result?.Principal is WindowsPrincipal wp)
        {
            // 我们将发出外部 cookie，
            // 然后将用户重定向回外部回调，
            // 本质上，将 windows auth 视为与任何其他外部身份验证机制相同
            var props = new AuthenticationProperties()
            {
                RedirectUri = Url.Action("Callback"),
                Items =
                {
                    { "returnUrl", returnUrl },
                    { "scheme", "Windows" },
                }
            };

            var id = new ClaimsIdentity("Windows");

            // sid 是一个很好的子值
            id.AddClaim(new Claim(JwtClaimTypes.Subject, wp.FindFirst(ClaimTypes.PrimarySid).Value));

            // 帐户名称是我们最接近显示名称的名称
            id.AddClaim(new Claim(JwtClaimTypes.Name, wp.Identity.Name));

            // 添加组作为声明 —— 如果组数太大，请小心
            var wi = wp.Identity as WindowsIdentity;

            // 将组 SID 转换为显示名称
            var groups = wi.Groups.Translate(typeof(NTAccount));
            var roles = groups.Select(x => new Claim(JwtClaimTypes.Role, x.Value));
            id.AddClaims(roles);
            

            await HttpContext.SignInAsync(
                IdentityServerConstants.ExternalCookieAuthenticationScheme,
                new ClaimsPrincipal(id),
                props);
            return Redirect(props.RedirectUri);
        }
        else
        {
            // 触发 windows auth 
            // 因为 windows auth 不支持重定向 uri，
            // 当我们调用 challenge 时会重新触发这个 URL
            return Challenge("Windows");
        }
    }

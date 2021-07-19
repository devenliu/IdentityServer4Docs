.. _refSignOutExternal:
注销外部身份提供商
=======================================

当用户从 IdentityServer :ref:`注销 <refSignOut>` 时，如果他们使用了 :ref:`外部身份提供商 <refExternalIdentityProviders>` 登录，则很可能还应该将他们重定向到注销外部提供商。
并非所有外部提供商都支持注销，因为这取决于他们支持的协议和功能。

检测用户是否必须重定向到外部身份提供商以进行注销，通常是通过使用在 IdentityServer 的 cookie 中发出的 ``idp`` 声明来完成的。
设置到此声明中的值是相应身份验证中间件的 ``AuthenticationScheme``。
在注销时，将咨询此声明以了解是否需要外部注销。

由于正常注销工作流已经需要清理和状态管理，因此将用户重定向到外部身份提供者是有问题的。
完成 IdentityServer 的正常注销和清理过程的唯一方法是，从外部身份提供商请求在其注销后，将用户重定向回 IdentityServer。
并非所有外部提供商都支持注销后重定向，因为这取决于它们支持的协议和功能。

注销时的工作流程是撤销 IdentityServer 的身份验证 cookie，然后重定向到请求注销后重定向的外部提供商。
注销后重定向应保持 :ref:`此处 <refSignOut>` 描述的必要注销状态（即 ``logoutId`` 参数值）。
要在外部提供商注销后重定向回 IdentityServer，例如，当使用 ASP.NET Core 的 ``SignOutAsync`` API 时，应在 ``AuthenticationProperties`` 上使用 ``RedirectUri``::

    [HttpPost]
    [ValidateAntiForgeryToken]
    public async Task<IActionResult> Logout(LogoutInputModel model)
    {
        // 建立一个模型，以便注销页面知道要显示什么
        var vm = await _account.BuildLoggedOutViewModelAsync(model.LogoutId);

        var user = HttpContext.User;
        if (user?.Identity.IsAuthenticated == true)
        {
            // 删除本地认证cookie
            await HttpContext.SignOutAsync();

            // 引发注销事件
            await _events.RaiseAsync(new UserLogoutSuccessEvent(user.GetSubjectId(), user.GetName()));
        }

        // 检查我们是否需要在上游身份提供商处触发注销
        if (vm.TriggerExternalSignout)
        {
            // 构建一个返回 URL，以便上游提供商在用户注销后重定向回我们。
            // 这样我们就可以完成单点注销处理。
            string url = Url.Action("Logout", new { logoutId = vm.LogoutId });

            // 这将触发重定向到外部提供商以进行注销
            return SignOut(new AuthenticationProperties { RedirectUri = url }, vm.ExternalAuthenticationScheme);
        }

        return View("LoggedOut", vm);
    }

一旦用户从外部提供商注销，然后重定向回来，IdentityServer 上的正常注销处理就应该执行，这包括处理 ``logoutId`` 并进行所有必要的清理。

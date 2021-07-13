.. _refQuickstartOverview:
概述
========
快速入门为各种常见的 IdentityServer 场景提供了分步说明。
他们从绝对的基础开始，然后变得更加复杂 —— 
建议您按顺序进行。

* 将 IdentityServer 添加到 ASP.NET Core 应用程序
* 配置 IdentityServer
* 为各种客户端颁发令牌
* 保护 Web 应用程序和 API
* 添加对基于 EntityFramework 的配置的支持
* 添加对 ASP.NET Identity 的支持

每个快速入门都有一个参考解决方案 —— 您可以在 `samples <https://github.com/IdentityServer/IdentityServer4/tree/main/samples/Quickstarts>`_ 文件夹中找到代码。

准备工作
^^^^^^^^^^^
您应该做的第一件事是安装我们的模板::

    dotnet new -i IdentityServer4.Templates

它们将用作各种教程的起点。

.. note:: 如果您使用私有 NuGet 源，请不要忘记添加 --nuget-source 参数: --nuget-source https://api.nuget.org/v3/index.json

OK —— 让我们开始吧！

.. note:: 快速入门针对 IdentityServer 4.x and ASP.NET Core 3.1.x —— 还有 `ASP.NET Core 2 <http://docs.identityserver.io/en/aspnetcore2/quickstarts/0_overview.html>`_ 和 `ASP.NET Core 1 <http://docs.identityserver.io/en/aspnetcore1/quickstarts/0_overview.html>`_ 的快速入门.

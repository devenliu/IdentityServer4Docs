打包和构建
====================

IdentityServer 由许多 nuget 包组成。

IdentityServer4 主仓库
^^^^^^^^^^^^^^^^^^^^^^
`github <https://github.com/identityserver/IdentityServer4>`_

包含核心 IdentityServer 对象模型、服务和中间件以及 EntityFramework 和 ASP.NET Identity 集成。

nugets:

* `IdentityServer4 <https://www.nuget.org/packages/IdentityServer4/>`_
* `IdentityServer4.EntityFramework <https://www.nuget.org/packages/IdentityServer4.EntityFramework>`_
* `IdentityServer4.AspNetIdentity <https://www.nuget.org/packages/IdentityServer4.AspNetIdentity>`_

快速入门用户界面
^^^^^^^^^^^^^^^
`github <https://github.com/IdentityServer/IdentityServer4.Quickstart.UI>`_

包含一个简单的入门 UI，包括登录、注销和同意页面。

访问令牌验证处理程序
^^^^^^^^^^^^^^^^^^
`nuget <https://www.nuget.org/packages/IdentityServer4.AccessTokenValidation>`_ | `github <https://github.com/IdentityServer/IdentityServer4.AccessTokenValidation>`_

用于验证 API 中的令牌的 ASP.NET Core 身份验证处理程序。处理程序允许在同一 API 中同时支持 JWT 和引用令牌。

模板
^^^^
`nuget <https://www.nuget.org/packages/IdentityServer4.Templates>`_ | `github <https://github.com/IdentityServer/IdentityServer4.Templates>`_

包含 dotnet CLI 的模板。

开发版本
^^^^^^^
此外，我们将 CI 构建发布到我们的包存储库。
将以下 ``nuget.config`` 添加到您的项目中::

    <?xml version="1.0" encoding="utf-8"?>
        <configuration>
            <packageSources>
                <clear />
                <add key="IdentityServer CI" value="https://www.myget.org/F/identity/api/v3/index.json" />
            </packageSources>
        </configuration>

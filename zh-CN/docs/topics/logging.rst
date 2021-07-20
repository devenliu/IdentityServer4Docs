
日志
=======
IdentityServer 使用 ASP.NET Core 提供的标准日志记录工具。
Microsoft `文档 <https://docs.microsoft.com/en-us/aspnet/core/fundamentals/logging>`_ 有一个很好的介绍和内置日志提供程序的描述。

我们大致遵循 Microsoft 的日志级别使用指南：

* ``Trace`` 用于仅对开发人员解决问题有价值的信息。 这些消息可能包含敏感的应用程序数据，如令牌，不应在生产环境中启用。
* ``Debug`` 用于遵循内部流程并理解做出某些决定的原因。 在开发和调试期间有短期的用处。
* ``Information`` 用于跟踪应用程序的一般流程。 这些日志通常具有一些长期价值。
* ``Warning`` 用于应用程序流中的异常或意外事件。 这些可能包括不会导致应用程序停止但可能需要调查的错误或其他情况。
* ``Error`` 用于无法处理的错误和异常。 示例：协议请求验证失败。
* ``Critical`` 用于需要立即关注的故障。 示例：缺少存储实现、无效的密钥材料...

Serilog 的设置
^^^^^^^^^^^^^^^^^
我们个人非常喜欢 `Serilog <https://serilog.net/>`_ 和 ``Serilog.AspNetCore`` 包。 试一试::

    public class Program
    {
        public static int Main(string[] args)
        {
            Activity.DefaultIdFormat = ActivityIdFormat.W3C;

            Log.Logger = new LoggerConfiguration()
                .MinimumLevel.Debug()
                .MinimumLevel.Override("Microsoft", LogEventLevel.Warning)
                .MinimumLevel.Override("Microsoft.Hosting.Lifetime", LogEventLevel.Information)
                .MinimumLevel.Override("System", LogEventLevel.Warning)
                .MinimumLevel.Override("Microsoft.AspNetCore.Authentication", LogEventLevel.Information)
                .Enrich.FromLogContext()
                .WriteTo.Console(outputTemplate: "[{Timestamp:HH:mm:ss} {Level}] {SourceContext}{NewLine}{Message:lj}{NewLine}{Exception}{NewLine}", theme: AnsiConsoleTheme.Code)
                .CreateLogger();

            try
            {
                Log.Information("Starting host...");
                CreateHostBuilder(args).Build().Run();
                return 0;
            }
            catch (Exception ex)
            {
                Log.Fatal(ex, "Host terminated unexpectedly.");
                return 1;
            }
            finally
            {
                Log.CloseAndFlush();
            }
        }

        public static IHostBuilder CreateHostBuilder(string[] args) =>
            Microsoft.Extensions.Hosting.Host.CreateDefaultBuilder(args)
                .UseSerilog()
                .ConfigureWebHostDefaults(webBuilder =>
                {
                    webBuilder.UseStartup<Startup>();
                });
    }
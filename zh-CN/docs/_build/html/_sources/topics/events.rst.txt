.. _refEvents:
事件
======
虽然日志记录是更低级别的 “printf” 样式 —— 但事件表示有关 IdentityServer 中某些操作的更高级别信息。
事件是结构化数据，包括事件 ID、成功/失败信息、类别和详细信息。
这使得查询和分析它们并提取可用于进一步处理的有用信息变得容易。

事件与诸如 `ELK <https://www.elastic.co/webinars/introduction-elk-stack>`_、 `Seq <https://getseq.net/>`_ 或 `Splunk <https://www.splunk.com/>`_ 之类的事件存储配合得很好。

发出事件
^^^^^^^^^^^^^^^
默认情况下没有打开事件 —— 但可以在 ``ConfigureServices`` 方法中全局配置，例如::

    services.AddIdentityServer(options =>
    {
        options.Events.RaiseSuccessEvents = true;
        options.Events.RaiseFailureEvents = true;
        options.Events.RaiseErrorEvents = true;
    });

要发出事件，请使用 DI 容器中的 ``IEventService`` 并调用 ``RaiseAsync`` 方法，例如::

    public async Task<IActionResult> Login(LoginInputModel model)
    {
        if (_users.ValidateCredentials(model.Username, model.Password))
        {
            // 发出带有 subject ID 和 username 的身份验证 cookie
            var user = _users.FindByUsername(model.Username);
            await _events.RaiseAsync(new UserLoginSuccessEvent(user.Username, user.SubjectId, user.Username));
        }
        else
        {
            await _events.RaiseAsync(new UserLoginFailureEvent(model.Username, "invalid credentials"));
        }
    }

自定义 sinks
^^^^^^^^^^^^
我们的默认事件接收器将简单地将事件类序列化为 JSON，并将其转发到 ASP.NET Core 日志记录系统。
如果要连接到自定义事件存储，请实现 ``IEventSink`` 接口并将其注册到 DI。

以下示例使用 `Seq <https://getseq.net/>`_ 来发出事件::

     public class SeqEventSink : IEventSink
    {
        private readonly Logger _log;

        public SeqEventSink()
        {
            _log = new LoggerConfiguration()
                .WriteTo.Seq("http://localhost:5341")
                .CreateLogger();
        }

        public Task PersistAsync(Event evt)
        {
            if (evt.EventType == EventTypes.Success ||
                evt.EventType == EventTypes.Information)
            {
                _log.Information("{Name} ({Id}), Details: {@details}",
                    evt.Name,
                    evt.Id,
                    evt);
            }
            else
            {
                _log.Error("{Name} ({Id}), Details: {@details}",
                    evt.Name,
                    evt.Id,
                    evt);
            }

            return Task.CompletedTask;
        }
    }

将 ``Serilog.Sinks.Seq`` 包添加到您的主机以使上述代码工作。

内置事件
^^^^^^^^^^^^^^^
IdentityServer 中定义了以下事件:

``ApiAuthenticationFailureEvent`` & ``ApiAuthenticationSuccessEvent``
    在内省端点处为成功/失败的 API 身份验证引发。
``ClientAuthenticationSuccessEvent`` & ``ClientAuthenticationFailureEvent``
    在令牌端点处为成功/失败的客户端身份验证引发。
``TokenIssuedSuccessEvent`` & ``TokenIssuedFailureEvent``
    为尝试请求身份令牌、访问令牌、刷新令牌和授权代码的成功/失败引发。
``TokenIntrospectionSuccessEvent`` & ``TokenIntrospectionFailureEvent``
    为成功的令牌自省请求引发。
``TokenRevokedSuccessEvent``
    为成功的令牌吊销请求引发。
``UserLoginSuccessEvent`` & ``UserLoginFailureEvent``
    由成功/失败的用户登录的快速入门 UI 引发。
``UserLogoutSuccessEvent``
    为成功的注销请求引发。
``ConsentGrantedEvent`` & ``ConsentDeniedEvent``
    在同意 UI 中引发。
``UnhandledExceptionEvent``
    为未处理的异常引发。
``DeviceAuthorizationFailureEvent`` & ``DeviceAuthorizationSuccessEvent``
    为成功/失败的设备授权请求引发。

自定义事件
^^^^^^^^^^^^^
您可以创建自己的事件并通过我们的基础设施发出它们。

您需要从我们的基础 ``Event`` 类派生，该类注入上下文信息，如 activity ID、timestamp 等。
然后您的派生类可以添加特定于事件上下文的任意数据字段::

    public class UserLoginFailureEvent : Event
    {
        public UserLoginFailureEvent(string username, string error)
            : base(EventCategories.Authentication,
                    "User Login Failure",
                    EventTypes.Failure, 
                    EventIds.UserLoginFailure,
                    error)
        {
            Username = username;
        }

        public string Username { get; set; }
    }

title: "Logging Execution Time Using AOP"
date: 2010-02-27 11:42:22
tags:

- C#

---

What happens if your client complains that your application is running very slow!!! or in your load/stress testing you found that some functionalities are very slow in executing than expected. This is the time where you go for profiling the execution, to analyse the root cause of these issues.

So how we could develop a profiler, where we don’t have to wrap our normal code in a profiling code.

Before going to create the profiler, we have to decide where to put the profiled information. In this tutorial, I am making use of [Log4Net] as underlying layer to store this information. If you have not used [Log4Net] before, I suggest you to read http://www.beefycode.com/post/Log4Net-Tutorial-pt-1-Getting-Started.aspx as a starting point.

With the help of **AOP (Aspect-Oriented Programming)** we could do the profiling task without interrupting the actual code.

{% blockquote Wikipedia %}
AOP is a programming paradigm in which secondary or supporting functions are isolated from the main program's business logic
{% endblockquote %}

So in order bring the AOP functionality into this application, I am going to use a third party library [PostSharp] which I believe this is one of the best that is available in the market.

So, now we have got the basic things to start with and now let’s start coding….

Start a new solution in visual studio and add a new console application project to it. Then add the below references to the newly created project

Add reference to the _Log4Net.dll_, _PostSharp.Laos.dll_ and _PostSharp.Public.dll_ (Please read https://www.postsharp.net/documentation to get the basic installation procedure). Next, create a new attribute class called `ProfileMethodAttribute` – this class is responsible for doing the profiling work. Make sure that you have decorated this class with `Serializable` attribute

```cs
[Serializable]
public class ProfileMethodAttribute : OnMethodBoundaryAspect
{
  public override void OnEntry(MethodExecutionEventArgs eventArgs)
  {
      ........
  }

  public override void OnExit(MethodExecutionEventArgs eventArgs)
  {
     ......
  }
}
```

This class actually derives from `OnMethodBoundaryAspect`, it has got two methods `OnEntry` and `OnExit` which we needed. These method will be called before the start of a method execution and at the end of method execution respectively, when this attribute is decorated against a method.

When a call comes to `OnEntry` method, we will first log the execution call using the `LoggerHelper`, then start a clock using another helper class `Profiler`

```cs
public class LoggerHelper
{
    /// <summary>
    /// Static instance of ILogger.
    /// </summary>
    private static ILog logger;

    /// <summary>
    /// Initializes static members of the <see cref="LoggerHelper"/> class.
    /// </summary>
    static LoggerHelper()
    {
        log4net.Config.XmlConfigurator.Configure();
        logger = LogManager.GetLogger(typeof(Program));
    }

    /// <summary>
    /// Logs the specified message.
    /// </summary>
    /// <param name="message">The message.</param>
    public static void Log(string message)
    {
        string enableProfiling = ConfigurationManager.AppSettings["EnableProfiling"];
        if (string.IsNullOrEmpty(enableProfiling) || enableProfiling.ToLowerInvariant() == "true")
        {
            logger.Debug(message);
        }
    }

    /// <summary>
    /// Logs the specified method name.
    /// </summary>
    /// <param name="methodName">Name of the method.</param>
    /// <param name="url">The URL to log.</param>
    /// <param name="executionFlowMessage">The execution flow message.</param>
    /// <param name="actualMessage">The actual message.</param>
    public static void Log(string methodName, string url, string executionFlowMessage, string actualMessage)
    {
        Log(ConstructLog(methodName, url, executionFlowMessage, actualMessage));
    }

    /// <summary>
    /// Logs the specified method name.
    /// </summary>
    /// <param name="methodName">Name of the method.</param>
    /// <param name="url">The URL to log.</param>
    /// <param name="executionFlowMessage">The execution flow message.</param>
    /// <param name="actualMessage">The actual message.</param>
    /// <param name="executionTime">The execution time.</param>
    public static void Log(string methodName, string url, string executionFlowMessage, string actualMessage, int executionTime)
    {
        Log(ConstructLog(methodName, url, executionFlowMessage, actualMessage, executionTime));
    }

    /// <summary>
    /// Constructs the log.
    /// </summary>
    /// <param name="methodName">Name of the method.</param>
    /// <param name="url">The URL to be logged.</param>
    /// <param name="executionFlowMessage">The execution flow message.</param>
    /// <param name="actualMessage">The actual message.</param>
    /// <returns>Formatted string.</returns>
    private static string ConstructLog(string methodName, string url, string executionFlowMessage, string actualMessage)
    {
        var sb = new StringBuilder();

        if (!string.IsNullOrEmpty(methodName))
        {
            sb.AppendFormat("MethodName : {0}, ", methodName);
        }

        if (!string.IsNullOrEmpty(url))
        {
            sb.AppendFormat("Url : {0}, ", url);
        }

        if (!string.IsNullOrEmpty(executionFlowMessage))
        {
            sb.AppendFormat("ExecutionFlowMessage : {0}, ", executionFlowMessage);
        }

        if (!string.IsNullOrEmpty(actualMessage))
        {
            sb.AppendFormat("ActualMessage : {0}, ", actualMessage);
        }

        string message = sb.ToString();

        message = message.Remove(message.Length - 2);

        return message;
    }

    /// <summary>
    /// Constructs the log.
    /// </summary>
    /// <param name="methodName">Name of the method.</param>
    /// <param name="url">The URL to be logged.</param>
    /// <param name="executionFlowMessage">The execution flow message.</param>
    /// <param name="actualMessage">The actual message.</param>
    /// <param name="executionTime">The execution time.</param>
    /// <returns>Formatted string.</returns>
    private static string ConstructLog(string methodName, string url, string executionFlowMessage, string actualMessage, int executionTime)
    {
        var sb = new StringBuilder();

        sb.Append(ConstructLog(methodName, url, executionFlowMessage, actualMessage));
        sb.AppendFormat(", ExecutionTime : {0}", executionTime);

        return sb.ToString();
    }
}
```

`LoggerHelper` uses the `Log4Net` objects to log the message to configured location. `Profiler` class implemented like below

```cs
/// <summary>
/// Helper class that wraps the timer based functionalities.
/// </summary>
internal static class Profiler
{
    /// <summary>
    /// Lock object.
    /// </summary>
    private static readonly object SyncLock = new object();

    /// <summary>
    /// Variable that tracks the time.
    /// </summary>
    private static readonly Dictionary<int, Stack<long>> ProfilePool;

    /// <summary>
    /// Initializes static members of the <see cref="Profiler"/> class.
    /// </summary>
    static Profiler()
    {
        ProfilePool = new Dictionary<int, Stack<long>>();
    }

    /// <summary>
    /// Starts this timer.
    /// </summary>
    public static void Start()
    {
        lock (SyncLock)
        {
            int currentThreadId = Thread.CurrentThread.ManagedThreadId;
            if (ProfilePool.ContainsKey(currentThreadId))
            {
                ProfilePool[currentThreadId].Push( Environment.TickCount );
            }
            else
            {
                var timerStack = new Stack<long>();
                timerStack.Push(DateTime.UtcNow.Ticks);
                ProfilePool.Add(currentThreadId, timerStack);
            }
        }
    }

    /// <summary>
    /// Stops timer and calculate the execution time.
    /// </summary>
    /// <returns>Execution time in milli seconds</returns>
    public static int Stop()
    {
        lock (SyncLock)
        {
            long currentTicks = DateTime.UtcNow.Ticks;
            int currentThreadId = Thread.CurrentThread.ManagedThreadId;

            if (ProfilePool.ContainsKey(currentThreadId))
            {
                long ticks = ProfilePool[currentThreadId].Pop();
                if (ProfilePool[currentThreadId].Count == 0)
                {
                    ProfilePool.Remove(currentThreadId);
                }

                var timeSpan = new TimeSpan(currentTicks - ticks);

                return (int)timeSpan.TotalMilliseconds;
            }
        }

        return 0;
    }
}
```

which stores the starting tick and calculate the time taken to execute when `Stop` is called.

Below is code snippet from `OnEntry` method

```cs
public override void OnEntry(MethodExecutionEventArgs eventArgs)
{
    this._methodName = this.ExcludeMethodName ? string.Empty : eventArgs.Method.Name;
    this._url = this.IncludeUrl ? string.Empty : (HttpContext.Current == null) ? string.Empty : HttpContext.Current.Request.Url.ToString();

    LoggerHelper.Log(this._methodName, this._url, this.EntryMessage, this.Message);
    Profiler.Start();
}
```

And `OnExist`, is almost similar expect there we will stop the profile timer.

```cs
public override void OnExit(MethodExecutionEventArgs eventArgs)
{
    int count = Profiler.Stop();
    LoggerHelper.Log(this._methodName, this._url, this.EntryMessage, this.Message, count);
}
```

Next step is to use this and see whether it is logged properly. In order enable profiling for a method, you just needs to decorate it with the `ProfileMethod` attribute. Like below

```cs
[ProfileMethod()]
public void SimpleMethod()
{
    Thread.Sleep(5000);
}
```

Then if you run the application and calls this above method, a log entry will be created where you have configured which looks like below

PROFILING 2010-02-26 00:19:59,838 [1] Log MethodName : SimpleMethod
PROFILING 2010-02-26 00:20:04,865 [1] Log MethodName : SimpleMethod, ExecutionTime : 5002

Please least me know, if this helped you.

> Download the demo code from [here](/download/ProfilingSample.zip)

[log4net]: http://logging.apache.org/log4net/
[postsharp]: https://www.postsharp.net

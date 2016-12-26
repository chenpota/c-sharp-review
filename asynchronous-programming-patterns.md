There are three asynchronous programming patterns in C#:  
* Asynchronous Programming Model (APM)  
* Event-based Programming Model (EPM)  
* Task-based Asynchronous Pattern (TAP)  

This article shows their differences briefly.

# Asynchronous Programming Model (APM)

APM would expose the two methods: *BeginDoSomething()* which returns *IAsyncResult* and *EndDoSomething()* which returns a result.  

```csharp
using System;
using System.Runtime.Remoting.Messaging;

public class Translator
{
    public IAsyncResult BeginIntToString(int integer, AsyncCallback callback = null);

    public string EndIntToString(IAsyncResult ar);
}
```

# Event-based Programming Model (EPM)

APM would expose a method named *DoSomethingAsync() and an event to callback the result.

```csharp
using System;

public class CustomEventArgs : EventArgs
{
    public string Result
    {
        get;
        internal set;
    }
}

public class Translator
{
    public event EventHandler<CustomEventArgs> IntToStringCompleted;

    public void IntToStringAsync(int number);
}
```

# Task-based Asynchronous Pattern (TAP)

TAP would expose a method *DoSomethingAsync() which returns a Task.

```csharp
using System;
using System.Threading.Tasks;

public class Translator
{
    public Task<string> IntToStringAsync(int number);
}
```

The [article]() shows APM, EPM, and TAP implementations using **wait-until-done** manner.  
The [article]() shows APM and TAP implementations using **polling** manner.  
The [article]() shows APM, EPM, and TAP implementations using **callback** manner.

------------------------------

# Complete Source Code
## APM

```csharp
using System;
using System.Runtime.Remoting.Messaging;

namespace AsynchronousProgrammingModel
{
    public class Translator
    {
        public IAsyncResult BeginIntToString(int integer, AsyncCallback callback = null)
        {
            AsyncCaller caller = new AsyncCaller(IntToString);

            return caller.BeginInvoke(integer, callback, this);
        }

        public string EndIntToString(IAsyncResult ar)
        {
            AsyncCaller caller = ((AsyncResult)ar).AsyncDelegate as AsyncCaller;

            return caller.EndInvoke(ar);
        }

        private delegate string AsyncCaller(int integer);

        private string IntToString(int integer)
        {
            return integer.ToString();
        }
    }
}
```
## EPM

```csharp
using System;

namespace EventBasedAsynchronousPattern
{
    public class Translator
    {
        public event EventHandler<CustomEventArgs> IntToStringCompleted;

        public void IntToStringAsync(int number)
        {
            Action<int> action = IntToString;

            action.BeginInvoke(number, null, null);
        }

        private void IntToString(int number)
        {
            CustomEventArgs args = new CustomEventArgs()
            {
                Result = number.ToString(),
            };

            EventHandler<CustomEventArgs> completed = IntToStringCompleted;

            completed?.Invoke(this, args);
        }
    }
}
```

## TAP

```csharp
using System;
using System.Threading.Tasks;

namespace TaskBasedAsynchronousPattern
{
    public class Translator
    {
        public Task<string> IntToStringAsync(int number)
        {
            TaskCompletionSource<string> tcs = new TaskCompletionSource<string>();

            Func<int, string> func = integer => integer.ToString();

            func.BeginInvoke(
                number,
                (ar) =>
                {
                    tcs.SetResult(func.EndInvoke(ar));
                },
                null);

            return tcs.Task;
        }
    }
}
```

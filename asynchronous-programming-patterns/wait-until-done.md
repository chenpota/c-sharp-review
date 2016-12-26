This article shows how to use Asynchronous Programming Model (APM), Event-based Programming Model (EPM), and Task-based Asynchronous Pattern (TAP) respectively to implement wait-until-done manner in C# asynchronous programming.

# Asynchronous Programming Model (APM)

```csharp
public class Translator
{
    public IAsyncResult BeginIntToString(int integer, AsyncCallback callback = null);

    public string EndIntToString(IAsyncResult ar);
}

class Program
{
	static void Main(string[] args)
	{
		Translator translator = new Translator();

		IAsyncResult ar = translator.BeginIntToString(1, null);

		string result = translator.EndIntToString(ar);

		Console.WriteLine("wait-until-done: " + result);
	}
}
```

# Event-based Programming Model (EPM)

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

class Program
{
    static void Main(string[] args)
    {
        Translator translator = new Translator();

        translator.IntToStringCompleted += WaitUntilDone;

        lock (_syncObj)
        {
            translator.IntToStringAsync(1);

            Monitor.Wait(_syncObj);
        }
	}

	private static object _syncObj = new object();

    private static void WaitUntilDone(object sender, CustomEventArgs e)
    {
        Translator translator = sender as Translator;

        translator.IntToStringCompleted -= WaitUntilDone;

        Console.WriteLine("wait-until-done: " + e.Result);

        lock (_syncObj)
        {
            Monitor.Pulse(_syncObj);
        }
    }
}
```

# Task-based Asynchronous Pattern (TAP)

```csharp
using System;
using System.Threading.Tasks;

public class Translator
{
    public Task<string> IntToStringAsync(int number);
}

class Program
{
    static void Main(string[] args)
    {
        Translator translator = new Translator();

        Task<string> task = translator.IntToStringAsync(1);

        task.Wait();
        
        Console.WriteLine("wait-until-done: " + task.Result);
    }
}
```

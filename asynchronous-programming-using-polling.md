This article shows how to use Asynchronous Programming Model (APM) and Task-based Asynchronous Pattern (TAP) respectively to implement polling manner in C# asynchronous programming.

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

		IAsyncResult ar = translator.BeginIntToString(2, null);

		while (ar.IsCompleted == false)
		{
			// do something
		}

		string result = translator.EndIntToString(ar);

		Console.WriteLine("polling: " + result);
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

		Task<string> task = translator.IntToStringAsync(2);

		while(task.IsCompleted==false)
		{
			// do something
		}

		Console.WriteLine("polling: " + task.Result);
    }
}
```
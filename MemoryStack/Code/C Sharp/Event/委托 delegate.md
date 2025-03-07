#委托 

在C#中,委托是一种类型安全的函数指针,可以用来封装方法的引用,允许将方法作为参数传递,或者赋值给变量

## 使用流程

1.定义委托
```cs
public delegate void MyDelegate(string message);
```

2.创建委托实例
创建一个委托实例，并将其指向一个符合委托签名的方法。
```cs
public class Example
{
    public void PrintMessage(string message)
    {
        Debug.Log(message);
    }

    public void Run()
    {
        // 创建委托实例
        MyDelegate del = new MyDelegate(PrintMessage);
        
        // 调用委托
        del("Hello, World!");
    }
}
```

3.使用匿名方法或 Lambda 表达式
你也可以使用匿名方法或 Lambda 表达式来创建委托实例。
```cs
public class Example
{
    public void Run()
    {
        // 使用 Lambda 表达式
        MyDelegate del = (message) => Debug.Log(message);
        
        // 调用委托
        del("Hello, World!");
    }
}
```

4.多播委托
委托可以指向多个方法，这称为多播委托。你可以使用 += 和 -= 操作符来添加或移除方法。
```cs
public class Example
{
    public void PrintMessage1(string message)
    {
        Debug.Log("Message 1: " + message);
    }

    public void PrintMessage2(string message)
    {
        Debug.Log("Message 2: " + message);
    }

    public void Run()
    {
        MyDelegate del = PrintMessage1;
        del += PrintMessage2; // 添加第二个方法

        // 调用委托
        del("Hello, World!");
    }
}
```

5.委托作为参数
你可以将委托作为参数传递给方法。
```cs
public class Example
{
    public void ExecuteAction(MyDelegate action)
    {
        action("Executing action...");
    }

    public void Start()
    {
        ExecuteAction(PrintMessage);
    }
}
```

6.委托与事件
委托通常与事件一起使用。你可以定义一个事件，并使用委托来处理事件。
```cs
public class Example
{
    public delegate void MyEventHandler(string message);
    public event MyEventHandler MyEvent;

    public void Start()
    {
        MyEvent += PrintMessage; // 订阅事件
        MyEvent?.Invoke("Event triggered!"); // 触发事件
    }
}
```
# Singleton Design Pattern in .NET Core

The **Singleton Design Pattern** ensures that only one instance of a class is created and shared throughout the application. This is useful for managing resources such as database connections, logging, or configuration settings.

---

## Key Characteristics of Singleton
1. Restricts the instantiation of a class to one object.
2. Ensures thread safety in multi-threaded environments.
3. Provides a global access point to the instance.

---

## Steps to Implement Singleton Pattern in .NET Core
1. **Static Member**: Holds the single instance of the class.
2. **Private Constructor**: Prevents instantiation from outside the class.
3. **Public Static Factory Method**: Provides access to the single instance.

---

## Implementations of Singleton Pattern

### 1. Eager Loading (Early Initialization)
The instance is created as soon as the application starts.

```csharp
public sealed class SingletonEager
{
    private static readonly SingletonEager _instance = new SingletonEager();

    // Private constructor prevents instantiation from outside.
    private SingletonEager() { }

    // Static property to provide access to the instance.
    public static SingletonEager Instance => _instance;
}

// Usage
var singleton = SingletonEager.Instance;
```
**Pros**:
- Simple to implement.
- No need to handle thread safety.

**Cons**:
- Instance is created even if it’s not used.

---

### 2. Lazy Loading
The instance is created only when it's first requested.

```csharp
public sealed class SingletonLazy
{
    private static SingletonLazy _instance;

    private SingletonLazy() { }

    public static SingletonLazy Instance
    {
        get
        {
            if (_instance == null)
            {
                _instance = new SingletonLazy(); // Lazy initialization
            }
            return _instance;
        }
    }
}

// Usage
var singleton = SingletonLazy.Instance;
```
**Pros**:
- Instance is created only when needed.

**Cons**:
- Not thread-safe; multiple threads might create separate instances.

---

### 3. Thread-Safe Singleton Using `lock`
Thread safety is ensured using a `lock`.

```csharp
public sealed class SingletonThreadSafe
{
    private static SingletonThreadSafe _instance;
    private static readonly object _lock = new object();

    private SingletonThreadSafe() { }

    public static SingletonThreadSafe Instance
    {
        get
        {
            lock (_lock)
            {
                if (_instance == null)
                {
                    _instance = new SingletonThreadSafe();
                }
                return _instance;
            }
        }
    }
}

// Usage
var singleton = SingletonThreadSafe.Instance;
```
**Pros**:
- Ensures thread safety.

**Cons**:
- Slight performance overhead due to the `lock`.

---

### 4. Double-Checked Locking
Reduces the performance overhead by ensuring `lock` is only used when the instance is null.

```csharp
public sealed class SingletonDoubleChecked
{
    private static SingletonDoubleChecked _instance;
    private static readonly object _lock = new object();

    private SingletonDoubleChecked() { }

    public static SingletonDoubleChecked Instance
    {
        get
        {
            if (_instance == null)
            {
                lock (_lock)
                {
                    if (_instance == null)
                    {
                        _instance = new SingletonDoubleChecked();
                    }
                }
            }
            return _instance;
        }
    }
}

// Usage
var singleton = SingletonDoubleChecked.Instance;
```
**Pros**:
- Efficient; locks only during the first initialization.

**Cons**:
- Slightly more complex to implement.

---

### 5. Lazy Initialization with `Lazy<T>`
Simplifies lazy initialization using .NET's `Lazy<T>` class.

```csharp
public sealed class SingletonLazyDotNet
{
    private static readonly Lazy<SingletonLazyDotNet> _instance = 
        new Lazy<SingletonLazyDotNet>(() => new SingletonLazyDotNet());

    private SingletonLazyDotNet() { }

    public static SingletonLazyDotNet Instance => _instance.Value;
}

// Usage
var singleton = SingletonLazyDotNet.Instance;
```
**Pros**:
- Thread-safe by default.
- Easy to implement.

**Cons**:
- Relies on .NET's `Lazy<T>`.

---

## Singleton in .NET Core Dependency Injection
In .NET Core, the Singleton pattern is often managed using the **Dependency Injection (DI) container**.

```csharp
// Register singleton service in Startup.cs
services.AddSingleton<IMyService, MyService>();

// Access the singleton instance via DI
public class MyController : ControllerBase
{
    private readonly IMyService _service;

    public MyController(IMyService service)
    {
        _service = service;
    }
}
```
**Pros**:
- Leverages built-in DI for singleton management.
- Simplifies the lifecycle management.

---

## Preventing Singleton Violations
1. **Reflection**: Prevent reflection attacks by throwing an exception in the private constructor.
   ```csharp
   private SingletonEager()
   {
       if (_instance != null)
       {
           throw new InvalidOperationException("Instance already created.");
       }
   }
   ```

2. **Serialization**: Implement `ISerializable` and override `readResolve`.
   ```csharp
   protected SingletonSerialized() { }
   protected SingletonSerialized(SerializationInfo info, StreamingContext context) { }
   private object readResolve() => Instance;
   ```

---

## Common Use Cases
1. **Database Connection**: Share a single DB connection across services.
2. **Configuration**: Centralized configuration management.
3. **Logging**: A single logger instance to avoid overhead.

---

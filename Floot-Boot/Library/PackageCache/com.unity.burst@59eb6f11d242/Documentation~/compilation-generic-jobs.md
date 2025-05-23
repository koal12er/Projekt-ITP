# Generic jobs

While Burst supports generics, it has limited support for generic jobs or function pointers. If you notice that a job runs at full speed in the Editor but not in a built Player, the problem might be related to generic jobs.

The following example defines a generic job:

```c#
// Direct generic job
[BurstCompile]
struct MyGenericJob<TData> : IJob where TData : struct { 
    public void Execute() { ... }
}
```

You can also nest generic jobs: 

```c#
// Nested generic job
public class MyGenericSystem<TData> where TData : struct {
    [BurstCompile]
    struct MyGenericJob  : IJob { 
        public void Execute() { ... }
    }

    public void Run()
    {
        var myJob = new MyGenericJob(); // implicitly MyGenericSystem<TData>.MyGenericJob
        myJob.Schedule();    
    }
}
```

Jobs that are Burst-compiled look like this:

```c#
// Direct Generic Job
var myJob = new MyGenericJob<int>();
myJob.Schedule();

// Nested Generic Job
var myJobSystem = new MyGenericSystem<float>();
myJobSystem.Run();
```

In both cases, in a Player build, the Burst compiler detects that it has to compile `MyGenericJob<int>` and `MyGenericJob<float>`. This is because the generic jobs (or the type surrounding it for the nested job) are used with fully resolved generic arguments (`int` and `float`).

However, if these jobs are used indirectly through a generic parameter, the Burst compiler can't detect the jobs it has to compile at Player build time:

```c#
public static void GenericJobSchedule<TData>() where TData: struct {
    // Generic argument: Generic Parameter TData
    // This Job won't be detected by the Burst Compiler at standalone-player build time.
    var job = new MyGenericJob<TData>();
    job.Schedule();
}

// The implicit MyGenericJob<int> will run at Editor time in full Burst speed
// but won't be detected at standalone-player build time.
GenericJobSchedule<int>();
```

The same restriction applies if you declare the job in the context of a generic parameter that comes from a type:

```c#
// Generic Parameter TData
public class SuperJobSystem<TData>
{
    // Generic argument: Generic Parameter TData
    // This Job won't be detected by the Burst Compiler at standalone-player build time.
    public MyGenericJob<TData> MyJob;
}
```

If you want to use generic jobs, you must use them directly with fully resolved generic arguments (for example, `int`, `MyOtherStruct`). You can't use them with a generic parameter indirection (for example, `MyGenericJob<TContext>`).

>[!IMPORTANT]
>Burst doesn't support scheduling generic Jobs through generic methods.

## Function pointers

Function pointers are restricted because you can't use a generic delegate through a function pointer with Burst:

```c#
public delegate void MyGenericDelegate<T>(ref TData data) where TData: struct;

var myGenericDelegate = new MyGenericDelegate<int>(MyIntDelegateImpl);
// Will fail to compile this function pointer.
var myGenericFunctionPointer = BurstCompiler.CompileFunctionPointer<MyGenericDelegate<int>>(myGenericDelegate);
```

This limitation is because of a limitation of the .NET runtime to interop with such delegates.

For more information, refer to [Function pointers](csharp-function-pointers.md).

## Additional resources

* [Burst compilation in Play mode](compilation-synchronous.md)
* [Job system](xref:um-job-system)
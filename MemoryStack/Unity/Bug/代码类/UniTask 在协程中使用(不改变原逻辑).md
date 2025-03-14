#UniTask  #协程 

在修改老项目时,需要在某处同步方法中添加异步网络请求,导致方法变为异步,进而形成连锁反应,很多方法都需要修改,直到遇到了协程中使用的问题

下面是在协程中不使用await把异步方法转为协程去返回结果

#### 通过回调参数（协程混合方案）

```cs
```csharp  //异步方法 -> InitRoles
IEnumerator LegacyCoroutine()
{
    RoleModel[] models = null;
    yield return InitRoles(playerCount).ToCoroutine(
        callback: result => models = result
    );
    
    // 需要手动检查 null
    if(models != null)
    {
        roleModels = models;
        // 这里使用数据
    }
}
```

- 使用协程等待异步操作完成
- 需要手动检查结果是否为 null
- 适合在只支持协程的旧代码中使用
#### 使用 UniTask 的 ContinueWith（高级用法）

```c's
private void Start()
{
    InitRoles(playerCount)
        .ContinueWith(result => 
        {
            roleModels = result;
            Debug.Log("数据就绪");
        })
        .Forget(); // 类似于 Fire and Forget
}
```

**注意事项**：
- .ContinueWith 相当于 "then"，用于处理异步操作完成后的回调
- .Forget() 表示不等待任务完成，立即返回
- 需要手动处理线程上下文
- 适用于不需要等待结果的场景
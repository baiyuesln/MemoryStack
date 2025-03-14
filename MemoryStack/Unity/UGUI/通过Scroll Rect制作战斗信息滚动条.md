#ugui  #ScrollRect  #对象池  #事件

[[事件管理器]] [[简单对象池]]

在挂机类战斗中,经常需要一个战斗信息展示面板,去展示我方和敌方的所有状态转换和伤害信息、战斗过程中或者结束后获得的奖励信息、战斗开始结束标识等.

![[Pasted image 20250313110507.png]]

- 实现方式我使用Scroll Rect配合Vertical Layout Group和Content Size Fitter组件,
- 通过在这个panel开始时注册战斗消息事件回调OnBattleMessage,在战斗中需要展示信息的节点触发这个事件.添加message Prefab到scroll list中. 实现消息滚动
- 优化方面使用内存池,限制最大prefab个数,如果大于最大个数,最上面的prefab被回收(false),需要时从新赋值并放到最下面.

代码实现:
```cs

/// <summary>
/// 战斗消息管理面板
/// </summary>
public class BattleMessageManager : MonoBehaviour
{
    [SerializeField] private ScrollRect scrollRect;
    [SerializeField] private RectTransform contentPanel;
    [SerializeField] private GameObject messagePrefab;
    [SerializeField] private int maxMessages = 50; //最大消息数量

    private Queue<GameObject> messagePool = new Queue<GameObject>();
    private bool autoScroll = true;

    private void Start()
    {
        // 注册战斗消息事件
        EventManager.Instance.Register(EventManager.BATTLE_MESSAGE, OnBattleMessage);
        MyArrayList<int> ints = new MyArrayList<int>();
        ints.AddLast(1);
        ints.AddLast(1);
        ints.AddLast(1);
        ints.RemoveFirst();
        ints.Get(1);
        ints.Set(1, 222);

    }

    /// <summary>
    /// 战斗消息事件回调
    /// </summary>
    private void OnBattleMessage(object[] args)
    {
        if (args.Length > 0 && args[0] is string message)
        {
            AddMessage(message);
        }
    }

    /// <summary>
    /// 添加消息
    /// </summary>
    public async UniTask AddMessage(string message)
    {
        // 从对象池获取或创建消息项
        GameObject messageObj = await GetMessageObject();
        messageObj.GetComponentInChildren<Text>().text = message;

        // 管理消息数量
        if (contentPanel.childCount > maxMessages)
        {
            ReturnToPool(contentPanel.GetChild(0).gameObject);
        }
        // 确保消息对象被正确设置为 contentPanel 的子物体
        messageObj.transform.SetParent(contentPanel, false);
        messageObj.transform.SetAsLastSibling();

        // 自动滚动到底部
        if (autoScroll)
        {
            Canvas.ForceUpdateCanvases();
            scrollRect.verticalNormalizedPosition = 0f;
        }
    }

    private async UniTask<GameObject> GetMessageObject()
    {
        GameObject messageObj;
        if (messagePool.Count > 0)
        {
            messageObj = messagePool.Dequeue();
            messageObj.SetActive(true);
        }
        else
        {
            messageObj = await ResManager.Instance.NewGetPrefabsAsync("BattleMessagePrefab", contentPanel);
        }
        return messageObj;
    }

    private void ReturnToPool(GameObject messageObj)
    {
        messageObj.SetActive(false);
        messageObj.transform.SetParent(null); // 将对象从UI层级中移除
        messagePool.Enqueue(messageObj);
    }

    private void OnDestroy()
    {
        EventManager.Instance.Unregister(EventManager.BATTLE_MESSAGE, OnBattleMessage);
    }
}

```

优化建议：

1. 性能方面 ：
   - 添加对象池预热机制，避免游戏开始时频繁创建对象 *
   - 使用对象池泛型类管理，提高复用性
   - 消息处理异步化，避免卡顿
   - 批量处理消息，减少每帧的操作次数
2. 内存方面 ：
   
   - 添加消息存活时间机制，自动清理旧消息 *
   - 使用消息队列处理突发大量消息 *
   - 考虑使用消息合并机制（例如相同类型的伤害信息）
3. UI体验方面 ：
   
   - 添加消息分类和过滤功能
   - 实现消息优先级机制
   - 添加消息动画效果（淡入淡出）*
   - 提供手动清理和暂停滚动的功能 *
1. 扩展性方面 ：
   
   - 支持富文本显示（不同颜色、字体等）*
   - 支持消息模板系统
   - 添加消息导出功能（用于调试）
5. 资源管理方面 ：
- 在场景切换时正确清理资源 *
- 提供消息持久化机制（如果需要）
- 考虑使用可配置的资源加载方式
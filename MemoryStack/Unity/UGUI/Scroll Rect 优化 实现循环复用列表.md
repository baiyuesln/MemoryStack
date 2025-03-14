#ugui  #ScrollRect  #对象池 #优化 #事件 

[[通过Scroll Rect制作战斗信息滚动条]]  

https://blog.csdn.net/linxinfa/article/details/115396546

### 核心机制 
- 只创建能填满可视区域（加上上下缓冲）的最少量列表项
- 通过移动这些列表项的位置和更新数据来实现无限滚动
- 使用对象池技术复用列表项，避免频繁创建销毁

### 使用方式
在Scroll Rect上挂载RecyclingListView,
在prefab上挂载继承RecyclingListViewItem的类
对于RecyclingListView的ItemCallback添加回调方法
在RecyclingListView中UpdateChild方法运行回调,刷新赋值给prefab,动态更新UI显示

### 代码展示

RecyclingListView:
```cs

using System;
using System.Threading.Tasks;
using Cysharp.Threading.Tasks;
using UnityEngine;
using UnityEngine.UI;

/// <summary>
/// 循环复用列表
/// </summary>
[RequireComponent(typeof(ScrollRect))]
public class RecyclingListView : MonoBehaviour
{
    [Tooltip("子节点物体")]
    public RecyclingListViewItem ChildObj;
    [Tooltip("行间隔")]
    public float RowPadding = 15f;
    [Tooltip("事先预留的最小列表高度")]
    public float PreAllocHeight = 0;

    public enum ScrollPosType
    {
        Top,
        Center,
        Bottom,
    }


    public float VerticalNormalizedPosition
    {
        get => scrollRect.verticalNormalizedPosition;
        set => scrollRect.verticalNormalizedPosition = value;
    }


    /// <summary>
    /// 列表行数
    /// </summary>
    protected int rowCount;

    /// <summary>
    /// 列表行数，赋值时，会执行列表重新计算
    /// </summary>
    public int RowCount
    {
        get => rowCount;
        set
        {
            if (rowCount != value)
            {
                rowCount = value;
                // 先禁用滚动变化
                ignoreScrollChange = true;
                // 更新高度
                UpdateContentHeight();
                // 重新启用滚动变化
                ignoreScrollChange = false;
                // 重新计算item
                ReorganiseContent(true);
            }
        }
    }

    /// <summary>
    /// item更新回调函数委托
    /// </summary>
    /// <param name="item">子节点对象</param>
    /// <param name="rowIndex">行数</param>
    public delegate void ItemDelegate(RecyclingListViewItem item, int rowIndex);

    /// <summary>
    /// item更新回调函数委托
    /// </summary>
    public ItemDelegate ItemCallback;

    protected ScrollRect scrollRect;
    /// <summary>
    /// 复用的item数组
    /// </summary>
    protected RecyclingListViewItem[] childItems;

    /// <summary>
    /// 循环列表中，第一个item的索引，最开始每个item都有一个原始索引，最顶部的item的原始索引就是childBufferStart
    /// 由于列表是循环复用的，所以往下滑动时，childBufferStart会从0开始到n，然后又从0开始，以此往复
    /// 如果是往上滑动，则是从0到-n，再从0开始，以此往复
    /// </summary>
    protected int childBufferStart = 0;
    /// <summary>
    /// 列表中最顶部的item的真实数据索引，比如有一百条数据，复用10个item，当前最顶部是第60条数据，那么sourceDataRowStart就是59（注意索引从0开始）
    /// </summary>
    protected int sourceDataRowStart;

    protected bool ignoreScrollChange = false;
    protected float previousBuildHeight = 0;
    protected const int rowsAboveBelow = 1;

    protected virtual void Awake()
    {
        scrollRect = GetComponent<ScrollRect>();
        ChildObj.gameObject.SetActive(false);
    }


    protected virtual void OnEnable()
    {
        scrollRect.onValueChanged.AddListener(OnScrollChanged);
        ignoreScrollChange = false;
    }

    protected virtual void OnDisable()
    {
        scrollRect.onValueChanged.RemoveListener(OnScrollChanged);
    }


    /// <summary>
    /// 供外部调用，强制刷新整个列表，比如数据变化了，刷新一下列表
    /// </summary>
    public virtual void Refresh()
    {
        ReorganiseContent(true);
    }

    /// <summary>
    /// 供外部调用，强制刷新整个列表的局部item
    /// </summary>
    /// <param name="rowStart">开始行</param>
    /// <param name="count">数量</param>
    public virtual void Refresh(int rowStart, int count)
    {
        int sourceDataLimit = sourceDataRowStart + childItems.Length;
        for (int i = 0; i < count; ++i)
        {
            int row = rowStart + i;
            if (row < sourceDataRowStart || row >= sourceDataLimit)
                continue;

            int bufIdx = WrapChildIndex(childBufferStart + row - sourceDataRowStart);
            if (childItems[bufIdx] != null)
            {
                UpdateChild(childItems[bufIdx], row);
            }
        }
    }

    /// <summary>
    /// 供外部调用，强制刷新整个列表的某一个item
    /// </summary>
    public virtual void Refresh(RecyclingListViewItem item)
    {

        for (int i = 0; i < childItems.Length; ++i)
        {
            int idx = WrapChildIndex(childBufferStart + i);
            if (childItems[idx] != null && childItems[idx] == item)
            {
                UpdateChild(childItems[i], sourceDataRowStart + i);
                break;
            }
        }
    }

    /// <summary>
    /// 清空列表
    /// </summary>
    public virtual void Clear()
    {
        RowCount = 0;
    }


    /// <summary>
    /// 供外部调用，强制滚动列表，使某一行显示在列表中
    /// </summary>
    /// <param name="row">行号</param>
    /// <param name="posType">目标行显示在列表的位置：顶部，中心，底部</param>
    public virtual void ScrollToRow(int row, ScrollPosType posType)
    {
        scrollRect.verticalNormalizedPosition = GetRowScrollPosition(row, posType);
    }

    /// <summary>
    /// 获得归一化的滚动位置，该位置将给定的行在视图中居中
    /// </summary>
    /// <param name="row">行号</param>
    /// <returns></returns>
    public float GetRowScrollPosition(int row, ScrollPosType posType)
    {
        // 视图高
        float vpHeight = ViewportHeight();
        float rowHeight = RowHeight();
        // 将目标行滚动到列表目标位置时，列表顶部的位置
        float vpTop = 0;
        switch (posType)
        {
            case ScrollPosType.Top:
                {
                    vpTop = row * rowHeight;
                }
                break;
            case ScrollPosType.Center:
                {
                    // 目标行的中心位置与列表顶部的距离
                    float rowCentre = (row + 0.5f) * rowHeight;
                    // 视口中心位置
                    float halfVpHeight = vpHeight * 0.5f;

                    vpTop = Mathf.Max(0, rowCentre - halfVpHeight);
                }
                break;
            case ScrollPosType.Bottom:
                {
                    vpTop = (row + 1) * rowHeight - vpHeight;
                }
                break;
        }


        // 滚动后，列表底部的位置
        float vpBottom = vpTop + vpHeight;
        // 列表内容总高度
        float contentHeight = scrollRect.content.sizeDelta.y;
        // 如果滚动后，列表底部的位置已经超过了列表总高度，则调整列表顶部的位置
        if (vpBottom > contentHeight)
            vpTop = Mathf.Max(0, vpTop - (vpBottom - contentHeight));

        // 反插值，计算两个值之间的Lerp参数。也就是value在from和to之间的比例值
        return Mathf.InverseLerp(contentHeight - vpHeight, 0, vpTop);
    }

    /// <summary>
    /// 根据行号获取复用的item对象
    /// </summary>
    /// <param name="row">行号</param>
    protected RecyclingListViewItem GetRowItem(int row)
    {
        if (childItems != null &&
            row >= sourceDataRowStart && row < sourceDataRowStart + childItems.Length &&
            row < rowCount)
        {
            // 注意这里要根据行号计算复用的item原始索引
            return childItems[WrapChildIndex(childBufferStart + row - sourceDataRowStart)];
        }

        return null;
    }

    /// <summary>
    /// 用于检查和确保有足够的列表项可用
    /// </summary>
    /// <returns>返回值表示是否需要重建列表项</returns>
    protected virtual async UniTask<bool> CheckChildItems()
    {
        // 列表视口高度
        float vpHeight = ViewportHeight();
        //计算构建高度，取视口高度和预设最小高度的较大值
        float buildHeight = Mathf.Max(vpHeight, PreAllocHeight);
        //判断是否需要重建列表,- 列表项数组为空- 新的构建高度大于之前的构建高度
        bool rebuild = childItems == null || buildHeight > previousBuildHeight;
        if (rebuild)
        {
            //计算需要的列表项数量：
            int childCount = Mathf.RoundToInt(0.5f + buildHeight / RowHeight());
            //额外添加上下缓冲区的数量（rowsAboveBelow * 2）
            childCount += rowsAboveBelow * 2;
            //如果数组不存在，创建新数组
            if (childItems == null)
                childItems = new RecyclingListViewItem[childCount];
            else if (childCount > childItems.Length) //如果需要更多列表项，扩展数组大小
                Array.Resize(ref childItems, childCount);

            // 创建item
            for (int i = 0; i < childItems.Length; ++i)
            {
                if (childItems[i] == null)//如果列表项不存在，异步加载预制体
                {
                    var item = await ResManager.Instance.NewGetPrefabsAsync("LevelSelectCell", null);
                    childItems[i] = item.GetComponent<RecyclingListViewItem>();
                }
                //设置父物体为 content
                childItems[i].RectTransform.SetParent(scrollRect.content, false);
                //初始状态设为不可见
                childItems[i].gameObject.SetActive(false);
            }
            //更新之前的构建高度
            previousBuildHeight = buildHeight;
        }
        //返回是否进行了重建
        return rebuild;
    }


    /// <summary>
    /// 列表滚动时，会回调此函数
    /// </summary>
    /// <param name="normalisedPos">归一化的位置</param>
    protected virtual void OnScrollChanged(Vector2 normalisedPos)
    {
        if (!ignoreScrollChange)
        {
            ReorganiseContent(false);
        }
    }

    /// <summary>
    /// 重新计算列表内容,处理列表项的复用和更新
    /// </summary>
    /// <param name="clearContents">是否要清空列表重新计算</param>
    protected virtual async Task ReorganiseContent(bool clearContents)
    {
        //// 如果需要清空列表
        if (clearContents)
        {
            // // 停止当前的滚动动画
            scrollRect.StopMovement();
            // 将列表滚动到顶部
            scrollRect.verticalNormalizedPosition = 1;
        }
        // 检查并确保有足够的列表项可用
        bool childrenChanged = await CheckChildItems();
        // 判断是否需要更新所有列表项
        bool populateAll = childrenChanged || clearContents;

        //// 获取content当前的垂直位置
        float ymin = scrollRect.content.localPosition.y;

        // 根据当前滚动位置计算第一个可见项的索引
        int firstVisibleIndex = (int)(ymin / RowHeight());

        // 计算新的起始行，考虑上方缓冲区
        int newRowStart = firstVisibleIndex - rowsAboveBelow;

        // 计算滚动变化量（新起始行与当前起始行的差值）
        int diff = newRowStart - sourceDataRowStart;
        // 如果需要更新所有项，或者滚动变化太大，需要重新计算所有列表项
        if (populateAll || Mathf.Abs(diff) >= childItems.Length)
        {
            // 更新起始行
            sourceDataRowStart = newRowStart;
            // 重置缓冲区起始索引
            childBufferStart = 0;
            int rowIdx = newRowStart;
            // 更新所有可见列表项
            foreach (var item in childItems)
            {
                UpdateChild(item, rowIdx++);
            }

        }
        // 如果有滚动变化但变化量较小
        else if (diff != 0)
        {
            // 计算新的缓冲区起始索引
            int newBufferStart = (childBufferStart + diff) % childItems.Length;
            // 向上滚动
            if (diff < 0)
            {
                // 更新新显示的项
                for (int i = 1; i <= -diff; ++i)
                {
                    // 计算要复用的列表项索引
                    int wrapIndex = WrapChildIndex(childBufferStart - i);
                    // 计算数据索引
                    int rowIdx = sourceDataRowStart - i;
                    // 更新列表项
                    UpdateChild(childItems[wrapIndex], rowIdx);
                }
            }
            else
            {
                // 计算当前最后一项的缓冲区索引和行索引
                int prevLastBufIdx = childBufferStart + childItems.Length - 1;
                int prevLastRowIdx = sourceDataRowStart + childItems.Length - 1;
                for (int i = 1; i <= diff; ++i)
                {
                    // 计算要复用的列表项索引
                    int wrapIndex = WrapChildIndex(prevLastBufIdx + i);
                    // 计算数据索引
                    int rowIdx = prevLastRowIdx + i;
                    UpdateChild(childItems[wrapIndex], rowIdx);
                }
            }
            // 更新起始行和缓冲区起始索引
            sourceDataRowStart = newRowStart;

            childBufferStart = newBufferStart;
        }
    }

    /// <summary>
    /// 计算循环列表中实际的子项索引
    /// </summary>
    private int WrapChildIndex(int idx)
    {
        //当索引为负数时，不断加上数组长度直到变成正数。
        while (idx < 0)
            idx += childItems.Length;
        //使用取模运算确保返回的索引在数组范围内
        return idx % childItems.Length;
    }

    /// <summary>
    /// 获取一行的高度，注意要加上RowPadding
    /// </summary>
    private float RowHeight()
    {
        return RowPadding + ChildObj.RectTransform.rect.height;
    }

    /// <summary>
    /// 获取列表视口的高度
    /// </summary>
    private float ViewportHeight()
    {
        return scrollRect.viewport.rect.height;
    }

    /// <summary>
    /// 更新item的位置和数据
    /// </summary>
    /// <param name="child">列表项对象</param>
    /// <param name="rowIdx">该项在整个数据中的索引</param>
    protected virtual void UpdateChild(RecyclingListViewItem child, int rowIdx)
    {
        //- 检查索引是否越界  如果索引无效，则隐藏该列表项
        if (rowIdx < 0 || rowIdx >= rowCount)
        {
            child.gameObject.SetActive(false);
        }
        else
        {
            //- 检查是否设置了数据更新回调 - 如果没有设置回调，输出错误日志并返回
            if (ItemCallback == null)
            {
                Debug.Log("RecyclingListView is missing an ItemCallback, cannot function", this);
                return;
            }

            // 获取列表项预制体的矩形信息和轴心点信息
            var childRect = ChildObj.RectTransform.rect;
            Vector2 pivot = ChildObj.RectTransform.pivot;
            // 计算当前项在整个列表中的垂直位置
            float ytoppos = RowHeight() * rowIdx;
            //- 根据轴心点计算最终的位置坐标- y轴位置需要考虑轴心点的偏移- x轴位置通常是固定的，但也要考虑轴心点
            float ypos = ytoppos + (1f - pivot.y) * childRect.height;
            float xpos = 0 + pivot.x * childRect.width;
            // 设置列表项的最终位置,y值取负是因为Unity UI中向下为负方向
            child.RectTransform.anchoredPosition = new Vector2(xpos, -ypos);
            //通知列表项它的新位置和索引
            child.NotifyCurrentAssignment(this, rowIdx);

            // 调用外部传入的回调函数,更新数据
            ItemCallback(child, rowIdx);
            //确保列表项在更新完成后可见
            child.gameObject.SetActive(true);
        }
    }

    /// <summary>
    /// 更新content的高度
    /// </summary>
    protected virtual void UpdateContentHeight()
    {
        // 列表高度  计算所有item的高度   高度加间隔*个数
        float height = ChildObj.RectTransform.rect.height * rowCount + (rowCount - 1) * RowPadding;
        // 得出宽度
        var sz = scrollRect.content.sizeDelta;
        // 更新content的高度
        scrollRect.content.sizeDelta = new Vector2(sz.x, height);
    }

    protected virtual void DisableAllChildren()
    {
        if (childItems != null)
        {
            for (int i = 0; i < childItems.Length; ++i)
            {
                childItems[i].gameObject.SetActive(false);
            }
        }
    }
}

```

RecyclingListViewItem:
```cs
using UnityEngine;

/// <summary>
/// 列表item，你自己写的列表item需要继承该类
/// </summary>
[RequireComponent(typeof(RectTransform))]
public class RecyclingListViewItem : MonoBehaviour
{

    private RecyclingListView parentList;

    /// <summary>
    /// 循环列表
    /// </summary>
    public RecyclingListView ParentList
    {
        get => parentList;
    }

    private int currentRow;
    /// <summary>
    /// 行号
    /// </summary>
    public int CurrentRow
    {
        get => currentRow;
    }

    private RectTransform rectTransform;
    public RectTransform RectTransform
    {
        get
        {
            if (rectTransform == null)
                rectTransform = GetComponent<RectTransform>();
            return rectTransform;
        }
    }

    private void Awake()
    {
        rectTransform = GetComponent<RectTransform>();
    }

    /// <summary>
    /// item更新事件响应函数
    /// </summary>
    public virtual void NotifyCurrentAssignment(RecyclingListView v, int row)
    {
        parentList = v;
        currentRow = row;
    }
}


```

LevelSelectWin:
```cs
public class Win
{
	private List<DungeonData> data = new List<DungeonData>();
    public RecyclingListView scrollList;
	void Start()
	{
		scrollList.ItemCallback = PopulateItem;
		for (int i = 0; i < datas.Count; ++i)
        {
            data.Add(new DungeonData(datas[i], i));
        }

        // 设置数据，此时列表会执行更新
        scrollList.RowCount = data.Count;
        .....
	}
	.....
}
```

cell:
```cs
public class cell:RecyclingListViewItem
{
	private DungeonData dungeonData;
    public DungeonData DungeonData
    {
        get { return dungeonData; }
        set
        {
            dungeonData = value;
            RefreshUI();
        }
    }
    private void RefreshUI()
    {
	    .....
    }
}
public struct DungeonData
{
    public Dungeon data;
    public int Row;

    public DungeonData(Dungeon data, int row)
    {
        this.data = data;
        Row = row;
    }
}
```
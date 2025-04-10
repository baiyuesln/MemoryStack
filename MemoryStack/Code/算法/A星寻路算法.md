#寻路  #算法

A*寻路是一种启发式搜索算法,通过评估每个潜在节点的综合代价,计算出最优路径.

![[Pasted image 20250207102226.png]]

### 原理

1. **节点评估模型**
    
    - **代价值（Cost）**：每个节点用三个关键值评估：
        - **G值**：从**起点**到当前节点的实际移动成本（如距离、地形影响）。
        - **H值**（启发值）：当前节点到**终点**的**预估成本**（如直线距离）。
        - **F值**：总评估值，即 `F = G + H`。
    - **目标导向**：优先扩展F值最小的节点，平衡实际代价与目标接近度。

2. **数据结构**
    
    - **开放列表（Open List）**：存储待探索的节点（优先队列，按F值排序）。
    - **关闭列表（Closed List）**：记录已处理的节点，避免重复计算。

3. 伪代码示例
```cs
public class AStar
{
    // 表示节点的基础类
    public class Node
    {
        public Vector2Int Position { get; set; }  // 节点坐标
        public Node Parent { get; set; }          // 父节点
        public int G { get; set; }                // 从起点到当前节点的实际代价
        public int H { get; set; }                // 从当前节点到终点的估计代价
        public int F => G + H;                    // 总代价 F = G + H
        
        public Node(Vector2Int pos)
        {
            Position = pos;
        }
    }

    private List<Node> openList;   // 开放列表
    private List<Node> closeList;  // 关闭列表
    private Node[,] nodeMap;       // 节点地图
    private Vector2Int mapSize;    // 地图大小

    // 主要寻路方法
    public List<Vector2Int> FindPath(Vector2Int start, Vector2Int end)
    {
        // 初始化列表
        openList = new List<Node>();
        closeList = new List<Node>();
        
        // 创建起点和终点节点
        Node startNode = new Node(start);
        Node endNode = new Node(end);
        
        // 将起点加入开放列表
        openList.Add(startNode);
        
        while (openList.Count > 0)
        {
            // 获取开放列表中F值最小的节点
            Node currentNode = GetMinFNode();
            
            // 如果当前节点是终点，返回路径
            if (currentNode.Position == end)
            {
                return GeneratePath(currentNode);
            }
            
            // 将当前节点从开放列表移到关闭列表
            openList.Remove(currentNode);
            closeList.Add(currentNode);
            
            // 检查相邻节点
            foreach (Node neighbor in GetNeighbors(currentNode))
            {
                // 如果邻居节点在关闭列表中，跳过
                if (closeList.Any(n => n.Position == neighbor.Position))
                    continue;
                
                // 计算新的G值
                int newG = currentNode.G + GetDistance(currentNode, neighbor);
                
                // 如果邻居节点不在开放列表中
                if (!openList.Any(n => n.Position == neighbor.Position))
                {
                    // 设置G、H值并添加到开放列表
                    neighbor.G = newG;
                    neighbor.H = GetDistance(neighbor, endNode);
                    neighbor.Parent = currentNode;
                    openList.Add(neighbor);
                }
                // 如果邻居节点已在开放列表中
                else
                {
                    Node existingNode = openList.First(n => n.Position == neighbor.Position);
                    //todo  如果它已经在 open list 中，检查这条路径 ( 即经由当前方格到达它那里 ) 是否更好，用 G 值作参考。更小的 G 值表示这是更好的路径。

                    // 如果新路径更好（G值更小），更新路径
                    if (newG < existingNode.G)
                    {
                        existingNode.G = newG;
                        existingNode.Parent = currentNode;
                    }
                }
            }
        }
        
        // 没找到路径，返回空列表
        return new List<Vector2Int>();
    }
    
    // 获取相邻节点
    private List<Node> GetNeighbors(Node node)
    {
        List<Node> neighbors = new List<Node>();
        
        // 8个方向的偏移量
        int[] dx = { -1, 1, 0, 0, -1, -1, 1, 1 };
        int[] dy = { 0, 0, -1, 1, -1, 1, -1, 1 };
        
        for (int i = 0; i < 8; i++)
        {
            Vector2Int newPos = new Vector2Int(
                node.Position.x + dx[i],
                node.Position.y + dy[i]
            );
            
            // 检查是否在地图范围内且可通行
            if (IsValidPosition(newPos))
            {
                neighbors.Add(new Node(newPos));
            }
        }
        
        return neighbors;
    }
    
    // 获取两个节点间的距离（曼哈顿距离）
    private int GetDistance(Node a, Node b)
    {
        int dx = Mathf.Abs(a.Position.x - b.Position.x);
        int dy = Mathf.Abs(a.Position.y - b.Position.y);
        
        // 使用对角线距离
        return 10 * (dx + dy) + (14 - 2 * 10) * Mathf.Min(dx, dy);
    }
    
    // 获取开放列表中F值最小的节点
    private Node GetMinFNode()
    {
        Node minNode = openList[0];
        foreach (Node node in openList)
        {
            if (node.F < minNode.F)
                minNode = node;
        }
        return minNode;
    }
    
    // 检查位置是否有效
    private bool IsValidPosition(Vector2Int pos)
    {
        // 检查是否在地图范围内
        if (pos.x < 0 || pos.x >= mapSize.x || 
            pos.y < 0 || pos.y >= mapSize.y)
            return false;
            
        // 检查是否可通行（这里需要根据你的地图数据来判断）
        return !IsObstacle(pos);
    }
    
    // 判断是否是障碍物（需要根据实际地图数据实现）
    private bool IsObstacle(Vector2Int pos)
    {
        // 示例实现，需要根据实际地图数据修改
        return false;
    }
    
    // 生成最终路径
    private List<Vector2Int> GeneratePath(Node endNode)
    {
        List<Vector2Int> path = new List<Vector2Int>();
        Node current = endNode;
        
        while (current != null)
        {
            path.Add(current.Position);
            current = current.Parent;
        }
        
        path.Reverse();
        return path;
    }
}
```

最后返回的结果为从终点开始，每个方格沿着父节点移动直至起点，这就是你的路径。

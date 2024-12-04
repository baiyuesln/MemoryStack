
#ScriptableObject 

介绍:
https://docs.unity.cn/cn/2021.3/Manual/class-ScriptableObject.html

	ScriptableObject 是一个可独立于类实例来保存大量数据的数据容器。ScriptableObject 的一个主要用例是通过避免重复值来减少项目的内存使用量。如果项目有一个[预制件]在附加的 MonoBehaviour 脚本中存储不变的数据，这将非常有用。

	每次实例化预制件时，都会产生单独的数据副本。这种情况下可以不使用该方法并且不存储重复数据，而是使用 ScriptableObject 来存储数据，然后通过所有预制件的引用访问数据。这意味着内存中只有一个数据副本。

在这里我使用ScriptableObject来配置关卡数据

定义:
```
[CreateAssetMenu(fileName = "New Level", menuName = "Level/LevelData")]

public class SpinTriLevelData : ScriptableObject

{

public int levelIndex;

public int targetScore;

public Vector2[] fixedBlock;

public Vector2[] fixedStar;

public Vector2[] fixedTriangle;

public Vector3[] fixedTriangleRotate;

public Vector2[] flexibleTriangle;

public Vector3[] flexibleTriangleRotate;

public Vector2 ball;

}
```

创建
![[Pasted image 20241204162312.png]]
编辑
![[Pasted image 20241204162355.png]]
使用
```
public SpinTriLevelData[] levels; // 存储所有关卡数据

//加载指定关卡
public void LoadLevel(int levelIndex)

{

if (levelIndex >= 0 && levelIndex < levels.Length)

{

SpinTriLevelData currentLevel = levels[levelIndex];

GenerateLevel(currentLevel);

}

}

//根据关卡数据生成关卡
void GenerateLevel(SpinTriLevelData levelData)

{

currLevelIndex = levelData.levelIndex;

targetScore = levelData.targetScore;

// 生成固定的方块

if (levelData.fixedBlock != null)

{

foreach (Vector2 position in levelData.fixedBlock)

{

Instantiate(blockPrefab, position, Quaternion.identity).transform.SetParent(transform);

}

}
......
}
```
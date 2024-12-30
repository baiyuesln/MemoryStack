#替换 #sprite #Editor

功能:预制体内sprite的统一替换,可以附加指定格式的修改

思路：

- 创建一个Unity编辑器工具窗口

- 允许用户选择要替换的原始Sprite和新的Sprite

- 在所有预制体中查找并替换指定的Sprite

- 提供操作反馈和错误处理

- 包含了一些被注释掉的特殊处理代码，可能用于特定情况下的UI调整

代码
```cs
using UnityEngine;
using UnityEditor;  // 引入Unity编辑器相关功能
using System.Drawing;
using UnityEngine.UI;

// Unity编辑器扩展窗口，用于替换预制体中的图片
public class TextureReplacer : EditorWindow
{
    // 存储需要被替换的原始Sprite
    private Sprite oldSprite;
    // 存储用于替换的新Sprite
    private Sprite newSprite;

    // 在Unity菜单栏添加工具选项
    [MenuItem("Tools/Texture Replacer")]
    static void Init()
    {
        // 创建编辑器窗口实例
        TextureReplacer window = (TextureReplacer)EditorWindow.GetWindow(typeof(TextureReplacer));
        // 显示窗口
        window.Show();
    }

    // 绘制编辑器窗口的GUI界面
    void OnGUI()
    {
        // 创建两个对象字段用于选择新旧Sprite
        oldSprite = (Sprite)EditorGUILayout.ObjectField("旧图片", oldSprite, typeof(Sprite), false);
        newSprite = (Sprite)EditorGUILayout.ObjectField("新图片", newSprite, typeof(Sprite), false);

        // 添加替换按钮
        if (GUILayout.Button("替换所有预制体中的图片"))
        {
            ReplaceInPrefabs();
        }
    }

    // 执行预制体中图片替换的主要方法
    void ReplaceInPrefabs()
    {
        // 检查是否已选择新旧图片
        if (oldSprite == null || newSprite == null)
        {
            EditorUtility.DisplayDialog("错误", "请先选择新旧图片", "确定");
            return;
        }

        // 记录替换次数
        int replacedCount = 0;
        // 获取项目中所有预制体资源
        string[] allPrefabs = AssetDatabase.FindAssets("t:Prefab");
        
        // 遍历所有预制体
        foreach (string prefabGUID in allPrefabs)
        {
            // 将GUID转换为资源路径
            string assetPath = AssetDatabase.GUIDToAssetPath(prefabGUID);
            // 加载预制体内容
            GameObject prefabContents = PrefabUtility.LoadPrefabContents(assetPath);
            
            // 获取预制体中所有Image组件（包括未激活的）
            UnityEngine.UI.Image[] renderers = prefabContents.GetComponentsInChildren<UnityEngine.UI.Image>(true);
            // 标记是否修改了预制体
            bool modified = false;
            
            // 遍历所有Image组件
            foreach (UnityEngine.UI.Image renderer in renderers)
            {
                // 检查Image组件的sprite是否匹配要替换的sprite
                if (renderer.sprite == oldSprite)
                {
                    // 替换为新的sprite
                    renderer.sprite = newSprite;
                    
                    // 注释掉的代码包含了一些特定情况下的额外处理
                    // 返回
                    // RectTransform rectTransform = renderer.GetComponent<RectTransform>();
                    // if (rectTransform != null)
                    // {
                    //     // 设置新的尺寸
                    //     Vector2 newSize = new Vector2(110, newSprite.rect.height);
                    //     rectTransform.sizeDelta = newSize;
                    // }
                    // red按钮
                    // Text textComponent = renderer.GetComponentInChildren<Text>();
                    // if (textComponent != null)
                    // {
                    //     Debug.Log($"找到Text组件: {textComponent.gameObject.name}");
                    //     RectTransform rectTransform = textComponent.GetComponent<RectTransform>();
                    //     if (rectTransform != null)
                    //     {
                    //         // 保持原有的左右偏移，只修改上下偏移
                    //         Vector2 currentOffsetMin = rectTransform.offsetMin;
                    //         Vector2 currentOffsetMax = rectTransform.offsetMax;
                            
                    //         // 设置上下偏移为0
                    //         rectTransform.offsetMin = new Vector2(currentOffsetMin.x, 0); // bottom
                    //         rectTransform.offsetMax = new Vector2(currentOffsetMax.x, 0); 
                    //     }
                    // }
                    // else
                    // {
                    //     Debug.Log($"在物体 {renderer.gameObject.name} 下没有找到Text组件");
                    // }
                    // 比如调整RectTransform尺寸和Text组件位置等
                    
                    modified = true;
                    replacedCount++;
                }
            }

            // 如果预制体被修改，保存更改
            if (modified)
            {
                PrefabUtility.SaveAsPrefabAsset(prefabContents, assetPath);
            }
            
            // 卸载预制体内容以释放内存
            PrefabUtility.UnloadPrefabContents(prefabContents);
        }
        
        // 刷新资源数据库
        AssetDatabase.Refresh();
        // 显示替换完成的提示对话框
        EditorUtility.DisplayDialog("完成", $"已完成替换，共替换了 {replacedCount} 处图片", "确定");
    }
}
```
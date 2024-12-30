
#Text  #Editor 

由于刚接手的一个项目中UI标题格式参差不齐,各个预制体中标题的命名和位置五花八门,所以写个小工具,使在组件通过点击添加一个脚本,自动运行脚本内容,添加Text,并自动填写统一的参数.

	(通过拓展可以做到一键修改,但是各个预制体实在太过混乱,无法统一筛选出标题.)

代码:
```cs
using UnityEngine;
using UnityEngine.UI;
using UnityEditor;

#if UNITY_EDITOR
[ExecuteInEditMode]
#endif
public class TextAutoConfig : MonoBehaviour
{
    private void ConfigureText()
    {
        // 获取或添加 Text 组件
        Text text = GetComponent<Text>();
        if (text == null)
        {
            text = gameObject.AddComponent<Text>();
        }

        // 将十六进制颜色 #383838 转换为 Color 并设置
        Color textColor;
        ColorUtility.TryParseHtmlString("#383838", out textColor);
        text.color = textColor;

        text.horizontalOverflow = HorizontalWrapMode.Overflow;
        text.verticalOverflow = VerticalWrapMode.Overflow;
        
        #if UNITY_EDITOR
        // 从项目中加载自定义字体
        Font customFont = AssetDatabase.LoadAssetAtPath<Font>("Assets/Framework/AssetsPackage/Fonts/方正行楷_GBK.ttf");
        if (customFont != null)
        {
            text.font = customFont;
        }
        else
        {
            // 如果找不到自定义字体，使用系统默认字体
            text.font = Resources.GetBuiltinResource<Font>("Arial.ttf");
        }
        #endif

        // 设置字体大小为55
        text.fontSize = 55;
        // 设置文本水平和垂直居中对齐
        text.alignment = TextAnchor.MiddleCenter;

        // 获取并禁用 Outline 组件（如果存在）
        Outline outline = GetComponent<Outline>();
        if (outline != null)
        {
            outline.enabled = false;
        }

        // 获取 RectTransform 组件
        RectTransform rectTransform = GetComponent<RectTransform>();
        if (rectTransform != null)
        {
            // 设置锚点为上中位置 (0.5, 1)
            rectTransform.anchorMin = new Vector2(0.5f, 1f);
            rectTransform.anchorMax = new Vector2(0.5f, 1f);
            
            // 设置轴心点为上中位置 (0.5, 1)
            rectTransform.pivot = new Vector2(0.5f, 1f);
            
            // 设置相对锚点的位置，Y轴位置为-62
            rectTransform.anchoredPosition = new Vector2(0f, -35f);
            
            // 设置矩形变换的尺寸
            rectTransform.sizeDelta = new Vector2(200f, 50f);
        }

        // 配置完成后，移除这个脚本组件
        #if UNITY_EDITOR
        // 使用 DestroyImmediate 在编辑器模式下立即销毁
        DestroyImmediate(this);
        #else
        // 在运行时使用 Destroy
        Destroy(this);
        #endif
    }

    private void OnEnable()
    {
        // 当脚本启用时执行配置
        ConfigureText();
    }
}
```
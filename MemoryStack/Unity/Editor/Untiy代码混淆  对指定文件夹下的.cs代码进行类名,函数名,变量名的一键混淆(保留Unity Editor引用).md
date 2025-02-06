#混淆 #Editor 

界面:

![[Pasted image 20250103172911.png]]

代码:

```cs
using UnityEngine;
using UnityEditor;
using System.IO;
using System.Text.RegularExpressions;
using System.Collections.Generic;
using System.Linq;
using UnityEditor.SceneManagement;
using UnityEngine.SceneManagement;
using System.Text;

/// <summary>
/// 数据混淆工具 
/// 传入一个文件夹，对文件夹下的所有脚本进行混淆 
/// 包括类名、方法名、变量名,并且保留Unity Editor对脚本的引用   
/// </summary>
public class CodeCustomObfuscator : EditorWindow
{
    // 目标文件夹
    private DefaultAsset targetFolder;
    string mappingPath;
    // 名称映射
    private Dictionary<string, string> nameMapping = new Dictionary<string, string>();
    // 是否修改类名
    private bool modifyClassName = true;
    // 是否修改方法名
    private bool modifyMethodNames = true;
    // 是否修改变量名
    private bool modifyFieldNames = true;

    // C#关键字列表(需过滤)
    private static readonly HashSet<string> CSharpKeywords = new HashSet<string>
    {
        "true", "false", "null", "new", "this", "base",
        "void", "int", "string", "bool", "float", "double",
        "public", "private", "protected", "internal", "static",
        "Init", "Awake", "Start", "Update", "LateUpdate", "FixedUpdate",
        "OnEnable", "OnDisable", "OnDestroy", "OnGUI", "OnValidate",
        "Reset", "OnApplicationQuit", "OnApplicationFocus", "OnApplicationPause",
        "OnApplicationQuit", "OnApplicationFocus", "OnApplicationPause",
        "GetComponent", "GetComponentInChildren", "GetComponentInParent",
        "GetComponents", "GetComponentsInChildren", "GetComponentsInParent",
        "GetComponentsInChildren", "GetComponentsInParent", "GetComponentsInChildren",
        "GetComponentsInParent", "GetComponentsInChildren", "GetComponentsInParent",
        "Instance","instance",
        "Get","get",
        "Set","set",
        "Add","add",
        "Remove","remove",
        "if","break",
        "else","continue","Query","query","Download","Report",
        "Find","find",
        "FindObjectOfType","findobjectoftype",
        "FindObjectsOfType","findobjects",
        "Sprite","sprite",
        "Texture2D","texture2d",
        "Texture","texture",
        "Material","material",
        "Shader","shader","Rect","rect",
        "Vector2","vector2","Vector3","vector3","Vector4","vector4",
        "Quaternion","quaternion","Color","color",
        "Transform","transform","GameObject","gameobject",
        "Object","object","Component","component",
        "AssetBundle","assetbundle","data","package","handle","operation","PlayMode","Progress","progress",
        "StartCoroutine","startcoroutine","StopCoroutine","stopcoroutine","go",
        "Start","start","Stop","stop","Pause","pause","Resume","resume",
        "OnClick","onclick","OnClick","onclick","OnClick","onclick","OnClick","onclick",
        "OnClick","onclick","OnClick","onclick","OnClick","onclick","OnClick","onclick","RemoteServices","assets",
        "url","downloader","ResManager","image","Image","texture","Texture","sprite","Sprite",
        "return","index","request","value","key","id","name","type","path","url","data","package","handle","operation","PlayMode","Progress","progress",
        "result","button","Button","gameObject","GameObject","first","count","child","timer","token","userInfo",
        "Enter","enter","Exit","Close","speed","Speed","success","Success","CreateItem","childCount","Register",
        "version","linkInfo","SetText","jiangLi","JiangLi","qiangHuaLevel","ZhuangBei","QiangHua","YaoQianShu","JiQingPaoDian","vipValue",
        "GetAwaiter","ScrollRect","default","content","WaitForSeconds","rectTransform","AddListener","WaitForEndOfFrame","Refresh",
        "textColor","ProcessPurchase","SetValue","maxValue","WebRequest","minValue","PopupMessage","RegionalInfo",
        "WaterImage","WaterIndex","RegionalInfo","noIsClickButton","WaterIndex",
        "ShuLing","MessageUp","MessageDown","MenuPanel","PopupMessage","ChatRoomLIs_Yoo_amber_tManager",
        "BackButton","userGetCustomInfoCommandList","ChuangTianGuan","qiangHua","YiJieFuBen","fuBenId","GuaiWuData","GuaiWuDataEnc"
        ,"netTime","JiangLiEnc","GuaiWuShuXingEnc","GuaiWuShuXing","ZhuangBeiJiangliDataEnc","ZhuangBeiJiangliData","guaiWuType",
        "GuaiWuData","NowGuaiWuFuBenEnc","NowGuaiWuFuBen","GuaiWuFuBenEnc","GuaiWuFuBen","DiTuEnc","BaseBeiBaoHorItem"
        ,"UserEcSignSecretDto","UserEcSect","UserEcRecordAddNumDto","UserEcInfo","UserEcCultivation",
        "UserEcCount","UserEcCardBag","UserEcAssets","HomePageEcInfoDto","zhanDouLi","chatRoomLIstManager","ChatRoomLIstManager",
        "startShuXing","endShuXing","baiZhuang","BaoShiXiangQianPanelData","LocalEquipmentInfoEnc","usersCustomInfoDtoList","Broadcast",
        "goodInfo","LocalEquipmentInfo","zhuangBei","builder","NanosecondsPerTick","shangCheng","goodsId",
        "goodsType","scrollRect",
        "gameManager","shengHuaPanel","createPosToDengjiPos","createPosToZhuangBeiPos",
        // 可以根据需要添加更多关键字
    };

    // 添加关键词列表 用于混淆类名的关键词
    private static readonly List<string> keywords = new List<string>
    {
        "_Yoo_Asset_",
        "_Yoo_Data_",
        "_Yoo_UI_",
        "_Yoo_Game_",
        "_Yoo_Net_",
        "_Yoo_Audio_",
        "_Yoo_Effect_",
        "_Yoo_Scene_",
        "_Yoo_Config_",
        "_Yoo_Manager_",
        "_Yoo_Helper_",
        "_Yoo_Utils_",
        "_Yoo_Tool_",
        "_Yoo_Service_",
        "_Yoo_Handler_",
        "_Yoo_Controller_",
        "_Yoo_Model_",
        "_Yoo_Item_",
        "_Yoo_Element_",
        "_Yoo_Node_",
        "_Yoo_Component_",
        "_Yoo_Module_",
        "_Yoo_System_",
        "_Yoo_Core_",
        "_Yoo_Base_",
        "_Yoo_Main_",
        "_Yoo_Sub_",
        "_Yoo_Child_",
        "_Yoo_Manager_",
        "_Yoo_Handler_",
        "_Yoo_Controller_",
        //不能重复,格式是_Yoo_xxx_
        "_Yoo_hello_",
        "_Yoo_world_",
        "_Yoo_test_",
        "_Yoo_demo_",
        "_Yoo_example_",
        "_Yoo_sample_",
        "_Yoo_alpha_",
        "_Yoo_beta_",
        "_Yoo_gamma_",
        "_Yoo_delta_",
        "_Yoo_epsilon_",
        "_Yoo_zeta_",
        "_Yoo_theta_",
        "_Yoo_iota_",
        "_Yoo_kappa_",
        "_Yoo_lambda_",
        "_Yoo_sigma_",
        "_Yoo_omega_",
        "_Yoo_phoenix_",
        "_Yoo_dragon_",
        "_Yoo_tiger_",
        "_Yoo_lion_",
        "_Yoo_eagle_",
        "_Yoo_hawk_",
        "_Yoo_wolf_",
        "_Yoo_bear_",
        "_Yoo_snake_",
        "_Yoo_fox_",
        "_Yoo_deer_",
        "_Yoo_rabbit_",
        "_Yoo_mouse_",
        "_Yoo_cat_",
        "_Yoo_dog_",
        "_Yoo_bird_",
        "_Yoo_fish_",
        "_Yoo_whale_",
        "_Yoo_shark_",
        "_Yoo_dolphin_",
        "_Yoo_turtle_",
        "_Yoo_crab_",
        "_Yoo_star_",
        "_Yoo_moon_",
        "_Yoo_sun_",
        "_Yoo_sky_",
        "_Yoo_cloud_",
        "_Yoo_rain_",
        "_Yoo_snow_",
        "_Yoo_wind_",
        "_Yoo_storm_",
        "_Yoo_thunder_",
        "_Yoo_lightning_",
        "_Yoo_fire_",
        "_Yoo_water_",
        "_Yoo_earth_",
        "_Yoo_metal_",
        "_Yoo_wood_",
        "_Yoo_gold_",
        "_Yoo_silver_",
        "_Yoo_bronze_",
        "_Yoo_iron_",
        "_Yoo_steel_",
        "_Yoo_crystal_",
        "_Yoo_diamond_",
        "_Yoo_ruby_",
        "_Yoo_emerald_",
        "_Yoo_sapphire_",
        "_Yoo_pearl_",
        "_Yoo_jade_",
        "_Yoo_amber_",
        "_Yoo_opal_",
        "_Yoo_topaz_",
        "_Yoo_quartz_",
        "_Yoo_onyx_",
        "_Yoo_agate_",
        "_Yoo_coral_",
        "_Yoo_jasper_",
        "_Yoo_garnet_",
        "_Yoo_zircon_",
        "_Yoo_beryl_",
        "_Yoo_aqua_",
        "_Yoo_lapis_",
        "_Yoo_malachite_",
        "_Yoo_obsidian_",
        "_Yoo_peridot_",
        "_Yoo_tanzanite_",
        "_Yoo_turquoise_",
        "_Yoo_alexandrite_",
        "_Yoo_amethyst_",
        "_Yoo_citrine_",
        "_Yoo_moonstone_",
        "_Yoo_sunstone_",
        "_Yoo_bloodstone_",
        "_Yoo_chrysolite_",
        "_Yoo_fluorite_",
        "_Yoo_hematite_",
        "_Yoo_howlite_",
        "_Yoo_kunzite_",
        "_Yoo_larimar_",
        "_Yoo_nephrite_",
        "_Yoo_rhodonite_",
        "_Yoo_serpentine_",
        "_Yoo_sodalite_",
        "_Yoo_spinel_",
        "_Yoo_sugilite_",
        "_Yoo_tektite_",
        "_Yoo_titanite_",
        "_Yoo_unakite_",
        "_Yoo_variscite_",
        "_Yoo_vesuvianite_",
        "_Yoo_wavelite_",
        "_Yoo_xenotime_",
        "_Yoo_yttrium_",
        "_Yoo_zoisite_",



    };

    // 添加一个新的集合来存储需要保护的文件名
    private HashSet<string> protectedNames = new HashSet<string>();

    // 初始化
    [MenuItem("Tools/CodeObfuscator")]
    static void Init()
    {
        var window = GetWindow<CodeCustomObfuscator>("数据混淆工具");
        window.Show();
    }

    // 绘制窗口
    void OnGUI()
    {
        GUILayout.Label("数据混淆设置", EditorStyles.boldLabel);
        mappingPath = Path.Combine(Application.dataPath, "ClassNameMapping.txt");
        // targetScript = EditorGUILayout.ObjectField("目标脚本:", targetScript, typeof(MonoScript), false) as MonoScript;
        // 目标文件夹
        EditorGUILayout.Space();
        targetFolder = EditorGUILayout.ObjectField("目标文件夹:", targetFolder, typeof(DefaultAsset), false) as DefaultAsset;

        EditorGUILayout.Space();
        modifyClassName = EditorGUILayout.Toggle("修改类名", modifyClassName);
        modifyMethodNames = EditorGUILayout.Toggle("修改方法名", modifyMethodNames);
        modifyFieldNames = EditorGUILayout.Toggle("修改变量名", modifyFieldNames);

        EditorGUILayout.Space();
        // 拿到目标文件夹下所有脚本 MonoScript[]
        MonoScript[] scripts;
        if (targetFolder == null)
        {
            scripts = new MonoScript[0];
        }
        else
        {
            // 获取目标文件夹在 Unity 项目中的相对路径
            string folderPath = AssetDatabase.GetAssetPath(targetFolder);

            // 使用 AssetDatabase.FindAssets 搜索指定文件夹及其子文件夹中的所有脚本文件
            // "t:Script" 是 Unity 的类型过滤器，只会查找脚本文件
            // new[] { folderPath } 指定了搜索范围
            string[] scriptGuids = AssetDatabase.FindAssets("t:Script", new[] { folderPath });

            // 添加调试输出
            foreach (var guid in scriptGuids)
            {
                string scriptPath = AssetDatabase.GUIDToAssetPath(guid);
                Debug.Log($"找到脚本: {scriptPath}");
            }

            // 将找到的 GUID 转换为 MonoScript 对象数组
            scripts = scriptGuids
                // 将每个 GUID 转换为资源路径，然后加载对应的 MonoScript 资源
                .Select(guid => AssetDatabase.LoadAssetAtPath<MonoScript>(AssetDatabase.GUIDToAssetPath(guid)))
                // 过滤掉任何无效（null）的脚本引用
                .Where(script => script != null)
                // 转换为数组
                .ToArray();
        }
        // 禁用按钮
        EditorGUI.BeginDisabledGroup(targetFolder == null);
        
        if (GUILayout.Button("开始混淆"))
        {
            if (EditorUtility.DisplayDialog("确认", "是否确定要混淆代码？此操作不可逆！", "确定", "取消"))
            {
                
                // 循环script
                foreach (var script in scripts)
                {
                    // 显示脚本名称
                    EditorGUILayout.ObjectField(script.name, script, typeof(MonoScript), false);
                    // 处理脚本
                    ProcessScript(script);
                }
            }
        }
        // 结束禁用按钮
        EditorGUI.EndDisabledGroup();

        // 如果修改了类名，显示修改映射
        if (nameMapping.Count > 0)
        {
            EditorGUILayout.Space();
            EditorGUILayout.LabelField("修改映射:", EditorStyles.boldLabel);
            foreach (var pair in nameMapping)
            {
                EditorGUILayout.LabelField($"{pair.Key} -> {pair.Value}");
            }
        }
    }

    // 处理脚本
    private void ProcessScript(MonoScript script)
    {
        if (script == null)
            return;

        // 获取脚本路径
        string scriptPath = AssetDatabase.GetAssetPath(script);
        // 读取脚本内容
        string conten_Yoo_Handler_t = File.ReadAllText(scriptPath);

        // 收集需要修改的名称
        CollectNames(conten_Yoo_Handler_t);

        // 在项目中查找所有C#文件
        string[] allScripts = Directory.GetFiles(Application.dataPath, "*.cs", SearchOption.AllDirectories);

        // 修改所有文件中的引用
        foreach (string file in allScripts)
        {
            // 处理文件
            ProcessFile(file);
        }

        // 如果修改了类名，同步更新Unity中的脚本文件名
        if (modifyClassName)
        {
            // 循环nameMapping
            foreach (var pair in nameMapping)
            {
                // 获取旧类名
                string oldName = pair.Key;
                // 获取新类名
                string newName = pair.Value;

                // 查找对应的脚本资源
                string[] guids = AssetDatabase.FindAssets($"t:Script {oldName}");
                foreach (string guid in guids)
                {
                    // 获取脚本资源路径
                    string assetPat_Yoo_Net_h = AssetDatabase.GUIDToAssetPath(guid);
                    // 如果文件名等于旧类名
                    if (Path.GetFileNameWithoutExtension(assetPat_Yoo_Net_h) == oldName)
                    {
                        using (StreamWriter writer = new StreamWriter(mappingPath, true)) // 改为 true 实现追加写入
                        {
                            // 只写入类名的映射
                            writer.WriteLine($"{oldName}==>{newName}==>{newName.Substring(newName.IndexOf("_"),newName.LastIndexOf("_")-newName.IndexOf("_"))}");
                        }
                        // 使用AssetDatabase.RenameAsset重命名文件
                        // 这会保持meta文件和引用关系
                        AssetDatabase.RenameAsset(assetPat_Yoo_Net_h, newName);
                        
                        // 更新预制体中的引用
                        // UpdatePrefabReferences(oldName, newName,script);
                        
                        // 更新场景中的引用
                        UpdateSceneReferences(oldName, newName,script);
                    }
                }
            }

            // 刷新资源数据库
            AssetDatabase.SaveAssets();
            AssetDatabase.Refresh();
        }

        EditorUtility.RequestScriptReload();
        Debug.Log("数据混淆完成！");

        
    }

    private void UpdatePrefabReferences(string oldName, string newName,MonoScript script)
    {
        // 查找所有预制体
        string[] prefabGuids = AssetDatabase.FindAssets("t:Prefab");
        // 循环预制体
        foreach (string guid in prefabGuids)
        {
            // 获取预制体路径   
            string prefabPath = AssetDatabase.GUIDToAssetPath(guid);
            // 加载预制体
            GameObject prefab = AssetDatabase.LoadAssetAtPath<GameObject>(prefabPath);
            bool modi_Yoo_Base_fied = false;

            // 更新脚本引用
            var components = prefab.GetComponentsInChildren<Component>(true);
            // 循环组件
            foreach (var component in components)
            {
                // 如果组件不为空且组件类型名等于旧类名
                if (component != null && component.GetType().Name == oldName)
                {
                    // 标记修改
                    modi_Yoo_Base_fied = true;
                    break;
                }
            }

            // 如果修改了
            if (modi_Yoo_Base_fied)
            {
                // 标记修改
                EditorUtility.SetDirty(prefab);
                // 保存预制体
                PrefabUtility.SavePrefabAsset(prefab);
            }
        }
    }

    // 更新场景中的引用
    private void UpdateSceneReferences(string oldName, string newName,MonoScript script)
    {
        // 获取所有场景文件
        string[] sceneGuids = AssetDatabase.FindAssets("t:Scene");
        // 循环场景文件
        foreach (string guid in sceneGuids)
        {
            string scenePath = AssetDatabase.GUIDToAssetPath(guid);
            Scene scene = EditorSceneManager.OpenScene(scenePath, OpenSceneMode.Additive);

            bool sceneModified = false;
            // 获取场景根对象
            var rootObjects = scene.GetRootGameObjects();
            
            // 保存所有MonoBehaviour的序列化数据
            foreach (var rootObject in rootObjects)
            {
                // 获取场景中所有MonoBehaviour
                var components = rootObject.GetComponentsInChildren<MonoBehaviour>(true);
                // 循环MonoBehaviour
                foreach (var component in components)
                {
                    if (component != null)
                    {
                        var componentType = component.GetType();
                        if (componentType.Name == oldName)
                        {
                            // 保存序列化数据
                            var serializedObject = new SerializedObject(component);
                            // 获取脚本属性
                            var scriptProperty = serializedObject.FindProperty("m_Script");
                            // 获取旧脚本
                            var oldScript = scriptProperty.objectReferenceValue;
                            
                            // 找到新的脚本
                            var newScript = AssetDatabase.LoadAssetAtPath<MonoScript>(
                                AssetDatabase.GetAssetPath(script));
                            // 如果新的脚本不为空
                            if (newScript != null)
                            {
                                // 设置新的脚本
                                scriptProperty.objectReferenceValue = newScript;
                                // 应用修改
                                serializedObject.ApplyModifiedProperties();
                                EditorUtility.SetDirty(component);
                                // 标记场景修改
                                sceneModified = true;
                            }
                        }
                    }
                }
            }

            if (sceneModified)
            {
                // 标记场景修改
                EditorSceneManager.MarkSceneDirty(scene);
                // 保存场景
                EditorSceneManager.SaveScene(scene);
            }

            // 关闭场景
            EditorSceneManager.CloseScene(scene, true);
        }
    }

    // 收集需要修改的名称
    private void CollectNames(string content)
    {
        nameMapping.Clear();
        protectedNames.Clear();

        // 首先收集所有字符串中的.txt文件名
        var txtFileRegex = new Regex("\"([^\"]*\\.txt)\"");
        foreach (Match match in txtFileRegex.Matches(content))
        {
            string fileName = match.Groups[1].Value;
            // 移除.txt后缀，保护文件名主体
            string nameWithoutExt = Path.GetFileNameWithoutExtension(fileName);
            protectedNames.Add(nameWithoutExt);
        }

        if (modifyClassName)
        {
            // 修改正则表达式以更准确地匹配继承自 MonoBehaviour 的类
            var classRegex = new Regex(@"class\s+([A-Za-z_][A-Za-z0-9_]*)\s*:\s*(?:[public|private|protected|internal]\s+)?BasePanel\s*(?:,\s*[A-Za-z_][A-Za-z0-9_<>,\s]*)*\s*{");
            foreach (Match match in classRegex.Matches(content))
            {
                string className = match.Groups[1].Value;
                if (!ShouldSkipName(className) && !protectedNames.Contains(className))
                {
                    nameMapping[className] = GenerateNameWithKeyword(className);
                }
            }
        }

        if (modifyMethodNames)
        {
            var methodRegex = new Regex(@"(?:public|private|protected|internal|\s)\s+(?:static\s+)?[A-Za-z_][A-Za-z0-9_]*\s+([A-Za-z_][A-Za-z0-9_]*)\s*\(");
            foreach (Match match in methodRegex.Matches(content))
            {
                string methodName = match.Groups[1].Value;
                if (!ShouldSkipName(methodName) && !protectedNames.Contains(methodName))
                {
                    nameMapping[methodName] = GenerateNameWithKeyword(methodName);
                }
            }
        }

        if (modifyFieldNames)
        {
            // 使用(?s)标志来让.匹配换行符，并使用更宽松的空白字符匹配
            var fieldRegex = new Regex(@"(?s)(?://[^\n]*\n)*\s*(\[SerializeField\])\s*(?:private|protected)\s+(?:static\s+)?[A-Za-z_][A-Za-z0-9_<>\.]*\s+([A-Za-z_][A-Za-z0-9_]*)\s*[;=]", RegexOptions.Multiline);
            
            // 首先找出所有序列化字段
            var serializedFields = new HashSet<string>();
            foreach (Match match in fieldRegex.Matches(content))
            {
                if (match.Groups[1].Success)
                {
                    serializedFields.Add(match.Groups[2].Value);
                }
            }

            // 然后匹配所有字段
            var allFieldsRegex = new Regex(@"(?:private|protected)\s+(?:static\s+)?[A-Za-z_][A-Za-z0-9_<>\.]*\s+([A-Za-z_][A-Za-z0-9_]*)\s*[;=]", RegexOptions.Multiline);
            
            foreach (Match match in allFieldsRegex.Matches(content))
            {
                string fieldName = match.Groups[1].Value;
                
                if (serializedFields.Contains(fieldName) || protectedNames.Contains(fieldName))
                {
                    continue;
                }
                
                if (!ShouldSkipName(fieldName))
                {
                    nameMapping[fieldName] = GenerateNameWithKeyword(fieldName);
                }
            }
        }
    }

    // 检查字段是否是序列化字段
    private bool IsSerializedField(string content, string fieldName)
    {
        // 修改正则表达式以更准确地匹配字段声明
        var fieldPattern = $@"(?:private|protected)\s+(?:static\s+)?[A-Za-z_][A-Za-z0-9_<>\.]*\s+{Regex.Escape(fieldName)}\s*[;=]";
        
        var matches = Regex.Matches(content, fieldPattern, RegexOptions.Multiline);
        foreach (Match match in matches)
        {
            // 获取字段声明的位置
            int fieldPosition = match.Index;
            
            // 向上查找最近的特性标记
            int searchStartPosition = Mathf.Max(0, fieldPosition - 200); // 向上搜索200个字符
            string precedingContent = content.Substring(searchStartPosition, fieldPosition - searchStartPosition);
            
            // 检查是否有序列化相关的特性
            if (precedingContent.Contains("[SerializeField]") ||
                precedingContent.Contains("[SerializedField]") ||
                precedingContent.Contains("[Header") ||
                precedingContent.Contains("[Tooltip"))
            {
                // 确保特性和字段声明之间没有其他字段声明
                string[] lines = precedingContent.Split('\n');
                bool hasOtherDeclarations = false;
                
                for (int i = lines.Length - 1; i >= 0; i--)
                {
                    string line = lines[i].Trim();
                    if (line.Contains("[SerializeField]") || 
                        line.Contains("[SerializedField]") ||
                        line.Contains("[Header") ||
                        line.Contains("[Tooltip"))
                    {
                        break;
                    }
                    
                    if (Regex.IsMatch(line, @"^\s*(?:private|protected|public)\s+.*[;=]"))
                    {
                        hasOtherDeclarations = true;
                        break;
                    }
                }
                
                if (!hasOtherDeclarations)
                {
                    return true;
                }
            }
        }
        
        return false;
    }

    // 检查字段是否属于MonoBehaviour子类
    private bool IsMonoBehaviourField(string conten_Yoo_Handler_t, string fieldName)
    {
        // 检查类是否继承自MonoBehaviour
        var classPattern = @"class\s+[A-Za-z_][A-Za-z0-9_]*\s*:\s*MonoBehaviour";
        if (!Regex.IsMatch(conten_Yoo_Handler_t, classPattern))
            return false;

        // 检查字段是否在这个类中
        var fieldInClassPattern = @"class\s+[A-Za-z_][A-Za-z0-9_]*\s*:\s*MonoBehaviour[^}]*" + fieldName;
        return Regex.IsMatch(conten_Yoo_Handler_t, fieldInClassPattern, RegexOptions.Singleline);
    }

    // 处理文件
    private void ProcessFile(string filePath)
    {
        // 读取文件内容
        string content = File.ReadAllText(filePath);
        bool modified = false;

        // 使用 StringBuilder 提升性能
        StringBuilder sb = new StringBuilder(content);
        
        // 创建一个综合的正则表达式模式
        string combinedPattern = string.Join("|", nameMapping.Keys.Select(name => 
            $@"\b{Regex.Escape(name)}\b(?=[^A-Za-z0-9_]|$|\.|\s*[:<\s{{(),}}\[\]]|\s*\.|=|;|\+|-|\*|/)"
        ));

        // 一次性找到所有需要替换的匹配项
        var matches = Regex.Matches(content, combinedPattern)
                          .Cast<Match>()
                          .OrderByDescending(m => m.Index) // 从后向前替换,避免位置偏移
                          .ToList();

        // 批量处理所有匹配项
        foreach (var match in matches)
        {
            string oldName = match.Value;
            if (nameMapping.TryGetValue(oldName, out string newName))
            {
                sb.Remove(match.Index, match.Length);
                sb.Insert(match.Index, newName);
                modified = true;
            }
        }

        // 特殊处理 using static 语句
        string usingPattern = @"(using\s+static\s+)([A-Za-z0-9_\.]+)\s*;";
        content = sb.ToString();
        content = Regex.Replace(content, usingPattern, match =>
        {
            string prefix = match.Groups[1].Value;
            string typeName = match.Groups[2].Value;
            
            if (nameMapping.TryGetValue(typeName, out string newName))
            {
                modified = true;
                return $"{prefix}{newName};";
            }
            return match.Value;
        });

        // 如果文件内容被修改则写入
        if (modified)
        {
            File.WriteAllText(filePath, content);
            Debug.Log($"已修改文件: {filePath}");
        }
    }

    private string ReplaceIdentifier(string conten_Yoo_Handler_t, string oldName, string newName)
    {
        return Regex.Replace(conten_Yoo_Handler_t, $@"\b{oldName}\b", newName);
    }

    private bool ShouldSkipName(string name)
    {
        // 跳过4个字符及以下的名称
        if (name.Length <= 6)
            return true;

        // 检查是否是C#关键字
        if (CSharpKeywords.Contains(name))
            return true;

        // 检查是否包含下划线
        if (name.Contains("_")|| name.Contains(".") ||name.Contains(" ")||name.Contains("@"))
            return true;

        // 检查特殊后缀
        if (name.EndsWith("Sync") || name.EndsWith("Async")|| name.Contains(".txt"))
            return true;

        // 检查特殊前缀
        if (name.StartsWith("On") || name.StartsWith("on"))
            return true;

        string[] skipNames = new[] 
        { 
            "Start", "Update", "Awake", "OnEnable", "OnDisable", 
            "OnDestroy", "OnGUI", "OnValidate", "Reset"
        };

        return skipNames.Contains(name) || 
               name.StartsWith("get_") || 
               name.StartsWith("set_") ||
               name.StartsWith("__") ||
               name.StartsWith("m_");

        // 添加序列化相关的跳过规则
        if (name.StartsWith("m_") || // Unity序列化系统使用的前缀
            name.StartsWith("_") ||  // 常用的序列化字段前缀
            name.EndsWith("ID") ||   // ID字段通常需要保持
            name.EndsWith("Id"))     // Id字段通常需要保持
        {
            return true;
        }

        return false;
    }

    private string GenerateNameWithKeyword(string originalName)
    {
        // 从关键词列表中随机选择一个
        string keyword = keywords[Random.Range(0, keywords.Count)];
        int insertPosition = Random.Range(0, originalName.Length);
        return originalName.Insert(insertPosition, keyword);
    }
}
```
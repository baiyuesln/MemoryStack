#inputField #ugui 

**解决代码修改inputField.contentType不更新问题，需要的点击下输入框才可以显示**

```cs
 
using UnityEngine;
using UnityEngine.UI;
 
public class ChangeContentType : MonoBehaviour
{
    public InputField inputField;
    public Toggle toggle;
    // Start is called before the first frame update
    void Start()
    {
        toggle.onValueChanged.AddListener(ToggleEvent);
    }
    private void ToggleEvent(bool isOn)
    {
        inputField.contentType = isOn ? InputField.ContentType.Standard : InputField.ContentType.Password;
 
        //解决代码修改inputField.contentType不更新问题，需要的点击下输入框才可以显示
        inputField.ForceLabelUpdate();
 
    }
 
}
```


https://blog.csdn.net/qq_34421469/article/details/127315741
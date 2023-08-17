==帮助我们通过代码快速地找到所有的子控件，方便我们在子类中处理逻辑，节约找控件的工作量==
之前找子控件的两种方法：
1，面板上直接拖动
![[Pasted image 20230618210159.png]]
2, 在面板中声明按钮
![[Pasted image 20230618210241.png]]
然后手动拖动
==这里的 UI 基类是为了提高代码复用率，降低面板相互调用时的耦合性，同时给 UI 管理器管理。==
字典的应用，类似于缓存池里的“衣柜”与“抽屉”。父类装子类，UIBehaviour 装控件。
```csharp
public class BasePanel : MonoBehaviour  
{  
    //通过里式转换原则 来存储所有的控件  
    private Dictionary<string, List<UIBehaviour>> controlDic = new Dictionary<string, List<UIBehaviour>>();  
  
   // Use this for initialization  
   protected virtual void Awake () {  
        FindChildrenControl<Button>();  
        FindChildrenControl<Image>();  
        FindChildrenControl<Text>();  
        FindChildrenControl<Toggle>();  
        FindChildrenControl<Slider>();  
        FindChildrenControl<ScrollRect>();  
        FindChildrenControl<InputField>();  
    }   /// <summary>  
    /// 显示自己  
    /// </summary>  
    public virtual void ShowMe()  
    {            }  
  
    /// <summary>  
    /// 隐藏自己  
    /// </summary>  
    public virtual void HideMe()  
    {  
    }  
    protected virtual void OnClick(string btnName)  
    {  
    }  
    protected virtual void OnValueChanged(string toggleName, bool value)  
    {  
    }  
    /// <summary>  
    /// 得到对应名字的对应控件脚本  
    /// </summary>  
    /// <typeparam name="T"></typeparam>    
    /// <param name="controlName"></param>    
    /// <returns></returns>    
    protected T GetControl<T>(string controlName) where T : UIBehaviour  
    {  
        if(controlDic.ContainsKey(controlName))  
        {            
	        for( int i = 0; i <controlDic[controlName].Count; ++i )  
            {                
	            if (controlDic[controlName][i] is T)  
                    return controlDic[controlName][i] as T;  
            }        
        }  
        return null;  
    }  
    /// <summary>  
    /// 找到子对象的对应控件  
    /// </summary>  
    /// <typeparam name="T"></typeparam>    
    private void FindChildrenControl<T>() where T:UIBehaviour  
    {  
        T[] controls = this.GetComponentsInChildren<T>();  
        for (int i = 0; i < controls.Length; ++i)  
        {            
	        string objName = controls[i].gameObject.name;  
            if (controlDic.ContainsKey(objName))  
                controlDic[objName].Add(controls[i]);  
            else  
                controlDic.Add(objName, new List<UIBehaviour>() { controls[i] });  
            //如果是按钮控件  
            if(controls[i] is Button)  
            {                
	            (controls[i] as Button).onClick.AddListener(()=>  
                {  
                    OnClick(objName);  //把有参的虚函数包裹在了无参的lamda表达式里面
                });            
            }            
            //如果是单选框或者多选框  
            else if(controls[i] is Toggle)  
            {                
	            (controls[i] as Toggle).onValueChanged.AddListener((value) =>  
                {  
                    OnValueChanged(objName, value);  
                });            
            }        
        }    
    }
}
```
使用：
```csharp
public class LoginPanel:BasePanel{
	protected override void Awake(){
		base.Awake();//这里不能少，因为要执行父类中的Awake
	}

	public void ClickStart(){}

	public void ClickQuit(){}

	public override void OnClick(string btnName){
		switch(btnName){
			case "btnStart":
				break;
			case "btnQuit":
				break;
		}
	}
}
```
==省掉了拖动的过程==
在找控件的同时判断它的类型，监听一个无参的 lamda 表达式，但其中包裹了一个在基类里面的虚函数。
在点击的时候把控件名字传进去，子类只用重写 OnClick 函数即可知道是哪个按钮被点击。
<font color="red">原因是 Button 的监听函数传入的就是无参的，而 Toggle 传入一个 bool 值。</font>
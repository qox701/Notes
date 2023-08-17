过场景不移除
让没有继承 Mono 的类可以开启协程，可以 Updata 更新，统一管理 Update
生命周期函数、事件、协程
```csharp
public class MonoController : MonoBehaviour {  
  
    private event UnityAction updateEvent;  
  
   // Use this for initialization  
   void Start () {  
        DontDestroyOnLoad(this.gameObject);  
   }   
    
   void Update () {  
        if (updateEvent != null)  
            updateEvent();  
    }  
    /// <summary>  
    /// 给外部提供的 添加帧更新事件的函数  
    /// </summary>  
    /// <param name="fun"></param>    
    public void AddUpdateListener(UnityAction fun)  
    {        
	    updateEvent += fun;  
    }  
    /// <summary>  
    /// 提供给外部 用于移除帧更新事件函数  
    /// </summary>  
    /// <param name="fun"></param>    
    public void RemoveUpdateListener(UnityAction fun)  
    {       
	    updateEvent -= fun;  
    }
}
```
但是要运用 MonoController 需要找到挂载到这个脚本的物体，再找到这个脚本，再执行里面的函数，十分麻烦，所以有新的 MonoMgr 来制造 MonoController 提供给外部的接口
```csharp
public class MonoMgr : BaseManager<MonoMgr>  
{  
    private MonoController controller;  
  
    public MonoMgr()  
    {        
	    //保证了MonoController对象的唯一性  
        GameObject obj = new GameObject("MonoController");  
        controller = obj.AddComponent<MonoController>();  
    }  
    /// <summary>  
    /// 给外部提供的 添加帧更新事件的函数  
    /// </summary>  
    /// <param name="fun"></param>    
    public void AddUpdateListener(UnityAction fun)  
    {        
	    controller.AddUpdateListener(fun);  
    }  
    /// <summary>  
    /// 提供给外部 用于移除帧更新事件函数  
    /// </summary>  
    /// <param name="fun"></param>    
    public void RemoveUpdateListener(UnityAction fun)  
    {        
	    controller.RemoveUpdateListener(fun);  
    }  
    
    public Coroutine StartCoroutine(IEnumerator routine)  
    {        
	    return controller.StartCoroutine(routine);  
    }  
    
    public Coroutine StartCoroutine(string methodName, [DefaultValue("null")] object value)  
    {        
	    return controller.StartCoroutine(methodName, value);  
    }  
    
    public Coroutine StartCoroutine(string methodName)  
    {        
	    return controller.StartCoroutine(methodName);  
    }
}
```
使用：
```csharp
public class TestTest{
	public TeseTest(){
	MonoMgr.GetInstance().StartCoroutine(Test123());
	}
	
	IEnumerator Test123(){
		yield return new WaitForSeconds(1f);
		Debug.Log("123123");
	}
	public void Update(){
		Debug.Log("TestTest");
	}
}

Public class Test:MonoBehaviour{
	void Start(){
		TestTest t=new TestTest();
		MonoMgr.GetInstance().AdUpdateListener(t.Update);
	}
}
```
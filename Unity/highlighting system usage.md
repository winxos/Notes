# Unity3d HighlightingSystem 插件使用

> HighlightingSystem  插件用于物体高亮效果，目前最新版本为4.0。

## 使用步骤

1. Unity测试版本为5.5，导入插件。

2. 将Assets-> HighlightingSystem-> Scripts 下的HighlightingRenderer绑定到摄像机。

3. 将Assets-> HighlightingSystem-> Scripts 下的Highlighter绑定到物体。

4. 新建C#脚本，绑定到物体，代码如下：

   ``` c#
   using UnityEngine;
   using System.Collections;
   using HighlightingSystem;
   public class NewBehaviourScript : MonoBehaviour {
   	// Use this for initialization
   	public Highlighter h;
   	void Start () {
   		h = this.transform.GetComponent<Highlighter> ();
   		h.FlashingOn (Color.red,Color.blue,0.1f);
   	}
   	// Update is called once per frame
   	void Update () {	
   	}
   }
   ```

   注意导入命名空间。

5. 通过Highlighter就可以控制全部参数，详细的函数参考在Assets -> HighlighterSystemDemo下Documentation中。

自己再配合其他代码使用，说明完毕。

winxos 2017-01-11
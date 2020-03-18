# 应用升级检测实现

> Unity3D，安卓应用为例，实现应用版本检测

## 业务流程

1. 应用启动时从服务器API检测最新版本号等信息。
2. 服务器返回JSON数据格式，包含当前最新版本，下载地址等信息。可以做成验证方式，链接定期失效。
3. 本地比对版本，如果版本新，引导用户前往指定地址下载新版。

## Unity3D中实现

### 客户端实现

应用启动时通过WWW进行远程JSON获取，代码如下：

```c#
public class updater : MonoBehaviour {
    public GameObject updaterui;
    const string UPDATE_SERVER = "http://a.aistl.com:666";
    const string UPDATE_API = UPDATE_SERVER + "/version.json";
    [Serializable]
    public class VersionInfo
    {
        public string name;
        public string version;
        public string url;
    }
    public void open_website()
    {
        Application.OpenURL(UPDATE_SERVER);
    }
    public void cancel_update()
    {
        AMain.ui_state = 0;
    }
    IEnumerator get_json()
    {
        Debug.Log(Application.internetReachability);
        if(Application.internetReachability==NetworkReachability.NotReachable)//no network
            yield break;
        WWW json_data = new WWW(UPDATE_API);
        yield return json_data;
        VersionInfo v=JsonUtility.FromJson<VersionInfo>(json_data.text);
        
        Version remote_v=new Version(v.version);
        Version local_v = new Version(Application.version);
        Debug.Log(remote_v);
        Debug.Log(local_v);
        if (remote_v > local_v)//update need
        {
            updaterui.transform.Find("Panel/local").gameObject.GetComponent<Text>().text =   "    本地版本：" + local_v;
            updaterui.transform.Find("Panel/server").gameObject.GetComponent<Text>().text = "服务器版本：" + remote_v;
            AMain.ui_state = -1;
        }
    }
    void Start()
    {
        StartCoroutine(get_json());
    }
    void Update () {
	}
}
```

使用时将脚本绑定到gameobject，再将更新时的ugui界面绑定到gameobject对应updater的updaterui

点击升级按钮后打开浏览器跳转到软件下载页面，手动进行升级。

> 提示：C#中提供了内置的Version类用于版本比较。

### 服务器端实现

最简便的方式是在服务器用nginx提供静态文件下载方式来实现，因为是测试，服务器就用家里树莓派搭建的，为相关应用分配特定域名下某个端口，再用nginx进行监听，指向应用相关文件夹。服务器上应该包含页面文件（如果需要），应用APK安装包，应用版本信息。这里版本信息采用json文件来进行存放，参考内容如下：

``` json
{
  "name":"ar_demo",
  "version":"0.3",
  "url":"http://a.aistl.com:666/ar_demo.apk"
}
```

里面存放了应用名称，服务器最新版本，还有apk下载地址。

如果是在浏览器页面进行下载，apk下载地址还可以省略。

如果以后要商用，只需把资源丢到CDN上就可以了。

## 小结

至此，给app添加自动升级功能就算初步实现了，当然我们还可以改进，比如不要跳转到浏览器下载，直接在应用内下载，这样可能更好一点。

winxos 2017-02-05
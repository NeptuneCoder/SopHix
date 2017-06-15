通过[这篇文章](https://mp.weixin.qq.com/s?__biz=MzIzOTU0NTQ0MA==&mid=2247485095&idx=1&sn=5a57f6e0ce147470aca5fb2425dd7fc9&chksm=e9293ba8de5eb2bee7a53323f71c70799b005d8e03a1565762140e2f927082d8990ad5a6d36d&mpshare=1&scene=1&srcid=0615xsa9fQtiZId268VBaIWA&key=dbf25e1666f5840ae2c067fe25093347aa23ebb5e373ae266979b24a4dd9274e5f14c39084faf53832f5cb273aed8beebd6395f0a19d38b508fdec78495789a4a7fee515bb7496563271bd2e2ae7b7f4&ascene=0&uin=MTY3NDYzMDU2Mg%3D%3D&devicetype=iMac+MacBookPro11%2C4+OSX+OSX+10.11.6+build(15G1510)&version=12020710&nettype=WIFI&fontScale=100&pass_ticket=8D1gIOpBnToDWH75FzNbE7iLVrGdKCEHDq8JNb518a%2Fr0gcuaLmMYbIgQ4ZBYixT)了解到阿里又出来了新的热修复框架。今天写了个demo体验了一下,记录一下步骤。
#### 一app要做的事情：[官方文章](https://help.aliyun.com/document_detail/53238.html?spm=5176.doc53247.6.545.CPxyW3)
1.申请测试账号，然后创建一个app，获取AppId，和AppSecret以及RSA密钥。上面三个参数需要配置在application节点下面：
```
      <meta-data
            android:name="com.taobao.android.hotfix.IDSECRET"
            android:value="AppId" />
        <meta-data
            android:name="com.taobao.android.hotfix.APPSECRET"
            android:value="AppSecret" />
        <meta-data
            android:name="com.taobao.android.hotfix.RSASECRET"
            android:value="RSA密钥" />
```
2.通过gradle依赖相应的库
```
 maven {
            url "http://maven.aliyun.com/nexus/content/repositories/releases"
        }
```
```
compile 'com.aliyun.ams:alicloud-android-hotfix:3.0.2'
```

3.在Application中配置如下代码：
```
SophixManager.getInstance().setContext(this)
                .setAppVersion(BuildConfig.VERSION_NAME)
                .setAesKey(null)
                .setEnableDebug(true)
                .setPatchLoadStatusStub(new PatchLoadStatusListener() {
                    @Override
                    public void onLoad(final int mode, final int code, final String info, final int handlePatchVersion) {
                        Log.i("code","mode = "+mode+"info = "+ info);
                        // 补丁加载回调通知
                        if (code == PatchStatus.CODE_LOAD_SUCCESS) {
                            // 表明补丁加载成功
                            Log.i("code","表明补丁加载成功");
                        } else if (code == PatchStatus.CODE_LOAD_RELAUNCH) {
                            // 表明新补丁生效需要重启. 开发者可提示用户或者强制重启;
                            // 建议: 用户可以监听进入后台事件, 然后应用自杀
                            Log.i("code","用户可以监听进入后台事件, 然后应用自杀");
                        } else if (code == PatchStatus.CODE_LOAD_FAIL) {
                            // 内部引擎异常, 推荐此时清空本地补丁, 防止失败补丁重复加载
                            // SophixManager.getInstance().cleanPatches();
                            Log.i("code","内部引擎异常, 推荐此时清空本地补丁, 防止失败补丁重复加载");
                        } else {
                            // 其它错误信息, 查看PatchStatus类说明
                            Log.i("code"," 其它错误信息, 查看PatchStatus类说明");
                        }
                    }
                }).initialize();
        SophixManager.getInstance().queryAndLoadNewPatch();
```
4.申请对应的权限
 ```
 <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
```

文章介绍SopHix是非侵入式的，意思就是在项目中依赖很少。以上就完成了所有的配置。

#### 二：生成补丁包[官方文章](https://help.aliyun.com/document_detail/53247.html?spm=5176.doc51434.6.548.cV4Txt)

通过Part2[Mac](http://ams-hotfix-repo.oss-cn-shanghai.aliyuncs.com/SophixPatchTool_macos.zip?spm=5176.doc53287.2.34.vZxNDm&file=SophixPatchTool_macos.zip) ---[Window](http://ams-hotfix-repo.oss-cn-shanghai.aliyuncs.com/SophixPatchTool_windows.zip?spm=5176.doc53287.2.35.vZxNDm&file=SophixPatchTool_windows.zip) 生成补丁包。相同的版本，一个包有bug，和一个已修复bug的包，通过Part2生成对应的补丁包。



#### 三：上传补丁包到后台。[官方文章](https://help.aliyun.com/document_detail/51434.html?spm=5176.doc53287.6.552.vZxNDm)
在后台新建的版本号要和app的版本号要一致，不然app到时不知道要下载那个补丁包。
  按照官网推荐的三步走发布流程。1.先本地测试，2，在灰度发布，3.最后全量发布。

遇到的问题：
1.今天我用的是华为手机测试的，前面几次测试一直失败,后面发现是没有把权限申请下来。如果不成功，建议检查一下app的权限，和网络。
2.我在后台发布成功了补丁包，有时候app需要启动好几次才能搞把补丁包加载成功，不清楚是不是我那个地方处理的不对！
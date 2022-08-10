# 注册应用

### 开通账号

LovinJoy给您开通产品服务后，会提供一个超级管理员账号，登录https://datatower.ai/ 可以创建项目

### 创建项目

登录后，进入**项目设置**

![image-image-project-setting](https://github.com/lovinjoy/datatower.ai-core-android/blob/main/resurce/image-project-setting.png)

切换到**系统设置**，点击**创建项目**

![image-create-project](https://github.com/lovinjoy/datatower.ai-core-android/blob/main/resurce/image-create-project.png)

输入项目名称，如：simple_android

![image-setting-name](https://github.com/lovinjoy/datatower.ai-core-android/blob/main/resurce/image-setting-name.png)

创建成功后可以看到APP_ID

![image-appid-view](https://github.com/lovinjoy/datatower.ai-core-android/blob/main/resurce/image-appid-view.png)

记下APP_ID，初始化SDK会用到

### 

# 初始化SDK

### 集成

1.下载 Unity SDK 资源包 

2.将上述文件通过 Assets → Import Package → Custom Package 添加到项目中

3.将 ROIQuerySDK/ROIQuerySDK/ 目录下的 ROIQuerySDK.prefab 预制体拖到需要加载的位置(一般放在第一个场景)

 ![image-appid-view](https://github.com/lovinjoy/datatower.ai-core-android/blob/main/resurce/unity_1.png)
 
4.配置SDK

 ![image-appid-view](https://github.com/lovinjoy/datatower.ai-core-android/blob/main/resurce/unity_2.png)
 
- App Id：项目Id，即上一步申请到的 APP_ID，必须填
- Channel：渠道名称，打多渠道包时需要用到，如gp、app_store，默认为“”，可不填
- Is Debug：是否打开调试，调试模式下将打印log, 默认为false
- Log Level：log 的级别，默认为 LogUtils.V，仅在 isDebug = true 有效
	


### 初始化

- 拖入场景自动初始化


# 自定义事件

调用track()方法来自定义一个事件，并自动上报

```c#
Dictionary<string, object> properties = Dictionary<string, object>();
properties.Add("test_property_3", false);
properties.Add("test_property_4", 2.3);

ROIQueryAnalytics.Track("test_track", properties);
// or 不传属性
ROIQueryAnalytics.Track("test_track");
```



# 广告事件（可选）

SDK  内置了一些有关广告相关的行为事件，可供开发者在收到广告SDK回调时，通过调用相关的接口进行上报，用于分析广告相关的信息

### 独立广告平台

插页广告的过程作为演示

```c#
 /*
 * 一次展示 MAX 激励广告的过程
 */
 
 
 MaxSdkCallbacks.Rewarded.OnAdDisplayedEvent += OnRewardedVideoShownEvent;
 MaxSdkCallbacks.Rewarded.OnAdClickedEvent += OnRewardedVideoClickedEvent;
 MaxSdkCallbacks.Rewarded.OnAdRevenuePaidEvent += OnRewardedAdRevenuePaidEvent;
 MaxSdkCallbacks.Rewarded.OnAdHiddenEvent += OnRewardedVideoClosedEvent;
 MaxSdkCallbacks.Rewarded.OnAdDisplayFailedEvent += OnRewardedVideoFailedToPlayEvent;
 MaxSdkCallbacks.Rewarded.OnAdReceivedRewardEvent += OnRewardedVideoReceivedRewardEvent;
 
 
//广告位，比如这页面是主页
string location = "main"
 
//MAX 广告单元
string maxAdUnit = "3940256099942544/1033173712"
 
//整个过程的行为系列标识
string seq = ROIQueryAdReport.GenerateUUID()
 
//1. 开始加载广告
loadInterstitialAd(maxAdUnit); 

//2.即将展示广告
ROIQueryAdReport.ReportToShow("", AdType.REWARDED, AdPlatform.IDLE, location, seq)

//3.展示广告
showInterstitialAd()

......

//广告回调

//4. 广告展示成功
private void OnRewardedVideoShownEvent(string adUnitId, MaxSdkBase.AdInfo adInfo){

    ROIQueryAdReport.ReportShow(GetNetworkAdunit(maxAdUnit, adInfo), AdType.REWARDED, GetNetwork(adInfo), location, seq)
}


//5. 广告展示失败
private void OnRewardedVideoFailedToPlayEvent(string adUnitId,MaxSdkBase.ErrorInfo error, MaxSdkBase.AdInfo adInfo){

    ROIQueryAdReport.ReportShowFailed(GetNetworkAdunit(maxAdUnit, adInfo), AdType.REWARDED, GetNetwork(adInfo), location, seq, adError.code, adError.msg)
}

//6. 广告被点击
private void OnRewardedVideoClickedEvent(string adUnitId, MaxSdkBase.AdInfo adInfo){     	

    ROIQueryAdReport.ReportClick(GetNetworkAdunit(maxAdUnit, adInfo), AdType.REWARDED,  GetNetwork(adInfo), location, seq)
    ROIQueryAdReport.ReportConversionByClick(GetNetworkAdunit(maxAdUnit, adInfo), AdType.REWARDED,  GetNetwork(adInfo), location, seq)
}

//7. 广告关闭
private void OnRewardedVideoClosedEvent(){
       	
	ROIQueryAdReport.ReportClose(GetNetworkAdunit(maxAdUnit, adInfo), AdType.REWARDED, GetNetwork(adInfo), location, seq)
}
	
//8.对于激励广告，会有获得激励回调
private void OnAdReceivedRewardEvent(){
		
    ROIQueryAdReport.ReportRewared(GetNetworkAdunit(maxAdUnit, adInfo), AdType.REWARDED, GetNetwork(adInfo), location, seq)
    ROIQueryAdReport.ReportConversionByRewared(GetNetworkAdunit(maxAdUnit, adInfo), AdType.REWARDED, GetNetwork(adInfo), location, seq)
}
   
//9. 广告用户层级展示数据
private void OnAdRevenuePaidEvent(string adUnitId, MaxSdkBase.AdInfo adInfo){
		
     string value = adInfo.Revenue.ToString(); //广告的价值
     string currency = "USD" //货币
     string precision = adInfo.RevenuePrecision; // 精确度
     
     ROIQueryAdReport.ReportPaid(GetNetworkAdunit(maxAdUnit, adInfo), AdType.INTERSTITIAL, GetNetwork(adInfo), location, seq, value, currency, precision)
 }

    

```





### 聚合广告平台

由于聚合广告平台展示广告的时候，没有返回具体是哪个广告平台的广告，所以需要在回调中判断，这里提供了MAX平台的判断方法，可以拷贝到项目的工具类

```c#
//聚合广告平台一般会返回广告相关的信息 AdInfo

public class AdUtils
    {
        private const string MAXPlatformApplovin = "AppLovin";
        private const string MAXPlatformApplovinExchange = "APPLOVIN_EXCHANGE";
        private const string MAXPlatformAdmob = "Google AdMob";
        private const string MAXPlatformAdx = "Google Ad Manager";
        private const string MAXPlatformBigo = "BigoAds";
        private const string MAXPlatformAdmobNative = "Google AdMob Native";
        private const string MAXPlatformAdxNative = "Google Ad Manager Native";
	
	
	public static string GetNetworkAdunit(string maxAdUnit, MaxSdkBase.AdInfo adInfo)(){
	      return GetNetwork(adInfo) == AdPlatform.APPLOVIN ? maxAdUnit : adInfo.NetworkPlacement;
	}

        public static AdPlatform GetNetwork(MaxSdkBase.AdInfo adInfo)
        {
            switch (adInfo.NetworkName)
            {
                case MAXPlatformApplovin:
                    return AdPlatform.APPLOVIN;
                case MAXPlatformApplovinExchange:
                    return AdPlatform.APPLOVIN_EXCHANGE;
                case MAXPlatformAdmob:
                    return AdPlatform.ADMOB;
                case MAXPlatformAdx:
                    return AdPlatform.ADX;
                case MAXPlatformBigo:
                    return AdPlatform.BIGO;
                case MAXPlatformAdmobNative:
                    return AdPlatform.ADMOB;
                case MAXPlatformAdxNative:
                    return AdPlatform.ADX;
                default:
                    return AdPlatform.IDLE;
            }
        }
    }

```




随着人脸识别技术在金融、医疗等多个领域的加速落地，网络安全、信息泄露等问题愈为突出，用户对应用稳定性和安全性的要求也更为严格。


HarmonyOS SDK [场景化视觉服务](https://github.com "场景化视觉服务")（Vision Kit）提供[人脸动作活体检测](https://github.com "人脸动作活体检测")能力，增强对于非活体攻击的防御能力和活体通过率。在投资理财、在线支付等高风险金融服务场景中，通过检测用户的组合动作等来验证用户为真实活体操作，抵御攻击，提高安全性，降低业务风险，全方位保障用户体验及信息安全。


动作活体检测支持实时捕捉人脸，采用指令动作配合的方式进行活体检测，开发者可以自定义检测的动作数量（最多为4个），系统会从6种预设动作（眨眼、张嘴、左摇头、右摇头、注视、点头）中随机生成所填数量的动作，用户按指令完成动作就可以判断是真实活体，还是非活体攻击（比如：人脸翻拍图片、人脸面具等）。


### 能力优势


**端侧闭环**：人脸活体检测作为纯端侧能力，数据不保存，杜绝篡改和泄漏，确保数据安全和用户隐私。


**空间小，响应快**：实现了高频攻击增强防护，同时保持ROM空间占用小，响应时间毫秒级，提供流畅的用户体验。


**防护升级**： Vision Kit的防护系统通过自研算法升级，能够有效抵御高精视频和高精面具攻击。


**开发简单**：支持UI定制，以满足个性化需求。经过两年以上的持续演进，积累了亿级数据和千万级的API调用量，成熟稳定。


### 功能演示


![image](https://img2024.cnblogs.com/blog/2396482/202501/2396482-20250107102741269-465257617.gif)


### 开发步骤


1\.将实现人脸活体检测相关的类添加至工程。



```
import { interactiveLiveness } from '@kit.VisionKit';

```

2\.在module.json5文件中添加CAMERA权限，其中reason，abilities标签必填，配置方式参见[requestPermissions标签说明](https://github.com "requestPermissions标签说明")。



```
"requestPermissions":[
  {
    "name": "ohos.permission.CAMERA",
    "reason": "$string:camera_desc",
    "usedScene": {"abilities": []}
  }
]

```

3\.简单配置页面的布局，选择人脸活体检测验证完后的跳转模式。如果使用back跳转模式，表示的是在检测结束后使用router.back()返回。如果使用replace跳转模式，表示的是检测结束后使用router.replaceUrl()去跳转相应页面。默认选择的是replace跳转模式。



```
Flex({ direction: FlexDirection.Row, justifyContent: FlexAlign.Start, alignItems: ItemAlign.Center }) {
  Text("验证完的跳转模式：")
    .fontSize(18)
    .width("25%")
  Flex({ direction: FlexDirection.Row, justifyContent: FlexAlign.Start, alignItems: ItemAlign.Center }) {
    Row() {
      Radio({ value: "replace", group: "routeMode" }).checked(true)
        .height(24)
        .width(24)
        .onChange((isChecked: boolean) => {
          this.routeMode = "replace"
        })
      Text("replace")
        .fontSize(16)
    }
    .margin({ right: 15 })

    Row() {
      Radio({ value: "back", group: "routeMode" }).checked(false)
        .height(24)
        .width(24)
        .onChange((isChecked: boolean) => {
          this.routeMode = "back";
        })
      Text("back")
        .fontSize(16)
    }
  }
  .width("75%")
}

```

4\.如果选择动作活体模式，可填写验证的动作个数。



```
Flex({ direction: FlexDirection.Row, justifyContent: FlexAlign.Start, alignItems: ItemAlign.Center }) {
  Text("动作数量：")
    .fontSize(18)
    .width("25%")
  TextInput({
    placeholder: this.actionsNum != 0 ? this.actionsNum.toString() : "动作数量最多4个"
  })
    .type(InputType.Number)
    .placeholderFont({
      size: 18,
      weight: FontWeight.Normal,
      family: "HarmonyHeiTi",
      style: FontStyle.Normal
    })
    .fontSize(18)
    .fontWeight(FontWeight.Bold)
    .fontFamily("HarmonyHeiTi")
    .fontStyle(FontStyle.Normal)
    .width("65%")
    .onChange((value: string) => {
      this.actionsNum = Number(value) as interactiveLiveness.ActionsNumber;
    })
}

```

5\.点击"开始检测"按钮，触发点击事件。



```
Button("开始检测", { type: ButtonType.Normal, stateEffect: true })
  .width(192)
  .height(40)
  .fontSize(16)
  .backgroundColor(0x317aff)
  .borderRadius(20)
  .margin({
    bottom: 56
  })
  .onClick(() => {
    this.privateStartDetection();
  })

```

6\.触发CAMERA权限校验。



```
// 校验CAMERA权限
private privateStartDetection() {
  abilityAccessCtrl.createAtManager().requestPermissionsFromUser(this.context, this.array).then((res) => {
    for (let i = 0; i < res.permissions.length; i++) {
      if (res.permissions[i] === "ohos.permission.CAMERA" && res.authResults[i] === 0) {
        this.privateRouterLibrary();
      }
    }
  }).catch((err: BusinessError) => {
    hilog.error(0x0001, "LivenessCollectionIndex", `Failed to request permissions from user. Code is ${err.code}, message is ${err.message}`);
  })
}

```

7\.配置人脸活体检测控件的配置项InteractiveLivenessConfig，用于跳转到人脸活体检测控件。配置中具体的参数可参考[API文档](https://github.com "API文档"):[veee\+加速器官方下载](https://58pifa.com)。



```
let routerOptions: interactiveLiveness.InteractiveLivenessConfig = {
  isSilentMode: this.isSilentMode as interactiveLiveness.DetectionMode,
  routeMode: this.routeMode as interactiveLiveness.RouteRedirectionMode,
  actionsNum: this.actionsNum
};

```

8\.调用interactiveLiveness的startLivenessDetection接口，判断跳转到人脸活体检测控件是否成功。



```
// 跳转到人脸活体检测控件
private privateRouterLibrary() {
  if (canIUse("SystemCapability.AI.Component.LivenessDetect")) {
    interactiveLiveness.startLivenessDetection(routerOptions).then((DetectState: boolean) => {
      hilog.info(0x0001, "LivenessCollectionIndex", `Succeeded in jumping.`);
    }).catch((err: BusinessError) => {
      hilog.error(0x0001, "LivenessCollectionIndex", `Failed to jump. Code：${err.code}，message：${err.message}`);
    })
  } else {
    hilog.error(0x0001, "LivenessCollectionIndex", 'this api is not supported on this device');
  }
}

```

9\.检测结束后回到当前界面，可调用interactiveLiveness的getInteractiveLivenessResult接口，验证人脸活体检测的结果。



```
// 获取验证结果
private getDetectionResultInfo() {
  // getInteractiveLivenessResult接口调用完会释放资源
  if (canIUse("SystemCapability.AI.Component.LivenessDetect")) {
    let resultInfo = interactiveLiveness.getInteractiveLivenessResult();
    resultInfo.then(data => {
      this.resultInfo = data;
    }).catch((err: BusinessError) => {
      this.failResult = {
        "code": err.code,
        "message": err.message
      }
    })
  } else {
    hilog.error(0x0001, "LivenessCollectionIndex", 'this api is not supported on this device');
  }
}

```

**了解更多详情\>\>**


访问[场景化视觉服务联盟官网](https://github.com "场景化视觉服务联盟官网")


获取[人脸活体检测开发指导文档](https://github.com "人脸活体检测开发指导文档")



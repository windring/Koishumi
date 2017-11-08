本文将引导完成一个简单的Bot Framework机器人的建立。

### 0x0 什么是Bot Framework?

[botframework](https://dev.botframework.com/ "")，顾名思义，是微软的一个机器人的框架。它能很好地与你的用户进行交互，比如文字交流。botframework还提供了跨平台连接器，使得你的机器人能在多平台（比如skype）上运行而不需要写额外的代码。

### 0x1 使用Bot Builder SDK for C#在本地开始一个bot

官方的[bot-builder-dotnet-quickstart](https://docs.microsoft.com/en-us/bot-framework/dotnet/bot-builder-dotnet-quickstart "")提供了很好的建立步骤，按照引导可以很快地建立一个bot。

**在以下过程中请确保你的软件和SDK是最新的**

1. 请安装好visual studio 2017，并且在visual studio installer中勾选安装VC#（你可以仅勾选"Azure开发"，这个负载中包含了核心的.NET开发工具，并且在后续步骤的发布方便许多）。
2. 下载[Bot Application](http://aka.ms/bf-bc-vstemplate "")，[Bot Controller](http://aka.ms/bf-bc-vscontrollertemplate "")，[Bot Dialog](http://aka.ms/bf-bc-vsdialogtemplate "")，将三个zip文件，不解压地放在目录```%USERPROFILE%\Documents\Visual Studio 2017\Templates\ProjectTemplates\Visual C#\```下，也就是模板文件夹。一般情况下，打开文档文件夹就能看到`Visual Studio 2017`文件夹。它们是快速开始一个C#bot的模板。
3. 新建项目Visual C#->Bot Application（如果你没有看到Bot Application模板，说明SDK安装异常）。完成向导。此时项目下已经包含完整的建立一个简单的bot所需的所有文件。
4. 更新SDK  
在解决方案资源管理器右键项目，选择```管理NuGet程序包```，在弹出的页面的搜索栏中键入```Microsoft.Bot.Builder```，在搜索结果中定位到到它（应该是唯一的），点击蓝色的更新按钮。完成向导。
5. 运行并选择一个浏览器，此时将会弹出一个地址为```http://localhost:3979/```的浏览器窗口。如果你看到
![you see the browser page](https://raw.githubusercontent.com/windring/Koishumi/markdown/img/Snipaste_2017-11-06_13-28-38.png "")
那么很好，后端端口似乎准备好了。

### 0x2 使用Bot Framework Emulator测试此bot

你需要安装一个[Bot Framework Emulator](https://emulator.botframework.com/ "")。安装，运行。在蓝色的地址栏中键入你的服务地址（通常是```http://localhost:3979/api/messages```），点击connect按钮，如果连接正常应该会在log栏产生如下的蓝色显示：

![Snipaste_2017-11-06_13-18-31.png](https://raw.githubusercontent.com/windring/Koishumi/markdown/img/Snipaste_2017-11-06_13-18-31.png "")

此时与你的bot进行通讯，按照模板所code，它应该会返回你发送的信息和所包含字符数。

###0x3连接此bot到QnA Maker

#### 什么是QnA Maker？

[QnA Maker](https://qnamaker.ai/ "")，微软的一个Bot FAQ Service，提供了一个问答（FAQ）匹配功能。你只需提供你认为需要的问题和答案匹配列表(或者其他方式)，当你发起申请时，它将返回最匹配的问题和答案，当然也可能找不到匹配内容。

1. 访问QnA Maker，使用你的微软账号登录，点击Create New Service，建立一个新服务。成功后，应该能在[MyServices](https://qnamaker.ai/Home/MyServices "")页面看到你新建的服务。
![Service](https://raw.githubusercontent.com/windring/Koishumi/markdown/img/Snipaste_2017-11-06_19-36-40.png "")

2. 点击"edit->Settings"，然后可以看到一系列设置。 要建立此服务对于问题的答案回应，我们可以提供一个包含问题与答案的文本文件。问题与答案用tab键分隔开，回车以表分隔问题与问题。在Files栏提交文件，并点击save and train按钮。QnA Maker会自动进行匹配训练，之后的提问并不需要百分之分地符合列出的问题也能找到匹配的答案，如果真的not found，则会返回```No good match found in the KB```。如果已经有现成的FAQ页面，比如微软俱乐部的问答页面[http://studentclub.msra.cn/bop2017/qa](http://studentclub.msra.cn/bop2017/qa "")，我们可以键到URLs，然后保存训练，点击publish按钮，即可使用问答匹配。
![Settings](https://raw.githubusercontent.com/windring/Koishumi/markdown/img/Snipaste_2017-11-06_19-43-34.png "")

3. 连接QnA到此bot。在Settings的最下方，我们能看到部署细节，也就是发起请求的必要参数。这些参数将在后续步骤使用。
![Snipaste_2017-11-06_21-03-11.png](https://raw.githubusercontent.com/windring/Koishumi/markdown/img/Snipaste_2017-11-06_21-03-11.png "")
如果发出的请求得到回复，将返回一个json数据，类似：
![json data](https://raw.githubusercontent.com/windring/Koishumi/markdown/img/Snipaste_2017-11-06_23-40-22.png "")
页面[ApiReference](https://westus.dev.cognitive.microsoft.com/docs/services/58994a073d9e04097c7ba6fe/operations/58994a073d9e041ad42d9ba9 "")给出了C#的调用方法。打开vs项目，编辑```Dialogs->RootDialog.cs```。在头部添加引用：
```cs
using System.Net.Http.Headers;
using System.Text;
using System.Net.Http;
using System.Web;
using System.Collections.Generic;
using Newtonsoft.Json;
```
在类RootDialog之前添加类QnAMakerResult，这是用于解析json数据的模板。
```cs
public class QnAMakerResult
{
    public class Answers
    {
        [JsonProperty(PropertyName = "answer")]
        public string answer { get; set; }
        [JsonProperty(PropertyName = "questions")]
        public List<string> questions { get; set;}
        [JsonProperty(PropertyName = "score")]
        public double score { get; set; }
    }
    [JsonProperty(PropertyName = "answers")]
    public List<Answers> answers { get; set; }
```
将行
```cs
await context.PostAsync($"You sent {activity.Text} which was {length} characters");
```
注释掉，用QnA Maker提供的方法替换。将相应的```Ocp-Apim-Subscription-Key```，```/knowledgebases*generrateAnswer```分别替换成此QnA Maker Service的Ocp-Apim-Subscription-Key，POST参数。
```cs
//await context.PostAsync($"You sent {activity.Text} which was {length} characters");
var client = new HttpClient();
var queryString = HttpUtility.ParseQueryString(string.Empty);
// Request headers
client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", "a79016116f*********24d20869e894a");
var uri = "https://westus.api.cognitive.microsoft.com/qnamaker/v2.0/knowledgebases/2d219575-*********-ac65-ebf9a5208cc7/generateAnswer?" + queryString;
HttpResponseMessage response;
// Request body
byte[] byteData = Encoding.UTF8.GetBytes($"{{\"question\": \"{activity.Text}\",\"top\":1}}");
using (var content = new ByteArrayContent(byteData))
{
    content.Headers.ContentType = new MediaTypeHeaderValue("application/json");
    response = await client.PostAsync(uri, content);
    await context.PostAsync(response.Content.ReadAsAsync<QnAMakerResult>().Result.answers[0].answer);
}
```
运行项目，回到Bot Framework Emulator，键入"微软俱乐部简介"，应该能得到如下回应:
![FAQ](https://raw.githubusercontent.com/windring/Koishumi/markdown/img/Snipaste_2017-11-06_23-51-46.png "")
接下来，请好好地训练此QnA Maker Service吧。

### 0x4 发布此bot到Azure

1. 注册Azure.cn  
如果你不使用Azure服务，也可以自行配置环境，并将release包放置在你的服务器上。由于某些原因，中国大陆的microsoft账号并不能使用Azure的国际服务。如果你想试试你的运气，你可以在[国际azure](https://azure.microsoft.com/zh-cn/ "")尝试注册，在此劝告你，如果你有非中国大陆的身份资料信息，请优先使用。如果你是学生，可以在此[imagine](https://imagine.microsoft.com/zh-cn "")使用校园邮箱或其他方式完成学生认证。如果你的国际azure账户没有出问题，应该能拿到一定的免费额度（比如web app服务）。对于在中国大陆的高校生，可参照此学生俱乐部的[FAQ](http://studentclub.msra.cn/bop2017/qa "")完成在世纪互联azure的注册，并获得1元订阅。  
注意，在此页，请先点击发送验证码按钮，然后点击验证按钮，最后继续按钮才会亮起，通常发送按钮和验证按钮是简明的icon形式。请注意记住你的id，分配的域名，和你的密码。通常登录名为```id@域名```。
![sign up](https://raw.githubusercontent.com/windring/Koishumi/markdown/img/Snipaste_2017-11-07_07-56-10.png "")
2. 创建web app  
在azure.cn的portal面板（请注意是以cn为结尾的中国区网站而不是以com为结尾的国际网站），点击```+新建```按钮，此时web app项目应该为亮起可用状态。点击web app下的create按钮，在创建向导中填写应用名称，其他使用默认设置，建议勾选固定到仪表盘，无误后点击创建按钮，此时应该会跳转回仪表盘页面，而web应用为正在部署状态。
![create app](https://raw.githubusercontent.com/windring/Koishumi/markdown/img/Snipaste_2017-11-07_08-06-34.png "")
![create app](https://raw.githubusercontent.com/windring/Koishumi/markdown/img/Snipaste_2017-11-07_08-12-03.png "")
片刻后转到刚刚创建的app，点击```部署凭据```，在此创建发布用的ftp用户并保存，此用户将在发布中使用。  
点击```概述```，在此页将看到ftp主机名和新创建的ftp部署用户名。
![信息](https://raw.githubusercontent.com/windring/Koishumi/markdown/img/Snipaste_2017-11-07_08-33-38.png "")
3. 使用ftp发布  
回到visual studio的项目，右键解决方案资源管理器中的项目，点击发布选项，呼出发布页面。请注意，请在发布页面选择ftp选项，如果你想使用Azure App Service发布选项，可以在此页[developerdifferences](https://docs.azure.cn/zh-cn/articles/guidance/developerdifferences "")获取相关步骤。
![发布](https://raw.githubusercontent.com/windring/Koishumi/markdown/img/Snipaste_2017-11-07_08-26-43.png "")
按照在azure app的概述页面提供的信息，填写好相应的参数，站点路径请填写```/site/wwwroot```，无误后可以验证连接，当绿色勾出现说明连接正确，最后点击保存按钮。
![data](https://raw.githubusercontent.com/windring/Koishumi/markdown/img/Snipaste_2017-11-07_08-28-12.png "")
此时vs应该已经开始自动发布，如果没有，可以手动点击发布按钮。
4. 验证发布  
在azure的app 概述页面，找到你的app url，通常为绿色框所示:
![url](https://raw.githubusercontent.com/windring/Koishumi/markdown/img/Snipaste_2017-11-07_08-33-38 - 1.png "")
在浏览器访问此url，如果能看到与0x1.4一样的页面，那么部署应该未现异常。  
在Bot Framework Emulator键入新的url，别忘了加上```/api/messages```，验证是否正确。
![yeah](https://raw.githubusercontent.com/windring/Koishumi/markdown/img/Snipaste_2017-11-07_08-56-00.png "")

### 0x5 连接此bot到botframework.com

此页[portal-register-bot](https://docs.microsoft.com/en-us/bot-framework/portal-register-bot "")很好地说明了注册和连接的具体步骤。
 1. 注册  
使用你的microsoft账户登录到[botframework.com](botframework.com "")，此账户不必是你的世纪互联账户。在```My Bot```页面点击```create a bot```。注意，请在选项中选择```Register an existing bot built using Bot Builder SDK```。
![mation](https://raw.githubusercontent.com/windring/Koishumi/markdown/img/Snipaste_2017-11-07_08-58-55.png "")
填写表单，你还可以上传一个卡哇伊的icon。url不是必须此时填写，但是必须使用https。待页面显示完整，点击```生成密码以继续```按钮，记录下MicrosoftAppId和其他重要信息（特别是密码）。将得到的MicrosoftAppId回填到注册页面。```Analytics```项信息可不必填写。完成注册。当显示```Bot created```对话框，bot注册完成。  
 2. 连接  
回到vs项目，编辑```Web.config```文件。找到：
```xml
<appSettings>
<!-- update these with your BotId, Microsoft App Id and your Microsoft App Password-->
<add key="BotId" value="YourBotId" />
<add key="MicrosoftAppId" value="" />
<add key="MicrosoftAppPassword" value="" />
</appSettings>
```  
将刚刚得到的MicrosoftAppId和生成的密码填在对应的value字串中。保存，运行调试，发布。注意，此后连接你的bot都必须提供MicrosoftAppId，MicrosoftAppPassword，比如使用Bot Framework Emulator连接时需要在MicrosoftAppId，MicrosoftAppPassword参数输入框分别填上相应字串。
 3. test  
转到botframework.com中你的bot的控制面板，点击test按钮，在对话框中试试，如果一切正常应该会有如下显示：
![test](https://raw.githubusercontent.com/windring/Koishumi/markdown/img/Snipaste_2017-11-07_12-56-52.png "")


### 0x6 连接到平台

1. 如果你要在skype上和你的bot聊天，在botframework.com的bot的控制面板，点击```Skype```，在弹出的窗口中登录并将此bot添加到你的Skype，即可。
2. 将此bot的web chat放在azure上  
在botframework.com的bot的控制面板，点击```Web Chat```的edit，在新页面中，点击第一行secret key右的show button，使secret key显示，记录下来。拷贝Embed code中的代码。
![set](https://raw.githubusercontent.com/windring/Koishumi/markdown/img/Snipaste_2017-11-07_13-02-57.png "")
打开vs项目，用以下内容替换```default.htm```的全部内容。（为简洁，我将不写出完整的html标签，但其显示效果是一样的）
```html
<style>
body,html,iframe{margin:0;padding:0;width:100%;height:100%;border:0;}
</style>
<iframe src='https://webchat.botframework.com/embed/***?s=YOUR_SECRET_HERE'></iframe>
```
将星号替换成你的Bot handle，或者将之前复制的Embed code替换掉```<iframe>***</iframe>```标签。然后将```YOUR_SECRET_HERE```替换成你的secrect key，保存，运行调试，发布。成功后，访问你的azure web app提供的url（在app概况中，形如```http://appname.chinacloudsites.cn```），应该能看到类似的页面：
![web chat](https://raw.githubusercontent.com/windring/Koishumi/markdown/img/Snipaste_2017-11-07_13-16-31.png "")
事实上，此配置正确的htm/html文件可以在任何地方使用。
3. 连接此bot到微信  
waiting...

### 0x7 使用turing bot的对话服务

1. to get api key  
此页[tuling123](http://www.tuling123.com/help/h_cent_webapi.jhtml?nav=doc "")为图灵机器人的web api说明。请先在[tuling123.com](http://www.tuling123.com/ "")注册，登录后，在页面[robot member](http://www.tuling123.com/member/robot/index.jhtml "")创建一个机器人，机器人类型根据需要选择，此处选择其他。在机器人设置页面，可以看到api入口和```APIKey```，记录此key。 
![api key](https://raw.githubusercontent.com/windring/Koishumi/markdown/img/Snipaste_2017-11-08_14-24-26.png "") 
2. 修改bot
在vs编辑```Dialogs->RootDialog.cs```文件，在命名空间下添加类```TulingAns```作为api返回数据的json解析模板。
```cs
public class TulingAns
{
    public int code;
    public string text;
}
```
3. 添加异步操作```TulingQna```  
在类```RootDialog```添加异步操作```TulingQna```如下：
```cs
 public async Task<string> TulingQna(string Text)
{
    var t_client = new HttpClient();
    var t_queryString = HttpUtility.ParseQueryString(string.Empty);
    HttpResponseMessage t_response;
    byte[] t_byteData = Encoding.UTF8.GetBytes(@"{
                                                    'key':'0c42686*********9e3dd707',
                                                    'info':'" + Text + @"',
                                                    'userid':'" + System.Web.HttpContext.Current.Request.UserHostAddress.ToString() + @"'
                                                  }");
    using (var t_content = new ByteArrayContent(t_byteData))
    {
        t_content.Headers.ContentType = new MediaTypeHeaderValue("application/json");
        try
        {
            t_response = await t_client.PostAsync("http://www.tuling123.com/openapi/api" + t_queryString, t_content);
            var t_json = JsonConvert.DeserializeObject<TulingAns>(await t_response.Content.ReadAsStringAsync());
            return t_json.text;
        }
        catch (Exception e)
        {
            throw new Exception(e.Message);
            return "erro";
        }
    }
}
```
并将key的值修改为刚刚获得的APIKey。
4. 修改异步操作```MessageReceivedAsync```  
当QnA Maker无法找到匹配答案时，会返回```No good match found in the KB```，在此时我们进行```TulingQna操作```。  
在异步操作```MessageReceivedAsync```中找到代码块：
```cs
using (var content = new ByteArrayContent(byteData))
{
 //...
}
```
 修改为：
```cs
using (var content = new ByteArrayContent(byteData))
{
    content.Headers.ContentType = new MediaTypeHeaderValue("application/json");
    response = await client.PostAsync(uri, content);
    var aqnanswer = response.Content.ReadAsAsync<QnAMakerResult>().Result.answers[0].answer;
    if (!aqnanswer.Equals("No good match found in the KB"))
    {
        await context.PostAsync(aqnanswer);
    }
    else
    {
        await context.PostAsync(await TulingQna(activity.Text));
    }
}
```
5. the end  
保存，运行调试，发布。

### 0x08 和你的用户一起和你的bot玩耍吧。

<a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc/4.0/88x31.png" /></a><br /><span xmlns:dct="http://purl.org/dc/terms/" property="dct:title">bot-framework-leader</span> by <span xmlns:cc="http://creativecommons.org/ns#" property="cc:attributionName">windring</span> is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/">Creative Commons Attribution-NonCommercial 4.0 International License</a>.
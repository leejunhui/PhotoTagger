# PhotoTagger
#[译] Alamofire Tutorial: Getting Started

转载请注明出处：[http://leejunhui.com/2017/01/23/Alamofire-Tutorial/](http://leejunhui.com/2017/01/23/Alamofire-Tutorial/)

> 本文翻译自[Raywenderlich](https://www.raywenderlich.com)的[Alamofire Tutorial: Getting Started](https://www.raywenderlich.com/147086/alamofire-tutorial-getting-started-2)

注：已更新到`Alamofire 4`, `Xcode 8.2`, `iOS 10`以及`Swift 3`.

`Alamofire`是一个为`iOS`和`macOS`打造的并基于`Swift`的网络库.它在Apple的基础网络架构上提供了更加优雅的接口来简化繁重而常用的网络请求任务。
`Alamofire`提供了链式的`request`/`response`方法，`JSON`的传参和响应序列化，身份认证和其他特性。在这篇`Alamofire`教程中，你将使用`Alamofire`来执行像从第三方提供的`RESTful`api接口上传文件，请求数据等基本网络操作。`Alamofire`的优雅之处在于它完完全全是由`Swift`写成的，并且没有从它的`Objective-C`版本-`AFNetworking`那继承任何特性。

你应该对于`HTTP`网络有一个概念性的理解，并且还应该接触过Apple的网络类比如`URLSession`.当然`Alamofire`中的一些细节会有点晦涩难懂，如果你有曾经解决过网络请求方面的经验也是极好的。你还需要使用`CocoaPods`来将`Alamofire`集成到项目中。

## Getting Started

下载项目代码 [摸我](https://koenig-media.raywenderlich.com/uploads/2016/12/PhotoTagger-starter-1.zip)

下面介绍的项目名为`PhotoTagger`,当我们完成该项目后，可以实现：从相册中选择一张照片(如果你用的是真机测试的话，可以拍取一张照片)，然后上传这张照片到一个第三方平台，平台会对该照片进行图像识别，然后返回一个tag的列表和这张图片的主色。

![img](https://koenig-media.raywenderlich.com/uploads/2015/11/PhotoTaggerDemo.gif)

编译并运行该项目，你就可以看到如下界面：

![img](https://koenig-media.raywenderlich.com/uploads/2015/11/PhotoTagger-start.png)

点击`Select Photo`，然后选择一张照片，接着界面背景图就会变成你选择的那张照片。打开`Main.stroyboard`,显示tags和colors的界面已经为你准备好了。剩下的工作就是上传照片和获取tags以及colors了。

## The Imagga API 

Imagga是一个为开发者和企业提供构建可伸缩的，以图片为主的云产品的图像识别服务的网站。你可以先尝试下官方的自动标记服务demo:[地址](https://imagga.com/auto-tagging-demo?url=https://imagga.com/static/images/tagging/vegetables.jpg).

你需要再Imagga为本文的项目创建一个免费的开发者账号。因为Imagga对于每个`HTTP`请求都进行了权限认证，所以只有拥有该网站账号的用户才可以使用他们的服务。来到 [https://imagga.com/auth/signup/hacker](https://imagga.com/auth/signup/hacker)，然后注册。完成注册后，检查下是否如下图所示：
![img](https://koenig-media.raywenderlich.com/uploads/2015/12/Imagga-Dashboard.png)

在`Authorization`区域列出了一个等会你将用到的token。该token将会被包含在每个发往Imagga的请求的头部。

> 提示：请确认你拷贝了整个token字符串，确认滑动了最右边并检查是否拷贝完整。

稍后你将会用到Imagga的api来上传图片，`tagging`api实现图像识别，`colors`api用来颜色识别。你可以在[http://docs.imagga.com](http://docs.imagga.com)上查看所有的api。

## Installing Dependencies 安装依赖

在项目主目录里创建`Podfile`文件，内容如下：

    platform :ios, '10.0'
 
    inhibit_all_warnings!
    use_frameworks!
     
    target 'PhotoTagger' do
      pod 'Alamofire', '~> 4.2.0'
    end

然后打开终端，来到项目主目录下，执行`pod install`命令。如果你的Mac上没有安装过`CocoaPods`的话，可以查看[How to Use CocoaPods with Swift](http://www.raywenderlich.com/97014/use-cocoapods-with-swift)教程。

> 确保你使用的是`CocoaPods`的最新版本，否则可能会在安装第三方库时报错。本文撰写时，最新版本为1.1.1。 

关闭Xcode项目，然后打开新生成的`PhotoTagger.xcworkspace`文件。编译然后运行，跑起来的效果应该和之前保持一致。你下一步的任务是添加一些HTTP请求来从RESTful服务获取一些JSON数据。

## REST, HTTP, JSON — 是什么呢?

如果你对使用HTTP不是特别有经验的话，那么你可能会很好奇这些缩写词到底是什么含义。
`HTTP`是一种应用层协议，或者可以理解为一套网站用来从web服务器传输数据到你的电脑屏幕上的规范。你应该看到了你在浏览器中所输入的每个URL前面都会有`HTTP`或`HTTPS`作为前缀。你可能也听过其它的应用层协议，比如`FTP`,`Telnet`和`SSH`。`HTTP`定义了一些客户端(你的浏览器或者app)用来指明所需的行为，请求的方法或者说动作：
`GET`: 获取数据，比如一个WEB页面。但是不更改服务器上的任何内容。
`HEAD`: 与`GET`相同，但是服务器只会返回头部，并不会返回实际的数据。
`POST`: 发送数据到服务器，通常用于表单提交。
`PUT`: 发送数据到指定的路径。
`DELETE`: 删除指定路径的数据。

**REST**, 或者表征状态转移，是用来设计可持续的，易于使用的以及易于维护的WEB API的一套规范。REST有几个体系结构规则，用于强制执行某些操作，例如不在请求之间保持状态，使请求可缓存，并提供统一的接口。这样，像您这样的应用开发者就可以轻松地将API集成到您的应用中，而无需跟踪请求之间的数据状态。
**JSON**代表JavaScript Object Notation; 它提供了一个用于在两个系统之间用于传输数据的直接的，人类可读的和便携的机制。JSON的数据类型有：string，boolean，array，object / dictionary，null和number; 整数和小数之间没有区别。Apple提供`JSONSerialization`类来帮助将内存中的对象转换为JSON，反之亦然。

`HTTP`，`REST`和`JSON`的组合构成了作为开发人员可用的Web服务的很好的一部分。试图理解每个细枝末节如何工作可能是令人难以应对的。 像Alamofire这样的库可以帮助减少使用这些服务的复杂性，并且在没有帮助的情况下让您的运行速度更快。

## Alamofire适合做什么？

你为什么需要`Alamofire`？苹果已经提供了`URLSession`和其他类来通过HTTP获取内容，所以为什么要使用一个第三方库来增加复杂度呢？
简单的答案是`Alamofire`是基于`URLSession`开发而成的，但它可以让你免于编写样板代码，使编写网络代码更容易。你可以花费很少的精力就可以让从网络上获取数据的操作代码变得更加简洁和易于阅读。
`Alamofire`有几个主要功能：

* **.upload**：以multipart，流，文件或数据方法上传文件。
* **.download**：下载文件或恢复正在进行的下载。
* **.request**：每个不与文件传输相关联的HTTP请求。

这些`Alamofire`函数作用于模块，而不是类或结构体。 `Alamofire`的基础部分是类和结构体，如`SessionManager`，`DataRequest`和`DataResponse`; 但是，您不需要完全了解`Alamofire`的整个结构即可开始使用它。
下面是使用Apple的URLSession和Alamofire的请求函数进行相同网络操作的示例：
    
    // With URLSession
    public func fetchAllRooms(completion: @escaping ([RemoteRoom]?) -> Void) {
      let url = URL(string: "http://localhost:5984/rooms/_all_docs?include_docs=true")!
     
      var urlRequest = URLRequest(
        url: url,
        cachePolicy: .reloadIgnoringLocalAndRemoteCacheData,
        timeoutInterval: 10.0 * 1000)
      urlRequest.httpMethod = "GET"
      urlRequest.addValue("application/json", forHTTPHeaderField: "Accept")
     
      let task = urlSession.dataTask(with: urlRequest)
      { (data, response, error) -> Void in
        guard error == nil else {
          print("Error while fetching remote rooms: \(error)")
          completion(nil)
          return
        }
 
    guard let data = data,
      let json = try? JSONSerialization.jsonObject(with: data) as? [String: Any] else {
        print("Nil data received from fetchAllRooms service")
        completion(nil)
        return
    }
 
    guard let rows = json["rows"] as? [[String: Any]] else {
      print("Malformed data received from fetchAllRooms service")
      completion(nil)
      return
    }
 
    let rooms = rows.flatMap({ (roomDict) -> RemoteRoom? in
      return RemoteRoom(jsonData: roomDict)
    })
 
    completion(rooms)
      }
     
      task.resume()
    }

对比:

    // With Alamofire
    func fetchAllRooms(completion: @escaping ([RemoteRoom]?) -> Void) {
      Alamofire.request(
        URL(string: "http://localhost:5984/rooms/_all_docs")!,
        method: .get,
        parameters: ["include_docs": "true"])
      .validate()
      .responseJSON { (response) -> Void in
        guard response.result.isSuccess else {
          print("Error while fetching remote rooms: \(response.result.error)")
          completion(nil)
          return
        }
 
    guard let value = response.result.value as? [String: Any],
      let rows = value["rows"] as? [[String: Any]] else {
        print("Malformed data received from fetchAllRooms service")
        completion(nil)
        return
    }
 
    let rooms = rows.flatMap({ (roomDict) -> RemoteRoom? in
      return RemoteRoom(jsonData: roomDict)
    })
 
    completion(rooms)
      }
    }
    
你可以看到`Alamofire`所需要的设置更加简单，函数可读性也更高。你使用`responseJSON(options:completionHandler:) `来反序列化相应结果，然后调用`validate()`方法来简化错误处理。现在是时候来实践使用`Alamofire`了。

## 上传文件

打开**ViewController.swift**文件，然后在顶部添加以下代码：
    
    import Alamofire

这使你可以在代码中使用`Alamofire`模块提供的功能，然后，在该文件中末尾处添加以下代码：
    
    // Networking calls
    extension ViewController {
      func upload(image: UIImage,
                  progressCompletion: @escaping (_ percent: Float) -> Void,
                  completion: @escaping (_ tags: [String], _ colors: [PhotoColor]) -> Void) {
        guard let imageData = UIImageJPEGRepresentation(image, 0.5) else {
          print("Could not get JPEG representation of UIImage")
          return
        }
      }
    }
    
将图片上传到Imagga的第一步是将图片转换为适用于API的正确格式。上面的图片选择方法会返回一个转化成JPEG格式的图片实例。
然后，来到**imagePickerController(_:didFinishPickingMediaWithInfo:)**方法，然后在你设置imageView的后面添加以下代码：
    
    // 1
    takePictureButton.isHidden = true
    progressView.progress = 0.0
    progressView.isHidden = false
    activityIndicatorView.startAnimating()
     
    upload(
      image: image,
      progressCompletion: { [unowned self] percent in
        // 2
        self.progressView.setProgress(percent, animated: true)
      },
      completion: { [unowned self] tags, colors in
        // 3
        self.takePictureButton.isHidden = false
        self.progressView.isHidden = true
        self.activityIndicatorView.stopAnimating()
 
    self.tags = tags
    self.colors = colors
 
    // 4
    self.performSegue(withIdentifier: "ShowResults", sender: self)
    })
    
`Alamofire`的一切行为都是异步的，这意味着你将以异步的方式更新UI。

1.隐藏上传按钮，并显示进度条和活动视图。
2.当文件上传时，你可以调用进度回调来获取实时的进度百分比，然后可以更新进度条上的进度。
3.当上传完成后将执行完成处理回调，控件的状态也将会恢复到初始状态。
4.最后，在一个成功或者失败的上传后进入结果界面，用户界面不会根据错误情况变化。
    
然后，我们回到**upload(image:progressCompletion:completion:)**方法，然后在转化**UIImage**实例代码后面添加如下代码：

    Alamofire.upload(
      multipartFormData: { multipartFormData in
        multipartFormData.append(imageData,
                                 withName: "imagefile",
                                 fileName: "image.jpg",
                                 mimeType: "image/jpeg")
      },
      to: "http://api.imagga.com/v1/content",
      headers: ["Authorization": "Basic xxx"],
      encodingCompletion: { encodingResult in
      }
    )

请确保从Imagga网站上获取到你自己的`Basic`授权令牌，然后替换掉代码中`Basic xxx`的部分。这里你将JPEG数据转换为MIME multipart请求并发送到Imagga内容服务api。
下一步，添加如下代码：
    
    switch encodingResult {
    case .success(let upload, _, _):
      upload.uploadProgress { progress in
        progressCompletion(Float(progress.fractionCompleted))
      }
      upload.validate()
      upload.responseJSON { response in
      }
    case .failure(let encodingError):
      print(encodingError)
    }

这两段代码通过调用`Alamofire`的`upload`方法，然后随着文件上传，通过一个简单的计算来更新进度条UI。之后验证响应的状态代码是否在默认可接受的范围内(在200和299之间)。

> 注意：在Alamofire 4之前，不能保证总是在主队列上调用进度回调。而从Alamofire 4开始，新的进度回调API则总是在主队列上调用。

接下来，在`upload.responseJSON`中添加以下代码：
    
    // 1.
    guard response.result.isSuccess else {
      print("Error while uploading file: \(response.result.error)")
      completion([String](), [PhotoColor]())
      return
    }
     
    // 2.
    guard let responseJSON = response.result.value as? [String: Any],
      let uploadedFiles = responseJSON["uploaded"] as? [[String: Any]],
      let firstFile = uploadedFiles.first,
      let firstFileID = firstFile["id"] as? String else {
        print("Invalid information received from service")
        completion([String](), [PhotoColor]())
        return
    }
     
    print("Content uploaded with ID: \(firstFileID)")
     
    // 3.
    completion([String](), [PhotoColor]())

下面是上面代码每个步骤的解释：
1.检查响应是否成功，如果失败，打印错误并调用完成回调函数。
2.检查响应的每个部分，并验证预期的类型是否就是接收到的实际类型，然后从响应中检索`firstFileID`，如果`firstFileID`无法解析，则输出错误信息并调用完成回调函数。
3.调用完成回调函数来更新UI。此时，你并没有下载到任何标签和颜色数据，所以只需要传入空数据即可。

> 注意：每个响应都有一个带有值和类型的**Result**枚举。 使用自动验证，当结果返回200到299之间的有效HTTP代码并且内容类型是在**Accept HTTP** header字段中指定的有效类型时，将认为结果成功。
你可以通过如下所示的添加**.validate**参数来使用手动验证响应结果：
>    
       Alamofire.request("https://httpbin.org/get", parameters: ["foo": "bar"])
      .validate(statusCode: 200..<300)
      .validate(contentType: ["application/json"])
      .response { response in
      // response handling code
    }
    
如果你在上传中发生错误的话，UI界面是不会提示错误信息的，它仅仅告诉用户没有颜色和标签返回。这不是最好的用户体验，但是对于这篇教程来说已经足够了。
编译并运行整个项目，选择一张图片然后静观其变。进度条渐进式的前进，你应该可以在控制台看到如下输出：

![img](https://koenig-media.raywenderlich.com/uploads/2015/11/ImaggaUploadConsole.png)

恭喜你，你已经成功的通过网络上传了一张图片了。

![img](https://koenig-media.raywenderlich.com/uploads/2015/11/sad-forever-alone-happy.jpg)

## 获取数据
将图片上传到Imagga之后，下一步则是从Imagga获取到它分析之后的标签数据了。在**ViewController**文件的扩展方法**upload(image:progress:completion:):**下添加以下代码：

    func downloadTags(contentID: String, completion: @escaping ([String]) -> Void) {
      Alamofire.request(
        "http://api.imagga.com/v1/tagging",
        parameters: ["content": contentID],
        headers: ["Authorization": "Basic xxx"]
      )
      .responseJSON { response in
        guard response.result.isSuccess else {
          print("Error while fetching tags: \(response.result.error)")
          completion([String]())
          return
        }
 
    guard let responseJSON = response.result.value as? [String: Any] else {
      print("Invalid tag information received from the service")
      completion([String]())
      return
    }
 
    print(responseJSON)
    completion([String]())
      }
    }

请再一次确认替换的**Basic xxx**是你自己账号的授权token。上述代码执行了一个对标签服务的GET请求，参数是通过上传成功后返回的一个ID。接着，我们回到**upload(image:progress:completion:)**方法，然后替换成功条件下的完成回调方法代码如下：

    self.downloadTags(contentID: firstFileID) { tags in
      completion(tags, [PhotoColor]())
    }
    
上述代码将获得到的标签数据通过完成回调函数返回给上级调用者。
编译然后运行整个项目，上传照片然后控制台会打印如下结果：
![img](https://koenig-media.raywenderlich.com/uploads/2015/11/Imagga-Tagging-Response.png)

本教程里你不用在意返回结果里的confidence score是什么意思，只需要关心标签名称的数组即可。下一步，回到**downloadTags(contentID:completion:)**方法，然后替换里面的**.responseJSON**方法，代码如下：

    // 1.
    guard response.result.isSuccess else {
      print("Error while fetching tags: \(response.result.error)")
      completion([String]())
      return
    }
     
    // 2.
    guard let responseJSON = response.result.value as? [String: Any],
      let results = responseJSON["results"] as? [[String: Any]],
      let firstObject = results.first,
      let tagsAndConfidences = firstObject["tags"] as? [[String: Any]] else {
        print("Invalid tag information received from the service")
        completion([String]())
        return
    }
     
    // 3.
    let tags = tagsAndConfidences.flatMap({ dict in
      return dict["tag"] as? String
    })
     
    // 4.
    completion(tags)

我们来逐步分解上面的代码：
1.检查返回结果是否成功，如果失败，打印错误信息，然后调用完成回调函数。
2.检查返回结果的每个部分，验证数据类型是否正确，从返回结果中获取**tagsAndConfidences**，如果解析失败，打印错误信息，然后调用完成回调函数。
3.遍历**tagsAndConfidences**数组里的每个字典对象，从中以`tag`作为key来提取对应的值。
4.将从服务器获取的标签数据返回给上层调用者。

> 注意：代码里使用Swift的`flatMap`方法来遍历数组里面的每个字典。这个方法在遍历到nil的值得时候并不会让程序crash掉，而是直接将这些nil值移除掉，然后只返回正确的结果。所以通过使用可选解包(as?)来验证字典中的值是否可以被转化成为字符串。 
编译然后运行整个项目，上传一张图片，然后你可以在界面上看到如下效果：

![img](https://koenig-media.raywenderlich.com/uploads/2015/11/PhotoTagger-tags.png)

纵享丝滑，Imagga不愧是一个智能的api。下一步，你将要获取的是图片的颜色。添加如下代码到**ViewController**中的**downloadTags(contentID:completion:):**方法下面：

     func downloadColors(contentID: String, completion: @escaping ([PhotoColor]) -> Void) {
      Alamofire.request(
        "http://api.imagga.com/v1/colors",
        parameters: ["content": contentID],
        // 1.
        headers: ["Authorization": "Basic xxx"]
      )
      .responseJSON { response in
        // 2.
        guard response.result.isSuccess else {
          print("Error while fetching colors: \(response.result.error)")
          completion([PhotoColor]())
          return
        }
 
    // 3.
    guard let responseJSON = response.result.value as? [String: Any],
      let results = responseJSON["results"] as? [[String: Any]],
      let firstResult = results.first,
      let info = firstResult["info"] as? [String: Any],
      let imageColors = info["image_colors"] as? [[String: Any]] else {
        print("Invalid color information received from service")
        completion([PhotoColor]())
        return
    }
 
    // 4.
    let photoColors = imageColors.flatMap({ (dict) -> PhotoColor? in
      guard let r = dict["r"] as? String,
        let g = dict["g"] as? String,
        let b = dict["b"] as? String,
        let closestPaletteColor = dict["closest_palette_color"] as? String else {
          return nil
      }
 
      return PhotoColor(red: Int(r),
                        green: Int(g),
                        blue: Int(b),
                        colorName: closestPaletteColor)
    })
 
    // 5.
    completion(photoColors)
      }
    }
    
下面按照编号来依次解析：
1.确保**Basic xxx**是你自己账号所属的认证token。
2.检查返回结果是否成功，如果失败，打印错误信息，然后调用完成回调函数。
3.检查返回结果的每个部分，验证数据类型是否正确。从返回结果中获取**imageColors**，如果解析失败，打印错误信息，然后调用完成回调函数。
4.再次使用`flatMap`方法，对服务器返回的`PhotoColors`对象进行遍历，将里面的符合RGB格式的数据转换字符串，然后封装成`PhotoColor`对象。
5.调用完成回调函数，传入服务器返回的图片颜色数据。

最后，回到**upload(image:progress:completion:)**方法，然后替换掉成功条件下的调用完成回调函数：

    self.downloadTags(contentID: firstFileID) { tags in
      self.downloadColors(contentID: firstFileID) { colors in
        completion(tags, colors)
      }
    }

上述代码嵌套了上传图片，获取标签以及获取颜色的操作。
编译然后运行整个项目，这一次当你选择Colors按钮后，界面会有以下效果：

![img](https://koenig-media.raywenderlich.com/uploads/2015/11/PhotoTagger-colors.png)

这块主要使用了映射到**PhotoColor**结构体的RGB颜色来渲染UI。你已经成功的往Imagga上传了一张图片，以及调用了2个不同的api来获取数据。你已经做得很不错了，但是在如何使用`Alamofire`上，我们还有改进的余地。

## 改进 PhotoTagger

你应该注意到了我们前面的代码中有许多重复代码。如果Imagga官方宣布废除掉v1版本的api，并推出v2版本的api。我们的应用将不再可用直到你将每个方法里面的URL都修改过来。同样的道理，如果你的Basic认证token发生了变化，那么你需要在所有用到的地方做出修改。
`Alamofire`提供了一个简单的方法来消除代码重复的问题并提供了集中式的配置方法。该技术涉及创建符合`URLRequestConvertible`协议的结构体，然后更新的上传和请求的方法。
创建一个新的`Swfit`文件，命名为`ImaggaRouter.swift`，然后在文件里替换成以下代码：

    import Foundation
    import Alamofire
     
    public enum ImaggaRouter: URLRequestConvertible {
      static let baseURLPath = "http://api.imagga.com/v1"
      static let authenticationToken = "Basic xxx"
     
      case content
      case tags(String)
      case colors(String)
     
      var method: HTTPMethod {
        switch self {
        case .content:
          return .post
        case .tags, .colors:
          return .get
        }
      }
     
      var path: String {
        switch self {
        case .content:
          return "/content"
        case .tags:
          return "/tagging"
        case .colors:
          return "/colors"
        }
      }
 
      public func asURLRequest() throws -> URLRequest {
        let parameters: [String: Any] = {
          switch self {
          case .tags(let contentID):
            return ["content": contentID]
          case .colors(let contentID):
            return ["content": contentID, "extract_object_colors": 0]
          default:
            return [:]
          }
        }()
 
    let url = try ImaggaRouter.baseURLPath.asURL()
 
    var request = URLRequest(url: url.appendingPathComponent(path))
    request.httpMethod = method.rawValue
    request.setValue(ImaggaRouter.authenticationToken, forHTTPHeaderField: "Authorization")
    request.timeoutInterval = TimeInterval(10 * 1000)
 
    return try URLEncoding.default.encode(request, with: parameters)
      }
    }

替换**Basic xxx**为你自己账号的认证Token。这个路由类通过提供三个不同的分类：`.content`, `.tags(String)`,`.colors(String)`来实现创建多个`URLRequest`实例。现在你的重复代码都集中到了一个地方，如果有需要在这里更改即可。
回到**ViewController.swift**文件，然后替换**upload(image:progress:completion:)**方法：
    
    Alamofire.upload(
      multipartFormData: { multipartFormData in
        multipartFormData.append(imageData,
                                 withName: "imagefile",
                                 fileName: "image.jpg",
                                 mimeType: "image/jpeg")
      },
      to: "http://api.imagga.com/v1/content",
      headers: ["Authorization": "Basic xxx"],

替换为：

    Alamofire.upload(
      multipartFormData: { multipartFormData in
      multipartFormData.append(imageData,
                               withName: "imagefile",
                               fileName: "image.jpg",
                               mimeType: "image/jpeg")
      },
      with: ImaggaRouter.content,

然后替换掉**downloadTags(contentID:completion:)**方法里的代码：
    
    Alamofire.request(ImaggaRouter.tags(contentID))
最后，替换掉**downloadColors(contentID:completion:)**方法里的代码：  

    Alamofire.request(ImaggaRouter.colors(contentID))
编译然后运行项目，结果应与之前的效果保持一致。这意味着你已经在不破坏你的app的前提下完成了重构，干得漂亮！

## 接下来做什么？

所有的代码文件都上传到了[github](https://github.com/LeeJunhui/PhotoTagger)上，不要忘了替换你自己的Basic认证token。
这篇教程只涵盖到了非常基础的知识点。你可以去`Alamofire`的官方网站[https://github.com/Alamofire/Alamofire](https://github.com/Alamofire/Alamofire)上进行更深入的学习。
进一步的话，你可以花一些时间来学习`Alamofire`底层使用的`Apple`的`URLSession`类的内容：

* [Apple WWDC 2015 – 711 – Networking with NSURLSession](https://developer.apple.com/videos/play/wwdc2015-711/)
* [Apple URL Session Programming Guide](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/URLLoadingSystem/URLLoadingSystem.html)
* [Ray Wenderlich – NSURLSession Tutorial](http://www.raywenderlich.com/51127/nsurlsession-tutorial)



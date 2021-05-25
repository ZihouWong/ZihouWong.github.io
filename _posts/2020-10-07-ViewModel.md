---

title: "iOS 开发中的 ViewModel"
date: 2020-10-07
categories: 

layout: post
<!--comments: true-->

---


# 
[image:E95D6D20-D033-4BE9-AEAE-CB8995A22C9B-7209-0000180234BA7959/626506-e9f0ec4fc8bfe938.png]
MVVM 这个模式可能大家耳朵都听出茧了，但却没有多少人真正在项目中应用过，毕竟 Cocoa Touch 整体是基于“MVC”的，没有 Controller 根本干不了活。而且这年头虽然各种 buzz word 盛嚣尘上，但不同领域的人对它们都有不同的理解，看多了说多了自己都有点烦。我今天也不想说到底什么是 MVC，什么是 MVVM，这些我之前在这篇 [文章](https://www.jianshu.com/p/2c3a0f109788) 有提过一点。如果大家真的对它们的原始定义感兴趣，可以看看这篇 [MVC 论文](https://link.jianshu.com?t=https://heim.ifi.uio.no/~trygver/themes/mvc/mvc-index.html) ，四人帮的《设计模式》也对 MVC 有所阐述；MVVM 的话我们知道它是微软在推出 WPF 时提出的，可以看看 [MSDN 的相关博客](https://link.jianshu.com?t=https://blogs.msdn.microsoft.com/johngossman/2005/10/) 。

言归正传，今天我主要想谈谈自己对 ViewModel 的一些理解。因为我们不一定要完全照搬某种模式，取其精华然后根据具体项目情况进行应用也挺好的。ViewModel 这个概念我就觉得蛮精华的。我们都知道 ViewModel 跟 View 是绑定的关系，那究竟怎么绑定呢？有几种方案：

* UI 布局尽量用 IB 来做，把绑定逻辑放到 View 中
* 把绑定逻辑放到 Model 中
* 定义单独的 ViewModel 加工 Model，并把适合展示的数据输出给 View

以上这几种方案主要说的是数据绑定，而且都是单向绑定，实际上 ViewModel 还可以跟 View 进行双向数据绑定、逻辑绑定等，这些先按下不表，下面举个例子分别说说这三种单向数据绑定的实现以及优缺点。

Profile.png

上图是我正在写的一个 Github 客户端的 Profile 页面，它的头部是一个单独的 View，包含了一些 UI 元素：

```
class ProfileHeader: UIView {

    @IBOutlet var contentView: UIView!
    @IBOutlet weak var backgroundView: UIImageView!
    @IBOutlet weak var avatarView: UIImageView!
    @IBOutlet weak var followersButton: UIButton!
    @IBOutlet weak var repositoriesButton: UIButton!
    @IBOutlet weak var followingButton: UIButton!
    @IBOutlet weak var nicknameLabel: UILabel!
    @IBOutlet weak var bioLabel: UILabel!

    // ...
}
```

通过网络请求拿到相关数据之后，怎么传递给这些 UI 元素来显示呢？当然我们可以放到 Controller 中，只不过 Controller 会变得非常臃肿。那我们就来尝试以上三种方案。

### 方案一：View 作为 ViewModel

```
protocol ViewModelType {
    associatedtype Model
    func bind(model: Model)
}

extension ProfileHeader: ViewModelType {
    func bind(model: Profile) {
        let avatarURL = NSURL(string: model.avatar)
        backgroundView.presentImageAnimated(avatarURL)
        avatarView.presentImageAnimated(avatarURL)

        nicknameLabel.text = model.login

        bioLabel.text = model.bio ?? "Not set yet"

        let attributedFollowers = (title: "\(model.followers) \nFollowers", attributingTitle: "Followers")
        let attributedRepositories = (title: "\(model.repos) \nRepositories", attributingTitle: "Repositories")
        let attributedFollowing = ("\(model.following) \nFollowing", attributingTitle: "Following")
        followersButton.setAttributedTitle(attributedFollowers)
        repositoriesButton.setAttributedTitle(attributedRepositories)
        followingButton.setAttributedTitle(attributedFollowing)
    }
}
```

这个是我以前比较喜欢的做法，优点是简洁明了，没有太多弯弯绕绕的东西，基本就是把原本写在 Controller 中的代码放到了 View 中。缺点也非常明显，就是这个“绑定”实在是绑得太死了，直接就写在了 View 里，那这个 View 基本就不能复用了。假设在另一个页面也有一个 Header，UI 跟这个 `ProfileHeader` 一样，展示的数据也是来自 `Profile` 这个 Model，但是展示的方式不一样，譬如 `nicknameLabel` 需要显示 Nickname: XXX，其它 UI 元素也需要加点附加信息什么的，那这个时候就很尴尬了。只能新建一个 `NewHeader` ，把之前的 xib 文件 copy 过来，把 class 改成 `NewHeader` ，把各个 label 啊 button 啊拉线拉过来，然后写一个新的 `bind` 方法。如果 `ProfileHeader` 中有很多其它的辅助方法，在 `NewHeader` 中也要用到，那 `NewHeader` 就得继承 `ProfileHeader` ，然后重写 `bind` 方法……所以这种方案啊，是不太科学的……想必你也发现了，这种情况下， `ViewModelType` 这个协议，其实并没有什么卵用。用协议作为类型，往往可以提供更大的灵活性和可扩展性，但是如果是由 View 来实现这个协议，由于 View 已经是数据流的终点了，一旦把处理数据的逻辑写在这里，就不存在什么替换的可能了，这个协议也就只是作为一个限制或者说标识了——一旦别人看到某个 View 实现了这个协议，就知道这个 View 内部有处理数据的逻辑。但按这个思路的话，其实基本每个 View 都需要处理数据，也就都需要实现这个协议。所以这个协议其实是可有可无的，它的存在除了让代码显得更“面向协议（POP）”一些，并没有带来太多实质性的好处。

### 方案二：Model 作为 ViewModel

这种方案我在一个 [演讲](https://link.jianshu.com?t=http://www.imooc.com/video/11109) 中看到过，思路也很简单，跟方案一恰恰相反，不是把 Model 注入 View 中，而是把 View 注入 Model 中，还是以 Profile 为例：

```
typealias AttributedTitle = (title: String, attributingTitle: String)

private let attributingFollowers = "Followers"
private let attributingRepos = "Repositories"
private let attributingFollowing = "Following"

protocol RenderContext {
    func renderText(texts: String...)
    func renderAttributedText(attributedTexts: AttributedTitle...)
    func renderImage(imageURLs: NSURL...)
}

protocol ViewModelType {
    func renderInContext(context: RenderContext)
}

extension ProfileHeader: RenderContext {
    func renderText(texts: String...) {
        nicknameLabel.text = texts.first
        bioLabel.text = texts.last
    }

    func renderAttributedText(attributedTexts: AttributedTitle...) {
        followersButton.setAttributedTitle(attributedTexts[0])
        repositoriesButton.setAttributedTitle(attributedTexts[1])
        followingButton.setAttributedTitle(attributedTexts[2])
    }

    func renderImage(imageURLs: NSURL...) {
        backgroundView.presentImageAnimated(imageURLs.first)
        avatarView.presentImageAnimated(imageURLs.first)
    }
}

struct Profile {
    let login: String
    let id: Int
    let avatar: String
    let name: String
    let bio: String?
    let company: String?
    let blog: String
    let location: String
    let email: String?
    let repos: Int
    let followers: Int
    let following: Int
}

extension Profile: ViewModelType {
    func renderInContext(context: RenderContext) {
        context.renderText(login, bio ?? "Not set yet")

        let attributedFollowers = (title: "\(followers) \n\(attributingFollowers)", attributingTitle: attributingFollowers)
        let attributedRepositories = (title: "\(repos) \n\(attributingRepos)", attributingTitle: attributingRepos)
        let attributedFollowing = (title: "\(following) \n\(attributingFollowing)", attributingTitle: attributingFollowing)
        context.renderAttributedText(attributedFollowers, attributedRepositories, attributedFollowing)

        let avatarURL = NSURL(string: avatar) ?? defaultImageURL
        context.renderImage(avatarURL)
    }
}
```

这个方案呢，看着花哨了一些，View 的复用性也提高了。但是它的局限其实跟方案一是差不多的，方案一是可以复用 Model，但 View 的复用性不高；而方案二恰恰相反，View 的复用性提高了，但 Model 不好复用……数据加工的逻辑写在 Model 的 `renderInContext` 方法中，一旦有业务场景需要不同的数据加工逻辑，就要新建一个 Model 或者继承 `Profile` 。而众所周知继承在 Swift 中是不被提倡的，我这边声明的 `Profile` 是个 `struct` ，是不能被继承的，所以这种方案也并不是最合适的方案。

### 方案三：定义单独的 ViewModel 加工 Model，并把适合展示的数据输出给 View

软件设计有个重要的原则——封装变化，分析前面两个方案的局限性，我们可以明确的知道，“数据加工”是一个比较容易变化的点，而 View 和 Model 相对来说要稳定一些。那我们就把“数据加工”这个逻辑单独封装起来，把变化的部分和不变的部分隔离，这样 View 和 Model 的复用性就都提高了。

```
protocol ViewModelType {
    var avatarURL: NSURL { get }
    var followers: AttributedTitle { get }
    var repositories: AttributedTitle { get }
    var following: AttributedTitle { get }
    var nickname: String { get }
    var bio: String { get }
}

struct ProfileHeaderViewModel: ViewModelType {

    // MARK: Output
    let avatarURL: NSURL
    let followers: AttributedTitle
    let repositories: AttributedTitle
    let following: AttributedTitle
    let nickname: String
    let bio: String

    init(input: Profile) {
        avatarURL = NSURL(string: input.avatar) ?? defaultImageURL

        nickname = input.login

        bio = input.bio ?? defailtProfileItem

        followers = (title: "\(input.followers) \n\(attributingFollowers)", attributingTitle: attributingFollowers)

        repositories = (title: "\(input.repos) \n\(attributingRepos)", attributingTitle: attributingRepos)

        following = (title: "\(input.following) \n\(attributingFollowing)", attributingTitle: attributingFollowing)
    }
}
```

这个 ViewModel 以一个 Model 为输入，以一些可以直接被 View 使用的数据为输出。然后我们把它注入到 View 中即可，注入的方式无所谓，无论是作为初始化参数，抑或是作为属性或者方法参数等等，都可以，只要它是能被外部注入的，而不是由 View 自己生成的即可。譬如把它作为属性：

```
var viewModel: ViewModelType! {
    didSet {
        nicknameLabel.text = viewModel.nickname

        configAvatar(viewModel.avatarURL)

        bioLabel.text = viewModel.bio

        configButtons(viewModel.followers, viewModel.repositories, viewModel.following)
    }
}
```

这样我们就可以声明多个 ViewModel，它们都遵守 ViewModelType，要展示不同数据的时候，只要赋给 `ProfileHeader` 不同的 ViewModel 就行了。

### 扩展思考

其实 View 是可以非常通用的，譬如 `ProfileHeaderView` 这个 View 不一定是用在 Profile 模块中，也可以用在 Repository 模块中（只是举个例子，一般 Repository 模块不会用这样的 Header 吧……）。那中间的 `avatarView` 就不是用来显示头像，而是显示项目的 Logo， `backgroundView` 也显示 Logo， `nicknameLabel` 用来显示项目名， `bioLabel` 显示项目描述等等，这个时候你就会发现 `ProfileHeaderView` 包括内部 UI 元素的那些命名都是不合适，跟具体业务耦合得太紧了。如果这个 Header 是到处都可能用到的，那命名就应该是 `CustomHeaderView`、`centerImageView`、`topLabel` 之类的跟业务无关的形式。这个时候对应的 ViewModel 只要保证输出是直接可以被使用数据即可，输入并不一定要是 `Profile` ，也可以是 `Repository` ，甚至不一定要是 Model，也可以是 Dictionary 、JSON 等等。但是越通用，往往可读性就越低，很显然 `nicknameLabel` 比 `topLabel` 要好懂得多；越通用，往往中间层就越多，模块间的关系就越不直观。所以在 **封装变化** 的时候，要 **确保抽象出来封装的内容是真的有可能变化的** ，否则就成了过度设计。譬如 ViewModel 这个东西，如果你的 View 是一个高度定制化的 View，几乎没有被复用的可能，那在命名的时候，大可以跟业务相关，数据处理也可以采用方案一，因为这是最容易理解的方式，也是最方便开发的方式。这世上很少有放之四海皆准的道理，还是要结合具体情况多思考权衡吧。

### RxSwift ＋ MVVM

由于 Cocoa Touch 本身并没有一个统一的数据绑定机制，MVVM 几乎是随着 RAC 这个 FRP 框架走近 iOS 开发者的视线的。利用 RAC 可以更优雅地实现数据绑定和 MVVM 模式。RxSwift 同样是个 FPR 框架，用它来实现 ViewModel 大概是这样的：

```
protocol ViewModelType {
    var avatarURL: Driver<NSURL> { get }
    var followers: Driver<AttributedTitle> { get }
    var repositories: Driver<AttributedTitle> { get }
    var following: Driver<AttributedTitle> { get }
    var nickname: Driver<String> { get }
    var bio: Driver<String> { get }
}

struct ProfileHeaderViewModel: ViewModelType {

    // MARK: Output
    let avatarURL: Driver<NSURL>
    let followers: Driver<AttributedTitle>
    let repositories: Driver<AttributedTitle>
    let following: Driver<AttributedTitle>
    let nickname: Driver<String>
    let bio: Driver<String>

    init(input: Profile) {
        let profileDriver = Driver.of(input)
        avatarURL = profileDriver
            .map { $0.avatar }
            .map { NSURL(string: $0) ?? defaultImageURL }

        followers = profileDriver
            .map { (title: "\($0.followers) \n\(attributingFollowers)", attributingTitle: attributingFollowers) }

        repositories = profileDriver
            .map { (title: "\($0.repos) \n\(attributingRepos)", attributingTitle: attributingRepos) }

        following = profileDriver
            .map { (title: "\($0.following) \n\(attributingFollowing)", attributingTitle: attributingFollowing) }

        nickname = profileDriver
            .map { $0.login }

        bio = profileDriver
            .map { $0.bio ?? defailtProfileItem }
    }
}

class ProfileHeader: UIView {
    // ...

    private let bag = DisposeBag()
    var viewModel: ViewModelType! {
        didSet {
            viewModel.nickname
                .drive(nicknameLabel.rx_text)
                .addDisposableTo(bag)

            viewModel.avatarURL
                .drive(onNext: configAvatar)
                .addDisposableTo(bag)

            viewModel.bio
                .drive(bioLabel.rx_text)
                .addDisposableTo(bag)

            Driver.zip(viewModel.followers, viewModel.repositories, viewModel.following) { ($0.0, $0.1, $0.2) }
                .drive(onNext: configButtons)
                .addDisposableTo(bag)
        }
    }

    // ...
}
```

FRP 其实比较适合业务复杂型的项目，在我这个简单的例子中表现并不比方案三中普通的 ViewModel 更好。但是 RxSwift 是很好玩的，推荐大家就算正式项目中用不到，私下也可以自己玩玩儿～

[iOS 开发中的 ViewModel](https://www.jianshu.com/p/823297d8c386)
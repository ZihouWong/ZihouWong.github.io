---

title: "iOS-Loading/Downloading image from URL on Swift"
date: 2019-08-21
categories: IOs

comments: true

---

-.md

# iOS - Loading/Downloading image from URL on Swift - Stack Overflow
**Synchronously:** 

```
if let filePath = Bundle.main.path(forResource: "imageName", ofType: "jpg"), let image = UIImage(contentsOfFile: filePath) {
    imageView.contentMode = .scaleAspectFit
    imageView.image = image
}
```

**Asynchronously:** 

**Create a method with a completion handler to get the image data from your url**

```
func getData(from url: URL, completion: @escaping (Data?, URLResponse?, Error?) -> ()) {
    URLSession.shared.dataTask(with: url, completionHandler: completion).resume()
}
```

**Create a method to download the image (start the task)**

```
func downloadImage(from url: URL) {
    print("Download Started")
    getData(from: url) { data, response, error in
        guard let data = data, error == nil else { return }
        print(response?.suggestedFilename ?? url.lastPathComponent)
        print("Download Finished")
        DispatchQueue.main.async() {
            self.imageView.image = UIImage(data: data)
        }
    }
}
```

**Usage:**

```
override func viewDidLoad() {
    super.viewDidLoad()
    // Do any additional setup after loading the view, typically from a nib.
    print("Begin of code")
    let url = URL(string: "https://cdn.arstechnica.net/wp-content/uploads/2018/06/macOS-Mojave-Dynamic-Wallpaper-transition.jpg")! 
    downloadImage(from: url)
    print("End of code. The image will continue downloading in the background and it will be loaded when it ends.")
}
```

---

**Extension**:

[ios - Loading/Downloading image from URL on Swift - Stack Overflow](https://stackoverflow.com/questions/24231680/loading-downloading-image-from-url-on-swift)
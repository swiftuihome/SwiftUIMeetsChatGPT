# 使用SwiftUI可以读取外部文件目录下的图片吗？

是的，SwiftUI 可以读取外部文件目录下的图片。你可以使用 `FileManager` 来访问文件系统，并将图片加载到 SwiftUI 视图中。以下是一个简单的示例，演示如何实现这一功能：

1. 首先，确保你的应用具有访问外部文件目录的权限。你可能需要在应用的 `Info.plist` 文件中添加相应的权限声明。

2. 使用 `FileManager` 读取文件目录下的图片，并将其转换为 SwiftUI 可以使用的格式。

以下是一个示例代码，展示了如何在 SwiftUI 中读取和显示外部文件目录下的图片：

```swift
import SwiftUI

struct ContentView: View {
    @State private var image: Image? = nil
    
    var body: some View {
        VStack {
            if let image = image {
                image
                    .resizable()
                    .scaledToFit()
            } else {
                Text("未找到图片")
            }
        }
        .onAppear(perform: loadImage)
    }
    
    func loadImage() {
        let fileManager = FileManager.default
        // 设置图片的路径，这里假设图片位于应用的 Documents 目录下
        let documentsDirectory = fileManager.urls(for: .documentDirectory, in: .userDomainMask).first!
        let fileURL = documentsDirectory.appendingPathComponent("example.jpg")
        
        if let imageData = try? Data(contentsOf: fileURL), let uiImage = UIImage(data: imageData) {
            image = Image(uiImage: uiImage)
        }
    }
}

struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}
```

在这个示例中：

1. `loadImage()` 函数使用 `FileManager` 来访问应用的 `Documents` 目录，并尝试加载名为 `example.jpg` 的图片。
2. 如果成功加载图片数据，并将其转换为 `UIImage`，则将其包装为 SwiftUI 的 `Image` 并显示在视图中。

请确保图片文件的路径正确，并且应用具有读取该路径下文件的权限。这样，你就可以在 SwiftUI 中读取并显示外部文件目录下的图片了。

# 可以读取手机下载目录中的图片吗？

在 iOS 中，由于沙盒安全机制，应用程序无法直接访问其他应用的文件，例如下载目录中的文件。iOS 应用程序只能访问自己的沙盒目录中的文件，包括应用的 Documents、Library 和 tmp 目录。如果你希望用户能够访问和选择设备中的图片（包括下载目录中的图片），通常使用 `UIImagePickerController` 或 `PHPickerViewController` 来实现。

以下是使用 `PHPickerViewController` 的一个示例代码，该控制器允许用户从设备的相册中选择图片，包括下载目录中的图片：

```swift
import SwiftUI
import PhotosUI

struct ContentView: View {
    @State private var selectedImage: UIImage? = nil
    @State private var isPickerPresented: Bool = false
    
    var body: some View {
        VStack {
            if let image = selectedImage {
                Image(uiImage: image)
                    .resizable()
                    .scaledToFit()
            } else {
                Text("选择一张图片")
            }
            
            Button("选择图片") {
                isPickerPresented = true
            }
            .sheet(isPresented: $isPickerPresented) {
                PhotoPicker(selectedImage: $selectedImage)
            }
        }
    }
}

struct PhotoPicker: UIViewControllerRepresentable {
    @Binding var selectedImage: UIImage?
    
    func makeUIViewController(context: Context) -> PHPickerViewController {
        var config = PHPickerConfiguration()
        config.filter = .images
        let picker = PHPickerViewController(configuration: config)
        picker.delegate = context.coordinator
        return picker
    }
    
    func updateUIViewController(_ uiViewController: PHPickerViewController, context: Context) {}
    
    func makeCoordinator() -> Coordinator {
        Coordinator(self)
    }
    
    class Coordinator: NSObject, PHPickerViewControllerDelegate {
        var parent: PhotoPicker
        
        init(_ parent: PhotoPicker) {
            self.parent = parent
        }
        
        func picker(_ picker: PHPickerViewController, didFinishPicking results: [PHPickerResult]) {
            picker.dismiss(animated: true)
            
            guard let provider = results.first?.itemProvider else { return }
            
            if provider.canLoadObject(ofClass: UIImage.self) {
                provider.loadObject(ofClass: UIImage.self) { (image, error) in
                    DispatchQueue.main.async {
                        self.parent.selectedImage = image as? UIImage
                    }
                }
            }
        }
    }
}

struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}
```

在这个示例中：

1. `ContentView` 包含一个按钮，当用户点击按钮时，会显示一个图片选择器。
2. `PhotoPicker` 使用 `PHPickerViewController` 来让用户选择图片。
3. `Coordinator` 类实现了 `PHPickerViewControllerDelegate` 协议，处理图片选择完成后的操作。

这样，你的应用就可以让用户选择和加载设备中的图片，包括下载目录中的图片。

# 我的意思不是访问用户手机相册中的图片，而是访问下载目录中的图片

在 iOS 上，应用程序无法直接访问下载目录中的文件，这是由于 iOS 的沙盒安全机制所决定的。每个应用都有自己的沙盒文件系统，不能访问其他应用的文件，包括下载目录中的文件。

如果你需要访问用户下载的文件，可以使用以下两种方法：

1. **通过文件共享功能**：用户可以通过 iTunes 文件共享或者通过 Files 应用将文件共享到你的应用程序的沙盒中。然后你的应用可以访问这些共享的文件。

2. **通过 Document Picker**：使用 `UIDocumentPickerViewController` 让用户选择文件。这种方法允许用户选择任何他们有权限访问的文件，包括下载目录中的文件。选中的文件会被拷贝到你的应用的沙盒中，供你的应用使用。

以下是使用 `UIDocumentPickerViewController` 的示例代码：

```swift
import SwiftUI
import UIKit

struct ContentView: View {
    @State private var selectedImage: UIImage? = nil
    @State private var isPickerPresented: Bool = false
    
    var body: some View {
        VStack {
            if let image = selectedImage {
                Image(uiImage: image)
                    .resizable()
                    .scaledToFit()
            } else {
                Text("选择一张图片")
            }
            
            Button("选择图片") {
                isPickerPresented = true
            }
            .sheet(isPresented: $isPickerPresented) {
                DocumentPicker(selectedImage: $selectedImage)
            }
        }
    }
}

struct DocumentPicker: UIViewControllerRepresentable {
    @Binding var selectedImage: UIImage?
    
    func makeUIViewController(context: Context) -> UIDocumentPickerViewController {
        let documentPicker = UIDocumentPickerViewController(forOpeningContentTypes: [.image], asCopy: true)
        documentPicker.delegate = context.coordinator
        return documentPicker
    }
    
    func updateUIViewController(_ uiViewController: UIDocumentPickerViewController, context: Context) {}
    
    func makeCoordinator() -> Coordinator {
        Coordinator(self)
    }
    
    class Coordinator: NSObject, UIDocumentPickerDelegate {
        var parent: DocumentPicker
        
        init(_ parent: DocumentPicker) {
            self.parent = parent
        }
        
        func documentPicker(_ controller: UIDocumentPickerViewController, didPickDocumentsAt urls: [URL]) {
            guard let url = urls.first else { return }
            if let data = try? Data(contentsOf: url), let uiImage = UIImage(data: data) {
                DispatchQueue.main.async {
                    self.parent.selectedImage = uiImage
                }
            }
        }
    }
}

struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}
```

在这个示例中：

1. `ContentView` 包含一个按钮，点击按钮后会显示 `DocumentPicker`。
2. `DocumentPicker` 使用 `UIDocumentPickerViewController` 让用户选择图片文件。
3. `Coordinator` 类实现了 `UIDocumentPickerDelegate` 协议，处理文件选择完成后的操作。

这样，用户可以通过文件选择器访问下载目录中的文件，并将其导入到你的应用程序中。

# 可以同时选择多张图片并显示吗

可以使用 `UIDocumentPickerViewController` 同时选择多张图片并显示。以下是如何在 SwiftUI 中实现这个功能的示例代码：

```swift
import SwiftUI
import UIKit

struct ContentView: View {
    @State private var selectedImages: [UIImage] = []
    @State private var isPickerPresented: Bool = false
    
    var body: some View {
        VStack {
            ScrollView(.horizontal) {
                HStack {
                    ForEach(selectedImages, id: \.self) { image in
                        Image(uiImage: image)
                            .resizable()
                            .scaledToFit()
                            .frame(width: 100, height: 100)
                            .clipped()
                    }
                }
            }
            
            Button("选择图片") {
                isPickerPresented = true
            }
            .sheet(isPresented: $isPickerPresented) {
                DocumentPicker(selectedImages: $selectedImages)
            }
        }
    }
}

struct DocumentPicker: UIViewControllerRepresentable {
    @Binding var selectedImages: [UIImage]
    
    func makeUIViewController(context: Context) -> UIDocumentPickerViewController {
        let documentPicker = UIDocumentPickerViewController(forOpeningContentTypes: [.image], asCopy: true)
        documentPicker.allowsMultipleSelection = true
        documentPicker.delegate = context.coordinator
        return documentPicker
    }
    
    func updateUIViewController(_ uiViewController: UIDocumentPickerViewController, context: Context) {}
    
    func makeCoordinator() -> Coordinator {
        Coordinator(self)
    }
    
    class Coordinator: NSObject, UIDocumentPickerDelegate {
        var parent: DocumentPicker
        
        init(_ parent: DocumentPicker) {
            self.parent = parent
        }
        
        func documentPicker(_ controller: UIDocumentPickerViewController, didPickDocumentsAt urls: [URL]) {
            for url in urls {
                if let data = try? Data(contentsOf: url), let uiImage = UIImage(data: data) {
                    DispatchQueue.main.async {
                        self.parent.selectedImages.append(uiImage)
                    }
                }
            }
        }
    }
}

struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}
```

在这个示例中：

1. `ContentView` 包含一个 `ScrollView` 来显示选择的图片，并有一个按钮来打开图片选择器。
2. `DocumentPicker` 使用 `UIDocumentPickerViewController` 来让用户选择图片，并允许多选（通过 `allowsMultipleSelection = true` 设置）。
3. `Coordinator` 类实现了 `UIDocumentPickerDelegate` 协议，处理选择完成后的操作，将选中的图片添加到 `selectedImages` 数组中。

这样，用户可以通过文件选择器选择多张图片，并在 SwiftUI 界面上显示这些图片。

# 非常棒，这些代码都能正确读取并显示图片

很高兴这些代码能帮到你！如果你有任何其他问题或需要进一步的帮助，随时告诉我。祝你在开发“星标相册”应用的过程中一切顺利！

# 当用户选择了图片并显示在页面上时，此时图片会被拷贝到应用程序的目录下吗？我不希望做这样的拷贝动作。因为我只是想在页面上展示一下而已。

在当前示例代码中，`UIDocumentPickerViewController` 会将用户选择的文件复制到应用程序的沙盒目录中。这是 `UIDocumentPickerViewController` 的默认行为，它确保应用能够访问用户选择的文件，但也意味着文件会被复制到应用的沙盒中。

如果你不希望复制文件，只是想在页面上展示图片，你可以使用 `Photos` 框架来直接访问和展示图片，而不进行文件的复制。以下是一个示例代码，展示如何使用 `PHPickerViewController` 直接选择并显示图片，而不进行文件的复制：

```swift
import SwiftUI
import PhotosUI

struct ContentView: View {
    @State private var selectedImages: [UIImage] = []
    @State private var isPickerPresented: Bool = false
    
    var body: some View {
        VStack {
            ScrollView(.horizontal) {
                HStack {
                    ForEach(selectedImages, id: \.self) { image in
                        Image(uiImage: image)
                            .resizable()
                            .scaledToFit()
                            .frame(width: 100, height: 100)
                            .clipped()
                    }
                }
            }
            
            Button("选择图片") {
                isPickerPresented = true
            }
            .sheet(isPresented: $isPickerPresented) {
                PhotoPicker(selectedImages: $selectedImages)
            }
        }
    }
}

struct PhotoPicker: UIViewControllerRepresentable {
    @Binding var selectedImages: [UIImage]
    
    func makeUIViewController(context: Context) -> PHPickerViewController {
        var config = PHPickerConfiguration()
        config.filter = .images
        config.selectionLimit = 0 // 0 代表不限制选择数量
        let picker = PHPickerViewController(configuration: config)
        picker.delegate = context.coordinator
        return picker
    }
    
    func updateUIViewController(_ uiViewController: PHPickerViewController, context: Context) {}
    
    func makeCoordinator() -> Coordinator {
        Coordinator(self)
    }
    
    class Coordinator: NSObject, PHPickerViewControllerDelegate {
        var parent: PhotoPicker
        
        init(_ parent: PhotoPicker) {
            self.parent = parent
        }
        
        func picker(_ picker: PHPickerViewController, didFinishPicking results: [PHPickerResult]) {
            picker.dismiss(animated: true)
            
            parent.selectedImages = []
            
            for result in results {
                if result.itemProvider.canLoadObject(ofClass: UIImage.self) {
                    result.itemProvider.loadObject(ofClass: UIImage.self) { (image, error) in
                        if let uiImage = image as? UIImage {
                            DispatchQueue.main.async {
                                self.parent.selectedImages.append(uiImage)
                            }
                        }
                    }
                }
            }
        }
    }
}

struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}
```

在这个示例中：

1. `PhotoPicker` 使用 `PHPickerViewController` 来让用户选择图片，并设置 `selectionLimit = 0` 来允许用户选择多张图片。
2. `Coordinator` 实现了 `PHPickerViewControllerDelegate` 协议，从 `PHPickerResult` 中直接加载 `UIImage`，无需将文件复制到应用的沙盒中。

这样，用户选择的图片会直接显示在页面上，而不会被复制到应用的目录中。

# 为什么UIDocumentPickerViewController 会将用户选择的文件复制到应用程序的沙盒目录中？如何用户选择了很多图片，这种复制操作会不会导致磁盘空间的浪费。图片被复制之后在何时被移除呢？

`UIDocumentPickerViewController` 将用户选择的文件复制到应用程序的沙盒目录中，是为了确保应用能够始终访问这些文件，并且保证文件不会因用户或其他应用的操作而被删除或更改。这种行为也符合 iOS 的沙盒安全机制，确保应用只能访问自己的文件。

如果用户选择了很多图片，这种复制操作确实会占用额外的磁盘空间。文件被复制到应用的沙盒目录后，除非开发者显式地删除它们，否则它们将一直存在。开发者需要在适当的时候手动管理这些文件，以防止磁盘空间浪费。

为了避免不必要的文件复制和磁盘空间浪费，可以考虑使用 `PHPickerViewController`，它不会复制文件，而是直接提供访问选中图片的方法。以下是一个如何使用 `PHPickerViewController` 的示例代码：

```swift
import SwiftUI
import PhotosUI

struct ContentView: View {
    @State private var selectedImages: [UIImage] = []
    @State private var isPickerPresented: Bool = false
    
    var body: some View {
        VStack {
            ScrollView(.horizontal) {
                HStack {
                    ForEach(selectedImages, id: \.self) { image in
                        Image(uiImage: image)
                            .resizable()
                            .scaledToFit()
                            .frame(width: 100, height: 100)
                            .clipped()
                    }
                }
            }
            
            Button("选择图片") {
                isPickerPresented = true
            }
            .sheet(isPresented: $isPickerPresented) {
                PhotoPicker(selectedImages: $selectedImages)
            }
        }
    }
}

struct PhotoPicker: UIViewControllerRepresentable {
    @Binding var selectedImages: [UIImage]
    
    func makeUIViewController(context: Context) -> PHPickerViewController {
        var config = PHPickerConfiguration()
        config.filter = .images
        config.selectionLimit = 0 // 0 代表不限制选择数量
        let picker = PHPickerViewController(configuration: config)
        picker.delegate = context.coordinator
        return picker
    }
    
    func updateUIViewController(_ uiViewController: PHPickerViewController, context: Context) {}
    
    func makeCoordinator() -> Coordinator {
        Coordinator(self)
    }
    
    class Coordinator: NSObject, PHPickerViewControllerDelegate {
        var parent: PhotoPicker
        
        init(_ parent: PhotoPicker) {
            self.parent = parent
        }
        
        func picker(_ picker: PHPickerViewController, didFinishPicking results: [PHPickerResult]) {
            picker.dismiss(animated: true)
            
            parent.selectedImages = []
            
            for result in results {
                if result.itemProvider.canLoadObject(ofClass: UIImage.self) {
                    result.itemProvider.loadObject(ofClass: UIImage.self) { (image, error) in
                        if let uiImage = image as? UIImage {
                            DispatchQueue.main.async {
                                self.parent.selectedImages.append(uiImage)
                            }
                        }
                    }
                }
            }
        }
    }
}

struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}
```

这个示例中，`PHPickerViewController` 直接从用户的照片库中加载图片，而不复制文件。这不仅可以避免磁盘空间浪费，还简化了文件管理的复杂性。

对于已经使用 `UIDocumentPickerViewController` 并担心磁盘空间浪费的情况，您可以在适当的时机删除不需要的文件。例如，当用户关闭应用或完成某项操作时，您可以清理这些临时文件。

```swift
func deleteTemporaryFiles() {
    let fileManager = FileManager.default
    let tempDirectory = fileManager.temporaryDirectory

    do {
        let fileURLs = try fileManager.contentsOfDirectory(at: tempDirectory, includingPropertiesForKeys: nil, options: [])
        for fileURL in fileURLs {
            try fileManager.removeItem(at: fileURL)
        }
    } catch {
        print("Error deleting temporary files: \(error)")
    }
}
```

在适当的地方调用 `deleteTemporaryFiles` 方法，以确保清理不需要的文件。这样可以有效地管理磁盘空间，避免浪费。

# UIDocumentPickerViewController 会将用户选择的文件复制到应用程序的哪个目录下？

当使用 `UIDocumentPickerViewController` 选择文件时，用户选择的文件会被复制到应用程序的临时目录（`tmp`）中。这个目录可以通过以下代码获取：

```swift
let tempDirectory = FileManager.default.temporaryDirectory
```

当文件被选择并复制到临时目录后，应用可以通过 `URL` 对象访问这些文件。以下是一个示例代码，展示如何使用 `UIDocumentPickerViewController` 选择文件，并确定文件被复制到的具体路径：

```swift
import SwiftUI
import UIKit

struct ContentView: View {
    @State private var selectedImage: UIImage? = nil
    @State private var isPickerPresented: Bool = false
    
    var body: some View {
        VStack {
            if let image = selectedImage {
                Image(uiImage: image)
                    .resizable()
                    .scaledToFit()
            } else {
                Text("选择一张图片")
            }
            
            Button("选择图片") {
                isPickerPresented = true
            }
            .sheet(isPresented: $isPickerPresented) {
                DocumentPicker(selectedImage: $selectedImage)
            }
        }
    }
}

struct DocumentPicker: UIViewControllerRepresentable {
    @Binding var selectedImage: UIImage?
    
    func makeUIViewController(context: Context) -> UIDocumentPickerViewController {
        let documentPicker = UIDocumentPickerViewController(forOpeningContentTypes: [.image], asCopy: true)
        documentPicker.delegate = context.coordinator
        return documentPicker
    }
    
    func updateUIViewController(_ uiViewController: UIDocumentPickerViewController, context: Context) {}
    
    func makeCoordinator() -> Coordinator {
        Coordinator(self)
    }
    
    class Coordinator: NSObject, UIDocumentPickerDelegate {
        var parent: DocumentPicker
        
        init(_ parent: DocumentPicker) {
            self.parent = parent
        }
        
        func documentPicker(_ controller: UIDocumentPickerViewController, didPickDocumentsAt urls: [URL]) {
            guard let url = urls.first else { return }
            
            // 输出文件复制到的临时路径
            print("Selected file copied to: \(url.path)")
            
            if let data = try? Data(contentsOf: url), let uiImage = UIImage(data: data) {
                DispatchQueue.main.async {
                    self.parent.selectedImage = uiImage
                }
            }
        }
    }
}

struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}
```

在这个示例中，当用户选择文件时，`documentPicker(_:didPickDocumentsAt:)` 方法会被调用，`urls` 数组包含了选择的文件在应用临时目录中的路径。通过打印 `url.path`，你可以看到文件被复制到的具体路径。

### 删除临时文件

由于文件被复制到临时目录，建议在不需要这些文件时删除它们，以避免浪费磁盘空间。你可以在适当的时候调用以下方法来清理临时目录：

```swift
func deleteTemporaryFiles() {
    let fileManager = FileManager.default
    let tempDirectory = fileManager.temporaryDirectory

    do {
        let fileURLs = try fileManager.contentsOfDirectory(at: tempDirectory, includingPropertiesForKeys: nil, options: [])
        for fileURL in fileURLs {
            try fileManager.removeItem(at: fileURL)
        }
    } catch {
        print("Error deleting temporary files: \(error)")
    }
}
```

在适当的地方调用 `deleteTemporaryFiles` 方法，以确保临时文件被删除，从而有效管理磁盘空间。

# 可以获取到用户通过UIDocumentPickerViewController 选择的图片的唯一标识吗？

通过 `UIDocumentPickerViewController` 选择文件后，无法直接获取文件的唯一标识符（如文件的原始路径或标识符），因为文件会被复制到应用的沙盒目录。不过，你可以获取文件的 `URL` 并使用该 `URL` 来生成一个唯一标识符。

你可以通过以下几种方法来生成和管理这些文件的唯一标识：

1. **使用文件的临时路径**：将文件的临时路径作为标识符，但这在文件被移动或删除时会失效。
2. **生成哈希值**：根据文件内容生成一个哈希值来作为唯一标识符。
3. **文件名**：如果文件名是唯一的，可以使用文件名作为标识符。

以下是如何使用文件的 `URL` 并生成一个哈希值作为唯一标识符的示例：

```swift
import SwiftUI
import UIKit
import CommonCrypto

struct ContentView: View {
    @State private var selectedImages: [ImageItem] = []
    @State private var isPickerPresented: Bool = false
    
    var body: some View {
        VStack {
            ScrollView(.horizontal) {
                HStack {
                    ForEach(selectedImages, id: \.id) { item in
                        Image(uiImage: item.image)
                            .resizable()
                            .scaledToFit()
                            .frame(width: 100, height: 100)
                            .clipped()
                    }
                }
            }
            
            Button("选择图片") {
                isPickerPresented = true
            }
            .sheet(isPresented: $isPickerPresented) {
                DocumentPicker(selectedImages: $selectedImages)
            }
        }
    }
}

struct ImageItem: Identifiable {
    var id: String
    var image: UIImage
}

struct DocumentPicker: UIViewControllerRepresentable {
    @Binding var selectedImages: [ImageItem]
    
    func makeUIViewController(context: Context) -> UIDocumentPickerViewController {
        let documentPicker = UIDocumentPickerViewController(forOpeningContentTypes: [.image], asCopy: true)
        documentPicker.delegate = context.coordinator
        return documentPicker
    }
    
    func updateUIViewController(_ uiViewController: UIDocumentPickerViewController, context: Context) {}
    
    func makeCoordinator() -> Coordinator {
        Coordinator(self)
    }
    
    class Coordinator: NSObject, UIDocumentPickerDelegate {
        var parent: DocumentPicker
        
        init(_ parent: DocumentPicker) {
            self.parent = parent
        }
        
        func documentPicker(_ controller: UIDocumentPickerViewController, didPickDocumentsAt urls: [URL]) {
            for url in urls {
                if let data = try? Data(contentsOf: url), let uiImage = UIImage(data: data) {
                    let id = generateHash(from: data)
                    let imageItem = ImageItem(id: id, image: uiImage)
                    DispatchQueue.main.async {
                        self.parent.selectedImages.append(imageItem)
                    }
                }
            }
        }
        
        func generateHash(from data: Data) -> String {
            var hash = [UInt8](repeating: 0, count: Int(CC_SHA256_DIGEST_LENGTH))
            data.withUnsafeBytes {
                _ = CC_SHA256($0.baseAddress, CC_LONG(data.count), &hash)
            }
            return hash.map { String(format: "%02x", $0) }.joined()
        }
    }
}

struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}
```

在这个示例中：

1. `ImageItem` 结构体包含一个唯一标识符 `id` 和图片 `image`。
2. 在 `Coordinator` 类中，使用 SHA-256 算法生成文件数据的哈希值，作为图片的唯一标识符。
3. 每当用户选择文件时，将图片和其唯一标识符添加到 `selectedImages` 数组中。

这样，即使文件被复制到临时目录，你仍然可以通过哈希值来唯一标识每张图片。哈希值是基于文件内容生成的，因此对于相同的文件，无论路径如何变化，其哈希值始终一致。

# 多次选择同一张图片，改图片的唯一标识会变化吗？

如果唯一标识符是基于文件内容生成的哈希值，那么多次选择同一张图片，唯一标识符不会变化，因为文件内容不变，哈希值也会保持一致。

以下是一个完整的示例代码，展示如何使用 `SHA-256` 算法生成文件内容的哈希值作为唯一标识符，以确保多次选择同一张图片时，唯一标识符保持一致：

```swift
import SwiftUI
import UIKit
import CommonCrypto

struct ContentView: View {
    @State private var selectedImages: [ImageItem] = []
    @State private var isPickerPresented: Bool = false
    
    var body: some View {
        VStack {
            ScrollView(.horizontal) {
                HStack {
                    ForEach(selectedImages, id: \.id) { item in
                        Image(uiImage: item.image)
                            .resizable()
                            .scaledToFit()
                            .frame(width: 100, height: 100)
                            .clipped()
                    }
                }
            }
            
            Button("选择图片") {
                isPickerPresented = true
            }
            .sheet(isPresented: $isPickerPresented) {
                DocumentPicker(selectedImages: $selectedImages)
            }
        }
    }
}

struct ImageItem: Identifiable {
    var id: String
    var image: UIImage
}

struct DocumentPicker: UIViewControllerRepresentable {
    @Binding var selectedImages: [ImageItem]
    
    func makeUIViewController(context: Context) -> UIDocumentPickerViewController {
        let documentPicker = UIDocumentPickerViewController(forOpeningContentTypes: [.image], asCopy: true)
        documentPicker.delegate = context.coordinator
        return documentPicker
    }
    
    func updateUIViewController(_ uiViewController: UIDocumentPickerViewController, context: Context) {}
    
    func makeCoordinator() -> Coordinator {
        Coordinator(self)
    }
    
    class Coordinator: NSObject, UIDocumentPickerDelegate {
        var parent: DocumentPicker
        
        init(_ parent: DocumentPicker) {
            self.parent = parent
        }
        
        func documentPicker(_ controller: UIDocumentPickerViewController, didPickDocumentsAt urls: [URL]) {
            for url in urls {
                if let data = try? Data(contentsOf: url), let uiImage = UIImage(data: data) {
                    let id = generateHash(from: data)
                    let imageItem = ImageItem(id: id, image: uiImage)
                    DispatchQueue.main.async {
                        // 检查是否已经存在相同的图片
                        if !self.parent.selectedImages.contains(where: { $0.id == id }) {
                            self.parent.selectedImages.append(imageItem)
                        }
                    }
                }
            }
        }
        
        func generateHash(from data: Data) -> String {
            var hash = [UInt8](repeating: 0, count: Int(CC_SHA256_DIGEST_LENGTH))
            data.withUnsafeBytes {
                _ = CC_SHA256($0.baseAddress, CC_LONG(data.count), &hash)
            }
            return hash.map { String(format: "%02x", $0) }.joined()
        }
    }
}

struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}
```

在这个示例中：

1. `generateHash(from:)` 函数使用 SHA-256 算法根据文件内容生成哈希值，确保唯一标识符不会变化。
2. 在 `documentPicker(_:didPickDocumentsAt:)` 方法中，当用户选择文件时，会检查该文件的哈希值是否已经存在于 `selectedImages` 数组中，如果不存在，则添加新图片。

通过这种方式，可以确保即使多次选择同一张图片，其唯一标识符也不会变化，避免重复添加相同的图片。

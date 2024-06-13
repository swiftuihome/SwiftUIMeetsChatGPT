# 使用SwiftUI从相册选择一张照片并显示

```swift

import SwiftUI
import PhotosUI

/// - 从相册选择一张照片并显示
struct PhotoPickerView: View {
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
            
            Spacer()
            
            Button("选择图片") {
                isPickerPresented = true
            }
            .buttonStyle(.borderedProminent)
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

struct PhotoPickerView_Previews: PreviewProvider {
    static var previews: some View {
        PhotoPickerView()
    }
}
```

# 使用SwiftUI从相册选择多张照片并显示

```swift

import SwiftUI
import PhotosUI

/// - 从相册选择多张照片并显示
struct MultiplePhotoPickerView: View {
    @State private var selectedImages: [UIImage] = []
    @State private var isPickerPresented: Bool = false
    
    var body: some View {
        VStack {
            ScrollView(.vertical) {
                VStack {
                    ForEach(selectedImages, id: \.self) { image in
                        Image(uiImage: image)
                            .resizable()
                            .scaledToFit()
                            .clipped()
                    }
                    Spacer()
                }
                .padding(0)
                .frame(maxWidth: .infinity)
            }
            
            Button("选择图片") {
                isPickerPresented = true
            }
            .buttonStyle(.borderedProminent)
            .sheet(isPresented: $isPickerPresented) {
                MultiplePhotoPicker(selectedImages: $selectedImages)
            }
        }
    }
}

struct MultiplePhotoPicker: UIViewControllerRepresentable {
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
        var parent: MultiplePhotoPicker
        
        init(_ parent: MultiplePhotoPicker) {
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

struct MultiplePhotoPickerView_Previews: PreviewProvider {
    static var previews: some View {
        MultiplePhotoPickerView()
    }
}
```

# 使用SwiftUI从文件夹中选择一张照片并显示

```swift
import SwiftUI
import UIKit

/// - 从文件夹中选择一张照片并显示
struct DocumentPickerView: View {
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
            
            Spacer()
            
            Button("选择图片") {
                isPickerPresented = true
            }
            .buttonStyle(.borderedProminent)
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

struct DocumentPickerView_Previews: PreviewProvider {
    static var previews: some View {
        DocumentPickerView()
    }
}
```

# 使用SwiftUI从文件夹中选择多张照片并显示

```swift
import SwiftUI
import UIKit

/// - 从文件夹中选择多张照片并显示
struct MultipleDocumentPickerView: View {
    @State private var selectedImages: [UIImage] = []
    @State private var isPickerPresented: Bool = false
    
    var body: some View {
        VStack {
            ScrollView(.vertical) {
                VStack {
                    ForEach(selectedImages, id: \.self) { image in
                        Image(uiImage: image)
                            .resizable()
                            .scaledToFit()
                            .clipped()
                    }
                    Spacer()
                }
                .padding(0)
                .frame(maxWidth: .infinity)
            }
            
            Button("选择图片") {
                isPickerPresented = true
            }
            .buttonStyle(.borderedProminent)
            .sheet(isPresented: $isPickerPresented) {
                MultipleDocumentPicker(selectedImages: $selectedImages)
            }
        }
    }
}

struct MultipleDocumentPicker: UIViewControllerRepresentable {
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
        var parent: MultipleDocumentPicker
        
        init(_ parent: MultipleDocumentPicker) {
            self.parent = parent
        }
        
        func documentPicker(_ controller: UIDocumentPickerViewController, didPickDocumentsAt urls: [URL]) {
            for url in urls {
                // 输出临时文件的路径
                print("Selected file copied to: \(url.path)")
                if let data = try? Data(contentsOf: url), let uiImage = UIImage(data: data) {
                    DispatchQueue.main.async {
                        self.parent.selectedImages.append(uiImage)
                    }
                }
            }
        }
    }
}

struct MultipleDocumentPickerView_Previews: PreviewProvider {
    static var previews: some View {
        MultipleDocumentPickerView()
    }
}
```


---
title: File importing and exporting in SwiftUI
layout: post
---

A few weeks ago, we talked about photo and video picking in SwiftUI. Today we will continue the topic and learn how to import and export files in SwiftUI views. Fortunately, it is straightforward to do with the help of new *fileImporter* and *fileExporter* view modifiers.

#### Import
The SwiftUI framework provides us the *fileImporter* view modifier allowing us to enable file-picking user experience easily. It completely handles the dialog and navigation between folders. Let's take a look at how we can use it.

```swift
struct ImportingExampleView: View {
    @State private var importing = false
    
    var body: some View {
        Button("Import") {
            importing = true
        }
        .fileImporter(
            isPresented: $importing,
            allowedContentTypes: [.plainText]
        ) { result in
            switch result {
            case .success(let file):
                print(file.absoluteString)
            case .failure(let error):
                print(error.localizedDescription)
            }
        }
    }
}
```

As you can see in the example above, we attach the *fileImporter* view modifier to a button that toggles the *importing* property, which we use as a binding to enable the file-picking experience. We also use the *allowedContentTypes* parameter on the *fileImporter* view modifier to pass an array of allowed file types. In the completion closure, we can handle the result and extract the URL of the selected file.

#### Export
Exporting files works in a very similar way, but we also should provide a document that we are going to export. In this case, the document type should conform to the *FileDocument* protocol.

```swift
struct TextDocument: FileDocument {
    static var readableContentTypes: [UTType] {
        [.plainText]
    }
    
    var text = ""
    
    init(text: String) {
        self.text = text
    }
    
    init(configuration: ReadConfiguration) throws {
        if let data = configuration.file.regularFileContents {
            text = String(decoding: data, as: UTF8.self)
        } else {
            text = ""
        }
    }
    
    func fileWrapper(configuration: WriteConfiguration) throws -> FileWrapper {
        FileWrapper(regularFileWithContents: Data(text.utf8))
    }
}
```

Here we define the *TextDocument* type conforming to the *FileDocument* protocol. As you can see, it implements plain string reading from a file and allows us to export string data to the file. Now we can use the *fileExporter* view modifier to export an instance of the *TextDocument* type.

```swift
struct ExportingExampleView: View {
    @State private var exporting = false
    @State private var document = TextDocument(text: "")
    
    var body: some View {
        TextEditor(text: $document.text)
            .toolbar {
                Button("Export") {
                    exporting = true
                }
                .fileExporter(
                    isPresented: $exporting,
                    document: document,
                    contentType: .plainText
                ) { result in
                    switch result {
                    case .success(let file):
                        print(file)
                    case .failure(let error):
                        print(error)
                    }
                }
            }
    }
}
```

As you can see in the example above, we use the *fileExporter* view modifier to enable file exporting user experience. It also needs binding to a boolean value to present the dialog. We must also pass the document we want to export and its content type. In the completion closure, we can verify that the document was exported correctly or check the reason for a failure.

#### File moving
As a bonus, the SwiftUI framework provides us with the *fileMover* view modifier, enabling a file-moving experience for our users.

```swift
struct MovingExampleView: View {
    @State private var moving = false
    let file: URL
    
    var body: some View {
        Button("Move files") {
            moving = true
        }
        .fileMover(isPresented: $moving, file: file) { result in
            switch result {
            case .success(let file):
                print(file.absoluteString)
            case .failure(let error):
                print(error.localizedDescription)
            }
        }
    }
}
```

In the example above, we use the *fileMover* view modifier to present the file-moving dialog to the user. We should pass the file URL we want to move and can verify the result in the completion closure.

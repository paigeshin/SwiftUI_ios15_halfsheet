# SwiftUI_ios15_halfsheet

```swift
//
//  DraggableModalView.swift
//  MiracleNight
//
//  Created by paige shin on 2023/06/12.
//

import SwiftUI

extension View {
    
    func halfSheet<SheetView: View>(showSheet: Binding<Bool>,
                                    @ViewBuilder sheetView: @escaping() -> SheetView,
                                    onDismiss: (() -> Void)? = nil) -> some View {
        // why we using background...
        // because it will automatically use the swift frame size only...
        return self
            .background(
                HalfSheetHelper(showSheet: showSheet, sheetView: sheetView())
            )
    }
    
}

struct HalfSheetHelper<SheetView: View>: UIViewControllerRepresentable {
    
    let controller = UIViewController()
    @Binding var showSheet: Bool
    var sheetView: SheetView
    var onDismiss: (() -> Void)?
    
    func makeUIViewController(context: Context) -> UIViewController {
        self.controller.view.backgroundColor = .clear
        return self.controller
    }
    
    func updateUIViewController(_ uiViewController: UIViewController, context: Context) {
        if self.showSheet {
            let sheetController = CustomHostingView(rootView: self.sheetView)
            sheetController.presentationController?.delegate = context.coordinator
            uiViewController.present(sheetController, animated: true)
        } else {
            uiViewController.dismiss(animated: true)
        }
    }
    
    func makeCoordinator() -> Coordinator {
        return Coordinator(parent: self)
    }
    
    // On Dismiss
    class Coordinator: NSObject, UISheetPresentationControllerDelegate {
        
        var parent: HalfSheetHelper
        
        init(parent: HalfSheetHelper) {
            self.parent = parent
        }
        
        func presentationControllerWillDismiss(_ presentationController: UIPresentationController) {
            self.parent.showSheet = false
        }
        
        func presentationControllerDidDismiss(_ presentationController: UIPresentationController) {
            self.parent.onDismiss?()
        }
        
    }
}

fileprivate class CustomHostingView<Content: View>: UIHostingController<Content> {
    
    override func viewDidLoad() {
        
        self.view.backgroundColor = .clear
        
        // setting presentation controller properties...
        if let presentationController = self.presentationController as? UISheetPresentationController {
            presentationController.detents = [.medium(), .large()]
            presentationController.prefersGrabberVisible = true
        }

    }
    
}

```

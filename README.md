# Camera-SDK

Why IOS SDK? Why we should use it?
- In IOS we can have one common module to use it in different applications. If we want to distribute that module to social world and we don’t want to reveal our logic then we need to create framework of our code. In this article we are going to create one simple custom camera implementation as framework. We can use that framework in any application. 
- Using this framework we can add different kind of functionalities like shutter speed, Video recording, slomotion recording, Photo timer, live filters. 

# Installation

1. Create new project and drag SOCameraManager.framework into the Root folder of your project.
Than Go to your project target, click on the "Build Settings" tab, and Add “SOCameraManager.framework" to Embedded Binaries in the app's General setting.

[![solarized dualmode](https://github.com/spaceotech/Camera-SDK/blob/master/1.png)](#features)

2. Create UI/UX for custom camera.
[![solarized dualmode](https://github.com/spaceotech/Camera-SDK/blob/master/2.png)](#features)

3. ViewController.swift

import UIKit
import SOCameraManager

class ViewController: UIViewController {
    // MARK: - Constants
    let cameraManager = SOCameraManager()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view, typically from a nib.
        cameraManager.showAccessPermissionPopupAutomatically = false
        
        let currentCameraState = cameraManager.currentCameraStatus()        
        if currentCameraState == .NotDetermined {
            
        } else if (currentCameraState == .Ready) {
            addCameraToView()
        }
        if !cameraManager.hasFlash {
            btnFlash.enabled = false
            btnFlash.setTitle("On", forState: UIControlState.Normal)
        }
    }
    
    override func viewWillAppear(animated: Bool) {
        super.viewWillAppear(animated)
        navigationController?.navigationBar.hidden = true
        cameraManager.resumeCaptureSession()
    }
    
    override func viewWillDisappear(animated: Bool) {
        super.viewWillDisappear(animated)
        cameraManager.stopCaptureSession()
    }
    
    // MARK: - Add camera to view 
    private func addCameraToView()   {
        cameraManager.addPreviewLayerToView(cameraView, newCameraOutputMode: SOCameraOutputMode.StillImage)
        cameraManager.showErrorBlock = { [weak self] (erTitle: String, erMessage: String) -> Void in
            
            let alertController = UIAlertController(title: erTitle, message: erMessage, preferredStyle: .Alert)
            alertController.addAction(UIAlertAction(title: "OK", style: UIAlertActionStyle.Default, handler: { (alertAction) -> Void in  }))            
            self?.presentViewController(alertController, animated: true, completion: nil)
        }
    }
    
    // MARK: - Change Flash Mode
    @IBAction func btnFlashClicekd(sender: AnyObject) {        
        switch (cameraManager.changeFlashMode()) {
        case .Off:
            sender.setTitle("Off", forState: UIControlState.Normal)
        case .On:
            sender.setTitle("On", forState: UIControlState.Normal)
        case .Auto:
            sender.setTitle("Auto", forState: UIControlState.Normal)
        }
    }
    
    // MARK: - Camera capture photo
    @IBAction func btnCaptureClicked(sender: AnyObject) {
        cameraManager.capturePictureWithCompletition({ (image, error) -> Void in
            let vc: ImageViewController? = self.storyboard?.instantiateViewControllerWithIdentifier("ImageVC") as? ImageViewController
            if let validVC: ImageViewController = vc {
                if let capturedImage = image {
                    validVC.image = capturedImage
                    self.navigationController?.pushViewController(validVC, animated: true)
                }
            }
        })
    }
    
    // MARK: - Rotate camera front and back 
    @IBAction func btnCameraFlipClicked(sender: AnyObject) {
        cameraManager.cameraDevice = cameraManager.cameraDevice == SOCameraDevice.Front ? SOCameraDevice.Back : SOCameraDevice.Front
        switch (cameraManager.cameraDevice) {
        case .Front:
            sender.setTitle("Front", forState: UIControlState.Normal)
        case .Back:
            sender.setTitle("Back", forState: UIControlState.Normal)
        }
    }
        // MARK: - Ask permission to user 
    @IBAction func askForCameraPermissions(sender: UIButton) {
        cameraManager.askUserForCameraPermissions({ permissionGranted in
            if permissionGranted {
                self.addCameraToView()
            }
        })
    }
}



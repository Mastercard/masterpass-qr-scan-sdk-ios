# Masterpass QR Scan SDK

This SDK provides UI components for QR scanning that allows to modify simple attributes of the views or use custom views for display.

*This SDK is developed in Objective-C and it works with Swift.*

This SDK is based on [QRCodeReaderViewController][1]

### Requirements:
1. Xcode 9.0.0+
2. iOS 8.0+

### Features:
1. Simple interface for QR scanning UI customization
2. Easily extendible by using custom UI
3. Scanning features for both portrait and landscape mode

### Installation

#### Cocoapods
- In your *Podfile* write the following

  ```cocoapods
  use_frameworks!
  pod 'MasterpassQRScanSDK'
  ```

- Do `pod install`
- Everything is setup now


#### Manual
##### Swift
- Download the latest release of [Masterpass QR Scan SDK][2].
- Unzip the file.
- Go to your Xcode project’s “General” settings. Drag MasterpassQRScanSDK.framework to the “Embedded Binaries” section. Make sure to select **Copy items if needed** and click Finish.
- Create a new **Run Script Phase** in your app’s target’s **Build Phases** and paste the following snippet in the script text field:

	`bash "${BUILT_PRODUCTS_DIR}/${FRAMEWORKS_FOLDER_PATH}/MasterpassQRScanSDK.framework/strip-frameworks.sh"`

  This step is required to work around an App Store submission bug when archiving universal binaries.


##### Objc
- Follow same instructions as Swift

[1]: https://github.com/yannickl/QRCodeReaderViewController
[2]: https://www.github.com/Mastercard/masterpass-qr-scan-sdk-ios/releases/download/2.0.1/masterpassqrscansdk-framework-ios.zip

### Usage

In iOS10+, you will need first provide a reasoning about the camera use. For that you'll need to add the **Privacy - Camera Usage Description** *(NSCameraUsageDescription)* field in your Info.plist

#### Simple
1. Check camera permissions. Make sure the use of camera is authorised.
2. Create and configure `QRReaderViewControllerBuilder` instance.
3. Create a `QRReaderViewController` with `QRReaderViewControllerBuilder` instance.
4. Set the delegate of `QRReaderViewController` instance.
5. The data also can be read using block/closure
6. Present the controller.

*Note that you should check whether the device supports the reader library by using the `QRCodeReader.isAvailable()` and the `QRCodeReader.supportsQRCode()` methods.*

__Swift__

```swift
import MasterpassQRScanSDK
import AVFoundation

@IBAction func scanWithOriginalTheme(_ sender: Any) {
    guard QRCodeReader.isAvailable() && QRCodeReader.supportsQRCode() else {
        return
    }
    // Presents the readerVC
    checkCameraPermission { [weak self] in
        guard let strongSelf = self else {
            return
        }

        var reader:QRCodeReader?

        let qrVC = QRCodeReaderViewController(builder: QRCodeReaderViewControllerBuilder {
            $0.startScanningAtLoad = false
            reader = $0.reader
        })

        //block to read the result
        reader?.setCompletionWith({ result in
            reader?.stopScanning()
            self?.dismiss(animated: true, completion: nil)
        })

        //block when cancel is pressed
        qrVC.setCompletionWith({ result in
            reader?.stopScanning()
            self?.dismiss(animated: true, completion: nil)
        })

        // Retrieve the QRCode content via delegate
        qrVC.delegate = self

        strongSelf.present(qrVC, animated: true, completion: {
            qrVC.startScanning()
        })
    }
}

// Check camera permissions
func checkCameraPermission(completion: @escaping () -> Void) {
    let cameraMediaType = AVMediaType.video
    let cameraAuthorizationStatus = AVCaptureDevice.authorizationStatus(for: cameraMediaType)

    switch cameraAuthorizationStatus {
    case .denied:
        showAlert(title: "Error", message: "Camera permissions are required for scanning QR. Please turn on Settings -> MasterpassQR Demo -> Camera")
        break
    case .restricted:
        showAlert(title: "Error", message: "Camera permissions are restricted for scanning QR")
        break
    case .authorized:
        completion()
    case .notDetermined:
        // Prompting user for the permission to use the camera.
        AVCaptureDevice.requestAccess(for: cameraMediaType) { [weak self] granted in
            guard let strongSelf = self else { return }

            DispatchQueue.main.async {
                if granted {
                    completion()
                } else {
                    strongSelf.showAlert(title: "Error", message: "Camera permissions are required for scanning QR. Please turn on Settings -> MasterpassQR Demo -> Camera")
                }
            }
        }
    }
}

func showAlert(title: String, message: String) {
    let alert = UIAlertController(title: title, message: message, preferredStyle: .alert)
    alert.addAction(UIAlertAction(title: "Ok", style: .cancel, handler: nil))
    present(alert, animated: true, completion: nil)
}

// MARK: - QRCodeReaderViewController Delegate Methods
func reader(_ reader: QRCodeReaderViewController, didScanResult result: String) {
    reader.stopScanning()
    dismiss(animated: true, completion: nil)
}

func readerDidCancel(_ reader: QRCodeReaderViewController) {
    reader.stopScanning()
    dismiss(animated: true, completion: nil)
}
```

__Objective-C__

```objc
@import MasterpassQRScanSDK;
@import AVFoundation;

- (IBAction)scanAction:(id)sender {
  if (![QRCodeReader isAvailable] || ![QRCodeReader supportsQRCode]) {
      return;
  }
  __weak typeof(self) weakSelf = self;
  [self checkCameraPermission: ^{

      __block __weak QRCodeReader* reader;
      QRCodeReaderViewController* qrVC = [QRCodeReaderViewController readerWithBuilderBlock:^(QRCodeReaderViewControllerBuilder *builder){
          reader = builder.reader;
      }];

      //block to read the result
      [reader setCompletionWithBlock:^(NSString *result) {
          [reader stopScanning];
          [self dismissViewControllerAnimated:YES completion: nil];
      }];

      //block when cancel is pressed
      [qrVC setCompletionWithBlock:^(NSString *result) {
          [reader stopScanning];
          [self dismissViewControllerAnimated:YES completion: nil];
      }];

      // Retrieve the QRCode content via delegate
      qrVC.delegate = weakSelf;

      [weakSelf presentViewController:qrVC animated:true completion:nil];
  }];
}

// Check camera permissions
- (void)checkCameraPermission:(void (^)(void))completion {
    AVAuthorizationStatus status = [AVCaptureDevice authorizationStatusForMediaType:AVMediaTypeVideo];
    if (status == AVAuthorizationStatusDenied) {
        [self showAlertWithTitle:@"Error" message: @"Camera permissions are required for scanning QR. Please turn on Settings -> MasterpassQR Demo -> Camera"];
        return;
    } else if (status == AVAuthorizationStatusRestricted) {
        [self showAlertWithTitle:@"Error" message: @"Camera permissions are restricted for scanning QR"];
        return;
    } else if (status == AVAuthorizationStatusAuthorized) {
        completion();
    } else if (status == AVAuthorizationStatusNotDetermined) {
        __weak __typeof(self) weakSelf = self;
        [AVCaptureDevice requestAccessForMediaType:AVMediaTypeVideo completionHandler:^(BOOL granted) {
            dispatch_async(dispatch_get_main_queue(), ^{
                if (granted) {
                    completion();
                } else {
                    [weakSelf showAlertWithTitle:@"Error" message: @"Camera permissions are required for scanning QR. Please turn on Settings -> MasterpassQR Demo -> Camera"];
                }
            });
        }];
    }
}

- (void)showAlertWithTitle:(NSString *)title message:(NSString *)message {
    UIAlertController *controller = [UIAlertController alertControllerWithTitle:title message:message preferredStyle:UIAlertControllerStyleAlert];
    [controller addAction:[UIAlertAction actionWithTitle:@"Ok" style:UIAlertActionStyleCancel handler:nil]];
    [self presentViewController:controller animated:true completion:nil];
}

# pragma mark - QRCodeReaderViewControllerDelegate Methods
- (void)reader:(QRCodeReaderViewController *)reader didScanResult:(NSString *)result {
    [reader stopScanning];
    [self dismissViewControllerAnimated:YES completion: nil];
}

- (void)readerDidCancel:(QRCodeReaderViewController *)reader {
    [reader stopScanning];
    [self dismissViewControllerAnimated:YES completion: nil];
}
```

#### Advanced

##### Custom View Controller
Using custom view controller with `QRReaderViewController` embedded as child view controller.

__Swift__

```swift

import MasterpassQRScanSDK

class CustomViewController : UIViewController, QRCodeReaderDelegate {

  lazy var reader: QRCodeReaderViewController = {
    return QRCodeReaderViewController(builder: QRCodeReaderViewControllerBuilder {
        let readerView = $0.readerView

        // Setup overlay view
        let overlayView = readerView.getOverlay()
        overlayView.cornerColor = UIColor.purple
        overlayView.cornerWidth = 6
        overlayView.cornerLength = 75
        overlayView.indicatorSize = CGSize(width: 250, height: 250)

        // Setup scanning region
        $0.scanRegionSize = CGSize(width: 250, height: 250)

        // Hide torch button provided by the default view
        $0.showTorchButton = false

        // Hide cancel button provided by the default view
        $0.showCancelButton = false

        // Don't start scanning when this view is loaded i.e initialized
        $0.startScanningAtLoad = false

        $0.showSwitchCameraButton = false;

    })
  }()

  override func viewDidLoad() {
      super.viewDidLoad()

      reader.delegate = self

      self.addChildViewController(reader)
      self.view.insertSubview(reader.view, at: 0)

      let viewDict = ["reader" : reader.view as Any]
      self.view.addConstraints(NSLayoutConstraint.constraints(withVisualFormat: "H:|[reader]|", options: [], metrics: nil, views: viewDict))
      self.view.addConstraints(NSLayoutConstraint.constraints(withVisualFormat: "V:|[reader]|", options: [], metrics: nil, views: viewDict))

      reader.didMove(toParentViewController: self)
  }

  override func viewDidAppear(_ animated: Bool) {
      super.viewDidAppear(animated)
      reader.startScanning()
  }

  override func viewWillDisappear(_ animated: Bool) {
      super.viewWillDisappear(animated)
      reader.stopScanning()
  }

  // MARK:- Actions
  @IBAction func toggleTorch(_ sender: Any) {
      reader.codeReader!.toggleTorch()
  }

  func reader(_ reader: QRCodeReaderViewController, didScanResult result: String) {
      reader.stopScanning()
  }

  func readerDidCancel(_ reader: QRCodeReaderViewController) {
      reader.stopScanning()
  }
}
```

__Objective-C__

```objc
@import MasterpassQRScanSDK;

@interface CustomViewController () <QRCodeReaderDelegate>

@property (nonatomic, strong) QRCodeReaderViewController *qrVC;

@end

@implementation CustomViewController

@implementation CustomViewController

- (void)viewDidLoad {
    [super viewDidLoad];

    QRCodeReaderViewControllerBuilder *builder = [[QRCodeReaderViewControllerBuilder alloc] init];
    QRCodeReaderView *readerView = (QRCodeReaderView *) builder.readerView;

    // Setup overlay view
    QRCodeReaderViewOverlay *overlayView = (QRCodeReaderViewOverlay *)[readerView getOverlay];
    overlayView.cornerColor = UIColor.purpleColor;
    overlayView.cornerWidth = 6;
    overlayView.cornerLength = 75;
    overlayView.indicatorSize = CGSizeMake(250, 250);

    // Setup scanning region
    builder.scanRegionSize = CGSizeMake(250, 250);

    // Hide torch button provided by default view
    builder.showTorchButton = false;

    // Hide cancel button provided by default view
    builder.showCancelButton = false;

    // Don't start scanning when this view is loaded i.e initialized
    builder.startScanningAtLoad = false;

    builder.showSwitchCameraButton = false;

    self.qrVC = [[QRCodeReaderViewController alloc] initWithBuilder:builder];
    self.qrVC.delegate = self;

    // Add the reader as child view controller
    [self addChildViewController:self.qrVC];

    // Add reader view to the bottom
    [self.view insertSubview:self.qrVC.view atIndex: 0];

    NSDictionary *dictionary = @{@"qrVC": self.qrVC.view};
    [self.view addConstraints:[NSLayoutConstraint constraintsWithVisualFormat:@"H:|[qrVC]|" options:0 metrics:nil views:dictionary]];
    [self.view addConstraints:[NSLayoutConstraint constraintsWithVisualFormat:@"V:|[qrVC]|" options:0 metrics:nil views:dictionary]];

    [self.qrVC didMoveToParentViewController:self];
}

- (void)viewDidAppear:(BOOL)animated {
    [super viewDidAppear:animated];
    [self.qrVC startScanning];
}

- (void)viewWillDisappear:(BOOL)animated {
    [super viewDidDisappear:animated];
    [self.qrVC stopScanning];
}

#pragma mark - Actions
- (IBAction)torchButtonPressed:(id)sender {
    [self.qrVC.codeReader toggleTorch];
}

#pragma mark - QRCodeReaderViewControllerDelegate methods
- (void)reader:(QRCodeReaderViewController *)reader didScanResult:(NSString *)result {
    [reader stopScanning];
}

- (void)readerDidCancel:(QRCodeReaderViewController *)reader {
    [reader stopScanning];
}
```

##### Subclass of QRCodeReaderViewController

You can subclass the QRCodeReaderViewController to create your own looks and feel, placement of the component of the class. You can modify the following component:
- cameraView
- cancelButton
- switchCameraButton
- toggleTorchButton

__Swift__

```swift
import UIKit
import MasterpassQRScanSDK

class QRCodeReaderViewControllerSubClass: QRCodeReaderViewController {

    //Mark: - Overriden
    override func setupUIComponents(withCancelButtonTitle cancelButtonTitle: String?, cameraView: QRCodeReaderView?) {
        if let cameraView = cameraView {
            self.cameraView = cameraView;
        }else
        {
            self.cameraView = QRCodeReaderView()
        }
        view.addSubview(self.cameraView!)
    }

    override func setupAutoLayoutConstraints() {
        let views = ["cameraView":self.cameraView!]
        view.addConstraints(NSLayoutConstraint.constraints(withVisualFormat: "V:|-50-[cameraView]-50-|", options: NSLayoutFormatOptions(rawValue: 0), metrics: nil, views: views))
        view.addConstraints(NSLayoutConstraint.constraints(withVisualFormat: "H:|-50-[cameraView]-50-|", options: NSLayoutFormatOptions(rawValue: 0), metrics: nil, views: views))

    }

}
```

__Objective-C__

```objc
#import <MasterpassQRScanSDK/MasterpassQRScanSDK.h>

@interface QRCodeReaderViewControllerSubClass : QRCodeReaderViewController
@end

@implementation QRCodeReaderViewControllerSubClass

#pragma mark - Overriden
- (void)setupUIComponentsWithCancelButtonTitle:(NSString *)cancelButtonTitle cameraView:(nullable QRCodeReaderView*) cameraView
{
    self.cameraView = cameraView;
    if (!self.cameraView) {
        self.cameraView                                       = [[QRCodeReaderView alloc] init];
        self.cameraView.translatesAutoresizingMaskIntoConstraints = NO;
        self.cameraView.clipsToBounds                             = YES;
    }
    [self.view addSubview:self.cameraView];

    //setup other components

}

- (void)setupAutoLayoutConstraints
{
    NSDictionary *views = @{ @"cameraView" : self.cameraView};
    [self.view addConstraints:
     [NSLayoutConstraint constraintsWithVisualFormat:@"V:|-50-[cameraView]-50-|" options:0 metrics:nil views:views]];
    [self.view addConstraints:
     [NSLayoutConstraint constraintsWithVisualFormat:@"H:|-50-[cameraView]-50-|" options:0 metrics:nil views:views]];

     //setup other components
}

@end
```

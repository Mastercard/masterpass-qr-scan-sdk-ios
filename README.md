# Masterpass QR Scan SDK

This SDK provides UI components for QR scanning that allows to modify simple attributes of the views or use custom views for display.

*This SDK is developed in Swift and it works with Objective-C but it is recommended to use Swift for development with this SDK.*

This SDK is based on [QRCodeReader.swift][1]

### Requirements:
1. Xcode 8
2. iOS >= 8.0

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
- Go to your Xcode project's **Build Settings** and set **Always Embed Swift Standard Libraries** to **YES**

[1]: https://github.com/yannickl/QRCodeReader.swift
[2]: https://www.github.com/Mastercard/masterpass-qr-scan-sdk-ios/releases/download/1.0.4/masterpassqrscansdk-framework-ios.zip

### Usage

In iOS10+, you will need first provide a reasoning about the camera use. For that you'll need to add the **Privacy - Camera Usage Description** *(NSCameraUsageDescription)* field in your Info.plist

#### Simple
1. Check camera permissions. Make sure the use of camera is authorized.
2. Create and configure `QRReaderViewControllerBuilder` instance.
2. Create a `QRReaderViewController` with `QRReaderViewControllerBuilder` instance.
3. Set the delegate of `QRReaderViewController` instance.
4. Present the controller.

*Note that you should check whether the device supports the reader library by using the `QRCodeReader.isAvailable()` and the `QRCodeReader.supportsQRCode()` methods.*

__Swift__

```swift
import MasterpassQRScanSDK

@IBAction func scanAction(_ sender: AnyObject) {
    guard QRCodeReader.isAvailable() && QRCodeReader.supportsQRCode() else {
        return
    }
    // Presents the readerVC
    checkCameraPermission { [weak self] in
        guard let strongSelf = self else {
            return
        }

        let qrVC = QRCodeReaderViewController(builder: QRCodeReaderViewControllerBuilder {
            $0.startScanningAtLoad = false
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
        let cameraMediaType = AVMediaTypeVideo
        let cameraAuthorizationStatus = AVCaptureDevice.authorizationStatus(forMediaType: cameraMediaType)

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
            AVCaptureDevice.requestAccess(forMediaType: cameraMediaType) { [weak self] granted in
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

// MARK: - QRCodeReaderViewController Delegate Methods

func reader(_ reader: QRCodeReader, didScanResult result: QRCodeReaderResult) {
    reader.stopScanning()

    // Use QR code result

    dismiss(animated: true, completion: nil)
}

func readerDidCancel(_ reader: QRCodeReader) {
    reader.stopScanning()

    dismiss(animated: true, completion: nil)
}
```

__Objective-C__

```objc
@import MasterpassQRScanSDK;

- (IBAction)scanAction:(id)sender {
  if (![QRCodeReader isAvailable] || ![QRCodeReader supportsQRCode]) {
      return;
  }
  __weak typeof(self) weakSelf = self;
  [self checkCameraPermission: ^{
      QRCodeReaderViewControllerBuilder *builder = [[QRCodeReaderViewControllerBuilder alloc] initWithBuildBlock:^(QRCodeReaderViewControllerBuilder * _Nonnull b) {
          QRReaderOverlayView *overlayView = (QRReaderOverlayView *) b.overlayView;
          overlayView.cornerColor = [UIColor purpleColor];
      }];

      QRCodeReaderViewController *qrVC = [[QRCodeReaderViewController alloc] initWithBuilder:builder];

      qrVC.delegate = weakSelf;

      [weakSelf presentViewController:qrVC animated:true completion:nil];
  }];
}

// Check camera permissions
- (void)checkCameraPermission:(void (^)())completion {
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

- (void)reader:(QRCodeReaderViewController *)reader didScanResult:(QRCodeReaderResult *)result {
    [reader stopScanning];

    [self dismissViewControllerAnimated:YES completion: nil]
}

- (void)readerDidCancel:(QRCodeReaderViewController *)reader {
    [reader stopScanning];

    [self dismissViewControllerAnimated:YES completion: nil]
}
```

#### Advanced

##### Custom View Controller
Using custom view controller with `QRReaderViewController` embedded as child view controller.

__Swift__

```swift
lazy var reader: QRCodeReaderViewController = {
    return QRCodeReaderViewController(builder: QRCodeReaderViewControllerBuilder {
        let readerView = $0.readerView.displayable

        // Setup overlay view
        let overlayView = readerView.overlayView as! QRReaderOverlayView
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
    })
}()

override func viewDidLoad() {
    super.viewDidLoad()

    reader.delegate = self

    // Add the reader as child view controller
    self.addChildViewController(reader)

    // Add reader view to the bottom
    self.view.insertSubview(reader.view, at: 0)

    let viewDict = ["reader" : reader.view]
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
    reader.codeReader.toggleTorch()
}

func reader(_ reader: QRCodeReaderViewController, didScanResult result: QRCodeReaderResult) {
    reader.stopScanning()

    // Use QR code result
}

func readerDidCancel(_ reader: QRCodeReaderViewController) {
    reader.stopScanning()
}
```

__Objective-C__

```objc
@interface CustomViewController : UIViewController<QRCodeReaderViewControllerDelegate>

@property (nonatomic, strong) QRCodeReaderViewController *qrVC;

@end

@implementation CustomViewController

- (void)viewDidLoad {
    [super viewDidLoad];

    QRCodeReaderViewControllerBuilder *builder = [[QRCodeReaderViewControllerBuilder alloc] init];
    QRCodeReaderView *readerView = (QRCodeReaderView *) builder.readerView.displayable;

    // Setup overlay view
    QRReaderOverlayView *overlayView = (QRReaderOverlayView *)readerView.overlayView;
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
- (void)reader:(QRCodeReaderViewController *)reader didScanResult:(QRCodeReaderResult *)result {
    [reader stopScanning];

    // Use QR code result
}

- (void)readerDidCancel:(QRCodeReaderViewController *)reader {
    [reader stopScanning];
}

@end
```

##### Custom View
Providing custom view. Using this method doesn't require a view controller subclass.

You can create your own interface to scan your QR codes by using the `QRCodeReaderDisplayable` protocol and the `readerView` property in the `QRCodeReaderViewControllerBuilder`:

__Swift__

```swift
class YourCustomView: UIView, QRCodeReaderDisplayable {
  let cameraView: UIView            = UIView()
  let cancelButton: UIButton?       = UIButton()
  let switchCameraButton: UIButton? = SwitchCameraButton()
  let toggleTorchButton: UIButton?  = ToggleTorchButton()

  func setupComponents(showCancelButton: Bool, showSwitchCameraButton: Bool, showTorchButton: Bool) {
    // addSubviews
    // setup constraints
    // etc.
  }
}

lazy var reader = QRCodeReaderViewController(builder: QRCodeReaderViewControllerBuilder {
  let readerView = QRCodeReaderContainer(displayable: YourCustomView())

  $0.readerView = readerView
})
```

__Objective-C__

```objc
@interface YourCustomView : UIView<QRCodeReaderDisplayable>

@property (nonatomic, strong) UIView *cameraView;
@property (nonatomic, strong) UIButton *cancelButton;
@property (nonatomic, strong) UIButton *toggleTorchButton;
@property (nonatomic, strong) UIView *overlayView;

@end

@implementation YourCustomView

- (void)setupComponentsWithShowCancelButton:(BOOL)showCancelButton showTorchButton:(BOOL)showTorchButton showOverlayView:(BOOL)showOverlayView {
    // addSubviews
    // setup constraints
    // etc.
}

@end
```

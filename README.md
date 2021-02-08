# Manual Camera Plugin

[![pub package](https://img.shields.io/pub/v/manual_camera.svg)](https://pub.dartlang.org/packages/manual_cameraj)

This plugin is a modified version of the native Flutter [camera](https://github.com/flutter/plugins/tree/master/packages/camera) plugin.

This version allows the definition of manual parameters for the Android camera:

| Feature       |    Values     |        Type        | Description                                                                                                                       |
| :------------ | :-----------: | :----------------: | :-------------------------------------------------------------------------------------------------------------------------------- |
| enableFlash   |  true/false   |        bool        | Turn flash on or off                                                                                                              |
| focusDistance | 0 (auto) - +∞ |       double       | Distance in meters from camera to focused object                                                                                  |
| iso           | 0 (auto) - +∞ |        int         | Sensor's sensitivity to light                                                                                                     |
| shutterSpeed  | 0 (auto) - +∞ |        int         | 1s / shutterSpeed = duration of the sensor exposure                                                                               |
| whiteBalance  |      ...      | WhiteBalancePreset | White balance Android [options](https://developer.android.com/reference/android/hardware/camera2/CaptureRequest#CONTROL_AWB_MODE) |

## Installation

First, add `manual_camera` as a [dependency in your pubspec.yaml file](https://flutter.io/using-packages/).

```yaml
dependencies:
  flutter:
    sdk: flutter
  manual_camera: ^0.0.3 // add manual_camera plugin
```

### iOS

Add two rows to the `ios/Runner/Info.plist`:

- one with the key `Privacy - Camera Usage Description` and a usage description.
- and one with the key `Privacy - Microphone Usage Description` and a usage description.

Or in text format add the key:

```xml
<key>NSCameraUsageDescription</key>
<string>Can I use the camera please?</string>
<key>NSMicrophoneUsageDescription</key>
<string>Can I use the mic please?</string>
```

### Android

Change the minimum Android sdk version to 21 (or higher) in your `android/app/build.gradle` file.

```
minSdkVersion 21
```

### Example

Here is a small example flutter app displaying a full screen camera preview.

```dart
import 'dart:async';
import 'package:manual_camera/camera.dart';
import 'package:camera/camera.dart';

List<CameraDescription> cameras;

Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  cameras = await availableCameras();
  runApp(CameraApp());
}

class CameraApp extends StatefulWidget {
  @override
  _CameraAppState createState() => _CameraAppState();
}

class _CameraAppState extends State<CameraApp> {
  CameraController controller;

  @override
  void initState() {
    super.initState();
    controller = CameraController(
      cameras[0],
      ResolutionPreset.medium,
      iso: 100,
      shutterSpeed: 30,
      whiteBalance: WhiteBalancePreset.cloudy,
      focusDistance: 0.1,
    );
    controller.initialize().then((_) {
      if (!mounted) {
        return;
      }
      setState(() {});
      Future.delayed(
        Duration(
          milliseconds: 1000,
        ),
      ).then((_) => _cameraController.flash(true));
    });
  }

  @override
  void dispose() {
    controller?.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    if (!controller.value.isInitialized) {
      return Container();
    }
    return AspectRatio(
        aspectRatio:
        controller.value.aspectRatio,
        child: CameraPreview(controller));
  }
}
```

_Note_: This plugin is still under development, and some APIs might not be available yet. I hope that in the future IOS manual exposition will also be available.

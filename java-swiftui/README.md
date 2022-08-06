# Webview Sensordata Communication

## Android (Java)

Android snippet is developed using latest android sdk 32.

The sensor data is obtained and updated using the `SensorManager` from `android.hardware` which is part of the android sdk.

Application is reading accelerometer, magnetometer and gyroscope data and is sending to WebView from `​​android.webkit` which is also part of the android sdk.

Sensor data is sent to the WebView as a JSON object.

```
{
  "accelerometer": "[x, y, z]",
  "gyroscope": "[x, y, z]",
  "magnetometer": "[x, y, z]"
}
```

The application is not using any third party dependencies, everything is part of the Android API.

### Complete java implementation:
```java
package com.example.sensordatapoc;

import android.annotation.SuppressLint;
import android.content.Context;
import android.hardware.Sensor;
import android.hardware.SensorEvent;
import android.hardware.SensorEventListener;
import android.hardware.SensorManager;
import android.os.Bundle;
import android.webkit.WebView;
import android.webkit.WebViewClient;

import androidx.appcompat.app.AppCompatActivity;

import org.json.JSONException;
import org.json.JSONObject;

import java.util.Arrays;

public class MainActivity extends AppCompatActivity implements SensorEventListener {
  // TODO change this to production url
  private static final String URL = "http://54.183.78.200:3005/";

  private SensorManager sensorManager;
  private Sensor accelerometer;
  private Sensor gyroscope;
  private Sensor magnetometer;
  private WebView webView;

  private String accelerometerData = null;
  private String gyroscopeData = null;
  private String magnetometerData = null;
  private String sensorData = null;

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    initWebView();
    initSensors();
  }

  @SuppressLint("SetJavaScriptEnabled")
  private void initWebView() {
    webView = findViewById(R.id.webView);
    webView.loadUrl(URL);
    webView.getSettings().setJavaScriptEnabled(true);
  }

  private void passSensorData(String data) {
    webView.loadUrl(URL);
    webView.setWebViewClient(new WebViewClient() {
      @Override
      public void onPageFinished(WebView view, String url) {
        view.loadUrl("javascript:loadSensorData('" + data + "')");
      }
    });
  }

  private void initSensors() {
    sensorManager = (SensorManager) getSystemService(Context.SENSOR_SERVICE);
    accelerometer = sensorManager.getDefaultSensor(Sensor.TYPE_ACCELEROMETER);
    gyroscope = sensorManager.getDefaultSensor(Sensor.TYPE_GYROSCOPE);
    magnetometer = sensorManager.getDefaultSensor(Sensor.TYPE_MAGNETIC_FIELD);
    // if there is no sensor available json object with null values will be passed to webview
    if (accelerometer == null && gyroscope == null && magnetometer == null) {
      passSensorData(generateJSONSensorData());
    }
  }

  @Override
  public final void onSensorChanged(SensorEvent event) {
    if (event.sensor.getType() == Sensor.TYPE_ACCELEROMETER) {
      accelerometerData = Arrays.toString(event.values);
    } else if (event.sensor.getType() == Sensor.TYPE_GYROSCOPE) {
      gyroscopeData = Arrays.toString(event.values);
    } else if (event.sensor.getType() == Sensor.TYPE_MAGNETIC_FIELD) {
      magnetometerData = Arrays.toString(event.values);
    }

    if (generateJSONSensorData().equals(sensorData)) {
      return;
    }

    sensorData = generateJSONSensorData();
    passSensorData(sensorData);
  }

  private String generateJSONSensorData() {
    String result = "";
    JSONObject json = new JSONObject();
    try {
      json.put("accelerometer", accelerometerData);
      json.put("gyroscope", gyroscopeData);
      json.put("magnetometer", magnetometerData);
      result = json.toString();
    } catch (JSONException e) {
      e.printStackTrace();
    }
    return result;
  }

  @Override
  public final void onAccuracyChanged(Sensor sensor, int accuracy) {
    // Do something here if sensor accuracy changes.
  }

  @Override
  protected void onResume() {
    super.onResume();
    sensorManager.registerListener(this, accelerometer, SensorManager.SENSOR_DELAY_NORMAL);
    sensorManager.registerListener(this, gyroscope, SensorManager.SENSOR_DELAY_NORMAL);
    sensorManager.registerListener(this, magnetometer, SensorManager.SENSOR_DELAY_NORMAL);
  }

  @Override
  protected void onPause() {
    super.onPause();
    sensorManager.unregisterListener(this);
  }
}
```  

***  

## iOS (SwiftUI)

iOS snippet is developed in Xcode using SwiftUI.

The sensor data is obtained using the `CMMotionManager`.

Application is reading accelerometer, magnetometer and gyroscope data and is sending to WebView.

Sensor data is sent to the WebView as a JSON object.

```
{
  "accelerometer": "[x, y, z]",
  "gyroscope": "[x, y, z]",
  "magnetometer": "[x, y, z]"
}
```

### Complete basic Swift implementation  
**SensorDataModel**  
```swift
import Foundation

struct SensorDataModel: Encodable {
  var accelerometer: String
  var magnetometer: String
  var gyroscope: String
}
```
  
**WebViewModel**
```swift
import Foundation
import Combine

class WebViewModel: ObservableObject {
  var valuePublisher = PassthroughSubject<String, Never>()
}

// For identifying what type of url should load into WebView
enum WebUrlType {
  case localUrl, publicUrl
}
```
  
**MotionManager**
```swift
import Combine
import CoreMotion
import SwiftUI

class MotionManager: ObservableObject {  
  private var motionManager: CMMotionManager
  private var accelerometerData: [Double] = []
  private var magnetometerData: [Double] = []
  private var gyroData: [Double] = []
  @Published var sensorDataJSON: String = "";
      
  init() {
    self.motionManager = CMMotionManager()
    updateAccelerometerData()
    updateMagnetometerData()
    updateGyroscopeData()
  }
  
  func updateAccelerometerData() {
    self.accelerometerData.removeAll()
    if (!self.motionManager.isAccelerometerAvailable){
      return;
    }
    self.motionManager.accelerometerUpdateInterval = 1
    self.motionManager.startAccelerometerUpdates(to: .main) { (accelerometerData, error) in
      guard error == nil else {
        print(error!)
        return
      }
      
      if let aData = accelerometerData {
        self.accelerometerData.append(aData.acceleration.x)
        self.accelerometerData.append(aData.acceleration.y)
        self.accelerometerData.append(aData.acceleration.z)
      }
      self.createSensorDataJSON()
    }
  }
  
  func updateMagnetometerData() {
    self.magnetometerData.removeAll()
    if (!self.motionManager.isMagnetometerAvailable){
      return;
    }
    self.motionManager.magnetometerUpdateInterval = 1
    self.motionManager.startMagnetometerUpdates(to: .main) { (magnetometerData, error) in
      guard error == nil else {
        print(error!)
        return
      }
      
      if let magnetData = magnetometerData {
        self.magnetometerData.append(magnetData.magneticField.x)
        self.magnetometerData.append(magnetData.magneticField.y)
        self.magnetometerData.append(magnetData.magneticField.z)
      }
      self.createSensorDataJSON()
    }
  }
  
  func updateGyroscopeData() {
    self.gyroData.removeAll()
    if (!self.motionManager.isGyroAvailable){
      return;
    }
    self.motionManager.gyroUpdateInterval = 1
    self.motionManager.startGyroUpdates(to: .main) { (gyroData, error) in
      guard error == nil else {
        print(error!)
        return
      }
      
      if let gData = gyroData {
        self.gyroData.append(gData.rotationRate.x)
        self.gyroData.append(gData.rotationRate.y)
        self.gyroData.append(gData.rotationRate.z)
      }
      self.createSensorDataJSON()
    }
  }
  
  private func createSensorDataJSON() {
    let sensorData = SensorDataModel(accelerometer: self.accelerometerData.isEmpty ? nil : self.accelerometerData.description, magnetometer: self.magnetometerData.isEmpty ? nil : self.magnetometerData.description, gyroscope: self.gyroData.isEmpty ? nil : self.gyroData.description)
    do {
      let jsonData = try JSONEncoder().encode(sensorData)
      sensorDataJSON = String(data: jsonData, encoding: String.Encoding.utf8) ?? ""
    } catch {
      print(error)
    }
  }
}
```
  
**WebView**  

```swift
import Foundation
import UIKit
import SwiftUI
import Combine
import WebKit

struct WebView: UIViewRepresentable {
  var urlType: WebUrlType
  var url: String
  @ObservedObject var viewModel: WebViewModel
  
  // Make a coordinator to co-ordinate with WKWebView's default delegate functions
  func makeCoordinator() -> Coordinator {
    Coordinator(self)
  }
  
  func makeUIView(context: Context) -> WKWebView {
    // Enable javascript in WKWebView
    let webView = WKWebView(frame: CGRect.zero)
    webView.navigationDelegate = context.coordinator
    return webView
  }
  
  func updateUIView(_ webView: WKWebView, context: Context) {
    if urlType == .localUrl {
      // Load local website
      if let url = Bundle.main.url(forResource: "webApp", withExtension: "html") {
        webView.loadFileURL(url, allowingReadAccessTo: url.deletingLastPathComponent())
      }
    } else if urlType == .publicUrl {
      // TODO change this to production url
      if let url = URL(string: "http://54.183.78.200:3005/") {
        webView.load(URLRequest(url: url))
      }
    }
  }
  
  class Coordinator : NSObject, WKNavigationDelegate {
    var parent: WebView
    var valueSubscriber: AnyCancellable? = nil
    var webViewNavigationSubscriber: AnyCancellable? = nil
    
    init(_ uiWebView: WebView) {
      self.parent = uiWebView
    }
    
    deinit {
      valueSubscriber?.cancel()
      webViewNavigationSubscriber?.cancel()
    }
    
    func webView(_ webView: WKWebView, didFinish navigation: WKNavigation!) {
      /* An observer that observes 'viewModel.valuePublisher' to get value from TextField and
        pass that value to web app by calling JavaScript function */
      valueSubscriber = parent.viewModel.valuePublisher.receive(on: RunLoop.main).sink(receiveValue: { value in
        let javascriptFunction = "loadSensorData('\(value)');"
        webView.evaluateJavaScript(javascriptFunction) { (response, error) in
          if let error = error {
            print("Error calling javascript:loadSensorData()")
            print(error.localizedDescription)
          }
        }
      })
    }
  }
}
```
  
**ContentView**  

```swift
import SwiftUI

struct ContentView: View {
  @ObservedObject var viewModel = WebViewModel()
  @ObservedObject var motion: MotionManager = MotionManager()
  
  var body: some View {
    VStack {
      // WebView(urlType: .publicUrl, url: "https://...", viewModel: viewModel)
      WebView(urlType: .localUrl, url: "", viewModel: viewModel).onReceive(motion.$sensorDataJSON) { value in
        self.viewModel.valuePublisher.send(motion.sensorDataJSON)
      }
    }
  }
}
```

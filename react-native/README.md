# Webview Sensordata Communication

## React Native (iOS & Android)

React native requires `react-native-sensors` library to be installed in order to access sensor data.  
Also `rxjs` library is required as well to subscribe to sensor updates and emit values.

### Installation

`npm install react-native-sensors rxjs --save`

> *react-native-sensors library requires further installation setup for each platform. Detailed instructions can be found at [react-native-sensors documentation](https://react-native-sensors.github.io/docs/Installation.html#installation) page.

Sensor data is sent to the WebView as a JSON object.

```
{
  "accelerometer": "[x, y, z]",
  "gyroscope": "[x, y, z]",
  "magnetometer": "[x, y, z]"
}
```

### Complete react-native implementation

```tsx
import React, { useEffect, useRef } from 'react';
import {
  SafeAreaView,
} from 'react-native';
import { WebView } from 'react-native-webview';
import { 
  gyroscope,
  accelerometer,
  magnetometer,
  SensorTypes,
  setUpdateIntervalForType,
} from 'react-native-sensors';
import { combineLatest } from 'rxjs';
import { debounceTime } from 'rxjs/operators';

interface CoordinateSensorData {
  x: number; 
  y: number; 
  z: number; 
  timestamp: number;
}

interface SensorData {
  [Sensors.gyroscope]: CoordinateSensorData;
  [Sensors.accelerometer]: CoordinateSensorData;
  [Sensors.magnetometer]: CoordinateSensorData;
  [Sensors.barometer]: BarometerSensorData;
}

setUpdateIntervalForType(SensorTypes.gyroscope, 500);
setUpdateIntervalForType(SensorTypes.accelerometer, 500);
setUpdateIntervalForType(SensorTypes.magnetometer, 500);

export const App = () => {
  const webViewRef = useRef<null | WebView>(null);
  // TODO change this to production url
  const uri = 'http://54.183.78.200:3005/';

  const handleUpdateSensorData = (data: SensorData) => {
    const injected = `
      setTimeout(() => {
        window.loadSensorData(${JSON.stringify(data)});
      }, 100);
    `;
    webViewRef.current?.injectJavaScript(injected);
  }

  useEffect(() => {
    const sensors$ = combineLatest([gyroscope, accelerometer, magnetometer]).pipe(debounceTime(0));
    const subscription = sensors$.subscribe(([
      gyroscopeSensor, 
      accelerometerSensor, 
      magnetometerSensor, 
    ]: [CoordinateSensorData, CoordinateSensorData, CoordinateSensorData]) => {
      const data: SensorData = {
        gyroscope: gyroscopeSensor || null,
        accelerometer: accelerometerSensor || null,
        magnetometer: magnetometerSensor || null,
      }
      handleUpdateSensorData(data);
    });

    return () => {
      subscription.unsubscribe();
    }
  }, []);

  return (
    <>
      <SafeAreaView
        style={{
          width: '100%',
          height: '100%',
        }}
      >
        <WebView
          ref={webViewRef}
          source={{ uri }}
        />
      </SafeAreaView>
    </>
  );
};

export default App;
```
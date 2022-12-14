# Webview Sensordata Communication

## IONIC Capacitor (iOS & Android)

IONIC requires `@unicorns-and-unicorns/capacitor-sensors` library to be installed in order to access device sensor data.

### Installation

```bash
npm install @unicorns-and-unicorns/capacitor-sensors --save
npx cap sync
```

> *IONIC uses the HTML `iframe` element to display content. Due to the `Same-Origin` policy on iframes, we cannot directly inject data into the window of the iframe since the IONIC app and the iframe web application are not served from the same origin. To enable `Cross-Origin` communication we must use the `frameRef.contentWindow.postMessage()` function to send sensor data over to the iframe web application.
Sensor data is sent to the iframe as a JSON object.

```
{
  "accelerometer": "[x, y, z]",
  "gyroscope": "[x, y, z]",
  "magnetometer": "[x, y, z]"
}
```

### Complete IONIC Capacitor implementation

```tsx
import { useEffect, useCallback, useState } from 'react';
import { Sensors, SensorData } from '@unicorns-and-unicorns/capacitor-sensors';
export interface MappedSensorData {
  magnetometer: SensorData | undefined;
  gyroscope: SensorData | undefined;
  accelerometer: SensorData | undefined;
}
// TODO change this to production url
const IFRAME_URL = 'https://application-url';
const App: React.FC = () => {
  const [frameRef, setFrameRef] = useState<HTMLIFrameElement | null>(null);
  const [magnetometer, setMagnetometer] = useState<SensorData>();
  const [gyroscope, setGyroscope] = useState<SensorData>();
  const [accelerometer, setAccelerometer] = useState<SensorData>();
  useEffect(() => {
    Sensors.addListener('magnetometerChange', (res: SensorData) => {
      setMagnetometer({
        x: res.x,
        y: res.y,
        z: res.z,
      });
    });
    Sensors.addListener('gyroscopeChange', (res: SensorData) => {
      setGyroscope({
        x: res.x,
        y: res.y,
        z: res.z,
      });
    });
    Sensors.addListener('accelerometerChange', (res: SensorData) => {
      setAccelerometer({
        x: res.x,
        y: res.y,
        z: res.z,
      });
    });
  }, []);
  useEffect(() => {
    postSensorData({
      magnetometer,
      gyroscope,
      accelerometer,
    });
  }, [magnetometer, gyroscope, accelerometer]);
  function postSensorData(data: MappedSensorData) {
    if (frameRef?.contentWindow) {
      frameRef.contentWindow.postMessage({
        call: 'loadSensorData', 
        value: data,
      }, IFRAME_URL);
    }
  }
  const handleRef = useCallback((node) => {
    setFrameRef(node)
  }, []);
  
  return (
    <div>
      <iframe
        ref={handleRef}
        src={IFRAME_URL}
        frameBorder="0"
        width="100%"
        height="100%"
      />
    </div>
  );
}
export default App;
```

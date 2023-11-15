# SharedMemory

## Constructor

|   | SharedMemory Name   | Use | Flow           |
|---|---------------------|-----|----------------|
|   | GeomagicCalibration | √   | Unity to FleX  |
|   | HapticDeviceStatus  | ?   | no use but doc |
|   | Evaluation          | √   | FleX to Unity  |
|   | SerialPort          | √   | Unity to FleX  |
|   | Endoscope           |     |                |
|   | Tube                |     |                |
|   | InsturmentSwitch    |     |                |
|   | nerve               |     |                |
|   | FlocculeSoftBody    |     |                |
|   | dfspace             |     |                |
|   | fat                 |     |                |
|   | vessel1             |     |                |
|   | vessel2             |     |                |
|   | posterior           |     |                |
|   | annulus             |     |                |
|   | nucleus             |     |                |
|   | dura                |     |                |

### GeomagicCalibration

In `FleX` `GeomagicCalibration.h`
```C++
void Initialize() override
{
	if (m_useSharedMemory)
		m_sharedMemory = std::make_shared<SharedMemory>(mName, 20);
}
```
```C++
void Update()
{
	int offset = 0;
	m_sharedMemory->CopyFromSharedMemory(m_data, offset);
	if (!m_data.empty())
	{
		if (m_data[0] > m_count)
		{
			assert(m_data[0] = m_count + 1);
			cout << "Calibration count:" << m_data[0] << endl;
			m_count = m_data[0];
			UpdateCalibration();
			cout << "Calibration complete" << endl;
		}
	}

}
```

In `Unity` `Calibration.cs`

```C#
void Start()
{
    _sharedMemory = new SharedMemory(MemName, 20);
    _data = new int[1];
    _data[0] = 1;
}
```

```C#
public void SendData()
{
    long offset = 0;
    _sharedMemory.CopyToSharedMemory(ref _data, ref offset);
}
```

### Evaluation 

```C++
vector<float> evaluationData;
evaluationData.push_back(m_worldXZForce[0]);
evaluationData.push_back(m_worldXZForce[1]);
evaluationData.push_back(m_touchingSoftBodyType);
evaluationData.push_back(m_tornSoftBodyType);
evaluationData.push_back(m_tornProportion);
evaluationData.push_back(m_graspingStatus);
evaluationData.push_back(m_touchingVesselType);

int offset = 0;
m_sharedMemory->CopyToSharedMemory(&evaluationData[0], evaluationData.size(), offset);
m_sharedMemory->CopyToSharedMemory(bloodingPoints, offset);
m_sharedMemory->CopyToSharedMemory(tipsPoints, offset);
```
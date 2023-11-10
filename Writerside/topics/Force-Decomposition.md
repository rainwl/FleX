# Force Decomposition

![](forceDecomposition.png)

## Location

`Geomagic.h` **->** `HDCallbackCode HDCALLBACK stateCallback(void* userData)`

```C++
if (driver->m_worldXZForce)
{
    Vec3 motorForce = Rotate(rDevice, Vec3(1, 0, 0)) * forces[0][0] + 
                      Rotate(rDevice, Vec3(0, 1, 0)) * forces[1][0];
    
    (*driver->m_worldXZForce)[1] = motorForce.y - motorForce.z;
    (*driver->m_worldXZForce)[0] = motorForce.x;
    
    if (driver->m_leftHandFrame)
        (*driver->m_worldXZForce)[0] = -motorForce.x;
}
```

## Annotation

Code explain
: - `forces[0][0]` and `forces[1][0]` represent `input forces` for two different DoF,related to the 0th and 1st DoF, respectively.
- `Rotate(rDevice, Vec3(1, 0, 0))` performs rotation around the Y-axis with a force in the X-axis direction.
- `Rotate(rDevice, Vec3(0, 1, 0))` performs rotation around the X-axis with a force in the Y-axis direction.

`motorForce` is calculated by projecting the two rotated forces separately into the `global coordinate system`.

For the 0th dof, it rotates around the Y-axis, but the force is in the X-axis direction,
so the force needs to be projected onto the X-axis.

For the 1st dof, it rotates around the X-axis, but the force is in the Y-axis direction,
so the force needs to be projected onto the Y-axis.

`(*driver->m_worldXZForce)[1]` stores the Y - Z component of the rotational force, obtained by subtracting the force on the Y-axis and the force on the Z-axis.

**rDevice**


```C++
Quat d_orientationBase; //Posture of the device base
Quat orient;
Matrix33 mrot;
DeviceData m_hapticData;

hdGetDoublev(HD_CURRENT_TRANSFORM, driver->m_hapticData.transform);
hdGetDoublev(HD_CURRENT_JOINT_ANGLES, driver->m_hapticData.angle1);
hdGetDoublev(HD_CURRENT_GIMBAL_ANGLES, driver->m_hapticData.angle2);

for (int u = 0; u < 3; u++)
    for (int j = 0; j < 3; j++)
        mrot.cols[j][u] = driver->m_hapticData.transform[j * 4 + u];

orient = mrot;

Quat rDevice = driver->d_orientationBase * orient;
```

Multiplies the quaternion `d_orientationBase` with `orient` to calculate a new quaternion `rDevice` representing the combined orientation of the device base and the current orientation.
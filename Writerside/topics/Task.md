# Task

|   | Verification item                                                                                                                          | Conclusion                                                                                                                                                                                                                                                                    | Next plan                                                            |
|---|--------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------|
| * | External data driven                                                                                                                       | Neither eCAL nor SharedMemory will drive the emulation properly.                                                                                                                                                                                                              | View loadScene and Sofa related codes                                |
| * | eCAL runs a separate thread in SerialPort                                                                                                  | This causes Flex to rise 7-10ms late, and each Vector reset causes a crash                                                                                                                                                                                                    | Change vector to array                                               |
|   | InstrumentSwitch and SerialPort, whose Initialize is executed first                                                                        | The initialization of SerialPort was performed first, creating a shared memory, but when initialized with the Instrument, it did not receive a value from the shared memory, which is normal because SerialPort is executed in Update and accepts data from the shared memory | So look at the value of m_pos_endoscope                              |
|   | m_pos_endoscope needs to verify its execution flow                                                                                         | m_posEndoscope = TranslationMatrix(Point3(pos))*RotationMatrix(q);Cause NaN in sofa scene                                                                                                                                                                                     | run sofa scene                                                       |
|   | Use a real machine to debug FleX                                                                                                           |                                                                                                                                                                                                                                                                               |                                                                      |
|   | The haptic needs to be isolated                                                                                                            |                                                                                                                                                                                                                                                                               |                                                                      |
|   | Force feedback in sofa, force feedback in Geomagic, or any other parameter that requires external input, look for it and do the separation |                                                                                                                                                                                                                                                                               |                                                                      |
|   | Look at the codes in geomagic and sofa that deal with the reception of force feedback data                                                 |                                                                                                                                                                                                                                                                               |                                                                      |
|   | Make eCAL a separate scene                                                                                                                 |                                                                                                                                                                                                                                                                               |                                                                      |
|   | Make the Flex project a relative path                                                                                                      |                                                                                                                                                                                                                                                                               |                                                                      |
|   | You can run through the shared memory case to see all the shared memory function calls, in what order, and by whom                         |                                                                                                                                                                                                                                                                               | LoadScenes does not read this array to array, it needs to be changed |
|   | Just change the part of SerialPort where eCAL accepts data,vector, to a fixed-length array, and then call the std::array                   | No additional performance cost                                                                                                                                                                                                                                                |                                                                      |
|   | Set the std::array address of tool_data in eCAL in SerialPort to LoadScene. Those two places need to be changed                            |                                                                                                                                                                                                                                                                               |                                                                      |
|   | What does the PostInitialize method do                                                                                                     |                                                                                                                                                                                                                                                                               |                                                                      |
|   | I can figure out how SharedMemory works, and then I can just replace SharedMemory                                                          |                                                                                                                                                                                                                                                                               |                                                                      |
|   | Encapsulate eCAL and use it as if calling shared memory.                                                                                   |                                                                                                                                                                                                                                                                               |                                                                      |
|   | eCAL has performance problem after using std::array<float,40>,need to check                                                                |                                                                                                                                                                                                                                                                               |                                                                      |
|   | Whether haptic needs to be separated from FleX                                                                                             |                                                                                                                                                                                                                                                                               |                                                                      |
|   | Analyze the driving logic of SOFA                                                                                                          |                                                                                                                                                                                                                                                                               |                                                                      |
 
## Learn

### A->a = &B->b;

```C++
geo_magic->m_worldXZForce = &evaluation->m_worldXZForce;
```

这行代码执行了一个赋值操作，将`evaluation`对象的成员变量`m_worldXZForce`的地址赋给了`geo_magic`
对象的成员变量`m_worldXZForce`。以下是解释：

- `geo_magic`是一个对象或指针，它有一个成员变量（或者可能是指针）`m_worldXZForce`。
- `evaluation`也是一个对象或指针，它有一个成员变量（或者可能是指针）`m_worldXZForce`。
- `&evaluation->m_worldXZForce`表示获取`evaluation`对象的`m_worldXZForce`成员的地址。

所以，这行代码的目的是将`evaluation`对象中`m_worldXZForce`的地址赋值给了`geo_magic`对象中的`m_worldXZForce`
。这样一来，两个对象的`m_worldXZForce`成员现在指向相同的内存位置，它们在程序中的改变将相互影响。

### GetProcAddress dll

```C++
// 这段 C++ 代码在Windows 的动态链接库（DLL）模块中查找一个名为 "createServoLoop" 的函数，并获取其地址
// m_hdPhantomDriverDLL 是一个包含 DLL 模块的句柄。这个句柄是通过 LoadLibrary 或者 GetModuleHandle 这样的函数获取的。
// GetProcAddress 是一个 Windows API 函数，它获取 DLL 模块中指定函数的地址.
// "createServoLoop" 是想要查找的函数名字.
// (int(*)()) 是一个函数指针类型转换。这表示该函数没有任何参数并返回一个整数。虽然 GetProcAddress 返回的是 FARPROC 类型，但通常我们会将其转换为正确的函数指针类型以便更安全地使用它。
// createServoLoop 是已经类型转换过的函数的地址存储变量。
// 总的来说，这段代码的功能是获取动态链接库中名为 "createServoLoop" 的函数的地址，并将其存储在 createServoLoop 变量中。通过这种方式，您可以在运行时调用 DLL 中的函数，而不需要在编译时知道这些函数的存在.
createServoLoop = (int(*)()) GetProcAddress(m_hdPhantomDriverDLL, "createServoLoop");//创建服务循环
```

### _stdcall

```C++
	//该函数似乎被设计成用作多线程应用程序的线程入口点或回调函数
	//该函数声明为静态，并且预期遵循_stdcall调用约定
	//总体而言，这个函数似乎是一个更大程序或系统的一部分，涉及与某个设备（可能是Geomagic触觉设备）交互，
	//以更新其状态并检索有关其关节角度和位置的信息。鉴于其结构和static关键字的使用，该函数可能被设计为在单独的线程中运行
	//它采用_stdcall调用约定，这是一种在Windows平台上常见的函数调用约定。
	static int _stdcall ServoRoutine(void *pParam)
	{
		GeomagicCalibration *geomagicCalibration = (GeomagicCalibration*)pParam;//将pParam指针强制转换为GeomagicCalibration类型的指针。这表明pParam预计指向GeomagicCalibration类或结构的实例
		geomagicCalibration->update_phantom(geomagicCalibration->m_nDevice);
		geomagicCalibration->get_phantom_joint_angles(geomagicCalibration->m_nDevice, geomagicCalibration->angles);
		geomagicCalibration->get_phantom_pos(geomagicCalibration->m_nDevice, geomagicCalibration->XYZ);
		return 0;
	}
```

### Tear

This code appears to be a part of a larger program and seems to be dealing with the tearing or splitting of tetrahedral meshes. Let's break down the code into smaller sections to understand its functionality.

### Section 1: Triangle Classification and Vertex Index Collection
```C++
// Create 3 vectors to store 3 types of triangles indices
// surface triangles
vector<int> surfaceTriIndices;
// triangles need to be split
vector<int> splitTriIndices;
// internal triangles
vector<int> internalTriIndices;
// all vertices indices of tetrahedrons
set<int> allVerticesIndices;

// Iterate all tetrahedrons, put all triangles into 3 different vectors and put all vertices indices into set
for (int i = 0; i < tetraIndices.size(); ++i)
{
    Tetrahedron &tet = mTetras[tetraIndices[i]];
    for (int j = 0; j < 4; ++j)
    {
        int triIndex = tet.tris[j];
        Triangle &tri = mTris[triIndex];

        // ... (skipping some conditions)

        if (tri.IsBoundary())
        {
            surfaceTriIndices.push_back(triIndex);
            continue;
        }

        // ... (skipping some conditions)

        if (!theOtherTetra.m_isSplit)
        {
            splitTriIndices.push_back(triIndex);
        }
        else
        {
            internalTriIndices.push_back(triIndex);
            tri.m_isInVoxel = true;
        }
    }
}
```
Explanation:
- Three vectors (`surfaceTriIndices`, `splitTriIndices`, `internalTriIndices`) are created to store indices of different types of triangles.
- The loop iterates through tetrahedrons and classifies triangles into surface, split, or internal categories based on certain conditions.
- All unique vertex indices of tetrahedra are collected in the `allVerticesIndices` set.

### Section 2: Finding Split Vertices
```C++
// Set to store split vertices indices
set<int> splitVerticesIndices;

// Iterate all vertices indices calculated above
for (set<int>::iterator it = allVerticesIndices.begin(); it != allVerticesIndices.end(); it++)
{
    set<int> &tetrahedronIndices = m_tetraIDsPerVertices[*it];
    for (set<int>::iterator tit = tetrahedronIndices.begin(); tit != tetrahedronIndices.end(); tit++)
    {
        if (!mTetras[*tit].m_isSplit)
        {
            splitVerticesIndices.insert(*it);
            break;
        }
    }
}
```
Explanation:
- This section identifies vertices that need to be split based on the information gathered earlier.
- It iterates through vertices and checks if any tetrahedron containing the vertex is marked as non-split. If found, the vertex is marked as a split vertex.

### Section 3: Creating Vertex Map and Updating Data Structures
```C++
// Map to map each vertex from the original to the new
std::map<int, int> newVerticesMap;
// Iterate through split vertices and update data structures
for (set<int>::iterator it = splitVerticesIndices.begin(); it != splitVerticesIndices.end(); it++)
{
    // ... (updating newVerticesMap, vertexCopies, and other data structures)
}

// Update triangles and tetrahedrons based on splitting
// ... (updating splitTriangles, mTris, tetrahedron vertices, etc.)
```
Explanation:
- A map (`newVerticesMap`) is created to map original vertices to new ones after splitting.
- The code updates various data structures based on the splitting process, such as creating new triangles, updating vertex indices, and handling texture mapping.

### Section 4: Calculating Torn Particles and Updating Rigid Bodies
```C++
// Calculate torn particles based on m_tearParticles
// ... (updating m_tornParticles and clearing m_tearParticles)

// Update rigid bodies based on split vertices
// ... (updating rigid body information, removing connections, and creating new rigid bodies)
```
Explanation:
- The code processes torn particles and updates rigid bodies based on the splitting information.
- It calculates which particles are torn and updates the rigid bodies accordingly.

In summary, this code is responsible for tearing or splitting a tetrahedral mesh, updating the mesh data structures, and handling the impact on rigid bodies. The process involves classifying triangles, identifying split vertices, creating new vertices, updating triangles, and managing rigid bodies.


这代码是一部分关于软体动力学或网格处理的程序，特别是用于处理物体撕裂效应。代码实现了一个名为 void Tear 的函数，该函数用于处理物体的撕裂，涉及到的数据结构包括三角形、四面体、顶点等。这个过程通常在物理模拟、游戏开发或者三维图形处理中使用。下面是对该代码段的详细解释：
函数 void Tear
参数
tetraIndices：四面体的索引数组。
vertexCopies：顶点副本的数组。
triangleUpdates：三角形更新的数组。
newTriangles：新创建的三角形的数组。
主要步骤
分类三角形：将所有四面体的三角形分为表面三角形（surfaceTriIndices）、需要分裂的三角形（splitTriIndices）和内部三角形（internalTriIndices）。同时收集所有顶点索引。
查找需要分裂的顶点：基于四面体的连接关系，找出所有需要分裂的顶点（splitVerticesIndices）。
映射新旧顶点：创建从原始顶点到新分裂顶点的映射（newVerticesMap）。
处理分裂的三角形：对于需要分裂的三角形，创建新的三角形实例，更新相关四面体和顶点的索引。
更新三角形和四面体：将新分裂的三角形添加到三角形数组中，并更新所有四面体的顶点索引。
计算需要分裂的粒子：基于四面体的撕裂状态，更新需要分裂的粒子集（m_tornParticles）。
更新刚体：处理不是新分裂但需要撕走的四面体的内部点，创建新的刚体并更新相关的索引和状态。
示意图解释
为了更直观地理解这个过程，我会创建一个示意图，展示物体在撕裂过程中四面体和三角形的变化。图中将展示四面体的初始状态、分类三角形的过程、顶点的分裂和新三角形的创建等关键步骤。这将帮助您更好地理解上述代码的功能和流程。
让我生成这张示意图。

![](Tear.png)

上面的图表展示了代码中描述的3D对象撕裂过程。它依次展示了：
初始状态：由四面体和三角形组成的3D对象。
分类三角形：展示了不同类型的三角形，表面三角形用蓝色表示，需要分裂的三角形用红色表示，内部三角形用绿色表示。
分裂顶点：聚焦于分裂的顶点，新顶点用黄色标记。
形成新三角形：展示了分裂后形成的新三角形，显示了网格结构的变化。
整个过程以清晰、步骤分明的方式展示，有助于理解代码在物理模拟或3D图形环境中的功能。


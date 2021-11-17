### 入门
> 使用的是GLAD库和GLFW库
1. 程序开始前需要实例化GLFW窗口
 `glfwInit()`进行初始化
 `glfwWindowHint(Const,value)`
 2. 创建window对象
`GLFWwindow* window = glfwCreateWindow(800,600,"title",NUILL,NULL)`
`glfwMakeContextCurrent(window);`
3. GLAD用来管理Opengl的函数指针。需要初始化GLAD
`gladLoadGLLoader((GLADloadproc)glfwGetProcAddress))` 返回状态值
4. 设置视口
`glViewport(0,0,800,600)`，左下角和宽度高度构成左边四个参数
视口是Opengl渲染窗口的尺寸大小
改变窗口大小时，视口需要被调整，通过回调函数callback function的方式
`void framebuffer_size_callback(GLFWWindow* window, int width, int height)`
函数具体实现就是`glViewport(..)`函数调用
使用时需要注册函数`glfwSetFramebufferSizeCallback(window,framebuffer_size_callback);`
 5. 渲染循环
 6. OpenGL仅当3D坐标在-1到1之间才处理（标准设备坐标）
 7. 通过顶点缓存对象VBO管理顶点信息
利用缓冲ID生成一个VBO对象
```C++
unsigned int VBO;
glGenBuffers(1,&VBO);
//绑定缓冲
glBindBuffer(GL_ARRAY_BUFFER,VBO);
```
绑定完成后，调用ARRAY_BUFFER缓冲被用于配置序号为VBO的缓冲。 
再将顶点数据载入缓冲中
`glBufferData(GL_ARRAY_BUFFER,sizeof(vertices),vertices,GL_STATIC_DRAW);`
GL_STATIC_DRAW参数指定数据被管理的方式
8. GLSL
9. 编译GLSL文本
```C++
unsigned int vertexShader;
vertexShader = glCreateShader(GL_VERTEX_SHAER);
glShaderSource(vertexShader,1,&vertexShaderSource,NULL);
//shadersource即为GLSL文本,第二个参数指定传递的字符串数量
glCompileShader(vertexShader); 
```
可以通过`glGetShaderiv()`获取编译状态信息，通过`glGetShaderInfoLog()`得到具体错误信息
10. 片段着色器
编译片段着色器只需要将常数改为GL_FRAGMENT_SHAER即可
11. 着色器程序
将编译后的Fragment shader和vertex shader进行链接，形成着色器程序对象。
```C++
unsigned int shaderProgram;
shaderProgram = glCreateProgram();
glAttachShader(shaderProgram,vertexShader);
glAttachShader(shaderProgram,fragmentShader);
glLinkProgram(shaderProgram);
```
通过`glUseProgram(shaderProgram)`激活程序，此时可以`glDelteShader()`删除顶点和片段着色器
12. 注意：从上文若干个程序的声明方式可知，Opengl反复使用ID的方式作为程序索引，不直接对类对象进行操作
13. 链接顶点属性
`glVertexAttribPointer(0,3,GL_FLOAT,GL_FALSE,3*sizeof(float),nullptr);`
0表示要配置的顶点属性，在GLSL中设定location=0，所以这里为0
3是因为顶点由vec3表示
GL_FALSE表明不要被标准化为-1~1之间
3\*sizeof(float)为步长
`glEnableVertexAttribArray(0);` 激活属性
14. 顶点数组对象VAO
绑定时绑定VAO对象，就能避免每一个物体绑定一次的缓冲对象，配置属性的麻烦
VAO储存所有状态配置，并自动调用（VAO储存的是设置，不是数据）
`unsigned int VAO;` 
`glGenVertexArrays(1,&VAO);`
`glBindVertexArray(VAO)` 
15. 索引缓冲对象
用于指定绘制顺序（每一个三角形顶点对应的vertices中的序号）
序号顺序储存在indices[]数组中
创建缓冲对象
```C++
unsigned int EBO;
glGenBuffers(1,&EBO);
```
在bind buffer and data时，选择GL_ELEMENT_ARRAY_BUFFER类型
最后使用`glDrawElements`替换`glDrawArrays`更换绘画模式（略）
16
> `glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, EBO);`
`glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, 0);`
第一个参数指定了我们绘制的模式，这个和glDrawArrays的一样。第二个参数是我们打算绘制顶点的个数，这里填6，也就是说我们一共需要绘制6个顶点。第三个参数是索引的类型，这里是GL_UNSIGNED_INT。最后一个参数里我们可以指定EBO中的偏移量（或者传递一个索引数组，但是这是当你不在使用索引缓冲对象的时候），但是我们会在这里填写0。glDrawElements函数从当前绑定到GL_ELEMENT_ARRAY_BUFFER目标的EBO中获取索引。这意味着我们必须在每次要用索引渲染一个物体时绑定相应的EBO，这还是有点麻烦。不过顶点数组对象同样可以保存索引缓冲对象的绑定状态。**VAO绑定时正在绑定的索引缓冲对象会被保存为VAO的元素缓冲对象**。绑定VAO的同时也会自动绑定EBO。
(意思是应用设定时只要bind VAO就能完成一次设置？）
17. 线框模式绘制
`glPolygonMode(GL_FRONT_AND_BLACK,GL_LINE)` 配置线框模式绘制
18. 绘制完成后，需要删除VAO,VBO和program
19. 程序结构
    +   初始化（GLFW,GLAD,GLFWwindow）
    +   设置VAO,VBO,EBO
    +   构造Shader
    +   渲染循环
        +   输入事件
        +   启动shaderProgram
        +   swapBuffer

#### Shaders
16. GLSL
着色器开头声明**版本**，接着声明**输入**和**输出**变量，**uniform**和**main**函数。
17. 数据类型
int,float,double,uint & bool
18. 容器类型: Matrix and vec 
`vecn,bvecn,ivecn,uvecn,dvecn` vecn默认为float分量
19. glsl进行vertex shader编写时，需要`layout(location=0)`，why？
指定每一个输入变量的读取顺序，如果有多个属性，则layout(location = 1) in vec3 ... 以此类推
vertex shader的out和fragment shader的in需要一致
20. uniform
从CPU向GPU的着色器发送数据的方式为uniform
uniform是**全局的**(Global)，即不可重复，可以被任何shader访问
uniform变量值的修改必须在shaderProgram运行时，但是定位uniform不需要program运行
21. **通常将Shader封装为一个类，包含ID和构建shader的函数，方便使用，网页中给出了完整框架**
22. uniform需要从程序中传入数据，需要先找到uniform的索引（位置值），然后更新uniform的值
`int uniformLocation = glGetUniformLoaction(program,"ourcolor")` ourcolor是shader中uniform变量名
`glUniform4f(vertexColorLocation,0.0f,greenValue,0.0f,1.0f)` 其中greenvalue为C程序中的变量，注意**在更新uniform前必须使用程序(glUseProgram)** ，4f表示需要读入4个float作为值，这里读入rgba值。
`fv`表示float数组，`ui`表示一个uint类型
23. 可以给顶点分配属性，这样在VAO绑定属性时需要绑定两次属性，一次为position，一次为color
`glVertexAttribPointer(1,3,GL_FLOAT,GL_FALSE,6*sizeof(float),(void*)(3*sizeof(float));` 
`glEnableVertexAttribArray(1);`
1：**配置属性位置值为1的顶点属性**，属性有3个float大小。第5个属性表示跨度，最后一个表示属性对应数据的偏移值。位置顶点属性在前，所以偏移量为0，颜色属性在后，所以偏移`3*sizeof(float)` 
#### 纹理
1. 纹理坐标
2. 环绕方式：四种
REPEAT,MIRRORED_REPEAT,CLAMP_TO_EDGE,CLAMP_TO_BORDER
可以为纹理轴（s，t，r）单独设置环绕方式
`glTexParameteri(GL_TEXTURE_2D,GL_TEXTRUE_WRAP_S,GL_MIRRORED_REPEAT);` 
3. 纹理过滤
GL_NEAREST & GL_LINEAR
当纹理分辨率较小时，采用纹理过滤的方式解决纹理投影问题，默认NEAREST
临近过滤产生锯齿，线性插值过滤会让图像模糊
`glTexParameteri(GL_TEXTURE_2D,GL_TEXTURE_MIN_FILTER,GL_NEAREST)`
Min表示进行缩小操作时的纹理过滤设置，MAG表示放大
4. Mipmap
`GL_NEAREST_MIPMAP_NEAREST` 
放大过滤不设置为Mipmap，缩小过滤可以设置为Mipmap
5. 加载纹理
`stb_image.h`库
`unsigned char * data = stbi_load("container.jpg",&width,&height,&nrChannels,0)`
6. 生成纹理
`glGenTextures(1,&texture)`，texture为ID，1表示生成纹理的数量，第二个参数表示一个ID的数组（这里只有一个ID）
`glBindTexture(GL_TEXTURE_2D,texture);`
`glTexImage2D(GL_TEXTURE_2D, 0, GL_RGB, width, height, 0, GL_RGB, GL_UNSIGNED_BYTE, data); glGenerateMipmap(GL_TEXTURE_2D);`
glTexImage2D的第二个参数表示多级渐远纹理级别，第五个参数恒为0，
后边两个GL_RGB和GL_UNSIGNED_BYTE表示源图的类型和存储方式
使用完data后要释放`stbi_image_free(data)`
7. 纹理应用
![53092895c36aba4eeb6c557a58b8a9dc.png](en-resource://database/1078:1)
这里需要增加一个glVertexAttribPointer的设置
**这里要注意，纹理坐标只有两个，glEnableVertexAttribArray()参数为2**
8. GLSL中的纹理
在fragment shader中通过uniform变量传递一个sampler2D类型的ourTexture变量，`FragColor = texture(ourTexture,TexCoord)` 取texture颜色
设置sampler2D变量，使用`glUniform1i`函数，可以给sampler分配位置值，这样能够在一个frag shader中设置多个纹理，一个纹理的位置被称为一个**纹理单元（Texture Unit）**，默认为0，
在绑定纹理时激活指定纹理单元`glActiveTexture(GL_TEXTURE0)` (Opengl支持16个纹理单元，从TEXTURE0-15,这几个宏定义数据上连续，可以用循环) 
9. 对多个纹理单元，在render时
`glActiveTexture(GL_TEXTURE1);`
`glBindTexture(GL_TEXTURE_2D, texture2);`
10. `glPixelStorei(GL_UNPACK_ALIGNMENT, 1);`防止图片因为像素问题产生倾斜
11. texcoord能否设置为0-2区间？会产生什么变化？

#### Transformations
1. GLM数学库
#### Coordinate Systems 
1. 坐标系统
    + 局部空间，物体空间 Object Space
    + 世界空间 WorldSpace
    + 观察空间 ViewSpace (Eye Space)
    + 剪裁空间Clip Space
    + 屏幕空间 Screen Space 
 2. 正交投影
`glm::ortho(left,right,bottom,top,near plane,far plane)`产生正交投影矩阵
3. 透视投影
`glm:: mat4 proj = glm::perspective(glm::radians(45.0f),(float)width/(float)height,0.1f,100.0f);` 
参数分别为fov,aspect ratio,near plane, far plane. 
4. 三维变换
旋转：`model = glm::rotate(model,glm::radians(-55.0f),glm::vec3(1.0f,0.0f,0.0f)`  model为模型矩阵，mat4类型，将顶点坐标变换到世界坐标
平移：`mat4 view; view = translate(view,glm::vec3(0.0f,0.0f,-3.0f);`
投影:  `mat4 projection; projection = perspective(radians(45.0f),screenWidth/screenHeight,0.1f,100.0f);`
在vertex shader定义（glsl文本）中，引入uniform类型的mat4变量model,view,projeciton，在void main()中相乘，得到最后的vertex position
调用shader时，传入uniform变量`int modelloc = glGetUniformLocation(ourShader.ID,"model"));` & `glUniformMatrix4fv(modelLoc,1,GL_FALSE,value_ptr(model));`
这里只传入了model，其他类似
5. 注意，opengl中，标准化设备空间（-1~1 Cube）是左手系
6. Z buffer
深度测试默认关闭，需要手动打开`glEnable(GL_DEPTH_TEST);` 
每一帧开始时清除缓冲（和颜色缓冲相同） `glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);` 
#### Camera
1. z轴指向屏幕外，相机后移意味着z轴正向移动，相机看向z轴负方向
2. 摄像机位置`vec3 cameraPos = vec3(0.0f,0.0f,3.0f)` 
3. 摄像机方向`vec3 cameraTarget = vec3(0.0f,0.0f,0.0f);` 
表示摄像机指向的方向
`vec3 cameraDirection = normalize(cameraPos - cameraTarget);` 
注意顺序，摄像机看向的方向是-z方向
4. **right Vector**: the x axis in camera space 
需要先用Up vector定义right vector，通过叉乘得到
`vec3 up = vec3(0.0f,1.0f,0.0f);` 
`vec3 cameraRight = normalize(cross(up,cameraDirection));` 
注意up不是相机的up，是随便定义的一个向y轴正方向的向量（因为cameraDirection在xoy平面上）
5. 上轴
`vec3 cameraUp = cross(cameraDirection,cameraRight);` 
6. Look at 矩阵
将以上的内容变成一个平移矩阵和变换矩阵
![56c2ed52177e0dcf3db149d234370626.png](en-resource://database/1086:1)
```python
mat4 view;
view = lookAt(vec3(0.0f,0.0f,3.0f),
            vec3(0.0f,0.0f,0.0f),
            vec3(0.0f,1.0f,0.0f));
```
传入一个位置、目标和上向量，创建lookat矩阵
view变量通过uniform类型变量传入shader中(vertex shader)，从而影响成像结果
可以传入随时间变化的相机位置从而模拟运动
7. 相机的移动
可以设置`cameraPos,camerFront,cameraUp`
`view  = lookAt(cameraPos,cameraPos+cameraFront,cameraUp);` 
保证了camera看向的方向不会变化
8. 平滑的运动
要更加平滑的运动，就要解决不同镇渲染时间带来的问题
```C++
float deltaTime = 0.0f;
float lastFrame = 0.0f;
float currentFrame = glfwGetTime();
deltaTime = currentFrame - lastFrame;
lastFrame = currentFrame;
//....
void processInput(GLFWwindow* window)
{
    float cameraSpeed = 2.5f * deltaTime;
    //....
}
```
**如果渲染时间长，就需要更多的位移，保证移动的平稳**
9. 视角移动：通过鼠标改变cameraFront向量
10. Euler angle
Pitch,Yaw and Roll 
`direction.y = sin(glm::radians(pitch))`
`direction.x = cos(glm::radians(pitch)) * cos(glm::radians(yaw));` 
`direction.z = cos(glm::radians(pitch)) * sin(glm::radians(yaw));`


#### 材质
1. material类的设计(shader中)
    +   ambient 
    +   diffuse 
    +   specualr 
    +   shiness（反光度），即为specular的指数项
 通过material中的属性，改写简单的shader，改进phong shading的效果
 opengl中设置的时候可以分别设置material类的成员
 2. 光的属性(shader中）
    +   position
    +   ambient
    +   diffuse
    +   specular
实际的分量效果为light的属性乘以material的属性
#### 光照贴图
1. 反射贴图diffuse map 
用贴图代替diffuse项和ambient
    +   sampler2D diffuse 和一个texcoord（非uniform），通过texture(diffuse,texcoord)来调用
2. 镜面光贴图：用贴图上的颜色乘以对应的光源镜面强度，让木头发光不再明显
3. 采样镜面光贴图：把specular也改成sample2d类型
4. 法线贴图和反射贴图（更好的效果：凹凸状，不同法线效果）
#### 投光物 casters
1. 平行光：用direction替代position
2. 光衰减：light结构体中加入linear和quadratic参数
3. 聚光：spotlight（手电筒）
略
3. 合并结果
4. 多光源
#### 网格
1. vertex顶点：position，normal，texcoord
2. texture结构体：id，type，整理纹理数据
3. mesh结构体
4. 优化纹理：将已经加载过的纹理存储，避免多次加载同一种纹理，造成资源浪费
修改Texture类，使之包含path 
```C++
struct Texture{
    unsigned int id; 
    string type;
    aiString path; //用于和其他纹理进行比较
}
```
用一个texture_loaded数组记录已经加载过的所有纹理
#### 深度测试
1. 开启深度测试glEnable(GL_DEPTH_TEST);
2. 需要每次renderloop开始时清空DEPTH_BUFFER_BIT
3. 可以通过`glDepthMask(GL_FALSE)`设置不更新深度缓存
4. 深度测试函数
glDepthFunc(GL_LESS)，设置特定比较运算符 
5. 深度缓冲可视化
`gl_FragCoord`（fragment shader中）包含z值特定深度值，可以将这个值传给fragColor达成可视化
6. z-fighting
#### 模板测试stencil test
1. 启动模板测试glEnable(GL_STENCIL_TEST)
2. 需要glClear中清楚GL_STENCIL_BUFFER_BIT
3. 允许设置掩码 glStencilMask(0xFF);
4. 模板函数 glStencilFunc(GLenum func, GLint ref,GLuint mask) 
5. 物体轮廓
    +   绘制物体前将模板函数设置为GL_ALWAYS，模板缓冲更新为1
    +   渲染物体
    +   禁用模板写入以及深度测试
    +   缩放物体
    +   使用片段着色器输出一个边框颜色
    +   再次绘制物体，只在片段模板值不等于1时绘制
    +   再次启用模板写入和深度测试
（略）
#### 混合 Blending
1. 混合是实现透明度的方法
2. 在纹理中加入alpha通道，实现透明贴图glTexImage2D(...,GL_RGBA)
并修改shader中的texture相关函数（已经是四通道）
3. shader中有discard命令，设置为a<0.1时discard 
4. 需要将环绕方式设置为GL_CLAMP_TO_EDG消除白色边框
5. 开启GL_BLEND
6. glBlendFunc设置因子（源的因子和叠加对象的因子）
7. glBlendFuncSeparate为RGB和alpha通道设置不同选项
8. 渲染半透明纹理
opengl只会自动混合当前颜色和zbuffer中的颜色，因此因为渲染顺序的原因，一些部分不会被着色
必须手动按照深度依次绘制
9. 第8点的解决方案：先绘制不透明物体，然后根据排序绘制所有透明的物体
通过map容器，实现对distance的排序
10. **次序无关透明度**
#### 面剔除 face culling 
1. 通过分析顶点数据的**环绕顺序**，判断哪些是正面和背面
**逆时针**顶点定义的三角形被理解为**正向三角形**
2. glEnable GL_CULL_FACE 
3. glCullFace
    +   GL_FRONT
    +   GL_BACK 
    +   GL_FRONT_AND_BACK 
设置要剔除的是正面还是背面
通过glFrontFace定义顺时针还是逆时针为正向面
#### 帧缓冲
1. 自定义帧缓冲
2. FBO对象
3. 
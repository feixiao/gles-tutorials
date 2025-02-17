## Draw a 3D Cube

在此示例中，将在 x、y、z 轴和轴的中心上绘制 3D 立方体。由于这是通过扩展前面的示例编写的，因此请参阅前面的示例文档以了解任何重叠的部分。

### Vertex position과 color를 위한 Buffer 생성

首先要看的是“initOpenGL”函数的添加。

```C++
    char vShaderStr[] =
    		"precision mediump float;                 \n"
    		"uniform mat4 u_mvpMat;\n"
    		"attribute vec4 a_position;                \n"
    		"attribute vec4 a_color;                  \n"
    		"varying vec4 v_color;                    \n"
    		"void main()                              \n"
    		"{                                        \n"
    		"   gl_Position = u_mvpMat * a_position;              \n"
    		"   v_color = a_color;                    \n"
    		"}                                        \n";
```

正如您在代码中看到的，您可以看到这次着色器源包含几行。这些代码用于：

- u_mvpMat : 通过围绕 x 和 y 轴旋转立方体来精确查看立方体形状的变量
- a_color, v_color : 用于以不同颜色绘制立方体每个面的变量

`u_mvpMat` 对于变量来说，主代码中进行变换的矩阵运算的结果被传递给着色器来表达变换。 `a_color` 是用户输入的颜色矩阵值，该值被复制为v_color值，并将该值传递给片段着色器。然后，像下面的代码一样：
```C++
    char fShaderStr[] =
    		"varying lowp vec4 v_color;                        \n"
    		"void main()                                     \n"
    		"{                                               \n"
    		"   gl_FragColor = v_color;                      \n"
    		"}                                               \n";
```  

片段的颜色使用传递的值来表示，无需任何转换。接下来添加的是为顶点位置和颜色创建缓冲区的部分。

在前面的示例中，顶点信息存储在 clint 内存中，当调用 glDrawArrays 或 glDrawElemets 函数时，必须将这些数据复制到图形内存中。但是，如果使用图形内存中缓存的数据而不是每次绘制时复制顶点信息，则可以提高渲染性能。 **顶点缓冲对象 (VBO)** 用于此目的。

```C++
    glGenBuffers(1, &vertexID);
    glBindBuffer(GL_ARRAY_BUFFER, vertexID);
    glBufferData(GL_ARRAY_BUFFER, sizeof(vCube), vCube, GL_STATIC_DRAW);

    glGenBuffers(1, &colorID);
    glBindBuffer(GL_ARRAY_BUFFER, colorID);
    glBufferData(GL_ARRAY_BUFFER, sizeof(colors), colors, GL_STATIC_DRAW);

    glGenBuffers(1, &axisVertexID);
    glBindBuffer(GL_ARRAY_BUFFER, axisVertexID);
    glBufferData(GL_ARRAY_BUFFER, sizeof(vAxis), vAxis, GL_STATIC_DRAW);

    glGenBuffers(1, &axisColorID);
    glBindBuffer(GL_ARRAY_BUFFER, axisColorID);
    glBufferData(GL_ARRAY_BUFFER, sizeof(axisColor), axisColor, GL_STATIC_DRAW);
```
初始化VBO分为三个步骤，如上面的代码所示。

- glGenBuffers: 返回缓冲区对象名称。用作一种标识符。
- glBindBuffers: Use a previously created buffer as the active VBO.
- glBufferData: 将客户端内存中的数据复制到 VBO。
通过使用这些复制的数据，您可以减少每次从客户端绘制到图形内存时复制数据的不便。

### X,Y,Z 축 과 3D Cube 그리기

使用上述缓冲区绘制 x、y 和 z 轴。

```C++
	glBindBuffer(GL_ARRAY_BUFFER, axisVertexID);
	glEnableVertexAttribArray(positionLoc);
	glVertexAttribPointer(positionLoc, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), 0);

	glBindBuffer(GL_ARRAY_BUFFER, axisColorID);
	glEnableVertexAttribArray(colorLoc);
	glVertexAttribPointer(colorLoc, 4, GL_FLOAT, GL_FALSE, 4 * sizeof(float), 0);
```

由于从正面观看时无法判断它是否为 3D，因此请应用如下所示的变换和投影。有关这方面的更多详细信息将在下一个示例中介绍。

```C++
	glExtRotateX(fRotateX, mRotX);
	glExtRotateY(fRotateY, mRotY);
	glExtMultiply(model, mRotX, mRotY);
	glExtFrustum(view, -2.0, 2.0, -2.0 * aspect, 2.0 * aspect, -2.0, 2.0);
	glExtMultiply(mvp, view, model);
	glUniformMatrix4fv(mvpLoc, 1, GL_FALSE, mvp);
```

这里还要看一下着色器变量中的uniform、attribute和variable，它们的定义如下。

**Uniform**
一个变量，用于存储通过 OpenGL ES API 从应用程序传递到 Shader 的只读值。 Uniform变量在Vertex Shader和Fragment Shader之间共享，主要用于存储Matrix、Lighting Parameter、Color等值。

```
uniform mat4 viewProjMatrix;
uniform mat4 viewMatrix;
uniform vec3 lightPosition;

glGetUniformLocation
glUniform*
```
 
**Attribute**
这是一种只能在顶点着色器中使用的类型，用于传达有关每个顶点的信息。一般传输位置、法线、纹理坐标、颜色等信息。

```
attribute vec4 a_position;
attribute vec2 a_texCoord0;

glVertexAttribPointer
glBindAttribLocation
```

**Varying**
用于指定用作顶点着色器的输出和片段着色器的输入的变量。由于它是应用程序端无法触及的变量，因此没有相关的API。
```
varying vec2 texCoord;
varying vec4 color;
```

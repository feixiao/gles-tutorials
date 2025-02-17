## Draw a triangle

第一个示例在 EGL 曲面上绘制默认图元（三角形）。在这里，我们来看看除了简单地绘制三角形之外，绘制三角形还需要哪些准备工作。

要使用OpenGL ES（以下简称gles）绘制三角形，必须准备一种画布来绘制它。为此，必须首先完成一些任务。

- 使用Screen创建一个窗口
- 使用 EGL 创建surface


每个任务的查看如下。

### 使用 Screen 创建窗口

在 QNX 中处理窗口的方法有多种，其中之一是使用称为 Screen 的图形子系统。首先，为屏幕创建一个上下文，并使用该上下文创建一个窗口。在以这种方式创建的窗口上设置属性。在此示例中，设置了颜色格式和 opengl es 版本。

```C++
screen_set_window_property_iv(screen_win, SCREEN_PROPERTY_FORMAT, (const int[]){ SCREEN_FORMAT_RGBX8888 });
screen_set_window_property_iv(screen_win, SCREEN_PROPERTY_USAGE, (const int[]){ SCREEN_USAGE_OPENGL_ES2 });
```
如果设置 Window 属性，最终会创建一个窗口缓冲区以进行双缓冲。

```C++
screen_create_window_buffers(screen_win, 2);
```

### 创建EGL Surface 

现在我们需要将 EGL 表面连接到创建的窗口。为了实现这一目标，需要完成以下任务。

- 获取当前连接的显示器信息并初始化显示器
- 获取与给定属性匹配的 EGL 帧缓冲区设置
- 创建egl context
- 从屏幕窗口创建egl surface
- 将 EGL 渲染上下文附加到当前Surface


如果您已经完成了这一步，则可以说绘图屏幕的设置已完成。接下来要做的是顶点着色器和片段着色器相关的设置。

### Vertex / fragment shader 设置

如果您不熟悉顶点着色器和片段着色器，请查看下面链接的内容。 (http://ptgmedia.pearsoncmg.com/images/chap1_9780321933881/elementLinks/01fig01_alt.jpg)

vs 和 fs 的设置是通过以下过程完成的。

- Shader创建和编译
- 程序创建和链接

#### Shader创建和编译

第一步是通过"glCreateShader"函数创建一个着色器对象。 如果正确创建了对象，则使用"glShaderSource"函数传递实际的着色器代码。最后，当您调用"glCompileShader"函数时，接收着色器源的对象将被编译。

```C++
GLuint LoadShader ( EGLenum type, const char *shaderSrc )
{
   GLuint shader = 0;
   GLint compiled;

   // Create the shader object
   shader = glCreateShader ( type );

   // Load the shader source
   glShaderSource( shader, 1, &shaderSrc, 0 );

   // Compile the shader
   glCompileShader( shader );

   ...

   return shader;
}
``` 

### 画一个三角形

我们要做的最后一件事是确保屏幕确实绘制了三角形。这是在示例中的 render 函数中完成的，调用顺序如下。


#### viewport设置

用户任意设置实际可见屏幕区域。如果不调用，则当前显示器所持有的信息基本上用作视口。
```C++
	// Set the viewport
	glViewport ( 0, 0, surface_width, surface_height );
```
#### 初始化color buffer

通过调用`glClear`函数，当前屏幕将被绘制为`glClearColor`设置的颜色。

#### 加载vertex

为了绘制图片，您需要读取顶点信息并将其传递给着色器。在此示例中，有一个顶点着色器源，如下所示。

```
    char vShaderStr[] =
    		"attribute vec4 vPosition;                \n"
    		"void main()                              \n"
    		"{                                        \n"
    		"   gl_Position = vPosition;              \n"
    		"}                                        \n";
```

这里，需要读取三角形的顶点信息并将其连接到vPosition，执行此操作的函数是`glVertexAttribPointer`。

```C++
glVertexAttribPointer (0, 3, GL_FLOAT, GL_FALSE, 0, vTriangle);
```
您可以看到第一个参数中的“0”表示 vPosition，并连接到最后一个参数 vTriangle。


> *NOTE :*
> 0如何变成vPosition？答案可以在以下链接中找到。 (https://www.opengl.org/wiki/Vertex_Shader#Inputs)
 
> 总之，如果用户输入（这里是三角形顶点信息）没有在着色器中或通过gles API显式绑定，则索引由OpenGL ES自动分配。因此，建议在编写代码时显式连接。您可以在其他示例中检查这一点。
    

#### 绘图函数调用
绘制三角形的最后一步是调用由S自动分配的“glDrawArrays”函数，以实际告诉OpenGL ES绘制三角形。因此，建议在编写代码时显式连接。您可以在其他示例中检查这一点。

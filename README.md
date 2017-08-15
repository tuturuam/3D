# Hello Triangle

## All information taken from [learnopengl](https://learnopengl.com/#!Getting-started/Hello-Triangle)

![](/home/tuturu/Projects/Games/3D/pipeline.png)

OpenGL manages 3D coordinates to 2D coordinates using its graphics pipeline.

- There is a difference between a 2D coordinate and a pixel. 

A list of three 3D coordinates is used to form a triangle. This array list is called **Vertex Data**.

The first part of the pipeline is the vertex shader.

- Its main purpose is to transform 3D coordinates into different 3D coordinates. 

The **primitive assembly** stage takes as input all the vertices from the vertex shader that form a primitive and asselbles all the points.

The output of the primitive assembly stage is passed to the **geometry shader**. This generates a second triangle out of the given shape.

The output of the geometry shader is then passed on to the **rasterization stage**. This stage maps the pixels to the final screen, resulting in *fragments*. Before the fragment shader runs, *clipping* is performed. Clipping discards all fragments, increasing performance.

Fragment shaders are used to calculate color, and is the stage where advanced OpenGL effects occur.

The final object will pass through a final stage that is called the **alpha test and blending** stage. 

 - This stage checks for a corresponding **depth** value to check if a fragment is behind or in front of another object and should be discarded accordingly. 
 - The stage also checks for alpha values and blends the objects accordingly.

## Vertex Input
OpenGL is a 3D graphics library so all coordinates are in x, y, and z coordinates.

```c++
float vertices[] = {
    -0.5f, -0.5f, 0.0f,
     0.5f, -0.5f, 0.0f,
     0.0f,  0.5f, 0.0f
};  
```
### From learnopengl.com -- Normalized Device Coordinates

<div style="text-align: center; color: #808080; font-size: 1em;">Once your vertex coordinates have been processed in the vertex shader, they should be in normalized device coordinates which is a small space where the x, y and z values vary from -1.0 to 1.0. Any coordinates that fall outside this range will be discarded/clipped and won't be visible on your screen. Below you can see the triangle we specified within normalized device coordinates (ignoring the z axis):

![](/home/tuturu/Projects/Games/3D/ndc.png)

Unlike usual screen coordinates the positive y-axis points in the up-direction and the (0,0) coordinates are at the center of the graph, instead of top-left. Eventually you want all the (transformed) coordinates to end up in this coordinate space, otherwise they won't be visible.

Your NDC coordinates will then be transformed to screen-space coordinates via the viewport transform using the data you provided with glViewport. The resulting screen-space coordinates are then transformed to fragments as inputs to your fragment shader.
</div>

From the vertex data to the vertex shader, we need to access the memory on the GPU. This data is managed by **vertex buffer objects (VBO)**.

The vertex buffer has a unique ID corresponding to that buffer. We can generate one with a buffer ID using **glGenBuffers** function. We can bind the newly created buffer to the GL_ARRAY_BUFFER target with the **glBindBuffer** function. The **glBufferData** function copies the previously defined vertex data into the buffer's memory.

```c++
	unsigned int VBO;
	glGenBuffers(1, &VBO);

	glBindBuffer(GL_ARRAY_BUFFER, VBO);
	glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
```

Any buffer calls we make on the GL_ARRAY_BUFFER target will configure the currently bound buffer, which is **VBO**.

The function **glBufferData** copies user-defined data into the currently bound buffer. The function is split into four arguments.

1. The type of buffer we want to copy data into: which is the VBO currently bound to the GL_ARRAY_BUFFER target.
2. The size of the data we want to pass to the buffer.
3. The actual data we want to send.
4. How the graphics card should manage the given data, which can come in three forms:
	- GL_STATIC_DRAW: the data will most likely not change at all or very rarely.
	- GL_DYNAMIC_DRAW: the data will likely change a lot.
	- GL_STREAM_DRAW: the data will change every time it is drawn.

Next we want to create a vertex and fragment shader that actually processes this data.

## Vertex Shader
Modern OpenGL requires that we at least set up a vertex and fragment shader to render.

We need to write the vertex shader in GLSL (OpenGL Shading Language).

```glsl
#version 330 core
layout (location = 0) in vec3 aPos;

void main()
{
    gl_Position = vec4(aPos.x, aPos.y, aPos.z, 1.0);
}
```

### From learnopengl.com
<div style="text-align: center; color: #808080; font-size: 1em;"> As you can see, GLSL looks similar to C. Each shader begins with a declaration of its version. Since OpenGL 3.3 and higher the version numbers of GLSL match the version of OpenGL (GLSL version 420 corresponds to OpenGL version 4.2 for example). We also explicitly mention we're using core profile functionality.

Next we declare all the input vertex attributes in the vertex shader with the in keyword. Right now we only care about position data so we only need a single vertex attribute. GLSL has a vector datatype that contains 1 to 4 floats based on its postfix digit. Since each vertex has a 3D coordinate we create a vec3 input variable with the name aPos. We also specifically set the location of the input variable via layout (location = 0) and you'll later see that why we're going to need that location.

Vector

In graphics programming we use the mathematical concept of a vector quite often, since it neatly represents positions/directions in any space and has useful mathematical properties. A vector in GLSL has a maximum size of 4 and each of its values can be retrieved via vec.x, vec.y, vec.z and vec.w respectively where each of them represents a coordinate in space. Note that the vec.w component is not used as a position in space (we're dealing with 3D, not 4D) but is used for something called perspective division. We'll discuss vectors in much greater depth in a later tutorial.

To set the output of the vertex shader we have to assign the position data to the predefined gl_Position variable which is a vec4 behind the scenes. At the end of the main function, whatever we set gl_Position to will be used as the output of the vertex shader. Since our input is a vector of size 3 we have to cast this to a vector of size 4. We can do this by inserting the vec3 values inside the constructor of vec4 and set its w component to 1.0f (we will explain why in a later tutorial). 
</div>

## Compiling a shader
After writing the source code, OpenGL has to dynamically compile the shader at run-time from its source code.

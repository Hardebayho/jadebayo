---
title: "OpenGL Tutorial Part 1"
date: 2021-02-20T13:38:53Z
description: "This introduces OpenGL basics and shows how to draw a triangle to the screen"
featured: true
draft: false
toc: true
featureImage: "/logos/opengl_logo.png"
thumbnail: "/logos/opengl_logo.png"
shareImage: "/logos/opengl_logo.png"
codeMaxLines: 10
codeLineNumbers: true
figurePositionShow: true
categories:
  - Graphics
  - C++
series: "opengl-tutorials"
tags:
  - C++
  - OpenGL
  - Programming
  - OpenGL Tutorials
# comment: true # Disable comment if false.
---

## Introduction
In this post, I'll be explaining about OpenGL and how it works. The purpose of this post is to explain OpenGL in plain english and make it easier to understand. So the approach I'll be taking is that I'll be breaking this down into the following components:
+ **What** &#8594; This is just the topic (Example: Displaying a triangle on the screen with OpenGL)
+ **How** &#8594; This shows the code to achieve the what clause above and provides some explanations on what the code does.
+ **Why** &#8594; This explains the background behind what is done and why it the approach is taken.

## Setting up an OpenGL Project
* ### What
    We want to create a project that allows us to work with OpenGL.
* ### How
    First, go to [GLFW](https://glfw.org) to download the GLFW library.

    If you're running on Windows, click on the *Download* tab and scroll down to *Windows pre-compiled binaries* section. Click on *64-bit Windows binaries* or *32-bit Windows binaries* depending on whatever architecture you have. Set up the library as you would any other library.

    If you're on a Mac, install glfw from brew or build the library from source. Linux users can download from their package managers too or build the library from source. If you want to build the library from source, just download the source package from top of the Download tab.

    Once you've downloaded the library, make sure you're linking to the library and including the header paths appropriately. This is platform and toolchain-specific, so I'm not going to cover it here.

    Next, we need to download the GLAD library. Click [here](https://glad.dav1d.de/) to go to the website. Once you're there, go to the Profile tab and change it to Core. Then go to the API tab and change the **gl** section from *none* to *Version 3.3*. Make sure the Profile tab stays at Core. If it changes back to Compatibility, change it back to Core. Next, click on generate. This will take you to a page where you can download a *glad.zip* file.

    Once the file is downloaded, extract the file to a suitable location. Copy src/glad.c file to your project or add it to the list of files to be built by your build system. Take the contents of the include directory and add it to your project or add it to the list of include directories of your build system. Once this is done, we're ready to start working.

    Create a main.cpp file in your project and add it to your build system. Include the following headers:

    ```c++
    #include "glad/glad.h"
    #include <GLFW/glfw3.h>
    #include <stdio.h>
    ```
    Next, create an ```int main() {}``` function to define our program entry point. Inside the main function, do the following:
    ```c++
    // Initialize GLFW
    glfwInit();
    
    glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 3);
    glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 3);
    glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);
    
    // Create the window
    GLFWwindow* window = glfwCreateWindow(800, 600, "OpenGL Tutorial", nullptr, nullptr);
    if (!window) {
        glfwTerminate();
        return -1;
    }
    
    // Make the window the current OpenGL context
    glfwMakeContextCurrent(window);
    
    // Initialize the loader
    if (!gladLoadGLLoader((GLADloadproc)glfwGetProcAddress)) {
        fprintf(stderr, "Failed to initialize GLAD\n");
        return -1;
    }
    
    // set OpenGL to use the entire size of our window
    glViewport(0, 0, 800, 600);
    
    // The application loop
    while (!glfwWindowShouldClose(window)) {
        glfwSwapBuffers(window);
        glfwPollEvents();
    }
    
    glfwDestroyWindow(window);
    glfwTerminate();
    
    return 0;
    ```
    Save the file and run your project. If you get a bunch of undefined reference errors, it means you're not linking GLFW properly. Make sure you check the documentation on their website to figure out how to properly set up GLFW for your platform.

    If all goes well, you should see a window pop up in your screen.

* ### Why
    First of all, we call the ```glfwInit()``` function, which simply initializes the internal components of the GLFW library. After that, we did some ```glfwWindowHint``` calls to set the OpenGL profile we want to operate in. This is going to affect the OpenGL functions we're going to be able to call in our program. Essentially, we're not going to be able to access the immediate mode APIs when we're using the 3.3 core profile. Next, we create a ```GLFWwindow*``` object. This represents the window we're going to be seeing on our screen. We create an instance of the object with ```glfwCreateWindow(800, 600, "OpenGL Tutorial", nullptr, nullptr)```. This creates a window of 800 width by 600 height, with the window's title "OpenGL Tutorial". We then call ```glfwTerminate()``` if the window creation was not successful to clean up GLFW.

    The ```glfwMakeContextCurrent``` function tells OpenGL that we want to make this window the current OpenGL context. This means that all draw calls and operations will be done on the specified window.

    The ```if (!gladLoadGLLoader((GLADloadproc)glfwGetProcAddress)) {
   fprintf(stderr, "Failed to initialize GLAD\n");
   return -1;
}``` just helps us map to the appropriate OpenGL functions and get the appropriate addresses for those functions for the current platform. We could do this ourselves, but it's pretty tedious and pointless.

    Afterwards, we just sit in a loop waiting for the user to close the GLFWwindow. Inside the loop, we're polling for events to check whether the user has triggered any mouse or keyboard events. We're also swapping the window buffers here. Windows typically have two display buffers. One is the off-screen buffer where the next graphics (frame) is drawn and the other one is the one currently displayed on the screen. When drawing calls are made, OpenGL draws to the off-screen buffer. ```glfwSwapBuffers``` will then take the off-screen buffer and make it the current display buffer, taking the current display buffer and making it the off-screen buffer (hence the 'swap'). This is typically done to eliminate [screen tearing](https://en.wikipedia.org/wiki/Screen_tearing).

    Once the user closes the window and the loop terminates, we clean up GLFW and finish our application.

## Displaying a triangle
* ### What
    We want to use OpenGL to display a triangle on the screen.
* ### How
    Using the structure we had in the first section, we start by creating a VAO (Vertex Array Object) like thus:
    ```c++
    uint32_t vao;
    glGenVertexArrays(1, &vao);
    ```
    Next, we need to bind to the VAO, so we do:
    ```c++
    glBindVertexArray(vao);
    ```
    Make sure that all these calls are made **before** the loop. Next, we'll need to create a VBO (Vertex Buffer Object), so we do:
    ```c++
    uint32_t vbo;
    glGenBuffers(1, &vbo);
    ```
    We also bind to it:
    ```c++
    glBindBuffer(GL_ARRAY_BUFFER, vbo);
    ```
    Then we'll copy the vertex data we want to render to this buffer as thus:
    ```c++
    float triangle[] {
        -0.5, -0.5,
        0.0, 0.5,
        0.5, -0.5
    };
    glBufferData(GL_ARRAY_BUFFER, sizeof(triangle), triangle, GL_STATIC_DRAW);
    ```
    Once that is done, we create an attribute array to define the vertex data:
    ```c++
    glEnableVertexAttribArray(0);
    glVertexAttribPointer(0, 2, GL_FLOAT, GL_FALSE, sizeof(float) * 2, nullptr);
    ```
    Next comes creating the shader program and the vertex and fragment shaders:
    ```c++
    int program = glCreateProgram();
    int vertexShader = glCreateShader(GL_VERTEX_SHADER);
    std::string vs = "#version 330 core\n"
            "layout(location = 0) in vec4 position;\n"
                     "void main() {\n"
                     "  gl_Position = position;\n"
                     "};";
    const char* result = vs.c_str();
    glShaderSource(vertexShader, 1, &result, nullptr);
    glCompileShader(vertexShader);
    glAttachShader(program, vertexShader);

    std::string fs = "#version 330 core\n"
            "layout(location = 0) out vec4 color;\n"
                     "void main() {\n"
                     "  color = vec4(1., 0.0, 0.0, 1.0);\n"
                     "};";
    int fragmentShader = glCreateShader(GL_FRAGMENT_SHADER);
    result = fs.c_str();
    glShaderSource(fragmentShader, 1, &result, nullptr);
    glCompileShader(fragmentShader);
    glAttachShader(program, fragmentShader);

    glLinkProgram(program);
    glValidateProgram(program);
    glUseProgram(program);

    glDeleteShader(vertexShader);
    glDeleteShader(fragmentShader);
    ```
    Finally, we go inside our loop and we draw the triangle:
    ```c++
    while (!glfwWindowShouldClose(window)) {
        glClearColor(0.0, 0, 0, 1.0);
        glClear(GL_COLOR_BUFFER_BIT);
        glDrawArrays(GL_TRIANGLES, 0, 3);
        glfwPollEvents();
        glfwSwapBuffers(window);
    }
    ```
    Run the program and you will see a triangle on your screen that looks like this:
    ![Output Preview](/images/opengl_tutorial_1/output_preview.png "Output preview")
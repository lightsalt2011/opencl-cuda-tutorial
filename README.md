# opencl-cuda


一 OpenCL 概述
OpenCL程序是分成两部分的：一部分是在设备上执行的（对于我们，是GPU），另一部分是在主机上运行的（对于我们，是CPU）。在设备上执行的程序或许是你比较关注的。它是OpenCL产生神奇力量的地方。为了能在设备上执行代码，程序员需要写一个特殊的函数（kernel函数）。这个函数需要使用OpenCL语言编写。OpenCL语言采用了C语言的一部分加上一些约束、关键字和数据类型。在主机上运行的程序提供了API，所以i可以管理你在设备上运行的程序。主机程序可以用C或者C++编写，它控制OpenCL的环境（上下文，指令队列…）。

二 编写opencl程序的步骤： 
1  Platform（平台）：主机加上OpenCL框架管理下的若干设备构成了这个平台，通过这个平台，应用程序可以与设备共享资源并在设备上执行kernel。平台通过cl_platform来展现，

2 Device（设备）：通过cl_device来表现

3 Context（上下文）：定义了整个OpenCL化境，包括OpenCL kernel、设备、内存管理、命令队列等。上下文使用cl_context来表现。使用以下代码初始化：

4 Command-Queue（指令队列）：就像它的名字一样，他是一个存储需要在设备上执行的OpenCL指令的队列。“指令队列建立在一个上下文中的指定设备上。多个指令队列允许应用程序在不需要同步的情况下执行多条无关联的指令。”

For examples
cl_int error = 0;   // Used to handle error codes
cl_platform_id platform;
cl_context context;
cl_command_queue queue;
cl_device_id device;

// Platform
error = oclGetPlatformID(&platform);
if (error != CL_SUCCESS) {
   cout << "Error getting platform id: " << errorMessage(error) << endl;
   exit(error);
}
// Device
error = clGetDeviceIDs(platform, CL_DEVICE_TYPE_GPU, 1, &device, NULL);
if (err != CL_SUCCESS) {
   cout << "Error getting device ids: " << errorMessage(error) << endl;
   exit(error);
}
// Context
context = clCreateContext(0, 1, &device, NULL, NULL, &error);
if (error != CL_SUCCESS) {
   cout << "Error creating context: " << errorMessage(error) << endl;
   exit(error);
}
// Command-queue
queue = clCreateCommandQueue(context, device, 0, &error);
if (error != CL_SUCCESS) {
   cout << "Error creating command queue: " << errorMessage(error) << endl;
   exit(error);
}

5 分配空间
主机的基本环境已经配置好了，为了可以执行我们的写的小kernel，我们需要分配3个向量的内存空间，在设备上分配内存，我们需要使用cl_mem类型

6  为数据分配好空间之后就要执行kernel程序了，此处引入program, 
 Program：OpenCL Program由kernel函数、其他函数和声明组成。它通过cl_program表示。当创建一个program时，你必须指定它是由哪些文件组成的，然后编译它。
我们需要“提取”program的入口点。使用clCreateKernel, 注意我们可以创建多个OpenCL program，每个program可以拥有多个kernel。

7 运行kernel
一旦我们的kernel建立好，我们就可以运行它。
首先，我们必须设置kernel的参数, 每个参数都需要调用一次clEnqueueNDRangeKernel函数, 当所有参数设置完毕，我们就可以调用这个kernel, clEnqueueNDRangeKernel

8 读取结果
读取结果非常简单。与之前讲到的写入内存（设备内存）的操作相似，现在我们需要存入队列一个读取缓冲区的操作clEnqueueReadBuffer

9 释放空间
调用相关release函数



三 OpenCL入门资料
https://www.zhihu.com/question/48984738
OpenCL入门还是比较容易的，只要有C语言基础就可以快速上手了~ 推荐《OpenCL实战》一书。
首先当然是从向量加法的并行化或者矩阵乘法的kernel代码来作为你的第一个例子，这里理解并行化的思想非常重要，并行化的编程模型把多核设备的计算单元抽象化，不用关心你的计算设备在物理上有多少个计算单元，而只要在逻辑上划分成你需要的任意多个计算单元即可(这里也就是global_size)，从逻辑上的多核到物理上多核的映射由编译器完成，这也就是多核编程非常方便的原因所在。此间，了解platform/context/device/queue之间的依赖关系，你的OpenCL代码是打包为一个个kernel在多核设备上以队列的形式挨个运行。

OpenCL相对于CUDA来说封装了更多的硬件细节，所以对硬件架构不需要做深入的了解，但还需要知道向量化、local memory、网格划分(也就是local size的划分)这些基本概念，在并行化编程中对这些具体细节的调优会给你带来性能上显著的提升

更核心的东西依然是算法, 了解算法的细节以窥探算法中可并行化的部分是哪些，如何灵活的应用数据并行和任务并行, 做到算法的最大加速和内存压榨，才是OpenCL编程需要解决的核心问题

http://www.cnblogs.com/leiben/archive/2012/06/05/2536508.html
http://www.kimicat.com/opencl-1/opencl-jiao-xue-yi
http://blog.csdn.net/leonwei/article/category/1410041
http://blog.csdn.net/mtt_sky/article/details/46491251


http://opencl.codeplex.com/wikipage?title=OpenCL%20Tutorials&referringTitle=Home
https://developer.apple.com/search/?q=opencl&type=Sample%20Code
---
title: TensorRT C++ API使用方法
categories:
  - - 工作技能
    - 神经网络
date: 2021-02-10 09:57:37
tags:
  - TensorRT
  - ONNX
  - inference
---
## 前言
把深度学习模型用pytorch等框架搭建、训练后，要把模型保存下来，部署到生产环境中服务于实际业务。

生产环境中运行的深度学习模型有两个特点，一当然是要快，二是只做单帧数据的前向计算（推理）。为了适应这两个特点，引入了TensorRT这个工具。

简单地，可以将它看作一个类似pytorch的深度学习框架。但是这个框架不提供模型训练相关的脚手架，只提供模型构建、前向计算和模型格式转换的工具。并且得益于NV一些未开源的黑科技，用TensorRT构建出来的模型能够更快地进行前向计算。

**关键概念**：
**ONNX parser**: Takes a converted PyTorch trained model into the ONNX format as input and populates a network object in TensorRT. 
**Builder**: Takes a network in TensorRT and generates an engine that is optimized for the target platform. 
**Engine**: Takes input data, performs inferences, and emits inference output.
**Logger**: Associated with the builder and engine to capture errors, warnings, and other information during the build and inference phases.

## 使用步骤
### 0. 头文件与namespace
```cpp
#include “NvInfer.h”

using namespace nvinfer1;
```
### 1. build阶段
#### 1.0 创建Logger
创建builder之前，需要先根据需要继承自己的Logger类。其中主要的任务是override父类中的log方法，定义哪个级别的信息会被log，如何log。
```cpp
class RTLogger : public nvinfer1::ILogger {
  void log(Severity severity, const char *msg) override {
    if (severity != Severity::kINFO)    {
      AINFO << msg;
    }
  }
} rt_gLogger;
```
#### 1.1 创建builder
```cpp
nvinfer1::IBuilder *builder_= nvinfer1::createInferBuilder(rt_gLogger);
```
#### 1.2 创建network
创建network实例的过程其实跟其他框架大同小异，只不过因为受众少，包装地没那么精致而已。
##### 1.2.1 创建空的network实例
```cpp
nvinfer1::INetworkDefinition *network_ = builder_->createNetwork();
```
##### 1.2.2 填充network实例
上一步创建的network实例是一个空的、不包含网络层的对象，需要在此基础上添加网络层。由于TensorRT作为框架，是不做BP只做前向的，所以在构建网络时要同步给入权重。
此时有两种选择来做这件事:
1. 拿一个现成的模型文件(包含权重)进来，用TensorRT的格式转换工具转换一下；以ONNX的模型文件为例(ONNX模型是包含权重的)：
  ```cpp
  #include “NvOnnxParser.h”

  using namespace nvonnxparser;

  IParser*  parser = createParser(*network, logger);
  parser->parseFromFile(modelFile, ILogger::Severity::kWARNING);
  for (int32_t i = 0; i < parser.getNbErrors(); ++i){
  std::cout << parser->getError(i)->desc() << std::endl;
  }
  ```
2. 拿一个描述模型结构的配置文件，用TensorRT的模型搭建工具依次添加各种各样的网络层，从无到有地搭建出来。同时，在添加网络时，要提供对应的网络权重。
  ```cpp
  auto data = network_->addInput(dims_pair.first.c_str(), nvinfer1::DataType::kFLOAT, dims_pair.second);
  auto convLayer = network->addConvolution(*inTensor, nbOutputs, nvinfer1::DimsHW{kernelH, kernelW}, wt, bias_weight);
  auto softmaxLayer = net->addSoftMax(*inputs[0]);
  ```
至此，填充完网络层和权重参数后，便完成了网络的搭建。

#### 1.3 创建engine
完成网络构建之后，其实只是把原有网络模型，以tensorRT的格式重新构建了一下，还没有进行优化加速。所以，现在要把优化前的network转成优化后的engine。

要做到这件事需要两种信息，一是优化什么，二是怎么优化。前者已经有了，就是network。后者需要构造一个config接口，用这个接口来配置优化项。

并且，在10.0之后的版本，创建engine之前还要先创建一个序列化模型，可用于保存到磁盘上。真正使用时，先反序列化模型，然后再构造出engine实例。
```cpp
IBuilderConfig* config = builder->createBuilderConfig();
config->setMaxWorkspaceSize(1U << 20);

IHostMemory*  serializedModel = builder->buildSerializedNetwork(*network, *config);

IRuntime* runtime = createInferRuntime(logger);
ICudaEngine* engine = runtime->deserializeCudaEngine(modelData, modelSize);
```
### 2. inference阶段
The engine holds the optimized model, but to perform inference we will need to manage additional state for intermediate activations. This is done via the `ExecutionContext` interface:
```cpp
IExecutionContext *context = engine->createExecutionContext();
```

然后，在GPU上为模型的输入和输出各申请一块buffer，
```cpp
int32_t inputIndex = engine->getBindingIndex(INPUT_NAME);
int32_t outputIndex = engine->getBindingIndex(OUTPUT_NAME);

void* buffers[2];
buffers[inputIndex] = inputBuffer;
buffers[outputIndex] = outputBuffer;
```
最后，执行推理（一般是GPU与CPU异步，即CPU上的控制流在调用GPU核函数之后并不阻塞）
```cpp
context->enqueueV2(buffers, stream, nullptr);
```

由于是异步调用，在推理执行完毕之后一般还要`cudaDeviceSynchronize()`一下，等待推理过程执行完毕，然后将output结果从GPU上拷贝到CPU上。

至此，整个推理过程完毕。


## Note

1. 不同的tensorRT版本API细节有所不同，这里以TensorRT 8.2.0 Early Access (EA)为例
1. A CUDA context is automatically created the first time TensorRT makes a call to CUDA, if none exists prior to that point. It is generally preferable to create and configure the CUDA context yourself before the first call to TensoRT.
1. Interface classes in the TensorRT™ C++ API begin with the prefix I
1. config interface has many properties that you can set in order to control how TensorRT optimizes the network. One important property is the maximum workspace size. Layer implementations often require a temporary workspace, and this parameter limits the maximum size that any layer in the network can use. If insufficient workspace is provided, it is possible that TensorRT will not be able to find an implementation for a layer.
1. Serialized engines are not portable across platforms or TensorRT versions. Engines are specific to the exact GPU model they were built on (in addition to the platform and the TensorRT version).
1. The serialized engine contains the necessary copies of the weights, the parser, network definition, builder configuration and builder are no longer necessary and may be safely deleted.

## Reference
1. [官方文档8.2.0 Early Access (EA)](https://docs.nvidia.com/deeplearning/tensorrt/developer-guide/index.html#perform_inference_c)
2. https://stackoverflow.com/questions/19193468/why-do-we-need-cudadevicesynchronize-in-kernels-with-device-printf
3. https://blog.csdn.net/Bruce_0712/article/details/73477126

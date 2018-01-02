# SigDCA

[this page under construction] The Deep Compression and Acceleration (DCA) SDK provides binary libraries, example C/C++ and Python source code, and demos for:

* deep learning model compression for real-time IoT and Edge embedded systems

* acceleration of deep learning model training and testing in servers

Functionality includes:

 - support for deep learning models AlexNet, VGG-16, LeNet-5 and LeNet-300
 
 - a process flow loop for cloud training and testing, compression, and embedded target training and testing
 
 - support for embedded targets Nvidia Jetson TX2 (Parker SoC, Pascal GPU), Atom x5-E3940 (quad-core 1.3 GHz)

# Table of Contents

[Model Compression](#ModelCompression)<br/>
&nbsp;&nbsp;&nbsp;&nbsp;[Compression Flow Diagram](#CompressionFlowDiagram)<br/>
[Model Acceleration](#ModelAcceleration)<br/>
&nbsp;&nbsp;&nbsp;&nbsp;[Acceleration Flow Diagram](#AccelerationFlowDiagram)<br/>

<a name="ModelCompression"></a>
# Model Compression


<a name="CompressionFlowDiagram"></a>
## Compression Flow Diagram

Below is a flow diagram showing the multi-iterative, multi-testpoint nature of model compression.  The objective is to map continuous integration and continuous deployment (CICD) onto available server resources (i.e. public cloud and/or private servers) and embedded targets (i.e. IoT and Edge products).

&nbsp;<br/>

![Image](https://github.com/signalogic/SigDCA/blob/master/images/Deep_Learning_Model_Compression_Flow_RevA2.png?raw=true "Deep learning model compression flow diagram")

&nbsp;<br/>

Notes about the above flow diagram:

1) All testing blocks include non-real-time inference.  Inference is shown explicitly when real-time performance is required.

2) Embedded target testing depends on whether cloud testing can exactly emulate the target; for example if cloud testing is not possible with a PCIe card containing target CPU/SoC devices (ARM, NPU, DSP, ASIC, etc) then it's likely that embedded target testing is needed.  Cloud testing for embedded targets using x86 CPU and/or GPU is expected to be an exact emulation.

3) Compression requires math and algorithm expertise and tradeoff analysis.  Some concepts are basic such as quantization and weight sharing, others require advanced math, for example sparse matrix and FFT based computation.

4) Some embedded targets may incorporate data acquisition for additional training and performance adaptation.

<a name="ModelAcceleration"></a>
# Model Acceleration

<a name="AccelerationFlowDiagram"></a>
## Acceleration Flow Diagram


&nbsp;<br/>


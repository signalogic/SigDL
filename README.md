# SigDCA

[this page under construction] The Deep Compression and Acceleration (DCA) SDK provides binary libraries, example C/C++ and Python source code, and demos for:

1. Deep learning model compression for real-time IoT and Edge embedded systems

2. Deep learning model acceleration of training and testing in servers

Functionality includes:

 - support for deep learning models AlexNet, VGG-16, LeNet-5 and LeNet-300
 
 - a process flow loop for cloud training and testing, compression, and embedded target training and testing
 
 - support for embedded targets Nvidia Jetson TX2 (Parker SoC, Pascal GPU), Atom x5-E3940 (quad-core 1.3 GHz)

# Table of Contents

[Model Compression](#ModelCompression)<br/>
&nbsp;&nbsp;&nbsp;&nbsp;[Compression Flow Diagram](#CompressionFlowDiagram)<br/>
[Model Acceleration](#ModelAcceleration)<br/>
&nbsp;&nbsp;&nbsp;&nbsp;[Acceleration Flow Diagram](#AccelerationFlowDiagram)<br/>
[Supported Embedded Targets](#SupportedEmbeddedTargets)<br/>
&nbsp;&nbsp;&nbsp;&nbsp;[Nvidida Jetson TX2](#NvidiaJetsonTX2)<br/>
&nbsp;&nbsp;&nbsp;&nbsp;[Intel x5-E3940](#IntelX5E3940)<br/>
&nbsp;&nbsp;&nbsp;&nbsp;[Texas Instruments c66x (pending, see notes)](#TexasInstrumentsC66x)<br/>

<a name="ModelCompression"></a>
# Model Compression


<a name="CompressionFlowDiagram"></a>
## Compression Flow Diagram

Below is a flow diagram showing the multi-iterative, multi-testpoint nature of model compression.  The objective is to map continuous integration and continuous deployment (CICD) onto available server resources (i.e. public cloud and/or private servers) and embedded targets (i.e. IoT and Edge products).

&nbsp;<br/>

![Image](https://github.com/signalogic/SigDCA/blob/master/images/Deep_Learning_Model_Compression_Flow_RevA2.png?raw=true "Deep learning model compression flow diagram")

&nbsp;<br/>

Notes about the above flow diagram:

1) All testing blocks include non-real-time inference.  An inference block is shown explicitly when real-time performance is required.

2) Embedded target testing depends in part on how well cloud testing can emulate the target; for example if cloud testing is not possible with PCIe cards containing the same CPU/SoC devices (ARM, NPU, DSP, ASIC, etc) then it's likely that embedded target testing will be needed to measure model real-time performance and accuracy.  For embedded targets using x86 CPU and/or GPU devices, limits on number of cores, clock rate, available memory, etc. can be enforced during cloud testing in order to give reliable predictions of real-time performance and accuracy.  In all cases, power consumption must be measured on the embedded target.

3) Compression requires math and algorithm expertise and tradeoff analysis.  Some concepts are basic such as quantization and weight sharing, others require advanced math, for example sparse matrix and FFT based computation.

4) Some embedded targets may incorporate data acquisition for additional training and performance adaptation.

<a name="ModelAcceleration"></a>
# Model Acceleration

<a name="AccelerationFlowDiagram"></a>
## Acceleration Flow Diagram


&nbsp;<br/>

<a name="SupportedEmbeddedTargets"></a>
# Supported Embedded Targets

<a name="NvidiaJetsonTX2"></a>
## Nvidia Jetson TX2

<a name="IntelX5E3940"></a>
## Intel x5-E3940

<a name="TexasInstrumentsC66x"></a>
## Texas Instruments c66x

Deep learning model support for c66x multicore CPUs is pending support from Texas Instruments.  The architecture shown in the pictures below -- a combination of Atom x86 and TI c66x cores in a mini-ITX format, with 32 to 64 c66x cores accessible as half-size PCIe card(s) -- currently supports cloud training/testing, OpenCV, C/C++, Python, math libraries, 8-bit MAC (32-bit accum) operations, and other essential deep learning components, but a substantial amount of additional work is needed.  If you are using or considering to use TI c66x and wish to discuss this, please contact Signalogic.

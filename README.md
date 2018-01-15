# SigDCA

 ```html
<! this page under construction
```
The Deep Compression and Acceleration (DCA) SDK provides binary libraries, example C/C++ and Python source code, and demos for:

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
[Supported Embedded Targets for Deep Learning](#SupportedEmbeddedTargets)<br/>
&nbsp;&nbsp;&nbsp;&nbsp;[Nvidia Jetson TX2](#NvidiaJetsonTX2)<br/>
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
# Supported Embedded Targets for Deep Learning

<a name="NvidiaJetsonTX2"></a>
## Nvidia Jetson TX2

Below is an image showing the Jetson TX2 board set up in the lab, with key TX2 peripherals labeled.  Note the small camera daughtercard at right.

![Image](https://github.com/signalogic/SigDCA/blob/master/images/Jetson_TX2_in_lab_with_labels.jpg?raw=true "Nvidia Jetson TX2 board in the lab, with peripherals labeled")

&nbsp;<br/>

Below is an image showing an expanded view of the Jetson TX2 board set up in the lab.  Note the requirement for a USB hub to use the integrated keyboard and mouse, and also the "TTL to USB" cable required for a serial debug console.

![Image](https://github.com/signalogic/SigDCA/blob/master/images/Jetson_TX2_in_lab_with_peripherals.jpg?raw=true "Nvidia Jetson TX2 board in the lab, expanded view, with external peripherals labeled")

&nbsp;<br/>

<a name="IntelX5E3940"></a>
## Intel x5-E3940

<a name="TexasInstrumentsC66x"></a>
## Texas Instruments c66x

Deep learning model support for c66x multicore CPUs is pending support from Texas Instruments. c66x devices can perform energy efficient, high performance deep learning, as demonstrated in this [deep learning embedded target comparison paper by Dr. Nachiket Kapre](http://nachiket.github.io/publications/caffepresso_cases2016.pdf).  Whether TI will embrace the need for a cloud based deep leaning solution to support training, testing, and model compression for embedded targets -- and the required investment, resources, and third-party support -- is a pending question.

As a potential basis for a cloud solution, the architecture shown in the pictures and links below already works.  Currently this architecture supports OpenCV, C/C++, Python, c66x math and DSP libraries, 8-bit MAC operations (32-bit accumulation), and other essential deep learning components, but a substantial amount of additional work is needed.

The images below show multiple 64-core PCIe cards (e.g. 384 or more cores, suitable for training on high resolution data sets), in standard 1U and 2U Dell and HP servers.

![Image](https://github.com/signalogic/SigDCA/blob/master/images/Dell_R720_128_c66x_cores.jpg?raw=true "Dell R720 server with 128 c66x cores, suitable for cloud based deep learning training, testing, and model compression")

![Image](https://github.com/signalogic/SigDCA/blob/master/images/HP_DL380_G9_128_c66x_cores.jpg?raw=true "HP DL380 server with 128 c66x cores, suitable for cloud based deep learning training, testing, and model compression")

More information on [c66x + x86 deep learning cloud solutions](http://processors.wiki.ti.com/index.php/HPC).

In addition to "pure cloud" solutions, the combined c66x + x86 architecture can also perform in small-form factor server in the 30 to 70 W range.  The images below show a combination of Atom x86 and TI c66x cores in a mini-ITX format, with 32 to 64 c66x cores accessible as half-size PCIe card(s).

![Image](https://github.com/signalogic/SigDCA/blob/master/images/deep_learning_c66x_x86_server_32cores_iso_view.png?raw=true "TI c66x + x86 deep learning small server solution, using mini-ITX (iso view)")

![Image](https://github.com/signalogic/SigDCA/blob/master/images/deep_learning_c66x_x86_server_32cores_top_view.png?raw=true "TI c66x + x86 deep learning small server solution, using mini-ITX (top view)")

If you are using or considering to use TI c66x and wish to discuss this either with Signalogic or 3-way with TI Sr. Management, please contact Signalogic.

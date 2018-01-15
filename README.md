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

Model compression is the process of compressing and transforming a deep learning model trained and tested in the cloud into a model suitable for real-time inference on an IoT or Edge application embedded target.  When tested on the embedded target with the identical data set used in cloud training, the model must perform with acceptable accuracy and meet strict power consumption requirements.  Compression methods include basic approaches such as pruning, weight quantization and sharing, and weight encoding (for example Huffman coding), and more advanced approaches such as math architecture that converts convolution layers to sparse matrices (under specific conditions) suitable for FFT processing; i.e. convolution in the frequency domain.

Whatever compression methods are chosen, the model <i>must be retrained</i> using fixed-point weight values, alternative math, and other algorithm changes due to compression, in order to reach an acceptable loss in accuracy (typically 2-3%).  The process of compression, testing, and retraining may require many iterations until acceptable tradeoffs are reached.  This is a time-consuming iterative process and must be done on high performance cloud servers and not on the embedded target, where it would take prohibitive amounts of time.

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

The Amazon AWS DeepLens uses a small motherboard based with an x5-E3940 CPU.  Here is a similar commercially available board <sup>1</sup>:

![Image](https://github.com/signalogic/SigDCA/blob/master/images/E3940_miniITX_conga-IA5_web.png?raw=true "x5-E3940 quad core Atom mini-ITX board")

<sup>1 </sup>Note - the above image is a web pic, we don't have one in our lab yet.

<a name="TexasInstrumentsC66x"></a>
## Texas Instruments c66x

Deep learning model support for c66x multicore CPUs is pending support from Texas Instruments. Fundamentally, c66x devices can perform energy-efficient, high performance deep learning, as demonstrated in this [deep learning embedded target comparison paper by Dr. Nachiket Kapre](http://nachiket.github.io/publications/caffepresso_cases2016.pdf) at the University of Waterloo.  Whether TI will embrace the need for a cloud based deep learning solution to support training, testing, and model compression for embedded targets -- and the required investment, resources, and third-party support -- is a pending question.

As a potential basis for a c66x deep learning cloud solution, the architecture shown in the pictures and links below already works.  Currently this architecture supports OpenCV, C/C++, Python, c66x math and DSP libraries, 8-bit MAC operations (32-bit accumulation), and other essential deep learning components, but a substantial amount of additional work is needed.

The images below show multiple 64-core PCIe cards (up to 384 or more c66x cores depending on server form-factor, suitable for training on high resolution data sets), in standard 1U and 2U Dell and HP servers.

![Image](https://github.com/signalogic/SigDCA/blob/master/images/Dell_R720_128_c66x_cores.jpg?raw=true "Dell R720 server with 128 c66x cores, suitable for cloud based deep learning training, testing, and model compression")

![Image](https://github.com/signalogic/SigDCA/blob/master/images/HP_DL380_G9_128_c66x_cores.jpg?raw=true "HP DL380 server with 128 c66x cores, suitable for cloud based deep learning training, testing, and model compression")

For detailed information on the underlying technology of combined c66x + x86 architecture solutions, see the TI wiki pages for [HPC](http://processors.wiki.ti.com/index.php/HPC), [c66x OpenCV](http://processors.wiki.ti.com/index.php/C66x_opencv), and [NFV Transcoding](http://processors.wiki.ti.com/index.php/NFV_Transcoding).

In addition to the above "pure cloud" solutions, the combined c66x + x86 architecture can also target small-form factor servers in the 30 to 70 W range.  The images below show a combination of Atom x86 and TI c66x cores in a mini-ITX format, with 32 to 64 c66x cores accessible as half-size PCIe card(s).

![Image](https://github.com/signalogic/SigDCA/blob/master/images/deep_learning_c66x_x86_server_32cores_iso_view.png?raw=true "TI c66x + x86 deep learning small server solution, using mini-ITX (iso view)")

In the mini-ITX images, the motherboard is a dual core Atom (C2358 @ 1.74 GHz) with 4x GbE interfaces, and 8 GB DDR3 mem @ 1333 MHz.  A more suitable motherboard would be a x5-E3940 quad core Atom, with either a x4 or x8 PCIe slot (or mPCIe interface).

![Image](https://github.com/signalogic/SigDCA/blob/master/images/deep_learning_c66x_x86_server_32cores_top_view.png?raw=true "TI c66x + x86 deep learning small server solution, using mini-ITX (top view)")

As an interesting side note, besides x86 it's also possible to combine a half-size c66x PCIe card Tegra GPU SoCs, for example the Jetson TX2 board has an x4 PCIe slot (see Jetson pics above).  What could be done with a next-gen Tegra (say 128 CUDA cores and 128 Tensor cores) and 32 c66x cores ?

If you are using or considering to use TI c66x and wish to discuss this either with Signalogic or 3-way with TI Sr. Management, please contact Signalogic.

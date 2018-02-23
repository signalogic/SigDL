# SigDL -- Deep Learning for IoT and Edge Embedded Targets

 ```html
<! this page under construction
```
The SigDL SDK provides binary libraries, example C/C++ and Python source code, and demos for:

1. Deep learning model compression for real-time IoT and Edge embedded systems

2. Deep learning model acceleration of training and testing in servers

Functionality includes:

 - support for deep learning models AlexNet, VGG-16, LeNet-5 and LeNet-300, and MobileNets
 
 - a process flow loop for cloud training and testing, compression, and embedded target training and testing
 
 - support for embedded targets Nvidia Jetson TX2 (Parker SoC, Pascal GPU), Atom x5-E3940 (quad-core 1.3 GHz), and Movidius MA2450

# Table of Contents

[Deep Learning Model Compression](#DeepLearningModelCompression)<br/>
&nbsp;&nbsp;&nbsp;&nbsp;[Model Compression Flow Diagram](#CompressionFlowDiagram)<br/>
[Deep Learning Model Acceleration](#DeepLearningModelAcceleration)<br/>
&nbsp;&nbsp;&nbsp;&nbsp;[Model Acceleration Flow Diagram](#AccelerationFlowDiagram)<br/>
[Deep Learning Embedded Targets](#SupportedEmbeddedTargets)<br/>
&nbsp;&nbsp;&nbsp;&nbsp;[Nvidia Jetson TX2](#NvidiaJetsonTX2)<br/>
&nbsp;&nbsp;&nbsp;&nbsp;[Atom (Intel) x5-E3940](#IntelX5E3940)<br/>
&nbsp;&nbsp;&nbsp;&nbsp;[Movidius (Intel) MA2450 Neural Net Chip](#MovidiusMA2450)<br/>
&nbsp;&nbsp;&nbsp;&nbsp;[Texas Instruments c66x (pending, see notes)](#TexasInstrumentsC66x)<br/>

<a name="DeepLearningModelCompression"></a>
# Deep Learning Model Compression

Model compression is the process of compressing and transforming a deep learning model trained and tested in the cloud into a model suitable for real-time inference on an IoT or Edge application embedded target.  When tested on the embedded target with the identical data set used in cloud training, the model must perform with acceptable accuracy, real-time performance (for example, frames per sec), and meet strict power consumption requirements.  Compression methods include basic approaches such as pruning, weight quantization and sharing, and weight encoding (for example Huffman coding), and more advanced approaches such as math architecture that converts convolution layers to sparse matrices (under specific conditions) suitable for FFT processing; i.e. convolution in the frequency domain.

After compression methods are selected, the model <i>must be retrained</i> using fixed-point weight values, alternative math, and other algorithm changes due to compression, in order to reach an acceptable loss in accuracy (typically 2-3%).  The process of training, testing, compression, more testing, and retraining may require many iterations until acceptable tradeoffs are reached.  This is a time-consuming, iterative process and must be done on high performance cloud servers and not on the embedded target, where it would take prohibitive amounts of time.

<a name="CompressionFlowDiagram"></a>
## Model Compression Flow Diagram

Below is a flow diagram showing the multi-iterative, multi-testpoint nature of model compression.  The objective is to map continuous integration and continuous deployment (CICD) onto available server resources (i.e. public cloud and/or private servers) and embedded targets (i.e. IoT and Edge products).

&nbsp;<br/>

![Deep learning model compression flow diagram](https://github.com/signalogic/SigDL/blob/master/images/Deep_Learning_Model_Compression_Flow_Diagram_RevA4.png?raw=true "Deep learning model compression flow diagram")

&nbsp;<br/>

Notes about the above flow diagram:

1) All testing blocks include non-real-time inference.  An inference block is shown explicitly when real-time performance is required.

2) Embedded target testing -- specifically the "Re-Training" block -- depends in part on how well cloud based compression testing can emulate the target.  If cloud testing is not possible with PCIe cards containing the same CPU/SoC devices (ARM, NPU, DSP, ASIC, etc) -- or software emulation is used (e.g. a "translater" or "optimizer" module of some type) -- then it's likely that embedded target testing will be needed to verify model real-time performance and accuracy.  In all cases, power consumption must be measured on the embedded target.<br/>&nbsp;<br/>If the embedded target contains an Nvidia SoC, then Nvidia GPU boards with the same CUDA and/or Tensor core type can be used for Re-Training.  If the embedded target uses x86 CPU cores, then x86 server CPUs can be used for Re-Training (preferably the same generation of x86 chip architecture).  For embedded targets using x86 CPU and/or GPU devices, limits on number of cores, clock rate, available memory, etc. can be enforced during cloud testing in order to give reliable predictions of real-time performance and accuracy.<br/>&nbsp;<br/>If the embedded target contains an SoC or CPU type that doesn't have an equivalent PCIe card (e.g. Intel Movidius) -- or it does but for whatever reason the vendor doesn't incorporate the PCIe card in their deep learning software tools (e.g. Texas Instruments) -- then a "translator" tool or module is required.  Such cross-chip approaches to model compression, re-training, and testing are likely to be problematic, for example the translator software may be closed source, may take excessive time to run (and not allow parallelization), may not precisely emulate the target, or in the worst case, may not be well supported by the vendor.

3) Compression requires math and algorithm expertise and tradeoff analysis.  Some concepts are basic such as quantization and weight sharing, others require advanced math, for example sparse matrix and FFT based computation.

4) It's expected that embedded targets will incorporate data acquisition for additional training and performance adaptation.  This leads to the CICD requirement, as noted above.

<a name="DeepLearningModelAcceleration"></a>
# Deep Learning Model Acceleration

<a name="AccelerationFlowDiagram"></a>
## Model Acceleration Flow Diagram

In the following description, "acceleration" refers to fast-as-possible training performance in the cloud, using whatever cloud resources are available or can be procurred.  Some deep learning researchers refer to acceleration to mean inference performance improvement; for embedded targets, this is an incorrect usage as the question is simply whether the embedded target / product is running inference in real-time or not.  I.e. for video or image based applications, does performance meet the fps (frames per sec) product requirement ?  The answer is a function of model compression and the resulting inference run-time, not training.

Below is a flow diagram showing deep learning model acceleration.  Like model compression above, the process is multi-iterative with multiple test points.  In this case, the focus is on accelerating model training, for example reducing training time to a day or several hours, or possibly less.  When combined with model compression, the primary objective is to map continuous integration and continuous deployment (CICD) onto available server resources (i.e. public cloud and/or private servers) and embedded targets (i.e. IoT and Edge products).

&nbsp;<br/>

![Deep learning model acceleration flow diagram](https://github.com/signalogic/SigDL/blob/master/images/Deep_Learning_Model_Acceleration_Flow_Diagram_RevA2.png?raw=true "Deep learning model acceleration flow diagram")

&nbsp;<br/>

Notes about the above flow diagram:

1) Training blocks are accelerated by parallelizing either (i) the training data set or (ii) the deep learning model.  Data set parallelization is used more often.  Tim Dettmer's [Parallelize Deep Learning on GPUs page](http://timdettmers.com/2014/10/09/deep-learning-data-parallelism) has a good explanation of the tradeoffs in these two approaches.

2) The initial Training block assumes use of GPU boards of some type.

3) The "Re-Training" block depends on "device accurate" emulation of the embedded target.  It can also be accelerated; however its situation with respect to training results, especially accuracy and performance measurement, is completely different than the "Training" block (see above model compression notes).

4) Testing can potentially be parallelized also, for example in cases where multiple compression models are concurrently being evaluated.

<a name="SupportedEmbeddedTargets"></a>
# Deep Learning Embedded Targets

Work is ongoing to support the following embedded targets with SigDL software.

<a name="NvidiaJetsonTX2"></a>
## Nvidia Jetson TX2

Below is an image showing the Jetson TX2 board set up in the lab, with key TX2 peripherals labeled.  Note the small camera daughtercard at right.

![Nvidia TX2 Jetson board in the lab, with peripherals labeled](https://github.com/signalogic/SigDL/blob/master/images/Jetson_TX2_in_lab_with_labels2.jpg?raw=true "Nvidia Jetson TX2 board in the lab, with peripherals labeled")

&nbsp;<br/>

Below is an image showing an expanded view of the Jetson TX2 board set up in the lab.  Note the requirement for a USB hub to use the integrated keyboard and mouse, and also the "TTL to USB" cable required for a serial debug console.

![Nvidia Jetson TX2 board in the lab, expanded view, with external peripherals labeled](https://github.com/signalogic/SigDL/blob/master/images/Jetson_TX2_in_lab_with_peripherals.jpg?raw=true "Nvidia Jetson TX2 board in the lab, expanded view, with external peripherals labeled")

Below the "ocean FFT" and "random fog" CUDA demos are running.  For deep learning, inference run-times can be downloaded and run via remote log-in, which is a requirement for CIDC cloud based model compression, as described above.

![Nvidia Jetson TX2 board running CUDA demos, using remote download and run](https://github.com/signalogic/SigDL/blob/master/images/Jetson_TX2_in_lab_CUDA_demos.jpg?raw=true "Nvidia Jetson TX2 board running CUDA demos, using remote download and run")

&nbsp;<br/>

<a name="IntelX5E3940"></a>
## Atom x5-E3940

Below is a partial disassembly view of Amazon's AWS DeepLens.  Note the small form-factor motherboard, with fine-pitch connector and small package size technology similar to a cell phone (e.g. 0201 and 1005 discrete components).  An Atom x5-E3940 quad-core CPU is under the heat sink.

![AWS DeepLens with Atom x5-E3940 quad-core CPU on small form-factor motherboard](https://github.com/signalogic/SigDL/blob/master/images/DeepLens_disassembly1.jpg?raw=true "partial disassembly view of AWS DeepLens motherboard.  The Atom x5-E3940 quad-core CPU is under the heat sink")

&nbsp;<br/>

Below is a top (component) side view of the AWS DeepLens motherboard:

![AWS DeepLens with Atom x5-E3940 quad-core CPU on small form-factor motherboard (component side view)](https://github.com/signalogic/SigDL/blob/master/images/DeepLens_motherboard_component_side_with_labels.jpg?raw=true "component side view of AWS DeepLens motherboard with Atom x5-E3940")

&nbsp;<br/>

![AWS DeepLens with Atom x5-E3940 quad-core CPU on small form-factor motherboard (solder side view)](https://github.com/signalogic/SigDL/blob/master/images/DeepLens_motherboard_solder_side.jpg?raw=true "solder side view of AWS DeepLens motherboard with Atom x5-E3940")

&nbsp;<br/>

Here is a commercially available mini-ITX form-factor board, also with an Atom x5-E3940 quad-core CPU similar to the DeepLens<sup>1</sup>

![x5-E3940 quad-core Atom CPU mini-ITX board](https://github.com/signalogic/SigDL/blob/master/images/E3940_miniITX_conga-IA5_web.png?raw=true "x5-E3940 quad core Atom CPU mini-ITX board")

<sup>1 </sup>Note - the above image is a web pic, we don't have one in our lab yet.

&nbsp;<br/>

<a name="MovidiusMA2450"></a>
## Movidius MA2450 Neural Net Chip

Below are some lab pics showing internal assembly and cabling of the AIY vision kit.  The assembly is formed by attaching a daughtermodule with the Movidius MA2450 device and a v2 camera to a Raspberry Pi Zero W module.  The Zero W is required as it has a fine pitch FPC (Flexible Printed Circuit) connector for the camera.

The image below shows the Raspberry Pi Zero W module by itself, without the Movidius daughtermodule attached.  The 40-pin header is used by the Raspberry Pi community for expansion purposes, usually in the form of attaching a wide variety of daughtermodules.  Note the camera connector at left, and also the ACT (activity LED) near the micro USB power connector, which is used to indicate boot activity.

![Raspberry Pi Zero W module without daughtermodule and camera connected](https://github.com/signalogic/SigDL/blob/master/images/Raspberry_pi_w_zero_basic_config_w_labels.jpeg?raw=true "Raspberry Pi Zero W module without daughtermodule and camera connected")

&nbsp;<br/>

The image below shows dmesg after a Raspberry Pi Zero W boot, still without the Movidius vision module or camera attached.  Note that the Raspberry Pi SoC contains a simple onchip boot loader which does a few SoC initialization steps, but from that point forward is entirely dependent on micro SD card contents (the boot process is [documented on this stackexchange thread](https://raspberrypi.stackexchange.com/questions/10489/how-does-raspberry-pi-boot/10490#10490)).

![Raspberry Pi Zero W module boot, without daughtermodule and camera connected](https://github.com/signalogic/SigDL/blob/master/images/Raspberry_pi_w_zero_basic_config_boot_dmesg.jpeg?raw=true "Raspberry Pi Zero W module boot, without daughtermodule and camera connected")

The next set of images show the Raspberry Pi Zero W module with the Movidius vision daughtermodule attached, and the camera connected, from different perspectives. 

![AIY Vision Kit, including Raspberry Pi Zero W module, Movidius MA2450 module, and v2 camera](https://github.com/signalogic/SigDL/blob/master/images/AIY_Movidius_internal_assembly_and_cabling.jpeg?raw=true "AIY Vision Kit, including Raspberry Pi Zero W module, Movidius MA2450 module, and v2 camera")

&nbsp;<br/>

![AIY Vision Kit partial assembly (including Raspberry Pi Zero W module, Movidius MA2450 module, and v2 camera](https://github.com/signalogic/SigDL/blob/master/images/AIY_Movidius_partial_assembly2.jpeg?raw=true "AIY Vision Kit partial assembly (including Raspberry Pi Zero W module, Movidius MA2450 module, and v2 camera")

&nbsp;<br/>

![AIY Vision Kit partial assembly (including Raspberry Pi Zero W module, Movidius MA2450 module, and v2 camera](https://github.com/signalogic/SigDL/blob/master/images/AIY_Movidius_partial_assembly3.jpeg?raw=true "AIY Vision Kit partial assembly (including Raspberry Pi Zero W module, Movidius MA2450 module, and v2 camera")

&nbsp;<br/>

The image below shows dmesg display, grepped for "googlevision", after booting with the Movidius vision module and camera attached.  Note the vision module ASIC chip is called "Myriad".  "Bonnet" is a typical Googly word for daughtermodule.  When solving embedded target debug and development problems, it can be helpful to keep in mind that embedded system fundamentals such as small form-factors, reduced power consumption and heat, expansion options, JTAG debug interface, and other key areas haven't changed since the 
1990s, even if Google would like to apply new terminology.

![dmesg display after booting Raspberry Pi Zero W module with Movidius MA2450 module and v2 camera attached](https://github.com/signalogic/SigDL/blob/master/images/Raspberry_pi_w_zero_Myriad_vision_module_boot.jpeg?raw=true "dmesg display after booting Raspberry Pi Zero W module with Movidius MA2450 module and v2 camera attached")

&nbsp;<br/>

<a name="TexasInstrumentsC66x"></a>
## Texas Instruments c66x

Deep learning model support for c66x multicore CPUs is pending support from Texas Instruments. Fundamentally, c66x devices are able to perform energy-efficient, high performance deep learning, as demonstrated in this [deep learning embedded target comparison paper by Dr. Nachiket Kapre](http://nachiket.github.io/publications/caffepresso_cases2016.pdf) at the University of Waterloo.  Whether TI will embrace the need for a cloud based c66x deep learning solution to support c66x device-accurate training, testing, and model compression for embedded targets -- and the required investment, resources, and third-party support -- is a pending question.

As a potential basis for a c66x device-accurate deep learning cloud solution, the architecture shown in the pictures and links below already works.  Currently this architecture supports OpenCV, C/C++, Python, c66x math and DSP libraries, 8-bit MAC operations (32-bit accumulation), and other essential deep learning components, but a substantial amount of additional work is needed.

The images below show multiple 64-core PCIe cards (up to 384 or more c66x cores depending on server form-factor, suitable for training on high resolution data sets), in standard 1U and 2U Dell and HP servers.

![Dell R720 server with 128 c66x cores, suitable for cloud based deep learning training, testing, and model compression](https://github.com/signalogic/SigDL/blob/master/images/Dell_R720_128_c66x_cores.jpg?raw=true "Dell R720 server with 128 c66x cores, suitable for cloud based deep learning training, testing, and model compression")

![HP DL380 server with 128 c66x cores, suitable for cloud based deep learning training, testing, and model compression](https://github.com/signalogic/SigDL/blob/master/images/HP_DL380_G9_128_c66x_cores.jpg?raw=true "HP DL380 server with 128 c66x cores, suitable for cloud based deep learning training, testing, and model compression")

For detailed information on the underlying technology of combined c66x + x86 architecture solutions, see the TI wiki pages for [HPC](http://processors.wiki.ti.com/index.php/HPC), [c66x OpenCV](http://processors.wiki.ti.com/index.php/C66x_opencv), and [NFV Transcoding](http://processors.wiki.ti.com/index.php/NFV_Transcoding).

In addition to the above "pure cloud" solutions, the combined c66x + x86 architecture can also target small-form factor servers in the 30 to 70 W range.  The images below show a combination of Atom x86 and TI c66x cores in a mini-ITX format, with 32 to 64 c66x cores accessible as half-size PCIe card(s).

![TI c66x + x86 deep learning small server solution, using mini-ITX (iso view)](https://github.com/signalogic/SigDL/blob/master/images/deep_learning_c66x_x86_server_32cores_iso_view.png?raw=true "TI c66x + x86 deep learning small server solution, using mini-ITX (iso view)")

In the mini-ITX images, the motherboard is a dual core Atom (C2358 @ 1.74 GHz) with 4x GbE interfaces, and 8 GB DDR3 mem @ 1333 MHz.  A more suitable motherboard would be a x5-E3940 quad core Atom, with either a x4 or x8 PCIe slot (or mPCIe interface).

![TI c66x + x86 deep learning small server solution, using mini-ITX (top view)](https://github.com/signalogic/SigDL/blob/master/images/deep_learning_c66x_x86_server_32cores_top_view.png?raw=true "TI c66x + x86 deep learning small server solution, using mini-ITX (top view)")

As an interesting side note, besides x86 it's also possible to combine a half-size c66x PCIe card Tegra GPU SoCs, for example the Jetson TX2 board has an x4 PCIe slot (see Jetson pics above).  What could be done with a next-gen Tegra (say 128 CUDA cores and 128 Tensor cores) and 32 c66x cores ?

If you are using or considering to use TI c66x and wish to discuss this either with Signalogic or 3-way with TI Sr. Management, please contact Signalogic.

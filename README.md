# SigDL -- Deep Learning for IoT and Edge Embedded Targets

 ```html
<! this page under construction
```
The SigDL SDK provides binary libraries, example C/C++ and Python source code, and demos for:

1. Deep learning model compression for real-time IoT and Edge embedded systems

2. Deep learning model acceleration of training and testing in servers

Functionality includes:

 - Support for deep learning models AlexNet, VGG-16, LeNet-5 and LeNet-300, and compressed models MobileNet and SqueezeNet
 
 - Process flow loop for cloud training and testing, compression, and embedded target training and testing
 
 - Support for embedded targets Nvidia Jetson TX2 (Parker SoC, Pascal GPU), Atom x5-E3930 and x5-E3940, and Movidius MA2450

# Table of Contents

[Deep Learning Model Compression](#DeepLearningModelCompression)<br/>
&nbsp;&nbsp;&nbsp;&nbsp;[Model Compression Flow Diagram](#CompressionFlowDiagram)<br/>
[Deep Learning Model Acceleration](#DeepLearningModelAcceleration)<br/>
&nbsp;&nbsp;&nbsp;&nbsp;[Model Acceleration Flow Diagram](#AccelerationFlowDiagram)<br/>
[Deep Learning Embedded Targets](#SupportedEmbeddedTargets)<br/>
&nbsp;&nbsp;&nbsp;&nbsp;[Nvidia Jetson TX2](#NvidiaJetsonTX2)<br/>
&nbsp;&nbsp;&nbsp;&nbsp;[Intel Atom](#IntelAtom)<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[Amazon DeepLens](#DeepLens_Atom)<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[Commercial mini-ITX](#miniITX_Atom)<br/>
&nbsp;&nbsp;&nbsp;&nbsp;[Movidius (Intel) MA2450 Neural Net Chip](#MovidiusMA2450)<br/>
&nbsp;&nbsp;&nbsp;&nbsp;[Texas Instruments c66x (pending, see notes)](#TexasInstrumentsC66x)<br/>

<a name="DeepLearningModelCompression"></a>
# Deep Learning Model Compression

Model compression is the process of compressing and optimizing a deep learning model trained and tested in the cloud into a model suitable for real-time inference on an IoT or Edge application embedded target.  When tested on the embedded target with the identical data set used in cloud training, the model must perform with acceptable accuracy, real-time performance (for example, frames per sec), and meet application-defined power consumption requirements.  Compression methods include basic approaches such as pruning, weight quantization and sharing, and weight encoding (for example Huffman coding), and more advanced approaches such as math algorithms that converts convolution layers to sparse matrices (under specific conditions) suitable for FFT processing; i.e. convolution in the frequency domain.

After compression methods are selected, the model <i>must be retrained</i> using fixed-point weight values, alternative math, and other algorithm changes due to compression, in order to reach (i) a tolerable loss in accuracy (typically 2 - 5%) and (ii) required real-time performance (typically some number of frames per sec).  The process of training, testing, compression, retraining, and more testing, may require many iterations until acceptable tradeoffs are reached.  This is a time-consuming, iterative process that must be done on high performance cloud or private servers and not on the embedded target, where it would take prohibitive amounts of time.

<a name="CompressionFlowDiagram"></a>
## Model Compression Flow Diagram

Below is a flow diagram showing the multi-iterative, multi-testpoint nature of model compression.  The objective is to map continuous integration and continuous deployment (CICD) onto available server resources (i.e. public cloud and/or private servers) and embedded targets (i.e. IoT and Edge products).

&nbsp;<br/>

<!--- standard image link and storage method recommended by Github, image not discoverable by Google
![Deep learning model compression flow diagram](https://github.com/signalogic/SigDL/blob/master/images/Deep_Learning_Model_Compression_Flow_Diagram_RevA4.png "Deep learning model compression flow diagram")
--->

<!--- direct link to image storage on githubusercontent.com, image is discoverable -->

![Deep learning model compression flow diagram](https://raw.githubusercontent.com/signalogic/SigDL/master/images/Deep_Learning_Model_Compression_Flow_Diagram_RevA4.png "Deep learning model compression flow diagram")

&nbsp;<br/>

Notes about the above flow diagram:

1) All testing blocks include inference.  For inference testing performance may vary from non-real-time to accelerated many times faster than real-time, depending on available cloud resources.  An inference block is shown explicitly when real-time performance is required.

2) Testing performed after Re-Training depends on how well cloud based compression testing can emulate the target.  If cloud testing is not possible with PCIe cards containing the same CPU/SoC devices (ARM, NPU, DSP, ASIC, etc) -- or software emulation is used (e.g. a "translater" or "optimizer" module of some type) -- then it's likely that testing on the embedded target itself will be needed to verify model real-time performance and accuracy.  In all cases, power consumption must be measured on the embedded target.<br/>&nbsp;<br/>If the embedded target contains an Nvidia SoC, then Nvidia GPU boards with the same CUDA and/or Tensor core type can be used for Re-Training.  If the embedded target uses x86 CPU cores, then x86 server CPUs can be used for Re-Training (preferably the same generation of x86 chip architecture).  For embedded targets using x86 CPU and/or GPU devices, limits on number of cores, clock rate, available memory, etc. can be enforced during cloud testing in order to give reliable predictions of real-time performance and accuracy.<br/>&nbsp;<br/>If the embedded target contains an SoC or CPU type that doesn't have an equivalent PCIe card (e.g. Intel Movidius) -- or it does but for whatever reason the vendor doesn't incorporate the PCIe card in their deep learning software tools (e.g. Texas Instruments) -- then a "translator" tool or module is required.  Such cross-chip approaches to model compression, re-training, and testing are likely to be problematic, for example the translator software may be closed source, may take excessive time to run (and not allow parallelization), may not precisely emulate the target, or in the worst case, may not be well supported by the vendor.

3) Compression requires math and algorithm expertise and tradeoff analysis.  Some concepts are basic such as quantization and weight sharing, others require advanced math, for example sparse matrix and FFT based computation.

4) It's expected that embedded targets will incorporate data acquisition for additional training and performance adaptation.  This leads to the CICD requirement, as noted above.

<a name="DeepLearningModelAcceleration"></a>
# Deep Learning Model Acceleration

<a name="AccelerationFlowDiagram"></a>
## Model Acceleration Flow Diagram

In the following description, "acceleration" means as-fast-as-possible training performance in the cloud, using whatever cloud resources are available or can be procurred.  Some deep learning researchers refer to acceleration to mean inference performance improvement; for embedded targets, this is an incorrect usage as the question is simply whether the embedded target / product is running inference in real-time, and what level of optimization is required to achieve that.  For example, for video or image based applications, does performance meet the fps (frames per sec) product requirement ?  The answer is a function of model compression and other optimizations applied to the inference run-time.  Maintaining a clear distinction in these terms allows an optimized process to also be accelerated, for example the testing block following Re-Training in the above and below flow diagrams is already optimized (due to compression), but can still be accelerated in the cloud.  It can be parallelized to run faster, or multiple compression algorithms can be evaluated concurrently.  To summarize, in the software and hardware descriptions on this page, "optimization" means achieving real-time inference (in the product), and "acceleration" means reducing time required for training (in the cloud).

Below is a flow diagram showing deep learning model acceleration.  Like model compression above, the process is multi-iterative with multiple test points.  In this case, the focus is on accelerating model training, for example reducing training time to a day or several hours, or possibly less.  When combined with model compression, the primary objective is to map continuous integration and continuous deployment (CICD) onto available server resources (i.e. public cloud and/or private servers) and embedded targets (i.e. IoT and Edge products).

&nbsp;<br/>

![Deep learning model acceleration flow diagram](https://raw.githubusercontent.com/signalogic/SigDL/master/images/Deep_Learning_Model_Acceleration_Flow_Diagram_RevA2.png "Deep learning model acceleration flow diagram")

&nbsp;<br/>

Notes about the above flow diagram:

1) Training blocks are accelerated by parallelizing either (i) the training data set or (ii) the deep learning model.  Data set parallelization is used more often.  Tim Dettmer's [Parallelize Deep Learning on GPUs page](http://timdettmers.com/2014/10/09/deep-learning-data-parallelism) has a good explanation of the tradeoffs in these two approaches.

2) The initial Training block assumes use of GPU boards of some type.

3) The "Re-Training" block depends on "device accurate" emulation of the embedded target.  It can also be accelerated; however its situation with respect to training results, especially accuracy and performance measurement, is completely different than the "Training" block (see above model compression notes).

4) Testing can potentially be parallelized also, for example in cases where multiple compression models are concurrently being evaluated.

<a name="SupportedEmbeddedTargets"></a>
# Deep Learning Embedded Targets

Work is ongoing to support the following embedded targets with SigDL software.  The following sections take a closer look at available deep learning product platforms and form-factors that range in power consumption from 2 to 30 W.

<a name="NvidiaJetsonTX2"></a>
## Nvidia Jetson TX2

Below is an image showing the Jetson TX2 board set up in the lab, with key TX2 peripherals labeled.  Note the small camera daughtercard at right.

![Nvidia TX2 Jetson board in the lab, with peripherals labeled](https://raw.githubusercontent.com/signalogic/SigDL/master/images/Jetson_TX2_in_lab_with_labels2.jpg "Nvidia Jetson TX2 board in the lab, with peripherals labeled")

&nbsp;<br/>

Below is an image showing an expanded view of the Jetson TX2 board set up in the lab.  Note the requirement for a USB hub to use the integrated keyboard and mouse, and also the "TTL to USB" cable required for a serial debug console.

![Nvidia Jetson TX2 board in the lab, expanded view, with external peripherals labeled](https://raw.githubusercontent.com/signalogic/SigDL/master/images/Jetson_TX2_in_lab_with_peripherals.jpg "Nvidia Jetson TX2 board in the lab, expanded view, with external peripherals labeled")

Below the "ocean FFT" and "random fog" CUDA demos are running.  For deep learning, inference run-times can be downloaded and run via remote log-in, which is a requirement for CIDC cloud based model compression, as described above.

![Nvidia Jetson TX2 board running CUDA demos, using remote download and run](https://raw.githubusercontent.com/signalogic/SigDL/master/images/Jetson_TX2_in_lab_CUDA_demos.jpg "Nvidia Jetson TX2 board running CUDA demos, using remote download and run")

&nbsp;<br/>

<a name="IntelAtom"></a>
## Intel Atom

The deep learning embedded targets below use Atom x5-E39xx family CPUs, but vary in SWaP<sup> 1</sup>, making them suitable for IoT and Edge products with power consumption ranging from 7 to 15 W.  Using an x86 compatible CPU offers advantages in (i) compatibility with a wide variety of software, (ii) fast development time, and (iii) support for a huge range of I/O and storage peripherals.

<a name="DeepLens_Atom"></a>
### Amazon DeepLens

In terms of SWaP, Amazon AWS's DeepLens represents the current state-of-the-art lower boundary for x86 CPU based deep learning IoT and Edge products.  In a way, Amazon has created the "smallest possible x86 server"; the images below provide some insight into the challenges they faced.  The first image is a partial disassembly view.  Note the small form-factor motherboard, incorporating mobile phone PCB technology (such as ultra fine-pitch connectors and FBGA packages, blind and buried vias and microvias, and extensive use of 0201 and 1005 discrete component package sizes).  An Atom x5-E3930 dual-core CPU (1.3 GHz, 6.5 W TDP) is under the heat sink (there are no fans in the enclosure).

Also note the use of a USB camera interface.  For applications requiring multiple cameras, or a multichannel combination of vision and audio I/O, adding more USB ports is straightforward when running Linux on an x86 CPU.

![AWS DeepLens with Atom x5-E3930 dual-core CPU on small form-factor motherboard](https://raw.githubusercontent.com/signalogic/SigDL/master/images/DeepLens_disassembly1.jpg "partial disassembly view of Amazon AWS DeepLens motherboard.  An Atom x5-E3930 dual-core CPU is under the heat sink")

&nbsp;<br/>

Below is a top (component) side view of the DeepLens motherboard:

![AWS DeepLens with Atom x5-E3930 dual-core CPU on small form-factor motherboard (component side view)](https://raw.githubusercontent.com/signalogic/SigDL/master/images/DeepLens_motherboard_component_side_with_labels.jpg "component side view of Amazon AWS DeepLens motherboard with Atom x5-E3930 CPU")

In any deep learning IoT product design, minimizing memory power consumption is crucial, but it's especially important for x86 CPU based designs, which require sizable external memory capacity in order to gain maximum advantage from running Linux and open source software (much of which is developed and runs in the cloud).  The DeepLens uses LPDDR4 DRAM, often referred to as "mobile DRAM" (i.e. cell phone memory technology).  The DeepLens has 8 GB of DRAM memory, implemented as four (4) Samsung 16 Gb x16 external devices (64-bit wide interface, maximum of 3733 Mbps transfer rate from a 233 MHz clock).  Non-volatile storage is provided by an SK Hynix 16 GB NAND Flash chip.

Although the fanless DeepLens enclosure provides a great example of what's currently possible with x86 CPUs inside IoT and Edge products, Intel still has a ways to go to improve the situation.  The x5 CPU family generates enough heat that external LPDDR4 devices must be placed a considerable distance from the CPU (due to the mechanical size of the heat sink), which impacts both memory performance and overall size of the motherboard.  A desirable objective for future x86 CPU chip designs would be to use PoP (package on package) technology to allow memory devices to "stack" directly on top of the CPU.  However, this will take another 2x or more reduction in CPU power consumption.

&nbsp;<br/>

Below is a bottom (solder) side view of the DeepLens motherboard:

![AWS DeepLens with Atom x5-E3930 dual-core CPU on small form-factor motherboard (solder side view)](https://raw.githubusercontent.com/signalogic/SigDL/master/images/DeepLens_motherboard_solder_side.jpg "solder side view of Amazon AWS DeepLens motherboard with Atom x5-E3930 CPU")

&nbsp;<br/>

<a name="miniITX_Atom"></a>
### Commercial mini-ITX

Here is a commercially available mini-ITX form-factor board<sup> 2</sup>, with an Atom x5-E3940 quad-core CPU.  Compared to the DeepLens CPU, the x5-E3940 has 2 additional cores, runs at 1.6 GHz and has a higher power rating (9.5 W vs. 6.5 W TDP).

![x5-E3940 quad-core Atom CPU mini-ITX board](https://raw.githubusercontent.com/signalogic/SigDL/master/images/E3940_miniITX_conga-IA5_web.png "x5-E3940 quad-core Atom CPU mini-ITX board")

<sup>1 </sup>SWaP = size, weight, and power consumption.<br/>
<sup>2 </sup>The commercial mini-ITX image is a web pic, we don't have one in our lab yet.

&nbsp;<br/>

<a name="MovidiusMA2450"></a>
## Movidius MA2450 Neural Net Chip

The Movidius neural net devices are essentially ASICs for use in Edge and IoT products that optimize deep learning model compression in order to minimize SWaP.  Without the need to run Linux and other general software, power consumption can be reduced to under 1 W.  Of course the tradeoff is that only one compressed model (in this case MobileNet) is supported, and flexibility in model parameters, such as input image resolution and convolutional layer resolution, is limited.

Below are some lab pics showing internal assembly and cabling of the AIY vision kit, which incorporates a Movidius MA2450 device (aka Myriad 2).  The assembly is formed by attaching a daughtermodule with the Movidius device and a v2 camera module to a Raspberry Pi Zero W.  The Zero W is required as it has a fine pitch FPC (Flexible Printed Circuit) connector for the camera.

The image below shows the Raspberry Pi Zero W module by itself, without the Movidius daughtermodule attached.  The 40-pin header is used by the Raspberry Pi community for expansion purposes by attaching a wide variety of daughtermodules.  Note the camera connector at left, and also the ACT (activity LED) near the micro USB power connector, which is used to indicate boot activity.

![Raspberry Pi Zero W module without daughtermodule and camera connected](https://raw.githubusercontent.com/signalogic/SigDL/master/images/Raspberry_pi_w_zero_basic_config_w_labels.jpeg "Raspberry Pi Zero W module without daughtermodule and camera connected")

You might ask, where is the memory ?  How can the PCB be so small ?  The answer is the Raspberry Pi is a highly advanced, state-of-the-art example of embedded system engineering.  Most impressive is the small size of the PCB, which eliminates all FBGA packages except a Broadcom ARM SoC, and uses PoP technology (package on package) to stack memory directly on top of the SoC.

&nbsp;<br/>

The image below shows dmesg after a Raspberry Pi Zero W boot, still without the Movidius vision module or camera attached.  Note that the Raspberry Pi SoC contains a simple onchip boot loader which does a few SoC initialization steps, but from that point forward is entirely dependent on micro SD card contents (the boot process is [documented on this stackexchange thread](https://raspberrypi.stackexchange.com/questions/10489/how-does-raspberry-pi-boot/10490#10490)).

![Raspberry Pi Zero W module boot, without daughtermodule and camera connected](https://raw.githubusercontent.com/signalogic/SigDL/master/images/Raspberry_pi_w_zero_basic_config_boot_dmesg.jpeg "Raspberry Pi Zero W module boot, without daughtermodule and camera connected")

The next set of images show the Raspberry Pi Zero W module with the Movidius vision daughtermodule attached, and the camera connected, from different perspectives. 

![AIY Vision Kit, including Raspberry Pi Zero W module, Movidius MA2450 module, and v2 camera](https://raw.githubusercontent.com/signalogic/SigDL/master/images/AIY_Movidius_internal_assembly_and_cabling.jpeg "AIY Vision Kit, including Raspberry Pi Zero W module, Movidius MA2450 module, and v2 camera")

&nbsp;<br/>

![AIY Vision Kit partial assembly (including Raspberry Pi Zero W module, Movidius MA2450 module, and v2 camera](https://raw.githubusercontent.com/signalogic/SigDL/master/images/AIY_Movidius_partial_assembly2.jpeg "AIY Vision Kit partial assembly (including Raspberry Pi Zero W module, Movidius MA2450 module, and v2 camera")

&nbsp;<br/>

![AIY Vision Kit partial assembly (including Raspberry Pi Zero W module, Movidius MA2450 module, and v2 camera](https://raw.githubusercontent.com/signalogic/SigDL/master/images/AIY_Movidius_partial_assembly3.jpeg "AIY Vision Kit partial assembly (including Raspberry Pi Zero W module, Movidius MA2450 module, and v2 camera")

&nbsp;<br/>

The image below shows dmesg display, grepped for "googlevision", after booting with the Movidius vision module and camera attached.  A couple of notes about AIY nomenclature:

  -as noted above, the Movidius ASIC chip is also known as "Myriad"
  
  -"Bonnet" is a typical Googly word for daughtermodule
  
Terms like AIY and Bonnet are friendly, but make no mistake:  when solving embedded target debug and development problems, it's important to keep in mind that embedded system fundamentals such as small form-factors, reduced power consumption and heat, expansion options, JTAG debug interface, and other key areas haven't changed since the 1990s.  Embedded system development and debug has a long history of established methods and techniques, even if Raspberry Pi and Google developers would like to apply marketing terminology.

![dmesg display after booting Raspberry Pi Zero W module with Movidius MA2450 module and v2 camera attached](https://raw.githubusercontent.com/signalogic/SigDL/master/images/Raspberry_pi_w_zero_Myriad_vision_module_boot.jpeg "dmesg display after booting Raspberry Pi Zero W module with Movidius MA2450 module and v2 camera attached")

&nbsp;<br/>

<a name="TexasInstrumentsC66x"></a>
## Texas Instruments c66x

Deep learning model support for c66x multicore CPUs is pending support from Texas Instruments. Fundamentally, c66x devices are able to perform energy-efficient, high performance deep learning, as demonstrated in this [deep learning embedded target comparison paper by Dr. Nachiket Kapre](http://nachiket.github.io/publications/caffepresso_cases2016.pdf) at the University of Waterloo.  Whether TI will embrace the need for a cloud based c66x deep learning solution to support c66x device-accurate  model compression, re-training, and testing for embedded targets -- and the required investment, resources, and third-party support -- is a pending question.

As a potential basis for a c66x device-accurate deep learning cloud solution, the architecture shown in the pictures and links below already works.  Currently this architecture supports OpenCV, C/C++, Python, c66x math and DSP libraries, 8-bit MAC operations (32-bit accumulation), and other essential deep learning components, but a substantial amount of additional work is needed to create a deep learning CICD reference solution.  At minimum, such a solution should incorporate TensorFlow, Caffe, the deep learning and compression models mentioned in the Readme intro above.

The images below show multiple 64-core PCIe cards (up to 384 or more c66x cores depending on server form-factor, suitable for training on high resolution data sets), in standard 1U and 2U Dell and HP servers.

![Dell R720 server with 128 c66x cores, suitable for cloud based deep learning training, testing, and model compression](https://raw.githubusercontent.com/signalogic/SigDL/master/images/Dell_R720_128_c66x_cores.jpg "Dell R720 server with 128 c66x cores, suitable for cloud based deep learning training, testing, and model compression")

![HP DL380 server with 128 c66x cores, suitable for cloud based deep learning training, testing, and model compression](https://raw.githubusercontent.com/signalogic/SigDL/master/images/HP_DL380_G9_128_c66x_cores.jpg "HP DL380 server with 128 c66x cores, suitable for cloud based deep learning training, testing, and model compression")

For detailed information on the underlying technology of combined c66x + x86 architecture solutions, see the TI wiki pages for [HPC](http://processors.wiki.ti.com/index.php/HPC), [c66x OpenCV](http://processors.wiki.ti.com/index.php/C66x_opencv), and [NFV Transcoding](http://processors.wiki.ti.com/index.php/NFV_Transcoding).

In addition to the above "pure cloud" solutions, the combined c66x + x86 architecture can also target small-form factor servers in the 30 to 70 W range.  The images below show a combination of Atom x86 and TI c66x cores in a mini-ITX format, with 32 to 64 c66x cores accessible as half-size PCIe card(s).

![TI c66x + x86 deep learning small server solution, using mini-ITX (iso view)](https://raw.githubusercontent.com/signalogic/SigDL/master/images/deep_learning_c66x_x86_server_32cores_iso_view.png "TI c66x + x86 deep learning small server solution, using mini-ITX (iso view)")

In the mini-ITX images, the motherboard is a dual core Atom (C2358 @ 1.74 GHz) with 4x GbE interfaces, and 8 GB DDR3 mem @ 1333 MHz.  A more suitable motherboard would be a x5-E3940 quad core Atom, with either a x4 or x8 PCIe slot (or mPCIe interface).

![TI c66x + x86 deep learning small server solution, using mini-ITX (top view)](https://raw.githubusercontent.com/signalogic/SigDL/master/images/deep_learning_c66x_x86_server_32cores_top_view.png "TI c66x + x86 deep learning small server solution, using mini-ITX (top view)")

As an interesting side note, besides x86 it's also possible to combine a half-size c66x PCIe card Tegra GPU SoCs, for example the Jetson TX2 board has an x4 PCIe slot (see Jetson pics above).  What could be done with a next-gen Tegra (say 128 CUDA cores and 128 Tensor cores) and 32 c66x cores ?

If you are using or considering to use TI c66x and wish to discuss this either with Signalogic or 3-way with TI Sr. Management, please contact Signalogic.

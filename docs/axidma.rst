Lab: Axistream Single DMA (axis)
================================

Simple streaming example
************************
In this example we learn how to use `Xilinx AXI_DMA <https://www.xilinx.com/products/intellectual-property/axi_dma.html>`_ to create streaming interfaces for and IP.

This class will not go too deep into AXI protocols and Vivado, but a nice tutorial of the AXI Direct Memory Access (DMA) exists `here. <https://www.fpgadeveloper.com/2014/08/using-the-axi-dma-in-vivado.html>`_

1) Vivado HLS: Generating RTL code from C/C++ code
-------------------------------------------------

In this section you learn how to create a project in Vivado HLS, synthesize your code, and generate RTL.

1.1) Download code and create a Vivado HLS project
#################################################

Download and unzip `streamMul.zip. <https://bitbucket.org/akhodamoradi/pynq_interface/downloads/streamMul.zip>`_ Generate your project using the provided script.tcl file:

Linux: open a terminal, make sure your environment is set, navigate to streamMul folder, and run the following ::

    $ vivado_hls script.tcl

Windows: Open Vitis and create a New Project and import **streamMul.cpp** and **streamMul.hpp** and set **smul** as the top function.

.. image:: https://github.com/KastnerRG/Read_the_docs/raw/master/docs/image/manual_add.png

**Your code is not complete!**, modify your code to become same as the following:

.. code-block :: c++

#include "ap_axi_sdata.h"
#include "hls_stream.h"

typedef ap_axiu<32, 0, 0, 0> trans_pkt;

void smul(hls::stream< trans_pkt > &INPUT, hls::stream< trans_pkt > &OUTPUT)
{
#pragma HLS INTERFACE s_axilite port = return bundle = CTRL

#pragma HLS INTERFACE axis port=INPUT
#pragma HLS INTERFACE axis port=OUTPUT
        trans_pkt data_p;

        INPUT.read(data_p);
        data_p.data *= 2;
        OUTPUT.write(data_p);
}


In this lab, since we are using an ap_axiu struct for out I/O variables, the `last` bit is handled for us. We must interact with them this way because we are dealing with an AXI stream, not an array.

1.2) Generate RTL code and export it
####################################

Click on Run **C Synthesis** to generate RTL code. After it is done, you can check your resource utilization and timing report. Your latency is unknown (?) because your loop size (*length*) is a variable.

.. image:: https://github.com/KastnerRG/pp4fpgas/raw/master/labs/images/dma3.png

Now you can export your RTL code by clicking on **Export RTL**:

.. image:: https://github.com/KastnerRG/pp4fpgas/raw/master/labs/images/dma4.png

After exporting is done, you can close and exit from Vitis.

2) Vivado: Generating bitstream from RTL code
---------------------------------------------

In this section we import our RTL code from last section, add some required IPs, and generate our bitstream

2.1) Create a new Vivado project
################################

Open your Vivado tool and create a new project. Select an appropriate location for your project and leave the default project name as is (**project_1**).

Select **RTL Project** and check **Do specify not sources** at this time.

Select **xc7z020clg400-1** for your part:

.. image :: https://bitbucket.org/repo/x8q9Ed8/images/3090594305-pynq4.png

2.2) Import RTL code
####################

Under **Flow Navigator**, click on **IP Catalog**. Right click on the opened window and select **Add Repository**. Navigate to your **Vivado HLS project > solution1 > impl > ip** and select it:

.. image :: https://bitbucket.org/repo/x8q9Ed8/images/3508043996-pynq5.png

2.3) Add IPs to your design
###########################
Under **Flow Navigator**, click on **Create Block Design**. Leave the design name as is (*design_1*). In the newly opened window you can add IPs by clicking on the plus sign.

Add **ZYNQ7 Processing System** to your design:

.. image :: https://bitbucket.org/repo/x8q9Ed8/images/3814633603-pynq6.png

Double click on **ZYNQ7 IP** to customize it. In the opened window, double click on **High Performance AXI 32b/64b Slave Parts**:

.. image :: https://bitbucket.org/repo/x8q9Ed8/images/148617913-pynq7.png

Select and check **S AXI HP0 interface**:

.. image :: https://bitbucket.org/repo/x8q9Ed8/images/3126944786-pynq8.png

Add a **Smul** to your design and rename it to **smul**:

Add a **AXI Direct Memory Access** to your design and rename it to **smul_dma**. 

.. image:: https://github.com/KastnerRG/pp4fpgas/raw/master/labs/images/dma5.png

Double click on your **AXI DMA** and change the following parameters: 1) uncheck **Enable Scatter Gather** Engine. 2) Change **Width of Buffer Length Register** to 23:

.. image:: https://github.com/KastnerRG/pp4fpgas/raw/master/labs/images/dma6.png

Add a **Constant** to your design

2.4) Manual connections
#######################

Connect the following ports:

**smul::OUTPUT_r to smul_dma::S_AXIS_S2MM**

**smul_dma::M_AXIS_MM2S to smul::INPUT_r**

**xlconstant_0 to smul::ap_ctrl::ap_start**

.. image:: https://github.com/KastnerRG/pp4fpgas/raw/master/labs/images/dma7.png

2.5) Automatic connections
##########################

Now you can leave the rest of the connections to the tool. There should be a highlighted strip on top of your diagram window.

1. Click on **Run Block Automation**

2. Click on **Run Connection Automation** and select all

.. image :: https://bitbucket.org/repo/x8q9Ed8/images/2236315451-pynq12.png

3. **IMPORTANT!** you have to click again on **Run Connection Automation**

.. image :: https://bitbucket.org/repo/x8q9Ed8/images/1550495145-pynq13.png

At this point your design should look like this:

.. image:: https://github.com/KastnerRG/pp4fpgas/raw/master/labs/images/dma8.png

2.6) Generate bitstream
#######################

1. Save your design **CTRL+S** or **File > Save Block Design.**

2. Validate your design: **Tools > Validate Design**

3. In Sources, right click on **design_1**, and **Create HDL Wrapper**. Now you should have **design_1_wrapper.**

4. Generate bitstream by clicking on **Generate Bitstream** in **Flow Navigator**

2.7) Note required addresses and copy generated files
####################################################

After bitstream generating process is done, open **Address Editor** from **window** menu.

Note that **smul address** is **0x40000000**, we need this address in our host program for sending **length** data.

#TODO: Need to update the image. 
.. image :: https://github.com/KastnerRG/pp4fpgas/raw/master/labs/images/dma8.png

In sources, expand **design_1_wrapper::design_1::design_1::streamMul::smul::design_1_smul_0_0::inst : smul**, double click on **smul_CTRL_s_axi_U**, and note the address for **length_r** is **0x10**. We need this address in our host program.

.. image :: https://bitbucket.org/repo/x8q9Ed8/images/2224243640-pynq18.png

Copy your **project directory > project_1 > project_1.runs > impl_1 > design_1_wrapper** to your **project directory > project_1** and rename it to **smul.bit.** 

Copy your **project directory > project_1 > project_1.gen > sources_1 > bd > design_1 > hw_handoff > design_1.hwh** to your **project directory > project_1** and rename it to **smul.hwh**.

You should have both **smul.bit** and **smul.hwh**.

You can close and exit from Vivado tool.

3) Host program
---------------

In this section we use Python to test our design.

3.1) Move your files
####################

Create a new folder in your PYNQ board and move both **smul.bit** and **smul.hwh** into it.

3.2) Python code
################

Create a new Jupyter notebook and run the following code to test your design:

.. code-block :: python3

	import time
	from pynq import Overlay
	import pynq.lib.dma
	import numpy as np
	from pynq import MMIO
	import random

	ol = Overlay('./design_1.bit') # check your path
	ol.download() # it downloads your bit to FPGA
	print("FPGA is programmed")


	# Getting HLS IP register map  
	hls_ip = ol.smul
	print(hls_ip)
	hls_ip.register_map


	dma = ol.smul_dma
	dma_send = ol.smul_dma.sendchannel
	dma_recv = ol.smul_dma.recvchannel

.. code-block :: python3

	print("Starting HLS IP")
	hls_ip.register_map
	CONTROL_REGISTER = 0x0
	hls_ip.write(CONTROL_REGISTER, 0x81) # 0x81 will set bit 0
	hls_ip.register_map
	
	from pynq import allocate
	import numpy as np
	data_size = 20
	input_buffer = allocate(shape=(data_size,), dtype=np.uint32)
	print("Sending data!")
	
	# Prepare input data 
	for i in range(data_size):
		input_buffer[i] = i+5

	print("Starting DMA transfer")
	dma_send.transfer(input_buffer)


	# Transfer the data and receive the data back 
	output_buffer = allocate(shape=(data_size,), dtype=np.uint32)
	dma_recv.transfer(output_buffer)

	# Print the received data 
	for i in range(data_size):
		#print('0x' + format(output_buffer[i], '02x'))
		print(i, input_buffer[i], output_buffer[i])

    
print("Finished!!!")


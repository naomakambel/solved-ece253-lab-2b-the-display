Download Link: https://assignmentchef.com/product/solved-ece253-lab-2b-the-display
<br>
In Lab 2B you will connect an LCD Display to your FPGA board. This is the first lab where you will build a QP-nano automata to handle multiple peripherals. The goal of this lab is to control a graphical volume bar on the LCD using the rotary encoder to adjust the volume level. When the rotary encoder is inactive for 2 seconds the overlay graphic of the LCD will disappear, leaving only the background. A click on the rotary encoder is to be used to make the volume level 0. A second click should restore the volume to its previous value. In all cases, you should only repaint the necessary parts of the volume bar – do not repaint the entire screen on updates of the volume bar! In addition to this, four push-button switches will be used to print different text items on the LCD when they are clicked. Your system will consist of:

<ul>

 <li>TFT LCD Display (ILI9340/ILI9341)</li>

 <li>Rotary Encoder</li>

 <li>Push-Button Switches</li>

</ul>

<h1>1           Introduction</h1>

The LCD peripheral uses a SPI digital interface. A SPI interface to handle the communication with the peripheral’s microcontroller (ILI9340/ILI9341) could be implemented using software bit-level protocol coding or by making use of a hardware FSM peripheral controller. The benefit of software is program level flexibility, while a hardware interface allows for a much higher speed interface (5Mb/s = 10MHz clock in this case). Typical Arduino solutions for this display use a software interface, limiting the rate to about 100kb/s. The SPI module provided by Vivado is named AXI Quad SPI and for our LCD screen to work properly we also need to add another AXI GPIO module to the block design. This additional GPIO sets the ‘Data/Command’ value for the Display (which ought to be part of the SPI protocol as a 9-bit word), but somehow that option does not work in these displays. Effectively, the interface needs to synchronize the actions of SPI hardware and the other GPIO interface so that commands and data are interpreted properly.

In this lab, you need to modify the FPGA firmware to add the SPI and GPIO peripherals to support the encoder, display and four push-button switches. You will then need to build QP-nano code to implement a screen overlay “volume” that displays the value of “volume” stored as a fixed point value from 0-63. The present value of “volume” is conveyed to the viewer by using a rectangular image on the LCD that is at least 1/2 of the display wide, with a bar <em>proportional to the set value </em>and a fixed background. The background can be of any color with 40×40 triangles on top of it. Each triangle will be filled with a color that should be different from the background color. It is up to you whether you create shapes with or without borders. You can be as creative as you like with your choice of background. Two examples are shown in Figure 1.

Figure 1: Two different backgrounds. Any of these can be implemented. Be creative with your design.

The overlay for the volume bar should appear on top of the background. It should appear immediately as the encoder is turned, and stay on, updating dynamically until 2 seconds after the last encoder update. In addition to the volume bar, different text items should be displayed in response to pressing of different push-button switches (For example: pressing btn1 prints “Mode:1”, pressing btn2 prints “Mode:2”, etc.). If there is no activity on the encoder or switches for 2 seconds, the volume bar and text disappear, leaving only the background on the display. The goal here is to get two timed protocol behaviors and timed events into the software behavior. You can arrive at the final design in several ways, but we suggest making a separate program to run the display and then incrementally merge the desired behavior into a single QP state

chart.

Figure 2: The modifications to the hardware Block Design are shown above. Detailed instructions are provided below. In particular, note the reset spi port and the Clocking Wizard module.

<h1>2           Hardware Modifications</h1>

<ol>

 <li>Add GPIO module to supply the Data / Command signal.

  <ul>

   <li>With the Block Design open, Right click and choose ‘Add IP’.</li>

   <li>Type ‘AXI GPIO’ and add the module to the design.</li>

   <li>Left click on the newly created module and change the name to spi dc.</li>

   <li>Click on ‘Run Connection Automation’ and select ‘/spi dc/S AXI/’</li>

   <li>Right click on the spi dc module, select ‘Customize Block’, choose ‘All Outputs’ and make the GPIO width ‘1’.</li>

   <li>Left Click on the ‘+’ next to ‘GPIO’ to expand the IO ports.</li>

   <li>Right click on the port dash next to the terminal labeled ‘gpio io o[0:0]’ and ‘make external’. Select the port and rename it to spi dc.</li>

  </ul></li>

 <li>Createing a 10MHz SPI Clock (ext spi clk )

  <ul>

   <li>Add a new “Clocking Wizard” block.</li>

   <li>Connect the input of this clock “clk in1” from the “ui clk” (main system clock) of the MIG.</li>

   <li>Right click on the new ‘Clocking Wizard’, select ‘Customize Block’ and click on the tab ‘Output Clocks’.</li>

   <li>On the first tab, Ensure that the input clock is set to 100MHz.</li>

   <li>Switch to the “Output Clocks” tab, ensure only clk out1 is selected and set it to 10MHz.</li>

   <li>This results in an effective SPI clock of 5MHz(the SPI block divides by 2). You may increase this clock to 20MHz for 10MHz SPI clock if you wish.</li>

   <li>Uncheck the “reset” and “locked” options in the “optional input/output” section.</li>

  </ul></li>

 <li>Add SPI module

  <ul>

   <li>With the Block Design open, Right click and choose ‘Add IP’.</li>

   <li>Type ‘AXI Quad SPI’ and add the module to the design.</li>

   <li>Left click on the newly created module and change the name to spi.</li>

   <li>Click on ‘Run Connection Automation’ and select ‘/spi/AXI lite/’</li>

   <li>Connect the Clocking Wizard pin clk out1 to ext spi clk</li>

   <li>Right click on spi module, select ‘Customize Block’ and make the settings: <strong>Mode</strong>: Standard, <strong>Transaction Width</strong>: 8, <strong>Frequency Ratio</strong>: 2, <strong> of Slaves</strong>: 1, Check ‘Enable Master Mode’, Check ‘Enable FIFO’, <strong>FIFO</strong></li>

  </ul></li>

</ol>

<strong>Depth: </strong>256. UNCHECK Enable STARTUPE2 Primitive.

<ul>

 <li>Left Click on the ‘+’ next to ‘SPI 0’ to expand the IO ports.</li>

 <li>Right Click on the port dash next to the terminal labeled ‘io0 o (letter i, letter o, number zero, underscore, letter o) and choose ‘Make External’.</li>

 <li>Repeat the step to ‘Make External’ for the signals sck o, ss o[0:0]</li>

 <li>If any of these ports end up with a trailing “ 0” on the end, remove it so they end up with the same name as the</li>

</ul>

constraints below.

<ul>

 <li>Right click near the btnCpuReset pin and ‘Create Port’.</li>

 <li>Name the new port reset spi and select ‘output’.</li>

 <li>Draw a wire to connect reset spi to the same wire at btnCpuReset</li>

</ul>

<h2>2.1         Update Constraints File</h2>

You now have to assign pins for the GPIO and the SPI signals spi dc<em>, </em>io0 o<em>, </em>sck o<em>, </em>ss o<em>, </em>and reset spi by updating the constraints file. You may want to use a Pmod header (for example, JB) for convenience. For the rotary encoder, the Pmod header pins should have been already assigned from Lab 2A.

<h2>2.2         Export to xSDK</h2>

<strong>After modifying your hardware, make sure that the depth of the debug core (created in Lab 2A) is less than or equal to 32768. Otherwise you may run out of space on the FPGA, and implementation will fail. </strong>Run

synthesis, implementation, generate a new bitstream and export your Vivado design to xSDK. Make sure the Board Support Package which you intend to use for the lab is updated. To make sure it is updated, open the BSP project, open <strong>system.mss </strong>and click ‘Re-generate BSP Sources’. In SDK, do not forget to run the linker script, map all sections of the memory to the DDR (mig 7series 0 memaddr…) and <strong>not </strong>the BRAM. Also make sure that the the heap and stack sizes are 4096 (4 KB).

<h2>2.3         Wiring LCD Display to Nexys4 DDR Board</h2>

Figure 3 shows the pinouts for connecting the LCD display to the board. Please view the Nexys 4 manual for the pinout of the Pmod (or whatever pins you have used). The connection should be achieved by soldering a male pin header to the display (if it didn’t already come with one soldered in), and then using male-female jumper wires back to the FPGA board. In Figure 3, we have used a HiLetgo ILI9341 2.8” SPI TFT LCD Display which already comes with the pin header.

In particular note that sticking a pin header through the holes on the LCD PCB will not result any sort of reliable connection. Pin header to pin header connections rely on spring action to achieve a usable connection, which an empty hole on the LCD board does not supply. Soldering is necessary in this case.

<h1>3           Software Test</h1>

Create a new Application project and copy four files <strong>lcd test.c</strong>, <strong>lcd.c</strong>, <strong>lcd.h </strong>and <strong>fonts.c </strong>. Program your FPGA and run the lcd test on your FPGA board. If the ‘Hello World’ displays on the screen your hardware configuration is correct.

If the LCD does not work right away, check the wiring to the LCD and check the names of your io ports. The LCD should turn white and then draw the provided test display.

Figure 3: The Figure above shows HiLetgo ILI9341 2.8” SPI TFT LCD Display and how to connect the wires. The list of wires next to the display is in the format of LCD Pin Name – Color, signal. To connect the signal wires to your Nexys4 DDR board, follow your updated constraints file. The wire color is consistent for the signal name.

<h1>4           QP Nano Program Design</h1>

The goal of this lab is to control a graphical volume bar on the LCD using the rotary encoder to adjust the volume. When the rotary encoder is inactive for 2 seconds the volume bar will disappear, leaving only the background.

<ul>

 <li>Provided are two sets of example code, one is a partial example of QP nano. The other is an example of using the LCD. You should build your application around the QP nano framework.

  <ul>

   <li>Copy the QP nano example files as a template for your project.</li>

   <li>Fill out <strong>BSP Init() </strong>with hardware initializations for your peripherals.</li>

   <li>Define interrupt service routines like in <strong>c</strong>. These will be similar to Lab 2a. Edit <strong>bsp.c </strong>to connect interrupts to your Finite State Machine. This file is heavily commented, use the comments to help fill in the missing code.</li>

   <li><strong>c </strong>is an example state machine definition. Read it to understand how QP Nano models a FSM. Replace this with your own state machine.</li>

   <li><strong>q*.c </strong>and <strong>q*.h </strong>contain the QP Nano code. <strong>Do not modify these</strong>. Read to understand.</li>

   <li><strong>lcd test.c </strong>is an example of using the LCD. Include this into your QP Nano project.</li>

  </ul></li>

 <li>Create a simple custom LCD function to draw your background.</li>

 <li>Draw a rectangular block on the LCD to represent the ‘volume’ the rotary encoder is at. The graphic must be proportional to the set value.</li>

 <li>When twisting ‘left’ decrease the ‘volume’. Twisting ‘right’ increase the ‘volume’.</li>

 <li>When the rotary encoder push button is pressed, reset the ‘volume’ to 0.</li>

 <li>When different push-button switches on your board are pressed, print different text values to the display.</li>

 <li>Consider the design of your GUI and how the ‘overlay’ should work for your volume bar. An ‘overlay’ is a shape (sprite) which appears on top of a <strong>background </strong>that could incorporate a custom image. They are things like video game characters which walk around of a fixed image of a background.</li>

 <li>Use the timer to measure the number of seconds since last activity on the rotary encoder or push-button switches.</li>

 <li>When the inactivity time is 2 seconds, leave only the background.</li>

</ul>

<h1>5           Reporting</h1>

<ol>

 <li>Report on how you created your function for drawing the background and include a picture of your background pattern with your LCD photos.</li>

 <li>Report where you acquired your LCD from, and include a photo of the front and back of the LCD with your report.</li>

</ol>

(Let us know if you got it on time and if you had any problems getting it setup.)

<ol start="3">

 <li>Explain how you integrated the LCD and the Rotary Encoder using QP-nano.</li>

 <li>Explain how you detect inactivity and how you quickly remove the overlay graphic or text.</li>

 <li>Draw a statechart that models the behavior of your system.</li>

</ol>
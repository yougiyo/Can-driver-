
<h1>STM32F446 Bare-Metal CAN Driver</h1>
<p>A from-scratch CAN (Controller Area Network) peripheral driver for the <strong>STM32F446RE</strong> microcontroller, developed without relying on STM32 HAL or Cube libraries. The project covers loopback testing, normal-mode transmission, and normal-mode reception, with a structured set of examples to guide progressive integration.</p>
<hr>
<h2>Table of Contents</h2>
<ul>
<li><a href="#overview">Overview</a></li>
<li><a href="#hardware-requirements">Hardware Requirements</a></li>
<li><a href="#project-structure">Project Structure</a></li>
<li><a href="#examples">Examples</a>
<ul>
<li><a href="#0_test--smoke-test">0_test — Smoke Test</a></li>
<li><a href="#1_f44_can_driver--core-driver">1_f44_can_driver — Core Driver</a></li>
<li><a href="#2_f44_can_loopback--loopback-mode">2_f44_can_loopback — Loopback Mode</a></li>
<li><a href="#3_f44_can_normal-mode_tx--normal-tx">3_f44_can_normal-mode_tx — Normal TX</a></li>
<li><a href="#3_f44_can_normal-mode_rx--normal-rx">3_f44_can_normal-mode_rx — Normal RX</a></li>
</ul>
</li>
<li><a href="#getting-started">Getting Started</a>
<ul>
<li><a href="#prerequisites">Prerequisites</a></li>
<li><a href="#cloning-the-repository">Cloning the Repository</a></li>
<li><a href="#building">Building</a></li>
<li><a href="#flashing">Flashing</a></li>
</ul>
</li>
<li><a href="#can-bus-wiring">CAN Bus Wiring</a></li>
<li><a href="#reference-documents">Reference Documents</a></li>
<li><a href="#license">License</a></li>
</ul>
<hr>
<h2>Overview</h2>
<p>This project implements a bare-metal driver for the bxCAN (Basic Extended CAN) peripheral found on the STM32F446RE, targeting the <strong>NUCLEO-F446RE</strong> development board. All register-level configuration is done by hand using CMSIS headers — no STM32 HAL, no generated code.</p>
<p>Key features:</p>
<ul>
<li>CAN peripheral clock and GPIO initialization (PA11 / PA12 for CAN1 RX/TX)</li>
<li>Bit-timing configuration (configurable baud rate)</li>
<li>Loopback mode for self-testing without external hardware</li>
<li>Normal mode with transmit mailbox management</li>
<li>Normal mode receive with FIFO polling</li>
<li>CMSIS headers included under <code>chip_headers/CMSIS</code> for portability</li>
</ul>
<hr>
<h2>Hardware Requirements</h2>
<table>
<thead>
<tr>
<th>Item</th>
<th>Details</th>
</tr>
</thead>
<tbody>
<tr>
<td>MCU Board</td>
<td>STM32 NUCLEO-F446RE</td>
</tr>
<tr>
<td>CAN Transceiver</td>
<td>SN65HVD230 or MCP2551 (for normal-mode examples)</td>
</tr>
<tr>
<td>Programmer</td>
<td>On-board ST-Link V2 (included on NUCLEO)</td>
</tr>
<tr>
<td>Power supply</td>
<td>3.3 V from NUCLEO USB</td>
</tr>
</tbody>
</table>
<blockquote>
<p><strong>Note:</strong> The loopback example (<code>2_f44_can_loopback</code>) requires <strong>no</strong> external transceiver — it loops CAN frames internally inside the MCU.</p>
</blockquote>
<hr>
<h2>Project Structure</h2>
<pre><code>Can-driver-/
│
├── 0_test/                         # Basic peripheral smoke test
├── 1_f44_can_driver/               # Reusable CAN driver source files
├── 2_f44_can_loopback/             # Loopback self-test (no hardware needed)
├── 3_f44_can_normal-mode_rx/       # Receive frames from CAN bus
├── 3_f44_can_normal-mode_tx/       # Transmit frames onto CAN bus
│
├── chip_headers/
│   └── CMSIS/                      # ARM CMSIS + STM32F4 device headers
│
├── metadata/                       # Project metadata
│
├── rm0390-stm32f446xx-...pdf       # STM32F446 Reference Manual (RM0390)
├── stm32f446re.pdf                 # STM32F446RE Datasheet
└── um1724-stm32-nucleo64-...pdf    # NUCLEO-64 User Manual
</code></pre>
<hr>
<h2>Examples</h2>
<h3><code>0_test</code> — Smoke Test</h3>
<p>A minimal project to verify the toolchain and board are working. Blinks the on-board LED (PA5 on NUCLEO-F446RE) using a simple delay loop.</p>
<p><strong>Use this to confirm your build environment is correct before moving to the CAN examples.</strong></p>
<hr>
<h3><code>1_f44_can_driver</code> — Core Driver</h3>
<p>The reusable driver module. Contains the CAN initialization routine, bit-timing setup, and send/receive primitives. The other examples depend on this code.</p>
<p>Key functions (check the source headers for the full API):</p>
<table>
<thead>
<tr>
<th>Function</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>CAN1_Init()</code></td>
<td>Initializes CAN1 peripheral, GPIO, and clocks</td>
</tr>
<tr>
<td><code>CAN1_SetBitTiming()</code></td>
<td>Configures prescaler, time quanta, and baud rate</td>
</tr>
<tr>
<td><code>CAN1_Transmit()</code></td>
<td>Loads a frame into a TX mailbox and requests transmission</td>
</tr>
<tr>
<td><code>CAN1_Receive()</code></td>
<td>Reads a received frame from RX FIFO0</td>
</tr>
</tbody>
</table>
<hr>
<h3><code>2_f44_can_loopback</code> — Loopback Mode</h3>
<p>Configures the CAN peripheral in <strong>loopback mode</strong>, where transmitted frames are immediately echoed back to the receive FIFO — no transceiver or second node required.</p>
<p><strong>Good for:</strong> Verifying your driver logic and bit-timing before connecting real hardware.</p>
<hr>
<h3><code>3_f44_can_normal-mode_tx</code> — Normal TX</h3>
<p>Transmits CAN frames onto the bus at regular intervals. Requires a CAN transceiver connected to PA11 (CAN1_RX) and PA12 (CAN1_TX), and at least one other node (or a CAN analyzer) on the bus to provide acknowledgement.</p>
<hr>
<h3><code>3_f44_can_normal-mode_rx</code> — Normal RX</h3>
<p>Listens on the CAN bus and receives incoming frames into FIFO0. Configure the acceptance filter to receive only specific CAN IDs, or open the filter to accept all frames.</p>
<hr>
<h2>Getting Started</h2>
<h3>Prerequisites</h3>
<ul>
<li><strong>Toolchain:</strong> <code>arm-none-eabi-gcc</code> (GCC for ARM Cortex-M)</li>
<li><strong>Build system:</strong> <code>make</code></li>
<li><strong>Flasher:</strong> OpenOCD or STM32CubeProgrammer</li>
<li><strong>OS:</strong> Linux, macOS, or Windows (WSL recommended on Windows)</li>
</ul>
<p>Install on Ubuntu/Debian:</p>
<pre><code class="language-bash">sudo apt update
sudo apt install gcc-arm-none-eabi make openocd
</code></pre>
<h3>Cloning the Repository</h3>
<pre><code class="language-bash">git clone https://github.com/yougiyo/Can-driver-.git
cd Can-driver-
</code></pre>
<h3>Building</h3>
<p>Navigate into the example you want to build and run <code>make</code>:</p>
<pre><code class="language-bash">cd 2_f44_can_loopback
make
</code></pre>
<p>The compiled <code>.elf</code> and <code>.bin</code> files will appear in the build output directory.</p>
<h3>Flashing</h3>
<p>Using <strong>OpenOCD</strong> with the ST-Link on the NUCLEO board:</p>
<pre><code class="language-bash">openocd -f interface/stlink.cfg \
        -f target/stm32f4x.cfg \
        -c &#34;program build/output.elf verify reset exit&#34;
</code></pre>
<p>Or using <strong>STM32CubeProgrammer</strong> GUI — connect via USB, select the <code>.elf</code> file, and click Download.</p>
<hr>
<h2>CAN Bus Wiring</h2>
<p>For the normal-mode examples you need a CAN transceiver (e.g. SN65HVD230) between the MCU and the bus:</p>
<pre><code>STM32F446RE          SN65HVD230
──────────────       ─────────────
PA12 (CAN1_TX) ───→  TXD
PA11 (CAN1_RX) ←───  RXD
3.3V           ────  VCC
GND            ────  GND
                     CANH ──┐
                     CANL ──┴── to CAN bus (120 Ω termination at each end)
</code></pre>
<blockquote>
<p>The NUCLEO-F446RE <strong>does not</strong> include an on-board CAN transceiver. You must add one externally.</p>
</blockquote>
<p><strong>Default baud rate:</strong> 500 kbps (adjust the prescaler in the driver to change this).</p>
<hr>
<h2>Reference Documents</h2>
<p>The following official STMicroelectronics documents are included in the repository root:</p>
<table>
<thead>
<tr>
<th>File</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>rm0390-stm32f446xx-...pdf</code></td>
<td>STM32F446 Reference Manual — full register descriptions for bxCAN</td>
</tr>
<tr>
<td><code>stm32f446re.pdf</code></td>
<td>STM32F446RE Datasheet — pin mapping and electrical specs</td>
</tr>
<tr>
<td><code>um1724-stm32-nucleo64-...pdf</code></td>
<td>NUCLEO-64 User Manual — board schematic and ST-Link wiring</td>
</tr>
</tbody>
</table>
<p>The most relevant section for this project is <strong>Chapter 32 – bxCAN</strong> in the Reference Manual (RM0390).</p>
<hr>
<h2>License</h2>
<p>No license file is currently included in this repository. If you intend to use or distribute this code, please contact the author <a href="https://github.com/yougiyo">@yougiyo</a> to clarify terms.</p>
<hr>
<p><em>Contributions, bug reports, and pull requests are welcome.</em></p>

</body></html>

# RISC-V Steel SoC </br><small>Reference Guide</small>

## Introduction

RISC-V Steel SoC is a configurable system-on-chip design featuring RISC-V Steel Core, a RAM memory and a UART module. It comes with an [API](steel-api.md) for software development that makes it easier for hardware engineers to develop and deploy new RISC-V embedded applications.

The figure below depicts the main components of RISC-V Steel SoC and how they interconnect. Detailed information about each device in the system is provided in the following sections.

<figure markdown>
  ![Image title](images/rvsteel-soc.drawio.svg){#figure-1}
  <figcaption><strong>Figure 1</strong> - Overview of RISC-V Steel SoC</figcaption>
</figure>

## General information

This section provides information about the source files, the input/output signals and the configuration parameters of RISC-V Steel SoC.

### Source files

**Table 1** - RISC-V Steel SoC source files

| File                 | Module name      | Location                |  Description                    |
| -------------------- | ---------------- | ----------------------- |------------------------------ |
| **`rvsteel-soc.v`**  | **rvsteel_soc**  | `riscv-steel/hardware/` | Top module of RISC-V Steel SoC |
| **`rvsteel-core.v`** | **rvsteel_core** | `riscv-steel/hardware/` | RISC-V Steel Core              |
| **`ram-memory.v`**   | **ram_memory**   | `riscv-steel/hardware/` | RAM memory                     |
| **`uart.v`**         | **uart**         | `riscv-steel/hardware/` | UART                           |
| **`system-bus.v`**   | **system_bus**   | `riscv-steel/hardware/` | System Bus                     |

### Input/output signals

**Table 2** - RISC-V Steel SoC top module input and output signals

| Pin name       | Direction | Size  | Description          |
| -------------- | --------- | ----- | -------------------- |
| **clock**      | Input     | 1 bit | Clock input.         |
| **reset**      | Input     | 1 bit | Reset (active-high). |
| **uart_rx**    | Input     | 1 bit | UART receiver pin. Must be connected to the transmitter (`TX`) pin of another UART device. |
| **uart_tx**    | Output    | 1 bit | UART transmitter pin. Must be connected to the receiver (`RX`) pin of another UART device. |

### Configuration

**Table 3** - Configuration parameters of RISC-V Steel SoC

| Parameter name       | Default value  | Value type and description                                                                    |
| -------------------- | -------------- | --------------------------------------------------------------------------------------------- |
| **BOOT_ADDRESS**     | 32'h00000000   | 32-bit hexadecimal value. Memory address of the first instruction to be fetched and executed. |
| **CLOCK_FREQUENCY**  | 50000000       | Integer. Frequency (in hertz) of the **clock** input signal.                                  |
| **UART_BAUD_RATE**   | 9600           | Integer. UART baud rate (in bauds per second).                                                |
| **MEMORY_SIZE**      | 8192           | Integer. RAM memory size (in bytes).                                             |
| **MEMORY_INIT_FILE** | (empty string) | String. Path to a memory initialization file.                                                 |

## Instantiation template

An instantiation template for RISC-V Steel SoC top module is provided below.

``` systemverilog
rvsteel_soc #(

  // Configuration parameters. For more information read the 'Configuration'
  // section of RISC-V Steel SoC Reference Guide

  .BOOT_ADDRESS             (),  // Default value: 32'h00000000
  .CLOCK_FREQUENCY          (),  // Default value: 50000000
  .UART_BAUD_RATE           (),  // Default value: 9600
  .MEMORY_SIZE              (),  // Default value: 8192
  .MEMORY_INIT_FILE         ())  // Default value: Empty string (uninitialized)

  rvsteel_soc_instance (

  // I/O signals. For more information read the 'Input/output signals'
  // section of RISC-V Steel SoC Reference Guide

  .clock                    (),  // Connect this pin to a clock source
  .reset                    (),  // Active-high reset. Connect to a reset switch
  .uart_rx                  (),  // Connect to the TX pin of other UART
  .uart_tx                  ()   // Connect to the RX pin of other UART

);
```

## Integrated devices

This section provides further information about the devices instantiated in RISC-V Steel SoC top module.

### RISC-V Steel Core

RISC-V Steel Core is the processing unit of RISC-V Steel SoC. Its design is quite large so it has its own [Reference Guide](core-reference.md). Please check it out for more information.

### RAM memory

RISC-V Steel SoC has a RAM memory tightly coupled to the processor core, with read/write latency of a single clock cycle.

The memory size can be changed by adjusting the `MEMORY_SIZE` parameter (see [Configuration](#configuration)). Memory addresses start at `0x00000000` and end at `0x(MEMORY_SIZE-1)`.

Check out the [Software Guide](software-guide.md) for instructions on how to generate a memory initialization file and load it into memory at startup.

### UART

RISC-V Steel SoC has an UART module with configurable baud rate. The module works with 8 data bits, 1 stop bit, no parity bits and no flow control signals (most UARTs work the same way).

[Steel API](steel-api.md) provides a set of function calls for sending and receiving data over the UART. Check it out for more information.

### System Bus

The system bus module interconnects RISC-V Steel Core (manager device) to the UART and the RAM memory (subordinate devices), as shown in [Figure 1](#figure-1). The module multiplexes the signals from the processor's memory interface to the appropriate subordinate device according to the address the processor is requesting.

Each subordinate device in RISC-V Steel SoC is assigned a portion of the address space. The address range to which each device is mapped is shown in [Table 4](#table-4).

## Memory Map

Like all RISC-V systems, I/O devices in RISC-V Steel SoC are *memory mapped*, so they share the address space and are assigned a region within it. The table below shows the address range that each device in RISC-V Steel SoC is assigned.

???+ info
    Reading data from a region not mapped to any device will return 0. Writing data to these regions will have no effect.

**Table 4**{#table-4} - Memory Map of RISC-V Steel SoC

| Start address     | Final address       | Mapped device              |
| ----------------- | ------------------- | -------------------------- |
| `0x00000000`      | `0x(MEMORY_SIZE-1)` | RAM memory                 |
| `0x(MEMORY_SIZE)` | `0x7fffffff`        | ---------                  |
| `0x80000000`      | `0x80000004`        | UART                       |
| `0x80000005`      | `0xffffffff`        | ---------                  |

## Adding new devices

A new device can be added to RISC-V Steel SoC by adapting the system bus module (**`system-bus.v`**) and the top module (**`rvsteel-soc.v`**). All changes that need to be made in these files were left as comments in their source code. You only need to uncomment.

All source code that needs to be uncommented follow the template below:

``` systemverilog
  /* Uncomment to add new devices

  ... code for adding the new device ...

  */
```

</br>
</br>
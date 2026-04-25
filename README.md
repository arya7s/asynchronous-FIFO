# Asynchronous FIFO 

## Overview
This project implements a parameterized Asynchronous FIFO (First-In-First-Out) buffer in Verilog. An asynchronous FIFO is a fundamental hardware design used to safely and reliably transfer data between two systems that operate on entirely different, independent clock frequencies—a process known as Clock Domain Crossing (CDC).

## How It Works
When passing data between different clock domains, sampling transitional signals can lead to metastability, causing data corruption. This design mitigates that risk by using **Gray code pointers** and **2-stage synchronizers**. 

Instead of passing standard binary pointers across the clock boundary, the read and write pointers are converted to Gray code. Because Gray code only changes one bit at a time, it guarantees that the synchronizing flip-flops will never capture a transitional, invalid state. Furthermore, the design uses "pessimistic" full and empty flags, meaning the FIFO will report being full or empty slightly longer than strictly necessary, acting as a safety buffer to prevent accidental data overwrites or invalid reads.

## Module Architecture
The design is partitioned into five core, synthesizable modules and one simulation testbench:

* **`FIFO` (Top Level):** The wrapper module that instantiates and connects all sub-components, routing the independent read/write clocks, resets, and data buses.
* **`FIFO_memory`:** A parameterized dual-port RAM block. It writes data synchronously to the write clock domain and reads asynchronously based on the read address.
* **`wptr_full`:** Operates strictly in the write clock domain. It tracks the write pointers, converts them to Gray code, and calculates the `wfull` (Write Full) flag by comparing the current write pointer against the synchronized read pointer.
* **`rptr_empty`:** Operates strictly in the read clock domain. It tracks the read pointers, converts them to Gray code, and calculates the `rempty` (Read Empty) flag by comparing the current read pointer against the synchronized write pointer.
* **`two_ff_sync`:** A standard 2-stage flip-flop synchronizer. Two instances of this module are used to safely bridge the Gray-coded read and write pointers across the clock boundaries.
* **`FIFO_tb`:** A verification testbench that simulates asynchronous read and write operations at different clock speeds to test data integrity and the behavior of the full/empty flags.

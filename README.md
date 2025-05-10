# Neural Network BRAM Interface - README

### Module:
nn_controller_bram_interface

### Design Date:
04/26/2025

### Description:
This module provides a memory-mapped interface between a Block RAM (BRAM) and a neural network controller (`nn_controller`). It allows external software or logic to read/write data via BRAM and control neural network operations.

The design assumes a native BRAM interface and uses a fixed address map to interact with the neural network.


## Interface Signals

### Inputs:

| Port    | Description |
| ------- | ------- |
| clk_i  | Clock signal    |
| rst_i  | Active-low reset    |
| BRAM_dout[31:0]  | Data output from BRAM    |
| BRAM_addr[31:0]  | BRAM address to access    |


### Outputs:

| Port    | Description |
| ------- | ------- |
| BRAM_clk          | Clock signal to BRAM (same as clk_i) |
| BRAM_din[31:0]    | Data input to BRAM |
| BRAM_en           | BRAM enable (always 1) |
| BRAM_rst          | Reset signal to BRAM (same as rst_i) |
| BRAM_we[3:0]      | BRAM write enable |


## BRAM Address Map

| Memory Range    | Name | Description |
| --------------- | ----------- | ----------- |
| 0x00000000   | ADDR_RANDOMSEED_BASE | Random seed input to the neural network
| 0x00000001   |  ADDR_INPUTS_BASE  | Input data
| 0x00000002   |  ADDR_TARGETS_BASE  | Target data
| 0x00000003   |  ADDR_RUNCOMMAND | Run command signal
| 0x00000004   |  ADDR_COMMANDCOMPLETE | Command completion flag
| 0x00000005   | ADDR_OUTPUTS_BASE  | Output data
| 0x00000006  |  ADDR_RUNCOMMAND_0 | Run command signal

## Note:

> Software must write values to the appropriate BRAM addresses and monitor completion via ADDR_COMMANDCOMPLETE.

## Usage:

1. Write random seed to 0x00000000 (ADDR_RANDOMSEED_BASE)
2. Write input to 0x00000001 (ADDR_INPUTS_BASE)
3. Write target to 0x00000002 (ADDR_TARGETS_BASE)
4. Write 1 to 0x00000003 (ADDR_RUNCOMMAND) to issue a run command
5. Wait/Read for command complete from 0x00000004 (ADDR_COMMANDCOMPLETE)
6. Write 0 to 0x00000006 (ADDR_RUNCOMMAND) to remove a run command_0
7. Read output from 0x00000005 (ADDR_OUTPUTS_BASE)



## Sample Code

```
from pynq import Overlay
import time
import random

# Load your overlay (adjust path as needed)
overlay = Overlay("/home/xilinx/pynq/overlays/nn_controller/nn_controller.bit")

# Access your custom IP
nn_controller = overlay.nn_controller_bram_interface_0  # adjust name as necessary

# Define addresses
ADDR_RANDOMSEED_BASE     = 0x00
ADDR_INPUTS_BASE         = 0x04
ADDR_TARGETS_BASE        = 0x08
ADDR_RUNCOMMAND          = 0x0C
ADDR_COMMANDCOMPLETE     = 0x10
ADDR_OUTPUTS_BASE        = 0x14

# Step 1: Write random seed
random_seed = random.randint(0, 2**32 - 1)
nn_controller.write(ADDR_RANDOMSEED_BASE, random_seed)
print(f"Random Seed Written: {hex(random_seed)}")

# Step 2: Write input
input_value = 0x12345678
nn_controller.write(ADDR_INPUTS_BASE, input_value)
print(f"Input Written: {hex(input_value)}")

# Step 3: Write target
target_value = 0x9ABCDEF0
nn_controller.write(ADDR_TARGETS_BASE, target_value)
print(f"Target Written: {hex(target_value)}")

# Step 4: Issue run command
nn_controller.write(ADDR_RUNCOMMAND, 1)
print("Run command issued.")

# Step 5: Wait for command complete
print("Waiting for command to complete...")
while nn_controller.read(ADDR_COMMANDCOMPLETE) == 0:
    time.sleep(0.001)

print("Command complete.")

# Step 6: Clear run command
nn_controller.write(ADDR_RUNCOMMAND, 0)
print("Run command cleared.")

# Step 7: Read output
output_value = nn_controller.read(ADDR_OUTPUTS_BASE)
print(f"Output Read: {hex(output_value)}")

```

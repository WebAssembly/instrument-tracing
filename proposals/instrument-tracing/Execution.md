# Execution

As native machine code, the trace instruction should execute as a `NOP`. On different platforms, this may have other semantic information. On Intel Architecture for example, The ID value should be placed into the `EBX` register, and then the following `NOP` sequence should be executed: `0x64 0x67 0x90 0x90 0x90`. `0x64 0x67` is a prefix to the first `NOP`, `0x90`, followed by 2 additional `NOP`s.

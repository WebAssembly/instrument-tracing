# Instrument and Tracing Proposal

The Instruction and Tracing Proposal gives WebAssembly developers the ability to add in a custom section that introduces native trace instructions. These native trace instructions will not impact functionality or the execution of the WebAssembly, but provide the developer and other tools a better ability to trace machine code instructions. By nature of this being a custom section, an engine can choose wether it not it wants to actually utilize this.

## Summary

The ability of tracing back code execution to the source code is critical for writing optimized software. This instrument allows WebAssembly developers to add in an instruction trace with an immediate value to start and stop tracing within a WebAssembly sequence. The instruction trace will insert a specific native code sequence that will identify a start or stop of a trace based on the immediate value. The first occurrence of the instruction associated with the immediate value will start the trace and the second occurrence of the instruction with the immediate value will stop the trace.

## Motivation

In order to determine the performance and optimization of WebAssembly on different architectures it is extremely helpful to be able to isolate a specific region of code to the source.  Being able to know the source that generated the specific machine code makes it possible to evaluate the use of the cache, register allocation, sequence of instructions and then be able to optimize the source code and improve the code generation.

## Overview

Instruction tracing in WebAssembly introduces a new custom section that does not change the results or execution of the code generated on the targeted platform. It is up to the implementation of the WebAssembly engine to implement a trace instruction from the custom section targeted for the platform. The immediate value shall be used to distinguish starting and stopping of trace instructions by the instrumentation code and have no impact on the functionality of the WebAssembly it is instrumenting.  There is no requirement for the WebAssembly runtime code to enforce a corresponding start and stop trace instructions of code being instrumented since it has no impact on the code or the runtime.

## Implementation details

The current implementation of the custom section is called `instTrace` and consists of a `uvar32` that describes how many trace instructions are encoded followed by a list of trace instructions. Each trace instruction is encoded as a `u32` offset into the code section of the WASM module and a `uvar32` immediate value, used as an id for the trace. During module decode, the code section offsets are translated to function indices and offsets and this information is stored for each function. Then during function decode at each byte offset where a trace instruction it is emitted.

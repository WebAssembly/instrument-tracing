# Instrument and Tracing Proposal

The instruction and tracing gives WebAssembly developers the ability to add in an instruction that does not impact the functionality or results of the application but provides the developer the ability to trace back machine code instructions back to the source code of the WebAssembly.

## Summary

The ability of tracing back code execution to the source code is critical for writing optimized software.  This instrument allows WebAssembly developers to add in an instruction trace with an immediate value to start and stop tracing within a WebAssembly sequence.  The instruction trace will create a specific WebAssembly instruction that will identify a start or stop of a trace based on the immediate value.  The first occurance of the instruction associated with the immediate value will start the trace and the second occurance of the instruction with the immediate value will stop the trace.  The trace instruction will be treated as a NOP in WebAssembly execution. 

## Motivation

In order to determine the performance and optimization of WebAssembly on different architectures it is extreamly helpful to be able to isolate a specific region of code to the source.  Being able to know the source that generated the specific machine code makes it possible to evaluate the use of the cache, register allocation, sequence of instructions and then be able to optimize both the source code and improve the code generation.

## Overview

Instruction tracing in WebAssembly introduces a new instruction that does not change the results or execution of the code generated on the targeted platform.  It is up to the implementation of the WebAssembly engine to implement the trace instruction targeted for the platfom.  The trace instruction look like an load immediate instruction but does not execute or operate on the instruction.  i32.itt immediate. The immediate value shall be used to distinguish starting and stoping of trace instructions by the instrumentation code and have no impact on the functionality of the WebAssembly it is intrumenting.  There is no requirement for the WebAssembly runtime code to enforce a corresponding start and stop trace instructions of code being intrumented since it has no impact on the code or the runtime.

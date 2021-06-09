# Tooling

## Code Generation

The custom section is inserted from a compiler intrinsic, `__builtin_wasm_trace(ID)`. This translates into a byte offset with an immediate value that can be stored into the custom section.

## Tracing

With or without tracing tools activated, there should be no change to the execution results with the trace instruction. 

With a tool attached, you should be able to configure how the resulting trace is collected, but the result of the code should stay the same.

### Tool Trace Example

**Note**: This WASM module is written in pseudo-code, trace instructions are encoded as custom sections with byte offsets, however this illustrates the point more clearly.

```wasm
(module
  (func $func (export "func")
    
  (trace_instruction 17)
  ;; point 1
  (trace_instruction 18)
  ;; point 2
  (trace_instruction 19)
  ;; point 3
  (trace_instruction 18)
  ;; point 4
  (trace_instruction 19)
  ;; point 5
  (trace_instruction 17)
  ;; point 6
  (trace_instruction 20)
  ;; point 7
  (trace_instruction 17)
 )
```

In pseudo-commands, we can demonstrate how traces would be collected.

__`trace -start:17 -stop:17:count2`__: The resulting trace should include points 1, 2, 3, 4, and 5

__`trace -start:18 -stop:19`__: The resulting trace should include point 2

__`trace -start:18:count2 -stop:19:count2`__ : The resulting trace should include point 4

__`trace -start:18:count2 -stop:17:count3`__: The resulting trace should include points 4, 5, 6, and 7

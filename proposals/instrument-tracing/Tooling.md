# Tooling

With or without tools activated, there should be no change to the execution results code with the trace instruction. 

With a tool attached, you should be able to configure how the resulting trace is collected, but the result of the code should stay the same.

## Example Code

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

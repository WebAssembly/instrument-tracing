# Example Code

Suppose we had the following simple C function to add 2 numbers.

```c
int add(int a, int b) {
    return a + b;
}
```

This translates to the following WASM module.

```wasm
(module
  (func $add (export "add")
    (param $0 i32) 
    (param $1 i32)
    (result i32)

    (i32.add
      (local.get $1)
      (local.get $0)
    )
  )
)
```

Ignoring the rest of the module, the code section would look like

```
0x0a0901 ; code section definition
0x0700   ; function header
0x2000   ; local 0
0x2001   ; local 1
0x6a     ; add
0x0b     ; end
]
```

Now suppose we want to add a trace instruction.

```c
int add(int a, int b) {
    __builtin_wasm_trace(17);
    int c = a + b;
    __builtin_wasm_trace(17);
    return c;
}
```

The code section would not change, but now we would have a custom section that appears after the code section.

```
    0x0015                    ; custom section
    0x09 0x696e73745472616365 ; len, instTrace
    0x02                      ; number trace instructions
    0x05000000 0x11           ; trace instruction at beginning of function
    0x0a000000 0x11           ; trace instruction at end of function
```

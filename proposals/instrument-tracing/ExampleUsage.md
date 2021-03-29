# Example Usage

## Code

Here is an example of an example kernel (SAXPY) surrounded by trace instructions

```wasm
(module
  ;; Try to lock a mutex at the given address.
  ;; Returns 1 if the mutex was successfully locked, and 0 otherwise.
  (func $saxpy (export "saxpy")
  ;; out ptr, scalar, x ptr, y ptr, size
  (param $0 i32)
  (param $1 f32)
  (param $2 i32)
  (param $3 i32)
  (param $4 i32)

  (local $5 i32)
  (local $6 i32)
  (local $7 i32)

  ;; inserted trace instruction with an immediate value of 0x11
  (trace_instruction 17)

  ;; saxpy implementation
  (block $label$1
   (br_if $label$1
    (i32.eqz
     (local.get $4)
    )
   )
   (if
    (i32.ne
     (local.get $4)
     (i32.const 1)
    )
    (block
     (local.set $7
      (i32.and
       (local.get $4)
       (i32.const -2)
      )
     )
     (loop $label$3
      (f32.store
       (i32.add
        (local.get $0)
        (local.tee $5
         (i32.shl
          (local.get $6)
          (i32.const 2)
         )
        )
       )
       (f32.add
        (f32.mul
         (f32.load
          (i32.add
           (local.get $2)
           (local.get $5)
          )
         )
         (local.get $1)
        )
        (f32.load
         (i32.add
          (local.get $3)
          (local.get $5)
         )
        )
       )
      )
      (f32.store
       (i32.add
        (local.get $0)
        (local.tee $5
         (i32.or
          (local.get $5)
          (i32.const 4)
         )
        )
       )
       (f32.add
        (f32.mul
         (f32.load
          (i32.add
           (local.get $2)
           (local.get $5)
          )
         )
         (local.get $1)
        )
        (f32.load
         (i32.add
          (local.get $3)
          (local.get $5)
         )
        )
       )
      )
      (local.set $6
       (i32.add
        (local.get $6)
        (i32.const 2)
       )
      )
      (br_if $label$3
       (local.tee $7
        (i32.sub
         (local.get $7)
         (i32.const 2)
        )
       )
      )
     )
    )
   )
   (br_if $label$1
    (i32.eqz
     (i32.and
      (local.get $4)
      (i32.const 1)
     )
    )
   )
   (f32.store
    (i32.add
     (local.get $0)
     (local.tee $4
      (i32.shl
       (local.get $6)
       (i32.const 2)
      )
     )
    )
    (f32.add
     (f32.mul
      (f32.load
       (i32.add
        (local.get $2)
        (local.get $4)
       )
      )
      (local.get $1)
     )
     (f32.load
      (i32.add
       (local.get $3)
       (local.get $4)
      )
     )
    )
   )
  )


  ;; inserted trace instruction with an immediate value of 0x12
  (trace_instruction 18)
 )

```

Calling from JavaScript

```js
WebAssembly.instantiateStreaming(fetch("saxpy.wasm"))
  .then((resp) => {
    //assume we compiled with a memory already created
    //assume we have some kind of allocator for that memory
    let inst = resp.instance;
    let memBuf = inst.exports.memory.buffer;
    let malloc = inst.exports.malloc;
    let free = inst.exports.free;
    let saxpy = inst.exports.saxpy;

    //helper object to store parameters
    let setupObj = {
      out_ptr: null,
      s: null,
      x_ptr: null,
      y_ptr: null,
      size: null,
    };

    //init the JS data
    setupObj.size = 2000 * 4;
    var x = new Float32Array(setupObj.size);
    var y = new Float32Array(setupObj.size);
    setupObj.s = Math.random() * 100 + 1;
    for (var i = 0; i < setupObj.size; i++) {
      x[i] = Math.random() * 100 + 1;
      y[i] = Math.random() * 100 + 1;
    }

    //allocate and copy over input arrays
    var byte_size = x.BYTES_PER_ELEMENT;
    var array_byte_size = setupObj.size * byte_size;
    setupObj.x_ptr = malloc(array_byte_size);
    setupObj.y_ptr = malloc(array_byte_size);

    let heap = new Float32Array(memBuf);
    //copy over data
    heap.set(x, setupObj.x_ptr / byte_size);
    heap.set(y, setupObj.y_ptr / byte_size);

    //allocate output
    setupObj.out_ptr = malloc(array_byte_size);

    //call our saxpy code
    saxpy(
        setupObj.out_ptr,
        setupObj.s,
        setupObj.x_ptr,
        setupObj.y_ptr,
        setupObj.size
      );

    free(setupObj.x_ptr);
    free(setupObj.y_ptr);
    free(setupObj.out_ptr);
  })
  .catch((reas) => {
    console.error(`Failed to compile: ${reas}`);
  });
```

## Tracing

Running the above code in the browser, we can observe the performance in a number of ways. We can use instruction tracing tools such as SDE to view native instructions executed. Using internal browser tracing, we can observe the effect of the code in terms of time. Both of these suffer from being too high of granularity. The traces are started and stopped manually and may or may not actually contain the code we wanted to observe. In the case of the instruction level trace, millions of instructions may be executed but we only care about a handful. In the case of a browser trace, we can maybe see what piece of code is executing, but not beyond a function level (ie we can see that `saxpy()` is slow, but not what is inside of `saxpy()` that makes it slow). 

These trace instructions provide a lower level of granularity that can be placed in production code. With no overhead and no other program changes (stack effects), they can be added and removed from code at will to allow tracing in both a development setting and on production sites. Since they execute what amounts to a NOP for all architecture, they have no impact on the performance of the code. For an instruction level trace, that `NOP` can be used as a trigger to start and stop the collection of data, giving just the code we want to look at. For a browser trace, it can serve multiple purposes. They can act as a breakpoint that can be toggled (similar to the `JS` `debugger` statement) or as an indicator to a profiler that it should start or stop a profile. The associated immediate values function as a id for each trace instruction, allowing tools to make smarter decisions such as `start after seeing ID 17` and `stop after seeing 4 ID 18's`.

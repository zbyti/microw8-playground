# The Complete MicroW8 Tutorial: From Basics to Advanced Demoscene

Welcome to the fascinating world of MicroW8, a virtual fantasy console designed for demosceners and sizecoders who want to create audiovisual masterpieces within strict hardware limitations. This tutorial is a comprehensive guide to programming on MicroW8 using the CurlyWASM language, combining fundamentals, advanced techniques, and the demoscene philosophy. Whether you're just starting your journey or you're an experienced programmer looking for sizecoding challenges, you'll find everything you need here to create impressive demos on this platform.
The tutorial is divided into three main parts and an inspiration section, which complement each other smoothly: 1.  **MicroW8 and CurlyWASM Basics** – introduction to the console, language, and key programming mechanisms. 2.  **Advanced Techniques** – in-depth discussion of optimization, memory management, demoscene effects, and sound. 3.  **Demoscene and Creativity** – the philosophy of the demoscene, key visual techniques, and a practical approach to demo creation. 4.  **Inspirations and Demoscene Resources** – an overview of iconic demos, examples, and communities.
Each section includes detailed explanations, code examples (CurlyWASM or pseudocode), optimization tips, and references to examples from files such as `mandelbrot.cwa`, `plasma_sizecoding_1.cwa`, or `urban_drift.cwa`. The document concludes with a glossary explaining key terms related to MicroW8, CurlyWASM, and the demoscene.
Prepare for a journey into a world where every byte matters, and creativity is born from limitations. Let's begin!
---

## Part 1: MicroW8 and CurlyWASM Basics

Greetings, coder! MicroW8 is a fascinating fantasy console, perfect for fans of sizecoding and the demoscene. In this part, you'll learn the basics of programming on this platform using CurlyWASM – a language that gives you control over every byte of the generated WebAssembly (WASM) code. We'll start by discussing the console, its limitations, and the memory map, then move through CurlyWASM syntax, and finally show you how to create a simple "Hello, World!".
### 1. What is MicroW8?

MicroW8 is a virtual console designed for creating small demos and games, inspired by classic 80s systems but running on modern WebAssembly. Its key features include:

* **Limitations:** WASM modules have a maximum size of 256 KB. Memory is limited to 256 KB (4 pages of 64 KB each). The screen resolution is 320x240 pixels with an 8-bit color palette. The lack of hardware vertical synchronization (V-sync) means that animation smoothness depends on the performance of your code in the `upd()` function, and you must handle timing manually. * **Main Entry Points:** Your code must export an `export fn upd()` function, called once per frame. An optional `export fn start()` function is called once after the module is loaded, ideal for initialization. * **Memory:** Available memory is imported as `"env.memory"` with a minimum number of pages. MicroW8 provides a memory map with predefined areas, such as the framebuffer (`FRAMEBUFFER`), color palette (`PALETTE`), font (`FONT`), and user memory area (`USER_MEM`).
### 2. CurlyWASM Language Basics

CurlyWASM is syntactic sugar for WebAssembly, designed to be close to WASM instructions but more convenient to write. The CurlyWASM compiler performs only basic optimizations (constant folding), so you are responsible for the code's performance and size.
#### 2.1. Code Structure and Comments

CurlyWASM code consists of imports, declarations of global variables and constants, data sections, and function definitions. Comments look like this:
```curlywasm
// This is a single-line comment

/*
  This
  is
  a block
  comment
*/
```

#### 2.2. Data Types

CurlyWASM (based on WASM MVP) supports four basic types:
* `i32`: 32-bit signed integer. **This is the preferred type for integer operations in MicroW8.** * `i64`: 64-bit signed integer. **Strongly avoid this type! WASM MVP focuses on 32-bit operations, and most (if not all) MicroW8 API functions use i32. Using i64 generates extra overhead and may be incompatible with the API.** * `f32`: 32-bit floating-point number. **This is the preferred type for floating-point operations.** * `f64`: 64-bit floating-point number. **Strongly avoid this type for the same reasons as i64. Most API functions use f32.**
#### 2.3. Literals

* **Integers:** Decimal or hexadecimal (e.g., `123`, `-7878`, `0xf00`). * **Floating-point numbers:** Decimal (e.g., `0.464`, `3.141`, `-10.0`). They can also be written with an `_f` suffix (e.g., `10_f`, `0x0a_f`) to explicitly specify they are `f32` literals. * **Character literals:** Single characters in single quotes (e.g., `'A'`). Can contain up to 4 characters, interpreted as a little-endian i32.
#### 2.4. Variables and Constants

* **Global variables:** Declared outside functions using `global`. Must be mutable (`mut`) and have an initial value.     ```curlywasm
    global mut myGlobal: i32 = 0;     ```
* **Global constants:** Declared outside functions using `const`. The value must be a constant expression.     ```curlywasm
    const SCREEN_SIZE = 320 * 240;
    const PI = 3.14159265;     ```
    **Important:** Global constants cannot be declared inside functions. You cannot use the `as` operator for type casting within the expression used for *declaring* a constant (`const`). However, the `as` operator is allowed and commonly used in other code contexts, outside of `const` declarations. * **Local variables:** Declared inside functions using `let`. They are always mutable, and the type is inferred during initialization.     ```curlywasm
    fn myFunction() {
      let counter: i32; // Declaration       let name = 'smok'; // Declaration and initialization (type inferred as i32, 4 bytes = 4 chars)       let distance = 100 as f32; // Using 'as' in an expression is correct     }
    ```
* **Local variable modifiers (`inline`, `lazy`):** Optimize code size by changing when the expression is evaluated.     ```curlywasm
    let inline row = calculateRow(y); // Calculated each time 'row' is used, like a macro     let lazy column = calculateColumn(x); // Calculated once, on the first use of 'column' (often uses local.tee)     ```

#### 2.5. Operators and Expressions

CurlyWASM uses infix notation for operators:
* **Arithmetic:** `+`, `-`, `*`, `/`, `%` (signed modulo), `#/`, `#%` (unsigned modulo). * **Bitwise:** `&` (AND), `|` (OR), `^` (XOR), `<<`, `>>` (signed shift), `#>>` (unsigned shift).     **Important:** Use `&` and `|` for conditional logic (`if`, `branch_if`). The operators `&&` and `||` do not exist in CurlyWASM. Using bitwise operators for logic is a common sizecoding technique. * **Comparison:** `==`, `!=`, `<`, `<=`, `>`, `>=` (signed), `#<`, `#<=`, `#>`, `#>=` (unsigned). * **Assignment with value usage:** `:=` assigns a value to a *local* variable and simultaneously returns that value. This is very useful in expressions and loops, e.g., `branch_if (i +:= 1): loop_label;`. It often compiles to the efficient WASM instruction `local.tee`. * **Sequencing operator:** `<|` executes the left-hand expression (for its side effects) and returns the value of the right-hand expression, e.g., `side_effect <| return_value`. Helps minimize stack overhead. * **Type casting:** `as` (e.g., `myInt as f32`). Allowed in most contexts except `const` declarations.
#### 2.6. Functions

Functions are declared with the `fn` keyword. The body is a block `{}`.
* **Implicit return:** The last expression in a function block is automatically returned. The `return` keyword is not used (it doesn't exist). Functions without a return value (or returning the empty type `()`) end with a semicolon-terminated statement or an empty block. * **Parameters:** A list of `name: type` in parentheses. * **Return type:** Optionally specified after `->`.     ```curlywasm
    fn add(a: i32, b: i32) -> i32 {
      a + b // Returned value (no semicolon)     }

    export fn drawSomething() {
      cls(0); // Statement ends with a semicolon, no return     }
    ```

#### 2.7. Control Flow

CurlyWASM supports WASM's structured control flow instructions:
* **If/Else:** `if condition { block } else { block }`. Can be an expression returning a value if both branches return the same type. * **Block:** `block name { ... }`. Defines a code block. A `branch name` or `branch_if condition: name` instruction jumps to the end of the block (after the closing brace `}`). * **Loop:** `loop name { ... }`. Defines a loop. A `branch name` or `branch_if condition: name` instruction jumps to the beginning of the loop (starting the next iteration). * **Important:** CurlyWASM **does not have** `break` or `continue` keywords. Exiting a loop from the middle to the point *after* the loop is achieved by using `branch` or `branch_if` targeting the *label of a block* (`block label`) that encloses the loop, or simply by "falling through" past the `branch_if` that conditions the loop's continuation. * **Labels:** Labels in `block label` and `loop label` serve as targets for `branch label` and `branch_if condition: label` instructions. A label on its own line, intended as a jump target *after* a `block` or `loop` structure, is rarely used and must be defined *without* a colon (e.g., `my_label`, not `my_label:`), but the WASM/CurlyWASM programming model naturally leads to using `branch` to jump to the end of a block or the start of a loop, rather than to external, arbitrary labels.
#### 2.8. Memory (Load/Store) and Direct Access

MicroW8 memory is linear, addresses are byte-based, starting from address 0. The total memory size is 256 KB.
* **Direct read:** CurlyWASM provides the syntax `base?offset` (read byte), `base!offset` (read 32-bit `i32` word), and `base$offset` (read 32-bit `f32` float). `base` is an `i32` expression (base address), and `offset` is an `i32` constant (offset from the base address). This syntax is preferred in loops processing large amounts of data (like the framebuffer) due to minimal overhead and generation of efficient WASM `load` instructions.     ```curlywasm
    // Read pixel color (byte) directly from the framebuffer
    let pixel_offset = y * 320 + x;     let color = (FRAMEBUFFER + pixel_offset)?0; // Read byte at address FRAMEBUFFER + pixel_offset + 0
    // Read time in ms (i32 word) from hardware register
    let time_ms = (TIME_MS)!0; // Read word at address TIME_MS + 0     ```
* **Direct write:** Similar to reading, we use the syntax `base?offset = value`, `base!offset = value`, `base$offset = value`.     ```curlywasm
    // Write pixel color (byte) directly to the framebuffer
    let pixel_offset = y * 320 + x;     (FRAMEBUFFER + pixel_offset)?0 = 15; // Write byte at address FRAMEBUFFER + pixel_offset + 0
    // Write i32 value (e.g., new TIME_MS value if we want to modify the counter)
    (TIME_MS)!0 = new_time_ms; // Write i32 word at address TIME_MS + 0     ```
* **Alignment:** Operations on 32-bit words (`!`, `$`, `i32.load`, `f32.load`, `i32.store`, `f32.store`) require the effective memory address (base + offset) to be aligned to 4 bytes (i.e., divisible by 4). Operations on bytes (`?`, `i32.load8_u/s`, `i32.store8`) do not have such requirements. Keep this in mind when designing data structures in memory. * **Intrinsics:** You can also directly use WASM instruction names for memory operations, e.g., `i32.load8_u(address, offset)`, `i32.store(value, address, offset)`. The `base?offset` etc. syntax is syntactic sugar for these instructions with an offset of 0.
#### 2.9. Data Sections (`data`)

Data sections are used to initialize memory areas at module startup. Data defined in them is loaded to a specific address in the console's RAM.
```curlywasm
data 0x14000 { // Address in memory, e.g., start of USER_MEM   i8(10, 20, 30) // Block of bytes. Values separated by commas.   i32(0x12345678) // Block of 32-bit words.   f32(3.14) // Block of 32-bit floats.   "Text\0" // Null-terminated string.   file("my_data.bin") // Content of an external binary file. }
// Important: Place data blocks of different types (i8(...), i32(...), "...", file(...)) one after another within the data { ... } section, WITHOUT COMMAS between the blocks. ```

### 3. MicroW8 API

MicroW8 provides a set of API functions in the `"env"` module, which must be imported in the `import` section. CurlyWASM also provides built-in math and bitwise functions that are available directly and **do not require import**. Attempting to import them will cause a compilation error.
#### 3.1. Graphics

Screen: 320x240, 8 bpp (8 bits per pixel, index into the palette). Functions taking `f32` parameters (e.g., `line`, `circle`, `rectangle`) have sub-pixel accuracy.
* `cls(color: i32)`: Clears the entire screen to the specified color index, resets the text cursor to (0,0), and disables graphics mode for text. * `setPixel(x: i32, y: i32, color: i32)`: Sets the color of the pixel at coordinates `(x, y)` to `color`. For optimization in loops processing many pixels, direct access to the framebuffer memory is preferred (see section 2.8). * `getPixel(x: i32, y: i32) -> i32`: Reads the color index of the pixel at coordinates `(x, y)`. Returns 0 if the coordinates are off-screen. * `hline(left: i32, right: i32, y: i32, color: i32)`: Draws a horizontal line from `left` to `right` (exclusive) at height `y` with `color`. * `line(x1: f32, y1: f32, x2: f32, y2: f32, color: i32)`: Draws a line between points `(x1, y1)` and `(x2, y2)` with `color`. * `rectangle(x: f32, y: f32, w: f32, h: f32, color: i32)`: Fills a rectangle with the top-left corner at `(x, y)`, width `w`, and height `h` with `color`. * `circle(cx: f32, cy: f32, radius: f32, color: i32)`: Fills a circle with center `(cx, cy)` and radius `radius` with `color`. * `rectangleOutline(x: f32, y: f32, w: f32, h: f32, color: i32)`: Draws the outline of a rectangle. * `circleOutline(cx: f32, cy: f32, radius: f32, color: i32)`: Draws the outline of a circle. * `blitSprite(spriteData: i32, size: i32, x: i32, y: i32, control: i32)`: Copies pixel data from the specified memory address (`spriteData`) to the screen at position `(x, y)`. The `size` parameter encodes width and height (`width | (height << 16)`). `control` allows for color masking and flipping (horizontal/vertical). * `grabSprite(spriteData: i32, size: i32, x: i32, y: i32, control: i32)`: Copies pixel data from the screen at the specified position `(x, y)` to memory at address `spriteData`. Parameters are the same as in `blitSprite`.
#### 3.2. Gamepad

Gamepad: Virtual gamepad with D-Pad and 4 buttons (A, B, X, Y). Button states are mapped to keyboard keys (e.g., Up -> Arrow-Up, A -> Z). Button states can be checked using API functions or by directly reading the bitfield in memory at address `GAMEPAD` (0x44).
* `isButtonPressed(btn: i32) -> i32`: Returns a non-zero value if the button with index `btn` is currently pressed. * `isButtonTriggered(btn: i32) -> i32`: Returns a non-zero value if the button with index `btn` was pressed in this frame (event). * **Example of direct reading of button states from memory:**
    The gamepad state (`GAMEPAD`, 0x44) is a 32-bit word (i32), where each bit corresponds to one button. It can be read using the `!` operator:     ```curlywasm
    let gamepad_state = (GAMEPAD)!0;     let is_button_a_pressed = (gamepad_state >> BUTTON_A) & 1; // Check bit for button A     let is_button_up_pressed = (gamepad_state >> BUTTON_UP) & 1; // Check bit for button Up     ```
    Button indices are defined as constants (BUTTON_UP, BUTTON_DOWN, etc.). Direct reading is often faster and generates smaller code than multiple calls to `isButtonPressed` if checking several buttons.
#### 3.3. Screen and Text

Text printing uses the built-in font (address `FONT`, 0x13400), which can also be replaced with a custom one. Two modes are available: normal (8x8 grid, default after `cls()`) and graphics mode (cursor position in pixels, transparent background).
* `printChar(char: i32)`: Prints a single character (lower 8 bits of `char`). Also handles control characters (0-31). * `printString(ptr: i32)`: Prints a null-terminated (`\0`) string pointed to by address `ptr`. * `printInt(num: i32)`: Prints the integer `num` as decimal text. * `setTextColor(color: i32)`: Sets the color index for printed text. * `setBackgroundColor(color: i32)`: Sets the background color index for printed text (only in normal text mode). * `setCursorPosition(x: i32, y: i32)`: Sets the cursor position. In normal mode, `(x, y)` are character grid coordinates (0-39, 0-29); in graphics mode, they are pixel coordinates (0-319, 0-239).
#### 3.4. Sound

MicroW8 can generate sound in two ways: through its own `export fn snd(sampleIndex: i32) -> f32` function (more advanced, low-level) or through the built-in `sndGes` synthesizer, controlled by 32 bytes of memory starting at address 0x50.
* `playNote(channel: i32, note: i32)`: A convenient API function to trigger notes (1-127, 69=A4) on one of 4 channels (0-3). Note 0 stops sound on the channel, 128-255 triggers attack and release without sustain. This function modifies the corresponding bytes in the `sndGes` registers (0x50-0x70). * `sndGes(sampleIndex: i32) -> f32`: Function implementing a 4-channel synthesizer controlled by registers in memory 0x50-0x70. If you don't define your own `snd()` function, MicroW8 will call `sndGes` twice per sample (left and right channel) at 44100 Hz, copying 32 bytes from address 0x50 from the main thread's memory to the sound memory after each `upd()` call. * **Direct control of the `sndGes` synthesizer:**
    Full control over sound is achieved by directly modifying the 32 bytes starting at address 0x50. For example, to set the wave type (square=0, saw=1, triangle=2, noise=3) and note on channel 0, you can write the bytes:     ```curlywasm
    // Set wave type (bits 7-6) and note on flag (bit 0) on channel 0
    // Address 0x50 + channel offset (0*6) + CTRL register offset (0) = 0x50
    let wave_type = 2; // Triangle     let note_on_flag = 1;     (0x50)?0 = (wave_type << 6) | note_on_flag;
    // Set note (69=A4) on channel 0
    // Address 0x50 + channel offset (0*6) + NOTE register offset (3) = 0x53
    (0x53)?0 = 69;     ```
    A detailed description of the `sndGes` registers can be found in the `microw8-api.txt` documentation.
#### 3.5. Math Functions

Operations on `f32`, angles in radians.
* **Imported from `"env"`:** (Require import in the `import` section)
    * `sin`, `cos`, `tan`, `asin`, `acos`, `atan`, `atan2`, `exp`, `log`, `pow`, `fmod`. * **Built-in in CurlyWASM:** (Available directly, **do not import them!**)
    * `sqrt`, `min`, `max`, `ceil`, `floor`, `trunc`, `nearest`, `abs`, `copysign`, `select`.
#### 3.6. Randomness

MicroW8 provides a pseudorandom number generator based on the xorshift64\* algorithm. It is initialized with a constant seed at startup unless you use `randomSeed`.
* `random() -> i32`: Returns a 32-bit pseudorandom integer. * `randomf() -> f32`: Returns a pseudorandom floating-point number in the range $[0.0, 1.0)$. * `randomSeed(seed: i32)`: Initializes the pseudorandom number generator with the given seed. In `platform.txt`, you can see the implementation of `randomSeed` and `random`, showing how xorshift64\* works.
#### 3.7. Time

* `time() -> f32`: Returns the time in seconds since the module started. This is the time in the main thread. * `TIME_MS` (0x40): Address in memory containing a 32-bit (`i32`) value of time in milliseconds since the module started. This register is incremented by the MicroW8 runtime in each `upd()` call. It is an R/W register, can be read (`(TIME_MS)!0`) and written, which is crucial for manual delta time and FPS measurement.
#### 3.8. Memory Operations (Intrinsics)

CurlyWASM allows the use of low-level WebAssembly instructions for memory operations. They are often generated by the `?`, `!`, `$` operators, but can also be used directly.
* `i32.load(base, offset)`: Reads a 32-bit `i32` word from memory at address `base + offset`. Requires 4-byte alignment. * `i32.load8_u(base, offset)`, `i32.load8_s(base, offset)`: Read an 8-bit byte as an unsigned/signed `i32`. No alignment required. * `i32.load16_u(base, offset)`, `i32.load16_s(base, offset)`: Read a 16-bit word as an unsigned/signed `i32`. Requires 2-byte alignment. * `i32.store(value, base, offset)`: Writes a 32-bit `value` word to memory at address `base + offset`. Requires 4-byte alignment. * `i32.store8(value, base, offset)`: Writes the lower 8 bits of `value` as a byte. No alignment required. * `i32.store16(value, base, offset)`: Writes the lower 16 bits of `value` as a 16-bit word. Requires 2-byte alignment. * `f32.load(base, offset)`: Reads a 32-bit `f32` float. Requires 4-byte alignment. * `f32.store(value, base, offset)`: Writes a 32-bit `value` float. Requires 4-byte alignment. * `memory.size() -> i32`: Returns the current memory size in 'page' units (1 page = 64KB). * `memory.grow(delta_pages) -> i32`: Increases memory size by `delta_pages`. Returns the previous memory size in pages, or -1 on error. MicroW8 has a fixed memory size (4 pages = 256KB), so this function might not be useful in practice. * `memory.fill(dest, value, size)`: Fills a memory area starting at `dest` of `size` bytes with the byte `value`. Very fast for clearing buffers. * `memory.copy(dest, src, size)`: Copies a memory block of `size` bytes from address `src` to address `dest`. Indispensable for double buffering.
#### 3.9. Other Built-in Functions

CurlyWASM also provides other built-in functions (intrinsics) for integer operations, **which we do not import**:

* `i32.clz(value: i32) -> i32`: Count Leading Zeros. Number of binary zeros from the most significant bit. * `i32.ctz(value: i32) -> i32`: Count Trailing Zeros. Number of binary zeros from the least significant bit. * `i32.popcnt(value: i32) -> i32`: Population Count. Number of set bits (ones). * `i32.bswap(value: i32) -> i32`: Byte Swap. Reverses the byte order in a 32-bit word (useful when working with different endianness). * `i32.trunc_sat_f32_u(value: f32) -> i32`, `i32.trunc_sat_f32_s(value: f32) -> i32`: Convert `f32` to `i32` (unsigned/signed) with truncation towards zero, saturating results to the `i32` range.
### 4. Memory Map

Knowing the memory map is crucial in sizecoding, as it allows direct access to important registers and data areas.
| Address       | Size          | Description                                      | CurlyWASM Constant |
| :------------ | :------------ | :----------------------------------------------- | :----------------- |
| `00000-00040` | 64 bytes      | User memory (small area)                         | -                  | | `00040-00044` | 4 bytes       | Time since start in ms (i32)                     | `TIME_MS`          | | `00044-0004c` | 8 bytes       | Gamepad state (`GAMEPAD`, `GAMEPAD + 4` for prev frame state) | `GAMEPAD`          | | `0004c-00050` | 4 bytes       | Number of frames since start (i32)               | -                  | | `00050-00070` | 32 bytes      | sndGes registers                                 | -                  | | `00070-00078` | 8 bytes       | Reserved                                         | -                  | | `00078-12c78` | 76800 bytes   | Framebuffer (320x240, 8bpp)                      | `FRAMEBUFFER`      | | `12c78-12c7c` | 4 bytes       | sndGes work area base address (default 0x50)     | -                  | | `12c7c-13000` | 836 bytes     | Reserved                                         | -                  | | `13000-13400` | 1024 bytes    | Color palette (256x32bpp, RGBA)                  | `PALETTE`          | | `13400-13c00` | 2048 bytes    | Font (256x8x8, 1bpp)                             | `FONT`             | | `13c00-14000` | 1024 bytes    | Reserved                                         | -                  | | `14000-40000` | 172032 bytes  | User memory (large area)                         | `USER_MEM`         |
Memory from 0x14000 to 0x40000 is the main area available for your data: global variables, arrays, graphics buffers (sprites), sound data, extra buffers for double buffering, etc. The effective size of this memory is $0x40000 - 0x14000 = 262144 - 81920 = 180224$ bytes.
### 5. Workflow

The `uw8` tool is your main companion when working with MicroW8. It supports:

* `uw8 compile <input.cwa> <output.wasm>`: Compiles CurlyWASM code to a binary WebAssembly module. * `uw8 run <file.wasm|.uw8|.cwa>`: Runs a WASM module, `.uw8` cart, or directly a CurlyWASM file. The `-w`, `--watch` option is very useful for automatically reloading the cartridge when the file changes. * `uw8 pack <infile> <outfile>`: Packs a WASM module or source file into the `.uw8` format. * `uw8 unpack <infile> <outfile>`: Unpacks an `.uw8` cart into a WASM module. * `uw8 filter-exports <infile> <outfile>`: A size optimization tool that removes unused exports and dead code from a WASM module.
### 6. Sizecoding and Basic Optimization

Sizecoding on MicroW8 involves minimizing the size of the generated WASM module. CurlyWASM supports this through:

* **Low-level WASM instructions:** CurlyWASM compiles close to WASM instructions, giving the programmer significant control. * **`let inline`/`lazy` modifiers:** Allow control over when and how local variable values are calculated, affecting code size and performance.     * `let lazy name = expression;`: Expression calculated only once, on first use. Can save code size if the value is used multiple times, but only in one execution path.     * `let inline name = expression;`: Expression is inserted at the point of use, like a macro. Can increase code size but eliminates overhead associated with local variables. Useful for dynamic values that need recalculation in each iteration. * **`:=` operator:** As discussed in section 2.5, crucial in loops and conditional expressions for combining assignment and value usage in one efficient instruction. Example: `branch_if (i +:= 1) < MAX_ITER: loop_label;`. * **Direct memory access (`base?offset`, `base!offset`, `base$offset`):** Significantly faster and smaller than API function calls (e.g., `setPixel`, `getPixel`) in loops operating on pixels. Minimizes function call overhead and allows direct data manipulation in memory. For instance, instead of `setPixel(x, y, color);` in a rendering loop, use `(FRAMEBUFFER + y * 320 + x)?0 = color;`. * **Bitwise operations:** Efficient use of bitwise operators (`&`, `|`, `^`, `<<`, `>>`, `#>>`) for data packing, fast math (e.g., $(x << 3)$ is $x \times 8$), and pattern generation can significantly reduce code size. * **Look-up Tables (LUTs):** Pre-calculating values of mathematical functions (e.g., sine, cosine) and storing them in memory (`data` section or initialization in `start()`) avoids costly real-time calculations in the `upd()` loop. Reading from memory is generally faster and generates less code than calling functions like `sin()`.     **Example LUT initialization in `start()`:**     ```curlywasm
    const SIN_TABLE_SIZE = 256;
    const SIN_TABLE = USER_MEM; // Store the table at the beginning of USER_MEM
    export fn start() {
      let i = 0;       let pi = 3.14159265;       loop fill_loop {
        let angle_rad = (i as f32 / SIN_TABLE_SIZE as f32) * pi * 2.0; // Angle in radians (0 to 2*PI)         let sin_val_f = sin(angle_rad);         let sin_val_i8 = (sin_val_f * 127.0 + 128.0) as i32; // Scale to range 0-255         (SIN_TABLE + i)?0 = sin_val_i8; // Store as byte in the table         branch_if (i := i + 1) < SIN_TABLE_SIZE: fill_loop;       }
      // ... rest of initialization ...     }

    // Usage of LUT in upd() function:
    /*
    fn upd() {
      let angle_index = ( (time() * some_speed_f) as i32 ) & (SIN_TABLE_SIZE - 1); // Calculate index (with wrapping)       let sin_value_i8 = (SIN_TABLE + angle_index)?0; // Read value from table (0-255)       let sin_value_f = (sin_value_i8 as f32 - 128.0) / 127.0; // Scale back to -1.0 .. 1.0       // Use sin_value_f for further calculations...     }
    */
    ```

### 7. FPS Measurement

MicroW8 lacks V-sync and a built-in FPS counter that you can read directly. To measure the current frames per second (FPS):
1.  Read the current value of `TIME_MS` (address `0x40`, time in ms) at any point in the `upd()` function and store it. Address `0x3c` (just before `TIME_MS` in the small user memory area) is often used for this purpose in sizecoding to avoid declaring a global variable. 2.  Read `TIME_MS` again during the *next* call to `upd()`. 3.  Calculate the difference (`delta`) in milliseconds between the newly read `TIME_MS` value and the one stored in the previous frame. 4.  Approximate FPS is $1000 / \text{delta}$.
**Note:** For an empty `upd()` function, MicroW8 typically achieves ~60 FPS (delta ≈ 16.67 ms), assuming no significant delays in the browser runtime or operating system. See a practical implementation example in Section 2.4 (the `printFps()` function), which displays FPS and warns when it drops below 60.
### 8. "Hello, World!" Example

This simple example demonstrates the basic structure of a CurlyWASM module, importing API functions, and a data section.
```curlywasm
// Import API functions needed for drawing and text
import "env.memory" memory(4); // Import memory import "env.cls" fn cls(i32); // Import cls function for clearing the screen import "env.printString" fn printString(i32); // Import function for printing strings import "env.setTextColor" fn setTextColor(i32); // Import function for setting text color import "env.setCursorPosition" fn setCursorPosition(i32, i32); // Import function for setting the cursor
// Define a constant for the user memory address where we'll place the text
const USER_MEM = 0x14000; // Address of the start of the larger user memory area
// Data section - place the string "Witaj, swiecie!" and a null terminator
data USER_MEM {   "Witaj, swiecie!" // String to display   i8(0) // Null byte - string terminator }

// Update function, called every frame
export fn upd() {
  cls(0); // Clear the screen with color 0 (black)   setTextColor(15); // Set text color to 15 (default white in standard palette)   setCursorPosition(5, 5); // Set text cursor position (in 8x8 character grid) to column 5, row 5   printString(USER_MEM); // Print the string from the USER_MEM address }

// Optional start function, called once when the module loads
export fn start() {
  // Initialization code can go here, e.g.,   // randomSeed(), initializing LUTs, etc.   // Not needed in this simple example. }
```

This code clears the screen, sets the text color and cursor position, and then displays the string "Witaj, świecie!", reading it from the data section in memory.
### 9. Examples from Files

Analyzing the code of some examples from the `src` directory is an invaluable source of knowledge about practical techniques in CurlyWASM and MicroW8. I encourage you to study their code:
* `control.cwa`: Demonstrates control characters for text formatting. * `scaled_text.cwa`: Uses graphics mode and control character 30 to draw scalable text. * `font_palette.cwa`: Manipulates the font and color palette, switching between them. * `Chaos_Rose.cwa`, `encounter.cwa`, `urban_drift.cwa`, `plasma_chars.cwa`, `plasma_sizecoding_1.cwa`, `sierpinski_triangle.cwa`, `mandelbrot.cwa`, `interference.cwa`: Examples of classic demoscene effects (fractals, plasma, raymarching, procedural effects) and their implementation techniques, often using sizecoding optimizations. * `cube_wireframe.cwa`: Renders a simple 3D model (cube) using projection and line drawing. A good example of using math and manipulating global variables. * `game_of_life.cwa`: Implements Conway's Game of Life, demonstrating the use of double buffering and efficient memory operations with wrap-around.
---

## Part 2: Advanced MicroW8 Techniques

Now that you know the basics, it's time to dive into more advanced demoscene techniques on MicroW8. In this part, we'll focus on code optimization, advanced memory management (including double buffering), implementing classic demoscene effects, precise timing based on delta time, and generating sound using `sndGes`. This is a challenge for true sizecoders!
### 1. CurlyWASM and Sizecoding: The Art of Compact Code

In MicroW8, every byte matters. CurlyWASM provides tools for precise control over code size through compilation close to WASM instructions.
* **`let` Modifiers (`lazy`, `inline`):** As discussed in Part 1, these allow optimizing the use of local variables and the generated code depending on whether a value is used multiple times (`lazy`) or as a one-time expression replacement (`inline`).     * `let lazy name = expression;`: Uses `local.tee` underneath, evaluates the expression once, assigns it to a local variable, and returns the value. Saves bytes if the expression is complex and the value is used multiple times, but be careful with unused code paths that might unnecessarily calculate the value.     * `let inline name = expression;`: Acts like a macro, inserting the expression at the point where the variable is used. Useful for dynamic values, but can increase code size if the variable is used multiple times. * **`:=` Operator:** Indispensable in sizecoding. Allows calculating a value, assigning it to a *local variable*, and simultaneously using that value in an expression. Most common use is in loops with `branch_if`, e.g., `branch_if (i +:= 1) < MAX_ITER: loop_label;`. The CurlyWASM compiler often generates the highly efficient `local.tee` instruction for this operator. * **Sequencing Operator `<|`:** Executes the side effect (left side) and returns the value (right side). Minimizes stack overhead. * **Direct Memory Access (`base?offset`, `base!offset`, `base$offset`):** Key in sizecoding because it bypasses the overhead of API function calls in loops processing large amounts of data (e.g., the framebuffer). It's generally faster and generates less code than calling functions like `setPixel` or `getPixel`. The `plasma_sizecoding_1.cwa` example shows how to minimize local variables and work with direct memory access in a pixel-drawing loop, reducing code size.
### 2. Advanced Memory Management and Double Buffering

Efficient use of the available 256 KB of memory is fundamental to advanced programming on MicroW8.
* **Framebuffer (`FRAMEBUFFER`, 0x78):** 320x240 pixels, 8 bpp (76800 bytes). The address of pixel (x, y) is calculated as `FRAMEBUFFER + y * 320 + x`. Use the `?` operator to write/read single bytes (color indices).     ```curlywasm
    let pixel_x = 10;     let pixel_y = 20;     let color_index = 12;
    let offset = pixel_y * 320 + pixel_x;     (FRAMEBUFFER + offset)?0 = color_index; // Write color byte     let read_color = (FRAMEBUFFER + offset)?0; // Read color byte     ```
* **Palette (`PALETTE`, 0x13000):** 256 colors, each 4 bytes (RGBA), totaling 1024 bytes. Remember that writing 32-bit RGBA values requires 4-byte alignment. Use the `!` operator for reading/writing 32-bit words or the `i32.load`/`i32.store` functions. Example: `font_palette.cwa` demonstrates palette manipulation.     ```curlywasm
    // Write color (RGBA) to index 10 in the palette
    let color_rgba = 0xff0000ff; // Blue (AA RR GG BB in little-endian, i.e., BB GG RR AA in memory)     let palette_offset = 10 * 4; // 4 bytes per color     (PALETTE + palette_offset)!0 = color_rgba; // Write 32-bit word     ```
* **User Memory (`USER_MEM`, 0x14000-0x40000):** A large area (approx. 180 KB) available for your data. You can use it for arrays, data structures, sprites, fonts, sound data, and also as a **back buffer** for double buffering. * **Double Buffering:** A technique involving drawing the next frame to a buffer in RAM (e.g., in the `USER_MEM` area) instead of directly to the `FRAMEBUFFER`. After drawing is complete, the contents of this buffer are copied to the `FRAMEBUFFER`. This prevents tearing and flickering, ensuring smooth transitions between frames. The `game_of_life.cwa` example uses two buffers (FRAMEBUFFER and BACKBUFFER in USER_MEM) and copies between them. Copying the entire buffer (76800 bytes) can be done very efficiently in MicroW8 using the `memory.copy` instruction.     **Example Double Buffering Implementation:**     1.  Reserve an area in `USER_MEM` for the back buffer, e.g., `const BACKBUFFER = USER_MEM;`. Ensure it's large enough (320x240 bytes).     2.  In the `upd()` function:         a.  Draw all scene elements to `BACKBUFFER`. Instead of `setPixel(x, y, c)`, use e.g., `set(BACKBUFFER, x, y, c)` (a helper function using `(BACKBUFFER + y * 320 + x)?0 = c;`) or direct writing.         b.  After drawing is complete, copy `BACKBUFFER` to `FRAMEBUFFER`:         ```curlywasm
        // Copy the entire content of BACKBUFFER (320*240 bytes) to FRAMEBUFFER
        memory.copy(FRAMEBUFFER, BACKBUFFER, 320 * 240);         ```
        This single `memory.copy` operation is significantly faster than drawing individual pixels to the framebuffer.
### 3. Implementing Demoscene Effects

The examples provided with MicroW8 demonstrate the implementation of many classic demoscene effects worth studying and adapting:
* **Plasma:** Generating mesmerizing, organic patterns using combinations of sine functions with different frequencies and phases. `plasma_sizecoding_1.cwa` shows a pixel version, and `plasma_chars.cwa` shows a text version using a custom font and a sine LUT. * **Fractals:** Drawing sets generated by iterative mathematical formulas, e.g., the Mandelbrot set ($z_{n+1} = z_n^2 + c$). `mandelbrot.cwa` demonstrates calculating the Mandelbrot set and coloring pixels based on the number of iterations needed to "escape". * **Raymarching / Signed Distance Fields (SDF):** 3D rendering techniques involving "marching" along a ray from the camera and using a distance function (SDF) to determine how far we are from an object's surface. `urban_drift.cwa` is an advanced example of raymarching rendering a dynamic "city" of procedurally generated buildings. * **3D Graphics (Wireframe):** Basics of 3D rendering: defining vertices and edges of an object, transformations (translation, rotation, scaling) in 3D, projection from 3D to 2D, and drawing edges using the `line` function. `cube_wireframe.cwa` is an example rendering a rotating cube. * **Cellular Automata:** Systems where the state of each cell on a grid changes depending on the state of its neighbors according to simple rules. `game_of_life.cwa` implements Conway's classic Game of Life, utilizing double buffering for smooth animation. * **Interference Patterns:** Generating patterns by summing or multiplying waves (often trigonometric functions) from several sources, animated over time. `interference.cwa` creates interference patterns based on the distance from two moving points. * **Noise Algorithms:** Algorithms generating procedural "noise" (e.g., Perlin noise, Simplex noise) can be used to create realistic textures, surface deformations, or other visual effects. Although the MicroW8 API only has simple `random()` and `randomf()` generators, more advanced noise algorithms can be implemented in CurlyWASM (e.g., 1D/2D versions of value noise based on `randomSeed(index)`).
### 4. Precise Timing and Delta Time

As you learned in Part 1, the lack of V-sync on MicroW8 means you must manually manage animation timing. Calculating delta time (the time between consecutive frames) is crucial for ensuring that movement and animations have a constant speed regardless of the instantaneous frame rate.
* **Measuring delta time:** Use the `TIME_MS` register (0x40). Read its value at the beginning of the `upd()` function, calculate the difference from the previous frame's value (stored in a global variable or in memory, e.g., at address `0x3c`), and then update the stored value. * **Time-dependent motion:** Use `speed * delta_time` instead of constant increments to make animations and movement smooth regardless of FPS.
**Example of calculating and displaying FPS (function `printFps`):**
The following code shows how to calculate and display FPS, using user memory at address `0x3c` (just before `TIME_MS`) to store the previous frame's time. This is a common sizecoding technique.
```curlywasm
// Required imports (ensure they are at the beginning of the file)
import "env.setBackgroundColor" fn setBackgroundColor(i32); import "env.setTextColor" fn setTextColor(i32); import "env.setCursorPosition" fn setCursorPosition(i32, i32);
import "env.printInt" fn printInt(i32);
import "env.printChar" fn printChar(i32);
const TIME_MS = 0x40; const LAST_TIME_MS_ADDR = 0x3c; // Address in user memory to store previous time
// Example color indices (adjust to your palette)
const COLOR_DARK_GREY = 1; const COLOR_WHITE = 15;
// Function to calculate and display FPS in the top-left corner
fn printFps() {
    let current_time_ms = (TIME_MS)!0; // Read current time     let last_time_ms = (LAST_TIME_MS_ADDR)!0; // Read previous frame's time from 0x3c     (LAST_TIME_MS_ADDR)!0 = current_time_ms; // Store current time as last time for the next call
    let delta_ms = current_time_ms - last_time_ms;     let fps: i32;
    if delta_ms > 0 {         // Use f32 for division to get more precision before converting to i32
        fps = (1000.0 / (delta_ms as f32)) as i32;     } else {
        fps = 999; // Handle very fast frames or the first call     }

    // Display FPS
    setBackgroundColor(COLOR_DARK_GREY); // Set background     setTextColor(COLOR_WHITE); // Set text color     setCursorPosition(0, 0); // Set cursor to top-left     printInt(fps); // Print FPS value
    // Optionally: display a warning if FPS drops below 60
    if fps < 60 {         printChar(' '); // Space for readability         printChar(0xe3); // Example warning character (e.g., down arrow - depends on font)         printInt(60 - fps); // Show how much FPS dropped below 60     }
    // Fill the rest of the line with background to overwrite old characters (optional)
    printString("    "); // Clear a few characters to the right }

// Start function - initializes the stored time
export fn start() {
    (LAST_TIME_MS_ADDR)!0 = (TIME_MS)!0; // Store initial time     // ... rest of initialization ... }

// Update function - calls printFps() every frame
export fn upd() {
    // ... rest of update code ...

    printFps(); // Display FPS }
```

**Notes on `printFps()`:**

* The function calculates FPS based on the time difference between consecutive `upd()` calls. * It uses address `0x3c` to store the previous frame's time – ensure you don't use this address for other purposes. * For an empty `upd()` loop, it should display ~60 FPS (assuming no runtime delays). * If FPS drops below 60, it displays the character `0xe3` (its appearance depends on the active font) and the difference. * Constants `COLOR_DARK_GREY` and `COLOR_WHITE` are example palette indices – adjust them. * The `start()` function is needed to initialize the value at `LAST_TIME_MS_ADDR`.
**Example of using delta time for animation:**
```curlywasm
// Global variables for timing (alternative to storing at 0x3c)
// global mut last_frame_time_ms: i32 = 0; // global mut delta_time_seconds: f32 = 0.0;
// Global variables for animation (e.g., rotation angle)
global mut rotation_angle: f32 = 0.0; const ROTATION_SPEED_DPS: f32 = 90.0; // Rotation speed in degrees per second const DEG_TO_RAD: f32 = 3.14159265 / 180.0; // Degrees to radians conversion
export fn start() {
  // Initialize previous frame time at start (if using a global variable)
  // last_frame_time_ms = (TIME_MS)!0;   (0x3c)!0 = (TIME_MS)!0; // Initialization if using address 0x3c }

export fn upd() {
  let current_time_ms = (TIME_MS)!0; // Read current time in ms   let last_time_ms = (0x3c)!0; // Read previous time   (0x3c)!0 = current_time_ms; // Store current time
  // Calculate delta time in milliseconds, then convert to seconds (as f32)
  let delta_ms = current_time_ms - last_time_ms;   let delta_time_seconds = (delta_ms as f32) / 1000.0;   // Prevent large jumps if a frame took very long (e.g., after a pause)
  if delta_time_seconds > 0.1 { delta_time_seconds = 0.0166; }
  // *** Example usage of delta time for animation ***
  // Calculate angle increment for this frame (speed in degrees/sec * delta_time_seconds)
  let angle_increment_deg = ROTATION_SPEED_DPS * delta_time_seconds;   // Convert increment to radians
  let angle_increment_rad = angle_increment_deg * DEG_TO_RAD;   // Update rotation angle
  rotation_angle = rotation_angle + angle_increment_rad;   // Clamp angle to range 0 - 2*PI to avoid overflow (optional, but good practice)
  let two_pi = 3.14159265 * 2.0;   rotation_angle = fmod(rotation_angle, two_pi);   if rotation_angle < 0.0 {
      rotation_angle = rotation_angle + two_pi;   }

  // Now you can use rotation_angle (or e.g., sin(rotation_angle), cos(rotation_angle))
  // to rotate objects in the scene, e.g., in the cube_wireframe.cwa example.   // The movement will have a constant speed regardless of FPS.   // ... rest of frame drawing code ...   printFps(); // Display FPS at the end }
```

### 5. Sound using `sndGes`

The `sndGes` synthesizer is MicroW8's built-in sound chip. It is controlled by 32 bytes of memory from address 0x50 to 0x70. These 32 bytes are copied from the main thread to the sound thread after each `upd()` call, making changes to the registers audible.
* **Register Structure (0x50 - 0x70):** The 32 bytes are divided into 4 blocks of 6 bytes for each of the 4 channels (starting at 0x50, 0x56, 0x5C, 0x62), 2 bytes for volume (0x68, 0x69), and 6 bytes for filters (0x6A - 0x6F). * **Channels (0-3):** Each channel has registers to control:     * CTRL (offset 0): Wave type, note on flag, attack trigger, filter selection, stereo panning (wide/narrow), ring modulation.     * PULS (offset 1): Pulse width for various wave types.     * FINE (offset 2): Fractional part of the note.     * NOTE (offset 3): Note number (69=A4).     * ENVA (offset 4): Envelope A parameters (Attack, Decay).     * ENVB (offset 5): Envelope B parameters (Sustain, Release). * **Volume (0x68, 0x69):** 4-bit fields for pairs of channels. * **Filters (0x6A - 0x6F):** Programmable filters. * **`playNote(channel, note)` function:** As mentioned, this is a convenient API function that modifies the appropriate bytes in the `sndGes` registers to trigger a note. It's a good starting point. * **Direct Register Manipulation:** For full control, you can write directly to the memory bytes starting from 0x50. For example, to set the wave and note on channel 1 (registers start at 0x50 + 1\*6 = 0x56):     ```curlywasm
    let channel1_base = 0x56;     let wave_type = 1; // Saw     let note_number = 72; // C5
    // Set wave (Saw = 1 << 6) and note on flag (bit 0)
    (channel1_base)?0 = (wave_type << 6) | 1;
    // Set note
    (channel1_base + 3)?0 = note_number;
    // To trigger the attack, you need to change the trigger bit (bit 1) in the CTRL register (offset 0)     // You can do this, e.g., by changing the bit value each frame when the note should be triggered.     // (channel1_base)?0 = (channel1_base)?0 ^ 2; // XOR the trigger bit
    // To stop the note, set the note on flag to 0:
    // (channel1_base)?0 = (channel1_base)?0 & ~1; // Clear bit 0     ```
    Studying the `playNote` code in `platform.txt` can help understand how this function modifies the registers.
### 6. Optimization and Tools

* **`uw8 filter-exports`:** An essential tool after compilation, especially from high-level languages (C, Rust, Zig), to remove unnecessary exports and dead code, reducing the WASM module size. It's worth using even after compiling CurlyWASM to ensure the compiler didn't leave any junk. * **`wasm2wat` (from WebAssembly Binary Toolkit - Wabt):** Allows decompiling a binary WASM module into the human-readable WAT text format. Analyzing WAT code is crucial in sizecoding to understand how CurlyWASM compiles your code and identify potential areas for further optimization at the WASM instruction level. * **`wasm-opt` (from Binaryen):** A powerful optimization tool for WASM modules. It can perform advanced optimizations that the (intentionally simple) CurlyWASM compiler doesn't. It's worthwhile to incorporate `wasm-opt` into your workflow after compilation to get smaller and faster code.
**Coding for Compo Limits**
In the demoscene, demos are often created for competitions (compos) with strict size limits, e.g., 4 KB or 64 KB. MicroW8, with its 256 KB limit, is ideal for such challenges. Here are tips for fitting within the limits:
* **Minimize API imports:** Import only necessary functions (`env.printInt`, `env.setPixel`) to reduce WASM module size. Each imported symbol adds bytes to the WASM import section. * **Compress data:** Pack textures or LUTs in `USER_MEM` using bitwise operations, e.g., store 2 4-bit pixels in one byte, or use simple RLE (Run-Length Encoding) schemes for repetitive data. * **Optimize code:** Use `uw8 pack` to compress the WASM module. Use `-l 9` for maximum compression (though it's slower). Test with `wasm-opt --enable-bulk-memory -Oz` (optimize for size) for additional savings. * **Test size:** Regularly check the `.uw8` file size after packing (`uw8 pack`). Analyze the decompiled code (`wasm2wat`) to see which parts generate the most bytes. * **Example:** In `plasma_sizecoding_1.cwa`, the effect is generated procedurally in `upd()` without storing a texture, saving bytes on data.
### 7. Debugging

Debugging in a low-level environment with limited tools requires creativity:
* **Debug output:** Use control characters to redirect text output to the debug console (control character 6) or the screen (control character 4). `printInt(value)` and `printChar('\n')` (control character 10) are invaluable for displaying variable values and tracing program flow.     ```curlywasm
    // Example logging a value to the debug console
    printChar('\6'); // Switch output to console     printString("X="); // Print label     printInt(my_x); // Print the value of variable my_x     printChar('\n'); // Print newline     printChar('\4'); // Switch output back to screen     ```
* **`uw8 --debug`:** Using this option during compilation (`uw8 compile --debug <input.cwa> <output.wasm>`) generates a name section in the WASM module. Debugging tools (e.g., the built-in debugger in Chrome/Firefox) can then display the original function and variable names from CurlyWASM instead of anonymous indices, significantly easing step-by-step code tracing. * **`wasm2wat`:** Analyzing the WAT code generated by `wasm2wat` allows you to see exactly which WASM instructions your CurlyWASM code produces. This helps understand the program's low-level behavior and detect errors related to types, the stack, or memory operations.
**Debugging Demoscene Effects**
Debugging in sizecoding is tough because every byte counts. Here are practical techniques for MicroW8:
* **Displaying variables:** Use `printInt` and `printChar` to show values (e.g., pixel position, FPS, sound amplitude) directly on the screen in a corner or using debug output (Ctrl Char 6). Example: in `printFps()` (section 2.4), we display FPS and the drop below 60. * **Visualization:** Draw lines (`line`) or points (`setPixel`) for particle trajectories, vectors, or SDF object boundaries. E.g., in a particle system, mark positions with pixels in a contrasting color (like 15 - white). In raymarching, you can color pixels based on the number of steps or distance to the surface. * **WASM Breakpoints:** Set breakpoints in the browser's devtools (if using `uw8 run --browser`), analyzing the decompiled WAT code (generated with `--debug`). This is useful for pixel loops or errors in complex mathematical calculations (e.g., raymarching). * **"Poor man's debugger":** Change the background color (`cls(color)`) or a corner pixel's color (`setPixel(0,0,color)`) at different points in the code to visually track which part is executing or if a condition is met. * **Tip:** Store key values (e.g., state machine state, last calculated SDF distance) in an easily accessible place in `USER_MEM` and read/display them in `upd()` to track changes without adding much debugging code to the main logic.
---

## Part 3: Demoscene and Creativity on MicroW8

In this part, we get to the heart of the demoscene: how to combine technical knowledge of MicroW8 and CurlyWASM with creative techniques to produce mesmerizing demos. The demoscene is a subculture from the 80s where creators compete in making audiovisual presentations under extreme limitations. MicroW8 is an ideal playground for modern demosceners, blending retro constraints with modern WASM.
### 1. Demoscene Philosophy on MicroW8

Programming on MicroW8 is a return to the roots of low-level programming, where every byte of code and every CPU cycle matters. Key aspects of this philosophy include:

* **Creativity in constraints:** The platform's limitations (256 KB, 320x240, no hardware 3D/sprite acceleration, no V-sync) are not obstacles but catalysts for creativity. They force thinking about clever algorithms and techniques that maximize visual and audio impact with minimal resources. * **Optimization as art:** Sizecoding is not just a necessity, but an art form. The process of optimizing code size and performance becomes an integral part of the creative process. Every saved byte is a reason for satisfaction. * **Impression over realism:** The goal in demoscene is usually not photorealism, but creating a strong, hypnotic audiovisual impression. Procedurally generated effects, abstract graphics, and synesthesia (connecting image and sound) are key. * **Knowledge sharing:** The demoscene traditionally promotes sharing knowledge, techniques, and code (e.g., at demoparties). Analyzing the code of other creators (like the MicroW8 examples) is an excellent way to learn and grow.
### 2. Key Demoscene Techniques in MicroW8

Knowledge of MicroW8 and CurlyWASM allows for the implementation of many classic and modern demoscene techniques:
* **Procedural Content Generation (PCG):** Generating graphics, sound, and other assets in real-time using algorithms. This is the essence of many demoscene effects and crucial for sizecoding, as an algorithm usually takes less space than pre-made data. * **Pixel Effects (Plasma, Interference):** Generating the color of each pixel based on its coordinates and time, often using trigonometric functions and noise. Examples: `plasma_sizecoding_1.cwa`, `interference.cwa`. * **Fractals:** Drawing fractal sets through iterative mathematical formulas. Examples: `mandelbrot.cwa`, `sierpinski_triangle.cwa`. * **Raycasting and Raymarching:** 3D/2D rendering techniques based on casting rays. Versions based on SDF (like in `urban_drift.cwa`) allow for procedural geometry definition. * **Noise Patterns:** Generating various types of noise (e.g., Perlin noise, Simplex noise - if you implement them) to create procedural textures or animations. * **Mathematics as a tool:** Mathematical functions (`sin`, `cos`, `sqrt`, `abs`, `fmod`, etc.) and bitwise operators are fundamental tools for creating visual and sound effects, as well as for optimizing calculations. In sizecoding, creative use of math to generate patterns minimizes the need to store large amounts of data. * **Palette and color cycling:** Dynamically changing colors in the palette (`PALETTE`, 0x13000) while keeping static color indices in the framebuffer allows creating animations, gradients, and "flowing" effects with minimal computational overhead. Example: `font_palette.cwa` shows palette manipulation. Color cycling is a technique where colors in the palette are cyclically shifted, creating an impression of movement. * **Memory as a resource:** Besides the framebuffer and palette, effective use of `USER_MEM` for:     * **Look-up Tables (LUTs):** As mentioned, pre-computed tables of values (e.g., sines) for fast retrieval.     * **Sprites/Fonts:** Storing custom graphic data if not generated procedurally.     * **Additional buffers:** E.g., for double buffering (`game_of_life.cwa`). * **Efficient loops:** Optimizing loops that process pixels or data is crucial for performance. Use `loop` and `branch_if`, minimize calculations inside loops, prefer direct memory access (`base?offset`). * **Bitwise operations:** Utilizing bitwise operations for fast math, data packing, and pattern generation (e.g., hashing positions for noise) is fundamental in sizecoding. * **Audiovisual synchronization:** Synchronizing visual effects with sound (generated e.g., by `sndGes`) is crucial for the demoscene feel. Time (`time()`, `TIME_MS`) is the common synchronization axis. Example: `Chaos_Rose.cwa` shows simple synchronization of note triggering with animation.
**Tunnel Effects**
Tunnels are a demoscene classic, creating the illusion of flying through a cylindrical or conical structure. They are compact and visually impressive, ideal for MicroW8 (320x240, 8 bpp). The idea is to map screen pixels to a 2D texture using distance from the center and angle as UV coordinates, with an animated offset for the motion effect.
* **Appearance:** On screen, you see a perspective tunnel with pulsating patterns, e.g., a checkerboard or gradient, moving inwards. * **Pseudocode:**     ```pseudocode
    Function tunnel_effect():
      t = get_time() // Time for animation
      texture_size = 64 // Texture size (e.g., 64x64)
      center_x = 160
      center_y = 120

      For each pixel (px, py) on the screen (0..319, 0..239):
        // Translate coordinates so the center is at (0,0)
        dx = px - center_x
        dy = py - center_y
        // Avoid division by zero at the center
        If dx == 0 and dy == 0:
            dx = 0.01 // Small value to prevent error
        // Calculate distance and angle (polar coordinates)
        dist = sqrt(dx*dx + dy*dy)
        angle = atan2(dy, dx) // Angle in radians
        // Map screen coordinates to texture coordinates (UV)
        // Use inverse distance for depth effect
        // Add time 't' for motion animation
        tex_u = (texture_size / dist) + t * motion_speed_u
        tex_v = (angle / (2 * PI)) * texture_size + t * motion_speed_v
        // Wrap texture coordinates (modulo)
        tex_u_int = floor(tex_u) mod texture_size
        tex_v_int = floor(tex_v) mod texture_size
        // Get color from the texture (stored e.g., in USER_MEM)
        color = get_pixel_from_texture(tex_u_int, tex_v_int)
        // Write color to Framebuffer
        write_pixel_to_framebuffer(px, py, color)     ```
* **Tip:** Pre-compute `sqrt` and `atan2` in a LUT in `USER_MEM` to speed up calculations in the pixel loop. The texture (e.g., 64x64) can be generated procedurally in the `start()` function or loaded from the `data` section. See `plasma_sizecoding_1.cwa` for a similar animation based on time and coordinate mapping.
**Rotozoomers**
A rotozoomer is a 2D effect that scales and rotates a texture (e.g., checkerboard, sprite) in real-time, creating dynamic motion. It's simple to implement and effective, especially in sizecoding.
* **Appearance:** A texture on the screen rotates and pulsates (changes size), creating a hypnotic effect, e.g., a spinning checkerboard. * **Pseudocode:**     ```pseudocode
    Function rotozoom_effect():
      t = get_time() // Time for animation
      texture_size = 32 // Texture size (e.g., 32x32)
      center_x = 160
      center_y = 120

      // Calculate transformation parameters (rotation and zoom)
      rotation_angle = t * rotation_speed
      scale = (sin(t * zoom_speed) + 1.0) * 0.5 + minimum_scale // Pulsating scale       cos_t = cos(rotation_angle)
      sin_t = sin(rotation_angle)
      For each pixel (px, py) on the screen (0..319, 0..239):
        // Translate coordinates so the center is at (0,0)
        dx = px - center_x
        dy = py - center_y

        // Apply inverse transformation (rotation and scaling)
        // to pixel coordinates to find the corresponding point in the texture
        tex_x = (dx * cos_t - dy * sin_t) / scale
        tex_y = (dx * sin_t + dy * cos_t) / scale
        // Wrap texture coordinates (modulo)
        tex_u_int = floor(tex_x) mod texture_size
        tex_v_int = floor(tex_y) mod texture_size
        // Correction for negative modulo results
        If tex_u_int < 0: tex_u_int += texture_size
        If tex_v_int < 0: tex_v_int += texture_size
        // Get color from the texture (stored e.g., in USER_MEM)
        color = get_pixel_from_texture(tex_u_int, tex_v_int)
        // Write color to Framebuffer
        write_pixel_to_framebuffer(px, py, color)     ```
* **Tip:** Store a small texture (e.g., 32x32) in `USER_MEM`. Use pre-computed `sin` and `cos` from a LUT for faster calculations. The effect can be combined with plasma (see `plasma_sizecoding_1.cwa`) for even more interesting patterns.
**Particle Systems**
Particle systems generate many small objects (e.g., sparks, stars, smoke) moving according to simple physics rules (velocity, gravity, lifetime). They are visually impressive and scale well on MicroW8, despite computational power limitations.
* **Appearance:** Tens or hundreds of points move across the screen, creating effects like rain, explosions, fog, often synchronized with music. * **Pseudocode:**     ```pseudocode
    // Structure (in memory) for a single particle
    // e.g., 4 x f32 = 16 bytes per particle     Structure Particle:
      x, y: position (f32)
      vx, vy: velocity (f32)
      lifetime: remaining lifetime (f32)
    // Array of particles in USER_MEM
    num_particles = 100
    particle_array = USER_MEM // Start address of the array
    Function init_particles():
      For i from 0 to num_particles - 1:
        particle_address = particle_array + i * size_of_particle_struct
        // Set initial values (e.g., random position, velocity, lifetime)         write_float(particle_address + offset_x, random_position_x())
        write_float(particle_address + offset_y, random_position_y())
        write_float(particle_address + offset_vx, random_velocity_x())
        write_float(particle_address + offset_vy, random_velocity_y())
        write_float(particle_address + offset_lifetime, random_lifetime())
    Function update_and_draw_particles():
      dt = calculate_delta_time() // Time since last frame
      For i from 0 to num_particles - 1:
        particle_address = particle_array + i * size_of_particle_struct
        // Read current particle state
        x = read_float(particle_address + offset_x)
        y = read_float(particle_address + offset_y)
        vx = read_float(particle_address + offset_vx)
        vy = read_float(particle_address + offset_vy)
        lifetime = read_float(particle_address + offset_lifetime)
        // Update state
        x = x + vx * dt
        y = y + vy * dt
        // Can add gravity: vy = vy + gravity * dt
        lifetime = lifetime - dt
        // Check if particle is "alive"
        If lifetime > 0:
          // Write updated state
          write_float(particle_address + offset_x, x)           write_float(particle_address + offset_y, y)
          write_float(particle_address + offset_vx, vx)
          write_float(particle_address + offset_vy, vy)
          write_float(particle_address + offset_lifetime, lifetime)
          // Draw the particle (e.g., as a pixel)
          px = floor(x)
          py = floor(y)
          If px >= 0 and px < 320 and py >= 0 and py < 240:
            write_pixel_to_framebuffer(px, py, particle_color)         Else:
          // Particle "died", reset it (e.g., to a new starting position)
          reset_particle(particle_address) // Sets new position, velocity, lifetime     ```
* **Tip:** Store particle data compactly in `USER_MEM`. Use `f32` for position and velocity for smooth motion. Reset particles that have gone off-screen or whose lifetime has ended to maintain a constant number of active particles. Combine with `sndGes` for music synchronization (e.g., emit particles on note triggers - see `Chaos_Rose.cwa`).
**Dynamic Palette Manipulation**
Palette manipulation allows changing colors in `PALETTE` (0x13000, 256x32 bpp RGBA) in real-time, creating effects like pulsating gradients, smooth tonal transitions, or "fading" without redrawing the entire framebuffer. This is a very efficient technique in modes with a limited number of colors (like 8 bpp in MicroW8).
* **Appearance:** Colors on the screen change smoothly, e.g., the background transitions from blue to red, objects pulse to the rhythm of the music, or the entire image smoothly fades to black. * **Pseudocode (example of animating a gradient in the palette):**     ```pseudocode
    Function animate_palette():
      t = get_time() // Time for animation

      // Loop through the part of the palette we want to animate (e.g., indices 16-31)
      For index from 16 to 31:
        // Calculate color based on time and index
        // Example: pulsating blue-red gradient
        blue = (sin(t * pulse_speed + index * blue_phase_shift) + 1.0) * 0.5 * 255
        red = (cos(t * pulse_speed + index * red_phase_shift) + 1.0) * 0.5 * 255
        green = 0 // No green in this example
        // Assemble RGBA color (or BGRA, depending on endianness and MicroW8 format)
        // Remember MicroW8 uses 32bpp RGBA in PALETTE
        color_rgba = (255 << 24) | (blue << 16) | (green << 8) | red // Example AA BB GG RR
        // Calculate address in the palette (index * 4 bytes)
        address_in_palette = PALETTE + index * 4
        // Write color to the palette (using 32-bit word write '!')
        write_word_to_memory(address_in_palette, color_rgba)     ```
* **Tip:** Limit the number of modified colors per frame if manipulating the entire palette is too computationally expensive. Pre-compute `sin`/`cos` in a LUT in `USER_MEM`. The effect can be combined with plasma or other effects based on color indices for greater dynamics (see `plasma_sizecoding_1.cwa`, `font_palette.cwa`). Remember to use the `!` operator (or `i32.store`) to write 32-bit color values and ensure 4-byte address alignment.
**Audiovisual Synchronization**
Synchronizing visuals with sound is a hallmark of the demoscene. MicroW8 enables this by reading the state of the `sndGes` sound channels (e.g., whether a note is active, its pitch, the current envelope value) or, if implementing your own `snd()` function, through direct communication between the audio and main threads (e.g., via shared memory). Visual effects reacting to music significantly enhance the experience.
* **Appearance:** Objects (e.g., circles, lines, particles) pulse, change color, size, or speed in rhythm with the music (e.g., grow on strong kick drum hits, change color with the melody). * **Pseudocode (example of reacting to a note trigger):**     ```pseudocode
    // Global variable or memory location to track trigger state
    previous_trigger_state_channel0 = 0

    Function sync_visuals_with_audio():
      // Read CTRL register for channel 0 of sndGes synthesizer (address 0x50)
      ctrl_register_channel0 = read_byte_from_memory(0x50)

      // Check trigger bit (bit 1)
      current_trigger_state = (ctrl_register_channel0 >> 1) & 1

      // Check if trigger changed from 0 to 1 (new note)       If current_trigger_state == 1 and previous_trigger_state_channel0 == 0:
        // Trigger visual effect, e.g., change background color or emit particles         change_background_color_briefly(random_color())
        emit_particle_explosion(center_x, center_y)
      // Store current trigger state for comparison in the next frame
      previous_trigger_state_channel0 = current_trigger_state

      // Another example: Scale effect based on channel volume
      // Read volume register for channel 0 (lower 4 bits of address 0x68)
      volume_channel0 = read_byte_from_memory(0x68) & 0x0F
      effect_scale = 1.0 + (volume_channel0 / 15.0) * max_enlargement
      draw_object_with_scale(effect_scale)     ```
* **Tip:** Reading `sndGes` registers every frame is cheap. React to key events (note trigger, pitch change) or averaged values (e.g., average volume) to avoid overly "jittery" visuals. Combine with particle systems (emit in rhythm) or rotozoomers (pulse in rhythm) for dynamic effects (see `Chaos_Rose.cwa` for a simple example of synchronizing a note with animation).
### 3. "Smoothness Above All"

The lack of V-sync requires striving for the most stable frame rate possible to ensure smooth animation.
* **Analyze bottlenecks:** Measure the execution time of the `upd()` function using `TIME_MS` (e.g., using the `printFps` function) to identify which parts of the code are slowest and require optimization. * **Minimize calculations in loops:** Move expensive calculations outside main loops (e.g., the pixel drawing loop). Use LUTs where possible. * **`i32` vs `f32`:** Operations on `i32` are generally faster than on `f32`. Use `f32` only when floating-point precision is needed (e.g., for position calculations, rotations, procedural patterns). Avoid unnecessary type conversions. * **Direct memory access:** In loops operating intensively on data (e.g., the framebuffer), use direct memory read/write (`base?offset`, `base!offset`, `base$offset`) instead of API function calls (`setPixel`, `getPixel`).
### 4. Creative Process Iteration

Creating demos is an evolutionary, often iterative process:
1.  **Idea:** Find inspiration in mathematics, nature, music, other demos, or simply the capabilities of MicroW8. 2.  **Minimal implementation:** Implement a basic version of the effect or concept. Don't worry about optimization immediately. 3.  **Optimization:** Once the basic version works, start optimizing: reduce code size (sizecoding), improve performance (timing). Use tools like `uw8 filter-exports`, `wasm2wat`, `wasm-opt`. Utilize direct memory access, bitwise operations, LUTs. 4.  **Add details:** Enhance the demo: add colors, animations, sound, post-processing effects (e.g., palette manipulation). 5.  **Re-optimization:** Adding new elements often introduces new overhead. Return to the optimization step. The sizecoding and optimization process is often continuous. 6.  **Polishing:** Refine audiovisual synchronization, improve animation smoothness (timing, delta time), adjust the palette, fix bugs.
**Prototyping Demos**
Creating demos is the art of starting with simple ideas and gradually developing them. Here's how to prototype on MicroW8:
* **Start from basics:** Implement a single effect (e.g., plasma, tunnel) in `upd()`. Focus on working code, even if it's simple and unoptimized. * **Test with a minimal palette:** Use the standard palette or just a few basic colors to speed up loading and iteration, focusing on the effect's logic. Refine colors later. * **Iterative addition:** After mastering one effect, add another (e.g., particles to the tunnel, sound sync with the rotozoomer), synchronizing them using `time()` or `TIME_MS`. * **Debugging:** Use `printInt` and `printFps` (from section 2.4) to check key values (e.g., FPS, particle count, coordinates). Use visualization (coloring pixels/background) to track state. * **Tip:** Test each stage on MicroW8 (or in the `uw8 run` emulator) to ensure FPS doesn't drop drastically below the target level (e.g., 60 FPS). Get inspired by simple examples like `cube_wireframe.cwa` or `plasma_sizecoding_1.cwa` to see basic code structures.
---

## Part 4: Inspirations and Demoscene Resources

The demoscene is a community full of creativity and technical virtuosity. Here's how to draw inspiration and join in:
**Iconic Demos and Their Techniques**
Analyzing classic demos can be a great source of ideas for effects that can be implemented (or adapted) on MicroW8:

* **Second Reality (Future Crew, 1993):** A PC demoscene classic, known for effects like rotozoomer, tunnel, 3D object morphing, and advanced vector graphics. Implement a simple rotozoomer (section 3.2) or a tunnel inspired by `plasma_sizecoding_1.cwa`. * **Chaos Theory (Conspiracy, 2006):** A PC demo showcasing advanced raymarching of procedural 3D scenes with excellent audiovisual synchronization. Adapt SDF techniques from `urban_drift.cwa` for similar, though simpler, effects in MicroW8. * **Elevated (RGBA, 2009):** A legendary 4KB intro on PC that renders a minimalistic but beautiful 3D scene with procedural textures and lighting, fitting everything into an extremely small size. Use `cube_wireframe.cwa` as a starting point for experiments with compact 3D demos.
**Examples on MicroW8**
Browse existing demos and examples created specifically for MicroW8, e.g., `mandelbrot.cwa` (fractals), `urban_drift.cwa` (raymarching), or `Chaos_Rose.cwa` (audiovisual synchronization). Check the official MicroW8 repository on GitHub and demos presented at demoparties (e.g., Revision, Assembly, Function) – they are often available online.
**Community Resources**
* **Pouet.net:** The largest online database of demos and intros, with download links, screenshots, comments, and results from demoparties. An indispensable source of inspiration and knowledge about demoscene history. * **Demoscene.pl / Forum.Demoscena.pl:** Polish discussion forums dedicated to the demoscene, computer graphics, tracker music, and programming. A good place to ask questions, share work, and meet the local community. * **Demoparties:** Events (gatherings) where creators present their demos, compete in contests (compos), and exchange knowledge. The largest international ones are Revision (Germany, online) and Assembly (Finland). Popular Polish parties include Riverwash, Function, Xenium. Join them (physically or online) to feel the scene's atmosphere and show your work. * **MicroW8 Community:** Look for dedicated channels on Discord, programming forums, or social media where MicroW8 users can share code, ask questions, and collaborate.
**Tip:** Start by creating a simple intro in MicroW8 (e.g., a tunnel, plasma, or particle effect with sound) and submit it to a compo in the appropriate category (e.g., Fantasy Console, Low-End Intro, or even PC 256KB Intro if you manage to optimize it that much!). Fellow coder, the demoscene world awaits!
---

## Glossary

* **Alignment:** Memory address alignment. In WASM and MicroW8, operations on multi-byte data (e.g., 32-bit `i32`, `f32` words) are most efficient (and sometimes required) when the memory address is a multiple of the data size (e.g., address divisible by 4 for i32/f32). * **API (Application Programming Interface):** A set of functions and procedures provided by MicroW8 (in the `"env"` module) for interacting with the console (drawing, sound, input, time, randomness). * **Back Buffer:** A buffer in RAM (e.g., in `USER_MEM`) used for drawing the next frame of an animation. After drawing is complete, its contents are quickly copied to the Front Buffer to avoid tearing and flickering. * **Bitfield:** A data structure where information is stored at the level of individual bits or small groups of bits within a single variable (e.g., a byte or word). The gamepad state in MicroW8 is a bitfield. * **Bitwise Operations:** Logical and shift operations performed directly on the bits of numerical values (AND `&`, OR `|`, XOR `^`, shifts `<<`, `>>`, `#>>`). Key in low-level programming, sizecoding, and procedural pattern generation. * **Built-in:** Functions or instructions natively available in the CurlyWASM language or WASM (MVP) environment that do not require importing from external modules (e.g., `sqrt`, `min`, `max`, memory and bitwise intrinsics). * **Byte:** The basic unit of digital information, consisting of 8 bits. Memory addresses in MicroW8 are byte addresses. * **Callback:** A function passed to another part of the program (e.g., the console runtime) that will be called at the appropriate time or in response to an event. In MicroW8, `start()` and `upd()` are callback functions called by the console runtime. * **Cellular Automata:** Computational models where a grid of cells with simple states evolves in discrete time steps according to rules dependent on the state of neighbors. Conway's Game of Life is an example of a cellular automaton implemented in `game_of_life.cwa`. * **Clever Algorithms:** Programming techniques that achieve a desired visual effect or functionality very efficiently in terms of resources (CPU time, memory, code size), which is crucial in sizecoding. * **Color Cycling:** An animation technique involving cyclically changing colors in the palette in the background, while pixel data in the framebuffer (indices into the palette) remains static. Creates the impression of movement, gradients, or color changes with minimal computational overhead per pixel. * **Color Palette:** A table of colors. In MicroW8, the framebuffer stores 8-bit indices into a 256-entry palette, which contains 32-bit RGBA color values. * **Compo:** (Short for Competition) Competitions organized at demoparties where creators present their productions (demos, intros, graphics, music) in specific categories (e.g., PC Demo, Oldskool Intro, Fantasy Console, 4KB Intro), competing for recognition from the audience and jury. * **Compiling:** The process of translating source code (e.g., CurlyWASM) into machine code or an intermediate format (e.g., WebAssembly) performed by a compiler (`uw8 compile`). * **Constant Folding:** A compiler optimization that involves calculating expressions composed solely of constants at compile time, rather than at program runtime. CurlyWASM performs basic constant folding. * **Control Characters:** Special characters (codes 0-31) in MicroW8 that are not printed as symbols but trigger specific text formatting actions or change the console state (e.g., setting color, cursor position, switching output to debug console). * **Control Flow:** The sequence of instructions executed in a program. Structured WASM/CurlyWASM instructions like `if`, `loop`, `block`, `branch_if` control this flow. * **CurlyWASM:** A high-level programming language with syntax similar to Rust, compiling to WebAssembly. Designed as syntactic sugar for WASM to ease coding while maintaining control over low-level aspects. * **Dead Code:** Code in a program that will never be executed. Optimization tools like `uw8 filter-exports` can remove dead code, reducing module size. * **Debug Output:** Information (e.g., variable values, status messages) printed to aid in finding and fixing errors. In MicroW8, achieved using control characters for the debug console and the `printInt` function. * **Debugging:** The process of identifying, locating, and fixing errors in a program. Tools like `uw8 --debug`, debug output, and WAT code analysis are helpful for debugging CurlyWASM/WASM code. * **Delta Time:** The time (usually in seconds or milliseconds) that has elapsed since the previous call to the scene update function (e.g., `upd()`). Used to create animations and movement with constant speed, regardless of frame rate fluctuations. Calculated from `TIME_MS` in MicroW8. * **Demoscene:** A computer subculture focused on creating and presenting (at demoparties) short, self-contained audiovisual programs (demos), often under very strict size and hardware constraints. * **Direct Memory Access:** Directly reading and writing data at specific RAM addresses (`base?offset`, `base!offset`, `base$offset`). In MicroW8, this is often more efficient and compact than using API functions, especially in loops operating on large datasets (like the framebuffer). * **Double Buffering:** A graphics rendering technique using two memory buffers: one displayed on the screen (Front Buffer) and another where the next frame is drawn (Back Buffer). After drawing is complete, the buffers are swapped (or the Back Buffer is copied to the Front Buffer). Prevents tearing and flickering. * **Dynamic Palette Manipulation:** The technique of changing colors in the palette (`PALETTE`) in real-time to achieve visual effects like animated gradients, pulsating colors, or smooth tonal transitions without modifying the framebuffer data. * **Efficient Loops:** Loops optimized for execution speed and/or minimal code size. Achieved by minimizing work inside the loop, using efficient instructions (e.g., memory operations, bitwise), and properly utilizing CurlyWASM's loop structure (`loop`, `branch_if`). * **Fantasy Console:** A virtual machine or emulator simulating simple, retro computer or console systems, but with modern development tools. MicroW8 is an example of a fantasy console. * **Flickering:** Image stuttering that can occur when screen data is updated during the drawing process. Prevented by using double buffering. * **Font:** A set of graphical representations (glyphs) of characters. MicroW8 has a built-in font in memory at address `FONT`, but a custom one can be used. * **Frame Time:** The time required to render and display a single animation frame. In MicroW8, without V-sync, it's variable and depends on the computational complexity of the `upd()` function. Measured using `TIME_MS`. * **Framebuffer:** The area of RAM that stores the graphic data (pixels) currently displayed on the screen. In MicroW8, it's an 8-bit buffer of color indices at address `FRAMEBUFFER`. * **Front Buffer:** In double buffering, the memory buffer whose contents are currently displayed on the screen. In MicroW8, this is natively the framebuffer. * **Gamepad State:** The state of the virtual gamepad in MicroW8, stored as a bitfield in memory at address `GAMEPAD`. Allows checking which buttons are pressed at any given moment. * **Gradients:** Smooth tonal transitions between colors. In MicroW8, they can be achieved by setting colors appropriately in the palette or by procedurally calculating pixel colors. * **Graphics Mode:** One of the text printing modes in MicroW8, where characters are drawn at any pixel position, and the character background is transparent. Contrasts with normal mode (8x8 grid with background). * **High-Level Syntax:** Programming language syntax that is more abstract and human-readable than low-level machine instructions. CurlyWASM is a high-level language relative to WebAssembly. * **Input:** Data originating from the user, e.g., gamepad button presses or keyboard key presses. Handled in MicroW8 via the API and the `GAMEPAD` register. * **Instruction Intrinsics:** Special keywords or functions in a programming language that map directly to single, low-level virtual machine (WASM) instructions, allowing for very precise control and optimization. Example: `i32.load`, `f32.store` in CurlyWASM. * **Intro:** A short demo, usually created with very strict size limits (e.g., 256 bytes, 1 KB, 4 KB, 64 KB), presenting condensed audiovisual effects and demonstrating the programmer's skill in sizecoding. * **Iterative Formulas:** Mathematical formulas that are repeated (iterated) multiple times, with the result of each iteration feeding into the next. Used to generate fractals, e.g., the Mandelbrot set. * **`local.tee`:** A WebAssembly (WASM) instruction that performs two operations: assigns a value to a local variable and simultaneously leaves a copy of that value on the stack, where it can be used in a subsequent expression. Often generated by the `:=` operator and the `let lazy` modifier in CurlyWASM. * **Look-up Tables (LUTs):** Tables of pre-computed values (e.g., trigonometric functions) calculated once (e.g., in `start()` or a `data` section) and stored in memory. During runtime, instead of calculating the function's value, it's read from the table based on an index. Significantly faster than repeated calculations in the main loop and key in sizecoding. * **Low-Level Programming:** Programming close to the machine or virtual machine instructions, allowing precise control over resources (memory, CPU cycles) and minimizing overhead. CurlyWASM, compiling to WASM, enables low-level programming. * **Memory Operations:** Programming language instructions or operators used for reading and writing data in RAM. In CurlyWASM, these include the `?`, `!`, `$` operators, as well as intrinsics `i32.load`, `f32.store`, `memory.copy`, `memory.fill`. * **Noise Algorithms:** Algorithms generating sequences of pseudorandom numbers with specific properties (e.g., "smooth" transitions, no visible patterns), used for creating procedural textures, terrains, or animations. * **Packing:** Compressing or packing data to reduce its size. In sizecoding, also refers to packing multiple pieces of information into fewer bits/bytes (e.g., using bitwise operations). The `uw8 pack` tool compresses the WASM module. * **Palette Manipulation:** Dynamically changing the color values stored in the palette (`PALETTE`) during program execution. This technique, combined with static color indices in the framebuffer, is used to create animation effects, e.g., color cycling. * **Particle System:** A computer graphics technique involving the simulation and rendering of a large number of small objects (particles) that move according to defined rules (e.g., physics, flocking behavior), creating complex, dynamic visual effects like smoke, fire, rain, explosions, or nebulae. * **Pattern Generation:** Creating visual or auditory patterns using mathematical or procedural algorithms, rather than using predefined graphics/samples. The basis of many demoscene effects. * **Photorealism:** Striving for realistic depiction of reality in computer graphics. In the demoscene, usually less important than creating an impressive, often abstract, visual experience. * **Pixel Data:** Data that defines the color or other properties of each individual point (pixel) of a digital image. In MicroW8, these are the 8-bit color indices in the framebuffer. * **Pixel Effects:** Visual effects where the color of each pixel is calculated independently or based on a small group of neighboring pixels, typically based on mathematical functions and animation parameters (e.g., time, source position). Examples: plasma, interference. * **Procedural Calculations:** Calculations performed at runtime to generate data (e.g., pixel colors, vertex coordinates, sound sample values) based on algorithms, rather than predefined data. * **Procedural Content Generation (PCG):** Generating assets (graphics, 3D models, textures, sounds, level data) in real-time using algorithms. A key technique in sizecoding for creating complex content with minimal data size. * **Procedural Effects:** Visual or sound effects generated in real-time using procedural algorithms, often dependent on time or other dynamic parameters. * **Programmable Filters:** Sound filters in MicroW8's `sndGes` synthesizer whose parameters (cutoff frequency, resonance) can be dynamically changed by the programmer by modifying corresponding registers in memory. * **Prototyping:** An early stage of program (e.g., demo) development, involving rapid implementation of basic functionality or a key effect, often in a simplified form, to test the concept, identify problems, and iteratively develop before full implementation and optimization. * **Randomness:** Generating sequences of numbers that appear random. Available in MicroW8 through `random()` and `randomf()` functions, controlled by `randomSeed()`. * **Raycasting:** A 3D (or 2D) graphics rendering technique involving casting rays from the viewpoint (camera) through each screen pixel into the 3D scene and calculating the pixel color based on the first object the ray hits. * **Raymarching:** A 3D rendering technique, often used with Signed Distance Fields (SDF). Instead of directly calculating the intersection of a ray with geometry, steps are taken along the ray, using the SDF value to estimate the minimum distance to the geometry and thus "march" towards the surface. * **Rotozoomer:** A 2D graphic effect involving the simultaneous rotation and scaling (zoom) of a texture or image in real-time. Popular in the demoscene for its dynamic, hypnotic appearance and relatively simple implementation. * **Runtime:** The environment in which a program is executed. For MicroW8, the runtime is the emulator or application that runs the WebAssembly module and provides access to the console's API. * **Screen Tearing:** A graphic artifact appearing as a "tear" in the image when the screen displays parts from different frames simultaneously. Caused by a lack of vertical synchronization (V-sync) between screen refresh and new frame rendering. Double buffering prevents this. * **Sequencing Operator:** The `<|` operator in CurlyWASM. Executes the left-hand expression (often for its side effects) and returns the value of the right-hand expression. Helps in sizecoding by minimizing the need for temporary variables. * **Sizecoding:** The art of writing programs (often demos or intros) of extremely small size (e.g., fitting within 256 bytes, 1 KB, 4 KB), requiring deep knowledge of the target platform, binary format, and creative programming techniques to maximize effect with minimal byte usage. * **Sound Synchronization:** Synchronizing sound effects or music with the visuals in a demo. Requires using a common time axis (e.g., `time()`, `TIME_MS`) or direct reaction to sound events to coordinate both elements. * **Stable FPS:** Maintaining a constant number of frames per second. In environments without V-sync (like MicroW8), achieving stable FPS depends on optimizing the code so that the `upd()` function execution time is relatively constant and fits within the time budget (e.g., 16.67 ms for 60 FPS). * **Syntactic Sugar:** Programming language elements that make code easier to write and read without adding fundamentally new functionality (they are translated into more basic constructs). CurlyWASM is syntactic sugar for WebAssembly. * **Time-Dependent Motion:** Animations and movement whose speed depends on delta time, rather than a constant increment value. Ensures that movement has a constant speed in the virtual world, regardless of instantaneous frame rate. * **Timing:** Managing time in programs, especially in animations and games. In MicroW8, without V-sync, requires manual delta time measurement and using it to control animation. * **Tunnel Effect:** A classic demoscene effect simulating flight through a three-dimensional tunnel. Usually implemented by mapping screen pixel coordinates to 2D texture coordinates (often in polar coordinates), where distance from the screen center corresponds to depth in the tunnel, and angle corresponds to position on the circumference. Animation is created by shifting texture coordinates over time. * **V-sync:** Vertical Synchronization. A technique synchronizing the moment a new image frame is sent to the monitor with the screen's refresh cycle. Prevents tearing. MicroW8 does not implement V-sync, requiring manual timing management and techniques like double buffering. * **Vector Operations:** Mathematical operations on vectors (e.g., addition, subtraction, scalar multiplication, dot/cross product). Key in 3D graphics, physics, and many procedural effects. In CurlyWASM, they require manual implementation or the use of libraries, if available. * **WASM Analysis:** The process of analyzing WebAssembly code (binary or WAT format) to understand its behavior, identify performance or size issues, and for reverse engineering purposes. Tools like `wasm2wat` and `wasm-opt` are helpful here. * **WASM Text Format (.wat):** A human-readable text representation of WebAssembly code. `.wat` files can be compiled to binary WASM (`wat2wasm`), and binary WASM can be decompiled to WAT (`wasm2wat`). * **WebAssembly (WASM):** A low-level, binary instruction format for a stack-based virtual machine. WASM is the compilation target for CurlyWASM and other languages. MicroW8 executes WASM modules. * **WebAssembly MVP:** Abbreviation for Minimum Viable Product. The first, basic version of the WebAssembly standard. CurlyWASM and MicroW8 currently target this version. * **Word:** In the context of 32-bit architectures (like WASM MVP), "word" often refers to a 32-bit (4-byte) unit of data (`i32`, `f32`). * **Wrap-around:** A technique used, for example, when accessing arrays or buffers (like the framebuffer), involving "wrapping" an index or coordinate when it exceeds the size limit. E.g., when accessing screen pixels, a pixel with X coordinate equal to the screen width is treated as X=0. Used in MicroW8, for example, in `game_of_life.cwa`.
---

Good luck creating demos on MicroW8! May your creations dazzle the scene!
# Exercises for [MicroW8](https://exoticorn.github.io/microw8/)

 WebAssembly-powered exercises for MicroW8 fantasy console, showcasing creative coding within platform constraints.

## About MicroW8

**MicroW8** loads WebAssembly modules with a maximum size of 256KB. Your module needs to export a function fn `upd()` which will be called once per frame. After calling `upd` MicroW8 will display the 320x240 8bpp framebuffer located at offset 120 in memory with the 32bpp palette located at 0x13000.

The memory has to be imported as `env` `memory` and has a maximum size of 256KB (4 pages).

If the module exports a function called `start`, it will be called once after the module is loaded.

### Key Features

- **Graphics Modes**: Supports text and graphics modes.
- **Primitive Functions**: Includes functions for drawing pixels, lines, circles, and rectangles.
- **Sound Capabilities**: Custom soundchip for generating sound effects and music.
- **PRNG**: Provides a good pseudo-random number generator (PRNG), namely xorshift64*, which can be initialized with a custom seed for varied random sequences.
- **Math Functions**: Supports mathematical functions such as sine, cosine, tangent, exponentiation, logarithms, and modulo operations.

### Memory map
```
00000-00040: user memory
00040-00044: time since module start in ms
00044-0004c: gamepad state
0004c-00050: number of frames since module start
00050-00070: sound data (synced to sound thread)
00070-00078: reserved
00078-12c78: frame buffer
12c78-12c7c: sound registers/work area base address (for sndGes function)
12c7c-13000: reserved
13000-13400: palette
13400-13c00: font
13c00-14000: reserved
14000-40000: user memory
```

## Some Exercises - Live Examples (more in carts/src)

1. [3D Wireframe Cube](https://exoticorn.github.io/microw8/v0.4.1/#AgEvmenHYIQ0/luYmvahuJDSK7ewWqC0bIlfA751sc6jBxGyedrH0fsQzSunqaGuAcJ8V6HiVrLGcd6DIQPMCza2VXzXiRTCtvt8L2gcMYNfNacvOupltyuXRztjlV69KL1APBHQy3NFg9k7XhY0XSfVAZF+sK/q6EGBeIYsZRipfIatJ2huh+QWbG5z72EmaMOFAbFYAAMbDPM5wUkdjS6GkGmj0uB9JDBAljdxJC6n/dfjCVbXG6HIxthhJpuzWjCKxyAxXsMPeq/mTKxEkWDSweAlW9jjMPTucnJyEVVgdTd/1kGhYtU1OIpLeFJi5fgCD8OLerrvBbvpeWmRfCRVqk/hBRJKe0c9M/D/2u7b1s1K8HPnUTigwvTtU17YQcup2JvCJ/vlH+o=) \[[source](https://github.com/zbyti/microw8-playground/blob/master/src/cube_wireframe.cwa)\]
2. [Plasma (chars)](https://exoticorn.github.io/microw8/v0.4.1/#AgF/iaR5p9nXsZHHwQXdhY0Xmeg9eA05SSIDDLvc1zq7a5ctQby0eLztGDNPm53Fk5eSR3PzqpV9mdbfcFBvuaDRSzqPvmMcHbv/IVhlHwhfThvLMcJSE6A9tIrNaMjYEdbzgKJm28rpVk85fwDmW350DD0ERfctOGaBjW4nmJ6lOG/0ypMF5ZarvBWqcM4oH2aJUtHo+fOVFqTr1gDxuaA0khofcTF7uKvnRveV5goyNUxma+NKSukTWz6g0ZFQDz6OSe37zdq4wpdDHymLs97gH3ivAHP8b7Qr95Lobvo4O3nlRvuHEIvOPpyKVFqeDLQkuJOjYbYgCISJfbd5MLzTHA==
) \[[source](https://github.com/zbyti/microw8-playground/blob/master/src/plasma_chars.cwa)\]

3. [Plasma (sizecoding)](https://exoticorn.github.io/microw8/v0.4.1/#AiXn5Zv5sAB8Jb9zSVP+RiomZbkpxOPfULXCgAfRz83hB63XJmi486XQms7kgICo8JXe6e6w1ss3FjzhN6ghO2V9IqlTJKKhKW8w6oBE9ShKD1Q96rTxWZja8b76ryWVdUcRVhk09NMD5h7q6u2CMIN7jw==) \[[source](https://github.com/zbyti/microw8-playground/blob/master/src/plasma_sizecoding_2.cwa)\]

4. [Wobbling tunnel](https://exoticorn.github.io/microw8/v0.4.1/#Ag2v9v7KuJ02v2JFdBHrzrmCo7iZoSSD+JsTOUzS12Rs2Lhfm+db5q5GKIMFv2TDi1hx+Abe5mMnTXzMoJJ9315RXfYP8VvIDF8lgebwrS+9xClGGCvPN5aBPh7Dh/8nil70ZNe/PxEwOa2XmF2cw6DVb0UYTJ2oIQbNkBoJOS1RE3LGjYw0EMenLIB/fLyBzJhmA61AgwjD) \[[source](https://github.com/zbyti/microw8-playground/blob/master/src/wobbling_tunnel.cwa)\]

5. [Wobbling tunnel with TWIRL](https://exoticorn.github.io/microw8/v0.4.1/#AjPr1wB5I2dXkp8aHDvw2oZqmu7OiMfyrqPeDJtQugM5k6f8I95C0qnW0wvmx2frTXQUVLztl6aPOWvvKDd+x+NgAUFNokxuujLPS/Z+61SrDb38Fllt1muqlDopD+NV/NF9gqcRj1ztguh9ybDo/3FJl3K8SnKkOELl46Si8JvgRRDUGgpj65wUCW2aEWAo9TGJiZJcA2/JjbJ/5fPQckGphGZSCMY=) \[[source](https://github.com/zbyti/microw8-playground/blob/master/src/wobbling_twirl_tunnel.cwa)\]

6. [Interference](https://exoticorn.github.io/microw8/v0.4.1/#AiPf29DQ9JGfyUOPOUyZVq01QvZDsPrtbWDaFM/wTNHkKqyvd6HJHxich9pmw8oha3ht8UWZsTEzwfim97ui3kyXckYVzLgFbnZlYqhngMGG77h81jEwM5oU9h38htkXC80xSZ+jYWU9l6XSyGaV0iCWY6yPN/is1gIhsTudm59xWqEEWkHeWYNm131C9QvcHaFy4TKDVKVVJIMfZDP8mmFk9X1zmYV5RjzIU0ve1ZkNj//P3PnyZMpKCqW+koD6PbJpZGfTBEfUIvVYwtxZiGZ3WMLFxKT86hf1uhmCGY1zomNXouAgSLRePTAT+eXMRXkzDT2Ea4qFbN9C9sgNg1luUmgfG50tOGc88uRCfcY1jbZ04yW/6TEfe5lXWQhQV/msZ1rrJPF6OnCa0JXN97wGGHrw0Cz9CmD/Bqd+wpQjoA6NadrDyVs94aJu/pZT3I7RVrUBZwSqpsCyz4qa/DbgEVar3q//5aq6Lxnkh+M7Dn9/Q5CTbZW+mPp88lynOWXgHkYgh7MuxTEScEtR18PtCcuJ+8oHokiCh5H3dfWQwdTVM7KdgVLqwgRmeHuqVZV7/BGiw9HXcOTzgWsgqlvRkM8DwGQDWbv27w4d7qnjw/2L6HWpg2FrAR8z7nJ7+ZkaEQKyJcOJ7c8RuGbRlI+MwqBo7wHGEQ7fqimBFoM1YKax+xjFEcSGde+zSy7FhDEHDA==) \[[source](https://github.com/zbyti/microw8-playground/blob/master/src/interference.cwa)\]

7. [Bubble Uiverse](https://exoticorn.github.io/microw8/v0.4.1/#AimPnnXU3uH5kPqYOnQA7SLsRPTyAG+VXXfRa2Yaj3LN/Xwo9LsPQdouPHiQnmU9tcmo4GqBlsPIeGn1ssPDYCH29+C8Dw33pbkJiQGhforpnqshPo/1Y+5q/jOiqSb1k3imCiNynboZp+6q00Si/+CARacq3cfRYIp8w7XbPygh0Vlu8jawEg5ETN4MAk8sV3rXSsw0Otu5JY/LRYS3DXOKve9CihSFkz3AhtTLIkNe4Zgtzu1p/Sua06Bpp7Ju3c6gttDx37AC4f5pQDB/tMyAaPSLr5hM5pMICdy/w6xOwkSTX87l7ClHSzqdaRkvoo0RjuxXwri30bJeQ9e9f4ctNyvoEz9i3edA2yOXqszVnUMwNQBpbJsHJJsJkdK1t7IaLWdmpREnafZ3OEPZcxuqv0kc4KJA2ANPLNC0/EFoZyA=) \[[source](https://github.com/zbyti/microw8-playground/blob/master/src/bubbleuniverse.cwa)\]

## Best demos

* [Ilmenit / Agenda - Urban Drift](https://exoticorn.github.io/microw8/v0.4.1/#AgEvierdnH17S/tTxm32eLToCJJFBNvK89T9q89dmLJGAIGAYeKq8LKYQ67hit19z4h+1zGX+oVGbKMC4p2H0lpiZnHdsG3Sa82XWZPpNHYxlKUah+SvTbC0H2ZCjJEniiDDrLBd+Kf/9uVzQ7DEJ+uFQTPWHxWtAkIujmb3wJhu3htxSiunTEl0KkOaaXu86bbY0cjjbyPWrx38rfx/3crZpI+bL7BQURAB/lRygznyPRnZHYMbCKEwcZqzQcjTiyLtMNSdBSpJKTyb8HTgvHpxPOP9ByJfmhj4Z+7ttUEUzDGrgVb07c6cM6CPgmTRCApZeBQEuC8C60ovHJwjJuEGFLt/rk5MvZc5j5sEb/eFra4bsbPclClVYj3WG3dUN3Jq8aEl/GoO+xlV+GGOBGFefkKfwmJ4ZlCLMDTLJVuUW7pj2aDDfjJsw4Vr95Wc3iNYZCu2SYDsWlZm6wkxK1ALSglp0N1QWsN9hghnnsgygKuZT3Wchi/QgJSX0AFP00MZ+p3uQCi3pLAKzoUVKJnHjDi66wLKOkGBpQ2Kt0LGToH0DVDrdFXvYT2O/zqX9eo+aGVMQyoUs7+0T4D6LCuwTNiG5viz+gkI/8kKcxUJhLTC/llAn4oM4o9EtHpQQOrjpQtMRFCU9Nr29hHiMHVSIlJuMbODMN61Jd+4vSQ=) \[[source](https://github.com/ilmenit/sizecoding/blob/main/urban_drift/urban_drift.cwa)\]
* [Ilmenit / Agenda - Encounter](https://exoticorn.github.io/microw8/v0.4.1/#AgMvvqs+jH95brXMAYjUjZwn1apTrm62ncvO+qq+kAesx0vh5NB3sa3YEg8JasHVk0OOFeN09Qi/yWyEuuIHweJv5+qt4lQhS0q/exKHo4rtSsnqkY7oWUwXXgbWfGEwKrTto4wxOG4JXZck7ehBB9YHmyanOZxFZeCkpib2M/JXhCmCfPb3mF6tq++ZG2Mm73NopaaKwUFHm2KjpEjYFMEzCZsu98uZmvhD5GzCUXSw8G5Z1V8nfv9uiIQ1+5N+rcjpFezbIXG5/haUR7Lnre3xZVJcp+I6rXkboKqK6SoG5h92w/jndB3sdZyT4G9Lq872lkEkUIM7ciqdsyYJMg==) \[[source](https://github.com/ilmenit/sizecoding/blob/main/Encounter/encounter.cwa)\]
* [Ilmenit / Agenda - Chaos Rose](https://exoticorn.github.io/microw8/v0.4.1/#AueZxpSGIG269YkjN454SUTUrnAegYdk24h0y2jdoPJUyS7IDp46h+WdzhmLFBQ5NrbUAPm6vGYBtNZpO1j1QYIgke2oA0ce4qyl5a9ycZC5d4+kSiBDkGyB0haQras3rA6KiGfoaad7eL9UTcQHs9RhI7sJjFQPBcT/FoWwxPu/uPlKGlUAIO7AlqHlbSn/36DUpSJmuSiIBaGkNRmG6rD7ypVjrGkk38Hz4JbXg/cnQ1n5s7krlWROCon2yfqKH6M2SNP57mmCZMAmHguprvRHGszclgATdSqwtJoByvw07UjXVP6fuk8U2Wmj+32fZQWbMu6TyMbkF5ndBw==) \[[source](https://github.com/ilmenit/sizecoding/blob/main/chaos_rose/Chaos_Rose.cwa)\]
* [RE:FORM by Rivals](https://www.pouet.net/prod.php?which=104008)

## Links

* [MicroW8](https://exoticorn.github.io/microw8/)
* [MicroW8 Docs](https://exoticorn.github.io/microw8/docs/)
* [MicroW8 GitHub repository](https://github.com/exoticorn/microw8)
* [CurlyWASM GitHub repository](https://github.com/exoticorn/curlywas)
* [WebAssembly](https://developer.mozilla.org/en-US/docs/WebAssembly)
* [MicroW8 DEMOZOO](https://demozoo.org/platforms/96/)
* [MicroW8 pouët.net](https://www.pouet.net/prodlist.php?platform[0]=MicroW8&page=1)
* [Ilmenit - sizecoding ](https://github.com/ilmenit/sizecoding)
* [Awesome Demoscene](https://github.com/psykon/awesome-demoscene)
* [Effects listing and demos references](https://democyclopedia.wordpress.com/2015/10/25/liste-des-effets/)
* [Fantasy consoles](https://github.com/paladin-t/fantasy)

## uw8 dev tool

```
uw8 run [<options>] <file>

Runs <file> which can be a binary WebAssembly module, an `.uw8` cart, a wat (WebAssembly text format) source file or a CurlyWas source file.

Options:

-b, --browser           : Run in browser instead of using native runtime
-t, --timeout FRAMES    : Sets the timeout in frames (1/60s)
-w, --watch             : Reloads the given file every time it changes on disk.
-p, --pack              : Pack the file into an .uw8 cart before running it and print the resulting size.
-u, --uncompressed      : Use the uncompressed uw8 format for packing.
-l LEVEL, --level LEVEL : Compression level (0-9). Higher compression levels are really slow.
-o FILE, --output FILE  : Write the loaded and optionally packed cart back to disk.

when using the native runtime:

-m, --no-audio          : Disable audio, also reduces cpu load a bit
--no-gpu                : Force old cpu-only window code
--filter FILTER         : Select an upscale filter at startup
--fullscreen            : Start in fullscreen mode
--fps                   : Show FPS (STDOUT)

Note that the cpu-only window does not support fullscreen nor upscale filters.

Unless --no-gpu is given, uw8 will first try to open a gpu accelerated window, falling back to the old cpu-only window if that fails.
Therefore you should rarely need to manually pass --no-gpu. If you prefer the old pixel doubling look to the now default crt filter,
you can just pass "--filter nearest" or "--filter 1".

The upscale filter options are:
1, nearest              : Anti-aliased nearest filter
2, fast_crt             : Very simple, cheap crt filter, not very good below a window size of 960x720
3, ss_crt               : Super sampled crt filter, a little more demanding on the GPU but scales well to smaller window sizes
4, chromatic_crt        : Variant of fast_crt with a slight offset of the three color dots of a pixel, still pretty cheap
5, auto_crt (default)   : ss_crt below 960x720, chromatic_crt otherwise

You can switch the upscale filter at any time using the keys 1-5. You can toggle fullscreen with F.

uw8 pack [<options>] <infile> <outfile>

Packs the WebAssembly module or text file, or CurlyWas source file into a .uw8 cart.

Options:

-u, --uncompressed      : Use the uncompressed uw8 format for packing.
-l LEVEL, --level LEVEL : Compression level (0-9). Higher compression levels are really slow.


uw8 unpack <infile> <outfile>

Unpacks a MicroW8 module into a standard WebAssembly module.


uw8 compile [<options>] <infile> <outfile>

Compiles a CurlyWas source file to a standard WebAssembly module. Most useful together with
the --debug option to get a module that works well in the Chrome debugger.

Options:

-d, --debug             : Generate a name section to help debugging


uw8 filter-exports <infile> <outfile>

Reads a binary WebAssembly module, removes all exports not used by the MicroW8 platform + everything that is unreachable without those exports and writes the resulting module to <outfile>.
```

## How to add custom Syntax Highlighting to [Sublime Text](https://www.sublimetext.com/)

#### **1. Create or obtain a [Syntax Definition File](https://github.com/zbyti/microw8-playground/blob/master/syntax/SublimeText/CurlyWASM.sublime-syntax)**
- The file must be in **`.sublime-syntax`** format (YAML-based).  
- Example: `CurlyWASM.sublime-syntax` (for your `.cwa` files).

#### 2. Locate the correct folder
- Open Sublime Text.
- Go to:  
  **`Preferences → Browse Packages`**  
  This opens the `Packages` folder where syntax files belong.

#### 3. Place the file
- Copy your `.sublime-syntax` file into:  
  - **`Packages/User/`** (recommended for custom syntaxes).  
  - *Alternatively*, create a subfolder like `Packages/CurlyWASM/` for organization.

#### 4. Refresh Sublime Text
- **Method 1**: Restart Sublime Text.  
- **Method 2**: Use the console (**`Ctrl + ~`**) and run:  
  ```python
  sublime.run_command("refresh_package_list")
  ```

#### 5. Assign the syntax to files
1. Open a `.cwa` file.  
2. Click the current syntax name in the **bottom-right corner** (e.g., "Plain Text").  
3. Select: **`Open all with current extension as... → CurlyWASM`**.

---
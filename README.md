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

## Exercises Live Examples

1. [3D Wireframe Cube](https://exoticorn.github.io/microw8/v0.4.1/#AgEvmenHYIQ0/luYmvahuJDSK7ewWqC0bIlfA751sc6jBxGyedrH0fsQzSunqaGuAcJ8V6HiVrLGcd6DIQPMCza2VXzXiRTCtvt8L2gcMYNfNacvOupltyuXRztjlV69KL1APBHQy3NFg9k7XhY0XSfVAZF+sK/q6EGBeIYsZRipfIatJ2huh+QWbG5z72EmaMOFAbFYAAMbDPM5wUkdjS6GkGmj0uB9JDBAljdxJC6n/dfjCVbXG6HIxthhJpuzWjCKxyAxXsMPeq/mTKxEkWDSweAlW9jjMPTucnJyEVVgdTd/1kGhYtU1OIpLeFJi5fgCD8OLerrvBbvpeWmRfCRVqk/hBRJKe0c9M/D/2u7b1s1K8HPnUTigwvTtU17YQcup2JvCJ/vlH+o=) \[[source](https://github.com/zbyti/microw8-playground/blob/master/src/cube_wireframe.cwa)\]
2. [Plasma (chars)](https://exoticorn.github.io/microw8/v0.4.1/#AgF/iaR5p9nXsZHHwQXdhY0Xmeg9eA05SSIDDLvc1zq7a5ctQby0eLztGDNPm53Fk5eSR3PzqpV9mdbfcFBvuaDRSzqPvmMcHbv/IVhlHwhfThvLMcJSE6A9tIrNaMjYEdbzgKJm28rpVk85fwDmW350DD0ERfctOGaBjW4nmJ6lOG/0ypMF5ZarvBWqcM4oH2aJUtHo+fOVFqTr1gDxuaA0khofcTF7uKvnRveV5goyNUxma+NKSukTWz6g0ZFQDz6OSe37zdq4wpdDHymLs97gH3ivAHP8b7Qr95Lobvo4O3nlRvuHEIvOPpyKVFqeDLQkuJOjYbYgCISJfbd5MLzTHA==
) \[[source](https://github.com/zbyti/microw8-playground/blob/master/src/plasma_chars.cwa)\]
3. [Plasma (chars) without LUT for sine](https://exoticorn.github.io/microw8/v0.4.1/#AjHnyuOJvwzFAi17m1Wb0Dzflox+KPvif2AAel/JMsHWqIc1KCmnHOnktl4nB7EtCwJNSYFB6q1X9cgsjKicDwcyJyUa7IdmVXazMtwFsNJZvYWS5nNHb8mkkxwn8GpM6+/9iIHLT+09a/WtdPFNT4OS1tSCdIELreaKbngHNZjcJcy+uys+4aQn/oBlTenaUr7dkssUqOf4s5naAj60Nv3kR5zVW/IvIISMQPsqalMFe+X3W5kPJ9BTqEbW8d4M6+Y=) \[[source](https://github.com/zbyti/microw8-playground/blob/master/src/plasma_chars_no_lut.cwa)\]

4. [Plasma 1 (sizecoding)](https://exoticorn.github.io/microw8/v0.4.1/#AgRfwyD42RDl5f3KVLZZbihhRlTzvpi6x5iiUuDH6l4MfabYlwjxGKogclynAPxG/cprkeiidc0DNC2R4lDMUbcUvQiT5MN3jeFDNa0kl46fELcPdW7Mf2mI8f7rUgZYNr0OgTiWc2is8KcbPw==) \[[source](https://github.com/zbyti/microw8-playground/blob/master/src/plasma_sizecoding_1.cwa)\]

5. [Plasma 2 (sizecoding)](https://exoticorn.github.io/microw8/v0.4.1/#AiXn5Zv5sAB8Jb9zSVP+RiomZbkpxOPfULXCgAfRz83hB63XJmi486XQms7kgICo8JXe6e6w1ss3FjzhN6ghO2V9IqlTJKKhKW8w6oBE9ShKD1Q96rTxWZja8b76ryWVdUcRVhk09NMD5h7q6u2CMIN7jw==) \[[source](https://github.com/zbyti/microw8-playground/blob/master/src/plasma_sizecoding_2.cwa)\]

## Best demos

* [Ilmenit / Agenda - Urban Drift](https://exoticorn.github.io/microw8/v0.4.1/#AgEvierdnH17S/tTxm32eLToCJJFBNvK89T9q89dmLJGAIGAYeKq8LKYQ67hit19z4h+1zGX+oVGbKMC4p2H0lpiZnHdsG3Sa82XWZPpNHYxlKUah+SvTbC0H2ZCjJEniiDDrLBd+Kf/9uVzQ7DEJ+uFQTPWHxWtAkIujmb3wJhu3htxSiunTEl0KkOaaXu86bbY0cjjbyPWrx38rfx/3crZpI+bL7BQURAB/lRygznyPRnZHYMbCKEwcZqzQcjTiyLtMNSdBSpJKTyb8HTgvHpxPOP9ByJfmhj4Z+7ttUEUzDGrgVb07c6cM6CPgmTRCApZeBQEuC8C60ovHJwjJuEGFLt/rk5MvZc5j5sEb/eFra4bsbPclClVYj3WG3dUN3Jq8aEl/GoO+xlV+GGOBGFefkKfwmJ4ZlCLMDTLJVuUW7pj2aDDfjJsw4Vr95Wc3iNYZCu2SYDsWlZm6wkxK1ALSglp0N1QWsN9hghnnsgygKuZT3Wchi/QgJSX0AFP00MZ+p3uQCi3pLAKzoUVKJnHjDi66wLKOkGBpQ2Kt0LGToH0DVDrdFXvYT2O/zqX9eo+aGVMQyoUs7+0T4D6LCuwTNiG5viz+gkI/8kKcxUJhLTC/llAn4oM4o9EtHpQQOrjpQtMRFCU9Nr29hHiMHVSIlJuMbODMN61Jd+4vSQ=) \[[source](https://github.com/ilmenit/sizecoding/blob/main/urban_drift/urban_drift.cwa)\]
* [Ilmenit / Agenda - Encounter](https://exoticorn.github.io/microw8/v0.4.1/#AgMvvqs+jH95brXMAYjUjZwn1apTrm62ncvO+qq+kAesx0vh5NB3sa3YEg8JasHVk0OOFeN09Qi/yWyEuuIHweJv5+qt4lQhS0q/exKHo4rtSsnqkY7oWUwXXgbWfGEwKrTto4wxOG4JXZck7ehBB9YHmyanOZxFZeCkpib2M/JXhCmCfPb3mF6tq++ZG2Mm73NopaaKwUFHm2KjpEjYFMEzCZsu98uZmvhD5GzCUXSw8G5Z1V8nfv9uiIQ1+5N+rcjpFezbIXG5/haUR7Lnre3xZVJcp+I6rXkboKqK6SoG5h92w/jndB3sdZyT4G9Lq872lkEkUIM7ciqdsyYJMg==) \[[source](https://github.com/ilmenit/sizecoding/blob/main/Encounter/encounter.cwa)\]
* [Ilmenit / Agenda - Chaos Rose](https://exoticorn.github.io/microw8/v0.4.1/#AueZxpSGIG269YkjN454SUTUrnAegYdk24h0y2jdoPJUyS7IDp46h+WdzhmLFBQ5NrbUAPm6vGYBtNZpO1j1QYIgke2oA0ce4qyl5a9ycZC5d4+kSiBDkGyB0haQras3rA6KiGfoaad7eL9UTcQHs9RhI7sJjFQPBcT/FoWwxPu/uPlKGlUAIO7AlqHlbSn/36DUpSJmuSiIBaGkNRmG6rD7ypVjrGkk38Hz4JbXg/cnQ1n5s7krlWROCon2yfqKH6M2SNP57mmCZMAmHguprvRHGszclgATdSqwtJoByvw07UjXVP6fuk8U2Wmj+32fZQWbMu6TyMbkF5ndBw==) \[[source](https://github.com/ilmenit/sizecoding/blob/main/chaos_rose/Chaos_Rose.cwa)\]

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

## **How to add custom Syntax Highlighting to Sublime Text**

#### **1. Create or obtain a [Syntax Definition File](https://github.com/zbyti/microw8-playground/blob/master/CurlyWASM.sublime-syntax)**
- The file must be in **`.sublime-syntax`** format (YAML-based).  
- Example: `CurlyWASM.sublime-syntax` (for your `.cwa` files).

#### **2. Locate the correct folder**
- Open Sublime Text.
- Go to:  
  **`Preferences → Browse Packages`**  
  This opens the `Packages` folder where syntax files belong.

#### **3. Place the file**
- Copy your `.sublime-syntax` file into:  
  - **`Packages/User/`** (recommended for custom syntaxes).  
  - *Alternatively*, create a subfolder like `Packages/CurlyWASM/` for organization.

#### **4. Refresh Sublime Text**
- **Method 1**: Restart Sublime Text.  
- **Method 2**: Use the console (**`Ctrl + ~`**) and run:  
  ```python
  sublime.run_command("refresh_package_list")
  ```

#### **5. Assign the syntax to files**
1. Open a `.cwa` file.  
2. Click the current syntax name in the **bottom-right corner** (e.g., "Plain Text").  
3. Select: **`Open all with current extension as... → CurlyWASM`**.

---
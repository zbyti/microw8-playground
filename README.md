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

1. [3D Wireframe Cube](https://exoticorn.github.io/microw8/v0.4.1/#AgEvmenHYIQ0/luYmvahuJDSK7ewWqC0bIlfA751sc6jBxGyedrH0fsQzSunqaGuAcJ8V6HiVrLGcd6DIQPMCza2VXzXiRTCtvt8L2gcMYNfNacvOupltyuXRztjlV69KL1APBHQy3NFg9k7XhY0XSfVAZF+sK/q6EGBeIYsZRipfIatJ2huh+QWbG5z72EmaMOFAbFYAAMbDPM5wUkdjS6GkGmj0uB9JDBAljdxJC6n/dfjCVbXG6HIxthhJpuzWjCKxyAxXsMPeq/mTKxEkWDSweAlW9jjMPTucnJyEVVgdTd/1kGhYtU1OIpLeFJi5fgCD8OLerrvBbvpeWmRfCRVqk/hBRJKe0c9M/D/2u7b1s1K8HPnUTigwvTtU17YQcup2JvCJ/vlH+o=)

## Best demos

* [Ilmenit / Agenda - Urban Drift](https://exoticorn.github.io/microw8/v0.4.1/#AgEvierdnH17S/tTxm32eLToCJJFBNvK89T9q89dmLJGAIGAYeKq8LKYQ67hit19z4h+1zGX+oVGbKMC4p2H0lpiZnHdsG3Sa82XWZPpNHYxlKUah+SvTbC0H2ZCjJEniiDDrLBd+Kf/9uVzQ7DEJ+uFQTPWHxWtAkIujmb3wJhu3htxSiunTEl0KkOaaXu86bbY0cjjbyPWrx38rfx/3crZpI+bL7BQURAB/lRygznyPRnZHYMbCKEwcZqzQcjTiyLtMNSdBSpJKTyb8HTgvHpxPOP9ByJfmhj4Z+7ttUEUzDGrgVb07c6cM6CPgmTRCApZeBQEuC8C60ovHJwjJuEGFLt/rk5MvZc5j5sEb/eFra4bsbPclClVYj3WG3dUN3Jq8aEl/GoO+xlV+GGOBGFefkKfwmJ4ZlCLMDTLJVuUW7pj2aDDfjJsw4Vr95Wc3iNYZCu2SYDsWlZm6wkxK1ALSglp0N1QWsN9hghnnsgygKuZT3Wchi/QgJSX0AFP00MZ+p3uQCi3pLAKzoUVKJnHjDi66wLKOkGBpQ2Kt0LGToH0DVDrdFXvYT2O/zqX9eo+aGVMQyoUs7+0T4D6LCuwTNiG5viz+gkI/8kKcxUJhLTC/llAn4oM4o9EtHpQQOrjpQtMRFCU9Nr29hHiMHVSIlJuMbODMN61Jd+4vSQ=)
* [Ilmenit / Agenda - Encounter](https://exoticorn.github.io/microw8/v0.4.1/#AgMvvqs+jH95brXMAYjUjZwn1apTrm62ncvO+qq+kAesx0vh5NB3sa3YEg8JasHVk0OOFeN09Qi/yWyEuuIHweJv5+qt4lQhS0q/exKHo4rtSsnqkY7oWUwXXgbWfGEwKrTto4wxOG4JXZck7ehBB9YHmyanOZxFZeCkpib2M/JXhCmCfPb3mF6tq++ZG2Mm73NopaaKwUFHm2KjpEjYFMEzCZsu98uZmvhD5GzCUXSw8G5Z1V8nfv9uiIQ1+5N+rcjpFezbIXG5/haUR7Lnre3xZVJcp+I6rXkboKqK6SoG5h92w/jndB3sdZyT4G9Lq872lkEkUIM7ciqdsyYJMg==)
* [Ilmenit / Agenda - Chaos Rose](https://exoticorn.github.io/microw8/v0.4.1/#AueZxpSGIG269YkjN454SUTUrnAegYdk24h0y2jdoPJUyS7IDp46h+WdzhmLFBQ5NrbUAPm6vGYBtNZpO1j1QYIgke2oA0ce4qyl5a9ycZC5d4+kSiBDkGyB0haQras3rA6KiGfoaad7eL9UTcQHs9RhI7sJjFQPBcT/FoWwxPu/uPlKGlUAIO7AlqHlbSn/36DUpSJmuSiIBaGkNRmG6rD7ypVjrGkk38Hz4JbXg/cnQ1n5s7krlWROCon2yfqKH6M2SNP57mmCZMAmHguprvRHGszclgATdSqwtJoByvw07UjXVP6fuk8U2Wmj+32fZQWbMu6TyMbkF5ndBw==)

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
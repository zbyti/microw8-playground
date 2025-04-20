# Kompletny Tutorial MicroW8: Od Podstaw do Zaawansowanego Demosceningu

Witaj w fascynującym świecie MicroW8, wirtualnej konsoli fantasy stworzonej z myślą o demoscenowcach i sizecoderach, którzy chcą tworzyć audiowizualne arcydzieła w ramach surowych ograniczeń sprzętowych. Ten tutorial jest kompleksowym przewodnikiem po programowaniu na MicroW8 z użyciem języka CurlyWASM, łączącym podstawy, zaawansowane techniki i filozofię demosceny. Niezależnie od tego, czy dopiero zaczynasz swoją przygodę, czy jesteś doświadczonym programistą szukającym wyzwań w sizecodingu, znajdziesz tu wszystko, czego potrzebujesz, aby tworzyć imponujące dema na tej platformie.

Tutorial został podzielony na trzy główne części oraz sekcję inspiracji, które płynnie się uzupełniają:
1.  **Podstawy MicroW8 i CurlyWASM** – wprowadzenie do konsoli, języka i kluczowych mechanizmów programowania.
2.  **Zaawansowane Techniki** – dogłębne omówienie optymalizacji, zarządzania pamięcią, efektów demoscenowych i dźwięku.
3.  **Demoscena i Kreatywność** – filozofia demosceny, kluczowe techniki wizualne i praktyczne podejście do tworzenia dem.
4.  **Inspiracje i Zasoby Demosceny** – przegląd kultowych dem, przykładów i społeczności.

Każda sekcja zawiera szczegółowe wyjaśnienia, przykłady kodu (CurlyWASM lub pseudokod), wskazówki dotyczące optymalizacji oraz odniesienia do przykładów z plików, takich jak `mandelbrot.cwa`, `plasma_sizecoding_1.cwa` czy `urban_drift.cwa`. Dokument kończy się słowniczkiem, który wyjaśnia kluczowe pojęcia związane z MicroW8, CurlyWASM i demosceną.

Przygotuj się na podróż przez świat, w którym każdy bajt ma znaczenie, a kreatywność rodzi się z ograniczeń. Zaczynajmy!

---

## Część 1: Podstawy MicroW8 i CurlyWASM

Witaj koderze! MicroW8 to fascynująca konsola fantasy, idealna dla miłośników sizecodingu i demosceny. W tej części poznasz podstawy programowania na tej platformie z użyciem CurlyWASM – języka, który daje Ci kontrolę nad każdym bajtem generowanego kodu WebAssembly (WASM). Zaczniemy od omówienia konsoli, jej ograniczeń i mapy pamięci, przejdziemy przez składnię CurlyWASM, a na koniec pokażemy, jak stworzyć proste „Hello, World!”.

### 1. Czym jest MicroW8?

MicroW8 to wirtualna konsola zaprojektowana z myślą o tworzeniu niewielkich dem i gier, inspirowana klasycznymi systemami z lat 80., ale działająca na nowoczesnym WebAssembly. Jej kluczowe cechy to:

* **Ograniczenia:** Moduły WASM mają maksymalny rozmiar 256 KB. Pamięć jest ograniczona do 256 KB (4 strony po 64 KB każda). Ekran ma rozdzielczość 320x240 pikseli z 8-bitową paletą kolorów. Brak sprzętowej synchronizacji pionowej (V-sync) oznacza, że płynność animacji zależy od wydajności Twojego kodu w funkcji `upd()`, a timing musisz obsłużyć ręcznie.
* **Główne punkty wejścia:** Twój kod musi eksportować funkcję `export fn upd()`, wywoływaną raz na klatkę. Opcjonalna funkcja `export fn start()` jest wywoływana jednorazowo po załadowaniu modułu, idealna do inicjalizacji.
* **Pamięć:** Dostępna pamięć jest zaimportowana jako `"env.memory"` z minimalną liczbą stron. MicroW8 udostępnia mapę pamięci z predefiniowanymi obszarami, takimi jak framebuffer (`FRAMEBUFFER`), paleta kolorów (`PALETTE`), font (`FONT`) i obszar pamięci użytkownika (`USER_MEM`).

### 2. Podstawy języka CurlyWASM

CurlyWASM to syntactic sugar dla WebAssembly, zaprojektowany tak, aby był bliski instrukcjom WASM, ale wygodniejszy w pisaniu. Kompilator CurlyWASM wykonuje jedynie podstawowe optymalizacje (constant folding), więc to Ty odpowiadasz za wydajność i rozmiar kodu.

#### 2.1. Struktura kodu i komentarze

Kod CurlyWASM składa się z importów, deklaracji globalnych zmiennych i stałych, sekcji danych oraz definicji funkcji. Komentarze wyglądają następująco:

```curlywasm
// To jest komentarz liniowy

/*
  To
  jest
  komentarz
  blokowy
*/
```

#### 2.2. Typy danych

CurlyWASM (oparty na WASM MVP) obsługuje cztery podstawowe typy:

* `i32`: 32-bitowa liczba całkowita ze znakiem. **To preferowany typ dla operacji na liczbach całkowitych w MicroW8.**
* `i64`: 64-bitowa liczba całkowita ze znakiem. **Zdecydowanie unikaj tego typu! WebAssembly MVP koncentruje się na 32-bitowych operacjach, a większość (jeśli nie wszystkie) funkcji API MicroW8 używa i32. Użycie i64 generuje dodatkowy narzut i może być niezgodne z API.**
* `f32`: 32-bitowa liczba zmiennoprzecinkowa. **To preferowany typ dla operacji zmiennoprzecinkowych.**
* `f64`: 64-bitowa liczba zmiennoprzecinkowa. **Zdecydowanie unikaj tego typu z tych samych powodów co i64. Większość funkcji API używa f32.**

#### 2.3. Literały

* **Liczby całkowite:** Dziesiętne lub szesnastkowe (np. `123`, `-7878`, `0xf00`).
* **Liczby zmiennoprzecinkowe:** Dziesiętne (np. `0.464`, `3.141`, `-10.0`). Można je także zapisywać z `_f` na końcu (np. `10_f`, `0x0a_f`) dla explicite określenia, że są to literały `f32`.
* **Literały znakowe:** Pojedyncze znaki w pojedynczych cudzysłowach (np. `'A'`). Mogą zawierać do 4 znaków, interpretowane jako little-endian i32.

#### 2.4. Zmienne i stałe

* **Zmienne globalne:** Deklarowane poza funkcjami przy użyciu `global`. Muszą być mutable (`mut`) i mieć wartość początkową.
    ```curlywasm
    global mut mojGlobal: i32 = 0;
    ```
* **Stałe globalne:** Deklarowane poza funkcjami przy użyciu `const`. Wartość musi być wyrażeniem stałym.
    ```curlywasm
    const ROZMIAR_EKRANU = 320 * 240;
    const PI = 3.14159265;
    ```
    **Ważne:** Stałe globalne nie mogą być deklarowane wewnątrz funkcji. W wyrażeniach użytych do *deklaracji* stałej (`const`) nie możesz używać operatora `as` do rzutowania typów. Operator `as` jest jednak dozwolony i powszechnie używany w innych kontekstach kodu, poza deklaracjami `const`.
* **Zmienne lokalne:** Deklarowane wewnątrz funkcji przy użyciu `let`. Są zawsze mutable, typ jest inferowany przy inicjalizacji.
    ```curlywasm
    fn mojaFunkcja() {
      let licznik: i32; // Deklaracja
      let nazwa = 'smok'; // Deklaracja i inicjalizacja (typ inferowany jako i32, 4 bajty = 4 znaki)
      let odleglosc = 100 as f32; // Użycie 'as' w wyrażeniu jest poprawne
    }
    ```
* **Modyfikatory zmiennych lokalnych (`inline`, `lazy`):** Optymalizują rozmiar kodu, zmieniając moment ewaluacji wyrażenia.
    ```curlywasm
    let inline wiersz = obliczWiersz(y); // Obliczane przy każdym użyciu 'wiersz', jak makro
    let lazy kolumna = obliczKolumne(x); // Obliczane raz, przy pierwszym użyciu 'kolumna' (często używa local.tee)
    ```

#### 2.5. Operatory i wyrażenia

CurlyWASM używa notacji infiksowej dla operatorów:

* **Arytmetyczne:** `+`, `-`, `*`, `/`, `%` (modulo signed), `#/`, `#%` (modulo unsigned).
* **Bitowe:** `&` (AND), `|` (OR), `^` (XOR), `<<`, `>>` (shift signed), `#>>` (shift unsigned). **Ważne:** Używaj `&` i `|` dla logiki warunkowej (`if`, `branch_if`). Operatory `&&` i `||` nie istnieją w CurlyWASM. Stosowanie operatorów bitowych do logiki jest powszechną techniką w sizecodingu.
* **Porównania:** `==`, `!=`, `<`, `<=`, `>`, `>=` (signed), `#<`, `#<=`, `#>`, `#>=` (unsigned).
* **Przypisanie z użyciem wartości:** `:=` przypisuje wartość do zmiennej *lokalnej* i jednocześnie zwraca tę wartość. Jest to bardzo użyteczne w wyrażeniach i pętlach, np. `branch_if (i +:= 1): loop_label;`. Często kompiluje się do efektywnej instrukcji WASM `local.tee`.
* **Sequencing operator:** `<|` wykonuje wyrażenie po lewej stronie (dla jego efektów ubocznych) i zwraca wartość wyrażenia po prawej stronie, np. `efekt_uboczny <| wartość_zwracana`. Pomaga minimalizować narzut stosu.
* **Rzutowanie typów:** `as` (np. `mojInt as f32`). Dozwolone w większości kontekstów poza deklaracjami stałych `const`.

#### 2.6. Funkcje

Funkcje deklaruje się słowem `fn`. Ciało to blok `{}`.

* **Implicit return:** Ostatnie wyrażenie w bloku funkcji jest zwracane automatycznie. Nie używa się słowa kluczowego `return` (ono nie istnieje). Funkcje bez zwracanej wartości (lub zwracające pusty typ `()`) kończą się instrukcją zakończoną średnikiem lub pustym blokiem.
* **Parametry:** Lista `nazwa: typ` w nawiasach.
* **Typ zwracany:** Opcjonalnie po `->`.
    ```curlywasm
    fn dodaj(a: i32, b: i32) -> i32 {
      a + b // Zwracana wartość (bez średnika)
    }

    export fn rysujCos() {
      cls(0); // Instrukcja zakończona średnikiem, brak zwrotu
    }
    ```

#### 2.7. Control flow

CurlyWASM wspiera strukturalne instrukcje WASM:

* **If/Else:** `if warunek { blok } else { blok }`. Może być wyrażeniem zwracającym wartość, o ile obie gałęzie zwracają ten sam typ.
* **Block:** `block nazwa { ... }`. Definiuje blok kodu. Wyskok na koniec bloku (po zamykającej klamrze `}`) wykonuje się instrukcją `branch nazwa` lub `branch_if warunek: nazwa`.
* **Loop:** `loop nazwa { ... }`. Definiuje pętlę. Wyskok na początek pętli (rozpoczynający następna iterację) wykonuje się instrukcją `branch nazwa` lub `branch_if warunek: nazwa`.
* **Ważne:** W CurlyWASM **nie ma** słów kluczowych `break` ani `continue`. Wyjście ze środka pętli do miejsca *po* pętli realizuje się przez użycie `branch` lub `branch_if` ze wskazaniem *etykiety bloku* (`block label`), która obejmuje pętlę, lub po prostu przez "przepadnięcie" poza `branch_if` warunkujące powrót na początek pętli.
* **Labels:** Etykiety w `block label` i `loop label` służą jako cele dla instrukcji `branch label` i `branch_if warunek: label`. Etykieta na osobnej linii, będąca celem skoku *po* strukturze `block` lub `loop`, jest rzadko używana i musi być zdefiniowana *bez* dwukropka (np. `my_label`, nie `my_label:`), ale model programowania WASM/CurlyWASM naturalnie prowadzi do użycia `branch` do skoku na koniec bloku lub początek pętli, a nie do zewnętrznych, arbitralnych etykiet.

#### 2.8. Pamięć (Load/Store) i Bezpośredni Dostęp

Pamięć MicroW8 jest liniowa, adresy są bajtowe, zaczynając od adresu 0. Całkowity rozmiar pamięci to 256 KB.

* **Bezpośredni odczyt:** CurlyWASM udostępnia składnię `baza?offset` (odczyt bajtu), `baza!offset` (odczyt 32-bitowego słowa `i32`) i `baza$offset` (odczyt 32-bitowej liczby zmiennoprzecinkowej `f32`). `baza` to wyrażenie typu `i32` (adres bazowy), a `offset` to stała `i32` (przesunięcie od adresu bazowego). Ta składnia jest preferowana w pętlach przetwarzających duże ilości danych (jak framebuffer) ze względu na minimalny narzut i generowanie efektywnych instrukcji WASM `load`.
    ```curlywasm
    // Odczyt koloru piksela (bajt) bezpośrednio z framebuffera
    let offset_piksela = y * 320 + x;
    let kolor = (FRAMEBUFFER + offset_piksela)?0; // Odczyt bajtu spod adresu FRAMEBUFFER + offset_piksela + 0

    // Odczyt czasu w ms (słowo i32) z rejestru sprzętowego
    let czas_ms = (TIME_MS)!0; // Odczyt słowa spod adresu TIME_MS + 0
    ```
* **Bezpośredni zapis:** Analogicznie do odczytu, używamy składni `baza?offset = wartość`, `baza!offset = wartość`, `baza$offset = wartość`.
    ```curlywasm
    // Zapis koloru piksela (bajt) bezpośrednio do framebuffera
    let offset_piksela = y * 320 + x;
    (FRAMEBUFFER + offset_piksela)?0 = 15; // Zapis bajtu pod adres FRAMEBUFFER + offset_piksela + 0

    // Zapis wartości i32 (np. nowej wartości TIME_MS, jeśli chcemy zmodyfikować licznik)
    (TIME_MS)!0 = nowy_czas_ms; // Zapis słowa i32 pod adres TIME_MS + 0
    ```
* **Alignment:** Operacje na 32-bitowych słowach (`!`, `$`, `i32.load`, `f32.load`, `i32.store`, `f32.store`) wymagają, aby efektywny adres pamięci (baza + offset) był wyrównany do 4 bajtów (czyli podzielny przez 4). Operacje na bajtach (`?`, `i32.load8_u/s`, `i32.store8`) nie mają takich wymagań. Należy o tym pamiętać przy projektowaniu struktur danych w pamięci.
* **Intrinsics:** Możesz również używać bezpośrednio nazw instrukcji WASM do operacji na pamięci, np. `i32.load8_u(adres, offset)`, `i32.store(wartość, adres, offset)`. Składnia `baza?offset` itp. jest syntactic sugar dla tych instrukcji z offsetem równym 0.

#### 2.9. Sekcje danych (`data`)

Sekcje danych służą do inicjalizacji obszarów pamięci przy starcie modułu. Dane w nich zdefiniowane są ładowane pod określony adres w pamięci RAM konsoli.

```curlywasm
data 0x14000 { // Adres w pamięci, np. początek USER_MEM
  i8(10, 20, 30) // Blok bajtów. Wartości oddzielone przecinkami.
  i32(0x12345678) // Blok słów 32-bitowych.
  f32(3.14) // Blok floatów 32-bitowych.
  "Tekst\0" // Ciąg znaków zakończony zerem.
  file("moje_dane.bin") // Zawartość zewnętrznego pliku binarnego.
}
// Ważne: Bloki danych różnych typów (i8(...), i32(...), "...", file(...)) umieszczamy jeden po drugim w sekcji data { ... }, BEZ PRZECINKÓW między blokami.
```

### 3. API MicroW8

MicroW8 udostępnia zestaw funkcji API w module `"env"`, które należy zaimportować w sekcji `import`. CurlyWASM udostępnia również wbudowane (built-in) funkcje matematyczne i bitowe, które są dostępne bezpośrednio i **nie wymagają importu**. Próba ich zaimportowania spowoduje błąd kompilacji.

#### 3.1. Grafika

Ekran: 320x240, 8 bpp (8 bitów na piksel, index do palety). Funkcje przyjmujące parametry `f32` (np. `line`, `circle`, `rectangle`) mają subpikselową dokładność.

* `cls(color: i32)`: Czyści cały ekran do podanego indeksu koloru, resetuje kursor tekstowy do pozycji (0,0), wyłącza graphics mode dla tekstu.
* `setPixel(x: i32, y: i32, color: i32)`: Ustawia kolor piksela o współrzędnych `(x, y)` na `color`. Dla optymalizacji w pętlach przetwarzających wiele pikseli, preferowany jest bezpośredni dostęp do pamięci framebuffera (patrz sekcja 2.8).
* `getPixel(x: i32, y: i32) -> i32`: Odczytuje indeks koloru piksela o współrzędnych `(x, y)`. Zwraca 0, jeśli współrzędne są poza ekranem.
* `hline(left: i32, right: i32, y: i32, color: i32)`: Rysuje poziomą linię od `left` do `right` (exclusive) na wysokości `y` kolorem `color`.
* `line(x1: f32, y1: f32, x2: f32, y2: f32, color: i32)`: Rysuje linię między punktami `(x1, y1)` i `(x2, y2)` kolorem `color`.
* `rectangle(x: f32, y: f32, w: f32, h: f32, color: i32)`: Wypełnia prostokąt o lewym górnym rogu `(x, y)`, szerokości `w` i wysokości `h` kolorem `color`.
* `circle(cx: f32, cy: f32, radius: f32, color: i32)`: Wypełnia koło o środku `(cx, cy)` i promieniu `radius` kolorem `color`.
* `rectangleOutline(x: f32, y: f32, w: f32, h: f32, color: i32)`: Rysuje obrys prostokąta.
* `circleOutline(cx: f32, cy: f32, radius: f32, color: i32)`: Rysuje obrys koła.
* `blitSprite(spriteData: i32, size: i32, x: i32, y: i32, control: i32)`: Kopiuje dane pikseli ze wskazanego adresu w pamięci (`spriteData`) na ekran w pozycji `(x, y)`. Parametr `size` koduje szerokość i wysokość (`width | (height << 16)`). `control` pozwala na maskowanie kolorem i flipping (odbicie w poziomie/pionie).
* `grabSprite(spriteData: i32, size: i32, x: i32, y: i32, control: i32)`: Kopiuje dane pikseli z ekranu ze wskazanej pozycji `(x, y)` do pamięci pod adres `spriteData`. Parametry jak w `blitSprite`.

#### 3.2. Gamepad

Gamepad: Wirtualny gamepad z D-Padem i 4 przyciskami (A, B, X, Y). Stan przycisków jest mapowany na klawisze klawiatury (np. Up -> Arrow-Up, A -> Z). Stan przycisków można sprawdzić za pomocą funkcji API lub odczytując bezpośrednio bitfield w pamięci pod adresem `GAMEPAD` (0x44).

* `isButtonPressed(btn: i32) -> i32`: Zwraca niezerową wartość, jeśli przycisk o indeksie `btn` jest aktualnie wciśnięty.
* `isButtonTriggered(btn: i32) -> i32`: Zwraca niezerową wartość, jeśli przycisk o indeksie `btn` został wciśnięty w tej klatce (zdarzenie).
* **Przykład bezpośredniego odczytu stanu przycisków z pamięci:**
    Stan gamepada (`GAMEPAD`, 0x44) to 32-bitowe słowo (i32), gdzie każdy bit odpowiada jednemu przyciskowi. Można go odczytać za pomocą operatora `!`:
    ```curlywasm
    let gamepad_state = (GAMEPAD)!0;
    let is_button_a_pressed = (gamepad_state >> BUTTON_A) & 1; // Sprawdzenie bitu dla przycisku A
    let is_button_up_pressed = (gamepad_state >> BUTTON_UP) & 1; // Sprawdzenie bitu dla przycisku Up
    ```
    Indeksy przycisków są zdefiniowane jako stałe (BUTTON_UP, BUTTON_DOWN, etc.). Bezpośredni odczyt jest często szybszy i generuje mniejszy kod niż wielokrotne wywołania `isButtonPressed`, jeśli sprawdzasz kilka przycisków.

#### 3.3. Ekran i tekst

Drukowanie tekstu wykorzystuje wbudowany font (adres `FONT`, 0x13400), który można też podmienić własnym. Dostępne są dwa tryby: normalny (siatka 8x8, domyślny po `cls()`) i graphics mode (położenie kursora w pikselach, tło przezroczyste).

* `printChar(char: i32)`: Drukuje pojedynczy znak (dolny 8 bitów `char`). Obsługuje też control characters (0-31).
* `printString(ptr: i32)`: Drukuje ciąg znaków zakończony zerem (`\0`) wskazywany przez adres `ptr`.
* `printInt(num: i32)`: Drukuje liczbę całkowitą `num` jako tekst dziesiętny.
* `setTextColor(color: i32)`: Ustawia indeks koloru dla drukowanego tekstu.
* `setBackgroundColor(color: i32)`: Ustawia indeks koloru tła dla drukowanego tekstu (tylko w normalnym trybie tekstowym).
* `setCursorPosition(x: i32, y: i32)`: Ustawia pozycję kursora. W normalnym trybie `(x, y)` to współrzędne w siatce znaków (0-39, 0-29), w graphics mode to współrzędne w pikselach (0-319, 0-239).

#### 3.4. Dźwięk

MicroW8 może generować dźwięk na dwa sposoby: przez własną funkcję `export fn snd(sampleIndex: i32) -> f32` (bardziej zaawansowane, niskopoziomowe) lub przez wbudowany syntezator `sndGes`, sterowany przez 32 bajty pamięci od adresu 0x50.

* `playNote(channel: i32, note: i32)`: Wygodna funkcja API do wyzwalania nut (1-127, 69=A4) na jednym z 4 kanałów (0-3). Nuta 0 zatrzymuje dźwięk na kanale, 128-255 wyzwala atak i wybrzmienie bez podtrzymania. Ta funkcja modyfikuje odpowiednie bajty w rejestrach `sndGes` (0x50-0x70).
* `sndGes(sampleIndex: i32) -> f32`: Funkcja implementująca 4-kanałowy syntezator sterowany rejestrami w pamięci 0x50-0x70. Jeśli nie zdefiniujesz własnej funkcji `snd()`, MicroW8 będzie wywoływać `sndGes` dwukrotnie na próbkę (lewy i prawy kanał) z częstotliwością 44100 Hz, kopiując do pamięci dźwięku 32 bajty z adresu 0x50 z pamięci głównego wątku po każdym wywołaniu `upd()`.
* **Bezpośrednie sterowanie syntezatorem `sndGes`:**
    Pełną kontrolę nad dźwiękiem daje bezpośrednia modyfikacja 32 bajtów od adresu 0x50. Przykładowo, aby ustawić typ fali (prostokątna=0, piła=1, trójkąt=2, szum=3) i nutę na kanale 0, można zapisać bajty:
    ```curlywasm
    // Ustaw falę (bits 7-6) i flagę note on (bit 0) na kanale 0
    // Adres 0x50 + offset kanału (0*6) + offset rejestru CTRL (0) = 0x50
    let wave_type = 2; // Trójkątna
    let note_on_flag = 1;
    (0x50)?0 = (wave_type << 6) | note_on_flag;
    // Ustaw nutę (69=A4) na kanale 0
    // Adres 0x50 + offset kanału (0*6) + offset rejestru NOTE (3) = 0x53
    (0x53)?0 = 69;
    ```
    Szczegółowy opis rejestrów `sndGes` znajduje się w dokumentacji `microw8-api.txt`.

#### 3.5. Funkcje matematyczne

Operacje na `f32`, kąty w radianach.

* **Importowane z `"env"`:** (Wymagają importu w sekcji `import`)
    * `sin`, `cos`, `tan`, `asin`, `acos`, `atan`, `atan2`, `exp`, `log`, `pow`, `fmod`.
* **Built-in w CurlyWASM:** (Dostępne bezpośrednio, **nie importujemy ich!**)
    * `sqrt`, `min`, `max`, `ceil`, `floor`, `trunc`, `nearest`, `abs`, `copysign`, `select`.

#### 3.6. Losowość

MicroW8 udostępnia generator liczb pseudolosowych oparty na algorytmie xorshift64\*. Jest inicjowany stałym ziarnem przy starcie, chyba że użyjesz `randomSeed`.

* `random() -> i32`: Zwraca 32-bitową liczbę całkowitą pseudolosową.
* `randomf() -> f32`: Zwraca liczbę zmiennoprzecinkową pseudolosową z zakresu $[0.0, 1.0)$.
* `randomSeed(seed: i32)`: Inicjuje generator liczb pseudolosowych podanym ziarnem. W `platform.txt` zobaczysz implementację `randomSeed` i `random`, pokazującą, jak działa xorshift64\*.

#### 3.7. Czas

* `time() -> f32`: Zwraca czas w sekundach od startu modułu. Jest to czas w wątku głównym.
* `TIME_MS` (0x40): Adres w pamięci zawierający 32-bitową (`i32`) wartość czasu w milisekundach od startu modułu. Ten rejestr jest inkrementowany przez runtime MicroW8 w każdym wywołaniu `upd()`. Jest to rejestr R/W, można go odczytywać (`(TIME_MS)!0`) i zapisywać, co jest kluczowe do manualnego pomiaru delta time i FPS.

#### 3.8.Operacje na pamięci (Intrinsics)

CurlyWASM umożliwia użycie niskopoziomowych instrukcji WebAssembly do operacji na pamięci. Są one często generowane przez operatory `?`, `!`, `$`, ale można ich też używać bezpośrednio.

* `i32.load(base, offset)`: Odczytuje 32-bitowe słowo `i32` z pamięci spod adresu `base + offset`. Wymaga wyrównania do 4 bajtów.
* `i32.load8_u(base, offset)`, `i32.load8_s(base, offset)`: Odczytują 8-bitowy bajt jako unsigned/signed `i32`. Nie wymagają wyrównania.
* `i32.load16_u(base, offset)`, `i32.load16_s(base, offset)`: Odczytują 16-bitowe słowo jako unsigned/signed `i32`. Wymagają wyrównania do 2 bajtów.
* `i32.store(value, base, offset)`: Zapisuje 32-bitowe słowo `value` do pamięci spod adresu `base + offset`. Wymaga wyrównania do 4 bajtów.
* `i32.store8(value, base, offset)`: Zapisuje dolny 8 bitów `value` jako bajt. Nie wymaga wyrównania.
* `i32.store16(value, base, offset)`: Zapisuje dolne 16 bitów `value` jako 16-bitowe słowo. Wymaga wyrównania do 2 bajtów.
* `f32.load(base, offset)`: Odczytuje 32-bitowy float `f32`. Wymaga wyrównania do 4 bajtów.
* `f32.store(value, base, offset)`: Zapisuje 32-bitowy float `value`. Wymaga wyrównania do 4 bajtów.
* `memory.size() -> i32`: Zwraca aktualny rozmiar pamięci w jednostkach 'page' (1 page = 64KB).
* `memory.grow(delta_pages) -> i32`: Zwiększa rozmiar pamięci o `delta_pages`. Zwraca poprzedni rozmiar pamięci w stronach lub -1 w przypadku błędu. MicroW8 ma stały rozmiar pamięci (4 strony = 256KB), więc ta funkcja może nie być użyteczna w praktyce.
* `memory.fill(dest, value, size)`: Wypełnia obszar pamięci od `dest` o rozmiarze `size` bajtami o wartości `value`. Bardzo szybkie do czyszczenia buforów.
* `memory.copy(dest, src, size)`: Kopiuje blok pamięci o rozmiarze `size` z adresu `src` pod adres `dest`. Niezastąpione do double bufferingu.

#### 3.9. Inne funkcje wbudowane

CurlyWASM udostępnia również inne wbudowane funkcje (intrinsics) do operacji na liczbach całkowitych, **których nie importujemy**:

* `i32.clz(value: i32) -> i32`: Count Leading Zeros. Liczba zer binarnych od najbardziej znaczącego bitu.
* `i32.ctz(value: i32) -> i32`: Count Trailing Zeros. Liczba zer binarnych od najmniej znaczącego bitu.
* `i32.popcnt(value: i32) -> i32`: Population Count. Liczba ustawionych bitów (jednostek).
* `i32.bswap(value: i32) -> i32`: Byte Swap. Zamienia kolejność bajtów w 32-bitowym słowie (przydatne przy pracy z różnymi endianness).
* `i32.trunc_sat_f32_u(value: f32) -> i32`, `i32.trunc_sat_f32_s(value: f32) -> i32`: Konwertują `f32` do `i32` (unsigned/signed) z zaokrągleniem do zera, saturując wyniki do zakresu `i32`.

### 4. Mapa pamięci

Znajomość mapy pamięci jest kluczowa w sizecodingu, ponieważ pozwala na bezpośredni dostęp do ważnych rejestrów i obszarów danych.

| Adres         | Rozmiar         | Opis                                         | Stała CurlyWASM      |
| :------------ | :-------------- | :------------------------------------------- | :------------------- |
| `00000-00040` | 64 bajty        | Pamięć użytkownika (mały obszar)            | -                    |
| `00040-00044` | 4 bajty         | Czas od startu w ms (i32)                    | `TIME_MS`            |
| `00044-0004c` | 8 bajtów        | Stan gamepada (`GAMEPAD`, `GAMEPAD + 4` dla stanu poprzedniej klatki) | `GAMEPAD`            |
| `0004c-00050` | 4 bajty         | Liczba klatek od startu (i32)                | -                    |
| `00050-00070` | 32 bajty        | Rejestry sndGes (`sndGes` registers)         | -                    |
| `00070-00078` | 8 bajtów        | Zarezerwowane                                | -                    |
| `00078-12c78` | 76800 bajtów    | Framebuffer (320x240, 8bpp)                  | `FRAMEBUFFER`        |
| `12c78-12c7c` | 4 bajty         | Adres pamięci roboczej sndGes (domyślnie 0x50) | -                    |
| `12c7c-13000` | 836 bajtów      | Zarezerwowane                                | -                    |
| `13000-13400` | 1024 bajty      | Paleta kolorów (256x32bpp, RGBA)             | `PALETTE`            |
| `13400-13c00` | 2048 bajtów     | Font (256x8x8, 1bpp)                         | `FONT`               |
| `13c00-14000` | 1024 bajty      | Zarezerwowane                                | -                    |
| `14000-40000` | 172032 bajty    | Pamięć użytkownika (duży obszar)             | `USER_MEM`           |

Pamięć od 0x14000 do 0x40000 jest głównym obszarem dostępnym dla Twoich danych: zmiennych globalnych, tablic, buforów na grafikę (sprite'y), danych dźwiękowych, dodatkowych buforów do double buffering itp. Efektywny rozmiar tej pamięci to $0x40000 - 0x14000 = 262144 - 81920 = 180224$ bajty.

### 5. Workflow

Narzędzie `uw8` to Twój główny towarzysz w pracy z MicroW8. Wspiera:

* `uw8 compile <input.cwa> <output.wasm>`: Kompilacja kodu CurlyWASM do binarnego modułu WebAssembly.
* `uw8 run <file.wasm|.uw8|.cwa>`: Uruchamianie modułu WASM lub `.uw8` cart, lub bezpośrednio pliku CurlyWASM. Opcja `-w`, `--watch` jest bardzo przydatna do automatycznego przeładowywania cartridża po zmianie pliku.
* `uw8 pack <infile> <outfile>`: Pakowanie modułu WASM lub pliku źródłowego do formatu `.uw8`.
* `uw8 unpack <infile> <outfile>`: Rozpakowywanie `.uw8` cart do modułu WASM.
* `uw8 filter-exports <infile> <outfile>`: Narzędzie do optymalizacji rozmiaru, usuwające nieużywane eksporty i martwy kod z modułu WASM.

### 6. Sizecoding i podstawowa optymalizacja

Sizecoding na MicroW8 polega na minimalizacji rozmiaru wygenerowanego modułu WASM. CurlyWASM wspiera to przez:

* **Low-level instrukcje WASM:** CurlyWASM kompiluje się blisko instrukcji WASM, dając programiście dużą kontrolę.
* **Modyfikatory `let inline`/`lazy`:** Pozwalają kontrolować, kiedy i jak obliczane są wartości lokalnych zmiennych, wpływając na rozmiar i wydajność kodu.
    * `let lazy nazwa = wyrażenie;`: Wyrażenie obliczane tylko raz, przy pierwszym użyciu. Może oszczędzić rozmiar kodu, jeśli wartość jest użyta wielokrotnie, ale tylko w jednej ścieżce wykonania.
    * `let inline nazwa = wyrażenie;`: Wyrażenie jest wstawiane w miejsce użycia, jak makro. Może zwiększyć rozmiar kodu, ale eliminuje narzut związany z lokalnymi zmiennymi. Przydatne dla dynamicznych wartości, które muszą być przeliczane w każdej iteracji.
* **Operator `:=`:** Jak omówiono w sekcji 2.5, jest kluczowy w pętlach i wyrażeniach warunkowych do łączenia przypisania i użycia wartości w jednej, efektywnej instrukcji. Przykład: `branch_if (i +:= 1) < MAX_ITER: loop_label;`.
* **Bezpośredni dostęp do pamięci (`baza?offset`, `baza!offset`, `baza$offset`):** Znacznie szybszy i mniejszy niż wywołania funkcji API (np. `setPixel`, `getPixel`) w pętlach operujących na pikselach. Minimalizuje narzut wywołania funkcji i pozwala na bezpośrednią manipulację danymi w pamięci. Przykładowo, zamiast `setPixel(x, y, color);` w pętli renderingu, użyj `(FRAMEBUFFER + y * 320 + x)?0 = color;`.
* **Bitwise operations:** Efektywne wykorzystanie operatorów bitowych (`&`, `|`, `^`, `<<`, `>>`, `#>>`) do pakowania danych, szybkiej matematyki (np. $(x << 3)$ to $x \times 8$) i generowania wzorców może znacząco zmniejszyć rozmiar kodu.
* **Look-up Tables (LUTs):** Prekomputowanie wartości funkcji matematycznych (np. sinusów, cosinusów) i przechowywanie ich w pamięci (`data` section lub inicjalizacja w `start()`) pozwala uniknąć kosztownych obliczeń w czasie rzeczywistym w pętli `upd()`. Odczyt z pamięci jest zazwyczaj szybszy i generuje mniej kodu niż wywołanie funkcji typu `sin()`.
    **Przykład inicjalizacji LUT w `start()`:**
    ```curlywasm
    const SIN_TABLE_SIZE = 256;
    const SIN_TABLE = USER_MEM; // Przechowamy tabelę na początku USER_MEM

    export fn start() {
      let i = 0;
      let pi = 3.14159265;
      loop fill_loop {
        let angle_rad = (i as f32 / SIN_TABLE_SIZE as f32) * pi * 2.0; // Kąt w radianach (0 do 2*PI)
        let sin_val_f = sin(angle_rad);
        let sin_val_i8 = (sin_val_f * 127.0 + 128.0) as i32; // Przeskaluj do zakresu 0-255
        (SIN_TABLE + i)?0 = sin_val_i8; // Zapisz jako bajt w tabeli
        branch_if (i := i + 1) < SIN_TABLE_SIZE: fill_loop;
      }
      // ... reszta inicjalizacji ...
    }

    // Użycie LUT w funkcji upd():
    /*
    fn upd() {
      let angle_index = ( (time() * jakas_predkosc_f) as i32 ) & (SIN_TABLE_SIZE - 1); // Oblicz index (z zawijaniem)
      let sin_value_i8 = (SIN_TABLE + angle_index)?0; // Odczytaj wartość z tabeli (0-255)
      let sin_value_f = (sin_value_i8 as f32 - 128.0) / 127.0; // Przeskaluj z powrotem do -1.0 .. 1.0
      // Użyj sin_value_f do dalszych obliczeń...
    }
    */
    ```

### 7. Pomiar FPS

MicroW8 nie ma V-sync ani wbudowanego licznika FPS, który byś mógł bezpośrednio odczytać. Aby zmierzyć aktualną liczbę klatek na sekundę (FPS):

1.  Odczytaj aktualną wartość `TIME_MS` (adres `0x40`, czas w ms) w dowolnym momencie funkcji `upd()` i zapisz ją. Adres `0x3c` (tuż przed `TIME_MS` w małym obszarze pamięci użytkownika) jest często używany do tego celu w sizecodingu, aby uniknąć deklarowania globalnej zmiennej.
2.  Odczytaj `TIME_MS` ponownie podczas *kolejnego* wywołania `upd()`.
3.  Oblicz różnicę (`delta`) w milisekundach pomiędzy świeżo odczytaną wartością `TIME_MS` a tą zapisaną w poprzedniej klatce.
4.  Przybliżony FPS to $1000 / \text{delta}$.

**Uwaga:** Dla pustej funkcji `upd()` MicroW8 zazwyczaj osiąga ~60 FPS (delta ≈ 16.67 ms), zakładając brak znaczących opóźnień w runtime’ie przeglądarki lub systemu operacyjnego. Zobacz praktyczny przykład implementacji w sekcji 2.4 (funkcja `printFps()`), która wyświetla FPS i ostrzega, gdy spada poniżej 60.

### 8. Przykład "Hello, World!"

Ten prosty przykład pokazuje podstawową strukturę modułu CurlyWASM, importowanie funkcji API i sekcję danych.

```curlywasm
// Importujemy funkcje API potrzebne do rysowania i tekstu
import "env.memory" memory(4); // Importujemy pamięć
import "env.cls" fn cls(i32); // Import funkcji cls do czyszczenia ekranu
import "env.printString" fn printString(i32); // Import funkcji do drukowania ciągów
import "env.setTextColor" fn setTextColor(i32); // Import funkcji do ustawiania koloru tekstu
import "env.setCursorPosition" fn setCursorPosition(i32, i32); // Import funkcji do ustawiania kursora

// Definiujemy stałą dla adresu pamięci użytkownika, gdzie umieścimy tekst
const USER_MEM = 0x14000; // Adres początku większego obszaru pamięci użytkownika

// Sekcja danych - umieszczamy ciąg znaków "Witaj, świecie!" i zero na końcu (terminator)
data USER_MEM {
  "Witaj, swiecie!" // Ciąg znaków do wyświetlenia
  i8(0) // Bajt zerowy - terminator ciągu
}

// Funkcja aktualizacji, wywoływana co klatkę
export fn upd() {
  cls(0); // Czyścimy ekran kolorem 0 (czarny)
  setTextColor(15); // Ustawiamy kolor tekstu na 15 (domyślnie biały w standardowej palecie)
  setCursorPosition(5, 5); // Ustawiamy pozycję kursora tekstowego (w siatce znaków 8x8) na kolumnę 5, wiersz 5
  printString(USER_MEM); // Drukujemy ciąg znaków z adresu USER_MEM
}

// Opcjonalna funkcja startowa, wywoływana raz przy ładowaniu modułu
export fn start() {
  // Można tutaj umieścić kod inicjalizacyjny, np. randomSeed(), inicjalizację LUTs itp.
  // W tym prostym przykładzie nie jest potrzebna.
}
```

Ten kod czyści ekran, ustawia kolor tekstu i pozycję kursora, a następnie wyświetla ciąg znaków "Witaj, świecie!", odczytując go z sekcji danych w pamięci.

### 9. Przykłady z plików

Analiza kodu niektó©ych przykładów z ktalogu `src` jest nieocenionym źródłem wiedzy o praktycznych technikach w CurlyWASM i MicroW8. Zachęcam do studiowania ich kodu:

* `control.cwa`: Demonstracja control characters do formatowania tekstu.
* `scaled_text.cwa`: Użycie graphics mode i kontrolera 30 do rysowania skalowalnego tekstu.
* `font_palette.cwa`: Manipulacja fontem i paletą kolorów, przełączanie między nimi.
* `Chaos_Rose.cwa`, `encounter.cwa`, `urban_drift.cwa`, `plasma_chars.cwa`, `plasma_sizecoding_1.cwa`, `sierpinski_triangle.cwa`, `mandelbrot.cwa`, `interference.cwa`: Przykłady klasycznych efektów demoscenowych (fraktale, plazma, raymarching, efekty proceduralne) i technik ich implementacji, często z użyciem optymalizacji sizecodingowych.
* `cube_wireframe.cwa`: Renderowanie prostego modelu 3D (sześcianu) z użyciem projekcji i rysowania linii. Dobry przykład użycia matematyki i manipulacji zmiennymi globalnymi.
* `game_of_life.cwa`: Implementacja gry w życie, demonstrująca użycie double buffering i efektywnych operacji na pamięci z zawijaniem (wrap-around).

---

## Część 2: Zaawansowane Techniki MicroW8

Teraz, gdy znasz podstawy, czas zagłębić się w bardziej zaawansowane techniki demoscenowe na MicroW8. W tej części skupimy się na optymalizacji kodu, zaawansowanym zarządzaniu pamięcią (w tym double buffering), implementacji klasycznych efektów demoscenowych, precyzyjnym timingu opartym na delta time oraz generowaniu dźwięku za pomocą `sndGes`. To wyzwanie dla prawdziwych sizecoderów!

### 1. CurlyWASM i Sizecoding: Sztuka Kompaktowego Kodu

W MicroW8 każdy bajt ma znaczenie. CurlyWASM daje narzędzia do precyzyjnego sterowania rozmiarem kodu poprzez kompilację bliską instrukcjom WASM.

* **Modyfikatory `let` (`lazy`, `inline`):** Jak omówiono w Części 1, pozwalają optymalizować użycie zmiennych lokalnych i generowany kod w zależności od tego, czy wartość jest używana wielokrotnie (`lazy`) czy jako jednorazowe zastępstwo wyrażenia (`inline`).
    * `let lazy nazwa = wyrażenie;`: Używa `local.tee` pod spodem, oblicza wyrażenie raz i przypisuje je do zmiennej lokalnej, zwracając wartość. Oszczędza bajty, jeśli wyrażenie jest złożone, a wartość używana wielokrotnie, ale uważaj na nieużywane gałęzie kodu, które mogą niepotrzebnie obliczać wartość.
    * `let inline nazwa = wyrażenie;`: Działa jak makro, wstawiając wyrażenie w miejsce użycia zmiennej. Przydatne dla dynamicznych wartości, ale może zwiększyć rozmiar kodu, jeśli zmienna jest użyta wielokrotnie.
* **Operator `:=`:** Niezastąpiony w sizecodingu. Pozwala na obliczenie wartości, przypisanie jej do *zmiennej lokalnej* i jednoczesne użycie tej wartości w wyrażeniu. Najczęstsze zastosowanie to pętle z `branch_if`, np. `branch_if (i +:= 1) < MAX_ITER: loop_label;`. Kompilator CurlyWASM często generuje dla tego operatora instrukcję `local.tee`, co jest bardzo efektywne.
* **Sequencing operator `<|`:** Wykonuje efekt uboczny (lewa strona) i zwraca wartość (prawa strona). Minimalizuje narzut stosu.
* **Bezpośredni dostęp do pamięci (`baza?offset`, `baza!offset`, `baza$offset`):** Kluczowy w sizecodingu, bo pozwala pominąć narzut wywołań funkcji API w pętlach przetwarzających duże ilości danych (np. framebuffer). Jest to zazwyczaj szybsze i generuje mniej kodu niż wywołanie funkcji typu `setPixel` czy `getPixel`. Przykład z `plasma_sizecoding_1.cwa` pokazuje, jak minimalizować zmienne lokalne i pracować z direct memory access w pętli rysującej piksele, redukując rozmiar kodu.

### 2. Zaawansowane zarządzanie pamięcią i Double Buffering

Efektywne wykorzystanie dostępnych 256 KB pamięci jest podstawą zaawansowanego programowania na MicroW8.

* **Framebuffer (`FRAMEBUFFER`, 0x78):** 320x240 pikseli, 8 bpp (76800 bajtów). Adres piksela (x, y) oblicza się jako `FRAMEBUFFER + y * 320 + x`. Używaj operatora `?` do zapisu/odczytu pojedynczych bajtów (indeksów kolorów).
    ```curlywasm
    let pixel_x = 10;
    let pixel_y = 20;
    let kolor_indeks = 12;
    let offset = pixel_y * 320 + pixel_x;
    (FRAMEBUFFER + offset)?0 = kolor_indeks; // Zapis bajtu koloru
    let odczytany_kolor = (FRAMEBUFFER + offset)?0; // Odczyt bajtu koloru
    ```
* **Paleta (`PALETTE`, 0x13000):** 256 kolorów, każdy po 4 bajty (RGBA), łącznie 1024 bajty. Pamiętaj, że do zapisu 32-bitowych wartości RGBA potrzebujesz wyrównania do 4 bajtów. Używaj operatora `!` dla odczytu/zapisu 32-bitowych słów lub funkcji `i32.load`/`i32.store`. Przykład: `font_palette.cwa` demonstruje manipulację paletą.
    ```curlywasm
    // Zapis koloru (RGBA) pod index 10 w palecie
    let kolor_rgba = 0xff0000ff; // Niebieski (AA RR GG BB w little-endian, czyli BB GG RR AA w pamięci)
    let paleta_offset = 10 * 4; // 4 bajty na kolor
    (PALETTE + paleta_offset)!0 = kolor_rgba; // Zapis 32-bitowego słowa
    ```
* **Pamięć użytkownika (`USER_MEM`, 0x14000-0x40000):** Duży obszar (ok. 180 KB) dostępny na Twoje dane. Możesz go wykorzystać na tablice, struktury danych, sprite'y, fonty, dane dźwiękowe, a także jako **back buffer** do double bufferingu.
* **Double buffering:** Technika polegająca na rysowaniu kolejnej klatki do bufora w pamięci RAM (np. w obszarze `USER_MEM`) zamiast bezpośrednio do `FRAMEBUFFER`. Po zakończeniu rysowania, zawartość tego bufora jest kopiowana do `FRAMEBUFFER`. Zapobiega to tearingowi i flickeringowi, zapewniając płynne przejścia między klatkami. Przykład `game_of_life.cwa` używa dwóch buforów (FRAMEBUFFER i BACKBUFFER w USER_MEM) i kopiuje między nimi. Kopiowanie całego bufora (76800 bajtów) można zrealizować bardzo efektywnie w MicroW8 za pomocą instrukcji `memory.copy`.
    **Przykład implementacji double buffering:**
    1.  Zarezerwuj obszar w `USER_MEM` na back buffer, np. `const BACKBUFFER = USER_MEM;`. Upewnij się, że jest wystarczająco duży (320x240 bajtów).
    2.  W funkcji `upd()`:
        a.  Rysuj wszystkie elementy sceny do `BACKBUFFER`. Zamiast `setPixel(x, y, c)` używaj np. `set(BACKBUFFER, x, y, c)` (funkcja pomocnicza używająca `(BACKBUFFER + y * 320 + x)?0 = c;`) lub bezpośredniego zapisu.
        b.  Po zakończeniu rysowania, skopiuj `BACKBUFFER` do `FRAMEBUFFER`:
        ```curlywasm
        // Skopiuj całą zawartość BACKBUFFER (320*240 bajtów) do FRAMEBUFFER
        memory.copy(FRAMEBUFFER, BACKBUFFER, 320 * 240);
        ```
        Ta pojedyncza operacja `memory.copy` jest znacznie szybsza niż rysowanie pojedynczych pikseli do framebuffera.

### 3. Implementacja efektów demoscenowych

Przykłady dostarczone z MicroW8 demonstrują implementację wielu klasycznych efektów demoscenowych, które warto studiować i adaptować:

* **Plasma:** Generowanie hipnotyzujących, organicznych wzorów za pomocą kombinacji funkcji sinus o różnych częstotliwościach i fazach. `plasma_sizecoding_1.cwa` pokazuje wersję pikselową, a `plasma_chars.cwa` wersję tekstową z użyciem własnego fontu i LUT na sinusy.
* **Fraktale:** Rysowanie zbiorów generowanych przez iteracyjne wzory matematyczne, np. Zbiór Mandelbrota ($z_{n+1} = z_n^2 + c$). `mandelbrot.cwa` demonstruje obliczanie Zbiór Mandelbrota i kolorowanie pikseli na podstawie liczby iteracji potrzebnych do "ucieczki".
* **Raymarching / Signed Distance Fields (SDF):** Techniki renderowania 3D polegające na "maszerowaniu" wzdłuż promienia z kamery i używaniu funkcji odległości (SDF) do określenia, jak daleko jesteśmy od powierzchni obiektu. `urban_drift.cwa` to zaawansowany przykład raymarchingu renderującego dynamiczne "miasto" z proceduralnie generowanych budynków.
* **Grafika 3D (Wireframe):** Podstawy renderowania 3D: definiowanie wierzchołków i krawędzi obiektu, transformacje (translacja, rotacja, skalowanie) w 3D, projekcja z 3D na 2D i rysowanie krawędzi za pomocą funkcji `line`. `cube_wireframe.cwa` to przykład renderujący obracający się sześcian.
* **Cellular Automata:** Systemy, gdzie stan każdej komórki na siatce zmienia się w zależności od stanu jej sąsiadów według prostych reguł. `game_of_life.cwa` implementuje klasyczną Grę w życie Conwaya, wykorzystując double buffering do płynnej animacji.
* **Interference Patterns:** Generowanie wzorów przez sumowanie lub mnożenie fal (często funkcji trygonometrycznych) od kilku źródeł, animowanych w czasie. `interference.cwa` tworzy wzory interferencyjne na podstawie odległości od dwóch ruchomych punktów.
* **Noise Algorithms:** Algorytmy generujące proceduralny "szum" (np. Perlin noise, Simplex noise) mogą być używane do tworzenia realistycznych tekstur, deformacji powierzchni lub innych efektów wizualnych. Chociaż MicroW8 API ma tylko prosty generator `random()` i `randomf()`, bardziej zaawansowane algorytmy szumu można zaimplementować w CurlyWASM (np. wersje 1D/2D szumu wartości oparte na `randomSeed(index)`).

### 4. Precyzyjny timing i Delta Time

Jak już wiesz z Części 1, brak V-sync na MicroW8 oznacza, że musisz ręcznie zarządzać timingiem animacji. Obliczanie delta time (czasu między kolejnymi klatkami) jest kluczowe, aby ruch i animacje miały stałą prędkość niezależnie od chwilowej liczby klatek na sekundę.

* **Pomiar delta time:** Wykorzystaj rejestr `TIME_MS` (0x40). Odczytaj jego wartość na początku funkcji `upd()`, oblicz różnicę w stosunku do wartości z poprzedniej klatki (zapamiętanej w zmiennej globalnej lub w pamięci, np. pod adresem `0x3c`), a następnie zaktualizuj zapamiętaną wartość.
* **Time-dependent motion:** Używaj `prędkość * delta_time` zamiast stałych przyrostów, aby animacje i ruch były płynne niezależnie od FPS.

**Przykład obliczania i wyświetlania FPS (funkcja `printFps`):**

Poniższy kod pokazuje, jak obliczyć i wyświetlić FPS, korzystając z pamięci użytkownika pod adresem `0x3c` (tuż przed `TIME_MS`) do przechowywania czasu poprzedniej klatki. Jest to powszechna technika sizecodingowa.

```curlywasm
// Potrzebne importy (upewnij się, że są na początku pliku)
import "env.setBackgroundColor" fn setBackgroundColor(i32);
import "env.setTextColor" fn setTextColor(i32);
import "env.setCursorPosition" fn setCursorPosition(i32, i32);
import "env.printInt" fn printInt(i32);
import "env.printChar" fn printChar(i32);

const TIME_MS = 0x40;
const LAST_TIME_MS_ADDR = 0x3c; // Adres w pamięci użytkownika do przechowania poprzedniego czasu

// Przykładowe indeksy kolorów (dostosuj do swojej palety)
const COLOR_DARK_GREY = 1;
const COLOR_WHITE = 15;

// Funkcja obliczająca i wyświetlająca FPS w lewym górnym rogu
fn printFps() {
    let current_time_ms = (TIME_MS)!0; // Odczytaj aktualny czas
    let last_time_ms = (LAST_TIME_MS_ADDR)!0; // Odczytaj czas poprzedniej klatki z 0x3c
    (LAST_TIME_MS_ADDR)!0 = current_time_ms; // Zapisz aktualny czas jako czas poprzedniej klatki dla następnego wywołania

    let delta_ms = current_time_ms - last_time_ms;
    let fps: i32;
    if delta_ms > 0 {
        // Używamy f32 do dzielenia, aby uzyskać większą precyzję przed konwersją na i32
        fps = (1000.0 / (delta_ms as f32)) as i32;
    } else {
        fps = 999; // Obsługa bardzo szybkich klatek lub pierwszego wywołania
    }

    // Wyświetl FPS
    setBackgroundColor(COLOR_DARK_GREY); // Ustaw tło
    setTextColor(COLOR_WHITE); // Ustaw kolor tekstu
    setCursorPosition(0, 0); // Ustaw kursor w lewym górnym rogu
    printInt(fps); // Wydrukuj wartość FPS

    // Opcjonalnie: wyświetl ostrzeżenie, jeśli FPS spadnie poniżej 60
    if fps < 60 {
        printChar(' '); // Spacja dla czytelności
        printChar(0xe3); // Przykładowy znak ostrzegawczy (np. strzałka w dół - zależy od fontu)
        printInt(60 - fps); // Pokaż, o ile FPS spadł poniżej 60
    }
    // Wypełnij resztę linii tłem, aby nadpisać stare znaki (opcjonalne)
    printString("    "); // Wyczyść kilka znaków na prawo
}

// Funkcja startowa - inicjalizuje zapisany czas
export fn start() {
    (LAST_TIME_MS_ADDR)!0 = (TIME_MS)!0; // Zapisz początkowy czas
    // ... reszta inicjalizacji ...
}

// Funkcja update - wywołuje printFps() w każdej klatce
export fn upd() {
    // ... reszta kodu update ...

    printFps(); // Wyświetl FPS
}
```

**Uwagi do `printFps()`:**

* Funkcja oblicza FPS na podstawie różnicy czasu między kolejnymi wywołaniami `upd()`.
* Używa adresu `0x3c` do przechowywania czasu poprzedniej klatki – upewnij się, że nie używasz tego adresu do innych celów.
* Dla pustej pętli `upd()` powinna wyświetlać ~60 FPS (przy założeniu braku opóźnień runtime).
* Jeśli FPS spada poniżej 60, wyświetla znak `0xe3` (jego wygląd zależy od aktywnego fontu) i różnicę.
* Stałe `COLOR_DARK_GREY` i `COLOR_WHITE` to przykładowe indeksy palety – dostosuj je.
* Funkcja `start()` jest potrzebna do zainicjalizowania wartości pod `LAST_TIME_MS_ADDR`.

**Przykład użycia delta time do animacji:**

```curlywasm
// Zmienne globalne do timingu (alternatywa dla zapisu w 0x3c)
// global mut last_frame_time_ms: i32 = 0;
// global mut delta_time_seconds: f32 = 0.0;

// Zmienne globalne do animacji (np. kąt rotacji)
global mut rotation_angle: f32 = 0.0;
const ROTATION_SPEED_DPS: f32 = 90.0; // Prędkość rotacji w stopniach na sekundę
const DEG_TO_RAD: f32 = 3.14159265 / 180.0; // Konwersja stopni na radiany

export fn start() {
  // Zainicjuj czas poprzedniej klatki przy starcie (jeśli używasz zmiennej globalnej)
  // last_frame_time_ms = (TIME_MS)!0;
  (0x3c)!0 = (TIME_MS)!0; // Inicjalizacja, jeśli używamy adresu 0x3c
}

export fn upd() {
  let current_time_ms = (TIME_MS)!0; // Odczytaj aktualny czas w ms
  let last_time_ms = (0x3c)!0; // Odczytaj poprzedni czas
  (0x3c)!0 = current_time_ms; // Zapisz aktualny czas

  // Oblicz delta time w milisekundach, a następnie skonwertuj na sekundy (jako f32)
  let delta_ms = current_time_ms - last_time_ms;
  let delta_time_seconds = (delta_ms as f32) / 1000.0;
  // Zapobiegaj dużym skokom, jeśli klatka trwała bardzo długo (np. po pauzie)
  if delta_time_seconds > 0.1 { delta_time_seconds = 0.0166; }

  // *** Przykład użycia delta time do animacji ***
  // Oblicz przyrost kąta w tej klatce (prędkość w stopniach/sek * delta_time_sekundy)
  let angle_increment_deg = ROTATION_SPEED_DPS * delta_time_seconds;
  // Konwertuj przyrost na radiany
  let angle_increment_rad = angle_increment_deg * DEG_TO_RAD;
  // Zaktualizuj kąt rotacji
  rotation_angle = rotation_angle + angle_increment_rad;
  // Ogranicz kąt do zakresu 0 - 2*PI, aby uniknąć przepełnienia (opcjonalne, ale dobra praktyka)
  let two_pi = 3.14159265 * 2.0;
  rotation_angle = fmod(rotation_angle, two_pi);
  if rotation_angle < 0.0 {
      rotation_angle = rotation_angle + two_pi;
  }

  // Teraz możesz użyć rotation_angle (lub np. sin(rotation_angle), cos(rotation_angle))
  // do obracania obiektów w scenie, np. w przykładzie cube_wireframe.cwa.
  // Ruch będzie miał stałą prędkość niezależnie od FPS.

  // ... reszta kodu rysującego klatkę ...
  printFps(); // Wyświetl FPS na koniec
}
```

### 5. Sound za pomocą `sndGes`

Syntezator `sndGes` to wbudowany układ dźwiękowy MicroW8. Jest on sterowany 32 bajtami pamięci od adresu 0x50 do 0x70. Te 32 bajty są kopiowane z głównego wątku do wątku dźwiękowego po każdym wywołaniu `upd()`, dzięki czemu zmiany w rejestrach stają się słyszalne.

* **Struktura rejestrów (0x50 - 0x70):** 32 bajty podzielone są na 4 bloki po 6 bajtów dla każdego z 4 kanałów (od 0x50, 0x56, 0x5C, 0x62), 2 bajty dla głośności (0x68, 0x69) i 6 bajtów dla filtrów (0x6A - 0x6F).
* **Kanały (0-3):** Każdy kanał ma rejestry do sterowania:
    * CTRL (offset 0): Typ fali, flaga note on, trigger ataku, wybór filtra, stereo panning (wide/narrow), ring modulation.
    * PULS (offset 1): Szerokość pulsu dla różnych typów fal.
    * FINE (offset 2): Ułamkowa część nuty.
    * NOTE (offset 3): Numer nuty (69=A4).
    * ENVA (offset 4): Parametry obwiedni A (Attack, Decay).
    * ENVB (offset 5): Parametry obwiedni B (Sustain, Release).
* **Głośność (0x68, 0x69):** 4-bitowe pola dla par kanałów.
* **Filtry (0x6A - 0x6F):** Programowalne filtry.
* **Funkcja `playNote(channel, note)`:** Jak wspomniano, jest to wygodna funkcja API, która modyfikuje odpowiednie bajty w rejestrach `sndGes`, aby wyzwolić nutę. Jest dobrym punktem wyjścia.
* **Bezpośrednia manipulacja rejestrami:** Dla pełnej kontroli, możesz pisać bezpośrednio do bajtów pamięci od 0x50. Na przykład, aby ustawić falę i nutę na kanale 1 (rejestry zaczynają się od 0x50 + 1\*6 = 0x56):
    ```curlywasm
    let channel1_base = 0x56;
    let wave_type = 1; // Piła
    let note_number = 72; // C5

    // Ustaw falę (Piła = 1 << 6) i flagę note on (bit 0)
    (channel1_base)?0 = (wave_type << 6) | 1;

    // Ustaw nutę
    (channel1_base + 3)?0 = note_number;

    // Aby wyzwolić atak, musisz zmienić bit trigger (bit 1) w rejestrze CTRL (offset 0)
    // Możesz to zrobić np. zmieniając wartość bitu w każdej klatce, gdy nuta powinna być wyzwolona.
    // (channel1_base)?0 = (channel1_base)?0 ^ 2; // XOR bitu triggera

    // Aby zatrzymać nutę, ustaw flagę note on na 0:
    // (channel1_base)?0 = (channel1_base)?0 & ~1; // Clear bit 0
    ```
    Studiowanie kodu `playNote` w `platform.txt` może pomóc zrozumieć, jak ta funkcja modyfikuje rejestry.

### 6. Optymalizacja i narzędzia

* **`uw8 filter-exports`:** Niezbędne narzędzie po kompilacji, zwłaszcza z języków wysokopoziomowych (C, Rust, Zig), aby usunąć niepotrzebne eksporty i dead code, co zmniejsza rozmiar modułu WASM. Warto używać go nawet po kompilacji CurlyWASM, aby upewnić się, że kompilator nie zostawił śmieci.
* **`wasm2wat` (z WebAssembly Binary Toolkit - Wabt):** Pozwala zdekompilować binarny moduł WASM do tekstowego formatu WAT, który jest czytelny dla człowieka. Analiza kodu WAT jest kluczowa w sizecodingu, aby zrozumieć, jak CurlyWASM kompiluje Twój kod i zidentyfikować potencjalne miejsca na dalsze optymalizacje na poziomie instrukcji WASM.
* **`wasm-opt` (z Binaryen):** Potężne narzędzie optymalizacyjne dla modułów WASM. Może wykonać zaawansowane optymalizacje, których kompilator CurlyWASM (celowo prosty) nie robi. Warto włączyć `wasm-opt` do swojego workflow po kompilacji, aby uzyskać mniejszy i szybszy kod.

**Kodowanie pod limity compo**

W demoscenie często tworzy się dema na zawody (compo) z rygorystycznymi limitami rozmiaru, np. 4 KB lub 64 KB. MicroW8, z limitem 256 KB, jest idealny do takich wyzwań. Oto wskazówki, jak zmieścić się w limitach:

* **Minimalizuj importy API:** Importuj tylko niezbędne funkcje (`env.printInt`, `env.setPixel`), by zmniejszyć rozmiar modułu WASM. Każdy importowany symbol dodaje bajty do sekcji importu w WASM.
* **Kompresuj dane:** Pakuj tekstury lub tabele LUT w `USER_MEM` za pomocą operacji bitowych, np. przechowuj 2 piksele 4-bitowe w jednym bajcie, lub używaj prostych schematów kompresji RLE (Run-Length Encoding) dla powtarzalnych danych.
* **Optymalizuj kod:** Używaj `uw8 pack` do kompresji modułu WASM. Użyj `-l 9` dla maksymalnej kompresji (choć jest wolniejsza). Testuj z `wasm-opt --enable-bulk-memory -Oz` (optymalizacja pod kątem rozmiaru) dla dodatkowych oszczędności.
* **Testuj rozmiar:** Regularnie sprawdzaj rozmiar pliku `.uw8` po spakowaniu (`uw8 pack`). Analizuj zdekompilowany kod (`wasm2wat`) aby zobaczyć, które części kodu generują najwięcej bajtów.
* **Przykład:** W `plasma_sizecoding_1.cwa` efekt jest generowany proceduralnie w `upd()` bez przechowywania tekstury, co oszczędza bajty na danych.

### 7. Debugowanie

Debugowanie w low-level środowisku z ograniczonymi narzędziami wymaga kreatywności:

* **Debug output:** Używaj control characters do przekierowania wyjścia tekstu do konsoli debugowania (control character 6) lub do ekranu (control character 4). `printInt(value)` i `printChar('\n')` (control character 10) są nieocenione do wyświetlania wartości zmiennych i śledzenia przebiegu programu.
    ```curlywasm
    // Przykład logowania wartości do konsoli debugowania
    printChar('\6'); // Przełącz wyjście na konsolę
    printString("X="); // Wydrukuj etykietę
    printInt(moj_x); // Wydrukuj wartość zmiennej moj_x
    printChar('\n'); // Drukuj newline
    printChar('\4'); // Przełącz wyjście z powrotem na ekran
    ```
* **`uw8 --debug`:** Użycie tej opcji podczas kompilacji (`uw8 compile --debug <input.cwa> <output.wasm>`) generuje sekcję nazw w module WASM. Narzędzia debugujące (np. wbudowany debugger w przeglądarce Chrome/Firefox) mogą wtedy wyświetlać oryginalne nazwy funkcji i zmiennych z CurlyWASM zamiast bezimiennych indeksów, co znacznie ułatwia śledzenie kodu krok po kroku.
* **`wasm2wat`:** Analiza kodu WAT wygenerowanego przez `wasm2wat` pozwala zobaczyć dokładnie, jakie instrukcje WASM generuje Twój kod CurlyWASM. Pomaga to zrozumieć zachowanie programu na niskim poziomie i wykryć błędy związane z typami, stosem czy operacjami pamięci.

**Debugowanie efektów demoscenowych**

Debugowanie w sizecodingu jest trudne, bo każdy bajt się liczy. Oto praktyczne techniki dla MicroW8:

* **Wyświetlanie zmiennych:** Używaj `printInt` i `printChar` do pokazywania wartości (np. pozycji piksela, FPS, amplitudy dźwięku) bezpośrednio na ekranie w rogu lub używając wyjścia debugowania (Ctrl Char 6). Przykład: w `printFps()` (sekcja 2.4) wyświetlamy FPS i spadek poniżej 60.
* **Wizualizacja:** Rysuj linie (`line`) lub punkty (`setPixel`) dla trajektorii cząsteczek, wektorów czy granic obiektów SDF. Np. w systemie cząsteczek oznacz pozycje pikselami w kontrastowym kolorze (np. 15 - biały). W raymarchingu możesz kolorować piksele na podstawie liczby kroków lub odległości do powierzchni.
* **Breakpointy WASM:** Ustaw breakpointy w devtools przeglądarki (jeśli używasz `uw8 run --browser`), analizując zdekompilowany kod WAT (wygenerowany z `--debug`). To przydatne dla pętli pikselowych lub błędów w skomplikowanych obliczeniach matematycznych (np. raymarching).
* **"Poor man's debugger":** Zmieniaj kolor tła (`cls(kolor)`) lub kolor piksela w rogu ekranu (`setPixel(0,0,kolor)`) w różnych punktach kodu, aby wizualnie śledzić, która część kodu jest wykonywana lub czy dany warunek jest spełniony.
* **Wskazówka:** Zapisuj kluczowe wartości (np. stan automatu, ostatnią obliczoną odległość SDF) w łatwo dostępnym miejscu w `USER_MEM` i odczytuj/wyświetlaj je w `upd()`, by śledzić zmiany bez dodawania dużej ilości kodu debugującego do głównej logiki.

---

## Część 3: Demoscena i Kreatywność na MicroW8

W tej części dotrzemy do sedna demosceny: jak połączyć techniczną wiedzę o MicroW8 i CurlyWASM z kreatywnymi technikami, aby tworzyć hipnotyzujące dema. Demoscena to subkultura z lat 80., gdzie twórcy rywalizują w tworzeniu audiowizualnych prezentacji w ekstremalnych ograniczeniach. MicroW8 to idealny poligon dla współczesnych demoscenerów, łączący retro ograniczenia z nowoczesnym WASM.

### 1. Filozofia demosceny na MicroW8

Programowanie na MicroW8 to powrót do korzeni low-level programmingu, gdzie każdy bajt kodu i każdy cykl procesora ma znaczenie. Kluczowe aspekty tej filozofii to:

* **Creativity in constraints:** Ograniczenia platformy (256 KB, 320x240, brak sprzętowej akceleracji grafiki 3D/spritów, brak V-sync) nie są przeszkodą, lecz katalizatorem kreatywności. Zmuszają do myślenia o sprytnych algorytmach i technikach, które maksymalizują efekt wizualny i dźwiękowy przy minimalnych zasobach.
* **Optimization as art:** Sizecoding to nie tylko konieczność, ale sztuka. Proces optymalizacji rozmiaru kodu i wydajności staje się integralną częścią procesu twórczego. Każdy zaoszczędzony bajt to powód do satysfakcji.
* **Impression over realism:** Celem demoscenowym zazwyczaj nie jest fotorealizm, lecz stworzenie silnego, hipnotyzującego wrażenia audiowizualnego. Proceduralnie generowane efekty, abstrakcyjna grafika, synestezja (połączenie obrazu i dźwięku) są kluczowe.
* **Knowledge sharing:** Demoscena tradycyjnie promuje dzielenie się wiedzą, technikami i kodem (np. na demoparties). Analiza kodu innych twórców (tak jak w przykładach MicroW8) jest doskonałym sposobem na naukę i rozwój.

### 2. Kluczowe techniki demoscenowe w MicroW8

Wiedza o MicroW8 i CurlyWASM pozwala na implementację wielu klasycznych i nowoczesnych technik demoscenowych:

* **Procedural Content Generation (PCG):** Generowanie grafiki, dźwięku i innych zasobów w czasie rzeczywistym za pomocą algorytmów. Jest to esencja wielu efektów demoscenowych i kluczowe dla sizecodingu, ponieważ algorytm zazwyczaj zajmuje mniej miejsca niż gotowe dane.
* **Pixel Effects (Plasma, Interference):** Generowanie koloru każdego piksela na podstawie jego współrzędnych i czasu, często z użyciem funkcji trygonometrycznych i szumu. Przykład: `plasma_sizecoding_1.cwa`, `interference.cwa`.
* **Fractals:** Rysowanie zbiorów fraktalnych przez iteracyjne wzory matematyczne. Przykład: `mandelbrot.cwa`, `sierpinski_triangle.cwa`.
* **Raycasting i Raymarching:** Techniki renderowania 3D/2D oparte na rzucaniu promieni. Wersje oparte na SDF (jak w `urban_drift.cwa`) pozwalają na proceduralne definiowanie geometrii.
* **Noise Patterns:** Generowanie różnych typów szumu (np. Perlin noise, Simplex noise - jeśli zaimplementujesz) do tworzenia tekstur proceduralnych lub animacji.
* **Mathematics jako narzędzie:** Funkcje matematyczne (`sin`, `cos`, `sqrt`, `abs`, `fmod`, etc.) i operatory bitowe są podstawowymi narzędziami do tworzenia efektów wizualnych i dźwiękowych, a także do optymalizacji obliczeń. W sizecodingu, kreatywne wykorzystanie matematyki do generowania wzorów minimalizuje potrzebę przechowywania dużych ilości danych.
* **Palette i color cycling:** Dynamiczna zmiana kolorów w palecie (`PALETTE`, 0x13000) przy zachowaniu statycznych indeksów kolorów w framebufferze pozwala tworzyć animacje, gradienty i efekty "płynięcia" przy minimalnym narzucie obliczeniowym. Przykład: `font_palette.cwa` pokazuje manipulację paletą. Colour cycling to technika, w której kolory w palecie są cyklicznie przesuwane, tworząc wrażenie ruchu.
* **Pamięć jako zasób:** Poza framebufferm i paletą, efektywne wykorzystanie pamięci `USER_MEM` na:
    * **Look-up Tables (LUTs):** Jak wspomniano, prekomputowane tablice wartości (np. sinusów) do szybkiego odczytu.
    * **Sprite'y/Fonty:** Przechowywanie własnych danych graficznych, jeśli nie są generowane proceduralnie.
    * **Dodatkowe bufory:** Np. do double bufferingu (`game_of_life.cwa`).
* **Efficient loops:** Optymalizacja pętli przetwarzających piksele lub dane jest kluczowa dla wydajności. Używaj `loop` i `branch_if`, minimalizuj obliczenia wewnątrz pętli, preferuj bezpośredni dostęp do pamięci (`baza?offset`).
* **Bitwise operations:** Wykorzystanie operacji bitowych do szybkiej matematyki, pakowania danych i generowania wzorów (np. do hashowania pozycji dla szumu) jest fundamentalne w sizecodingu.
* **Audiovisual synchronization:** Synchronizacja efektów wizualnych z dźwiękiem (generowanym np. przez `sndGes`) jest kluczowa dla wrażenia demoscenowego. Czas (`time()`, `TIME_MS`) jest wspólną osią synchronizacji. Przykład: `Chaos_Rose.cwa` pokazuje prostą synchronizację wyzwalania nut z animacją.

**Efekty tunelowe**

Tunele to klasyka demosceny, tworząca iluzję lotu przez cylindryczną lub stożkową strukturę. Są kompaktowe i widowiskowe, idealne dla MicroW8 (320x240, 8 bpp). Idea polega na mapowaniu pikseli ekranu na teksturę 2D, używając odległości od centrum i kąta jako współrzędnych UV, z animowanym przesunięciem dla efektu ruchu.

* **Wygląd:** Na ekranie widzisz perspektywiczny tunel z pulsującymi wzorami, np. szachownicą lub gradientem, które przesuwają się w głąb.
* **Pseudokod:**
    ```pseudocode
    Funkcja tunnel_effect():
      t = pobierz_czas() // Czas do animacji
      rozmiar_tekstury = 64 // Rozmiar tekstury (np. 64x64)
      srodek_x = 160
      srodek_y = 120

      Dla każdego piksela (px, py) na ekranie (0..319, 0..239):
        // Przesuń współrzędne, by środek był w (0,0)
        dx = px - srodek_x
        dy = py - srodek_y

        // Unikaj dzielenia przez zero w centrum
        Jeśli dx == 0 i dy == 0:
            dx = 0.01 // Mała wartość, by uniknąć błędu

        // Oblicz odległość i kąt (współrzędne biegunowe)
        dist = sqrt(dx*dx + dy*dy)
        angle = atan2(dy, dx) // Kąt w radianach

        // Mapuj współrzędne ekranu na współrzędne tekstury (UV)
        // Użyj odwrotności odległości dla efektu głębi
        // Dodaj czas 't' dla animacji ruchu
        tex_u = (rozmiar_tekstury / dist) + t * predkosc_ruchu_u
        tex_v = (angle / (2 * PI)) * rozmiar_tekstury + t * predkosc_ruchu_v

        // Zawijanie współrzędnych tekstury (modulo)
        tex_u_int = floor(tex_u) mod rozmiar_tekstury
        tex_v_int = floor(tex_v) mod rozmiar_tekstury

        // Pobierz kolor z tekstury (przechowywanej np. w USER_MEM)
        kolor = pobierz_piksel_z_tekstury(tex_u_int, tex_v_int)

        // Zapisz kolor do Framebuffera
        zapisz_piksel_do_framebuffera(px, py, kolor)
    ```
* **Wskazówka:** Prekomputuj `sqrt` i `atan2` w tabeli LUT w `USER_MEM`, aby przyspieszyć obliczenia w pętli pikselowej. Tekstura (np. 64x64) może być generowana proceduralnie w funkcji `start()` lub załadowana z sekcji `data`. Zobacz `plasma_sizecoding_1.cwa` dla podobnej animacji opartej na czasie i mapowaniu współrzędnych.

**Rotozoomery**

Rotozoomer to efekt 2D, który skaluje i obraca teksturę (np. szachownicę, sprite) w czasie rzeczywistym, dając dynamiczny ruch. Jest prosty w implementacji i efektowny, szczególnie w sizecodingu.

* **Wygląd:** Tekstura na ekranie obraca się i pulsuje (zmienia rozmiar), tworząc hipnotyczny efekt, np. wirująca szachownica.
* **Pseudokod:**
    ```pseudocode
    Funkcja rotozoom_effect():
      t = pobierz_czas() // Czas do animacji
      rozmiar_tekstury = 32 // Rozmiar tekstury (np. 32x32)
      srodek_x = 160
      srodek_y = 120

      // Oblicz parametry transformacji (rotacja i zoom)
      kat_rotacji = t * predkosc_rotacji
      skala = (sin(t * predkosc_zoom) + 1.0) * 0.5 + minimalna_skala // Skala pulsuje
      cos_t = cos(kat_rotacji)
      sin_t = sin(kat_rotacji)

      Dla każdego piksela (px, py) na ekranie (0..319, 0..239):
        // Przesuń współrzędne, by środek był w (0,0)
        dx = px - srodek_x
        dy = py - srodek_y

        // Zastosuj odwrotną transformację (rotacja i skalowanie)
        // do współrzędnych piksela, aby znaleźć odpowiadający punkt w teksturze
        tex_x = (dx * cos_t - dy * sin_t) / skala
        tex_y = (dx * sin_t + dy * cos_t) / skala

        // Zawijanie współrzędnych tekstury (modulo)
        tex_u_int = floor(tex_x) mod rozmiar_tekstury
        tex_v_int = floor(tex_y) mod rozmiar_tekstury
        // Poprawka dla ujemnych wyników modulo
        Jeśli tex_u_int < 0: tex_u_int += rozmiar_tekstury
        Jeśli tex_v_int < 0: tex_v_int += rozmiar_tekstury

        // Pobierz kolor z tekstury (przechowywanej np. w USER_MEM)
        kolor = pobierz_piksel_z_tekstury(tex_u_int, tex_v_int)

        // Zapisz kolor do Framebuffera
        zapisz_piksel_do_framebuffera(px, py, kolor)
    ```
* **Wskazówka:** Przechowuj małą teksturę (np. 32x32) w `USER_MEM`. Użyj prekomputowanego `sin` i `cos` z tabeli LUT dla szybszych obliczeń. Efekt można połączyć z plazmą (patrz `plasma_sizecoding_1.cwa`) dla jeszcze ciekawszych wzorów.

**Systemy cząsteczek**

Systemy cząsteczek generują wiele małych obiektów (np. iskry, gwiazdy, dym), poruszających się według prostych reguł fizyki (prędkość, grawitacja, czas życia). Są widowiskowe i dobrze skalują się na MicroW8, mimo ograniczeń mocy obliczeniowej.

* **Wygląd:** Dziesiątki lub setki punktów poruszają się po ekranie, tworząc efekty jak deszcz, eksplozje, mgła, często zsynchronizowane z muzyką.
* **Pseudokod:**
    ```pseudocode
    // Struktura (w pamięci) dla pojedynczej cząsteczki
    // np. 4 x f32 = 16 bajtów na cząsteczkę
    Struktura Cząsteczka:
      x, y: pozycja (f32)
      vx, vy: prędkość (f32)
      czas_zycia: pozostały czas życia (f32)

    // Tablica cząsteczek w USER_MEM
    liczba_czasteczek = 100
    tablica_czasteczek = USER_MEM // Adres początku tablicy

    Funkcja init_particles():
      Dla i od 0 do liczba_czasteczek - 1:
        adres_czasteczki = tablica_czasteczek + i * rozmiar_struktury_cząsteczka
        // Ustaw początkowe wartości (np. losowa pozycja, prędkość, czas życia)
        zapisz_float(adres_czasteczki + offset_x, losowa_pozycja_x())
        zapisz_float(adres_czasteczki + offset_y, losowa_pozycja_y())
        zapisz_float(adres_czasteczki + offset_vx, losowa_predkosc_x())
        zapisz_float(adres_czasteczki + offset_vy, losowa_predkosc_y())
        zapisz_float(adres_czasteczki + offset_czas_zycia, losowy_czas_zycia())

    Funkcja update_and_draw_particles():
      dt = oblicz_delta_time() // Czas od ostatniej klatki

      Dla i od 0 do liczba_czasteczek - 1:
        adres_czasteczki = tablica_czasteczek + i * rozmiar_struktury_cząsteczka

        // Odczytaj aktualny stan cząsteczki
        x = odczytaj_float(adres_czasteczki + offset_x)
        y = odczytaj_float(adres_czasteczki + offset_y)
        vx = odczytaj_float(adres_czasteczki + offset_vx)
        vy = odczytaj_float(adres_czasteczki + offset_vy)
        czas_zycia = odczytaj_float(adres_czasteczki + offset_czas_zycia)

        // Aktualizuj stan
        x = x + vx * dt
        y = y + vy * dt
        // Można dodać grawitację: vy = vy + grawitacja * dt
        czas_zycia = czas_zycia - dt

        // Sprawdź, czy cząsteczka "żyje"
        Jeśli czas_zycia > 0:
          // Zapisz zaktualizowany stan
          zapisz_float(adres_czasteczki + offset_x, x)
          zapisz_float(adres_czasteczki + offset_y, y)
          zapisz_float(adres_czasteczki + offset_vx, vx)
          zapisz_float(adres_czasteczki + offset_vy, vy)
          zapisz_float(adres_czasteczki + offset_czas_zycia, czas_zycia)

          // Narysuj cząsteczkę (np. jako piksel)
          px = floor(x)
          py = floor(y)
          Jeśli px >= 0 i px < 320 i py >= 0 i py < 240:
            zapisz_piksel_do_framebuffera(px, py, kolor_czasteczki)
        Inaczej:
          // Cząsteczka "umarła", zresetuj ją (np. w nowej pozycji startowej)
          zresetuj_czasteczke(adres_czasteczki) // Ustawia nową pozycję, prędkość, czas życia
    ```
* **Wskazówka:** Przechowuj dane cząsteczek kompaktowo w `USER_MEM`. Używaj `f32` dla pozycji i prędkości, aby uzyskać płynny ruch. Resetuj cząsteczki, które wyleciały poza ekran lub których czas życia się skończył, aby utrzymać stałą liczbę aktywnych cząsteczek. Połącz z `sndGes` dla synchronizacji z muzyką (np. emituj cząsteczki przy triggerze nuty - patrz `Chaos_Rose.cwa`).

**Dynamiczna manipulacja paletą**

Manipulacja paletą pozwala zmieniać kolory w `PALETTE` (0x13000, 256x32 bpp RGBA) w czasie rzeczywistym, tworząc efekty jak pulsujące gradienty, płynne przejścia tonalne czy „fading” bez konieczności przerysowywania całego framebuffer’a. To bardzo efektywna technika w trybach z ograniczoną liczbą kolorów (jak 8 bpp w MicroW8).

* **Wygląd:** Kolory na ekranie płynnie się zmieniają, np. tło przechodzi z niebieskiego w czerwone, obiekty pulsują w rytm muzyki, lub cały obraz płynnie zanika do czerni.
* **Pseudokod (przykład animacji gradientu w palecie):**
    ```pseudocode
    Funkcja animate_palette():
      t = pobierz_czas() // Czas do animacji

      // Pętla przez część palety, którą chcemy animować (np. indeksy 16-31)
      Dla indeks od 16 do 31:
        // Oblicz kolor na podstawie czasu i indeksu
        // Przykład: pulsujący niebiesko-czerwony gradient
        niebieski = (sin(t * szybkosc_pulsowania + indeks * przesuniecie_fazy_nieb) + 1.0) * 0.5 * 255
        czerwony = (cos(t * szybkosc_pulsowania + indeks * przesuniecie_fazy_czerw) + 1.0) * 0.5 * 255
        zielony = 0 // Bez zielonego w tym przykładzie

        // Złóż kolor RGBA (lub BGRA, zależnie od endianness i formatu MicroW8)
        // Pamiętaj, że MicroW8 używa 32bpp RGBA w PALETTE
        kolor_rgba = (255 << 24) | (niebieski << 16) | (zielony << 8) | czerwony // Przykład AA BB GG RR

        // Oblicz adres w palecie (indeks * 4 bajty)
        adres_w_palecie = PALETTE + indeks * 4

        // Zapisz kolor do palety (używając zapisu 32-bitowego słowa '!')
        zapisz_slowo_do_pamieci(adres_w_palecie, kolor_rgba)
    ```
* **Wskazówka:** Ogranicz liczbę modyfikowanych kolorów w każdej klatce, jeśli manipulacja całą paletą jest zbyt kosztowna obliczeniowo. Prekomputuj `sin`/`cos` w tabeli LUT w `USER_MEM`. Efekt można połączyć z plazmą lub innymi efektami opartymi na indeksach kolorów dla uzyskania większej dynamiki (patrz `plasma_sizecoding_1.cwa`, `font_palette.cwa`). Pamiętaj o używaniu operatora `!` (lub `i32.store`) do zapisu 32-bitowych wartości kolorów i o wyrównaniu adresu do 4 bajtów.

**Synchronizacja audio-wizualna**

Synchronizacja wizualizacji z dźwiękiem to znak rozpoznawczy demosceny. MicroW8 umożliwia to przez odczyt stanu kanałów dźwiękowych `sndGes` (np. czy nuta jest aktywna, jaka jest jej wysokość, jaka jest aktualna wartość obwiedni) lub, jeśli implementujesz własną funkcję `snd()`, przez bezpośrednią komunikację między wątkami audio i głównym (np. przez współdzieloną pamięć). Reakcja efektów wizualnych na muzykę znacząco podnosi wrażenia.

* **Wygląd:** Obiekty (np. koła, linie, cząsteczki) pulsują, zmieniają kolor, rozmiar lub prędkość w rytm muzyki (np. rosną na mocnych uderzeniach stopy perkusyjnej, zmieniają kolor wraz z melodią).
* **Pseudokod (przykład reakcji na trigger nuty):**
    ```pseudocode
    // Zmienna globalna lub w pamięci do śledzenia stanu triggera
    poprzedni_stan_triggera_kanal0 = 0

    Funkcja sync_visuals_with_audio():
      // Odczytaj rejestr CTRL dla kanału 0 syntezatora sndGes (adres 0x50)
      rejestr_ctrl_kanal0 = odczytaj_bajt_z_pamieci(0x50)

      // Sprawdź bit triggera (bit 1)
      aktualny_stan_triggera = (rejestr_ctrl_kanal0 >> 1) & 1

      // Sprawdź, czy nastąpiła zmiana triggera z 0 na 1 (nowa nuta)
      Jeśli aktualny_stan_triggera == 1 i poprzedni_stan_triggera == 0:
        // Wyzwól efekt wizualny, np. zmień kolor tła lub wyemituj cząsteczki
        zmien_kolor_tla_na_chwile(losowy_kolor())
        emituj_eksplozje_czasteczek(srodek_x, srodek_y)

      // Zapisz aktualny stan triggera do porównania w następnej klatce
      poprzedni_stan_triggera_kanal0 = aktualny_stan_triggera

      // Inny przykład: Skaluj efekt na podstawie głośności kanału
      // Odczytaj rejestr głośności dla kanału 0 (dolne 4 bity adresu 0x68)
      glosnosc_kanal0 = odczytaj_bajt_z_pamieci(0x68) & 0x0F
      skala_efektu = 1.0 + (glosnosc_kanal0 / 15.0) * maksymalne_powiekszenie

      rysuj_obiekt_ze_skala(skala_efektu)
    ```
* **Wskazówka:** Odczytywanie rejestrów `sndGes` w każdej klatce jest tanie. Reaguj na kluczowe zdarzenia (trigger nuty, zmiana wysokości dźwięku) lub uśrednione wartości (np. średnia głośność), aby uniknąć zbyt "nerwowych" wizualizacji. Połącz z systemami cząsteczek (emisja w rytm) lub rotozoomerem (pulsowanie w rytm) dla dynamicznych efektów (patrz `Chaos_Rose.cwa` dla prostego przykładu synchronizacji nuty z animacją).

### 3. "Płynność ponad wszystko"

Brak V-sync wymaga, abyś dążył do jak najbardziej stabilnej liczby klatek na sekundę, co zapewni płynność animacji.

* **Analyze bottlenecks:** Mierz czas wykonania funkcji `upd()` za pomocą `TIME_MS` (np. używając funkcji `printFps`), aby zidentyfikować, które części kodu są najwolniejsze i wymagają optymalizacji.
* **Minimalizuj obliczenia w pętlach:** Przenieś kosztowne obliczenia poza pętle główne (np. pętlę rysowania pikseli). Używaj LUTs tam, gdzie to możliwe.
* **`i32` vs `f32`:** Operacje na `i32` są zazwyczaj szybsze niż na `f32`. Używaj `f32` tylko wtedy, gdy potrzebujesz precyzji zmiennoprzecinkowej (np. w obliczeniach pozycji, rotacji, wzorów proceduralnych). Unikaj niepotrzebnych konwersji typów.
* **Bezpośredni dostęp do pamięci:** W pętlach intensywnie operujących na danych (np. na framebufferze), używaj bezpośredniego odczytu/zapisu pamięci (`baza?offset`, `baza!offset`, `baza$offset`) zamiast wywołań funkcji API (`setPixel`, `getPixel`).

### 4. Iteracja procesów kreatywnych

Tworzenie dem to proces ewolucyjny, często iteracyjny:

1.  **Idea:** Znajdź inspirację w matematyce, naturze, muzyce, innych demach, lub po prostu w możliwościach MicroW8.
2.  **Minimal implementation:** Zaimplementuj podstawową wersję efektu lub koncepcji. Nie przejmuj się od razu optymalizacją.
3.  **Optimization:** Gdy podstawowa wersja działa, zacznij optymalizować: redukuj rozmiar kodu (sizecoding), poprawiaj wydajność (timing). Używaj narzędzi takich jak `uw8 filter-exports`, `wasm2wat`, `wasm-opt`. Wykorzystaj bezpośredni dostęp do pamięci, operacje bitowe, LUTs.
4.  **Add details:** Wzbogać demo: dodaj kolory, animacje, dźwięk, efekty post-processingu (np. manipulacja paletą).
5.  **Re-optimization:** Dodanie nowych elementów często wprowadza nowy narzut. Wróć do kroku optymalizacji. Proces sizecodingu i optymalizacji jest często ciągły.
6.  **Polishing:** Dopracuj synchronizację audio-wizualną, popraw płynność animacji (timing, delta time), dostosuj paletę, usuń błędy.

**Prototypowanie dem**

Tworzenie dem to sztuka zaczynania od prostych pomysłów i stopniowego ich rozwijania. Oto jak prototypować na MicroW8:

* **Start od podstaw:** Zaimplementuj pojedynczy efekt (np. plazma, tunel) w `upd()`. Skup się na działającym kodzie, nawet jeśli jest prosty i nieoptymalny.
* **Testuj z minimalną paletą:** Używaj standardowej palety lub tylko kilku podstawowych kolorów, aby przyspieszyć ładowanie i iteracje, skupiając się na logice efektu. Dopracujesz kolory później.
* **Iteracyjne dodawanie:** Po opanowaniu jednego efektu dodaj kolejny (np. cząsteczki do tunelu, synchronizacja dźwięku z rotozoomerem), synchronizując je za pomocą `time()` lub `TIME_MS`.
* **Debugowanie:** Używaj `printInt` i `printFps` (z sekcji 2.4) do sprawdzania kluczowych wartości (np. FPS, liczba cząsteczek, współrzędne). Używaj wizualizacji (kolorowanie pikseli/tła) do śledzenia stanu.
* **Wskazówka:** Testuj każdy etap na MicroW8 (lub w emulatorze `uw8 run`), by upewnić się, że FPS nie spada drastycznie poniżej docelowego poziomu (np. 60 FPS). Inspiruj się prostymi przykładami, jak `cube_wireframe.cwa` czy `plasma_sizecoding_1.cwa`, aby zobaczyć podstawowe struktury kodu.

---

## Część 4: Inspiracje i Zasoby Demosceny

Demoscena to społeczność pełna kreatywności i technicznej wirtuozerii. Oto, jak czerpać inspirację i dołączyć do niej:

**Kultowe dema i ich techniki**

Analiza klasycznych dem może być świetnym źródłem pomysłów na efekty możliwe do zaimplementowania (lub adaptacji) na MicroW8:

* **Second Reality (Future Crew, 1993):** Klasyk PC demosceny, znany m.in. z efektów rotozoomera, tunelu, morphingu obiektów 3D i zaawansowanej grafiki wektorowej. Zaimplementuj prosty rotozoomer (sekcja 3.2) lub tunel inspirowany `plasma_sizecoding_1.cwa`.
* **Chaos Theory (Conspiracy, 2006):** Demo PC pokazujące zaawansowany raymarching proceduralnych scen 3D z doskonałą synchronizacją audio-wizualną. Adaptuj techniki SDF z `urban_drift.cwa` dla podobnych, choć prostszych, efektów w MicroW8.
* **Elevated (RGBA, 2009):** Legendarne 4KB intro na PC, które renderuje minimalistyczną, ale piękną scenę 3D z proceduralnymi teksturami i oświetleniem, mieszcząc wszystko w ekstremalnie małym rozmiarze. Użyj `cube_wireframe.cwa` jako punktu wyjścia do eksperymentów z kompaktowymi demami 3D.

**Przykłady na MicroW8**

Przeglądaj istniejące dema i przykłady stworzone specjalnie dla MicroW8, np. `mandelbrot.cwa` (fraktale), `urban_drift.cwa` (raymarching) czy `Chaos_Rose.cwa` (synchronizacja audio-wizualna). Sprawdź oficjalne repozytorium MicroW8 na GitHubie oraz dema prezentowane na demoparty (np. Revision, Assembly, Function) – często są dostępne online.

**Zasoby społeczności**

* **Pouet.net:** Największa internetowa baza danych dem i intr, z linkami do pobrania, screenshotami, komentarzami i wynikami z demoparty. Niezastąpione źródło inspiracji i wiedzy o historii demosceny.
* **Demoscene.pl / Forum.Demoscena.pl:** Polskie fora dyskusyjne poświęcone demoscenie, grafice komputerowej, muzyce trackerowej i programowaniu. Dobre miejsce do zadawania pytań, dzielenia się pracami i poznawania lokalnej społeczności.
* **Demoparty:** Wydarzenia (zjazdy), na których twórcy prezentują swoje dema, rywalizują w konkursach (compo) i wymieniają się wiedzą. Największe międzynarodowe to Revision (Niemcy, online), Assembly (Finlandia). W Polsce popularne są Riverwash, Function, Xenium. Dołącz do nich (fizycznie lub online), by poczuć atmosferę sceny i pokazać swoje prace.
* **MicroW8 Community:** Szukaj dedykowanych kanałów na Discordzie, forach programistycznych lub w mediach społecznościowych, gdzie użytkownicy MicroW8 mogą dzielić się kodem, zadawać pytania i współpracować.

**Wskazówka:** Zacznij od stworzenia prostego intra w MicroW8 (np. tunel, plazma lub efekt cząsteczkowy z dźwiękiem) i zgłoś je na compo w odpowiedniej kategorii (np. Fantasy Console, Low-End Intro, lub nawet PC 256KB Intro, jeśli uda Ci się tak zoptymalizować!). Kolego koderze, świat demosceny czeka!

---

## Słowniczek

* **Alignment:** Wyrównanie adresów pamięci. W WASM i MicroW8, operacje na danych wielobajtowych (np. 32-bitowe słowa `i32`, `f32`) są najefektywniejsze (a czasem wymagane), gdy adres pamięci jest wielokrotnością rozmiaru danych (np. adres podzielny przez 4 dla i32/f32).
* **API (Application Programming Interface):** Zestaw funkcji i procedur udostępnianych przez MicroW8 (w module `"env"`) do interakcji z konsolą (rysowanie, dźwięk, input, czas, losowość).
* **Back Buffer:** Bufor w pamięci RAM (np. w `USER_MEM`) używany do rysowania kolejnej klatki animacji. Po zakończeniu rysowania, jego zawartość jest szybko kopiowana do Front Buffera, aby uniknąć tearingu i flickeringu.
* **Bitfield:** Struktura danych, w której informacje są przechowywane na poziomie pojedynczych bitów lub małych grup bitów w ramach jednej zmiennej (np. bajtu lub słowa). Stan gamepada w MicroW8 jest bitfieldem.
* **Bitwise Operations:** Operacje logiczne i przesunięcia wykonywane bezpośrednio na bitach wartości numerycznych (AND `&`, OR `|`, XOR `^`, przesunięcia `<<`, `>>`, `#>>`). Kluczowe w low-level programmingu, sizecodingu i generowaniu proceduralnych wzorów.
* **Built-in:** Funkcje lub instrukcje, które są natywnie dostępne w języku CurlyWASM lub środowisku WASM (MVP) i nie wymagają importowania z zewnętrznych modułów (np. `sqrt`, `min`, `max`, intrinsics pamięci i bitowe).
* **Byte:** Podstawowa jednostka informacji cyfrowej, składająca się z 8 bitów. Adresy pamięci w MicroW8 są bajtowe.
* **Callback:** Funkcja przekazywana do innej części programu (np. runtime konsoli), która zostanie wywołana w odpowiednim momencie lub w odpowiedzi na zdarzenie. W MicroW8 `start()` i `upd()` to funkcje callback wywoływane przez runtime konsoli.
* **Cellular Automata:** Modele obliczeniowe, w których siatka komórek o prostych stanach ewoluuje w dyskretnych krokach czasowych zgodnie z regułami zależnymi od stanu sąsiadów. Gra w życie Conwaya to przykład automatu komórkowego zaimplementowanego w `game_of_life.cwa`.
* **Clever Algorithms:** "Sprytne algorytmy" – techniki programistyczne, które pozwalają osiągnąć zamierzony efekt wizualny lub funkcjonalność w sposób bardzo efektywny pod względem zasobów (czasu procesora, pamięci, rozmiaru kodu), co jest kluczowe w sizecodingu.
* **Color Cycling:** Technika animacji polegająca na cyklicznej zmianie kolorów w palecie w tle, podczas gdy dane pikseli w framebufferze (indeksy do palety) pozostają statyczne. Tworzy wrażenie ruchu, gradientów lub zmian kolorów przy minimalnym narzucie obliczeniowym na piksel.
* **Color Palette:** Tabela kolorów. W MicroW8, framebuffer przechowuje 8-bitowe indeksy do 256-pozycyjnej palety, która zawiera 32-bitowe wartości kolorów RGBA.
* **Compo:** (Skrót od Competition) Zawody organizowane na demoparty, gdzie twórcy prezentują swoje produkcje (dema, intra, grafiki, muzykę) w określonych kategoriach (np. PC Demo, Oldskool Intro, Fantasy Console, 4KB Intro), rywalizując o uznanie publiczności i jury.
* **Compiling:** Proces tłumaczenia kodu źródłowego (np. CurlyWASM) na kod maszynowy lub pośredni format (np. WebAssembly) wykonywany przez kompilator (`uw8 compile`).
* **Constant Folding:** Optymalizacja kompilatora polegająca na obliczaniu wyrażeń złożonych wyłącznie ze stałych w czasie kompilacji, a nie w czasie wykonania programu. CurlyWASM wykonuje podstawowy constant folding.
* **Control Characters:** Specjalne znaki (o kodach 0-31) w MicroW8, które nie są drukowane jako symbole, lecz wywołują określone akcje formatujące tekst lub zmieniające stan konsoli (np. ustawienie koloru, pozycji kursora, przełączenie wyjścia na konsolę debugowania).
* **Control Flow:** Sekwencja instrukcji wykonywanych w programie. Strukturalne instrukcje WASM/CurlyWASM takie jak `if`, `loop`, `block`, `branch_if` kontrolują ten przepływ.
* **CurlyWASM:** Język programowania wysokiego poziomu o składni zbliżonej do Rust, kompilujący się do WebAssembly. Zaprojektowany jako syntactic sugar dla WASM, aby ułatwić pisanie kodu przy jednoczesnym zachowaniu kontroli nad niskopoziomowymi aspektami.
* **Dead Code:** Kod w programie, który nigdy nie zostanie wykonany. Narzędzia optymalizacyjne takie jak `uw8 filter-exports` potrafią usunąć martwy kod, zmniejszając rozmiar modułu.
* **Debug Output:** Informacje (np. wartości zmiennych, komunikaty o stanie) drukowane w celu pomocy w znajdowaniu i naprawianiu błędów. W MicroW8 realizowane za pomocą control characters do konsoli debugowania i funkcji `printInt`.
* **Debugging:** Proces identyfikacji, lokalizowania i naprawiania błędów w programie. Narzędzia takie jak `uw8 --debug`, debug output i analiza kodu WAT są pomocne w debugowaniu kodu CurlyWASM/WASM.
* **Delta Time:** Czas (zazwyczaj w sekundach lub milisekundach) który upłynął od poprzedniego wywołania funkcji aktualizującej scenę (np. `upd()`). Używany do tworzenia animacji i ruchu o stałej prędkości, niezależnie od wahań liczby klatek na sekundę. Obliczany z `TIME_MS` w MicroW8.
* **Demoscene:** Subkultura komputerowa zajmująca się tworzeniem i prezentowaniem (na tzw. demoparties) krótkich, samowystarczalnych programów audiowizualnych (dem), często z bardzo surowymi ograniczeniami rozmiaru i sprzętu.
* **Direct Memory Access:** Bezpośrednie odczytywanie i zapisywanie danych w określonych adresach pamięci RAM (`baza?offset`, `baza!offset`, `baza$offset`). W MicroW8 jest to często bardziej efektywne i kompaktowe niż używanie funkcji API, zwłaszcza w pętlach operujących na dużych zbiorach danych (jak framebuffer).
* **Double Buffering:** Technika renderowania grafiki polegająca na użyciu dwóch buforów pamięci: jednego wyświetlanego na ekranie (Front Buffer) i drugiego, do którego rysowana jest kolejna klatka (Back Buffer). Po ukończeniu rysowania, bufory są zamieniane (lub Back Buffer jest kopiowany do Front Buffera). Zapobiega tearingowi i flickeringowi.
* **Dynamic Palette Manipulation:** Technika zmiany kolorów w palecie (`PALETTE`) w czasie rzeczywistym dla uzyskania efektów wizualnych, takich jak animowane gradienty, pulsowanie kolorów, czy płynne przejścia tonalne, bez modyfikowania danych w framebufferze.
* **Efficient Loops:** Pętle zoptymalizowane pod kątem szybkości wykonania i/lub minimalnego rozmiaru kodu. Osiąga się to przez minimalizację pracy wewnątrz pętli, użycie efektywnych instrukcji (np. operacje pamięci, bitowe) i właściwe wykorzystanie struktury pętli w CurlyWASM (`loop`, `branch_if`).
* **Fantasy Console:** Wirtualna maszyna lub emulator symulujący proste, retro systemy komputerowe lub konsolowe, ale z nowoczesnymi narzędziami deweloperskimi. MicroW8 jest przykładem konsoli fantasy.
* **Flickering:** Migotanie obrazu, które może wystąpić, gdy dane wyświetlane na ekranie są aktualizowane w trakcie rysowania. Zapobiega mu stosowanie double bufferingu.
* **Font:** Zestaw graficznych reprezentacji (glifów) znaków. MicroW8 ma wbudowany font w pamięci pod adresem `FONT`, ale można użyć własnego.
* **Frame Time:** Czas potrzebny na wyrenderowanie i wyświetlenie pojedynczej klatki animacji. W MicroW8, przy braku V-sync, jest zmienny i zależy od złożoności obliczeń w funkcji `upd()`. Mierzony za pomocą `TIME_MS`.
* **Framebuffer:** Obszar pamięci RAM, który przechowuje dane graficzne (piksele) aktualnie wyświetlane na ekranie. W MicroW8 to 8-bitowy bufor indeksów kolorów pod adresem `FRAMEBUFFER`.
* **Front Buffer:** W technice double bufferingu, bufor pamięci, którego zawartość jest aktualnie wyświetlana na ekranie. W MicroW8 jest to natywnie framebuffer.
* **Gamepad State:** Stan wirtualnego gamepada w MicroW8, przechowywany jako bitfield w pamięci pod adresem `GAMEPAD`. Pozwala na sprawdzenie, które przyciski są wciśnięte w danej chwili.
* **Gradients:** Płynne przejścia tonalne między kolorami. W MicroW8 można je uzyskać poprzez odpowiednie ustawienie kolorów w palecie lub przez proceduralne obliczanie kolorów pikseli.
* **Graphics Mode:** Jeden z trybów drukowania tekstu w MicroW8, w którym znaki są rysowane w dowolnej pozycji pikselowej, a tło znaków jest przezroczyste. Kontrastuje z trybem normalnym (siatka 8x8 z tłem).
* **High-Level Syntax:** Składnia języka programowania, która jest bardziej abstrakcyjna i czytelna dla człowieka niż niskopoziomowe instrukcje maszynowe. CurlyWASM jest językiem wysokiego poziomu w stosunku do WebAssembly.
* **Input:** Dane pochodzące od użytkownika, np. wciśnięcia przycisków gamepada lub klawiszy klawiatury. W MicroW8 obsługiwane przez API i rejestr `GAMEPAD`.
* **Instruction Intrinsics:** Specjalne słowa kluczowe lub funkcje w języku programowania, które mapują się bezpośrednio na pojedyncze, niskopoziomowe instrukcje maszyny wirtualnej (WASM), pozwalając na bardzo precyzyjną kontrolę i optymalizację. Przykład: `i32.load`, `f32.store` w CurlyWASM.
* **Intro:** Krótkie demo, zwykle tworzone z myślą o bardzo rygorystycznych limitach rozmiaru (np. 256 bajtów, 1 KB, 4 KB, 64 KB), prezentujące skondensowane efekty audiowizualne i demonstrujące umiejętności programisty w zakresie sizecodingu.
* **Iterative Formulas:** Formuły matematyczne, które są powtarzane (iterowane) wielokrotnie, a wynik każdej iteracji jest podstawą do następnej. Używane do generowania fraktali, np. Zbiór Mandelbrota.
* **`local.tee`:** Instrukcja WebAssembly (WASM), która wykonuje dwie operacje: przypisuje wartość do zmiennej lokalnej i jednocześnie pozostawia kopię tej wartości na stosie, skąd może być użyta w dalszym wyrażeniu. Często generowana przez operator `:=` i modyfikator `let lazy` w CurlyWASM.
* **Look-up Tables (LUTs):** Tablice prekomputowanych wartości (np. funkcji trygonometrycznych), które są obliczane raz (np. w `start()` lub sekcji `data`) i przechowywane w pamięci. W czasie działania programu, zamiast obliczać wartość funkcji, odczytuje się ją z tabeli na podstawie indeksu. Znacznie szybsze niż powtarzające się obliczenia w pętli głównej i kluczowe w sizecodingu.
* **Low-Level Programming:** Programowanie blisko instrukcji maszyny lub maszyny wirtualnej, pozwalające na precyzyjną kontrolę nad zasobami (pamięcią, cyklami procesora) i minimalizację narzutu. CurlyWASM, kompilując do WASM, umożliwia low-level programming.
* **Memory Operations:** Instrukcje lub operatory języka programowania służące do odczytu i zapisu danych w pamięci RAM. W CurlyWASM obejmują operatory `?`, `!`, `$`, a także intrinsics `i32.load`, `f32.store`, `memory.copy`, `memory.fill`.
* **Noise Algorithms:** Algorytmy generujące ciągi liczb pseudolosowych o określonych właściwościach (np. "gładkie" przejścia, brak widocznych wzorców), używane do tworzenia proceduralnych tekstur, terenów, czy animacji.
* **Packing:** Kompresja lub upakowywanie danych w celu zmniejszenia ich rozmiaru. W sizecodingu odnosi się też do upakowywania wielu informacji w mniejszej liczbie bitów/bajtów (np. za pomocą operacji bitowych). Narzędzie `uw8 pack` kompresuje moduł WASM.
* **Palette Manipulation:** Dynamiczna zmiana wartości kolorów przechowywanych w palecie (`PALETTE`) w trakcie działania programu. Technika ta, w połączeniu ze statycznymi indeksami kolorów w framebufferze, jest wykorzystywana do tworzenia efektów animacji, np. color cycling.
* **Particle System:** Technika grafiki komputerowej polegająca na symulacji i renderowaniu dużej liczby małych obiektów (cząsteczek), które poruszają się według określonych reguł (np. fizyki, zachowań stadnych), tworząc złożone, dynamiczne efekty wizualne, takie jak dym, ogień, deszcz, eksplozje czy mgławice.
* **Pattern Generation:** Tworzenie wizualnych lub dźwiękowych wzorców za pomocą algorytmów matematycznych lub proceduralnych, zamiast używania predefiniowanych grafik/sampli. Podstawa wielu efektów demoscenowych.
* **Photorealism:** Dążenie do realistycznego odwzorowania rzeczywistości w grafice komputerowej. W demoscenie zazwyczaj mniej ważne niż tworzenie imponującego wrażenia wizualnego, często abstrakcyjnego.
* **Pixel Data:** Dane, które określają kolor lub inne właściwości każdego pojedynczego punktu (piksela) obrazu cyfrowego. W MicroW8 są to 8-bitowe indeksy kolorów w framebufferze.
* **Pixel Effects:** Efekty wizualne, w których kolor każdego piksela jest obliczany niezależnie lub na podstawie niewielkiej grupy sąsiadujących pikseli, zazwyczaj w oparciu o funkcje matematyczne i parametry animacji (np. czas, pozycja źródła). Przykłady: plasma, interference.
* **Procedural Calculations:** Obliczenia wykonywane w czasie działania programu w celu wygenerowania danych (np. kolorów pikseli, współrzędnych wierzchołków, wartości próbek dźwiękowych) w oparciu o algorytmy, a nie o predefiniowane dane.
* **Procedural Content Generation (PCG):** Generowanie zasobów (grafiki, modeli 3D, tekstur, dźwięków, danych leveli) w czasie rzeczywistym za pomocą algorytmów. Kluczowa technika w sizecodingu do tworzenia złożonej zawartości przy minimalnym rozmiarze danych.
* **Procedural Effects:** Efekty wizualne lub dźwiękowe generowane w czasie rzeczywistym za pomocą algorytmów proceduralnych, często zależne od czasu lub innych dynamicznych parametrów.
* **Programmable Filters:** Filtry dźwiękowe w syntezatorze `sndGes` MicroW8, których parametry (częstotliwość odcięcia, rezonans) mogą być dynamicznie zmieniane przez programistę poprzez modyfikację odpowiednich rejestrów w pamięci.
* **Prototyping:** Wczesny etap tworzenia programu (np. dema), polegający na szybkiej implementacji podstawowej funkcjonalności lub kluczowego efektu, często w uproszczonej formie, w celu przetestowania koncepcji, zidentyfikowania problemów i iteracyjnego rozwoju przed pełną implementacją i optymalizacją.
* **Randomness:** Generowanie sekwencji liczb, które wyglądają na losowe. W MicroW8 dostępne przez funkcje `random()` i `randomf()`, a także sterowane przez `randomSeed()`.
* **Raycasting:** Technika renderowania grafiki 3D (lub 2D), polegająca na rzucaniu promieni z punktu widzenia (kamery) przez każdy piksel ekranu w scenę 3D i obliczaniu koloru piksela na podstawie punktu, w którym promień napotyka pierwszy obiekt.
* **Raymarching:** Technika renderowania 3D, często używana z Signed Distance Fields (SDF). Zamiast bezpośrednio obliczać przecięcie promienia z geometrią, wykonuje się kroki wzdłuż promienia, używając wartości SDF, aby oszacować minimalną odległość do geometrii i w ten sposób "maszerować" w kierunku powierzchni.
* **Rotozoomer:** Efekt graficzny 2D polegający na jednoczesnym obracaniu (rotation) i skalowaniu (zoom) tekstury lub obrazu w czasie rzeczywistym. Popularny w demoscenie ze względu na dynamiczny, hipnotyzujący wygląd i stosunkowo prostą implementację.
* **Runtime:** Środowisko, w którym wykonywany jest program. W przypadku MicroW8, runtime to emulator lub aplikacja uruchamiająca moduł WebAssembly i zapewniająca dostęp do API konsoli.
* **Screen Tearing:** Wada graficzna objawiająca się "rozdarciem" obrazu, gdy ekran wyświetla fragmenty z różnych klatek jednocześnie. Spowodowane brakiem synchronizacji pionowej (V-sync) między odświeżaniem ekranu a renderowaniem nowej klatki. Double buffering zapobiega temu.
* **Sequencing Operator:** Operator `<|` w CurlyWASM. Wykonuje wyrażenie po lewej stronie (często dla jego efektów ubocznych) i zwraca wartość wyrażenia po prawej stronie. Pomaga w sizecodingu, minimalizując potrzebę używania zmiennych pomocniczych.
* **Sizecoding:** Sztuka pisania programów (często dem lub intr) o ekstremalnie małym rozmiarze (np. mieszczących się w 256 bajtach, 1 KB, 4 KB), wymagająca dogłębnej znajomości platformy docelowej, formatu binarnego i kreatywnych technik programistycznych w celu maksymalizacji efektu przy minimalnym zużyciu bajtów.
* **Sound Synchronization:** Synchronizacja efektów dźwiękowych lub muzyki z wizualizacjami w demo. Wymaga użycia wspólnej osi czasu (np. `time()`, `TIME_MS`) lub bezpośredniej reakcji na zdarzenia dźwiękowe do koordynacji obu elementów.
* **Stable FPS:** Utrzymanie stałej liczby klatek na sekundę. W środowiskach bez V-sync (jak MicroW8), osiągnięcie stabilnego FPS zależy od optymalizacji kodu tak, aby czas wykonania funkcji `upd()` był w miarę stały i mieścił się w budżecie czasowym (np. 16.67 ms dla 60 FPS).
* **Syntactic Sugar:** Elementy języka programowania, które ułatwiają pisanie i czytanie kodu, nie dodając przy tym fundamentalnie nowej funkcjonalności (są tłumaczone na bardziej podstawowe konstrukcje). CurlyWASM jest syntactic sugar dla WebAssembly.
* **Time-Dependent Motion:** Animacje i ruch, których prędkość zależy od delta time, a nie od stałej wartości przyrostu. Zapewnia to, że ruch ma stałą prędkość w świecie wirtualnym, niezależnie od chwilowej liczby klatek na sekundę.
* **Timing:** Zarządzanie czasem w programach, zwłaszcza w animacjach i grach. W MicroW8 przy braku V-sync wymaga manualnego pomiaru delta time i używania go do sterowania animacją.
* **Tunnel Effect:** Klasyczny efekt demoscenowy symulujący lot przez trójwymiarowy tunel. Zazwyczaj implementowany przez mapowanie współrzędnych pikseli ekranu na współrzędne tekstury 2D (często w układzie biegunowym), gdzie odległość od centrum ekranu odpowiada głębokości w tunelu, a kąt - pozycji na obwodzie. Animacja jest tworzona przez przesuwanie współrzędnych tekstury w czasie.
* **V-sync:** Vertical Synchronization. Technika synchronizująca moment wysyłania nowej klatki obrazu do monitora z momentem odświeżania ekranu. Zapobiega tearingowi. MicroW8 nie implementuje V-sync, co wymaga manualnego zarządzania timingiem i stosowania technik jak double buffering.
* **Vector Operations:** Operacje matematyczne na wektorach (np. dodawanie, odejmowanie, mnożenie przez skalar, iloczyn skalarny/wektorowy). Kluczowe w grafice 3D, fizyce i wielu efektach proceduralnych. W CurlyWASM wymagają manualnej implementacji lub użycia bibliotek, o ile są dostępne.
* **WASM Analysis:** Proces analizy kodu WebAssembly (binarnego lub w formacie WAT) w celu zrozumienia jego działania, zidentyfikowania problemów z wydajnością lub rozmiaru, a także do celów reverse engineeringu. Narzędzia takie jak `wasm2wat` i `wasm-opt` są w tym pomocne.
* **WASM Text Format (.wat):** Tekstowa reprezentacja kodu WebAssembly, czytelna dla człowieka. Pliki `.wat` mogą być kompilowane do binarnego WASM (`wat2wasm`) i binarny WASM może być dekompilowany do WAT (`wasm2wat`).
* **WebAssembly (WASM):** Niskopoziomowy, binarny format instrukcji dla maszyny wirtualnej opartej na stosie. WASM jest celem kompilacji dla CurlyWASM i innych języków. MicroW8 wykonuje moduły WASM.
* **WebAssembly MVP:** Skrót od Minimum Viable Product. Pierwsza, podstawowa wersja standardu WebAssembly. CurlyWASM i MicroW8 obecnie celują w tę wersję.
* **Word:** W kontekście 32-bitowych architektur (jak WASM MVP), "word" często odnosi się do 32-bitowej (4-bajtowej) jednostki danych (`i32`, `f32`).
* **Wrap-around:** Technika używana np. przy dostępie do tablic lub buforów (jak framebuffer), polegająca na "zawijaniu" indeksu lub współrzędnej, gdy przekracza granicę rozmiaru. Np. przy dostępie do pikseli na ekranie, piksel o współrzędnej X równej szerokości ekranu jest traktowany jak piksel o X=0. W MicroW8 stosowane np. w `game_of_life.cwa`.

---

Powodzenia w tworzeniu dem na MicroW8! Niech Twoje dzieła oczarują scenę!

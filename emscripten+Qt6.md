# **Qt** z **emscripten**

## Linki

+ [Qt for WebAssembly](https://doc-snapshots.qt.io/qt6-dev/wasm.html)

## Budowanie prostych projektów **C++** (bez **Qt**) w **emscripten**

+ Zainstalować [**Git**](https://git-scm.com/downloads), [**sed** for Windows](http://gnuwin32.sourceforge.net/packages/sed.htm), [**Python**](https://www.python.org/downloads/), [**Perl**](https://strawberryperl.com) ustawić PATH dla tych narzędzi.
+ Zainstalować **emscripten** z [Download and install](https://emscripten.org/docs/getting_started/downloads.html)
  > `git clone https://github.com/emscripten-core/emsdk.git`  
  > `cd emsdk`  
  > `emsdk install latest`  
+ Aktywować środowisko emscripten
  > `emsdk activate latest`  
  > `emsdk_env.bat`  
  > `emsdk install tot`
+ Przetestować jakiś prosty *`test.cpp`* z hello world:
  >```cpp
  >  #include <iostream>  
  >  int main() { std::cout << "Hello world!" << std::endl; }
  >```
  
  > `em++ -O2 -o test.html test.cpp`  
  
+ Uruchomić program wg jednej z metod:  
  + > `python -m http.server 8000`  
    Otworzyć w przeglądarce *`http://localhost:8080/test.html`*
  + `emrun --browser=firefox test.html`
  

## Kompilowanie projektów **Qt** w emscripten

+ Zainstalować **Qt** zaznaczając w instalatorze dodatkowo:
  + W **Qt** lub **Preview**:
    + Źródła Qt - domyślnie do *`C:\Qt\<wersja>\Src`*
    + Skompilowane w **MinGW** binarki - domyślnie do *`C:\Qt\<wersja>\mingw_64`* 
  + W **Developer and Designer tools**:  
    + Najnowsza wersja **MinGW**
    + Najnowsza wersja **CMake**
+ Dodać:
  + *`C:\Qt\Tools\mingw<wersja>_64\bin`*  
  + *`C:\Qt\Tools\CMake_64\bin`*   
  do zmiennej środowiskowej `PATH`
+ Zbudować **Qt** -  W tym samym okienku konsoli przejść do *`C:\Qt\<wersja>\Src`* i uruchomić:
  > `configure -qt-host-path C:\Qt\<wersja>\mingw_64 -no-warnings-are-errors -platform wasm-emscripten -feature-thread -release -prefix %CD%/qtbase -skip qtquick3d -skip qtquick3dphysics [-skip another_module]`  
  
  + `-feature-thread` dodaje obsługę wątków
  + można dodać `-msimd128`, co włącza [konwersję **x86 SSE** na **WASM SIMD128**](https://emscripten.org/docs/porting/simd.html)

  Konfiguracja powinna zakończyć się komunikatami:
  > `-- Configuring done`  
  > `-- Generating done`  
  > `-- Build files have been written to: C:/Qt/<wersja>/Src`
  

  > `cmake --build . -t qtbase qtsvg qtimageformats [<another modules>]`
  
  Kolejne etapy budowania powinny kończyć się komunikatami typu:
  > `[100%] Built target qtbase`  
  > `[100%] Built target qtsvg`


+ Projekty **Qt** można teraz budować za pomocą:
  > `C:\Qt\<wersja>\Src\qtbase\bin\qmake.bat <parametry> <projekt>.pro`
  `
+ Zrobić przykładowy projekt z GUI, np. *`C:\Dev\QtProjects\WebAssemblyTest1.pro`*, wybrać **qmake** jako system budowania *(**CMake** u mnie ma problem z biblioteką **pthreads**)*.

+ Wejść w **cmd** do folderu *`C:\Dev\QtProjects\WebAssemblyTest1`* i uruchomić budowanie:
  > `C:\Qt\<wersja>\Src\qtbase\bin\qmake.bat "CONFIG+=optimize_full" "QT_WASM_PTHREAD_POOL_SIZE=8" "QT_WASM_INITIAL_MEMORY=2GB" WebAssemblyTest1.pro`  
  > `mingw32-make.exe`
  + `QT_WASM_PTHREAD_POOL_SIZE` ustawia maksymalną liczbę wątków w WebAssembly.
  + `QT_WASM_INITIAL_MEMORY` ustawia początkową pulę pamięci. Przy włączonej wielowątkowości jest ona stała i nie może rosnąć, więc powinien być wystarczający zapas.
  
+ Uruchomić program w przeglądarce:
  > `emrun --browser=chrome WebAssemblyTest1.html`

  Jeśli używane są wątki, muszą zostać ustawione nagłówki **COEP** i **COOP**. Server **qtwasmserver** robi to automatycznie:
  > `python C:\Qt\<wersja>\Src\qtbase\util\wasm\qtwasmserver\qtwasmserver.py --all`

# **Emscripten** w **QtCreator**

+ Ustawić kompilator **emscripten** w **QtCreator** wg instrukcji [Building Applications for the Web](https://doc.qt.io/qtcreator/creator-setup-webassembly.html)
+ Ustawić zbudowane biblioteki w **QtCreator** (Kits - Qt Versions - *`C:\Qt\<wersja>\Src\qtbase\bin\qmake.exe`*)

# Umieszczanie zasobów w aplikacji
## [TODO: Dołączanie i osadzanie plików]

# Cross-origin resource sharing
## [TODO: użycie cors-anywhere]

# Użycie `QDialog::exec()`

+ Funkcja `QDialog::exec()` nie jest w pełni funkcjonalna w **WebAssembly**. Uruchamia dialog, po czym zwraca natychmiast, nie czekając na reakcję użytkownika. Ten problem można obejść w ten sposób, mają przykładowy dialog `class CustomDialog : public QDialog`:

  ```cpp
  auto dialog { new CustomDialog{this} };

  connect(dialog, &QDialog::finished, this, [=](int ret) {
    if (ret == QDialog::Rejected) return;
    (...) // instrukcje do wykonania, jeśli dialog został zaakceptowany
  });

  dialog->exec();
  ```

# Osadzenie kodu JavaScript w C++
## [TODO: `emscripten_run_script()`, `EM_JS()` i `EM_ASM()`]

# Hosting

## [TODO: GitHub Pages]

# Wskazówki

+ **QtMultimedia**  
  Pakiet **QtMultimedia** domyślnie nie jest wspierany przez wyłączoną obsługę wątków. Jeśłi użyto opcji `-feature-thread`, można dodać `qtmultimedia` do listy budowanych pakietów.

+ [**Embind**](https://emscripten.org/docs/porting/connecting_cpp_and_javascript/embind.html) - obustronne mieszanie **JavaScript** z **C++**.

+ Ikona strony internetowej: Umieścić plik _favicon.ico_ w katalogu głównym aplikacji (obok pliku HTML).

+ [TODO: Obrazek przy ładowaniu aplikacji]


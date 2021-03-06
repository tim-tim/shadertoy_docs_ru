# Создание шейдеров в ShaderToy

## О ShaderToy

Shadertoy.com - это онлайн-сообщество и платформа для профессионалов компьютерной графики, академиков и энтузиастов, которые делятся, учатся и экспериментируют с техниками рендеринга и процедурным искусством с помощью кода GLSL. WebGL позволяет Shadertoy получить доступ к вычислительной мощности GPU для создания процедурного искусства, анимации, моделей, освещения, логики на основе состояний и звука.

## Инструментарий

Существуют несколько способов разработки ShaderToy GLSL шейдеров:

1. Интерактивный веб-редактор, доступный на [официальном странице](www.shadertoy.com) проекта ShaderToy.

2. IDE Microsoft VS Code c раcширением ShaderToy (`stevensona.shader-toy`) от Adam Stevenson, которое добавляет возможность просмотра шейдера в дополнительной панели (без отладки). 

    > Существует также возможность подключения отладчика(debugger) SHADERed к VSCode с помощью расширения `dfranx.shadered
` для того, чтобы иметь возможность отладки и прототипирования шейдеров.

3. [SHADERed](https://shadered.org/) — специализированная среда разработки (IDE), посвященная исключительно разработке шейдеров на языках GLSL/HLSL, а также их отладке (имеется встроенный отладчик).

## Основы ShaderToy

ShaderToy позволяет писать фрагментные("пиксельные") шейдеры в GLSL-стиле с учетом собственных особенностей работы ST.

Сначала рассмотрим работу над шейдерами в веб-редакторе ShaderToy, а поздее — покажем, как перенести разработку в VSCode.

### Hello, ShaderToy

Простейший фрагментный шейдер, который предлагается нам в качестве "Hello, world!" шейдера при открытии веб-редактора, содержит следующий код:

```c++
void mainImage( out vec4 fragColor, in vec2 fragCoord )
{
    //Нормализованные координаты пикселя (от 0 до 1)
    vec2 uv = fragCoord/iResolution.xy;

    // Время, варьирующее цвет пикселя
    vec3 col = 0.5 + 0.5*cos(iTime+uv.xyx+vec3(0,2,4));

    // Вывод на экран
    fragColor = vec4(col,1.0);
}
```

В это примере весь ST код представлен определением функции фрагментного шейдера, который имеет сигнатуру:

```c++
void mainImage(out vec4, in vec2)
```

В качестве аргументов функции указываем:

- `fragColor` – возвращаемый цвет фрагмента/пикселя;
- `fragCoord` – координаты фрагмента, передаваемого из вершинного шейдера.

А весь код внутри функции используется для различных преобразований каждого пикселя для получеия финальной "картинки" согласно задумке/цели/видению автора шейдера.

Тело функции начинается со строки кода, в которой происходит *нормализация*, "вмещение" коодринат фрагмента в рисуемую область "экрана":

```c++
 vec2 uv = fragCoord/iResolution.xy;
```

Здесь:

- `iResolution` – это один из предопределенных *uniform* входов шейдера, представляющий разрешение вьюпорта в пикселях. Подробнее все возможные входы будут рассмотрены ниже.

Далее следует арифметическое выражение для вычисления изменения цвета во времени с помощью использования в нем ещё одного шейдерного входа — `iTime` (*float* время работы шейдера в секундах).

```c++
// Время, варьирующее цвет пикселя
    vec3 col = 0.5 + 0.5*cos(iTime+uv.xyx+vec3(0,2,4));
```

Последней строкой мы устанавливаем финальный цвет пикселя в виде 4-х компонентного вектора RGBA (на деле альфа-канал игнорируется при выводе, но мы обязаны его предоставить):

```c++
// Вывод на экран
    fragColor = vec4(col,1.0);
```

Таким образом фрагментный шейдер обрабатывает каждый пиксель будущего изображения и формурует финальный результат согласно заложенной логике.

Использование процедурных алгоритмов, готовых файлов изображений и видео, клавиатурного ввода и мыши, звука и иных доступных источников входного "сигнала", которые вы можете использовать в Shadertoy, позволяют создавать довольно сложные, зрелищные эффекты и демо-сцены.

## Входные данные шейдера (Shader Inputs)

В качестве "глобальных" *uniform* переменных доступны следующие входы для шейдера Shadertoy:

| тип         | идентификатор           | назначение                                                   |      |
| ----------- | ----------------------- | ------------------------------------------------------------ | ---- |
| `vec3`      | `iResolution`           | вьюпорт: (ширина в пикс., высота в пикс., пропорция пикселя) |      |
| `float`     | `iTime`                 | время "работы"(playback) шейдера (в секундах)                |      |
| `float`     | `iTimeDelta`            | время визуализации кадра (в секундах)                        |      |
| `int`       | `iFrame`                | порядковый номер кадра воспроизведения шейдера               |      |
| `float`     | `iChannelTime[4]`       | время воспроизведения канала (в секундах)                    |      |
| `vec3`      | `iChannelResolution[4]` | разрешение канала (в пикселях)                               |      |
| `vec4`      | `iMouse`                | координаты курсора мыши в пикс. <br />xy: текущие координаты (если ЛКМ нажата)<br />zw: клик |      |
| `samplerXX` | `iChannel0..3`          | входящий канал, где XX — 2D или Cube                         |      |
| `vec4`      | `iDate`                 | (год, месяц, день, время в секундах)                         |      |
| `float`     | `iSampleRate`           | частота дискретизации аудио (например, 44100)                |      |

Та же самая информация на английском языке, которую можно найти в заголовке редактора кода в раскрываемом свитке "Shader Inputs":

```c++
uniform vec3      iResolution;           // viewport resolution (in pixels)
uniform float     iTime;                 // shader playback time (in seconds)
uniform float     iTimeDelta;            // render time (in seconds)
uniform int       iFrame;                // shader playback frame
uniform float     iChannelTime[4];       // channel playback time (in seconds)
uniform vec3      iChannelResolution[4]; // channel resolution (in pixels)
uniform vec4      iMouse;                // mouse pixel coords. xy: current (if MLB down), zw: click
uniform samplerXX iChannel0..3;          // input channel. XX = 2D/Cube
uniform vec4      iDate;                 // (year, month, day, time in seconds)
uniform float     iSampleRate;           // sound sample rate (i.e., 44100)
```

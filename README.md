[![Foo](https://img.shields.io/badge/Version-2.0-brightgreen.svg?style=flat-square)](#versions)
[![Foo](https://img.shields.io/badge/Website-AlexGyver.ru-blue.svg?style=flat-square)](https://alexgyver.ru/)
[![Foo](https://img.shields.io/badge/%E2%82%BD$%E2%82%AC%20%D0%9D%D0%B0%20%D0%BF%D0%B8%D0%B2%D0%BE-%D1%81%20%D1%80%D1%8B%D0%B1%D0%BA%D0%BE%D0%B9-orange.svg?style=flat-square)](https://alexgyver.ru/support_alex/)

[![Foo](https://img.shields.io/badge/README-ENGLISH-brightgreen.svg?style=for-the-badge)](https://github-com.translate.goog/GyverLibs/EncButton?_x_tr_sl=ru&_x_tr_tl=en)

# EncButton
Лёгкая и функциональная библиотека для энкодера, энкодера с кнопкой или просто кнопки с Arduino
- Максимально быстрое чтение пинов для AVR (ATmega328/ATmega168, ATtiny85/ATtiny13)
- Быстрые оптимизированные алгоритмы опроса действий с кнопки/энкодера
- Лёгкий вес во Flash и SRAM памяти
- Энкодер: обычный поворот, нажатый поворот, быстрый поворот, доступ к счётчику
    - Поддержка двух типов инкрементальных энкодеров (полношаговый и полушаговый)
    - Высокоточный алгоритм определения позиции, работает даже с некачественными энкодерами
- Кнопка: антидребезг, нажатие, отпускание, клик, несколько кликов, счётчик кликов, удержание, импульсное удержание
- Ручной опрос действий или режим функций-обработчиков
- Виртуальный режим (кнопка, энкодер, энкодер с кнопкой)
- Оптимизирована для работы на прерываниях

### Совместимость
Совместима со всеми Arduino платформами (используются Arduino-функции)

## Содержание
- [Установка](#install)
- [Введение](#base)
- [Инициализация](#init)
- [Использование](#usage)
- [Пример](#example)
- [Версии](#versions)
- [Баги и обратная связь](#feedback)

<a id="install"></a>
## Установка
- Библиотеку можно найти по названию **EncButton** и установить через менеджер библиотек в:
    - Arduino IDE
    - Arduino IDE v2
    - PlatformIO
- [Скачать библиотеку](https://github.com/GyverLibs/EncButton/archive/refs/heads/main.zip) .zip архивом для ручной установки:
    - Распаковать и положить в *C:\Program Files (x86)\Arduino\libraries* (Windows x64)
    - Распаковать и положить в *C:\Program Files\Arduino\libraries* (Windows x32)
    - Распаковать и положить в *Документы/Arduino/libraries/*
    - (Arduino IDE) автоматическая установка из .zip: *Скетч/Подключить библиотеку/Добавить .ZIP библиотеку…* и указать скачанный архив
- Читай более подробную инструкцию по установке библиотек [здесь](https://alexgyver.ru/arduino-first/#%D0%A3%D1%81%D1%82%D0%B0%D0%BD%D0%BE%D0%B2%D0%BA%D0%B0_%D0%B1%D0%B8%D0%B1%D0%BB%D0%B8%D0%BE%D1%82%D0%B5%D0%BA)

<a id="base"></a>
## Введение
### Оптимизация
Библиотека универсальная:
- Может работать с "физической" кнопкой, энкодером или энкодером с кнопкой
- Может работать с "виртуальной" кнопкой, энкодером или энкодером с кнопкой
- Может работать в двух режимах: ручной опрос действий или режим с функциями-обработчиками

Библиотека сильно оптимизирована: главные настройки задаются как `<параметры шаблона>`, поэтому в зависимости от выбранного железа и режима работы весь лишний код жёстко вырезается компилятором и программа занимает минимальный объём памяти.

### Версия библиотеки
- **EncButton** - хранит номера пинов как параметры шаблона, обеспечивает максимальную скорость работы и лёгкий вес. Могут возникнуть сложности с динамическим созданием объекта, также нельзя создать массив объектов.
- **EncButton2** - хранит номера пинов как члены класса. Можно создавать динамически, делать массивы, а пины задавать позже в программе. Отличается от **EncButton** только инициализацией!

### Железо
Для работы по сценарию "энкодер с кнопкой" рекомендую вот такие ([ссылка](https://ali.ski/CYir4), [ссылка](https://ali.ski/49q5hy)) круглые китайские модули с распаянными цепями антидребезга:  
![scheme](/doc/encAli.png)

Самостоятельно обвязать энкодер можно по следующей схеме (RC фильтры на каналы энкодера + подтяжка всех пинов к VCC):  
![scheme](/doc/enc.png)

### Тип энкодера
Существует два типа инкрементальных энкодеров, я назвал их полношаговыми и полушаговыми. Полношаговый даёт 4 сигнала на один щелчок, полушаговый - 2. 
Если библиотека выдаёт один "поворот" при двух щелчках энкодера - смените тип на `EB_HALFSTEP`.

### Производительность
Время холостого выполнения функции tick() при реальном устройстве (кнопка/энкодер подключены к пинам МК) на ATmega328, библиотека EncButton:
| Режим | Время, мкс |
| ----- | ---------- |
| Энкодер + кнопка | 3.8 |
| Энкодер | 2.4 |
| Кнопка | 1.9 |

*Для сравнения, стандартный digitalRead() на AVR выполняется 3.5 us*  

### Сравнение с аналогами
- EncButton в режиме кнопки на 6 мкс быстрее, на ~450 байт Flash и 12 байт SRAM легче моей старой библиотеки [GyverButton](https://github.com/GyverLibs/GyverButton), имея при этом больше возможностей
- EncButton в режиме энкодера с кнопкой на 6 мкс быстрее, на ~400 байт Flash и 18 байт SRAM легче моей старой библиотеки [GyverEncoder](https://github.com/GyverLibs/GyverEncoder), имея при этом больше возможностей

<a id="init"></a>
## Инициализация
Библиотека может работать в двух режимах (указывается при инициализации):
- `EB_TICK` - опрос действий кнопки/энкодера вручную в основном цикле программы. Оптимальный вариант.
- `EB_CALLBACK` - подключение своих функций-обработчиков на нужные действия. Добавляет 28Б SRAM на каждый объект.

Подробнее ниже в главе "Использование"

<details>
<summary>Инициализация EncButton</summary>

```cpp
// ============ ПОДТЯЖКА ============
// При инициализации можно указать режим работы ВСЕХ пинов
// По умолч. INPUT_PULLUP. Если используется внешняя подтяжка - лучше перевести в INPUT
EncButton<...> enc(pinmode);
// pinmode: INPUT или INPUT_PULLUP

// ========= ФИЗИЧЕСКАЯ ЖЕЛЕЗКА ========
// Тип железки задаётся количеством указанных пинов:
EncButton<MODE, A, B, KEY> enc;     // энкодер с кнопкой
EncButton<MODE, A, B> enc;          // просто энкодер
EncButton<MODE, KEY> btn;           // просто кнопка
// A, B, KEY: номера пинов
// MODE: EB_TICK или EB_CALLBACK - режим работы ручной или с обработчиками

// ========= ВИРТУАЛЬНЫЙ РЕЖИМ =========
EncButton<MODE, VIRT_BTN> enc;     // виртуальная кнопка
EncButton<MODE, VIRT_ENCBTN> enc;  // виртуальный энк с кнопкой
EncButton<MODE, VIRT_ENC> enc;     // виртуальный энк
// MODE: EB_TICK или EB_CALLBACK - режим работы ручной или с обработчиками
```
</details>
<details>
<summary>Инициализация EncButton2</summary>

```cpp
// ================ TICK ===============
EncButton2<EB_ENCBTN> enc(pinmode, A, B, KEY);    // энкодер с кнопкой
EncButton2<EB_ENC> enc(pinmode, A, B);            // просто энкодер
EncButton2<EB_BTN> enc(pinmode, KEY);             // просто кнопка
// pinmode - режим пинов INPUT/INPUT_PULLUP
// A, B, KEY: номера пинов

// ============== CALLBACK =============
EncButton2<EB_ENCBTN, EB_CALLBACK> enc(pinmode, A, B, KEY);    // энкодер с кнопкой
EncButton2<EB_ENC, EB_CALLBACK> enc(pinmode, A, B);            // просто энкодер
EncButton2<EB_BTN, EB_CALLBACK> enc(pinmode, KEY);             // просто кнопка
// pinmode - режим пинов INPUT/INPUT_PULLUP
// A, B, KEY: номера пинов

// ============== VIRT TICK ============
EncButton2<VIRT_ENCBTN> enc;        // энкодер с кнопкой
EncButton2<VIRT_ENC> enc;           // просто энкодер
EncButton2<VIRT_BTN> enc;           // просто кнопка

// ============ VIRT CALLBACK ==========
EncButton2<VIRT_ENCBTN, EB_CALLBACK> enc;   // энкодер с кнопкой
EncButton2<VIRT_ENC, EB_CALLBACK> enc;      // просто энкодер
EncButton2<VIRT_BTN, EB_CALLBACK> enc;      // просто кнопка
```
</details>
<details>
<summary>Массив экземпляров EncButton2</summary>

```cpp
EncButton2<EB_ENCBTN> enc[количество];
EncButton2<EB_ENC> enc[количество];
EncButton2<EB_BTN> enc[количество];

EncButton2<EB_ENCBTN, EB_CALLBACK> enc[количество];
EncButton2<EB_ENC, EB_CALLBACK> enc[количество];
EncButton2<EB_BTN, EB_CALLBACK> enc[количество];
// и так далее

// Задавать пины можно через setPins()
setPins(uint8_t mode, uint8_t P1, uint8_t P2, uint8_t P3);
// mode - INPUT/INPUT_PULLUP (для всех пинов)
// указываем только нужные для выбранного режима пины:
// EB_ENCBTN - A, B, KEY
// EB_ENC - A, B
// EB_BTN - KEY
// см. пример EucButton2_array
```
</details>

## Документация
<details>
<summary>ПОЛНОЕ ОПИСАНИЕ КЛАССА</summary>

```cpp
// =============== SETTINGS ==============
void setButtonLevel(bool level);    // уровень кнопки: LOW - кнопка подключает GND (по умолч.), HIGH - кнопка подключает VCC
void setHoldTimeout(int tout);      // установить время удержания кнопки, мс (64.. 8 000, шаг 64 мс)
void setStepTimeout(int tout);      // установить период импульсов step, мс (32.. 4 000, шаг 32 мс)

void holdEncButton(bool state);     // виртуально зажать кнопку энкодера (для срабатывания нажатых поворотов)
void setEncReverse(bool rev);       // true - инвертировать направление энкодера (умолч. false)
void setEncType(bool type);         // тип энкодера: EB_FULLSTEP (0) по умолч., EB_HALFSTEP (1) если энкодер делает один поворот за два щелчка

// ================= TICK ================
// тикер, вызывать в главном цикле программы loop(). Здесь происходит опрос пинов и все расчёты
uint8_t tick();

// виртуальный тикер может принимать виртуальный сигнал при режиме VIRT_xxx:
uint8_t tick(uint8_t s1 = 0, uint8_t s2 = 0, uint8_t key = 0);
// (сигнал кнопки)
// (сигнал энкодера А, сигнал энкодера B)
// (сигнал энкодера А, сигнал энкодера B, сигнал кнопки)

// Тикер для вызова в прерывании. Требует наличия обычного tick() в loop() (см. примеры tickISR и callbackISR)
uint8_t tickISR();

// все тикеры возвращают отличное от нуля значение, если произошло какое то событие (см. список статусов ниже)

// проверяет и вызывает подключенные функции для режима callback
// Встроено в tick(), но вынесено отдельной функцией для нестандартных сценариев работы
void checkCallback();

// =============== STATE ================
void resetState();      // сбросить статус
uint8_t getState();     // получить статус кнопки/энкодера
// 0 - idle
// 1 - left
// 2 - right
// 3 - leftH
// 4 - rightH
// 5 - click
// 6 - held
// 7 - step
// 8 - press

// =============== ENCODER ===============
bool turn();            // поворот на один щелчок в любую сторону
bool turnH();           // поворот на один щелчок в любую сторону с зажатой кнопкой
bool fast();            // быстрый поворот на один щелчок в любую сторону
bool right();           // поворот на один щелчок направо
bool left();            // поворот на один щелчок налево
bool rightH();          // поворот на один щелчок направо с зажатой кнопкой
bool leftH();           // поворот на один щелчок налево с зажатой кнопкой
int8_t dir();           // направление последнего поворота, 1 или -1
int16_t counter;        // доступ к счётчику энкодера

// ================ BUTTON ================
bool busy();                      // вернёт true, если всё ещё нужно вызывать tick для опроса таймаутов
bool state();                     // текущее состояние кнопки (true нажата, false не нажата)
bool press();                     // кнопка была нажата
bool release();                   // кнопка была отпущена
bool click();                     // клик (нажата и отпущена)
bool held();                      // кнопка была удержана
bool held(uint8_t clicks);        // кнопка была удержана с предварительным накликиванием
bool hold();                      // кнопка удерживается
bool hold(uint8_t clicks);        // кнопка удерживается с предварительным накликиванием
bool step();                      // режим импульсного удержания
bool step(uint8_t clicks);        // режим импульсного удержания с предварительным накликиванием
bool releaseStep();               // отпущена после режима step
bool releaseStep(uint8_t clicks); // отпущена после режима step с предварительным накликиванием
uint8_t hasClicks();              // вернёт количество кликов, если они есть
bool hasClicks(uint8_t num);      // проверка на наличие указанного количества кликов
uint8_t clicks;                   // доступ к счётчику кликов

// =============== CALLBACK ===============
void attach(TYPE, void (*handler)());       // подключить обработчик
void detach(TYPE);                          // отключить обработчик
void attachClicks(uint8_t amount, handler); // подключить обработчик на количество кликов (может быть только один), до 16 кликов!
void detachClicks();                        // отключить обработчик на количество кликов

// TYPE может быть:
TURN_HANDLER
TURN_H_HANDLER
RIGHT_HANDLER
LEFT_HANDLER
RIGHT_H_HANDLER
LEFT_H_HANDLER
CLICK_HANDLER
HOLDED_HANDLER
STEP_HANDLER
HOLD_HANDLER
CLICKS_HANDLER
PRESS_HANDLER
RELEASE_HANDLER

// =========== Дополнительно EncButton2 ===========
// настроить пины
void setPins(uint8_t mode, uint8_t P1, uint8_t P2, uint8_t P3);
// mode - INPUT/INPUT_PULLUP (для всех пинов)
// пины - указываем только нужные для выбранного режима:
// EB_ENCBTN - (A, B, KEY)
// EB_ENC - (A, B)
// EB_BTN - (KEY)


// =============== ДЕФАЙНЫ НАСТРОЕК ===============
// дефайнить ПЕРЕД ПОДКЛЮЧЕНИЕМ БИБЛИОТЕКИ, показаны значения по умолчанию
#define EB_FAST 30      // таймаут быстрого поворота энкодера, мс
#define EB_DEB 50       // дебаунс кнопки, мс
#define EB_CLICK 400    // таймаут накликивания кнопки, мс
```
</details>

<a id="usage"></a>
## Особенности и сценарии использования
### Опрос пинов
Опрос пинов энкодера/кнопки, расчёт таймаутов и вызов коллбэков происходит внутри функции `tick()`. Эту функцию нужно **однократно** вызывать в главном цикле программы `loop()`.
```cpp
void loop() {
  btn.tick();
}
```

### Режим EB_TICK
В этом режиме нужно вручную опрашивать действия с энкодера/кнопки при помощи `if ()`:
- По логике работы все опросы нужно располагать после вызова `tick()`
- Почти все функции опроса имеют механизм "однократного срабатывания", то есть возвращают `true` и автоматически сбрасываются в `false` до наступления следующего события.

Пример:
```cpp
void loop() {
  btn.tick();
  if (btn.click()) Serial.println("Click");
}
```

#### Кнопка
- `press()` - кнопка была нажата. *[однократно вернёт true]*
- `release()` - кнопка была отпущена. *[однократно вернёт true]*
- `click()` - кнопка была кликнута, т.е. нажата и отпущена до таймаута удержания. *[однократно вернёт true]*
- `held()` - кнопка была удержана дольше таймаута удержания. *[однократно вернёт true]*
- `held(clicks)` - то же самое, но функция принимает количество кликов, сделанных до удержания. Примечание: held() без аргумента перехватит вызов! См. пример *preClicks*. *[однократно вернёт true]*
- `hold()` - кнопка была удержана дольше таймаута удержания. *[возвращает true, пока удерживается]*
- `hold(clicks)` - то же самое, но функция принимает количество кликов, сделанных до удержания. Примечание: hold() без аргумента перехватит вызов! См. пример *preClicks*. *[возвращает true, пока удерживается]*
- `step()` - режим "импульсного удержания": после удержания кнопки дольше таймаута данная функция *[возвращает true с периодом EB_STEP]*. Удобно использовать для пошагового изменения переменных: `if (btn.step()) val++;`.
- `step(clicks)` - то же самое, но функция принимает количество кликов, сделанных до удержания. Примечание: step() без аргумента перехватит вызов! См. пример *StepMode* и *preClicks*.
- `releaseStep()` - кнопка была отпущена после импульсного удержания. Может использоваться для изменения знака инкремента переменной. См. пример *StepMode*. *[однократно вернёт true]*
- `releaseStep(clicks)` - то же самое, но функция принимает количество кликов, сделанных до удержания. Примечание: releaseStep() без аргумента перехватит вызов! См. пример *StepMode* и *preClicks*. *[однократно вернёт true]*
- `hasClicks(clicks)` - было сделано указанное количество кликов с периодом менее *EB_CLICK*. *[однократно вернёт true]*
- `state()` - возвращает теукщее состояние кнопки (сигнал с пина, без антидребезга): `true` - нажата, `false` - не нажата.
- `busy()` - вернёт `true`, если всё ещё нужно вызывать tick для опроса таймаутов
- `hasClicks()` - вернёт количество кликов, сделанных с периодом менее *EB_CLICK*. В противном случае вернёт 0.
- `uint8_t clicks` - публичная переменная, хранит количество сделанных кликов с периодом менее *EB_CLICK*. Сбрасывается в 0 после нового клика.
![diagram](/doc/diagram.png)

#### Энкодер
- `turn()` - поворот на один щелчок в любую сторону. *[однократно вернёт true]*
- `turnH()` - поворот на один щелчок в любую сторону с зажатой кнопкой. *[однократно вернёт true]*
- `fast()` - был совершён быстрый поворот (с периодом менее *EB_FAST* мс) на один щелчок в любую сторону. *[возвращает true, пока энкодер крутится быстро]*
- `right()` - поворот на один щелчок направо. *[однократно вернёт true]*
- `left()` - поворот на один щелчок налево. *[однократно вернёт true]*
- `rightH()` - поворот на один щелчок направо с зажатой кнопкой. *[однократно вернёт true]*
- `leftH()` - поворот на один щелчок налево с зажатой кнопкой. *[однократно вернёт true]*
- `dir()` - направление последнего поворота, 1 или -1.
- `int16_t counter` - публичная переменная, хранит счётчик энкодера.

### Режим callback
В данном режиме можно подключить свою функцию-обработчик на любое действие кнопки/энкодера. Функции будут автоматически вызываться при наступлении события. 
Для работы также нужно вызывать `tick()` в основном цикле программы, в нём происходит чтение и обработка сигналов. Отсюда же вызываются подключенные функции-обработчики. Смотри пример *callbackMode*

```cpp
void attach(type, func);                   // подключить обработчик
void detach(type);                         // отключить обработчик
void attachClicks(uint8_t amount, func);   // подключить обработчик на количество кликов (может быть только один), до 16 кликов!
void detachClicks();                       // отключить обработчик на количество кликов
```

Где `type` - тип события:
- `TURN_HANDLER` - поворот
- `TURN_H_HANDLER` - нажатый поворот
- `RIGHT_HANDLER` - поворот направо
- `LEFT_HANDLER` - поворот налево
- `RIGHT_H_HANDLER` - нажатый поворот направо
- `LEFT_H_HANDLER` - нажатый поворот налево
- `PRESS_HANDLER` - нажатие
- `RELEASE_HANDLER` - отпускание
- `CLICK_HANDLER` - клик
- `HOLDED_HANDLER` - удержание (однократное срабатывание)
- `HOLD_HANDLER` - удержание (постоянное срабатывание)
- `STEP_HANDLER` - импульсное удержание
- `CLICKS_HANDLER` - несколько кликов

Пример:
```cpp
void setup() {
  btn.attach(CLICK_HANDLER, myClick);
}

void myClick() {
  Serial.println("Click");
}

void loop() {
  btn.tick();
}
```

### Виртуальный режим
Виртуальный режим позволяет получить все возможности библиотеки EncButton в ситуациях, когда кнопка не подключена напрямую к микроконтроллеру, либо для её опроса используется другая библиотека:
- Аналоговая клавиатура (например через библиотеку [AnalogKey](https://github.com/GyverLibs/AnalogKey)). Смотри пример *virtual_AnalogKey*
- Матричная клавиатура (например через библиотеку [SimpleKeypad](https://github.com/maximebohrer/SimpleKeypad)). Смотри пример *virtual_SimpleKeypad* и *virtual_SimpleKeypad_array*
- Кнопки или энкодеры, подключенные через расширители пинов или сдвиговые регистры  

Таким образом можно получить несколько нажатий с матричной клавиатуры, удержание кнопок матричной клавиатуры, импульсное удержание и прочие фишки EncButton.  

Для работы нужно выбрать тип при инициализации и передать в `tick()` текущие состояния "пинов" кнопки/энкодера как `bool` переменную:
- Кнопка - сигнал кнопки: `tick(btn)`
- Энкодер - сигнал энкодера А, сигнал энкодера B: `tick(A, B)`
- Энкодер с кнопкой - сигнал энкодера А, сигнал энкодера B, сигнал кнопки: `tick(A, B, btn)`

### Оптимизация
`tick()` возвращает текущий статус энкодера/кнопки:
- 0 - никаких действий не было
- 1 - left + turn
- 2 - right + turn
- 3 - leftH + turnH
- 4 - rightH + turnH
- 5 - click
- 6 - held
- 7 - step
- 8 - press

Это позволяет слегка оптимизировать программу: производить дальнейший опрос действий кнопки/энкодера только по факту их совершения: 
можно поместить весь опрос в блок `if (enc.tick()) {}`. В конце рекомендуется вызвать `resetState()` для сборса неопрошенных флагов, 
чтобы `tick()` перестал "сигналить" о действии. Подробнее смотри в примере *optimisation*.

### Прерывания
Для повышения качества обработки энкодера/кнопки в загруженной программе (чтобы не пропустить поворот или клик) рекомендуется 
продублировать опрос в прерывании по *CHANGE*. Для энкодера - на оба пина. Внутри обработчика прерывания вызываем специальный тикер `tickISR()`, 
а в основном цикле программы оставляем обычный `tick()` - он нужен для того, чтобы корректно считались все таймауты. 
В режиме `EB_CALLBACK` все подключенные функции вызываются из `tick()`, то есть не из обработчика прерывания, а из основного цикла программы!

Также есть функция `busy()` - она возвращает `true`, пока обработка таймаутов нажатия не завершена, т.е. всё ещё нужно обязательно вызывать `tick()` в `loop()`. 

<a id="example"></a>
## Примеры
Остальные примеры смотри в **examples**!

### Полное демо EB_TICK
```cpp
// Опциональные дефайн-настройки (показаны по умолчанию)
//#define EB_FAST 30     // таймаут быстрого поворота, мс
//#define EB_DEB 50      // дебаунс кнопки, мс
//#define EB_CLICK 400   // таймаут накликивания, мс

#include <EncButton.h>
EncButton<EB_TICK, 2, 3, 4> enc;  // энкодер с кнопкой <A, B, KEY>
//EncButton<EB_TICK, 2, 3> enc;     // просто энкодер <A, B>
//EncButton<EB_TICK, 4> enc;        // просто кнопка <KEY>

void setup() {
  Serial.begin(9600);
  //enc.setButtonLevel(HIGH);     // уровень кнопки: LOW - кнопка подключает GND (по умолч.), HIGH - кнопка подключает VCC
  //enc.setHoldTimeout(1000);     // установить время удержания кнопки, мс (до 8 000)
  //enc.setStepTimeout(500);      // установить период импульсов step, мс (до 4 000)

  //enc.holdEncButton(true);      // виртуально зажать кнопку энкодера (для срабатывания нажатых поворотов)
  //enc.setEncReverse(true);      // true - инвертировать направление энкодера (умолч. false)
  //enc.setEncType(EB_HALFSTEP);  // тип энкодера: EB_FULLSTEP (0) по умолч., EB_HALFSTEP (1) если энкодер делает один поворот за два щелчка
}

void loop() {
  enc.tick();                     // опрос происходит здесь

  // =============== ЭНКОДЕР ===============
  // обычный поворот
  if (enc.turn()) {
    Serial.println("turn");

    // можно ещё:
    Serial.println(enc.counter);  // вывести счётчик
    Serial.println(enc.fast());   // проверить быстрый поворот
    Serial.println(enc.dir());    // вывести направление поворота
  }

  // "нажатый поворот"
  if (enc.turnH()) Serial.println("hold + turn");

  if (enc.left()) Serial.println("left");     // поворот налево
  if (enc.right()) Serial.println("right");   // поворот направо
  if (enc.leftH()) Serial.println("leftH");   // нажатый поворот налево
  if (enc.rightH()) Serial.println("rightH"); // нажатый поворот направо

  // =============== КНОПКА ===============
  if (enc.press()) Serial.println("press");
  if (enc.click()) Serial.println("click");
  if (enc.release()) Serial.println("release");

  if (enc.held()) Serial.println("held");     // однократно вернёт true при удержании
  //if (enc.hold()) Serial.println("hold");   // будет постоянно возвращать true после удержания
  if (enc.step()) Serial.println("step");     // импульсное удержание
  if (enc.releaseStep()) Serial.println("release step");  // отпущена после импульсного удержания
  
  // проверка на количество кликов
  if (enc.hasClicks(1)) Serial.println("action 1 clicks");
  if (enc.hasClicks(2)) Serial.println("action 2 clicks");

  // вывести количество кликов
  if (enc.hasClicks()) {
    Serial.print("has clicks ");
    Serial.println(enc.clicks);
  }
}
```

### Массив кнопок EncButton2
```cpp
// объявляем массив кнопок
#define BTN_AMOUNT 5
#include <EncButton2.h>
EncButton2<EB_BTN> btn[BTN_AMOUNT];

void setup() {
  Serial.begin(9600);
  btn[0].setPins(INPUT_PULLUP, D3);
  btn[1].setPins(INPUT_PULLUP, D2);
}

void loop() {
  for (int i = 0; i < BTN_AMOUNT; i++) btn[i].tick();
  for (int i = 0; i < BTN_AMOUNT; i++) {
    if (btn[i].click()) {
      Serial.print("click btn: ");
      Serial.println(i);
    }
  }
}
```

### Одна кнопка управляет несколькими переменными
```cpp
// используем одну КНОПКУ для удобного изменения трёх переменных
// первая - один клик, затем удержание (нажал-отпустил-нажал-держим)
// вторая - два клика, затем удержание
// третья - три клика, затем удержание
// смотри монитор порта

#include <EncButton.h>
EncButton<EB_TICK, 3> btn;

// переменные для изменения
int val_a, val_b, val_c;

// шаги изменения (signed)
int8_t step_a = 1;
int8_t step_b = 5;
int8_t step_c = 10;

void setup() {
  Serial.begin(9600);
}

void loop() {
  btn.tick();

  // передаём количество предварительных кликов
  if (btn.step(1)) {
    val_a += step_a;
    Serial.print("val_a: ");
    Serial.println(val_a);
  }
  if (btn.step(2)) {
    val_b += step_b;
    Serial.print("val_b: ");
    Serial.println(val_b);
  }
  if (btn.step(3)) {
    val_c += step_c;
    Serial.print("val_c: ");
    Serial.println(val_c);
  }

  // разворачиваем шаг для изменения в обратную сторону
  // передаём количество предварительных кликов
  if (btn.releaseStep(1)) step_a = -step_a;
  if (btn.releaseStep(2)) step_b = -step_b;
  if (btn.releaseStep(3)) step_c = -step_c;
}
```

<a id="versions"></a>
## Версии
- v1.1 - пуллап отдельныи методом
- v1.2 - можно передать конструктору параметр INPUT_PULLUP / INPUT(умолч)
- v1.3 - виртуальное зажатие кнопки энкодера вынесено в отдельную функцию + мелкие улучшения
- v1.4 - обработка нажатия и отпускания кнопки
- v1.5 - добавлен виртуальный режим
- v1.6 - оптимизация работы в прерывании
- v1.6.1 - подтяжка по умолчанию INPUT_PULLUP
- v1.7 - большая оптимизация памяти, переделан FastIO
- v1.8 - индивидуальная настройка таймаута удержания кнопки (была общая на всех)
- v1.8.1 - убран FastIO
- v1.9 - добавлена отдельная отработка нажатого поворота и запрос направления
- v1.10 - улучшил обработку released, облегчил вес в режиме callback и исправил баги
- v1.11 - ещё больше всякой оптимизации + настройка уровня кнопки
- v1.11.1 - совместимость Digispark
- v1.12 - добавил более точный алгоритм энкодера EB_BETTER_ENC
- v1.13 - добавлен экспериментальный EncButton2
- v1.14 - добавлена releaseStep(). Отпускание кнопки внесено в дебаунс
- v1.15 - добавлен setPins() для EncButton2
- v1.16 - добавлен режим EB_HALFSTEP_ENC для полушаговых энкодеров
- v1.17 - добавлен step с предварительными кликами
- v1.18 - не считаем клики после активации step. held() и hold() тоже могут принимать предварительные клики. Переделан и улучшен дебаунс
- v1.18.1 - исправлена ошибка в releaseStep() (не возвращала результат)
- v1.18.2 - fix compiler warnings
- v1.19 - оптимизация скорости, уменьшен вес в sram
- v1.19.1 - ещё чутка увеличена производительность
- v1.19.2 - ещё немного увеличена производительность, спасибо XRay3D
- v1.19.3 - сделал высокий уровень кнопки по умолчанию в виртуальном режиме
- v1.19.4 - фикс EncButton2
- v1.20 - исправлена критическая ошибка в EncButton2
- v1.21 - EB_HALFSTEP_ENC теперь работает для обычного режима
- v1.22 - улучшен EB_HALFSTEP_ENC для обычного режима
- v1.23 - getDir() заменил на dir()
- v2.0 
    - Алгоритм EB_BETTER_ENC оптимизирован и установлен по умолчанию, дефайн EB_BETTER_ENC упразднён
    - Добавлен setEncType() для настройки типа энкодера из программы, дефайн EB_HALFSTEP_ENC упразднён
    - Добавлен setEncReverse() для смены направления энкодера из программы
    - Добавлен setStepTimeout() для установки периода импульсного удержания, дефайн EB_STEP упразднён
    - Мелкие улучшения и оптимизация
    
<a id="feedback"></a>
## Баги и обратная связь
При нахождении багов создавайте **Issue**, а лучше сразу пишите на почту [alex@alexgyver.ru](mailto:alex@alexgyver.ru)  
Библиотека открыта для доработки и ваших **Pull Request**'ов!

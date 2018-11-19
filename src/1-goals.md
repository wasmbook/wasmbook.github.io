# Главные цели WebAssembly

1. Определить [преносимый](https://webassembly.org/docs/portability/), эфективный по размеру и времени закгрузки [двоичный формат](https://webassembly.org/docs/mvp/#binary-format), который будет целью для компиляции, и может быть скомпилированым для выолнения на собственной скорости с использованием общих аппаратных возможностей, доступный на широком спекте платформы, включая мобильные устройсвта и интернет вещей

2. Пошаговая спецификация и реализация:
- [Minimum Viable Product](https://webassembly.org/docs/mvp/) (MVP) для стандарта с приблизительно похожей функциональности как в [asm.js](http://asmjs.org/), главным образом направленная на [C/C++](https://webassembly.org/docs/c-and-c++/);
- [дополнительные функции](https://webassembly.org/docs/future-features/) - изначально ориентированные на ключевые функции, такие как [потоки](https://github.com/WebAssembly/design/issues/1073), [исключения с нулевой стоимостью](https://github.com/WebAssembly/design/issues/1078), и [SIMD](https://github.com/WebAssembly/design/issues/1075), а затем дополнительные функции, приоритетные по откхывам и включащие поддержку языков, отличных от C/C++.

3. Спроектировано для выполнения внутри и с интеграцией с существующей [веб-платформой](https://webassembly.org/docs/web/):
- maintain the versionless, [feature-tested](https://webassembly.org/docs/feature-test/) и обратной совсместимости в эволюции в веб;
- исполнение в том же семантическом пространстве, что и JavaScript;
- позволять синхронные вызовы в и из JavaScript;
- осуществить единство проихождения (same-origin) и разрешений политик безопасности;
- доступ к функциональности браузера через теже веб API, которые доступны JavaScript;
- определить редактируемый человеком текстовый формат, который конвертируется в двоичный формат и из него, поддерживая фукнциональность View Source.

4. Проектировно для поддержки [внебразуерного](https://webassembly.org/docs/non-web/) встраивания.

5. Создание великой платформы:
- построить новый LLVM бэкенд для WebAssembly и сопутсвующее портирование clang ([почему LLVM первый?](https://webassembly.org/docs/faq/#which-compilers-can-i-use-to-build-webassembly-programs));
- продвижение других компиляторов и инструментов ориентированных на WebAssembly;
- включить другие полезные [инструменты](https://webassembly.org/docs/tooling/).

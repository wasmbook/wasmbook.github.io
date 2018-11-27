# Экспорт структуры в JS

До сих пор мы рассматривали объекты JS, импортировали функции и экспортировали функции. Это дало нам довольно богатую базу для развития. Хотя мы иногда хотим пойти дальше и определить JS `class` в Rust. Или же другими словами, мы хотим выставить объект с методами из Rust в JS, а не просто импортировать\экспортировать свободные функции.

Атрибут `#[wasm_bindgen]` может анотировать оба блока и `struct` и `impl`, которые позволяют:

```rust
#[wasm_bindgen]
pub struct Foo {
    internal: i32,
}

#[wasm_bindgen]
impl Foo {
    pub fn new(val: i32) -> Foo {
        Foo { internal: val }
    }

    pub fn get(&self) -> i32 {
        self.internal
    }

    pub fn set(&mut self, val: i32) {
        self.internal = val;
    }
}
```

В этом, достаточно типичном для Rust определения `структуры` для типа с конструктооом и несколькими методами. Анотирование структуры с помощью `#[wasm_bindgen]` значит, что мы будем генерировать необходимые трейты для преобразования этого типа в границы или из границ JS. Анотированный блок `impl` здесь означает, что функции внутри также будут сделаны доступны для JS с помощью сгененрированых прокладок. Если мы посмотрим на сгенерированный JS код, то мы увидим:

```js
import * as wasm from './js_hello_world_bg';

export class Foo {
    static __construct(ptr) {
        return new Foo(ptr);
    }

    constructor(ptr) {
        this.ptr = ptr;
    }

    free() {
        const ptr = this.ptr;
        this.ptr = 0;
        wasm.__wbg_foo_free(ptr);
    }

    static new(arg0) {
        const ret = wasm.foo_new(arg0);
        return Foo.__construct(ret)
    }

    get() {
        const ret = wasm.foo_get(this.ptr);
        return ret;
    }

    set(arg0) {
        const ret = wasm.foo_set(this.ptr, arg0);
        return ret;
    }
}
```

Мы можем увить здесь всё то, что мы перевели с Rust на JS:

* Связанные функции в Rust (без `self`) превращаются в `static` функции в JS.
* Методы в Rust превращабтся в методы wasm.
* Ручное управление памятью также отобпражается в JS. Функция `free` является необходимой для вызова для освобождения ресурсов со стороны Rust.

Чтобы иметь возможность использовать `new Foo()`, вы должны аннотировать `new` как `#[wasm_bindgen(constructor)]`.

Однако один важный аспект заключается в том, что после того, как `free` вызывается объектом JS, он "стерилизуется", поскольку его внутренний указатель обнуляется. Это значит, что будущее использование объекта, должно вызывать панику в Rust.

Однако реальная хитрость с этими биндингами заканчивается в Rust, поэтому давайте посмотрим на это.

```rust
// исходный ввод в `#[wasm_bindgen]` опущен ...

#[export_name = "foo_new"]
pub extern fn __wasm_bindgen_generated_Foo_new(arg0: i32) -> u32
    let ret = Foo::new(arg0);
    Box::into_raw(Box::new(WasmRefCell::new(ret))) as u32
}

#[export_name = "foo_get"]
pub extern fn __wasm_bindgen_generated_Foo_get(me: u32) -> i32 {
    let me = me as *mut WasmRefCell<Foo>;
    wasm_bindgen::__rt::assert_not_null(me);
    let me = unsafe { &*me };
    return me.borrow().get();
}

#[export_name = "foo_set"]
pub extern fn __wasm_bindgen_generated_Foo_set(me: u32, arg1: i32) {
    let me = me as *mut WasmRefCell<Foo>;
    ::wasm_bindgen::__rt::assert_not_null(me);
    let me = unsafe { &*me };
    me.borrow_mut().set(arg1);
}

#[no_mangle]
pub unsafe extern fn __wbindgen_foo_free(me: u32) {
    let me = me as *mut WasmRefCell<Foo>;
    wasm_bindgen::__rt::assert_not_null(me);
    (*me).borrow_mut(); // ensure no active borrows
    drop(Box::from_raw(me));
}
```

As with before this is cleaned up from the actual output but it's the same idea
as to what's going on! Here we can see a shim for each function as well as a
shim for deallocating an instance of `Foo`. Recall that the only valid wasm
types today are numbers, so we're required to shoehorn all of `Foo` into a
`u32`, which is currently done via `Box` (like `std::unique_ptr` in C++).
Note, though, that there's an extra layer here, `WasmRefCell`. This type is the
same as [`RefCell`] and can be mostly glossed over.

The purpose for this type, if you're interested though, is to uphold Rust's
guarantees about aliasing in a world where aliasing is rampant (JS).
Specifically the `&Foo` type means that there can be as much aliasing as you'd
like, but crucially `&mut Foo` means that it is the sole pointer to the data
(no other `&Foo` to the same instance exists). The [`RefCell`] type in libstd
is a way of dynamically enforcing this at runtime (as opposed to compile time
where it usually happens). Baking in `WasmRefCell` is the same idea here,
adding runtime checks for aliasing which are typically happening at compile
time. This is currently a Rust-specific feature which isn't actually in the
`wasm-bindgen` tool itself, it's just in the Rust-generated code (aka the
`#[wasm_bindgen]` attribute).

[`RefCell`]: https://doc.rust-lang.org/std/cell/struct.RefCell.html

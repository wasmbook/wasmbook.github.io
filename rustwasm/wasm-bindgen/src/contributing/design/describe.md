# Communicating types to `wasm-bindgen`

The last aspect to talk about when converting Rust/JS types amongst one another
is how this information is actually communicated. The `#[wasm_bindgen]` macro is
running over the syntactical (unresolved) structure of the Rust code and is then
responsible for generating information that `wasm-bindgen` the CLI tool later
reads.

Для этого используется несколько нетрадиционный подход. Статическая информация о структуре кода Rust сереализуется через JSON (в настоящий момент) в кастомнуый раздел исполняемого файла wasm. Другая информация, например какие типы на самом деле есть, к сожалению, не известна до конца в компиляторе из-за таких штук, как связанные проекции типов и typedefs. Также выясняется что мы хотим передать "богатые" типы, такие как `FnMut(String, Foo, &JsValue)` в CLI `wasm-bindgen`, и обработка всего этого очень сложна.

Для решения этой проблемы макрос `#[wasm_bindgen]` генерирует **исполняемые функции**, которые "описывают сигнатуру типа импорта или экспорта". Эти исполняемые функции это как раз всё то о чем говорит `WasmDescribe`:

```rust
pub trait WasmDescribe {
    fn describe();
}
```

While deceptively simple this trait is actually quite important. When you write,
an export like this:

```rust
#[wasm_bindgen]
fn greet(a: &str) {
    // ...
}
```

In addition to the shims we talked about above which JS generates the macro
*also* generates something like:

```
#[no_mangle]
pub extern fn __wbindgen_describe_greet() {
    <Fn(&str)>::describe();
}
```

Or in other words it generates invocations of `describe` functions. In doing so
the `__wbindgen_describe_greet` shim is a programmatic description of the type
layouts of an import/export. These are then executed when `wasm-bindgen` runs!
These executions rely on an import called `__wbindgen_describe` which passes one
`u32` to the host, and when called multiple times gives a `Vec<u32>`
effectively. This `Vec<u32>` can then be reparsed into an `enum Descriptor`
which fully describes a type.

All in all this is a bit roundabout but shouldn't have any impact on the
generated code or runtime at all. All these descriptor functions are pruned from
the emitted wasm file.

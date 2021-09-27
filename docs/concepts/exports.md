:::info
Unlike defining modules in `NodeJS`, we don't need to explicitly register exports like `module.exports.xxx = xxx`.

`#[napi]` macro will automatically generate module registering code for you.
The auto registering idea inspired by [node-bindgen](https://github.com/infinyon/node-bindgen)
:::

### Prelude

Before using

### Function

exports a function is incredibly simple. What you should do is just decorate a normal rust function with `#[napi]` macro:

```rust title=lib.rs
#[napi]
fn sum(a: u32, b: u32) -> u32 {
	a + b
}
```

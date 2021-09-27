Conversion relation between Rust and JavaScript types

### Undefined

Represent `undefined` in JavaScript.

```rust title=lib.rs
#[napi]
fn get_undefined() -> Undefined {
	Undefined
}

// default return or empty tuple `()` are `undefined` after converted into JS value.
#[napi]
fn log(n: u32) {
	println!("{}", n);
}
```

```ts title=type.d.ts
export function getUndefined(): undefined
export function log(n: number): void
```

### Null

Represent `null` value in JavaScript.

```rust title=lib.rs
#[napi]
fn get_null() -> Null {
	Null
}

#[napi]
fn get_env(env: String) -> Option<String> {
	match std::env::var(env) {
		Ok(val) => Some(val),
		Err(e) => None,
	}
}
```

```ts title=type.d.ts
export function getNull(): null
export function getEnv(env: string): string | null
```

### Numbers

JavaScript `Number` type with Rust Int/Float types: `u32`, `i32`, `i64`, `f64`.

For Rust types like `u64`, `u128`, `i128`, checkout [`BigInt`](#bigint) section.

```rust title=lib.rs
fn sum(a: u32, b: i32) -> i64 {
	(b + a as i32).into()
}
```

```ts title=type.d.ts
export function sum(a: number, b: number): number
```

### String

Represent JavaScript `String` type.

```rust title=lib.rs
#[napi]
fn greet(name: String) -> String {
	format!("greeting, {}", name)
}
```

```ts title=type.d.ts
export function greet(name: string): string
```

### Boolean

Represent JavaScript `Boolean` type.

```rust title=lib.rs
#[napi]
fn is_good() -> bool {
	true
}
```

```ts title=type.d.ts
export function isGood(): boolean
```

### Buffer

```rust title=lib.rs
#[napi]
fn with_buffer(buf: Buffer) {
	let buf = Vec::<u8>::from(buf);
	// do something
}

fn read_buffer(file: String): Buffer {
	Buffer::from(std::fs::read(file).unwrap())
}
```

```ts title=type.d.ts
export function withBuffer(buf: Buffer): void
export function readBuffer(file: string): Buffer
```

### Object

Represent JavaScript anonymous object values.

:::caution Performance
The costs of `Object` conversions between JavaScript and Rust are higher than other primitive types.

Every call of `Object.get("key")` is actually dispatched to node side including two steps: fetch value, convert JS to rust value, and so as `Object.set("key", v)`.
:::

```rust title=lib.rs
#[napi]
fn keys(obj: Object) -> Vec<String> {
	Object::keys(&obj).unwrap()
}

#[napi]
fn log_string_field(obj: Object, field: String) {
	println!("{}: {:?}", &field, obj.get::<String>::(field.as_ref()));
}

#[napi]
fn crete_obj(env: Env) -> Object {
	let mut obj = env.create_object();
	obj.set("test", 1).unwrap();
	obj
}
```

```ts title=type.d.ts
export function keys(obj: object): Array<string>
export function logStringField(obj: object): void
export function createObj(): object
```

If you want `napi-rs` convert objects from JavaScript with the same shape defined in Rust, you could use `#[napi]` macro with `object` attribute.

```rust title=lib.rs
/// #[napi(object)] requires all struct fields to be public
#[napi(object)]
struct PackageJson {
	pub name: String,
	pub version: String,
	pub dependencies: Option<HashMap<String, String>>,
	pub dev_dependencies: Option<HashMap<String, String>>,
}

#[napi]
fn log_package_name(package_json: PackageJson) {
	println!("name: {}", package_json.name);
}

#[napi]
fn read_package_json() -> PackageJson {
	// ...
}
```

```ts title=type.d.ts
export interface PackageJson {
  name: string
  version: string
  dependencies: Record<string, string> | null
  devDependencies: Record<string, string> | null
}
export function logPackageName(packageJson: PackageJson)
export function readPackageJson(): PackageJson
```

### Array

Because `Array` values in JavaScript can hold elements with different types, but rust `Vec<T>`
can only contains same type elements. So there two different way for array types.

:::caution Performance
Because JavaScript `Array` type is backed with `Object` actually, so the performance of manipulating `Array`s would be the same as `Object`s.

The conversion between `Array` and `Vec<T>` is even heavier, which is in `O(2n)` complexity.
:::

```rust title=lib.rs
#[napi]
fn arr_len(arr: Array): u32 {
	arr.len()
}

#[napi]
fn get_tuple_array(env: Env) -> Array {
  let mut arr = env.create_array(2).unwrap();

  arr.push(1).unwrap();
  arr.push("test").unwrap();

  arr
}

#[napi]
fn vec_len(nums: Vec<u32>): u32 {
	nums.len()
}

#[napi]
fn get_nums() -> Vec<u32> {
	vec![1, 1, 2, 3, 5, 8]
}
```

```ts title=type.d.ts
export function arrLen(arr: Array<any>): number
export function getTupleArray(): Array<any>
export function vecLen(nums: Array<number>): number
export function getNums(): Array<number>
```

### BigInt

:::caution
UNIMPLEMENTED
:::

### TypedArray

:::caution
UNIMPLEMENTED
:::

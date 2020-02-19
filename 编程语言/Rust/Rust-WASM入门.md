# Rust - WASM 入门

首先安装必要的软件，安装 Rust，带有 Cargo：

```
$ curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

安装 `wasm-pack`，用于编译 WebAssembly 项目：

```
$ curl https://rustwasm.github.io/wasm-pack/installer/init.sh -sSf | sh
```

安装 npm，更新NPM：

```
$ npm install npm@latest -g
```

## 建立 WASM 项目：

建立 Rust 项目

```
$ cargo new --lib wasm-startup
```

然后 在 Cargo.toml 中加入以下代码：

```
[lib]
crate-type = ["cdylib"]

[dependencies]
wasm-bindgen = "0.2.50"
```

lib.rs 改为以下代码：

```rust
use wasm_bindgen::prelude::*;

#[wasm_bindgen]
extern "C" {
    fn alert(s: &str);
}

#[wasm_bindgen]
pub fn greet(name: &str) {
    alert(&format!("Hello, {}!", name));
}
```

然后编译：

```
$ wasm-pack build
```

编译完成后会生成一个 pkg 子目录

生成 node项目：

```
$ npm init wasm-app www
```

这会生成一个 www 的目录，修改其中的 package.json，添加以下代码：

```
"dependencies": {
    "wasm-startup": "file:../pkg"
 }
```

然后修改 index.js 为以下代码：

```
import * as wasm from "wash-startup";

wasm.greet("xujiyou");
```

然后执行以下命令

```
$ cd www
$ npm install
$ npm run start
```

打开 http://localhost:8080 ，大功告成！

如果想更改 Rust 代码了，直接执行 `wasm-pack build` 即可，不必重启 npm。


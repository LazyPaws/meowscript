# MeowScript 🐾

![Made in Vietnam](https://img.shields.io/badge/made%20in-Vietnam-ff0000) ![Self-hosted](https://img.shields.io/badge/self--hosted-100%25-brightgreen) ![Topics](https://img.shields.io/badge/topics-programming--language%20%7C%20compiler%20%7C%20vm-lightgrey)

> **Tagline:** *MeowScript — ngôn ngữ mèo tự nuôi: đủ mạnh để viết chính mình.*

---

## Tổng quan

**Ngôn ngữ lập trình** là cách con người mô tả ý tưởng bằng cú pháp chặt chẽ để máy tính hiểu và thực thi. Thay vì làm việc trực tiếp với dãy số 0–1, lập trình viên viết bằng ngôn ngữ bậc cao hơn; compiler và VM sẽ dịch xuống dạng máy hiểu được.

**Self-hosting** nghĩa là ngôn ngữ đủ trưởng thành để viết compiler (và thường là toolchain) của chính nó. Với MeowScript, toàn bộ compiler và VM đều được viết bằng MeowScript — vừa là bài toán kỹ thuật (vòng lặp gà-và-trứng), vừa là bằng chứng rằng ngôn ngữ đủ mạnh để “tự nuôi sống mình”.

MeowScript là một **ngôn ngữ lập trình self-hosting**: compiler và VM đều bằng `.meow`. Dự án đang được **refactor** để tối ưu hóa **register-based VM** và **bootstrap-interpreter (`meow-alpha`)** — mục tiêu: khi đã chạy thì phải nhanh hơn, mạnh hơn.

> **Lưu ý:** `meow-alpha` nằm ở repo riêng `github.com/LazyPaws/meow-alpha` (hiện **private**) — README này mô tả workflow và cách bạn sẽ dùng `meow-alpha` khi có access/binary.

---

## Trạng thái hiện tại

* Self-hosted 100%: tất cả code compiler + VM bằng `.meow`.
* Đang refactor để tối ưu register allocator, instruction encoding, inline caches, và dispatch performance.
* `meow-alpha` (bootstrap interpreter) — `github.com/LazyPaws/meow-alpha` — **private** (đang refactor).
* `meow-vm` (register-based virtual machine) — `github.com/LazyPaws/meow-vm` — hiện cũng **private**.

---

## Workflow & công cụ

**Ý tưởng chung:**

1. `meow-alpha` (bootstrap interpreter) chạy `src/main.meow` — đây là compiler viết bằng MeowScript — và dùng compiler đó để **dịch** file `.meow` mục tiêu thành `.meowb` (Meow bytecode).
2. `meow-vm` chạy trực tiếp file `.meowb` (nhanh hơn vì là bytecode).

### Sơ đồ pipeline (ASCII)

```
+--------------+    runs     +------------+    emits    +-----------+    runs
| meow-alpha   | --------->  | compiler   | --------->  | .meowb    | --------->
| (interpreter)|             | (src/main) |             | (bytecode)|  meow-vm
+--------------+             +------------+             +-----------+         
     (optionally runs target)                       (fast execution)
```

### Quickstart (cheatsheet)

* Biên dịch (chỉ build):

```bash
meow-alpha src/main.meow tests/test.meow
# Kết quả: build/test.meowb
```

* Biên dịch rồi chạy ngay:

```bash
meow-alpha src/main.meow tests/test.meow --run
# Kết quả: build/test.meowb → meow-vm chạy build/test.meowb
```

* Self-hosting (dùng compiler MeowScript để dịch chính nó):

```bash
meow-alpha src/main.meow src/main.meow
# Kết quả: build/main.meowb
```

* Dùng compiler đã biên dịch (bytecode) để compile/rerun file khác:

```bash
meow-vm build/main.meowb any_file.meow
# Kết quả: build/any_file.meowb (hoặc chạy tuỳ flag)

# Tái biên dịch compiler bằng meow-vm:
meow-vm build/main.meowb src/main.meow
# Kết quả: build/main.meowb
```

---

## Khi `meow-alpha` private — cách thử ngay

1. **Clone meowscript repo** như bình thường.
2. Nếu bạn có access tới `meow-alpha` (private):

```bash
# clone meow-alpha (SSH) - cần quyền truy cập
git clone git@github.com:LazyPaws/meow-alpha.git third_party/meow-alpha

# hoặc thêm như submodule
git submodule add git@github.com:LazyPaws/meow-alpha.git third_party/meow-alpha
```

3. Build hoặc copy binary `meow-alpha` vào PATH hoặc đặt ở repo root để dùng theo Quickstart.

**Nếu không có quyền truy cập:** chờ repo `meow-alpha` public hoặc yêu cầu tác giả (LazyPaws) cung cấp binary tạm.

---

## Tích hợp CI / Releases (gợi ý cross-repo)

Một số lựa chọn để đồng bộ `meowscript` và `meow-alpha`:

* **Git submodule:** dễ dev cục bộ; hoạt động nếu SSH key có quyền truy cập.
* **Release artifacts:** `meow-alpha` publish binary trong GitHub Releases; `meowscript` CI tải artifact.
* **GitHub Actions + permissions:** dùng `actions/checkout@v4` với token có `contents: read` để checkout repo private trong workflow.
* **Repository dispatch / workflow triggers:** trigger cross-repo khi `meow-alpha` build xong.

---

## Cấu trúc repo (rút gọn)

```
.
├── README.md
├── src
│   ├── backend
│   │   ├── bytecodeStringifier.meow
│   │   ├── codeGenerator.meow
│   │   ├── compiler.meow
│   │   ├── constantPool.meow
│   │   ├── emitter.meow
│   │   ├── opCodes.meow
│   │   ├── registerAllocator.meow
│   │   ├── utils
│   │   │   └── makeCacheFile.meow
│   │   └── visitor
│   │       ├── expression.meow
│   │       ├── literal.meow
│   │       ├── statement.meow
│   │       └── visitor.meow
│   ├── frontend
│   │   ├── ast.meow
│   │   ├── checkType.meow
│   │   ├── diagnostic.meow
│   │   ├── lexer.meow
│   │   ├── parser.meow
│   │   └── token.meow
│   └── main.meow
└── tests
    ├── math.meow
    ├── meow.meow
    ├── simple.meow
    ├── super_test.meow
    ├── test_closure.meow
    ├── test_function.meow
    ├── test_has.meow
    ├── test_lexer.meow
    ├── test_lib.meow
    ├── test.meow
    ├── test_switch.meow
    └── test_unless.meow
```

---

## TODOs ưu tiên liên quan tới refactor

* Hoàn thiện refactor `meow-alpha` rồi public repo hoặc publish binary release.
* Tối ưu register allocator, instruction combining, và inline caches cho VM.
* Thêm CI để build `meow-alpha` và upload release artifact; `meowscript` CI tải artifact để chạy test.
* Thêm test coverage & benchmarks để theo dõi hiệu năng (micro-benchmarks cho instruction dispatch, inline cache hits, v.v.).

---

## License

Tác giả (LazyPaws) vẫn chưa nghĩ ra nên chọn LICENSE nào

---

## Ways to help / Contributing

Nếu bạn muốn đóng góp:

* Mở issue nếu bạn gặp bug hoặc có feature request.
* PR cho tests, docs, hoặc tối ưu hóa nhỏ (register allocator, emitter).
* Nếu bạn có access `meow-alpha`, test workflow self-hosting và report bootstrap edge-cases.

---

## Liên hệ

Tác giả: `LazyPaws` — (GitHub: `github.com/LazyPaws`).

## Tác giả

* Tác giả: LazyPaws (hiện đang là học sinh lớp 9)
* GitHub: https://github.com/LazyPaws
* Note: Tác giả là người cầu toàn nên thấy meow-alpha, meow-vm vẫn chưa tối ưu nên hơi khó chịu nên chưa public

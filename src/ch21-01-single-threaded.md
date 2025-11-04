## 构建单线程 Web 服务器

我们将首先让单线程 Web 服务器工作。在开始之前，让我们快速了解构建 Web 服务器所涉及的协议。这些协议的详细信息超出了本书的范围，但简要概述将为你提供所需的信息。

Web 服务器涉及的两个主要协议是_超文本传输协议_（_HTTP_）和_传输控制协议_（_TCP_）。这两种协议都是_请求-响应_协议，这意味着_客户端_发起请求，_服务器_监听请求并向客户端提供响应。这些请求和响应的内容由协议定义。

TCP 是较低级别的协议，描述了信息如何从一台服务器传输到另一台服务器的详细信息，但不指定该信息是什么。HTTP 通过定义请求和响应的内容构建在 TCP 之上。从技术上讲，可以将 HTTP 与其他协议一起使用，但在绝大多数情况下，HTTP 通过 TCP 发送其数据。我们将处理 TCP 和 HTTP 请求和响应的原始字节。

### 监听 TCP 连接

我们的 Web 服务器需要监听 TCP 连接，所以这是我们首先要处理的部分。标准库提供了一个 `std::net` 模块，让我们可以做到这一点。让我们以通常的方式创建一个新项目：

```console
$ cargo new hello
     Created binary (application) `hello` project
$ cd hello
```

现在在 _src/main.rs_ 中输入代码清单21-1中的代码以开始。此代码将在本地地址 `127.0.0.1:7878` 上监听传入的 TCP 流。当它收到传入流时，它将打印 `Connection established!`。

<Listing number="21-1" file-name="src/main.rs" caption="监听传入流并在收到流时打印消息">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-01/src/main.rs}}
```

</Listing>

使用 `TcpListener`，我们可以在地址 `127.0.0.1:7878` 上监听 TCP 连接。在地址中，冒号前的部分是表示你计算机的 IP 地址（这在每台计算机上都相同，不代表作者特定的计算机），`7878` 是端口。我们选择此端口有两个原因：HTTP 通常不接受此端口，因此我们的服务器不太可能与你在机器上运行的任何其他 Web 服务器冲突，并且 7878 是电话上的_rust_。

在这种情况下，`bind` 函数的工作方式类似于 `new` 函数，因为它将返回一个新的 `TcpListener` 实例。该函数被称为 `bind`，因为在网络中，连接到端口进行监听被称为“绑定到端口”。

`bind` 函数返回一个 `Result<T, E>`，这表明绑定可能会失败，例如，如果我们运行两个程序实例，因此有两个程序监听同一端口。因为我们只是出于学习目的编写基本服务器，所以我们不会担心处理这些类型的错误；相反，如果发生错误，我们使用 `unwrap` 停止程序。

`TcpListener` 上的 `incoming` 方法返回一个迭代器，该迭代器为我们提供一系列流（更具体地说，是类型为 `TcpStream` 的流）。单个_流_表示客户端和服务器之间的开放连接。_连接_是完整请求和响应过程的名称，在该过程中，客户端连接到服务器，服务器生成响应，然后服务器关闭连接。因此，我们将从 `TcpStream` 读取以查看客户端发送的内容，然后将响应写入流以将数据发送回客户端。总的来说，这个 `for` 循环将依次处理每个连接并为我们生成一系列要处理的流。

目前，我们对流的处理包括调用 `unwrap` 以在流有任何错误时终止程序；如果没有错误，程序将打印一条消息。我们将在下一个代码清单中为成功情况添加更多功能。当客户端连接到服务器时，我们可能从 `incoming` 方法收到错误的原因是我们实际上不是在迭代连接。相反，我们正在迭代_连接尝试_。连接可能由于多种原因而不成功，其中许多是特定于操作系统的。例如，许多操作系统对它们可以支持的并发打开连接数量有限制；超出该数量的新连接尝试将产生错误，直到关闭一些打开的连接。

让我们尝试运行这段代码！在终端中调用 `cargo run`，然后在 Web 浏览器中加载 _127.0.0.1:7878_。浏览器应该显示类似“Connection reset”的错误消息，因为服务器当前没有发送回任何数据。但是当你查看终端时，你应该看到在浏览器连接到服务器时打印的几条消息！

```text
     Running `target/debug/hello`
Connection established!
Connection established!
Connection established!
```

有时你会看到一条浏览器请求打印多条消息；原因可能是浏览器正在为页面发出请求，也为其他资源（如浏览器选项卡中显示的 _favicon.ico_ 图标）发出请求。

也可能是浏览器多次尝试连接到服务器，因为服务器没有用任何数据响应。当 `stream` 超出作用域并在循环结束时被丢弃时，连接会作为 `drop` 实现的一部分关闭。浏览器有时通过重试来处理关闭的连接，因为问题可能是暂时的。

浏览器有时也会在未发送任何请求的情况下打开与服务器的多个连接，以便如果它们*确实*稍后发送请求，这些请求可以更快地发生。当发生这种情况时，我们的服务器将看到每个连接，无论该连接上是否有任何请求。例如，许多基于 Chrome 的浏览器版本都会这样做；你可以通过使用隐私浏览模式或使用不同的浏览器来禁用该优化。

重要的因素是我们已经成功获得了 TCP 连接的句柄！

记住在运行特定版本的代码后按 <kbd>ctrl</kbd>-<kbd>C</kbd> 停止程序。然后，在每次代码更改后，通过调用 `cargo run` 命令重新启动程序，以确保你运行的是最新代码。

### 读取请求

让我们实现从浏览器读取请求的功能！为了分离首先获取连接然后对连接执行某些操作的关注点，我们将启动一个新函数来处理连接。在这个新的 `handle_connection` 函数中，我们将从 TCP 流读取数据并打印它，以便我们可以看到从浏览器发送的数据。将代码更改为代码清单21-2所示。

<Listing number="21-2" file-name="src/main.rs" caption="从 `TcpStream` 读取并打印数据">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-02/src/main.rs}}
```

</Listing>

我们将 `std::io::BufReader` 和 `std::io::prelude` 引入作用域，以访问允许我们从流读取和写入的 traits 和类型。在 `main` 函数的 `for` 循环中，我们现在调用新的 `handle_connection` 函数并将 `stream` 传递给它，而不是打印一条说我们建立连接的消息。

在 `handle_connection` 函数中，我们创建一个新的 `BufReader` 实例，它包装对 `stream` 的引用。`BufReader` 通过为我们管理对 `std::io::Read` trait 方法的调用来添加缓冲。

我们创建一个名为 `http_request` 的变量来收集浏览器发送到我们服务器的请求行。我们通过添加 `Vec<_>` 类型注释来指示我们希望将这些行收集到向量中。

`BufReader` 实现了 `std::io::BufRead` trait，它提供了 `lines` 方法。`lines` 方法通过在看到换行字节时拆分数据流，返回 `Result<String, std::io::Error>` 的迭代器。为了获取每个 `String`，我们对每个 `Result` 进行 `map` 和 `unwrap`。如果数据不是有效的 UTF-8 或者从流读取时出现问题，`Result` 可能是错误。同样，生产程序应该更优雅地处理这些错误，但为了简单起见，我们选择在错误情况下停止程序。

浏览器通过连续发送两个换行字符来发出 HTTP 请求结束的信号，因此为了从流中获取一个请求，我们获取行直到得到空字符串行。一旦我们将行收集到向量中，我们就使用漂亮的调试格式打印它们，以便我们可以查看 Web 浏览器发送到我们服务器的指令。

让我们尝试这段代码！启动程序并在 Web 浏览器中再次发出请求。请注意，我们仍然会在浏览器中收到错误页面，但我们在终端中的程序输出现在将类似于：

<!-- manual-regeneration
cd listings/ch21-web-server/listing-21-02
cargo run
make a request to 127.0.0.1:7878
Can't automate because the output depends on making requests
-->

```console
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.42s
     Running `target/debug/hello`
Request: [
    "GET / HTTP/1.1",
    "Host: 127.0.0.1:7878",
    "User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:99.0) Gecko/20100101 Firefox/99.0",
    "Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8",
    "Accept-Language: en-US,en;q=0.5",
    "Accept-Encoding: gzip, deflate, br",
    "DNT: 1",
    "Connection: keep-alive",
    "Upgrade-Insecure-Requests: 1",
    "Sec-Fetch-Dest: document",
    "Sec-Fetch-Mode: navigate",
    "Sec-Fetch-Site: none",
    "Sec-Fetch-User: ?1",
    "Cache-Control: max-age=0",
]
```

根据你的浏览器，你可能会得到略有不同的输出。现在我们已经打印了请求数据，我们可以通过查看请求第一行中 `GET` 之后的路径来了解为什么一个浏览器请求会得到多个连接。如果重复的连接都请求 _/_，我们知道浏览器正在尝试反复获取 _/_，因为它没有从我们的程序获得响应。

让我们分解这个请求数据，以了解浏览器对我们的程序的要求。

<!-- Old headings. Do not remove or links may break. -->

<a id="a-closer-look-at-an-http-request"></a>
<a id="looking-closer-at-an-http-request"></a>

### 更仔细地查看 HTTP 请求

HTTP 是基于文本的协议，请求采用以下格式：

```text
Method Request-URI HTTP-Version CRLF
headers CRLF
message-body
```

第一行是_请求行_，包含有关客户端请求的信息。请求行的第一部分指示使用的方法，例如 `GET` 或 `POST`，它描述了客户端如何发出此请求。我们的客户端使用了 `GET` 请求，这意味着它正在请求信息。

请求行的下一部分是 _/_，它指示客户端请求的_统一资源标识符_（_URI_）：URI 几乎但不完全与_统一资源定位符_（_URL_）相同。URI 和 URL 之间的差异对于我们本章的目的并不重要，但 HTTP 规范使用术语 _URI_，因此我们可以在心理上用 _URL_ 替换 _URI_。

最后一部分是客户端使用的 HTTP 版本，然后请求行以 CRLF 序列结束。（_CRLF_ 代表_回车_和_换行_，这是打字机时代的术语！）CRLF 序列也可以写成 `\r\n`，其中 `\r` 是回车，`\n` 是换行。_CRLF 序列_将请求行与请求数据的其余部分分开。请注意，当打印 CRLF 时，我们看到新行开始而不是 `\r\n`。

查看到目前为止从运行我们的程序收到的请求行数据，我们看到 `GET` 是方法，_/_ 是请求 URI，`HTTP/1.1` 是版本。

请求行之后，从 `Host:` 开始的剩余行是标头。`GET` 请求没有主体。

尝试从不同的浏览器发出请求或请求不同的地址，例如 _127.0.0.1:7878/test_，以查看请求数据如何变化。

现在我们知道浏览器在请求什么，让我们发送回一些数据！

### 编写响应

我们将实现发送数据以响应客户端请求。响应具有以下格式：

```text
HTTP-Version Status-Code Reason-Phrase CRLF
headers CRLF
message-body
```

第一行是_状态行_，包含响应中使用的 HTTP 版本、总结请求结果的数字状态代码，以及提供状态代码文本描述的原因短语。CRLF 序列之后是任何标头、另一个 CRLF 序列和响应的主体。

以下是一个使用 HTTP 版本 1.1 且状态代码为 200、OK 原因短语、无标头和无主体的示例响应：

```text
HTTP/1.1 200 OK\r\n\r\n
```

状态代码 200 是标准的成功响应。该文本是一个微小的成功 HTTP 响应。让我们将其写入流作为对成功请求的响应！从 `handle_connection` 函数中，删除打印请求数据的 `println!`，并用代码清单21-3中的代码替换它。

<Listing number="21-3" file-name="src/main.rs" caption="将微小的成功 HTTP 响应写入流">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-03/src/main.rs:here}}
```

</Listing>

第一行定义了 `response` 变量，该变量保存成功消息的数据。然后，我们在 `response` 上调用 `as_bytes` 以将字符串数据转换为字节。`stream` 上的 `write_all` 方法接受 `&[u8]` 并将这些字节直接发送到连接。因为 `write_all` 操作可能会失败，所以我们像以前一样对任何错误结果使用 `unwrap`。同样，在实际应用程序中，你应该在此处添加错误处理。

通过这些更改，让我们运行代码并发出请求。我们不再向终端打印任何数据，因此除了 Cargo 的输出外，我们不会看到任何输出。当你在 Web 浏览器中加载 _127.0.0.1:7878_ 时，你应该得到一个空白页面而不是错误。你刚刚手动编码了接收 HTTP 请求并发送响应！

### 返回真实的 HTML

让我们实现返回不仅仅是空白页面的功能。在你的项目目录的根目录中创建新文件 _hello.html_，而不是在 _src_ 目录中。你可以输入任何你想要的 HTML；代码清单21-4显示了一种可能性。

<Listing number="21-4" file-name="hello.html" caption="要在响应中返回的示例 HTML 文件">

```html
{{#include ../listings/ch21-web-server/listing-21-05/hello.html}}
```

</Listing>

这是一个带有标题和一些文本的最小 HTML5 文档。为了在收到请求时从服务器返回此内容，我们将修改 `handle_connection`，如代码清单21-5所示，以读取 HTML 文件，将其作为主体添加到响应中，然后发送它。

<Listing number="21-5" file-name="src/main.rs" caption="将 *hello.html* 的内容作为响应主体发送">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-05/src/main.rs:here}}
```

</Listing>

我们已将 `fs` 添加到 `use` 语句中，以将标准库的文件系统模块引入作用域。将文件内容读取到字符串的代码应该看起来很熟悉；当我们在代码清单12-4中为 I/O 项目读取文件内容时使用了它。

接下来，我们使用 `format!` 将文件的内容添加为成功响应的主体。为了确保有效的 HTTP 响应，我们添加了 `Content-Length` 标头，该标头设置为响应主体的大小——在这种情况下，是 `hello.html` 的大小。

使用 `cargo run` 运行此代码，并在浏览器中加载 _127.0.0.1:7878_；你应该看到你的 HTML 被渲染！

目前，我们忽略 `http_request` 中的请求数据，只是无条件地发送回 HTML 文件的内容。这意味着如果你尝试在浏览器中请求 _127.0.0.1:7878/something-else_，你仍然会得到相同的 HTML 响应。目前，我们的服务器非常有限，不会执行大多数 Web 服务器所做的事情。我们希望根据请求自定义响应，并且只对格式良好的 _/_ 请求发送回 HTML 文件。

### 验证请求并选择性响应

目前，无论客户端请求什么，我们的 Web 服务器都会返回文件中的 HTML。让我们添加功能来检查浏览器是否在返回 HTML 文件之前请求 _/_，如果浏览器请求其他任何内容，则返回错误。为此，我们需要修改 `handle_connection`，如代码清单21-6所示。此新代码将收到的请求内容与我们已知的对 _/_ 的请求进行比较，并添加 `if` 和 `else` 块以不同方式处理请求。

<Listing number="21-6" file-name="src/main.rs" caption="以不同于其他请求的方式处理对 */* 的请求">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-06/src/main.rs:here}}
```

</Listing>

我们只查看 HTTP 请求的第一行，因此我们调用 `next` 从迭代器获取第一项，而不是将整个请求读入向量。第一个 `unwrap` 处理 `Option`，如果迭代器没有项，则停止程序。第二个 `unwrap` 处理 `Result`，与在代码清单21-2中添加的 `map` 中的 `unwrap` 具有相同的效果。

接下来，我们检查 `request_line` 以查看它是否等于对 _/_ 路径的 GET 请求的请求行。如果是，`if` 块返回我们 HTML 文件的内容。

如果 `request_line` 不等于对 _/_ 路径的 GET 请求，这意味着我们收到了其他请求。我们将稍后在 `else` 块中添加代码以响应所有其他请求。

现在运行此代码并请求 _127.0.0.1:7878_；你应该获得 _hello.html_ 中的 HTML。如果你发出任何其他请求，例如 _127.0.0.1:7878/something-else_，你将收到连接错误，就像你在运行代码清单21-1和代码清单21-2中的代码时看到的错误一样。

现在让我们将代码清单21-7中的代码添加到 `else` 块，以返回状态代码为 404 的响应，该响应表示未找到请求的内容。我们还将返回一些 HTML，用于在浏览器中呈现页面，向最终用户指示响应。

<Listing number="21-7" file-name="src/main.rs" caption="如果请求了除 */* 以外的任何内容，则使用状态代码 404 和错误页面响应">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-07/src/main.rs:here}}
```

</Listing>

这里，我们的响应具有状态代码为 404 的状态行和原因短语 `NOT FOUND`。响应的主体将是文件 _404.html_ 中的 HTML。你需要在 _hello.html_ 旁边创建一个 _404.html_ 文件作为错误页面；同样，你可以使用任何你想要的 HTML，或使用代码清单21-8中的示例 HTML。

<Listing number="21-8" file-name="404.html" caption="要随任何 404 响应发送回的页面示例内容">

```html
{{#include ../listings/ch21-web-server/listing-21-07/404.html}}
```

</Listing>

通过这些更改，再次运行服务器。请求 _127.0.0.1:7878_ 应该返回 _hello.html_ 的内容，而任何其他请求（如 _127.0.0.1:7878/foo_）应该返回 _404.html_ 中的错误 HTML。

<!-- Old headings. Do not remove or links may break. -->

<a id="a-touch-of-refactoring"></a>

### 重构

目前，`if` 和 `else` 块有很多重复：它们都在读取文件并将文件内容写入流。唯一的区别是状态行和文件名。让我们通过将这些差异提取到单独的 `if` 和 `else` 行中来使代码更简洁，这些行将状态行和文件名的值分配给变量；然后我们可以在代码中无条件地使用这些变量来读取文件并写入响应。代码清单21-9显示了替换大的 `if` 和 `else` 块后的结果代码。

<Listing number="21-9" file-name="src/main.rs" caption="重构 `if` 和 `else` 块，使其仅包含两种情况之间不同的代码">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-09/src/main.rs:here}}
```

</Listing>

现在 `if` 和 `else` 块只在元组中返回状态行和文件名的适当值；然后我们使用解构将这些两个值分配给 `status_line` 和 `filename`，使用第19章中讨论的 `let` 语句中的模式。

之前重复的代码现在在 `if` 和 `else` 块之外，并使用 `status_line` 和 `filename` 变量。这使得更容易看到两种情况之间的差异，并且意味着如果我们想要更改文件读取和响应写入的工作方式，我们只有一个地方可以更新代码。代码清单21-9中的代码行为将与代码清单21-7中的代码行为相同。

太棒了！我们现在有一个大约 40 行 Rust 代码的简单 Web 服务器，它用一个内容页面响应一个请求，并用 404 响应响应所有其他请求。

目前，我们的服务器在单线程中运行，这意味着它一次只能处理一个请求。让我们通过模拟一些慢速请求来检查这如何成为问题。然后，我们将修复它，以便我们的服务器可以同时处理多个请求。

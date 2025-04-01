### **`bufio` 包详解**

**📌 核心作用**：提供带缓冲的 I/O 操作（读写文件、网络数据等），**减少系统调用次数，提升性能**。

---

## **🔧 常用函数清单**

|函数/类型|作用|典型场景|
|---|---|---|
|`bufio.Reader`|缓冲读取|逐行读取大文件|
|`bufio.Writer`|缓冲写入|批量写入减少磁盘操作|
|`bufio.Scanner`|高级文本扫描|按行/分隔符读取文本|
|`bufio.ReadWriter`|组合读写器|网络套接字读写|

---

## **📚 核心功能详解**

### **1. `bufio.Reader`（缓冲读取）**

#### **作用**

将底层 `io.Reader`（如文件、网络连接）包装为带缓冲的读取器，减少直接 I/O 操作。

#### **常用方法**

|方法|说明|
|---|---|
|`Read(p []byte)`|读取数据到字节切片|
|`ReadString(delim byte)`|读取直到遇到分隔符（如 `\n`）|
|`ReadLine()`|读取一行（不推荐，用 `Scanner`）|
|`Peek(n int)`|预览前 n 字节但不移动指针|

#### **示例：逐行读取文件**

```go
package main

import (
	"bufio"
	"fmt"
	"os"
)

func main() {
	file, _ := os.Open("data.txt")
	defer file.Close()

	reader := bufio.NewReader(file)
	for {
		line, err := reader.ReadString('\n') // 按行读取
		if err != nil {
			break // 读到文件尾
		}
		fmt.Print("Line:", line)
	}
}
```
**输出**：
```
Line:Hello, world!
Line:This is a test.
```
---

### **2. `bufio.Writer`（缓冲写入）**

#### **作用**

将多次小数据写入合并为单次大块写入，减少磁盘/网络 I/O 次数。

#### **常用方法**

|方法|说明|
|---|---|
|`Write(p []byte)`|写入字节切片|
|`WriteString(s string)`|直接写入字符串|
|`Flush()`|强制将缓冲数据写入底层流|

#### **示例：高效写入文件**

```go
package main

import (
	"bufio"
	"os"
)

func main() {
	file, _ := os.Create("output.txt")
	defer file.Close()

	writer := bufio.NewWriter(file)
	writer.WriteString("Hello, ")  // 写入缓冲区
	writer.WriteString("bufio!\n") // 仍在缓冲区
	writer.Flush()                 // 实际写入磁盘
}
```
**文件内容**：
```
Hello, bufio!
```
---

### **3. `bufio.Scanner`（高级扫描）**

#### **作用**

提供更简单的文本逐行/逐词扫描，自动处理缓冲和内存分配。

#### **常用方法**

|方法|说明|
|---|---|
|`Scan()`|读取下一个 token（如行）|
|`Text()`|获取当前 token 的文本|
|`Split(splitFunc)`|自定义分割逻辑（如按单词）|

#### **示例：统计文件行数**

```go
package main

import (
	"bufio"
	"fmt"
	"os"
)

func main() {
	file, _ := os.Open("data.txt")
	defer file.Close()

	scanner := bufio.NewScanner(file)
	lineCount := 0
	for scanner.Scan() { // 默认按行扫描
		lineCount++
	}
	fmt.Println("Total lines:", lineCount)
}
```
**输出**：
```
Total lines: 42
```
---

## **⚠️ 注意事项**

1. **必须调用 `Flush()`**
    
    - `bufio.Writer` 的数据在缓冲区中，直到调用 `Flush()` 才会真正写入。
        
2. **缓冲区大小**
    
    - 默认缓冲区大小 4096 字节，可通过 `bufio.NewReaderSize` 调整。
        
3. **`Scanner` 的内存限制**
    
    - 单行默认最大 64KB，超长行需用 `reader.ReadString()`。
        

---

## **🚀 性能优化场景**

|场景|推荐工具|优势|
|---|---|---|
|读取大文件|`bufio.Reader`|减少系统调用|
|高频写入小数据|`bufio.Writer`|合并写入降低 I/O 压力|
|解析结构化文本|`bufio.Scanner`|简化代码，自动处理分隔符|

---

## **💡 进阶技巧**

### **1. 自定义 `Scanner` 分割逻辑**

```go
scanner := bufio.NewScanner(file)
scanner.Split(bufio.ScanWords) // 改为按单词扫描
for scanner.Scan() {
    fmt.Println("Word:", scanner.Text())
}
```
### **2. 复用缓冲区（零分配）**

```go
reader := bufio.NewReader(file)
buf := make([]byte, 1024) // 复用缓冲区
for {
    n, _ := reader.Read(buf)
    if n == 0 { break }
    fmt.Println(string(buf[:n]))
}
```
---

## **📌 总结**

- **何时用 `bufio`**：需要高频或大块 I/O 时（如日志处理、网络通信）。
    
- **核心类型**：
    
    - `Reader`：缓冲读取 → 适合大文件
        
    - `Writer`：缓冲写入 → 适合高频小写入
        
    - `Scanner`：文本扫描 → 适合按行/词解析
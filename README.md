<!--

Editing this document:

- Discuss all changes in GitHub issues first.
- Update the table of contents as new sections are added or removed.
- Use tables for side-by-side code samples. See below.

Code Samples:

Use 2 spaces to indent. Horizontal real estate is important in side-by-side
samples.

For side-by-side code samples, use the following snippet.

~~~
<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
BAD CODE GOES HERE
```

</td><td>

```go
GOOD CODE GOES HERE
```

</td></tr>
</tbody></table>
~~~

(You need the empty lines between the <td> and code samples for it to be
treated as Markdown.)

If you need to add labels or descriptions below the code samples, add another
row before the </tbody></table> line.

~~~
<tr>
<td>DESCRIBE BAD CODE</td>
<td>DESCRIBE GOOD CODE</td>
</tr>
~~~

-->

# Uber Go Style Guide

## Table of Contents

- [Introduction](#introduction)
- [Guidelines](./docs/guidelines.md#guidelines)
  - [Pointers to Interfaces](./docs/guidelines.md#pointers-to-interfaces)
  - [Receivers and Interfaces](./docs/guidelines.md#receivers-and-interfaces)
  - [Zero-value Mutexes are Valid](./docs/guidelines.md#zero-value-mutexes-are-valid)
  - [Copy Slices and Maps at Boundaries](./docs/guidelines.md#copy-slices-and-maps-at-boundaries)
  - [Defer to Clean Up](./docs/guidelines.md#defer-to-clean-up)
  - [Channel Size is One or None](./docs/guidelines.md#channel-size-is-one-or-none)
  - [Start Enums at One](./docs/guidelines.md#start-enums-at-one)
  - [Error Types](./docs/guidelines.md#error-types)
  - [Error Wrapping](./docs/guidelines.md#error-wrapping)
  - [Handle Type Assertion Failures](./docs/guidelines.md#handle-type-assertion-failures)
  - [Don't Panic](./docs/guidelines.md#dont-panic)
  - [Use go.uber.org/atomic](./docs/guidelines.md#use-gouberorgatomic)
- [Performance](./docs/performance.md#performance)
  - [Prefer strconv over fmt](./docs/performance.md#prefer-strconv-over-fmt)
  - [Avoid string-to-byte conversion](./docs/performance.md#avoid-string-to-byte-conversion)
  - [Prefer Specifying Map Capacity Hints](./docs/performance.md#prefer-specifying-map-capacity-hints)
- [Style](./docs/style.md#style)
  - [Be Consistent](./docs/style.md#be-consistent)
  - [Group Similar Declarations](./docs/style.md#group-similar-declarations)
  - [Import Group Ordering](./docs/style.md#import-group-ordering)
  - [Package Names](./docs/style.md#package-names)
  - [Function Names](./docs/style.md#function-names)
  - [Import Aliasing](./docs/style.md#import-aliasing)
  - [Function Grouping and Ordering](./docs/style.md#function-grouping-and-ordering)
  - [Reduce Nesting](./docs/style.md#reduce-nesting)
  - [Unnecessary Else](./docs/style.md#unnecessary-else)
  - [Top-level Variable Declarations](./docs/style.md#top-level-variable-declarations)
  - [Prefix Unexported Globals with _](./docs/style.md#prefix-unexported-globals-with-_)
  - [Embedding in Structs](./docs/style.md#embedding-in-structs)
  - [Use Field Names to Initialize Structs](./docs/style.md#use-field-names-to-initialize-structs)
  - [Local Variable Declarations](./docs/style.md#local-variable-declarations)
  - [nil is a valid slice](./docs/style.md#nil-is-a-valid-slice)
  - [Reduce Scope of Variables](./docs/style.md#reduce-scope-of-variables)
  - [Avoid Naked Parameters](./docs/style.md#avoid-naked-parameters)
  - [Use Raw String Literals to Avoid Escaping](./docs/style.md#use-raw-string-literals-to-avoid-escaping)
  - [Initializing Struct References](./docs/style.md#initializing-struct-references)
  - [Initializing Maps](./docs/style.md#initializing-maps)
  - [Format Strings outside Printf](./docs/style.md#format-strings-outside-printf)
  - [Naming Printf-style Functions](./docs/style.md#naming-printf-style-functions)
- [Patterns](./docs/patterns.md#patterns)
  - [Test Tables](./docs/patterns.md#test-tables)
  - [Functional Options](./docs/patterns.md#functional-options)



## Introduction

สไตล์เป็นเหมือนข้อตกลงที่ช่วยจัดระเบียบโค้ดของเรา แต่คำว่าสไตล์ก็อาจจะทำให้สับสนนิดหน่อย เพราะ ข้อตกลงนี้มันครอบคลุมไปมากกว่าแค่เรื่องไฟล์ซอสโค้ด เพราะถ้าเป็นอย่างนั้น gofmt ก็จัดการให้เราได้อยู่แล้ว

เป้าหมายของคำแนะนำชุดนี้ คือการลดความซับซ้อนด้วยการอธิบายว่าที่ Uber เราทำ หรือไม่ทำอะไรตอนที่เราเขียน Go กันบ้าง และกฎนี้มีไว้เพื่อช่วยให้โค้ดมันดูแลจัดการได้ง่าย ในขณะที่ก็ยอมให้วิศกรซอฟต์แวร์ใช้มันได้อย่างมีประสิทธิภาพด้วย

คำแนะนำชุดนี้เดิมถูกเขียนขึ้นโดย [Prashant Varanasi] และ [Simon Newton] เพื่อช่วยให้เพื่อนร่วมงานเริ่มต้นเขียน Go กันได้เร็วขึ้น แต่หลังจากผ่านไปหลายปี มันก็ถูกแก้ไขเพิ่มเติมจากข้อเสนอแนะต่างๆที่ได้รับ

  [Prashant Varanasi]: https://github.com/prashantv
  [Simon Newton]: https://github.com/nomis52

สำนวนการเขียน Go ในเอกสารนี้เป็นแบบฉบับที่ใช้กันที่ Uber ซึ่งปกติก็เป็นแนวทางเดียวกับการเขียน Go ทั่วไปอยู่แล้ว ซึ่งถ้าจะมีเพิ่มเติมจากภายนอกก็มาจากที่เหล่านี้:

1. [Effective Go](https://golang.org/doc/effective_go.html)
2. [The Go common mistakes guide](https://github.com/golang/go/wiki/CodeReviewComments)

โค้ดทั้งหมดควรจะต้องไม่มี error ใดๆจาก `golint` และ `go vet` เราแนะนำให้คุณตั้งค่าใน editor ตามนี้:

- Run `goimports` on save
- Run `golint` and `go vet` to check for errors

คุณสามารถหาข้อมูลเพิ่มเติมเกี่ยวกับการเครื่องมือช่วยใน editors ได้จากที่นี่:
<https://github.com/golang/go/wiki/IDEsAndTextEditorPlugins>

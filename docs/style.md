# Style

## Be Consistent

คำแนะนำบางส่วนที่ระบุในเอกสารชุดนี้ วัดผลได้จริง เว้นไว้แต่เพียง พฤติกรรม บริบท หรือหัวข้อต่างๆ

นอกเหนือจากที่กล่าวมาก็คือ **ทำให้เป็นจังหวะเดียวกัน**

โค้ดที่ลายมือเดียวกัน มันดูแลรักษาง่าย มันง่ายที่จะเข้าใจ ไม่ทำให้เสียเวลาต้องมานั่งแกะ แล้วถ้าแก้ไขย้ายที่มันก็ยังทำได้ง่ายกว่า รวมถึงตอนแก้บั๊กด้วย

ตรงกันข้าม ถ้าเขียนมาคนละแบบ หรือสไตล์ไม่เข้ากันทั้งๆที่โค้ดชุดเดียวกัน มันจะทำให้เสียเวลาในการดูแล เปราะบาง และไม่เข้ากัน ทั้งหมดทั้งมวลนี้จะทำให้ทำงานได้ช้า รีวิวโค้ด จะเหนื่อยมาก และเต็มไปด้วยบั๊ก

เวลาจะนำเอาคำแนะนำชุดนี้ไปปรับใช้จริง แนะนำว่าให้ทำกันในระดับแพ็กเกจ (หรือใหญ่กว่า): ถ้าทำแค่ในแพ็กเกจย่อยๆ มันจะขัดกับสิ่งที่กล่าวมาข้างต้น เพราะมันจะมีหลายสไตล์ในโค้ดชุดเดียว

## Group Similar Declarations

Go สนับสนุนการจัดกลุ่มการการประกาศที่เป็นพวกเดียวกัน

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
import "a"
import "b"
```

</td><td>

```go
import (
  "a"
  "b"
)
```

</td></tr>
</tbody></table>

การทำแบบนี้ยังสามารถทำได้กับการประกาศ constant ตัวแปร และการประกาศ type

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go

const a = 1
const b = 2



var a = 1
var b = 2



type Area float64
type Volume float64
```

</td><td>

```go
const (
  a = 1
  b = 2
)

var (
  a = 1
  b = 2
)

type (
  Area float64
  Volume float64
)
```

</td></tr>
</tbody></table>

จัดกลุ่มเฉพาะสิ่งที่สัมพันธ์กัน อย่าไปทำกับอะไรที่ไม่เกี่ยวข้องกัน

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
type Operation int

const (
  Add Operation = iota + 1
  Subtract
  Multiply
  ENV_VAR = "MY_ENV"
)
```

</td><td>

```go
type Operation int

const (
  Add Operation = iota + 1
  Subtract
  Multiply
)

const ENV_VAR = "MY_ENV"
```

</td></tr>
</tbody></table>

การจัดกลุ่มสามารถทำในฟังก์ชันก็ได้ ดังแสดงในตัวอย่าง

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
func f() string {
  var red = color.New(0xff0000)
  var green = color.New(0x00ff00)
  var blue = color.New(0x0000ff)

  ...
}
```

</td><td>

```go
func f() string {
  var (
    red   = color.New(0xff0000)
    green = color.New(0x00ff00)
    blue  = color.New(0x0000ff)
  )

  ...
}
```

</td></tr>
</tbody></table>

## Import Group Ordering

แบ่งกลุ่มการอิมพอร์ตเป็นสองชุด:

- Standard library
- Everything else

การจัดกลุ่มนี้ goimports ทำให้โดยปกติอยู่แล้ว

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
import (
  "fmt"
  "os"
  "go.uber.org/atomic"
  "golang.org/x/sync/errgroup"
)
```

</td><td>

```go
import (
  "fmt"
  "os"

  "go.uber.org/atomic"
  "golang.org/x/sync/errgroup"
)
```

</td></tr>
</tbody></table>

## Package Names

เวลาจะประกาศชื่อแพ็กเกจ ให้เลือกแบบนี้:

- ใช้ตัวอักษรเล็กทั้งหมด ไม่มีตัวใหญ่ หรือขีดล่าง
- ไม่เปลี่ยนชื่อมันตอนที่ผู้ใช้ import มันเข้าไป
- สั้นและกระชับ เพราะมันจะถูกอ้างถึงในทุกที่จะมาเรียกใช้
- ไม่ต้องทำเป็นพหูพจน์
- อย่าตั้งชื่อ "common", "util", "shared" ชื่อพวกนี้มันห่วย เพราะไม่ได้ช่วยให้เรารู้อะไรเลย

ดูเพิ่มเติมได้ที่ [Package Names] และ [Style guideline for Go packages]

[Package Names]: https://blog.golang.org/package-names
[Style guideline for Go packages]: https://rakyll.org/style-packages/

## การตั้งชื่อฟังก์ชัน

เราทำแบบเดียวกับที่ชุมชนคนเขียน go ทำกันด้วยการใช้ [MixedCaps for function
names] (การผสมตัวอักษรเล็กและใหญ่) ยกเว้นเฉพาะเวลาเขียนเทส สามารถใช้ขีดล่างได้ เพื่อจัดกลุ่มการทดสอบที่สัมพันธ์กัน ตัวอย่างเช่น
`TestMyFunction_WhatIsBeingTested`.

[MixedCaps for function names]: https://golang.org/doc/effective_go.html#mixed-caps

## Import Aliasing

การตั้งชื่อแฝงให้แพ็กเกจที่ import ทำเมื่อชื่อแพ็กเกจที่นำเข้ามาไม่ตรงกับส่วนสุดท้ายของพาร์ท

```go
import (
  "net/http"

  client "example.com/client-go"
  trace "example.com/trace/v2"
)
```

ในกรณีอื่นๆ การตั้งชื่อแฝงให้การ import ไม่ควรทำ เว้นเสียแต่ว่ามันจะไปซ้ำกันกับแพ็กเกจอื่น

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
import (
  "fmt"
  "os"


  nettrace "golang.net/x/trace"
)
```

</td><td>

```go
import (
  "fmt"
  "os"
  "runtime/trace"

  nettrace "golang.net/x/trace"
)
```

</td></tr>
</tbody></table>

## Function Grouping and Ordering

- ควรเรียงฟังก์ชันตามลำดับการเรียกใช้
- ฟังก์ชันในไฟล์ควรจัดกลุ่มตาม receiver

ฟังก์ชันที่เปิดเผยไปข้างนอกควรอยู่ในส่วนแรกๆของไฟล์ หลังการประกาศ `struct`, `const`, `var`

ฟังก์ชันแบบนี้ `newXYZ()`/`NewXYZ()` ควรอยู่หลังการประกาศ type แต่อยู่ก่อนเมธอดที่ใช้ type นี้เป็นตัว receiver

พอฟังก์ชันถูกจัดกลุ่มแบบนี้ พวกฟังก์ชันที่ใช้งานทั่วไปก็ควรไปอยู่ส่วนท้ายๆของไฟล์

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
func (s *something) Cost() {
  return calcCost(s.weights)
}

type something struct{ ... }

func calcCost(n []int) int {...}

func (s *something) Stop() {...}

func newSomething() *something {
    return &something{}
}
```

</td><td>

```go
type something struct{ ... }

func newSomething() *something {
    return &something{}
}

func (s *something) Cost() {
  return calcCost(s.weights)
}

func (s *something) Stop() {...}

func calcCost(n []int) int {...}
```

</td></tr>
</tbody></table>

## Reduce Nesting

โค้ดควรลดความยุ่งเหยิงด้วยการจัดการ error ก่อนแล้วรีเทิร์นออกไป หรือไปเริ่มต้นลูปใหม่ ให้เร็วที่สุด เพื่อลดโค้ดที่ซ้อนกันหลายๆชั้น

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
for _, v := range data {
  if v.F1 == 1 {
    v = process(v)
    if err := v.Call(); err == nil {
      v.Send()
    } else {
      return err
    }
  } else {
    log.Printf("Invalid v: %v", v)
  }
}
```

</td><td>

```go
for _, v := range data {
  if v.F1 != 1 {
    log.Printf("Invalid v: %v", v)
    continue
  }

  v = process(v)
  if err := v.Call(); err != nil {
    return err
  }
  v.Send()
}
```

</td></tr>
</tbody></table>

## Unnecessary Else

ถ้าตัวแปรจะถูกกำหนดค่าทั้งใน if และ else มันควรจะเหลือแค่ if ก็ได้

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
var a int
if b {
  a = 100
} else {
  a = 10
}
```

</td><td>

```go
a := 10
if b {
  a = 100
}
```

</td></tr>
</tbody></table>

## Top-level Variable Declarations

การใช้คียเวิร์ด `var` ไม่ต้องบอก type ก็ได้ เว้นเสียแต่ว่ามันจะคืน type ไม่ตรงกับที่ต้องการ

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
var _s string = F()

func F() string { return "A" }
```

</td><td>

```go
var _s = F()
// Since F already states that it returns a string, we don't need to specify
// the type again.

func F() string { return "A" }
```

</td></tr>
</tbody></table>

ระบุ type ถ้า type ที่ได้รับมาไม่ตรงกับที่อยากได้

```go
type myError struct{}

func (myError) Error() string { return "error" }

func F() myError { return myError{} }

var _e error = F()
// F returns an object of type myError but we want error.
```

## Prefix Unexported Globals with _

ตั้งชื่อขึ้นต้นด้วย ขีดล่าง เวลาประกาศด้วย `var`s และ `const`s ให้ตัวแปรที่ไม่เปิดเผยสู่ภายนอก เพื่อทำให้ชัดเจนว่ามันถูกใช้เป็น global อยู่ภายในแพ็กเกจ

ข้อยกเว้น: ตัวแปร error ที่ไม่เปิดเผยสู่ภายนอก ควรตั้งขื่อขึ้นต้นด้วย `err`

หลักการและเหตุผล: ตัวแปรที่ประกาศไว้ตั้งแต่ต้น และพวก constants มีขอบเขตในแพ็กเกจ เพราะฉะนั้น การตั้งชื่อแบบกลางๆ มันจะทำให้เกิดเรื่องไม่คาดคิดได้ ทำให้ได้ค่าผิดในไฟล์อื่นได้ง่ายมาก

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
// foo.go

const (
  defaultPort = 8080
  defaultUser = "user"
)

// bar.go

func Bar() {
  defaultPort := 9090
  ...
  fmt.Println("Default port", defaultPort)

  // We will not see a compile error if the first line of
  // Bar() is deleted.
}
```

</td><td>

```go
// foo.go

const (
  _defaultPort = 8080
  _defaultUser = "user"
)
```

</td></tr>
</tbody></table>

## Embedding in Structs

type ที่ถูกฝังไว้ (เช่น mutexes) ควรอยู่บนสุดของรายการใน struct และควรเว้นบรรทัดว่างๆไว้สักบรรทัด

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
type Client struct {
  version int
  http.Client
}
```

</td><td>

```go
type Client struct {
  http.Client

  version int
}
```

</td></tr>
</tbody></table>

## Use Field Names to Initialize Structs

คุณควรระบุชื่อฟิลด์เสมอเมื่อประกาศตัวแปรจาก struct ซึ่งตอนนี้เวลานี้ถูกบังคับโดย [`go vet`] เรียบร้อยแล้ว

[`go vet`]: https://golang.org/cmd/vet/

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
k := User{"John", "Doe", true}
```

</td><td>

```go
k := User{
    FirstName: "John",
    LastName: "Doe",
    Admin: true,
}
```

</td></tr>
</tbody></table>

ข้อยกเว้น: ชื่อฟิลด์ *อาจจะ* ละไว้ได้ในตารางการทดสอบถ้ามันมี 3 ฟิลด์หรือน้อยกว่า

```go
tests := []struct{
  op Operation
  want string
}{
  {Add, "add"},
  {Subtract, "subtract"},
}
```

## Local Variable Declarations

การประกาศตัวแปรแบบสั้น (`:=`) ควรถูกใช้เมื่อต้องการกำหนดค่าให้ตัวแปรอยู่แล้ว

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
var s = "foo"
```

</td><td>

```go
s := "foo"
```

</td></tr>
</tbody></table>

อย่างไรก็ดี บางกรณีการปล่อยให้มันเป็นค่าเริ่มต้นก็อาจจะชัดเจนกว่า ด้วยการใช้คีย์เวิร์ด `var` [Declaring Empty Slices] ตัวอย่างเช่น

[Declaring Empty Slices]: https://github.com/golang/go/wiki/CodeReviewComments#declaring-empty-slices

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
func f(list []int) {
  filtered := []int{}
  for _, v := range list {
    if v > 10 {
      filtered = append(filtered, v)
    }
  }
}
```

</td><td>

```go
func f(list []int) {
  var filtered []int
  for _, v := range list {
    if v > 10 {
      filtered = append(filtered, v)
    }
  }
}
```

</td></tr>
</tbody></table>

## nil is a valid slice

`nil` เป็นค่าที่เหมาะสมที่จะใช้แทน slice ขนาด 0 หมายความว่า

- คุณไม่ควรคืน slice ที่มีขนาดเท่ากับศูนย์ออกไปตรงๆ แต่ให้คืน `nil` ออกไปแทน

  <table>
  <thead><tr><th>Bad</th><th>Good</th></tr></thead>
  <tbody>
  <tr><td>

  ```go
  if x == "" {
    return []int{}
  }
  ```

  </td><td>

  ```go
  if x == "" {
    return nil
  }
  ```

  </td></tr>
  </tbody></table>

- การตรวจสอบว่า slice นั้นว่างเปล่าหรือไม่ ให้ใช้ `len(s) == 0` อย่าไปตรวจสอบ `nil`

  <table>
  <thead><tr><th>Bad</th><th>Good</th></tr></thead>
  <tbody>
  <tr><td>

  ```go
  func isEmpty(s []string) bool {
    return s == nil
  }
  ```

  </td><td>

  ```go
  func isEmpty(s []string) bool {
    return len(s) == 0
  }
  ```

  </td></tr>
  </tbody></table>

- zero value (slice ที่ประกาศด้วย `var`) สามารถใช้งานได้เลย โดยไม่ต้อง `make()` ก่อน

  <table>
  <thead><tr><th>Bad</th><th>Good</th></tr></thead>
  <tbody>
  <tr><td>

  ```go
  nums := []int{}
  // or, nums := make([]int)

  if add1 {
    nums = append(nums, 1)
  }

  if add2 {
    nums = append(nums, 2)
  }
  ```

  </td><td>

  ```go
  var nums []int

  if add1 {
    nums = append(nums, 1)
  }

  if add2 {
    nums = append(nums, 2)
  }
  ```

  </td></tr>
  </tbody></table>

## Reduce Scope of Variables

ถ้ามีโอกาสลดขอบเขตของตัวแปรก็ควรทำ แต่อย่าไปลดมันถ้ามันขัดแย้งกับ [Reduce Nesting](#reduce-nesting)

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
err := ioutil.WriteFile(name, data, 0644)
if err != nil {
 return err
}
```

</td><td>

```go
if err := ioutil.WriteFile(name, data, 0644); err != nil {
 return err
}
```

</td></tr>
</tbody></table>

ถ้าคุณต้องการผลลัพธ์ของฟังก์ชันไปใช้หลัง if ต่อ งั้นคุณก็ไม่ควรลดขอบเขตมัน

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
if data, err := ioutil.ReadFile(name); err == nil {
  err = cfg.Decode(data)
  if err != nil {
    return err
  }

  fmt.Println(cfg)
  return nil
} else {
  return err
}
```

</td><td>

```go
data, err := ioutil.ReadFile(name)
if err != nil {
   return err
}

if err := cfg.Decode(data); err != nil {
  return err
}

fmt.Println(cfg)
return nil
```

</td></tr>
</tbody></table>

## Avoid Naked Parameters

พารามิเตอร์เปลือยๆที่ใส่ไปตอนที่เรียกฟังก์ชัน มันอ่านยาก ให้เพิ่มคอมเม้นท์แบบ C-style ลงไป (`/* ... */`) ให้ความหมายชัดเจนขึ้น

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
// func printInfo(name string, isLocal, done bool)

printInfo("foo", true, true)
```

</td><td>

```go
// func printInfo(name string, isLocal, done bool)

printInfo("foo", true /* isLocal */, true /* done */)
```

</td></tr>
</tbody></table>

แต่มันก็ยังไม่ดีที่สุด เราควรแทนที่ type `bool` ที่เปลือยๆอยู่นี้ด้วยการสร้าง type ขึ้นมาให้มันอ่านง่ายขึ้น และยังรองรับหากในอนาคตต้องการมีมากกว่าสองสถานะ (true/false)

```go
type Region int

const (
  UnknownRegion Region = iota
  Local
)

type Status int

const (
  StatusReady = iota + 1
  StatusDone
  // Maybe we will have a StatusInProgress in the future.
)

func printInfo(name string, region Region, status Status)
```

## Use Raw String Literals to Avoid Escaping

Go สนับสนุน [raw string literals](https://golang.org/ref/spec#raw_string_lit) ซึ่งสามารถใส่ได้หลายบรรทัดรวมทั้งเครื่องหมายคำพูดได้ด้วย ซึ่งการใช้แบบนี้เพื่อป้องกันการทำ hand-escaped เพราะมันจะทำให้อ่านยาก

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
wantError := "unknown name:\"test\""
```

</td><td>

```go
wantError := `unknown error:"test"`
```

</td></tr>
</tbody></table>

## Initializing Struct References

ใช้ `&T{}` แทนการใช้ `new(T)` เมื่อต้องการสร้างตัวแปรแบบอ้างอิงจาก struct จะดูดีกว่า

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
sval := T{Name: "foo"}

// inconsistent
sptr := new(T)
sptr.Name = "bar"
```

</td><td>

```go
sval := T{Name: "foo"}

sptr := &T{Name: "bar"}
```

</td></tr>
</tbody></table>

## Initializing Maps

เสนอให้ใช้ `make(..)` เพื่อสร้าง maps ว่างๆ และเอาไปเขียนโปรแกรมต่อได้ ซึ่งมันทำการประกาศตัวแปรให้พร้อมใช้งานดูมีความต่างจากการประกาศเฉยๆ และมันยังทำให้ง่ายต่อการเพิ่มการใบ้ขนาดให้ในภายหลังด้วย

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
var (
  // m1 is safe to read and write;
  // m2 will panic on writes.
  m1 = map[T1]T2{}
  m2 map[T1]T2
)
```

</td><td>

```go
var (
  // m1 is safe to read and write;
  // m2 will panic on writes.
  m1 = make(map[T1]T2)
  m2 map[T1]T2
)
```

</td></tr>
<tr><td>

การประกาศให้พร้อมใช้งาน กับการประกาศแล้วยังไม่พร้อมใช้งาน ดูคล้ายๆกัน

</td><td>

การประกาศให้พร้อมใช้งาน กับการประกาศแล้วยังไม่พร้อมใช้งาน ดูแตกต่างกัน

</td></tr>
</tbody></table>

ถ้าทำได้ ก็ให้ใบ้ความจุตอนที่ประกาศ maps ด้วยคำสั่ง `make()` ลองดูที่ [Prefer Specifying Map Capacity Hints](#prefer-specifying-map-capacity-hints) สำหรับข้อมูลเพิ่มเติม

หรือในทางกลับกัน ถ้า map นั้นจะต้องเก็บค่าที่แน่นอน ก็ให้ใช้การประกาศด้วยปีกกาได้เลย

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
m := make(map[T1]T2, 3)
m[k1] = v1
m[k2] = v2
m[k3] = v3
```

</td><td>

```go
m := map[T1]T2{
  k1: v1,
  k2: v2,
  k3: v3,
}
```

</td></tr>
</tbody></table>

กฎพื้นฐานของนิ้วหัวแม่มือก็คือ ใช้ปีกกาประกาศเมื่อต้องใส่ค่าคงที่ลงไปตั้งแต่ต้น ไม่เช่นนั้นก็ใช้ `make` (และใส่การใบ้ความจุถ้าทำได้)

## Format Strings outside Printf

ถ้าคุณประกาศการจัดรูปแบบสตริงสำหรับใช้กับฟังก์ชัน `Printf`-style ให้ทำเป็น `const`

มันจะช่วยให้ `go vet` ได้วิเคราะห์การจัดรูปแบบให้

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
msg := "unexpected values %v, %v\n"
fmt.Printf(msg, 1, 2)
```

</td><td>

```go
const msg = "unexpected values %v, %v\n"
fmt.Printf(msg, 1, 2)
```

</td></tr>
</tbody></table>

## Naming Printf-style Functions

เมื่อคุณประกาศฟังก์ชัน `Printf`-style ช่วยทำให้มั่นใจว่า `go vet` จะสามารถตรวจเจอมันและจะได้ตรวจสอบรูปแบบสตริงได้

หมายความว่า คุณควรใช้ชื่อที่ตั้งเผื่อไว้แล้วตามสไตล์ `Printf` ถ้าทำได้ `go vet` จะได้ตรวจสอบได้เอง ดูรายละเอียดเพิ่มเติมได้ที่ [Printf family]

[Printf family]: https://golang.org/cmd/vet/#hdr-Printf_family

ถ้าการใช้ชื่อที่ตั้งเผื่อไว้ ไม่ใช่ทางเลือกของคุณ งั้นก็ให้ตั้งชื่อลงท้ายด้วย f: เช่น `Wrapf` ไม่ใช่ `Wrap` เฉยๆ โดยสามารถบอกให้ `go vet` ตรวจสอบฟังก์ชันสไตล์ `Printf` ได้ แต่มันจะต้องลงท้ายด้วยตัว f เท่านั้น

```shell
$ go vet -printfuncs=wrapf,statusf
```

See also [go vet: Printf family check].

[go vet: Printf family check]: https://kuzminva.wordpress.com/2017/11/07/go-vet-printf-family-check/
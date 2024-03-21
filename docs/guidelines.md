# Guidelines

## Pointers to Interfaces

คุณแทบไม่จำเป็นต้องใช้พอยน์เตอร์เพื่อใส่ใน interface คุณแค่ส่งค่าตรงๆเข้าไป แต่จะส่งเป็นพอยน์เตอร์ก็ได้เช่นกัน

interface ประกอบไปด้วยสองสิ่ง:

1. พอยน์เตอร์ ชี้ไปที่ type ของสิ่งที่เก็บ คุณจะคิดซะว่ามันเป็น "type" เลยก็ได้
2. พอยน์เตอร์ ของสิ่งที่เก็บ ถ้าสิ่งนั้นเป็นพอยน์เตอร์ ก็จะเก็บตรงๆ แต่ถ้ามันเป็นค่าใดๆก็ตาม มันจะเก็บเป็นพอยน์เตอร์ของค่านั้นแทน

ถ้าคุณต้องการให้เมธอดแก้ไขค่าในตัวมันเองได้ด้วย นั่นคุณถึงจะต้องใช้พอยเตอร์

## Receivers and Interfaces

เมธอดที่มีตัวรับเป็นค่าปกติ สามารถเรียกใช้บนตัวแปรพอยน์เตอร์ ได้เลย

For example,

```go
type S struct {
  data string
}

func (s S) Read() string {
  return s.data
}

func (s *S) Write(str string) {
  s.data = str
}

sVals := map[int]S{1: {"A"}}

// คุณเรียกใช้ Read ได้อย่างเดียว
sVals[1].Read()

// This will not compile:
//  sVals[1].Write("test")

sPtrs := map[int]*S{1: {"A"}}

// คุณเรียกใช้ได้ทั้ง Read และ Write ผ่านพอยน์เตอร์
sPtrs[1].Read()
sPtrs[1].Write("test")
```

และในทางกลับกัน interface ยอมให้คุณแทนที่ด้วยพอยน์เตอร์ได้ แม้ว่าเมธอดจะใช้ตัวรับเป็นแค่ค่าปกติ

```go
type F interface {
  f()
}

type S1 struct{}

func (s S1) f() {}

type S2 struct{}

func (s *S2) f() {}

s1Val := S1{}
s1Ptr := &S1{}
s2Val := S2{}
s2Ptr := &S2{}

var i F
i = s1Val
i = s1Ptr
i = s2Ptr

// โค้ดด้านล่างนี้ไม่สามารถทำงานได้ เนื่องจาก s2Val เป็นค่าปกติ ในขณะที่ตัวรับในเมธอดไม่ใช่ค่าปกติแต่เป็นพอยน์เตอร์
```

Effective Go เขียนเรื่องนี้ไว้ได้ดีมากในเรื่อง [Pointers vs. Values]

[Pointers vs. Values]: https://golang.org/doc/effective_go.html#pointers_vs_values

## Zero-value Mutexes are Valid

ค่า zero-value ของ `sync.Mutex` และ `sync.RWMutex` สามารถใช้งานได้โดยไม่ต้อง initial นั่นแปลว่าคุณแทบไม่ต้องใช้พอยน์เตอร์กับ mutex เลย

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
mu := new(sync.Mutex)
mu.Lock()
```

</td><td>

```go
var mu sync.Mutex
mu.Lock()
```

</td></tr>
</tbody></table>

ถ้าคุณใช้ struct ด้วยพอยเตอร์ mutex จะสามารถเป็นแบบ ไม่มีพอยน์เตอร์ให้

struct ที่ไม่ได้เปิดเผยสู่ภายนอกที่ใช้ mutex ปกป้องฟิลด์ในตัวมันเอง อาจจะฝัง mutext ไว้แบบนี้

<table>
<tbody>
<tr><td>

```go
type smap struct {
  sync.Mutex // ใช้เฉพาะ type ที่ไม่เปิดเผยสู่ภายนอก

  data map[string]string
}

func newSMap() *smap {
  return &smap{
    data: make(map[string]string),
  }
}

func (m *smap) Get(k string) string {
  m.Lock()
  defer m.Unlock()

  return m.data[k]
}
```

</td><td>

```go
type SMap struct {
  mu sync.Mutex

  data map[string]string
}

func NewSMap() *SMap {
  return &SMap{
    data: make(map[string]string),
  }
}

func (m *SMap) Get(k string) string {
  m.mu.Lock()
  defer m.mu.Unlock()

  return m.data[k]
}
```

</td></tr>

</tr>
<tr>
<td>การฝัง ใช้กับ type ที่อยู่ภายใน หรือ type ที่ต้องการทำตัวเองเป็น Mutext interface</td>
<td>สำหรับ type ที่ต้องการเปิดเผยสู่ภายนอก ให้ใช้แบบ ฟิลด์ ภายใน struct</td>
</tr>

</tbody></table>

## Copy Slices and Maps at Boundaries

Slices และ maps เก็บของเป็นพอยน์เตอร์ ดังนั้นให้ระมัดระวังเวลาที่จะ copy ค่าเหล่านี้

### Receiving Slices and Maps

ต้องจำไว้นะว่า map หรือ slice ที่คุณรับเข้ามาเป็นอากิวเม้นต์ ก็ถูกคนที่ใช้มันแก้ไขได้ ถ้าคุณเก็บข้อมูลชนิดที่มันอ้างถึงกัน

<table>
<thead><tr><th>Bad</th> <th>Good</th></tr></thead>
<tbody>
<tr>
<td>

```go
func (d *Driver) SetTrips(trips []Trip) {
  d.trips = trips
}

trips := ...
d1.SetTrips(trips)

// คุณต้องการจะแก้ไขค่า d1.trips หรือเปล่า?
trips[0] = ...
```

</td>
<td>

```go
func (d *Driver) SetTrips(trips []Trip) {
  d.trips = make([]Trip, len(trips))
  copy(d.trips, trips)
}

trips := ...
d1.SetTrips(trips)

// ตอนนี้เราก็สามารถแก้ไขค่า trips[0] โดยไม่กระทบ d1.trips ได้แล้ว
trips[0] = ...
```

</td>
</tr>

</tbody>
</table>

### Returning Slices and Maps

ในทางกลับกัน ให้ระมัดระวังการแก้ไขค่าไปที่ map หรือ slices ที่เปิดเผยสู่ภายนอกในระดับภายใน

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
type Stats struct {
  mu sync.Mutex
  counters map[string]int
}

// Snapshot คืน ค่า ณ เวลาปัจจุบัน
func (s *Stats) Snapshot() map[string]int {
  s.mu.Lock()
  defer s.mu.Unlock()

  return s.counters
}

// snapshot ไม่ถูกป้องกันโดย mutex ดังนั้น
// ใครก็ตามที่เข้ามาถึง snapshot อาจจะเกิดการแย่งของกัน
snapshot := stats.Snapshot()
```

</td><td>

```go
type Stats struct {
  mu sync.Mutex
  counters map[string]int
}

func (s *Stats) Snapshot() map[string]int {
  s.mu.Lock()
  defer s.mu.Unlock()

  result := make(map[string]int, len(s.counters))
  for k, v := range s.counters {
    result[k] = v
  }
  return result
}

// Snapshot is now a copy.
snapshot := stats.Snapshot()
```

</td></tr>
</tbody></table>

## Defer to Clean Up

ใช้ defer เพื่อ คืน resource หรือทรัพยากร ที่จองหรือนำไปใช้งานต่างๆเช่น ไฟล์ และ อะไรที่ถูกล็อคไว้

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
p.Lock()
if p.count < 10 {
  p.Unlock()
  return p.count
}

p.count++
newCount := p.count
p.Unlock()

return newCount

// มันง่ายที่ลืมแก้ล็อค เวลาที่มีการรีเทิร์นหลายๆที่
```

</td><td>

```go
p.Lock()
defer p.Unlock()

if p.count < 10 {
  return p.count
}

p.count++
return p.count

// more readable
```

</td></tr>
</tbody></table>

Defer ใช้เวลาทำงานน้อยมาก ถ้าจะไม่ใช้มันก็ต่อเมื่อคุณมั่นใจแล้วว่าฟังก์ชันคุณจะทำงานเร็วในระดับ nanoseconds ถ้าคุณใช้ defer มันอ่านง่ายแน่นอนและคุ้มค่าที่จะใช้ โดยเฉพาะอย่างยิ่งเมื่อคุณมีเมธอดขนาดใหญ่ที่มีการใช้หน่วยความจำแบบท่ายาก และมีการคำนวณอย่างอื่นที่สำคัญกว่า การใช้ `defer`

## Channel Size is One or None

Channels ปกติควรมีขนาดอยู่ที่ 1 หรือไม่มีบัฟเฟอร์เลย โดยค่าตั้งต้น channels จะเป็นแบบไม่มีบัฟเฟอร์ และมีขนาดเป็นศูนย์ ขนาดอื่นๆ ขึ้นอยู่กับวิจารณญาณ ขึ้นอยู่กับว่า จะป้องกันการเติมของ ในขณะที่กำลังโหลด และมีการเขียน อย่างไร

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
// หวังว่าจะพอสำหรับทุกคนนะ!
c := make(chan int, 64)
```

</td><td>

```go
// ให้ขนาดเป็นหนึ่ง
c := make(chan int, 1) // or
// ไม่มีตัวกันชนเลย หรือมีขนาดเท่ากับศูนย์
c := make(chan int)
```

</td></tr>
</tbody></table>

## Start Enums at One

วิธีมาตรฐานในการทำ enum ใน go คือการ สร้าง type ขึ้นมาเอง หรือประกาศเป็นกลุ่ม `const` ด้วยการใช้ `iota` ซึ่งโดยปกติตัวแปรจะมีค่าตั้งต้นเป็น 0 เสมอ เพราะฉะนั้นเวลาที่คุณจะทำ enum ควรจะเริ่มด้วยค่าที่ไม่ใช่ศูนย์นะ

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
type Operation int

const (
  Add Operation = iota
  Subtract
  Multiply
)

// Add=0, Subtract=1, Multiply=2
```

</td><td>

```go
type Operation int

const (
  Add Operation = iota + 1
  Subtract
  Multiply
)

// Add=1, Subtract=2, Multiply=3
```

</td></tr>
</tbody></table>

มันก็มีบางกรณีเหมือนกันที่การใช้ศูนย์อาจจะเหมาะสมกว่า ขึ้นอยู่กับสถานการณ์

```go
type LogOutput int

const (
  LogToStdout LogOutput = iota
  LogToFile
  LogToRemote
)

// LogToStdout=0, LogToFile=1, LogToRemote=2
```

<!-- TODO: section on String methods for enums -->

## Error Types

การสร้าง error ทำได้หลายวิธี:

- [`errors.New`] เมื่อสร้างจากสตริงง่ายๆ
- [`fmt.Errorf`] เมื่อต้องการจัดรูปแบบข้อความ
- สร้าง type ที่ implement `Error` เมธอด
- หุ้ม error ด้วยการใช้ [`"pkg/errors".Wrap`]

เมื่อจะทำการคืน errors ทางเลือกไหนถึงจะดีที่สุด ลองตั้งคำถามดูว่า:

- นี่เป็น error ที่ต้องการข้อมูลเพิ่มเป็นพิเศษไหม ถ้าไม่ ก็ใช้ [`errors.New`] ก็น่าจะพอแล้ว
- คนที่จะเอา error นี้ไปใช้ต่อ เขาต้องการจะสืบหาไหมว่า นี่เป็นความผิดพลาดแบบไหน ถ้าใช่ คุณควรสร้าง type ที่มีเมธอด `Error()` ขึ้นมาใช้เองจะดีกว่า
- คุณต้องการจะบอกคนอื่นไหมว่า นี่เป็น error ที่เกิดขึ้นตรงไหน ถ้าใช่ ลองใช้ตัวนี้ดู [section on error wrapping](#error-wrapping)
- ในกรณีอื่นๆ [`fmt.Errorf`] ก็เป็นตัวเลือกที่ดี

  [`errors.New`]: https://golang.org/pkg/errors/#New
  [`fmt.Errorf`]: https://golang.org/pkg/fmt/#Errorf
  [`"pkg/errors".Wrap`]: https://godoc.org/github.com/pkg/errors#Wrap

ถ้าผู้เรียก ต้องการสืบว่านี่เป็น error อะไร และคุณอยากจะสร้างมันด้วย [`errors.New`] ก็ขอให้ ทำให้มันเป็นตัวแปรดีกว่า

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
// package foo

func Open() error {
  return errors.New("could not open")
}

// package bar

func use() {
  if err := foo.Open(); err != nil {
    if err.Error() == "could not open" {
      // handle
    } else {
      panic("unknown error")
    }
  }
}
```

</td><td>

```go
// package foo

var ErrCouldNotOpen = errors.New("could not open")

func Open() error {
  return ErrCouldNotOpen
}

// package bar

if err := foo.Open(); err != nil {
  if err == foo.ErrCouldNotOpen {
    // handle
  } else {
    panic("unknown error")
  }
}
```

</td></tr>
</tbody></table>

ถ้าคุณมี error ที่ผู้เรียกต้องการสืบหาว่าเป็นแบบไหน แต่คุณก็อยากจะเพิ่มข้อมูลลงไปในนั้น (ไม่ใช่ค่าคงที่) ถ้างั้นคุณก็น่าจะสร้าง type มาใช้เอง

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
func open(file string) error {
  return fmt.Errorf("file %q not found", file)
}

func use() {
  if err := open(); err != nil {
    if strings.Contains(err.Error(), "not found") {
      // handle
    } else {
      panic("unknown error")
    }
  }
}
```

</td><td>

```go
type errNotFound struct {
  file string
}

func (e errNotFound) Error() string {
  return fmt.Sprintf("file %q not found", e.file)
}

func open(file string) error {
  return errNotFound{file: file}
}

func use() {
  if err := open(); err != nil {
    if _, ok := err.(errNotFound); ok {
      // handle
    } else {
      panic("unknown error")
    }
  }
}
```

</td></tr>
</tbody></table>

ขอให้ระมัดระวังการเปิดเผย error type ที่คุณสร้างมันขึ้นมาออกสู่ภายนอกโดยตรง เราแนะทำให้คุณเปิดฟังก์ชันที่ใช้เช็ค type ของ error นี้ออกไปแทนจะดีกว่า

```go
// package foo

type errNotFound struct {
  file string
}

func (e errNotFound) Error() string {
  return fmt.Sprintf("file %q not found", e.file)
}

func IsNotFoundError(err error) bool {
  _, ok := err.(errNotFound)
  return ok
}

func Open(file string) error {
  return errNotFound{file: file}
}

// package bar

if err := foo.Open("foo"); err != nil {
  if foo.IsNotFoundError(err) {
    // handle
  } else {
    panic("unknown error")
  }
}
```

<!-- TODO: Exposing the information to callers with accessor functions. -->

## Error Wrapping

มีสามวิธีที่จะบอกให้ผู้ที่เรียกใช้รู้ว่าการทำงานผิดพลาด:

- คืน error เดิมๆออกไปเลย ถ้าคุณไม่ต้องการเพิ่มคำอธิบายใดๆ และอยากให้เห็น error ดิบๆแบบนั้น
- เพิ่มคำอธิบายลงไปด้วยการใช้ [`"pkg/errors".Wrap`] และใช้ [`"pkg/errors".Cause`] เวลาที่ต้องการถอดเอาเฉพาะ error เดิมออกมา
- ใช้ [`fmt.Errorf`] ถ้าผู้เรียกไม่อยากรู้ว่าเป็น error แบบไหนให้ชัดเจน

เราแนะนำให้เพิ่มคำอธิบายลงไปถ้าทำได้ แทนที่จะให้เห็น error แบบคลุมเครือเช่น "connection refused" แล้วเพิ่มคำอธิบายให้มีประโยชน์มากกว่าลงไป เช่น "call service foo: connection refused"

เวลาที่คุณจะเพิ่มคำอธิบายใน error ให้ใช้ประโยคที่กระชับ แล้วไม่ต้องใส่คำเวิ่นเว้อเช่น "failed to" ไม่งั้นเวลามันผ่านหลายๆชั้นแล้วมันจะดูเป็นคำขยะ:

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
s, err := store.New()
if err != nil {
    return fmt.Errorf(
        "failed to create new store: %s", err)
}
```

</td><td>

```go
s, err := store.New()
if err != nil {
    return fmt.Errorf(
        "new store: %s", err)
}
```

<tr><td>

```
failed to x: failed to y: failed to create new store: the error
```

</td><td>

```
x: y: new store: the error
```

</td></tr>
</tbody></table>

แต่ไม่ว่ายัง เวลาที่ error ถูกส่งไปที่ระบบอื่น มันควรมีความชัดเจนในข้อความ (ตัวอย่างเช่น ติดป้ายว่า `err` หรือใช้คำนำหน้า "Failed" ตอนที่ลง logs)

See also [Don't just check errors, handle them gracefully].

[`"pkg/errors".Cause`]: https://godoc.org/github.com/pkg/errors#Cause
[Don't just check errors, handle them gracefully]: https://dave.cheney.net/2016/04/27/dont-just-check-errors-handle-them-gracefully

## Handle Type Assertion Failures

การรับค่าเดียวตอนที่ทำ [type assertion] มันอาจจะ panic ถ้า type มันไม่ถูก ดังนั้นให้ใช้สำนวนแบบ "comma ok" เสมอ

[type assertion]: https://golang.org/ref/spec#Type_assertions

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
t := i.(string)
```

</td><td>

```go
t, ok := i.(string)
if !ok {
  // handle the error gracefully
}
```

</td></tr>
</tbody></table>

<!-- TODO: There are a few situations where the single assignment form is
fine. -->

## Don't Panic

โค้ดที่จะขึ้น Production อย่าใช้ panics เพราะ Panic เป็นตัวหลักของการเกิด [cascading failures] ถ้ามันเกิด error ขึ้น ก็ให้ฟังก์ชันคืน error ออกไป ให้คนที่เรียกเขาไปตัดสินใจจัดการเอาเองเถิด

[cascading failures]: https://en.wikipedia.org/wiki/Cascading_failure

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
func foo(bar string) {
  if len(bar) == 0 {
    panic("bar must not be empty")
  }
  // ...
}

func main() {
  if len(os.Args) != 2 {
    fmt.Println("USAGE: foo <bar>")
    os.Exit(1)
  }
  foo(os.Args[1])
}
```

</td><td>

```go
func foo(bar string) error {
  if len(bar) == 0 {
    return errors.New("bar must not be empty")
  }
  // ...
  return nil
}

func main() {
  if len(os.Args) != 2 {
    fmt.Println("USAGE: foo <bar>")
    os.Exit(1)
  }
  if err := foo(os.Args[1]); err != nil {
    panic(err)
  }
}
```

</td></tr>
</tbody></table>

Panic/recover ไม่ใช่วิธีการจัดการ error เพราะโปรแกรมจะ panic เฉพาะเมื่อเกิดเหตุที่คาดไม่ถึงเช่น ไปอ้างถึงอะไรก็แล้วแต่ กับค่า nil เว้นแค่จะเป็นช่วงเตรียมของก่อนเริ่มโปรแกรม ถ้าเกิดเหตุที่ไม่คาดคิดก็ควรจะหยุดการทำงานของโปรแกรมไปเลย

```go
var _statusTemplate = template.Must(template.New("name").Parse("_statusHTML"))
```

แม้กระทั่งใน tests ก็แนะนำให้่ใช้ `t.Fatal` หรือ `t.FailNow` มากกว่าการทำให้มัน panic เพื่อบอกให้เทสรู้ว่าเกิดข้อผิดพลาด

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
// func TestFoo(t *testing.T)

f, err := ioutil.TempFile("", "test")
if err != nil {
  panic("failed to set up test")
}
```

</td><td>

```go
// func TestFoo(t *testing.T)

f, err := ioutil.TempFile("", "test")
if err != nil {
  t.Fatal("failed to set up test")
}
```

</td></tr>
</tbody></table>

<!-- TODO: Explain how to use _test packages. -->

## Use go.uber.org/atomic

ตัวทำ automic ในแพ็กเกจ [sync/automic] ใช้ได้กับ type ดิบๆ (`int32`, `int64`, etc.) เราเลยลืมที่จะใช้มันเวลาจะอ่านหรือแก้ไขค่าตัวแปร

[go.uber.org/atomic] ได้เพิ่ม type ที่ปลอดภัยเข้าไปอีก โดยซ่อน type จริงๆไว้ข้างล่าง นอกจากนี้ยังเพิ่ม type `atomic.Bool` เพื่อให้สะดวกขึ้นอีก

[go.uber.org/atomic]: https://godoc.org/go.uber.org/atomic
[sync/atomic]: https://golang.org/pkg/sync/atomic/

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
type foo struct {
  running int32  // atomic
}

func (f* foo) start() {
  if atomic.SwapInt32(&f.running, 1) == 1 {
     // already running…
     return
  }
  // start the Foo
}

func (f *foo) isRunning() bool {
  return f.running == 1  // race!
}
```

</td><td>

```go
type foo struct {
  running atomic.Bool
}

func (f *foo) start() {
  if f.running.Swap(true) {
     // already running…
     return
  }
  // start the Foo
}

func (f *foo) isRunning() bool {
  return f.running.Load()
}
```

</td></tr>
</tbody></table>
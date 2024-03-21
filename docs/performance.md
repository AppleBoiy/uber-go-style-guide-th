# Performance

คำแนะนำโดยตรงเกี่ยวกับประสิทธิภาพ คือทำเฉพาะส่วนที่เป็น hot path (ส่วนที่ถูกเรียกใช้งานหนักๆ)

## Prefer strconv over fmt

เมื่อต้องการแปลงชนิดไปมา กับสตริง ให้ใช้ `strconv` จะเร็วกว่าใช้ `fmt`

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
for i := 0; i < b.N; i++ {
  s := fmt.Sprint(rand.Int())
}
```

</td><td>

```go
for i := 0; i < b.N; i++ {
  s := strconv.Itoa(rand.Int())
}
```

</td></tr>
<tr><td>

```
BenchmarkFmtSprint-4    143 ns/op    2 allocs/op
```

</td><td>

```
BenchmarkStrconv-4    64.2 ns/op    1 allocs/op
```

</td></tr>
</tbody></table>

## Avoid string-to-byte conversion

อย่าสร้าง slices ของ byte จากสตริงในลูป ให้ทำครั้งเดียวพอ

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
for i := 0; i < b.N; i++ {
  w.Write([]byte("Hello world"))
}
```

</td><td>

```go
data := []byte("Hello world")
for i := 0; i < b.N; i++ {
  w.Write(data)
}
```

</tr>
<tr><td>

```
BenchmarkBad-4   50000000   22.2 ns/op
```

</td><td>

```
BenchmarkGood-4  500000000   3.25 ns/op
```

</td></tr>
</tbody></table>

## Prefer Specifying Map Capacity Hints

ถ้าทำได้ ให้บอกใบ้ขนาดให้กับ map ตอนที่เรียก `make()`

```go
make(map[T1]T2, hint)
```

การใบ้ค่าความจุกับ `make()` อย่างน้อยพยายามให้มันใกล้เคียงที่สุด ตอนที่สร้าง map จะช่วยลดเวลาตอนที่ต้องเพิ่มขนาดมันทีหลัง ซึ่งอันที่จริงการใส่ความจุแบบนี้ก็ไม่รับประกันว่ามันจะไม่เสียเวลา เพราะบางทีการเพิ่มของเข้าไปก็อาจจะเกิดกระบวนการจองหน่วยความจำได้ ทั้งๆที่ก็ได้ให้ความจุไปก่อนแล้ว

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
m := make(map[string]os.FileInfo)

files, _ := ioutil.ReadDir("./files")
for _, f := range files {
    m[f.Name()] = f
}
```

</td><td>

```go

files, _ := ioutil.ReadDir("./files")

m := make(map[string]os.FileInfo, len(files))
for _, f := range files {
    m[f.Name()] = f
}
```

</td></tr>
<tr><td>

`m` ถูกสร้างโดยไม่ใบ้ขนาดให้มัน ซึ่งมันอาจจะทำให้ต้องเสียเวลาจังหวะที่จะจองหน่วยความจำ

</td><td>

`m` ถูกสร้างโดยใบ้ขนาดให้ด้วย ซึ่งจะเสียเวลาจองหน่วยความจำนิดเดียว

</td></tr>
</tbody></table>
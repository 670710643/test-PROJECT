# 7. Common Mistakes

## Mistake 1 — พยายามคืน `Fn(...)` ตรง ๆ จากฟังก์ชัน

**Problem**

ต้องการคืน Closure จากฟังก์ชันโดยระบุชนิดคืนค่าเป็น `Fn(...)` ตรง ๆ แต่เกิด error เพราะ `Fn(...)` เป็น trait ไม่ใช่ concrete type และ trait ตรง ๆ ไม่มีขนาดแน่นอนในช่วง compile time ขณะที่ Rust ต้องรู้ขนาดของค่าที่ฟังก์ชันจะคืนเสมอ

**Incorrect Code**

```rust
fn factory() -> Fn(i32) -> i32 {
    let num = 5;

    |x| x + num
}

fn main() {
    let f = factory();
    println!("{}", f(1));
}
```
**Output**
```rust
error[E0277]: the trait bound Fn(i32) -> i32: Sized is not satisfied

```

**Correct Code**

```rust
fn factory() -> Box<dyn Fn(i32) -> i32> {
    let num = 5;

    Box::new(move |x| x + num)
}

fn main() {
    let f = factory();
    println!("{}", f(1));
}
```
**Output**
```rust
6
```

**Why?**

ในโค้ดนี้มีการพยายามคืนค่าเป็น:

```rust
Fn(i32) -> i32
```

แต่ `Fn(i32) -> i32` เป็น trait ไม่ใช่ชนิดข้อมูลแบบ concrete ที่มีขนาดแน่นอน
Rust จึงไม่สามารถรู้ได้ว่าค่าที่จะคืนจากฟังก์ชันมีขนาดเท่าไรในช่วง compile time

ฟังก์ชันใน Rust ต้องมีชนิดคืนค่าที่ทราบขนาดแน่นอน เว้นแต่จะใช้การห่อด้วยชนิดที่มีขนาดแน่นอน เช่น pointer หรือ smart pointer

ดังนั้นแนวทางที่ใช้ได้คือห่อ Closure ด้วย `Box<dyn Fn(i32) -> i32>`:

```rust
fn factory() -> Box<dyn Fn(i32) -> i32> {
    let num = 5;

    Box::new(move |x| x + num)
}
```
`Box` มีขนาดแน่นอน จึงสามารถใช้เป็นชนิดคืนค่าของฟังก์ชันได้ ส่วน `dyn Fn(...)` คือ trait object ที่ใช้แทน Closure ที่แท้จริงซึ่งมีชนิดเป็น anonymous type


ใน Rust สมัยใหม่ อีกทางเลือกหนึ่งคือใช้ `impl Fn(i32) -> i32` หากฟังก์ชันคืน Closure เพียงชนิดเดียว
```rust
fn factory() -> impl Fn(i32) -> i32 {
    let num = 5;
    move |x| x + num
}
```

---

## Mistake 2 — ลืมใช้ `move` ตอนคืน Closure ที่ capture ตัวแปรภายในฟังก์ชัน

**Problem**

ต้องการคืน Closure จากฟังก์ชัน และ Closure นั้นมีการใช้งานตัวแปร `num` ที่อยู่ภายในฟังก์ชันเดียวกัน แต่เกิด error เพราะแม้จะใช้ `Box` เพื่อเก็บ Closure แล้ว ก็ยังไม่เพียงพอ หาก Closure ยัง borrow ตัวแปรจาก stack frame เดิมของฟังก์ชันอยู่ จึงต้องใช้ `move` เพื่อย้ายค่าที่ capture เข้าไปใน Closure โดยตรง

**Incorrect Code**

```rust
fn factory() -> Box<dyn Fn(i32) -> i32> {
    let num = 5;

    Box::new(|x| x + num)
}

fn main() {
    let f = factory();
    println!("{}", f(1));
}

```

**Output**
```rust
error[E0373]: closure may outlive the current function, but it borrows `num`
```

**Correct Code**

```rust
fn factory() -> Box<dyn Fn(i32) -> i32> {
    let num = 5;

    Box::new(move |x| x + num)
}

fn main() {
    let f = factory();
    println!("{}", f(1));
}
```
**Output**
```rust
6
```

**Why?**

ในโค้ดนี้ Closure มีการอ้างถึงตัวแปร `num`

```rust
|x| x + num
```
ถ้าไม่ใส่ `move` Rust จะพยายามให้ Closure borrow `num` จาก scope เดิมของฟังก์ชัน `factory()`

ปัญหาคือเมื่อ `factory()` ทำงานจบลง ตัวแปร `num` จะถูกทำลายไปพร้อมกับ stack frame ของฟังก์ชันนั้น ทำให้ Closure ที่ถูกคืนออกไปไม่สามารถอ้างถึง `num` ที่ยืมมาได้อย่างปลอดภัย

ดังนั้นจึงต้องใช้ `move`:

```rust
Box::new(move |x| x + num)
```
เพื่อย้ายค่า `num` เข้าไปเก็บใน Closure environment โดยตรง ทำให้ Closure มีข้อมูลของตัวเองและสามารถถูกคืนออกจากฟังก์ชันได้อย่างปลอดภัย

---

# 8. Exercises

## Exercise 1 — Once Function

**Problem**

จงเขียน Higher-Order Function ชื่อ `once(fn)` ที่รับฟังก์ชัน `fn` เข้ามา แล้วคืนค่าเป็นฟังก์ชันใหม่ที่สามารถ เรียกทำงานได้เพียงครั้งเดียวเท่านั้น
เมื่อเรียกใช้งานครั้งแรก ให้ทำการประมวลผลและคืนค่าผลลัพธ์ของ `fn` ตามปกติ
หากมีการเรียกใช้งานในครั้งถัด ๆ ไป ไม่ว่าจะส่งพารามิเตอร์อะไร ให้คืนค่าผลลัพธ์เดิมที่เคยคำนวณได้จากครั้งแรก โดยไม่มีการเรียกใช้ `fn` ซ้ำอีก


**Hint**

ใช้ Closure ในการเก็บสถานะด้วยตัวแปร `Boolean` (เช่น `hasRun`) เพื่อเช็กว่าฟังก์ชันเคยถูกเรียกหรือยัง และเก็บตัวแปร `result` เพื่อจำผลลัพธ์จากการเรียกใช้งานครั้งแรกไว้

**Solution**
```javascript
function once(fn) {
  let hasRun = false;
  let result;

  return function (...args) {
    if (!hasRun) {
      result = fn(...args);
      hasRun = true;
    }
    return result;
  };
}

// ตัวอย่างการใช้งาน
const initializeApp = once((appName) => {
  console.log(`Initializing ${appName}...`);
  return { status: 'ready', appName };
});

console.log(initializeApp('MySystem'));
// Output: "Initializing MySystem..." 
// Returns: { status: 'ready', appName: 'MySystem' }

console.log(initializeApp('OtherSystem'));
// Output: (ไม่มีการพิมพ์)
// Returns: { status: 'ready', appName: 'MySystem' }
```

**Explanation**

1. `once` สร้างขอบเขตตัวแปรด้วย Closure เพื่อเก็บตัวแปร `hasRun` และ `result`

2. ฟังก์ชันที่ถูกคืนค่ากลับไปจะตรวจสอบ `hasRun` ก่อนเสมอ

3. ในการเรียกครั้งแรก `hasRun` ยังเป็น `false` ทำให้โค้ดรันฟังก์ชัน `fn(...args)` บันทึกผลลัพธ์ลง `result` แล้วเปลี่ยน `hasRun` เป็น `true`

4. การเรียกครั้งถัดไป `hasRun` เป็น `true` แล้ว ระบบจะข้ามการประมวลผล `fn` และคืนค่า `result` เดิมทันที

---

## Exercise 2 — Higher-Order Array Filter Generator

**Problem**

จงเขียนฟังก์ชัน `createFilter(property, conditionFn)` ที่รับชื่อ `property` ของ Object และฟังก์ชันเงื่อนไข `conditionFn` จากนั้นคืนค่าเป็น Predicate Function ที่สามารถนำไปใช้กับ `.filter()` ของ Array เพื่อคัดกรองข้อมูลตามเงื่อนไขที่กำหนดได้

**Hint**

ใช้หลักการ Higher-Order Function และ Closure โดยฟังก์ชัน `createFilter` จะคืนค่าฟังก์ชันที่รับออบเจกต์ `item` เข้ามา แล้วนำค่า `item[property]` ไปส่งต่อให้ `conditionFn(val)` เพื่อรีเทิร์นค่า Boolean (`true`/`false`)

**Solution**

```javascript
function createFilter(property, conditionFn) {
  return function (item) {
    // เข้าถึง property ของ item แล้วส่งให้ conditionFn ประมวลผล
    return conditionFn(item[property]);
  };
}

// ตัวอย่างการใช้งาน
const products = [
  { name: 'Laptop', price: 1200 },
  { name: 'Mouse', price: 25 },
  { name: 'Keyboard', price: 75 }
];

// สร้าง Filter Reusable Functions
const isPriceOver50 = createFilter('price', (price) => price > 50);
const isNameStartsWithK = createFilter('name', (name) => name.startsWith('K'));

console.log(products.filter(isPriceOver50));
// Output: [ { name: 'Laptop', price: 1200 }, { name: 'Keyboard', price: 75 } ]

console.log(products.filter(isNameStartsWithK));
// Output: [ { name: 'Keyboard', price: 75 } ]
```

**Explanation**

1. `createFilter` ทำหน้าที่เป็น Factory สร้างฟังก์ชันสำหรับคัดกรอง โดยจำค่า `property` และ `conditionFn` ไว้ใน Closure

2. ฟังก์ชันที่ถูกคืนค่ากลับมาจะรับ `item จาก` `.filter()` ทีละตัว แล้วดึงค่า `item[property]` ออกมา

3. ส่งค่านั้นเข้าไปใน `conditionFn` เพื่อคืนค่ากลับมาเป็น `true` หรือ `false` ให้กับ `.filter()`

4. วิธีนี้ช่วยให้เราเขียนโค้ดสไตล์ Functional Programming ที่อ่านง่าย และสามารถนำ Filter Logic กลับมาใช้ซ้ำ (Reusable) ได้อย่างยืดหยุ่น

---




















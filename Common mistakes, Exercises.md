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

จงเขียน Higher-Order Function ชื่อ `once` ที่รับ Closure `fn` เข้ามา แล้วคืนค่าเป็น Closure ใหม่ที่สามารถเรียกใช้งานฟังก์ชัน `fn` ได้เพียงครั้งเดียว เมื่อเรียกใช้งานครั้งแรก ให้ Closure ทำการประมวลผลและเก็บผลลัพธ์ที่ได้ไว้ หากมีการเรียกใช้งานในครั้งถัดไป ไม่ว่าจะส่ง Parameter อะไรเข้ามา ให้คืนค่าผลลัพธ์เดิมที่คำนวณได้จากครั้งแรก โดยไม่เรียกใช้ `fn` ซ้ำอีก

**Hint**

ใช้ Closure ในการเก็บสถานะ โดยเก็บผลลัพธ์ที่คำนวณได้จากการเรียกครั้งแรกไว้ภายใน Closure

เนื่องจาก Closure ที่คืนออกมาต้องสามารถเปลี่ยนแปลงสถานะภายในได้ จึงสามารถใช้ `FnMut` ได้

**Solution**

```rust
fn once<F, A, R>(mut f: F) -> impl FnMut(A) -> R
where
    F: FnMut(A) -> R,
    R: Clone,
{
    let mut result: Option<R> = None;

    move |arg| {
        if let Some(value) = &result {
            return value.clone();
        }

        let value = f(arg);
        result = Some(value.clone());

        value
    }
}

fn main() {
    let mut initialize_app = once(|app_name: &str| {
        println!("Initializing {}...", app_name);
        format!("{} is ready", app_name)
    });

    println!("{}", initialize_app("MySystem"));
    println!("{}", initialize_app("OtherSystem"));
}
```

**Output**

```text
Initializing MySystem...
MySystem is ready
MySystem is ready
```

**Explanation**

1. `once` เป็น Higher-Order Function เพราะรับ Closure `f` เข้ามาเป็น Argument และคืน Closure กลับออกมา

2. ตัวแปร `result` ใช้สำหรับเก็บผลลัพธ์จากการเรียก `f` ครั้งแรก

   ```rust
   let mut result: Option<R> = None;
   ```

   ในตอนเริ่มต้น `result` ยังไม่มีค่า จึงเป็น `None`

3. `move` ทำให้ Closure ที่ถูกคืนออกมาสามารถเป็นเจ้าของ `result` และ `f` ได้เอง

   ```rust
   move |arg| {
       ...
   }
   ```

4. ในการเรียกครั้งแรก `result` เป็น `None` ดังนั้น `f(arg)` จะถูกเรียก:

   ```rust
   let value = f(arg);
   ```

   จากนั้นผลลัพธ์จะถูกเก็บไว้ใน `result`

5. ในการเรียกครั้งถัดไป `result` มีค่าแล้ว:

   ```rust
   if let Some(value) = &result {
       return value.clone();
   }
   ```

   Closure จึงไม่เรียก `f` อีก แต่คืนผลลัพธ์เดิมออกมา

6. `R: Clone` จำเป็นในตัวอย่างนี้ เพราะผลลัพธ์ `R` ต้องสามารถถูกนำกลับมาคืนซ้ำในการเรียกครั้งถัดไป Closure ที่คืนออกมาจึงมี State ของตัวเอง คือค่า `result` ที่ถูกเก็บไว้ภายใน Closure

แนวคิดสำคัญของ Exercise นี้คือ **Closure สามารถเก็บ State และรักษา State นั้นไว้ระหว่างการเรียกใช้งานแต่ละครั้ง** ซึ่งเป็นหนึ่งในคุณสมบัติสำคัญของ Closure ใน Rust

---

## Exercise 2 — Higher-Order Iterator Filter Generator

**Problem**

จงเขียน Higher-Order Function ชื่อ `create_filter(property, condition)` ที่รับ Closure สำหรับดึงค่าจาก `item` และฟังก์ชันเงื่อนไข `condition` จากนั้นคืนค่าเป็น Predicate Function ที่สามารถนำไปใช้กับ `.filter()` ของ Iterator เพื่อคัดกรองข้อมูลตามเงื่อนไขที่กำหนดได้

**Hint**

ใช้หลักการ Higher-Order Function และ Closure โดยฟังก์ชัน `create_filter` จะรับ Closure `property` สำหรับดึงค่าที่ต้องการจาก `item` และรับ Closure `condition` สำหรับตรวจสอบค่านั้น จากนั้นคืน Closure ที่รับ `item` และส่งค่าที่ดึงออกมาให้ `condition`

**Solution**

```rust
fn create_filter<T, V, P>(
    property: impl Fn(&T) -> V,
    condition: P,
) -> impl Fn(&T) -> bool
where
    P: Fn(V) -> bool,
{
    move |item| {
        let value = property(item);
        condition(value)
    }
}

fn main() {
    let products = vec![
        ("Laptop", 1200),
        ("Mouse", 25),
        ("Keyboard", 75),
    ];

    let is_price_over_50 =
        create_filter(|product: &(&str, i32)| product.1, |price| price > 50);

    let result: Vec<_> = products
        .iter()
        .filter(|product| is_price_over_50(product))
        .collect();

    println!("{:?}", result);
}
```

**Output**

```text
[("Laptop", 1200), ("Keyboard", 75)]
```

**Explanation**

1. `create_filter` เป็น Higher-Order Function เพราะรับ Closure เข้ามาเป็น Argument และคืน Closure กลับออกมา

2. `property` ทำหน้าที่กำหนดว่าเราต้องการดึงข้อมูลส่วนไหนจาก `item`

3. `condition` ทำหน้าที่ตรวจสอบค่าที่ `property` ดึงออกมา และต้องคืนค่า `bool`

4. `create_filter` คืน Closure นี้ออกมา:

```rust
move |item| {
    let value = property(item);
    condition(value)
}
```

Closure ที่คืนออกมาจึงมีหน้าที่รับ `item` แล้วนำไปผ่านขั้นตอน `property` → `condition`

5. เมื่อใช้กับ `.filter()`:

```rust
products
    .iter()
    .filter(|product| is_price_over_50(product))
```

`.filter()` จะส่งแต่ละ `item` เข้ามาให้ `is_price_over_50` และ Closure จะคืน `true` หรือ `false` เพื่อกำหนดว่าจะเก็บ `item` นั้นไว้หรือไม่

จุดสำคัญคือ `property` และ `condition` ถูกเก็บไว้ใน Closure ที่ `create_filter` คืนกลับมา ทำให้เราสามารถสร้าง Predicate Function ที่นำกลับมาใช้ซ้ำได้

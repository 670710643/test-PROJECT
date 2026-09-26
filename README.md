# 7. Common Mistakes
# Mistake 1 — Closure แชร์ State โดยไม่ตั้งใจ

**Problem**

การกำหนด counter2 = counter1 ทำให้ counter1 และ counter2 อ้างอิง Object เดียวกัน จึงใช้ Closure และ State (count) ชุดเดียวกัน เมื่อ counter1 เปลี่ยนค่า count ค่าเดียวกันนั้นจึงสามารถเข้าถึงได้ผ่าน counter2 ด้วย

**Incorrect Code**

```javascript
function createCounter() {
  let count = 0;

  return {
    increment: () => ++count,
    getCount: () => count
  };
}

const counter1 = createCounter();
const counter2 = counter1;

counter1.increment();
counter1.increment();

console.log(counter2.getCount());

```
**Output**
```javascript
2
```

**Correct Code**

```javascript
function createCounter() {
  let count = 0;

  return {
    increment: () => ++count,
    getCount: () => count
  };
}

const counter1 = createCounter();
const counter2 = createCounter();

counter1.increment();
counter1.increment();

console.log(counter1.getCount());
console.log(counter2.getCount());

```
**Output**
```javascript
2
0
```

**Why?**

สิ่งสำคัญคือความแตกต่างระหว่าง:

```javascript
const counter2 = counter1;
```
ไม่ได้สร้าง Counter และ Closure ใหม่ แต่ทำให้ counter2 อ้างอิง Object เดียวกับ counter1

เป็นการอ้างอิง Object เดียวกัน ทำให้ใช้ State เดียวกัน

```javascript
const counter1 = createCounter();
const counter2 = createCounter();
```
แต่ละครั้งจะสร้าง Object, Closure และตัวแปร count ชุดใหม่

```javascript
counter1 → Closure → count = 2
counter2 → Closure → count = 0
```
ดังนั้นแต่ละ Counter จึงมี State แยกจากกัน

เป็นการสร้าง Object และ Closure ใหม่ ทำให้แต่ละ Counter มี State เป็นของตัวเอง


---

# Mistake 2 — Closure เก็บ &mut ทำให้ใช้ตัวแปรข้างนอกไม่ได้

**Problem**

ต้องการให้ Closure เพิ่มค่า count แล้วหลังจากเรียก Closure ต้องการนำ count ไปใช้ต่อ แต่เกิด error เพราะ Closure ยังถือ mutable borrow อยู่

**Incorrect Code**

```javascript
fn main() {
    let mut count = 0;

    let add = || {
        count += 1;
    };

    add();

    println!("{}", count);
}
```
**Output**
```javascript
error[E0596]: cannot borrow `add` as mutable, as it is not declared as mutable

```

**Correct Code**

```javascript
fn main() {
    let mut count = 0;

    let mut add = || {
        count += 1;
    };

    add();

    println!("{}", count);
}
```
**Output**
```javascript
1
```

**Why?**

Closure มีการแก้ไข count:
```javascript
count += 1;
```
ดังนั้น Closure ต้อง mutably borrow count

จึงทำให้ Closure นี้เป็น FnMut และตัวแปรที่เก็บ Closure ต้องประกาศเป็น mut:

```javascript
let mut add = || {
    count += 1;
};
```
จากนั้นจึงเรียก:

```javascript
add();
```
การเรียก Closure สามารถเปลี่ยนแปลง state ที่มัน capture เอาไว้ได้ ดังนั้นตัวแปร add ที่เก็บ Closure จึงต้องประกาศเป็น mut ด้วย ไม่ใช่แค่ count ที่ต้องเป็น mut เท่านั้น เพราะ Rust มองว่าการเรียก add() ในกรณีนี้เป็นการใช้งาน Closure แบบ mutable หากเขียน let add = ... ตัว add จะไม่สามารถถูกยืมแบบ mutable ตอนเรียก add() ได้ จึงเกิด error ขึ้น การแก้ปัญหาคือเปลี่ยนเป็น let mut add = ... เพื่ออนุญาตให้ Closure ถูกเรียกในลักษณะที่สามารถเปลี่ยนแปลงค่าที่มัน capture ไว้ได้

---

# 8. Exercises

# Exercise 1 — `Once Function`

**Problem**

`จงเขียน Higher-Order Function ชื่อ once(fn) ที่รับฟังก์ชัน fn เข้ามา แล้วคืนค่าเป็นฟังก์ชันใหม่ที่สามารถ เรียกทำงานได้เพียงครั้งเดียวเท่านั้น
เมื่อเรียกใช้งานครั้งแรก ให้ทำการประมวลผลและคืนค่าผลลัพธ์ของ fn ตามปกติ
หากมีการเรียกใช้งานในครั้งถัดๆ ไป ไม่ว่าจะส่งพารามิเตอร์อะไรมาก็ตาม ให้คืนค่าผลลัพธ์เดิมที่เคยคำนวณได้จากครั้งแรก โดยไม่มีการเรียกใช้ fn ซ้ำอีก
`

**Hint**

`ใช้ Closure ในการเก็บสถานะด้วยตัวแปร Boolean (เช่น hasRun) เพื่อเช็กว่าฟังก์ชันเคยถูกเรียกหรือยัง และเก็บตัวแปร result เพื่อจำผลลัพธ์จากการเรียกใช้งานครั้งแรกไว้`

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

1.) once สร้างขอบเขตตัวแปรด้วย Closure เพื่อเก็บตัวแปร hasRun และ result

2.) ฟังก์ชันที่ถูกคืนค่ากลับไปจะตรวจสอบ hasRun ก่อนเสมอ

3.) ในการเรียกครั้งแรก hasRun ยังเป็น false ทำให้โค้ดรันฟังก์ชัน fn(...args) บันทึกผลลัพธ์ลง result แล้วเปลี่ยน hasRun เป็น true

4.) การเรียกครั้งถัดไป hasRun เป็น true แล้ว ระบบจะข้ามการประมวลผล fn และคืนค่า result เดิมทันที

---

# Exercise 2 — `Higher-Order Array Filter Generator`

**Problem**

`จงเขียนฟังก์ชัน createFilter(property, conditionFn) ที่รับชื่อ property ของ Object และฟังก์ชันเงื่อนไข conditionFn จากนั้นคืนค่าเป็น Predicate Function ที่สามารถนำไปใช้กับ .filter() ของ Array เพื่อคัดกรองข้อมูลตามเงื่อนไขที่กำหนดได้`

**Hint**

`ใช้หลักการ Higher-Order Function และ Closure โดยฟังก์ชัน createFilter จะคืนค่าฟังก์ชันที่รับออบเจกต์ item เข้ามา แล้วนำค่า item[property]  ไปส่งต่อให้ conditionFn(val) เพื่อรีเทิร์นค่า Boolean (true/false)`

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

1.) createFilter ทำหน้าที่เป็น Factory สร้างฟังก์ชันสำหรับคัดกรอง โดยจำค่า property และ conditionFn ไว้ใน Closure

2.) ฟังก์ชันที่ถูกคืนค่ากลับมาจะรับ item จาก .filter() ทีละตัว แล้วดึงค่า item[property] ออกมา

3.) ส่งค่านั้นเข้าไปใน conditionFn เพื่อคืนค่ากลับมาเป็น true หรือ false ให้กับ .filter()

4.) วิธีนี้ช่วยให้เราเขียนโค้ดสไตล์ Functional Programming ที่อ่านง่าย และสามารถนำ Filter Logic กลับมาใช้ซ้ำ (Reusable) ได้อย่างยืดหยุ่น

---




















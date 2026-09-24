# 6. Common Mistakes 
# Mistake 1 — [การใช้ var ใน Loop ที่มีโค้ด Asynchronous]

**Problem**

[การใช้ var ประกาศตัวแปรใน Loop ที่มีโค้ด Asynchronous เช่น setTimeout() อาจทำให้ Callback ทุกตัวอ้างอิงถึงตัวแปร i เดียวกัน เมื่อ Callback ทำงานภายหลัง Loop จบไปแล้ว ค่า i จึงกลายเป็นค่าหลังจากจบ Loop]

**Incorrect Code**

```javascript
for (var i = 1; i <= 3; i++) {
  setTimeout(() => {
    console.log(i); // พิมพ์ค่า 4 ออกมา 3 รอบ
  }, 1000);
}
```
**Output**
```javascript
4
4
4
```

**Correct Code**

```javascript
for (let i = 1; i <= 3; i++) {
  setTimeout(() => {
    console.log(i); // พิมพ์ค่า 1, 2, 3 ตามลำดับ
  }, 1000);
}
```
**Output**
```javascript
1
2
3
```

**Why?**

[var มีขอบเขตแบบ Function Scope ดังนั้นในตัวอย่างนี้จึงมีตัวแปร i ที่ Callback ทั้งหมดอ้างอิงร่วมกัน เมื่อ for Loop ทำงานจนจบ ค่า i จะเป็น 4 แล้ว และ Callback ที่ทำงานภายหลังจึงอ่านค่า 4 เหมือนกันทั้งหมด

ในทางกลับกัน let มีขอบเขตแบบ Block Scope และใน for Loop จะมี binding ของ i สำหรับแต่ละรอบ ทำให้ Callback ของแต่ละรอบสามารถเข้าถึงค่าของ i ที่สอดคล้องกับรอบนั้น]

---

# Mistake 2 — [เปลี่ยนแปลงข้อมูลใน State หรือ Array โดยตรง (Mutating State)]

**Problem**

[การแก้ไข Array หรือ Object ที่อยู่ใน State โดยตรง อาจทำให้ React ไม่ตรวจพบการเปลี่ยนแปลงตามที่คาดไว้ เพราะเราไม่ได้สร้าง Reference ใหม่ให้กับ State]

**Incorrect Code**

```javascript
const [items, setItems] = useState(['A', 'B']);

const addItem = () => {
  items.push('C'); // ยัดค่าลง Array เดิมตรงๆ 
  setItems(items); // React จะมองว่าเป็น Array ตัวเดิม จึงไม่ re-render
};
```
**Output**
```javascript
['A', 'B']
```

**Correct Code**

```javascript
const [items, setItems] = useState(['A', 'B']);

const addItem = () => {
  setItems([...items, 'C']); // สร้าง Array ใหม่ขึ้นมารับค่าเดิม + ค่าใหม่
};
```
**Output**
```javascript
['A', 'B', 'C']
```

**Why?**

[การใช้ .push() เป็นการ Mutate Array เดิม ซึ่งหมายความว่า Reference ของ Array ไม่เปลี่ยนแปลง 

ในทางกลับกัน Spread Operator: setItems([...items, 'C']); จะสร้าง Array object ใหม่ขึ้นมา โดยนำสมาชิกเดิมมารวมกับ 'C']

---

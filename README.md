# 517-312 Operating Systems

# TERM PROJECT - Mini Job Scheduler & Shared Resource Manager

## Group Member
## 1.นางสาวกัญญรัชต์ สมหวังพรเจริญ รหัสนักศึกษา 670710122
## 2.นางสาวอนัญญณัชช์ ขจรศิริผล รหัสนักศึกษา 670710637
## 3.นางสาวจุฑารัตน์ รู้วงษ์ รหัสนักศึกษา 670710643

## 1. Compile 

เปิด Terminal (PowerShell) ใน VS Code แล้วเข้าโฟลเดอร์ `src`

```powershell
cd src
javac *.java
```

## 2. Run

รูปแบบคำสั่งสำหรับรันโปรแกรม:

```text
java -cp . Main <workload.csv> <fcfs|priority> <workers> <printerPermits> <databasePermits>
```

**ตัวอย่างการรัน FCFS**

```powershell
java -cp . Main "..\OS input\jobs_standard.csv" fcfs 3 1 2
```

**ตัวอย่างการรัน Priority**

```powershell
java -cp . Main "..\OS input\jobs_standard.csv" priority 3 1 2
```

## 3. Command-Line Arguments (ค่าที่ใส่ตอนรันโปรแกรม)

| Argument | Description |
|---|---|
| `workload.csv` | ตำแหน่งไฟล์ CSV ที่เก็บข้อมูล Job |
| `fcfs\|priority` | เลือกนโยบายการจัดตารางงาน FCFS หรือ Priority |
| `workers` | จำนวน Worker Threads ที่ใช้ประมวลผล Job (มากกว่า 0) |
| `printerPermits` | จำนวน Permits สำหรับ Printer Semaphore (มากกว่า 0) |
| `databasePermits` | จำนวน Permits สำหรับ Database Semaphore (มากกว่า 0) |

**หมายเหตุ:** ตัวอย่างคำสั่งสมมติว่าโฟลเดอร์ `src` และ `OS input` อยู่ภายใต้โฟลเดอร์โปรเจกต์เดียวกัน

## 4. Limitations (ข้อจำกัดที่ควรรู้)

- **Non-preemptive Scheduling:** โปรแกรมไม่สามารถขัดจังหวะ Job ที่กำลังทำงานอยู่ได้ แม้จะมี Job ที่ Priority สูงกว่าเข้ามาใหม่
- **Shared Resources:** Job แต่ละตัวสามารถใช้ทรัพยากรได้เพียง 1 ประเภท ได้แก่ `NONE`, `PRINTER` หรือ `DATABASE`
- **Semaphore Permits:** จำนวน Job ที่ใช้ Printer หรือ Database พร้อมกันถูกจำกัดตามจำนวน Permits ที่กำหนด
- **Execution Time:** ค่า Waiting Time, Turnaround Time และ Throughput อาจแตกต่างกันในแต่ละรอบ เนื่องจากการทำงานพร้อมกันของหลาย Thread และการจัดสรร CPU ของระบบปฏิบัติการ
- **CSV Format:** ไฟล์ Workload ต้องมีรูปแบบข้อมูลถูกต้องตามที่โปรแกรมกำหนด และต้องระบุ Path ของไฟล์ให้ถูกต้อง
- **Monitor Snapshot:** ค่า Printer และ Database Permits ถูกอ่านแยกจากสถานะ READY, RUNNING และ COMPLETED จึงอาจไม่ใช่ข้อมูล ณ เวลาเดียวกันทั้งหมด

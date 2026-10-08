# 517-312 Operating Systems

# TERM PROJECT - Mini Job Scheduler & Shared Resource Manager

### Group Member
### 1.นางสาวกัญญรัชต์ สมหวังพรเจริญ รหัสนักศึกษา 670710122
### 2.นางสาวอนัญญณัชช์ ขจรศิริผล รหัสนักศึกษา 670710637
### 3.นางสาวจุฑารัตน์ รู้วงษ์ รหัสนักศึกษา 670710643

## 1. Compile 

เปิด Terminal (PowerShell) ใน VS Code แล้วเข้าโฟลเดอร์ `src`

```powershell
cd "D:\Year_3\OS\OS101 - Copy (2)\OS101\src"
```

```powershell
cd src
javac *.java
```

## 2. Run

รูปแบบคำสั่งสำหรับรันโปรแกรม:

```text
java -cp . Main <workload.csv> <fcfs|priority> <workers> <printerPermits> <databasePermits>
```

**ตัวอย่างการรัน งานที่ 1 - 10**

### 01: Standard FCFS
```powershell
java Main jobs_standard.csv fcfs 3 1 2 | Tee-Object ..\logs\01_standard_fcfs.txt
```

### 02: Standard Priority
```powershell
java Main jobs_standard.csv priority 3 1 2 | Tee-Object ..\logs\02_standard_priority.txt
```

### 03: Priority Worker = 1
```powershell
java Main jobs_standard.csv priority 1 1 2 | Tee-Object ..\logs\03_priority_worker1.txt
```

### 04: Priority Worker = 5
```powershell
java Main jobs_standard.csv priority 5 1 2 | Tee-Object ..\logs\04_priority_worker5.txt
```

### 05: Printer Permit = 1
```powershell
java Main jobs_printer.csv fcfs 3 1 2 | Tee-Object ..\logs\05_printer1.txt
```

### 06: Printer Permit = 2
```powershell
java Main jobs_printer.csv fcfs 3 2 2 | Tee-Object ..\logs\06_printer2.txt
```

### 07: Priority Repeat
```powershell
java Main jobs_standard.csv priority 3 1 2 | Tee-Object ..\logs\07_priority_repeat.txt
```

### 08: Database Permit = 2
```powershell
java Main jobs_db.csv fcfs 3 1 2 | Tee-Object ..\logs\08_database.txt
```

### 09: Same Priority
```powershell
java Main jobs_same_priority.csv priority 3 1 2 | Tee-Object ..\logs\09_same_priority.txt
```

### 10: Database Permit = 1
```powershell
java Main jobs_db.csv fcfs 3 1 1 | Tee-Object ..\logs\10_database1.txt
```
### ตรวจสอบจำนวนงานที่ประมวลผลเสร็จสิ้น

```powershell
Get-ChildItem "..\logs\*.txt" |
    Sort-Object Name |
    ForEach-Object {
        Write-Host "`n========== $($_.Name) =========="
        Select-String -Path $_.FullName -Pattern "Completed Jobs"
    }
```
### แสดงผลสรุปการรันทั้งหมด

```powershell
Get-ChildItem "..\logs\*.txt" |
    Sort-Object Name |
    ForEach-Object {
        Write-Host "`n========== $($_.Name) =========="
        Get-Content $_.FullName -Tail 8
    }
```



## 3. Command-Line Arguments (ค่าที่ใส่ตอนรันโปรแกรม)

| Argument | Description |
|---|---|
| `workload.csv` | ตำแหน่งไฟล์ CSV ที่เก็บข้อมูล Job |
| `fcfs\|priority` | เลือกการจัดตารางงานแบบ FCFS หรือ Priority |
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

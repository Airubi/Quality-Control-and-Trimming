# Quality Control and Trimming of Raw Sequencing Reads

แบบฝึกหัดตรวจคุณภาพ sequencing reads ด้วย FastQC และกรองหรือ trim reads ด้วย fastp ครอบคลุม Illumina single-end, Illumina paired-end และ Oxford Nanopore (ONT) simplex

## ลำดับการทำงาน

1. ตรวจ FASTQ ดิบด้วย FastQC เพื่อดูค่าตั้งต้นและปัญหาที่ควรสังเกต
2. ใช้ fastp กับ Illumina paired-end และ ONT ตามคำสั่งของแต่ละแบบฝึกหัด
3. ตรวจไฟล์ output หลัง fastp ด้วย FastQC
4. เปรียบเทียบรายงานก่อนและหลัง โดยดูจำนวน reads, จำนวนเบส, Q20/Q30, ความยาว และสถานะ FastQC

FastQC สร้างรายงานและไม่แก้ไข FASTQ ส่วน fastp สร้างไฟล์ FASTQ output ที่ผ่านการกรอง/trim พร้อมรายงาน HTML และ JSON

## โครงสร้างโฟลเดอร์

~~~text
Quality-Control-and-Trimming/
├── README.md
├── Exercise_1A_Illumina_SE/
│   ├── simpleIlluminaSE.fastq.gz
│   ├── simpleIlluminaSE_fastqc.html
│   └── simpleIlluminaSE_fastqc.zip
├── Exercise_1B_ONT_simplex/
│   ├── ONTsimplex.fastq.gz
│   ├── ONTsimplex_fastqc.html
│   └── ONTsimplex_fastqc.zip
├── Exercise_2_1_Illumina_PE/
│   ├── PE_R1.fastq.gz
│   ├── PE_R2.fastq.gz
│   ├── PEreport.html
│   ├── PEreport.json
│   ├── trimmed_R1.fastq.gz
│   ├── trimmed_R1_fastqc.html
│   ├── trimmed_R1_fastqc.zip
│   ├── trimmed_R2.fastq.gz
│   ├── trimmed_R2_fastqc.html
│   └── trimmed_R2_fastqc.zip
└── Exercise_2_2_ONT_simplex/
    ├── ONTsimplex.fastq.gz
    ├── ont_report.html
    ├── ont_report.json
    ├── trimmed_ont.fastq.gz
    ├── trimmed_ont_fastqc.html
    └── trimmed_ont_fastqc.zip
~~~

ไฟล์ `.fastq.gz` คือ reads ที่บีบอัดด้วย gzip; `trimmed_*` คือ output จาก fastp. รายงาน FastQC มี HTML สำหรับดูกราฟและ ZIP สำหรับดูข้อมูลสรุป ส่วนรายงาน fastp มี HTML และ JSON

## วิธีรันคำสั่ง

เปิด WSL แล้วเข้าโฟลเดอร์ของแบบฝึกหัดก่อน เช่น:

~~~bash
cd ~/Quality-Control-and-Trimming/Exercise_2_2_ONT_simplex
~~~

ต้องมี FastQC และ fastp ติดตั้งใน WSL. ตรวจข้อมูลดิบด้วย FastQC โดยระบุชื่อไฟล์ เช่น:

~~~bash
cd ~/Quality-Control-and-Trimming/Exercise_1A_Illumina_SE
fastqc simpleIlluminaSE.fastq.gz

cd ~/Quality-Control-and-Trimming/Exercise_1B_ONT_simplex
fastqc ONTsimplex.fastq.gz
~~~

คำสั่งเหล่านี้สร้างรายงาน FastQC ของไฟล์ input และไม่เปลี่ยนไฟล์ FASTQ

## แบบฝึกหัดและวิธีจับคู่ไฟล์

### 1A — Illumina single-end: ตรวจข้อมูลดิบ

ใช้ `simpleIlluminaSE.fastq.gz` เป็น input สำหรับ FastQC. รายงานที่ได้คือ `simpleIlluminaSE_fastqc.html` และ `simpleIlluminaSE_fastqc.zip`. จุดประสงค์คือดูคุณภาพเบสและลักษณะข้อมูลก่อน trimming

Basic Statistics: **3,000 sequences**, ความยาว **150 bp**, **39% GC**.

ผล FastQC: Per Base Sequence Quality, Per Sequence Quality Scores, Per Base N Content, Sequence Length Distribution, Sequence Duplication Levels และ Adapter Content = PASS; Per Base Sequence Content และ Per Sequence GC Content = FAIL; Overrepresented Sequences = WARN.

### 1B — ONT simplex: ตรวจข้อมูลดิบ

ใช้ `ONTsimplex.fastq.gz` เป็น input สำหรับ FastQC. รายงานคือ `ONTsimplex_fastqc.html` และ `ONTsimplex_fastqc.zip`. จุดประสงค์คือสำรวจคุณภาพและความยาวที่หลากหลายของ long reads ก่อนกรอง

Basic Statistics: **4,000 sequences**, ความยาว **150–104,329 bp**, **35% GC**.

ผล FastQC: Per Base Sequence Quality และ Per Base Sequence Content = FAIL; Per Sequence Quality Scores, Per Sequence GC Content, Sequence Length Distribution และ Overrepresented Sequences = WARN; Per Base N Content, Sequence Duplication Levels และ Adapter Content = PASS.

### 2.1 — Illumina paired-end: trimming และตรวจหลังกรอง

`PE_R1.fastq.gz` และ `PE_R2.fastq.gz` เป็น reads คู่กันของตัวอย่างเดียวกัน. fastp รับ R1/R2 และสร้าง `trimmed_R1.fastq.gz` กับ `trimmed_R2.fastq.gz`; รายงาน fastp คือ `PEreport.html` และ `PEreport.json`. จากนั้น FastQC ตรวจ trimmed R1/R2 แยกกัน โดยผลอยู่ในไฟล์ `trimmed_R1_fastqc.*` และ `trimmed_R2_fastqc.*`

คำสั่งที่บันทึกใน `PEreport.json`:

~~~bash
fastp -i PE_R1.fastq.gz -I PE_R2.fastq.gz -o trimmed_R1.fastq.gz -O trimmed_R2.fastq.gz -h PEreport.html -j PEreport.json
fastqc trimmed_R1.fastq.gz trimmed_R2.fastq.gz
~~~

- `-i` / `-I`: input ของ R1 / R2
- `-o` / `-O`: output ของ R1 / R2
- `-h`: สร้างรายงาน HTML
- `-j`: สร้างรายงาน JSON

ผล fastp: ก่อนกรอง **20,000 reads**, **1,987,519 bases**; หลังกรอง **19,638 reads**, **1,949,053 bases**. Q20 เปลี่ยนจาก **96.82% เป็น 97.37%** และ Q30 จาก **92.36% เป็น 93.21%**. รายงานระบุ 362 reads ถูกคัดออกเพราะ low quality และไม่มี reads ถูกคัดออกเพราะ N มากเกินไป, adapter dimer, สั้นเกินไป หรือยาวเกินไป

FastQC หลังกรอง (ต่อไฟล์): **9,819 sequences**, ความยาว **32–100 bp**, **44% GC**. ทั้ง R1 และ R2 มี Per Base Sequence Quality, Per Sequence Quality Scores, Per Base N Content, Sequence Duplication Levels และ Adapter Content = PASS; Per Base Sequence Content และ Per Sequence GC Content = FAIL; Sequence Length Distribution และ Overrepresented Sequences = WARN.

### 2.2 — ONT simplex: กรองความยาวขั้นต่ำและตรวจหลังกรอง

ใช้ `ONTsimplex.fastq.gz` เป็น input และสร้าง `trimmed_ont.fastq.gz`; รายงาน fastp คือ `ont_report.html` และ `ont_report.json`. FastQC หลังกรองสร้าง `trimmed_ont_fastqc.html` และ `trimmed_ont_fastqc.zip`. กำหนดความยาวขั้นต่ำเพื่อทดลองผลของการตัด reads สั้นออกจากข้อมูล long-read

คำสั่งที่บันทึกใน `ont_report.json`:

~~~bash
fastp -i ONTsimplex.fastq.gz -o trimmed_ont.fastq.gz --length_required 1000 -h ont_report.html -j ont_report.json
fastqc trimmed_ont.fastq.gz
~~~

- `-i` / `-o`: input / output
- `--length_required 1000`: กำหนดความยาวขั้นต่ำ 1,000 bp
- `-h` / `-j`: สร้างรายงาน HTML / JSON

ผล fastp: ก่อนกรอง **4,000 reads**, **12,250,137 bases**; หลังกรอง **1,930 reads**, **10,765,241 bases** หรือเหลือ **48.25%** ของจำนวน reads. ถูกคัดออก 1,879 reads เพราะสั้นเกินไป และ 191 reads เพราะ low quality; ไม่มี reads ที่มี N มากเกินไปหรือ adapter dimer. Q20 เปลี่ยนจาก **73.95% เป็น 75.37%** และ Q30 จาก **51.31% เป็น 52.48%**.

FastQC หลังกรอง: **1,930 sequences**, ความยาว **1,000–104,329 bp**, **35% GC**. Per Sequence Quality Scores, Per Base N Content, Sequence Duplication Levels และ Adapter Content = PASS; Per Base Sequence Quality และ Per Base Sequence Content = FAIL; Per Sequence GC Content, Sequence Length Distribution และ Overrepresented Sequences = WARN.

## วิธีอ่านผล QC

PASS/WARN/FAIL เป็นสถานะตามเกณฑ์ FastQC ไม่ตัดสินโดยตรงว่าข้อมูลใช้ได้หรือใช้ไม่ได้ ควรเปิดรายงาน HTML ดูกราฟและพิจารณาชนิดข้อมูลประกอบ โดยเฉพาะ ONT ซึ่ง reads มีความยาวหลากหลายและอาจทำให้บางโมดูลขึ้น WARN หรือ FAIL

ในงานนี้ FastQC ZIP ระบุเวอร์ชัน **0.13.0** และรายงาน fastp ระบุเวอร์ชัน **1.4.0**. สำหรับ ONT Exercise 2.2 การกำหนดขั้นต่ำ 1,000 bp ทำให้ reads ที่สั้นกว่าค่าเกณฑ์ถูกคัดออกจำนวนมาก ขณะที่ Q20/Q30 หลังกรองเพิ่มขึ้นเล็กน้อย
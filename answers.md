# Lab L1 · answers.md (TH)

ชื่อ-สกุล: ฮาซูวัน ขวัญ (Hasuwan Khwan)
รหัสนักศึกษา: 67108951
ลิงก์ repo: https://github.com/hasuwan111/lab01_profile

## Q1 · เครื่องมือ
จากการรัน `flutter doctor` ไม่พบปัญหาใดๆ (`No issues found!`) โดยได้ตรวจสอบบรรทัดสำคัญ 2 บรรทัด คือ:
- `[√] Flutter (Channel stable, 3.50.0, on Microsoft Windows ...)` หมายถึงติดตั้ง Flutter SDK และ Dart SDK เรียบร้อยและเรียกใช้งานผ่าน command line ได้
- `[√] Android toolchain - develop for Android devices (Android SDK version 36.0.0)` หมายถึงติดตั้ง Android SDK, command-line tools และยอมรับ Android licenses ครบถ้วนพร้อม build แอป Android

นอกจากนี้ตอนเตรียมรันบน emulator พบปัญหาเพิ่มเติม 2 จุดและได้แก้ไขดังนี้:
1. `flutter emulators` มองไม่เห็น AVD เพราะตัวแปร `ANDROID_AVD_HOME` ชี้ไปที่ `D:\Android\avd` แต่ไฟล์ AVD (`Medium_Phone_API_37.0`) อยู่ที่ `C:\Users\HP\.android\avd` จึงตั้งค่า `ANDROID_AVD_HOME` ให้ตรงตำแหน่งจนเปิด emulator ได้
2. ตอนสั่ง `flutter run` ครั้งแรก Gradle แจ้งว่าไม่พบ NDK `28.2.13676358` จึงใช้เครื่องมือ Android SDK CLI (`android sdk install "ndk;28.2.13676358"`) ตามเอกสารทางการของ Android Developers และ `docs.flutter.dev` ติดตั้ง NDK เพิ่มจน build และรันบน emulator สำเร็จ

## Q2 · Reload
เมื่อ `Likes` เป็น 3 อยู่:
- **กด `r` (Hot Reload)**: ค่า `Likes` **ยังคงเป็น 3 (`Likes: 3`)** (ขณะที่ข้อความที่แก้บน UI เช่น title เปลี่ยนเป็นค่าใหม่ทันที) เพราะ Hot Reload จะส่งโค้ดที่แก้เข้าไปใน Dart VM แล้วเรียก `build()` ใหม่โดย **รักษา `State` object (`_HomePageState`) และค่าตัวแปร `_likes` เดิมในหน่วยความจำไว้**
- **กด `R` (Hot Restart)**: ค่า `Likes` **รีเซ็ตกลับเป็น 0 (`Likes: 0`)** เพราะ Hot Restart จะทำลาย widget tree และ `State` object เดิมทั้งหมด แล้วเริ่มรันแอปใหม่ตั้งแต่ฟังก์ชัน `main()` ทำให้ `_HomePageState` ถูกสร้างใหม่และ `_likes` เริ่มต้นที่ `0` อีกครั้ง

## Q3 · โค้ดและ Git
- **ทำไม `_likes++` ต้องอยู่ใน `setState`**: หากเขียนเพียง `_likes++` ค่าของตัวแปร `_likes` ในหน่วยความจำจะเพิ่มขึ้นจริง แต่ Flutter framework จะไม่ทราบว่า state เปลี่ยนแปลง จึงไม่เรียกเมธอด `build()` เพื่อวาดหน้าจอใหม่ (หน้าจอจะยังแสดงตัวเลขเดิม) การใส่ `_likes++` ไว้ใน `setState(() => _likes++)` เป็นการแจ้ง Flutter ให้ mark widget นี้ว่าต้องอัปเดตและเรียก `build()` ใหม่ในเฟรมถัดไปเพื่อให้ `Text('Likes: $_likes')` แสดงค่าล่าสุด
- **ตัวอย่าง commit**: `Add like counter with setState`
  - เปลี่ยน `HomePage` จาก `StatelessWidget` เป็น `StatefulWidget` และสร้างคลาส `_HomePageState` เพื่อเก็บสถานะ `int _likes = 0;` พร้อมเมธอด `void _like() => setState(() => _likes++);`
  - ลบ `const` หน้า `Center` เนื่องจากมี `Text('Likes: $_likes')` ที่ค่าเปลี่ยนแปลงได้ขณะรันแอป
  - เพิ่ม `Text('Likes: $_likes')` และปุ่ม `FilledButton(onPressed: _like, child: const Text('Like'))` ใน `Column` รวมถึงเปิดรัน widget test ใน `test/home_page_test.dart`

## ตารางทดสอบ

| ประเภท | input หรือการกระทำ | ผลที่คาดหวัง | ผลจริง |
|---|---|---|---|
| Normal | เปิดแอป | title มีรหัส ไอคอน ชื่อ “Likes: 0” | ผ่าน: แสดง AppBar “Lab L1 · 67108951”, ไอคอนคน, “Hasuwan Khwan - 67108951”, “Likes: 0” และปุ่ม Like |
| Normal | กด Like หนึ่งครั้ง | “Likes: 1” | ผ่าน: ข้อความเปลี่ยนเป็น “Likes: 1” ทันที |
| Boundary | กด Like เร็ว ๆ 20 ครั้ง | “Likes: 20” ไม่มี error | ผ่าน: แสดง “Likes: 20” ไม่มี error หรือ exception (`flutter test` ผ่าน) |
| Boundary | หมุนจอเป็นแนวนอน | ยังอยู่กลางจอ ไม่ overflow | ผ่าน: เนื้อหายังอยู่กึ่งกลางจอ ไม่เกิด overflow (`flutter test` ที่ขนาดแนวนอน 800×360 ผ่าน) |
| Reload | กด 3 ครั้ง แล้ว r แล้ว R | หลัง r ได้ 3 · หลัง R ได้ 0 | ผ่าน: หลัง `r` ได้ “Likes: 3” · หลัง `R` รีเซ็ตกลับเป็น “Likes: 0” |
| Failure | ปิด emulator แล้ว `flutter run` | “No supported devices connected.” | ผ่าน: ไม่พบ Android emulator ที่เชื่อมต่ออยู่ (เมื่อไม่มีอุปกรณ์ที่รองรับจะแจ้ง “No supported devices connected.”) |

## การใช้ AI
ใช้ Antigravity (AI assistant) ช่วยตรวจเช็กและแก้ปัญหา environment (`ANDROID_AVD_HOME`, ติดตั้ง Android NDK ผ่าน `android sdk install`), ช่วยแนะนำลำดับการเขียนโค้ดและตรวจคำตอบใน `answers.md` โดยได้ตรวจสอบและเข้าใจการทำงานของ `StatelessWidget`, `StatefulWidget`, `setState`, Hot Reload/Hot Restart และ Git commits ทั้งหมดด้วยตนเอง

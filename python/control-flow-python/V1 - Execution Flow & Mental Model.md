# V1 — Execution Flow & Mental Model

_Beyond The Syntax — Control Flow Series_

---

## Pembukaan

Kamu sudah menulis ratusan baris kode Python. Kamu tahu cara membuat list, manipulasi dictionary, bahkan paham kapan harus pakai tuple ketimbang list biasa. Tapi coba jawab pertanyaan ini dengan jujur:

**Apa yang sebenarnya terjadi di antara saat kamu menekan Run dan saat output muncul di terminal?**

Kalau jawabanmu adalah "Python membaca kode dari atas ke bawah" — itu benar, tapi dangkal. Jawaban itu cukup buat lulus quiz, tapi tidak cukup buat menjelaskan kenapa kode yang _terlihat_ benar kadang menghasilkan error yang aneh, atau kenapa urutan dua baris yang "sama saja secara logika" bisa menghasilkan perilaku berbeda.

Volume ini tidak mengajarkan `if`, `for`, atau `while`. Itu tugas V2 dan V3. Volume ini membangun sesuatu yang lebih mendasar: **mental model tentang apa itu "eksekusi"** — supaya ketika nanti kamu bertemu percabangan dan perulangan, kamu tidak menghafal sintaks baru, tapi mengenali variasi dari sesuatu yang sudah kamu pahami strukturnya.

---

## Bab 1 — Kode Bukan Daftar, Kode Adalah Proses

### Intuisi

Bayangkan kamu menulis resep masakan di atas kertas. Resep itu adalah teks statis — kalimat-kalimat yang diam, tidak berubah, bisa dibaca berkali-kali tanpa terjadi apa-apa di dapur. Tapi begitu seseorang benar-benar _memasak_ mengikuti resep itu, sesuatu yang berbeda terjadi: air mulai mendidih, bawang berubah warna, waktu berjalan, dan dapur — yang tadinya diam — sekarang punya keadaan yang terus berubah.

Source code Python persis seperti resep itu. File `.py` yang kamu tulis adalah teks diam. Tidak ada yang "terjadi" di dalamnya sampai sesuatu benar-benar menjalankannya. Kebanyakan pemula (dan kejutannya, beberapa yang sudah lama coding) memperlakukan kode sebagai daftar instruksi statis yang "sudah ada di sana", padahal yang sebenarnya berharga untuk dipahami adalah **proses yang lahir ketika daftar itu dijalankan**.

Perbedaan ini terasa remeh sampai kamu menghadapi situasi seperti ini:

```python
print(suhu)
suhu = 30
```

Kode di atas terlihat masuk akal kalau kamu membaca teksnya sebagai dua fakta independen. Tapi Python tidak membaca dua fakta — Python _menjalani_ dua momen yang berurutan. Pada momen pertama, nama `suhu` belum pernah diikat ke nilai apa pun. Hasilnya: `NameError`.

Error ini bukan karena Python "bodoh" atau "aneh". Error ini adalah bukti bahwa ada sesuatu yang nyata sedang berjalan — sebuah proses yang punya keadaan (_state_) di setiap titik waktu, dan keadaan itu berbeda di baris pertama dibanding baris kedua.

### Mental Model

Mulai dari sini, ganti caramu membayangkan kode. Jangan bayangkan kode sebagai **teks yang dibaca sekaligus**. Bayangkan sebagai **garis waktu yang dijalani selangkah demi selangkah**, di mana setiap langkah punya kemampuan mengubah apa yang "diketahui" oleh langkah berikutnya.

```text
Waktu →
  [t0]──────[t1]──────[t2]──────[t3]──────→
   |          |          |          |
  state₀     state₁     state₂     state₃
```

Di setiap titik waktu, ada satu _snapshot_ keadaan. Tapi hati-hati menyederhanakan "keadaan" ini menjadi sekadar "daftar variabel" — itu definisi yang terlalu sempit. **State adalah kondisi penuh program pada satu titik waktu**: nama-nama apa saja yang sudah diikat dan nilai apa yang mereka pegang, _tapi juga_ informasi lain yang memengaruhi apa yang bisa dan akan terjadi di langkah berikutnya — modul apa yang sudah dimuat, posisi mana di source code yang sedang dieksekusi, dan (nanti, mulai V2) jalur mana yang sudah dipilih ketika program punya lebih dari satu kemungkinan arah. Variabel hanyalah salah satu komponen state yang paling mudah dilihat, bukan satu-satunya.

Dengan definisi ini, mental model utama volume ini bisa dirumuskan dalam satu kalimat: **execution adalah perubahan state dari waktu ke waktu**. Setiap baris yang dijalankan berpotensi memindahkan program dari satu state ke state berikutnya. Kode yang kamu tulis bukan deskripsi tentang dunia — ia adalah **resep untuk mengubah satu keadaan menjadi keadaan berikutnya, berkali-kali, secara berurutan**.

### Penjelasan Teknis

Pada **level bahasa** (_language level_) — yaitu apa yang dijamin oleh spesifikasi Python, terlepas dari implementasinya — Python menjamin bahwa statement-statement dalam satu blok kode dieksekusi **secara berurutan, sesuai urutan penulisannya**, dari atas ke bawah. Ini bukan detail kecil — ini adalah salah satu kontrak paling fundamental yang dipegang bahasa ini. Tanpa jaminan ini, tidak ada cara untuk bernalar tentang program sama sekali, karena urutan baris tidak akan punya makna apa pun.

Pada **level implementasi**, CPython (implementasi Python yang paling umum dipakai) mewujudkan jaminan ini dengan mengompilasi kode sumbermu menjadi _bytecode_ — instruksi tingkat rendah — lalu menjalankannya satu per satu menggunakan sesuatu yang disebut _evaluation loop_, dengan sebuah penanda (mirip "penunjuk halaman") yang bergerak maju mengikuti instruksi mana yang harus dieksekusi selanjutnya.

Penting: **jangan tukar dua level ini**. Fakta bahwa Python dieksekusi "berurutan" adalah jaminan bahasa, bukan kebetulan implementasi CPython. Implementasi lain seperti PyPy punya mekanisme internal yang berbeda, tapi tetap wajib memenuhi jaminan urutan eksekusi yang sama — karena itu bagian dari kontrak bahasa, bukan pilihan teknis yang bisa diabaikan.

### Eksperimen

```python
print("satu")
print("dua")
print("tiga")
```

Jalankan ini. Hasilnya selalu `satu`, `dua`, `tiga` — tidak pernah berubah urutan, tidak pernah berjalan paralel, tidak pernah "diacak" oleh optimisasi apa pun. Sekarang bandingkan dengan ini:

```python
nilai = 10
nilai = nilai + 5
print(nilai)
```

Coba tebak sebelum menjalankan: apa yang terjadi di baris kedua? Python tidak melihat `nilai = nilai + 5` sebagai persamaan matematika ("nilai sama dengan nilai ditambah 5", yang secara matematis kontradiktif). Python menjalaninya sebagai dua momen berurutan: pertama, ambil nilai `nilai` yang _sudah ada_ di state saat ini (10), tambahkan 5, hasilkan 15 — _baru kemudian_ ikat ulang nama `nilai` ke nilai baru itu. State berubah dari `{nilai: 10}` menjadi `{nilai: 15}`. Output: `15`.

### Miskonsepsi Umum

**"Python membaca seluruh file dulu sebelum menjalankan apa pun, jadi semua nama sudah 'ada' sejak awal."**

Ini sebagian benar dan sebagian menyesatkan. Python memang melakukan tahap kompilasi (mengubah teks menjadi bytecode) sebelum eksekusi — tapi kompilasi hanya memvalidasi struktur sintaksis, bukan menjalankan apa pun. Nama-nama (variabel) tidak "ada" sampai statement yang mengikatnya benar-benar dieksekusi. Inilah sebabnya `print(suhu)` sebelum `suhu = 30` menghasilkan error, bukan nilai kosong atau `None`. Tidak ada "pra-pengetahuan" tentang masa depan program — hanya ada apa yang sudah terjadi di state saat ini.

### Simpulan Mini

Kode bukan daftar fakta yang dibaca sekaligus. Kode adalah resep yang dijalani selangkah demi selangkah, di mana setiap langkah mewarisi keadaan dari langkah sebelumnya dan berpotensi mengubahnya untuk langkah berikutnya. Tapi pertanyaan yang belum kita jawab: **apa sebenarnya yang disebut "langkah" itu?** Tidak semua baris kode setara — beberapa baris "melakukan sesuatu" pada state, sementara yang lain hanya "menghasilkan nilai" tanpa benar-benar mengubah apa pun dengan sendirinya. Perbedaan ini adalah fondasi bab berikutnya.

---

## Bab 2 — Statement: Unit yang Mengubah Dunia

### Intuisi

Bandingkan dua hal ini:

```python
3 + 4
```

```python
suhu = 30
```

Keduanya "valid" secara sintaks. Tapi rasakan bedanya: yang pertama terasa seperti **bertanya** ("berapa hasil dari 3 ditambah 4?"), sementara yang kedua terasa seperti **memerintah** ("ikat nama `suhu` ke nilai 30, sekarang"). Yang pertama menghasilkan sesuatu yang langsung "hilang" kalau tidak ditangkap — kamu tidak akan melihat efek apa pun di state program. Yang kedua benar-benar mengubah apa yang "diketahui" program setelahnya.

Ini bukan perbedaan gaya penulisan. Ini perbedaan kategori. Python membedakan antara unit kode yang **menghasilkan nilai** dan unit kode yang **melakukan aksi terhadap state**. Kategori kedua inilah yang disebut **statement**.

### Mental Model

Bayangkan program sebagai serangkaian _perintah kerja_ yang dijalankan oleh seorang pekerja (interpreter). Setiap statement adalah satu instruksi yang merepresentasikan sebuah aksi atau struktur instruksi — sesuatu yang dijalankan, bukan sesuatu yang sekadar dijawab. Statement bukan "menjawab pertanyaan", statement _melakukan_ sesuatu: mengikat nama, mengimpor modul, atau (nanti, di V2 dan V3) membentuk struktur instruksi yang lebih besar yang mengubah arah alur program.

```text
Statement = unit sintaksis yang merepresentasikan
            aksi atau struktur instruksi yang dapat dieksekusi
```

### Penjelasan Teknis

Pada level bahasa, **statement** adalah unit sintaksis yang merepresentasikan sebuah aksi atau struktur instruksi yang dapat dieksekusi — sesuatu yang dijalankan oleh Python untuk menghasilkan efek terhadap state, bukan sekadar menghasilkan nilai. Definisi ini sengaja dirumuskan luas, karena Python sendiri membedakan dua bentuk statement yang berbeda struktur, meski sama-sama "instruksi yang dieksekusi":

- **Simple statement** — instruksi yang berdiri sendiri dalam satu baris logis, tanpa membungkus instruksi lain di dalamnya. Inilah satu-satunya bentuk yang dibahas di volume ini.
- **Compound statement** — instruksi yang membungkus satu atau lebih statement lain di dalam strukturnya (misalnya struktur percabangan atau perulangan). Bentuk ini sengaja **tidak** dibahas di sini — ia adalah materi utama V2 dan V3, karena compound statement justru mendefinisikan _cara baru_ program menentukan statement mana yang akan dieksekusi selanjutnya, bukan sekadar urutan lurus dari atas ke bawah.

Volume ini berfokus pada simple statement. Beberapa contoh yang relevan:

- **Assignment statement** — `suhu = 30`, mengikat nama ke nilai.
- **Expression statement** — sebuah expression yang berdiri sendiri sebagai satu baris, seperti `print("halo")`. Nilai yang dihasilkan expression-nya (kalau ada) dibuang, kecuali ada efek samping (seperti `print` yang menulis ke layar).
- **Import statement** — `import math`, memuat modul ke dalam state program.
- **Pass statement** — `pass`, sengaja tidak melakukan apa-apa, tapi tetap valid sebagai instruksi yang "kosong".

Yang menyatukan semua simple statement ini: masing-masing, ketika dieksekusi, dianggap Python sebagai satu langkah yang selesai sebelum melanjutkan ke langkah berikutnya. Statement adalah unit granular dari "garis waktu" yang kita bahas di Bab 1 — dan definisi ini tetap berlaku konsisten nanti ketika compound statement diperkenalkan, karena compound statement pun, pada akhirnya, tetap "diselesaikan" sebagai satu langkah sebelum eksekusi berpindah ke langkah berikutnya — hanya saja jalan di dalamnya tidak selalu lurus.

### Eksperimen

```python
suhu = 30          # statement: mengikat nama
print(suhu)        # statement: memanggil fungsi (efek samping: mencetak)
suhu + 5            # statement: expression berdiri sendiri, hasilnya dibuang
print(suhu)        # masih 30 — baris sebelumnya tidak mengubah apa pun
```

Baris ketiga (`suhu + 5`) adalah jebakan klasik bagi pemula yang berpikir matematis: nilai 35 _dihitung_, tapi tidak pernah disimpan ke mana pun. Tidak ada nama yang menerimanya, tidak ada efek samping yang menampilkannya. Nilai itu dihitung lalu dibuang — sebuah jawaban yang tidak pernah ditulis di mana pun. Statement itu valid, tapi tidak mengubah state.

### Miskonsepsi Umum

**"Setiap baris kode adalah 'function call' atau 'command' — semuanya setara."**

Tidak. Ada baris yang murni menghasilkan nilai tanpa efek (expression biasa), ada baris yang mengikat nama (assignment), dan ada baris yang memanggil sesuatu yang _mungkin_ punya efek samping selain menghasilkan nilai (seperti `print()`, yang menghasilkan `None` tapi efek sampingnya adalah menulis ke layar). Mencampur ketiganya tanpa membedakannya membuat sulit menjelaskan kasus seperti baris ketiga di eksperimen di atas — kenapa `suhu + 5` "tidak melakukan apa-apa" padahal terlihat seperti perhitungan yang sah.

### Simpulan Mini

Statement adalah unit kerja lengkap yang menggerakkan garis waktu program selangkah demi selangkah. Tapi kita baru menyebut sepintas bahwa beberapa statement "menghasilkan nilai" sebelum melakukan aksinya — seperti assignment yang harus tahu _nilai apa_ yang akan diikat sebelum benar-benar mengikatnya. Dari mana nilai itu berasal? Itulah yang akan kita bedah berikutnya: **expression**, dan proses yang menghasilkan nilai darinya — **evaluation**.

---

## Bab 3 — Expression & Evaluation: Bagaimana Nilai Lahir

### Intuisi

Perhatikan baris ini:

```python
hasil = (3 + 4) * 2
```

Sebelum Python bisa mengikat apa pun ke nama `hasil`, ia harus tahu dulu: _nilai apa_ yang akan diikat? Untuk menjawab itu, Python harus "menjalani" bagian kanan dari tanda `=` — menghitungnya sampai tuntas menjadi satu nilai tunggal. Proses menghitung-sampai-tuntas inilah yang disebut **evaluation**, dan setiap potongan kode yang _bisa_ dievaluasi menjadi nilai disebut **expression**.

Bedanya dengan statement di Bab 2: statement adalah "apa yang dilakukan", expression adalah "apa yang dihasilkan". `(3 + 4) * 2` bukan perintah — ia adalah pertanyaan yang punya jawaban tunggal, dan jawaban itu adalah nilai `14`.

### Mental Model

Bayangkan expression sebagai sebuah **mesin penghasil nilai** yang bisa disarangkan satu di dalam yang lain — seperti boneka Rusia. Python harus membongkar sarang itu dari bagian paling dalam, menghasilkan nilai di setiap lapisan, sampai akhirnya tersisa satu nilai tunggal di lapisan paling luar.

```text
(3 + 4) * 2
   |
   ↓ evaluasi (3 + 4) dulu
   7  * 2
   |
   ↓ evaluasi 7 * 2
   14
```

Setiap anak panah di atas adalah satu langkah evaluasi — Python tidak "melihat" `(3 + 4) * 2` sebagai satu kesatuan ajaib yang langsung jadi 14. Ia menjalani proses bertahap, sama seperti garis waktu yang kita bahas dari Bab 1, hanya saja terjadi dalam skala yang lebih kecil — di dalam satu statement, bukan di antara banyak statement.

### Penjelasan Teknis

Pada level bahasa, **expression** adalah konstruksi sintaksis apa pun yang dapat dievaluasi untuk menghasilkan sebuah nilai. Ini mencakup literal (`42`, `"halo"`), nama yang sudah diikat (`suhu`), operasi (`3 + 4`), pemanggilan fungsi (`len(daftar)`), dan kombinasi dari semuanya secara bersarang.

**Evaluation** adalah proses Python menjalankan expression tersebut hingga tuntas menjadi satu nilai, dan proses ini tidak berjalan sembarangan — ia mengikuti aturan yang pasti:

- **Operator precedence (presedensi operator)** — menentukan operator mana yang dievaluasi lebih dulu ketika beberapa operator muncul tanpa tanda kurung. Pada `3 + 4 * 2`, perkalian punya presedensi lebih tinggi dari penjumlahan, sehingga dievaluasi lebih dulu menjadi `3 + 8`, baru kemudian penjumlahan menghasilkan `11` — bukan `(3 + 4) * 2 = 14`. Tanda kurung, seperti yang kita lihat di awal bab ini, memaksa urutan evaluasi tertentu di luar presedensi default.
- **Associativity (asosiativitas)** — menentukan urutan evaluasi ketika beberapa operator dengan presedensi yang _sama_ muncul berurutan. Sebagian besar operator aritmetika di Python bersifat _left-associative_: `10 - 3 - 2` dievaluasi dari kiri ke kanan menjadi `(10 - 3) - 2 = 5`, bukan `10 - (3 - 2) = 9`.
- **Operand evaluation order** — ketika sebuah operasi punya lebih dari satu operand (seperti `a() + b()`), Python mengevaluasi operand di sisi kiri terlebih dahulu, baru sisi kanan, sebelum operasi di antara keduanya benar-benar dijalankan.

Tiga aturan ini bekerja bersama setiap kali Python membongkar expression bersarang: presedensi dan asosiativitas menentukan _bagian mana_ yang dievaluasi lebih dulu, sementara operand evaluation order menentukan _arah_ pembongkarannya. Catatan untuk volume ini: ada satu kategori aturan evaluasi lain — bagaimana Python mengevaluasi operasi logis seperti `and`/`or` secara _short-circuit_ — yang sengaja tidak dibahas di sini, karena ia baru bermakna penuh setelah kamu memahami boolean logic dan condition di **V2 — Decision Making**.

Hubungan antara statement dan expression sekarang bisa dirangkai: **assignment statement bekerja dalam dua fase yang berurutan** — pertama, evaluasi penuh terhadap expression di sisi kanan tanda `=` hingga menghasilkan satu nilai; _baru setelah itu_, nama di sisi kiri diikat ke nilai hasil evaluasi tersebut. Dua fase ini tidak bisa dibalik, dan urutan inilah yang menjelaskan kasus `nilai = nilai + 5` di Bab 1: sisi kanan dievaluasi dulu menggunakan nilai `nilai` yang lama, _sebelum_ nama `nilai` diikat ulang ke hasil yang baru.

### Eksperimen

```python
x = 5
y = x * 2 + 1
print(y)        # 11

x = 100          # mengubah x SETELAH y sudah dihitung
print(y)         # tetap 11, bukan 201
```

Ini sering membingungkan pemula yang mengharapkan `y` "terhubung" secara hidup dengan `x`, seperti rumus di spreadsheet. Tapi `y = x * 2 + 1` bukan rumus yang terus dievaluasi ulang — ia adalah **satu evaluasi sesaat** yang terjadi tepat di momen statement itu dijalankan, menghasilkan nilai `11`, lalu mengikatnya ke `y` secara permanen sampai ada statement lain yang mengikat ulang `y`. Mengubah `x` setelahnya tidak "menarik" perubahan ke `y`, karena evaluasi itu sudah selesai dan hasilnya sudah dibekukan menjadi nilai konkret.

### Miskonsepsi Umum

**"Tanda `=` di Python sama seperti tanda sama-dengan di matematika."**

Ini salah satu sumber kebingungan paling umum, dan sengaja kita tunda jawabannya sampai sekarang karena butuh konsep evaluation untuk dijelaskan dengan benar. Di matematika, `a = b` menyatakan **kesetaraan permanen** dua sisi — keduanya saling menjelaskan satu sama lain selamanya. Di Python, `=` adalah **operasi assignment satu arah dan sesaat**: evaluasi sisi kanan terjadi sekali, hasilnya diikat ke nama di sisi kiri, dan setelah itu hubungan tersebut tidak dipertahankan secara otomatis. Inilah yang membuat `nilai = nilai + 5` masuk akal di Python tapi mustahil sebagai persamaan matematika — keduanya hanya terlihat sama secara visual, bukan secara makna.

### Simpulan Mini

Expression adalah mesin penghasil nilai, dan evaluation adalah proses menjalankannya hingga tuntas. Assignment membutuhkan hasil evaluasi ini sebelum bisa mengikat nama — itulah jembatan antara "menghasilkan nilai" dan "mengubah state" yang sudah kita singgung sejak Bab 1. Tapi ada satu detail yang belum kita bedah dalam-dalam: **urutan yang tepat** antara evaluasi dan pengikatan. Kenapa urutan itu penting, dan apa konsekuensinya kalau dibalik secara mental? Itu yang akan kita selesaikan di bab berikutnya.

---

## Bab 4 — Assignment: Titik Pertemuan Nilai dan Keadaan

### Intuisi

Ini adalah soal klasik yang sering dipakai untuk menguji pemahaman: tukar nilai dua variabel.

```python
a = 1
b = 2

a, b = b, a

print(a, b)   # 2 1
```

Tanpa memahami urutan evaluasi, baris `a, b = b, a` terlihat seperti sihir. Bagaimana mungkin `a` berubah jadi `b` dan `b` berubah jadi `a` _dalam satu baris_, padahal kalau ditulis manual biasanya kita butuh variabel sementara (`temp`) untuk menampung salah satu nilai supaya tidak hilang?

Jawabannya ada di urutan fase yang sudah kita bangun di Bab 3, hanya diterapkan pada assignment yang lebih kompleks: **seluruh sisi kanan dievaluasi penuh menjadi pasangan nilai `(2, 1)` terlebih dahulu — menggunakan nilai `a` dan `b` yang lama — sebelum satu pun nama di sisi kiri diikat ulang.** Tidak ada momen di mana `a` "sudah berubah" sementara `b` "belum dibaca". Evaluasi selesai total dulu, baru pengikatan terjadi.

### Mental Model

Bayangkan assignment sebagai dua fase yang benar-benar terpisah waktu, tidak pernah tumpang tindih:

```text
FASE 1: EVALUASI               FASE 2: PENGIKATAN
(baca state LAMA,              (tulis ke state BARU,
 hasilkan nilai)                 pakai nilai dari Fase 1)

   b, a                              a ← 2
    |                                b ← 1
    ↓
  (2, 1)  ──────────────────────────→
```

Fase 1 tidak peduli apa yang akan terjadi di Fase 2. Ia hanya membaca apa yang _sudah ada_ di state saat ini dan menghasilkan nilai. Fase 2 tidak bisa dimulai sebelum Fase 1 benar-benar tuntas untuk _seluruh_ sisi kanan — bukan satu per satu secara tercampur.

### Penjelasan Teknis

Pada level bahasa, Python menjamin bahwa untuk setiap assignment statement, **evaluasi sisi kanan diselesaikan secara penuh sebelum binding (pengikatan nama) dimulai** — termasuk pada assignment ganda seperti `a, b = b, a`, di mana sisi kanan dievaluasi menjadi satu tuple nilai `(2, 1)` terlebih dahulu, baru kemudian setiap elemen tuple itu diikat ke nama yang sesuai di sisi kiri, secara berurutan.

Penting dibedakan secara hati-hati: **assignment mengikat nama ke nilai, bukan menyalin isi sebuah "kotak" ke kotak lain.** Pemahaman tentang apa sebenarnya yang "diikat" oleh sebuah nama — terutama untuk objek yang lebih kompleks seperti list atau dictionary — sudah dibahas tuntas di seri _Struktur Data Python_, terutama V4 (_Behavioral Semantics_) dan V5 (_Hashability & Identity_). Volume ini sengaja tidak mengulang itu; fokus kita di sini murni pada **urutan waktu** dari proses assignment, bukan pada apa yang terjadi pada level memori objeknya.

### Eksperimen

```python
x = 10

x = x + x

print(x)   # 20, bukan error, bukan ambigu
```

Coba telusuri dengan mental model dua fase: Fase 1 membaca `x` (masih 10) dua kali, menghasilkan `10 + 10 = 20`. Fase 2 baru mengikat nama `x` ke hasil itu. Tidak ada momen aneh di mana `x` "setengah berubah". Sekarang bandingkan dengan kasus yang sedikit lebih jauh:

```python
a, b, c = 1, 2, 3
a, b, c = c, a, b

print(a, b, c)   # 3 1 2
```

Fase 1 menghasilkan tuple `(3, 1, 2)` dari nilai lama. Fase 2 mengikat `a, b, c` secara berurutan ke nilai-nilai itu. Tidak peduli seberapa rumit sisi kiri-kanannya, pola dua fase ini tidak pernah berubah.

### Miskonsepsi Umum

**"`a, b = b, a` bekerja karena Python 'pintar' mendeteksi maksud penukaran."**

Tidak ada deteksi maksud apa pun. Python tidak tahu kamu sedang "menukar" dua variabel — Python hanya menjalankan dua fase yang sudah kita bahas, secara mekanis dan konsisten, tanpa peduli konteks atau niat. Yang membuat penukaran "berhasil" bukan kecerdasan Python membaca maksudmu, melainkan konsekuensi murni dari urutan evaluasi-lalu-binding yang berlaku untuk _semua_ assignment, bukan hanya kasus penukaran.

### Simpulan Mini

Assignment adalah titik pertemuan antara dua hal yang sudah kita bangun: expression yang menghasilkan nilai (Bab 3), dan statement yang mengubah state (Bab 2). Urutannya kaku dan tidak bisa dibalik: evaluasi penuh, baru binding. Sekarang kita punya semua komponen kecil — statement, expression, evaluation, binding. Yang tersisa adalah merangkainya menjadi gambaran besar: bagaimana semua komponen ini, dijalankan berulang kali dari atas ke bawah, membentuk apa yang sebenarnya kita sebut "menjalankan program".

---

## Bab 5 — Garis Waktu Eksekusi: Dari Source Code ke Proses Berjalan

### Intuisi

Sekarang kumpulkan semua yang sudah kita bangun, dan terapkan pada program yang sedikit lebih panjang:

```python
suhu_awal = 20
kenaikan = 5
suhu_akhir = suhu_awal + kenaikan
print(suhu_akhir)
suhu_akhir = suhu_akhir + 10
print(suhu_akhir)
```

Pertanyaannya bukan lagi "apa output-nya" — itu mudah ditebak (25, lalu 35). Pertanyaan yang lebih dalam: **bisakah kamu menceritakan, baris demi baris, bagaimana state program berubah?** Kalau kamu bisa melakukan itu tanpa menjalankan kodenya di kepala secara "ajaib", berarti mental model dari empat bab sebelumnya sudah benar-benar terinternalisasi.

### Mental Model

Inilah diagram yang merangkum seluruh volume ini — alur penuh dari teks statis menjadi proses yang hidup:

```text
Source Code
     |
     ↓
Statement          (unit kerja yang akan dieksekusi, satu per satu, berurutan)
     |
     ↓
Expression Evaluation   (jika statement butuh nilai, evaluasi dulu sampai tuntas)
     |
     ↓
Value              (nilai tunggal hasil evaluasi)
     |
     ↓
State Change       (binding/efek samping mengubah keadaan program)
     |
     ↓
Next Execution Step    (lanjut ke statement berikutnya, mewarisi state baru)
```

Diagram ini bukan sekadar urutan abstrak — ia adalah siklus yang berulang untuk **setiap** statement dalam programmu, dari baris pertama sampai baris terakhir. Setiap kali siklus ini selesai untuk satu statement, ia langsung dimulai lagi untuk statement berikutnya, membawa state yang sudah diperbarui sebagai titik awal yang baru.

### Penjelasan Teknis

Mari kita bedakan dua level pemahaman secara eksplisit, karena keduanya sering tercampur:

**Level bahasa**: Python menjamin bahwa statement-statement pada satu blok kode dieksekusi sesuai urutan tekstualnya, dari atas ke bawah, dan setiap statement melihat state hasil dari semua statement sebelumnya yang sudah dieksekusi. Ini adalah aturan yang harus dipegang oleh _setiap_ implementasi Python yang valid, bukan kebetulan satu mesin tertentu.

**Level implementasi (CPython)**: source code dikompilasi menjadi bytecode, kemudian sebuah _evaluation loop_ mengeksekusi instruksi bytecode tersebut satu per satu menggunakan instruction pointer yang bergerak maju. Detail ini menjelaskan _bagaimana_ jaminan level bahasa diwujudkan secara teknis — tapi jaminan urutannya sendiri bukan milik CPython; itu milik bahasa Python.

Kenapa pembedaan ini penting di luar sekadar trivia? Karena kalau kamu mencampur keduanya, kamu berisiko menganggap detail implementasi CPython (misalnya bagaimana bytecode bekerja) sebagai _aturan bahasa_ yang harus selalu berlaku — padahal implementasi lain seperti PyPy boleh berbeda secara internal, selama jaminan level bahasa tetap terpenuhi.

### Eksperimen — Menelusuri State Secara Eksplisit

Mari telusuri program di awal bab ini sebagai tabel state, langkah demi langkah:

```text
Statement                          | State setelah statement
------------------------------------|----------------------------
suhu_awal = 20                      | {suhu_awal: 20}
kenaikan = 5                        | {suhu_awal: 20, kenaikan: 5}
suhu_akhir = suhu_awal + kenaikan    | {..., suhu_akhir: 25}
print(suhu_akhir)                   | (state tidak berubah, efek: cetak 25)
suhu_akhir = suhu_akhir + 10        | {..., suhu_akhir: 35}
print(suhu_akhir)                   | (state tidak berubah, efek: cetak 35)
```

Latihan seperti ini — menelusuri state secara manual, baris demi baris — adalah cara paling jujur untuk menguji apakah mental modelmu benar. Kalau kamu bisa membuat tabel seperti ini untuk program apa pun (tanpa branching atau loop, untuk sekarang), kamu sudah punya fondasi yang cukup untuk Volume II.

### Miskonsepsi Umum

**"Python 'tahu' seluruh program sejak awal sebelum menjalankannya, jadi urutan baris tidak terlalu penting selama semua nama akhirnya terdefinisi."**

Ini salah, dan berbahaya kalau dibawa ke kode yang lebih kompleks. Kompilasi memang memeriksa struktur sintaksis program secara keseluruhan, tapi **eksekusi tetap murni sekuensial** — tidak ada "pengetahuan masa depan" yang dipakai saat menjalankan satu statement. Setiap statement hanya punya akses ke state yang sudah dibangun oleh statement-statement _sebelumnya_, tidak peduli apakah ada statement "yang akan mendefinisikan sesuatu" di baris selanjutnya. Inilah sebabnya urutan penulisan kode bukan estetika — ia adalah bagian dari maknanya.

### Simpulan Mini

Kita sekarang punya mental model penuh: source code adalah teks diam yang menjadi proses hidup melalui siklus statement → evaluation → value → state change → langkah berikutnya, berulang dari atas ke bawah. Tapi sejauh ini, seluruh siklus ini berjalan dalam satu jalur lurus — tidak pernah bercabang, tidak pernah berulang. Pertanyaan yang mulai muncul secara natural: **bagaimana kalau program harus memilih salah satu dari beberapa jalur yang mungkin?** Bagaimana Python memutuskan jalur mana yang akan dijalani ketika ada lebih dari satu kemungkinan?

Itu bukan pertanyaan yang bisa dijawab volume ini — dan memang sengaja begitu. Itu adalah pertanyaan besar yang menunggu di **V2 — Decision Making**.

---

## Penutup Volume

Volume ini tidak mengajarkan satu pun keyword baru. Tidak ada `if`, tidak ada `for`. Yang dibangun di sini adalah sesuatu yang lebih tahan lama: cara melihat program bukan sebagai teks, tapi sebagai proses yang hidup di sepanjang garis waktu — di mana setiap statement adalah satu langkah yang mewarisi dan mengubah state, dan setiap nilai lahir dari evaluasi yang tuntas sebelum dipakai untuk mengubah apa pun.

Mental model ini bukan trivia yang bisa dilupakan setelah dipahami sekali. Ia adalah lensa yang akan kamu pakai terus-menerus mulai dari volume berikutnya — karena percabangan dan perulangan, pada akhirnya, hanyalah cara-cara baru untuk **memutuskan langkah mana di garis waktu yang akan dijalani selanjutnya**. Bukan konsep yang sama sekali baru — melainkan variasi dari fondasi yang baru saja kamu bangun.

**Yang sengaja belum dibahas di volume ini** (dan akan dibahas di volume berikutnya): bagaimana program memilih di antara beberapa jalur eksekusi yang mungkin (V2), dan bagaimana satu bagian program bisa dijalani berulang kali tanpa menulis ulang kodenya (V3). Batas ini bukan kelalaian — ia sengaja dijaga agar setiap volume punya satu pertanyaan besar yang benar-benar tuntas dijawab, bukan disinggung sekilas dan ditinggalkan setengah jadi.

---

_Beyond The Syntax — Control Flow Series, Volume I_
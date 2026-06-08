# Beyond The Syntax — Python Data Structures
## Volume 6 — Internal Mechanics
### The Machinery Beneath Abstractions

---

> *Volume ini adalah bagian keenam dari seri Struktur Data Python.*
> *Volume 4 memperlihatkan bahwa objek Python adalah entitas hidup —*
> *mereka bermutasi, berbagi state, dan menyebarkan perubahan melalui referensi.*
> *Volume 5 mengungkap bagaimana Python mengenali dan menemukan objek melalui identitas dan hashing.*
> *Tapi satu pertanyaan besar masih menggantung di udara:*
> *mengapa semua itu bekerja seperti yang kita lihat?*
> 
> *Mengapa `append()` terasa instan, tapi `insert(0, x)` terasa lambat?*
> *Mengapa dict bisa menemukan data di antara satu juta entri hampir seketika?*
> *Mengapa collision tidak menghancurkan segalanya?*
> 
> *Jawaban dari pertanyaan-pertanyaan ini tidak tersembunyi di dalam kode yang kamu tulis.*
> *Ia tersembunyi di dalam arsitektur yang Python bangun di balik setiap `[`, `{`, dan `(` yang kamu ketikkan.*
> *Volume ini membuka arsitektur itu, satu lapisan demi satu lapisan.*

---

## Daftar Isi

### Part I — The Memory Landscape

1. [[#1. Objects Live Somewhere|Objects Live Somewhere]]
2. [[#2. References as Connections|References as Connections]]
3. [[#3. Variables as Namespace Entries|Variables as Namespace Entries]]

### Part II — Dynamic Arrays (The Truth Behind Lists)

4. [[#4. Why List Append Is Fast|Why List Append Is Fast]]
5. [[#5. Capacity vs Size|Capacity vs Size]]
6. [[#6. Dynamic Resizing|Dynamic Resizing]]
7. [[#7. Why Insertions Can Be Expensive|Why Insertions Can Be Expensive]]

### Part III — Hash Tables (The Engine Behind Dict & Set)

8. [[#8. Internal Layout of a Hash Table|Internal Layout of a Hash Table]]
9. [[#9. Buckets and Lookup|Buckets and Lookup]]
10. [[#10. Collision Resolution|Collision Resolution]]
11. [[#11. Resizing and Rehashing|Resizing and Rehashing]]

### Part IV — Performance as Emergent Behavior

12. [[#12. Understanding Big O Through Reality|Understanding Big O Through Reality]]
13. [[#13. Why List, Dict, and Set Behave Differently|Why List, Dict, and Set Behave Differently]]
14. [[#14. Hidden Costs|Hidden Costs]]

### Part V — Design Trade-Offs

15. [[#15. Memory vs Speed|Memory vs Speed]]
16. [[#16. Mutability vs Stability|Mutability vs Stability]]
17. [[#17. Why Python Chose These Designs|Why Python Chose These Designs]]

### Part VI — Integration Layer

18. [[#18. Connecting Volume 4, 5, and 6|Connecting Volume 4, 5, and 6]]
19. [[#19. Unified Mental Model|Unified Mental Model]]
20. [[#20. Common Misconceptions Revisited|Common Misconceptions Revisited]]
21. [[#21. Transition to the Next Volume|Transition to the Next Volume]]

---

# Part I — The Memory Landscape

> *Tujuan Part I: Membangun fondasi konkret tentang bagaimana objek Python hidup di memori.*
> *Kita sudah memahami variabel sebagai label dan referensi sebagai koneksi dari sudut pandang perilaku.*
> *Di sini, kita melihatnya dari sudut pandang fisik — bukan "apa yang terjadi," melainkan "di mana ia terjadi."*
> *Part ini adalah pondasi mutlak untuk memahami mengapa dynamic array dan hash table bekerja seperti yang mereka lakukan.*

---

## 1. Objects Live Somewhere

### 1.1 RAM sebagai Panggung

Ketika program Python kamu berjalan, ada satu tempat di mana semua data sungguh-sungguh *berada*: **RAM** (Random Access Memory). Bukan di hard disk, bukan di cloud, bukan di abstraksi yang melayang-layang. RAM.

Bayangkan RAM sebagai sebuah jalan besar yang sangat panjang, terdiri dari miliaran "sel memori" yang bernomor berurutan. Setiap sel memori sangat kecil — biasanya hanya satu byte. Nomor urut sel itu disebut **alamat memori** (*memory address*). Dan setiap objek Python yang kamu buat akan menempati sekumpulan sel memori yang bersebelahan di jalan itu.

```
RAM (representasi konseptual):

Alamat: 1000  1001  1002  1003  1004  1005  1006  1007  ...
        ┌────┬────┬────┬────┬────┬────┬────┬────┐
        │    │    │    │    │    │    │    │    │  ...
        └────┴────┴────┴────┴────┴────┴────┴────┘
```

Ketika Python menciptakan sebuah objek `list`, ia tidak sekadar "menyimpan data" secara abstrak. Python meminta sistem operasi untuk menyisihkan sejumlah sel memori bersebelahan di jalan itu, lalu menuliskan struktur data listnya di sana. Setiap objek punya alamat awal — nomor sel memori pertama yang ia tempati.

### 1.2 Anatomi Sebuah Objek Python

Di balik setiap objek Python — sekecil apapun, bahkan angka `1` — terdapat sebuah struktur tersembunyi yang disebut **header objek**. Header ini adalah informasi administrasi yang Python butuhkan untuk mengelola objek tersebut.

Secara konseptual, setiap objek Python membawa dua hal dalam headernya:

Yang pertama adalah **penghitung referensi** (*reference count*): sebuah angka yang melacak berapa banyak variabel saat ini menunjuk ke objek ini. Ketika tidak ada lagi variabel yang menunjuk ke objek, Python tahu objek itu sudah tidak diperlukan dan bisa dihapus dari memori.

Yang kedua adalah **penunjuk tipe** (*type pointer*): informasi tentang tipe objek ini (`list`, `dict`, `int`, dll.). Inilah yang membuat Python tahu method apa saja yang tersedia dan bagaimana menginterpretasikan data di dalamnya.

```
Struktur sebuah objek Python di memori (konseptual):

┌─────────────────────────────────────┐
│           HEADER OBJEK              │
│  reference_count  │   type_pointer  │
├─────────────────────────────────────┤
│           DATA OBJEK                │
│  (nilai/isi yang spesifik per tipe) │
└─────────────────────────────────────┘
```

Ini berarti bahkan `1 + 1` di Python bukan operasi yang "ringan" seperti di C. Python harus mengurus struktur objek penuh untuk setiap nilai. Tapi trade-off ini adalah yang membuat Python fleksibel, aman dari kebocoran memori, dan dinamis secara tipe.

### 1.3 Setiap Objek Punya Alamat

Karena objek menempati lokasi fisik di RAM, setiap objek bisa diidentifikasi oleh alamat awalnya. Python menyediakan fungsi `id()` yang mengembalikan alamat ini (setidaknya secara konseptual — di CPython, `id()` memang mengembalikan alamat memori fisik).

```python
x = [1, 2, 3]
print(id(x))      # misalnya: 140734012345600
# Ini adalah "nomor sel memori pertama" yang ditempati list ini di RAM

y = "Halo"
print(id(y))      # misalnya: 140734012899712
# String ini menempati lokasi berbeda
```

Dua objek yang berbeda tidak pernah menempati lokasi memori yang sama pada saat yang bersamaan. Alamat memori adalah identitas fisik yang absolut dan tidak bisa dipalsukan.

> **Behavior Note:** `id()` di CPython mengembalikan alamat memori objek. Ini berarti setelah sebuah objek dihapus (garbage collected), objek baru bisa mendapat alamat yang sama. Jangan mengandalkan `id()` untuk identitas jangka panjang — gunakan `is` untuk pengecekan identitas saat ini.

> **Mini Conclusion:** Objek Python bukan abstraksi yang melayang. Mereka menempati ruang fisik yang nyata di RAM dengan alamat yang konkret. Setiap objek membawa header administrasi (reference count dan type pointer) di samping datanya sendiri. Pemahaman ini adalah pondasi untuk memahami mengapa ukuran dan layout objek memengaruhi performa.

---

## 2. References as Connections

### 2.1 Apa Sebenarnya Sebuah Referensi

Ketika Volume 4 dan 5 mengatakan "variabel adalah label" dan "variabel menyimpan referensi," itu adalah penjelasan yang benar secara perilaku. Tapi di Volume 6, kita perlu lebih konkret: apa *sebenarnya* yang disimpan oleh sebuah variabel?

Jawabannya sederhana namun powerful: **variabel menyimpan sebuah alamat memori**.

Ketika kamu menulis `a = [1, 2, 3]`, Python:

1. Menciptakan objek list di suatu lokasi di RAM, misalnya alamat `140000`.
2. Menyimpan angka `140000` di dalam variabel `a`.

Variabel `a` tidak menyimpan `1, 2, 3`. Ia hanya menyimpan angka `140000` — sebuah "peta jalan" yang menunjuk ke tempat di mana list itu sungguh-sungguh berada.

```
Variabel 'a':
┌──────────┐
│ 140000   │  ← ini yang sebenarnya disimpan variabel
└────┬─────┘
     │
     ▼ (mengikuti alamat)
┌────────────────────────────────┐
│ Objek List di alamat 140000    │
│  refcount=1 | type=list        │
│  data: [1, 2, 3]               │
└────────────────────────────────┘
```

Inilah mengapa referensi juga disebut *pointer* dalam istilah sistem: ia "menunjuk" ke lokasi memori, bukan menyalin isinya.

### 2.2 Mengapa Ini Membuat Aliasing Tak Terhindarkan

Dari perspektif fisik, aliasing adalah konsekuensi yang sangat natural. Ketika kita menulis `b = a`, Python hanya menyalin angka alamat dari `a` ke `b`. Tidak ada data yang digandakan.

```
Setelah b = a:

Variabel 'a':          Variabel 'b':
┌──────────┐           ┌──────────┐
│ 140000   │           │ 140000   │
└────┬─────┘           └────┬─────┘
     │                      │
     └──────────┬───────────┘
                ▼
┌────────────────────────────────┐
│ Objek List di alamat 140000    │
│  refcount=2 | type=list        │  ← refcount naik!
│  data: [1, 2, 3]               │
└────────────────────────────────┘
```

Perhatikan: *reference count* naik dari 1 menjadi 2. Python tahu sekarang ada dua hal yang bergantung pada objek ini — jadi ia tidak boleh dihapus dari memori selama salah satu dari keduanya masih hidup.

### 2.3 Reference Counting sebagai Sistem Manajemen Memori

Python menggunakan **reference counting** sebagai mekanisme utama manajemen memori. Aturannya sederhana: ketika *reference count* sebuah objek mencapai nol, tidak ada lagi yang membutuhkannya — Python langsung membebaskan memori yang ia tempati.

```python
x = [1, 2, 3]   # list dibuat, refcount = 1
y = x            # aliasing, refcount = 2
del x            # x dihapus, refcount turun ke 1
del y            # y dihapus, refcount = 0 → objek dihapus dari memori!
```

Ini menjelaskan mengapa Python tidak membutuhkan programmer untuk memanggil `free()` atau `delete` secara manual seperti di C. Python melacak secara otomatis, via reference count, kapan sebuah objek sudah tidak diperlukan.

> **Mini Conclusion:** Referensi adalah alamat memori — bukan data itu sendiri. Ketika variabel menyimpan sebuah objek, ia sebenarnya menyimpan angka alamat tempat objek itu berada. Aliasing terjadi secara natural ketika dua variabel menyimpan alamat yang sama. Reference counting memungkinkan Python membersihkan memori secara otomatis ketika tidak ada variabel yang menyimpan suatu alamat.

---

## 3. Variables as Namespace Entries

### 3.1 Menyelesaikan Gambaran Fisik

Volume 4 dan 5 menetapkan bahwa variabel adalah label, bukan kotak. Di sini kita turun ke lapisan fisik yang paling konkret: variabel bukan sekadar 'nama', melainkan entri dalam sebuah tabel bernama namespace — dan memahami tabel itu menjelaskan bagaimana Python menemukan objek dari namanya.

Di bahasa seperti C, ketika kamu menulis `int x = 5;`, variabel `x` *benar-benar* adalah sebuah kotak di memori yang menyimpan angka 5 secara langsung. Alamat `x` *adalah* alamat tempat angka 5 berada.

Python bekerja secara fundamental berbeda. Ketika kamu menulis `x = 5`, yang terjadi di memori adalah:

1. Python menciptakan objek integer `5` di suatu lokasi, misalnya alamat `50000`.
2. Variabel `x` adalah sebuah entri dalam *namespace* — sebuah tabel yang memetakan nama ke alamat.
3. Entri `x` dalam tabel menyimpan angka `50000`.

```
Namespace (tabel nama → alamat):
┌────────┬────────┐
│  nama  │ alamat │
├────────┼────────┤
│  "x"   │ 50000  │
│  "y"   │ 70000  │
└────────┴────────┘

Objek di memori:
  [50000]: integer 5
  [70000]: string "halo"
```

Variabel bukan kotak yang menyimpan data. Variabel adalah nama yang terdaftar di tabel, dengan nilai entri berupa alamat tempat data sungguh-sungguh berada.

### 3.2 Mengapa Ini Penting untuk Part II dan III

Pemahaman bahwa variabel menyimpan *alamat*, bukan *data*, adalah kunci untuk memahami seluruh Part II dan Part III.

Ketika kita membahas **dynamic array** (struktur di balik list), kita akan melihat bahwa list menyimpan array dari *alamat-alamat*. Bukan array dari nilai-nilai. Setiap "elemen" list adalah sebuah alamat memori yang menunjuk ke objek elemen sesungguhnya.

```
List [1, "halo", 3.14] di memori:

  Array internal list:
  ┌──────────┬──────────┬──────────┐
  │  50000   │  70000   │  90000   │
  └────┬─────┴────┬─────┴────┬─────┘
       │          │          │
       ▼          ▼          ▼
   int(1)     str("halo")  float(3.14)
```

Ini menjelaskan mengapa list Python bisa menyimpan tipe yang berbeda: ia hanya menyimpan alamat, dan alamat adalah angka yang bisa menunjuk ke jenis objek apapun.

Demikian pula dengan **hash table** (struktur di balik dict): setiap "slot" dalam hash table menyimpan pasangan alamat key dan alamat value — bukan data key dan value secara langsung.

> **Mini Conclusion:** Di level fisik, variabel adalah entri dalam tabel nama→alamat. List adalah array dari alamat. Dict adalah array sparse dari pasangan alamat. Semua yang tampak sebagai "data" di permukaan Python adalah sebenarnya jaringan alamat yang saling menunjuk satu sama lain. Ini adalah fondasi dari seluruh fleksibilitas sekaligus seluruh kompleksitas Python.

---

### Mental Model Part I

```
RAM
└── Miliaran sel memori beralamat

Objek Python
└── Menempati kotak-kotak bersebelahan
└── Dimulai dengan header (refcount, type)
└── Diikuti data spesifik tipe

Variabel
└── Bukan kotak data
└── Adalah nama dalam namespace
└── Menyimpan alamat objek (reference)

Referensi
└── Adalah alamat memori
└── Copying referensi ≠ copying data
└── Aliasing = dua variabel, satu alamat

Reference Counting
└── Setiap objek punya hitungan referensinya
└── refcount = 0 → objek dihapus dari memori
```

---

# Part II — Dynamic Arrays (The Truth Behind Lists)

> *Tujuan Part II: Memahami mengapa operasi list berperforma seperti yang kita ukur.*
> *`append()` terasa instan. `insert(0, x)` terasa lambat.*
> *`len()` instan. Pencarian linear lambat.*
> *Semua perilaku ini bukan kebetulan — mereka adalah konsekuensi logis*
> *dari satu keputusan desain: list adalah dynamic array.*

---

## 4. Why List Append Is Fast

### 4.1 Intuisi: Meja Kerja yang Dipesan Lebih Besar dari Kebutuhan

Bayangkan kamu sedang menata dokumen di atas meja kerja. Setiap kali ada dokumen baru yang datang, kamu meletakkannya di tempat kosong berikutnya. Proses ini sangat cepat — tidak perlu memindahkan dokumen lain, tidak perlu mencari tempat.

Tapi bayangkan jika meja kerjamu persis pas dengan jumlah dokumen saat ini. Setiap kali ada dokumen baru, kamu terpaksa pindah ke ruangan yang lebih besar, memindahkan semua dokumen yang ada, lalu meletakkan dokumen baru. Sangat mahal.

Python memilih strategi yang lebih bijak: **selalu sediakan meja yang sedikit lebih besar dari yang diperlukan sekarang**. Ketika dokumen baru datang dan masih ada ruang kosong, letakkan saja. Cepat sekali. Hanya ketika meja benar-benar penuh, Python berpindah ke meja yang lebih besar — dan inilah proses yang mahal, namun jarang terjadi.

### 4.2 List di Memori: Array Bersebelahan

Secara internal, Python mengimplementasikan list sebagai **array yang bersebelahan** (*contiguous array*) di memori. "Bersebelahan" berarti seluruh elemen disimpan dalam blok memori yang tidak terputus — satu demi satu, tanpa celah.

Lebih tepatnya: yang disimpan bersebelahan bukan nilai elemen, melainkan *alamat* (referensi) ke objek elemen. Setiap "slot" dalam array internal list menyimpan satu alamat.

```
List ['a', 'b', 'c'] di memori (internal array):

Alamat array: 140000
┌──────────┬──────────┬──────────┬──────────┬──────────┬──────────┐
│ addr('a')│ addr('b')│ addr('c')│  (kosong)│  (kosong)│  (kosong)│
└──────────┴──────────┴──────────┴──────────┴──────────┴──────────┘
  slot 0     slot 1     slot 2     slot 3     slot 4     slot 5

Size = 3   (ada 3 elemen)
Capacity = 6  (ada 6 slot yang dialokasikan)
```

Perhatikan: ada 6 slot yang sudah dialokasikan, tapi hanya 3 yang terisi. Tiga slot sisanya menunggu seperti kursi kosong di meja kerja.

### 4.3 Append ketika Masih Ada Ruang: O(1)

Ketika kamu memanggil `append('d')` dan masih ada slot kosong, Python melakukan operasi yang sangat sederhana:

1. Tulis `addr('d')` ke slot berikutnya (slot 3).
2. Tambah 1 pada counter `size`.

```
Setelah append('d'):

┌──────────┬──────────┬──────────┬──────────┬──────────┬──────────┐
│ addr('a')│ addr('b')│ addr('c')│ addr('d')│  (kosong)│  (kosong)│
└──────────┴──────────┴──────────┴──────────┴──────────┴──────────┘
  slot 0     slot 1     slot 2     slot 3     slot 4     slot 5

Size = 4   (naik 1)
Capacity = 6  (tidak berubah)
```

Ini adalah operasi *konstan* — **O(1)**. Tidak peduli list berisi 10 atau 10 juta elemen sebelumnya, operasi ini selalu melibatkan jumlah langkah yang sama: satu tulis, satu increment.

### 4.4 Amortized Analysis: Rata-rata Jangka Panjang

Tapi apa yang terjadi ketika meja kerja penuh? Python harus melakukan operasi yang mahal: *resizing*. Kita akan membahasnya detail di chapter 6. Untuk sekarang, yang penting dipahami adalah konsep **amortized O(1)**.

Kata "amortized" berasal dari dunia akuntansi: biaya besar yang dibayar sekali diratakan ke banyak operasi kecil. Seperti membeli mesin yang mahal untuk pabrik — biayanya besar di awal, tapi karena mesin menghasilkan ribuan produk, biaya per produk menjadi sangat kecil.

Dalam konteks `append`: sesekali Python harus melakukan resizing yang mahal (O(n)). Tapi karena ini terjadi sangat jarang, dan antara satu resizing dengan resizing berikutnya ada ratusan atau ribuan `append` yang murah (O(1)), *rata-rata* biaya per `append` tetaplah O(1).

> **Behavior Note:** `list.append()` adalah **O(1) amortized**, bukan O(1) strict. Artinya, dalam jangka panjang, setiap append membutuhkan waktu konstan rata-rata. Tapi ada momen sesekali di mana satu `append` tertentu lebih lambat karena memicu resizing. Untuk kebanyakan kode praktis, perbedaan ini tidak terasa.

> **Mini Conclusion:** List Python adalah contiguous array dari alamat di memori. Python selalu mengalokasikan lebih banyak slot dari yang diisi saat ini. Ketika ada slot kosong, `append` adalah operasi O(1) murni: tulis alamat baru, increment size. Ketika tidak ada slot kosong, terjadi resizing yang mahal tapi jarang — membuat append "O(1) amortized" secara keseluruhan.

---

## 5. Capacity vs Size

### 5.1 Dua Angka yang Berbeda

Setiap list Python mengelola dua angka internal yang berbeda namun sering dikacaukan:

**Size** adalah jumlah elemen yang saat ini ada dalam list. Inilah yang dikembalikan oleh `len()`. Ini adalah angka yang programmer lihat dan gunakan.

**Capacity** adalah jumlah slot yang sudah dialokasikan dalam array internal. Ini adalah angka yang Python kelola secara internal dan tidak langsung terlihat oleh programmer.

Keduanya selalu memenuhi hubungan: `size ≤ capacity`.

```
Representasi visual:

List setelah beberapa append:

┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐
│  1  │  2  │  3  │  4  │     │     │     │     │
└─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘
 ◄──────────────────────►◄────────────────────────►
        Size = 4                 Slack = 4
 ◄───────────────────────────────────────────────►
                  Capacity = 8
```

Slot yang berisi elemen adalah `size`. Slot yang sudah dialokasikan (termasuk yang kosong) adalah `capacity`. Selisih antara capacity dan size disebut *slack* — ruang cadangan yang Python siapkan untuk `append` berikutnya.

### 5.2 Mengamati Ini di Python

Python tidak menyediakan cara langsung untuk membaca capacity dari sebuah list. Tapi kita bisa mengamatinya secara tidak langsung melalui `sys.getsizeof()` yang mengukur ukuran memori objek.

```python
import sys

lst = []
kapasitas_sebelumnya = sys.getsizeof(lst)

for i in range(20):
    lst.append(i)
    ukuran_sekarang = sys.getsizeof(lst)
    if ukuran_sekarang != kapasitas_sebelumnya:
        print(f"len={len(lst)}: ukuran naik ke {ukuran_sekarang} bytes")
        kapasitas_sebelumnya = ukuran_sekarang

# Output (kira-kira, tergantung platform):
# len=1:  ukuran naik ke 88 bytes   (capacity menjadi 4)
# len=5:  ukuran naik ke 120 bytes  (capacity menjadi 8)
# len=9:  ukuran naik ke 184 bytes  (capacity menjadi 16)
# len=17: ukuran naik ke 216 bytes  (capacity menjadi 25)
```

Perhatikan pola lompatan: ukuran memori tidak naik setiap `append`, melainkan melompat pada momen-momen tertentu — itulah saat resizing terjadi. Di antara lompatan-lompatan itu, `append` terjadi tanpa perubahan alokasi memori sama sekali.

### 5.3 Perbedaan Ini Penting untuk Pemahaman Performa

Memahami perbedaan size dan capacity secara eksplisit membuka pemahaman berikut:

- `len(lst)` selalu cepat: Python hanya membaca counter `size`, tidak menghitung elemen.
- `append()` biasanya cepat: Python hanya menulis ke slot kosong dan increment `size`.
- `append()` sesekali lambat: ketika `size == capacity`, Python harus resize.

> **Mini Conclusion:** `size` adalah berapa banyak elemen yang ada (apa yang `len()` kembalikan). `capacity` adalah berapa banyak slot yang sudah dialokasikan. Perbedaan inilah yang membuat `append` efisien: Python tidak perlu meminta memori baru untuk setiap elemen baru — hanya ketika cadangan habis.

---

## 6. Dynamic Resizing

### 6.1 Ketika Cadangan Habis

Saat `size == capacity`, slot cadangan telah habis. Append berikutnya tidak bisa langsung menulis ke slot yang sudah ada — tidak ada lagi. Python harus melakukan **resizing**: mencari blok memori baru yang lebih besar, memindahkan semua konten yang ada, lalu menggunakannya.

Prosesnya, langkah demi langkah:

**Langkah 1:** Python meminta sistem operasi untuk mengalokasikan blok memori baru yang lebih besar dari yang sekarang.

**Langkah 2:** Python menyalin semua alamat elemen dari array lama ke array baru.

**Langkah 3:** Python membebaskan blok memori lama.

**Langkah 4:** Python menulis elemen baru ke slot berikutnya yang sekarang sudah tersedia.

```
Sebelum resize (capacity penuh):
┌─────┬─────┬─────┬─────┐
│  A  │  B  │  C  │  D  │    Size = 4, Capacity = 4
└─────┴─────┴─────┴─────┘

append('E') dipanggil → capacity habis!

Langkah 1: Alokasikan blok baru yang lebih besar:
┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐
│     │     │     │     │     │     │     │     │    (kosong semua)
└─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘

Langkah 2: Salin semua elemen lama:
┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐
│  A  │  B  │  C  │  D  │     │     │     │     │
└─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘

Langkah 3: Bebaskan blok lama (hilang dari memori)

Langkah 4: Tulis elemen baru:
┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐
│  A  │  B  │  C  │  D  │  E  │     │     │     │
└─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘
                              Size = 5, Capacity = 8
```

### 6.2 Strategi Pertumbuhan

Python tidak menggandakan capacity secara tepat dua kali lipat. Ia menggunakan rumus pertumbuhan yang lebih nuansif, dirancang untuk menyeimbangkan antara efisiensi waktu dan penggunaan memori.

Secara kasar, capacity baru adalah sekitar `capacity_lama + capacity_lama // 8 + (3 atau 6)`. Untuk list kecil, ini mendekati dua kali lipat. Untuk list besar, pertumbuhannya lebih konservatif — sekitar 12,5%.

Urutan capacity yang kamu akan lihat dalam praktik (kira-kira):

```
Jumlah elemen → Capacity yang dialokasikan:
0    →   0
1    →   4
5    →   8
9    →  16
17   →  25
26   →  35
35   →  46
46   →  58
...
```

Setiap kali terjadi resizing, lompatan capacity cukup besar sehingga berikutnya bisa menampung banyak `append` sebelum perlu resize lagi.

### 6.3 Mengapa Ini O(1) Amortized: Intuisi Matematisnya

Bayangkan kita mulai dari list kosong dan melakukan N kali `append`. Berapa banyak operasi total yang terjadi?

Setiap `append` biasa: 1 operasi (tulis + increment). Setiap resizing: O(k) operasi di mana k adalah jumlah elemen yang disalin.

Dengan strategi pertumbuhan geometris, resizing terjadi secara berkala seiring list tumbuh — misalnya pada ukuran 1, 2, 4, 8, 16, 25, 35...
(Python menggunakan faktor ~1.125x, bukan tepat 2x seperti yang sering diasumsikan — tapi argumen amortized O(1) berlaku untuk pertumbuhan geometris apapun, selama faktornya lebih besar dari 1.)
dan setiap resizing menyalin seluruh elemen. Total operasi penyalinan adalah:

```
1 + 2 + 4 + 8 + ... + N = O(N)
```

Total N operasi `append` membutuhkan O(N) + O(N) = O(N) operasi total. Dibagi N, rata-rata per operasi adalah O(1).

Inilah arti **amortized O(1)**: biaya mahal dari resizing tersebar rata ke seluruh `append` yang murah, menghasilkan rata-rata konstan per operasi.

> **Mini Conclusion:** Dynamic resizing adalah proses tiga langkah: alokasi blok baru, salin semua elemen, bebaskan blok lama. Python menggunakan strategi pertumbuhan sekitar 1.125x yang menjamin amortized O(1) untuk `append`. Setiap resizing mahal, tapi karena jarang terjadi dan selalu diikuti banyak `append` murah, biayanya tersebar sehingga rata-rata tetap konstan.

---

## 7. Why Insertions Can Be Expensive

### 7.1 Sifat Kontinu yang Tak Bisa Dihindari

Array yang bersebelahan di memori memiliki satu kekuatan dan satu kelemahan dari sifat yang sama: kontinu.

Kekuatannya: akses berdasarkan indeks sangat cepat. Untuk mengakses elemen ke-N, Python hanya perlu menghitung `alamat_awal + N * ukuran_satu_slot`. Satu operasi aritmetika, langsung ke lokasi yang tepat. O(1).

Kelemahannya: menyisipkan elemen di tengah memaksa semua elemen setelahnya untuk bergeser satu posisi ke kanan, agar urutan tetap terjaga.

### 7.2 Visualisasi: Insert di Posisi 0

Anggap kita punya list dengan 5 elemen dan ingin menyisipkan elemen baru di posisi 0 (`insert(0, 'X')`).

```
Sebelum insert(0, 'X'):
┌─────┬─────┬─────┬─────┬─────┬─────┐
│  A  │  B  │  C  │  D  │  E  │     │
└─────┴─────┴─────┴─────┴─────┴─────┘
  [0]   [1]   [2]   [3]   [4]   [5]

Langkah 1: Geser SEMUA elemen satu posisi ke kanan:
┌─────┬─────┬─────┬─────┬─────┬─────┐
│     │  A  │  B  │  C  │  D  │  E  │
└─────┴─────┴─────┴─────┴─────┴─────┘

Langkah 2: Tulis 'X' ke posisi 0:
┌─────┬─────┬─────┬─────┬─────┬─────┐
│  X  │  A  │  B  │  C  │  D  │  E  │
└─────┴─────┴─────┴─────┴─────┴─────┘
```

Untuk menyisipkan di posisi 0 dengan list berisi N elemen, semua N elemen harus digeser. Itulah O(N).

### 7.3 O(N) Insertion: Hubungannya dengan Posisi

Biaya insert bergantung pada *posisi* penyisipan:

- `insert(-1, x)` atau `append(x)`: 0 elemen digeser → O(1)
- `insert(N//2, x)`: sekitar N/2 elemen digeser → O(N)
- `insert(0, x)`: N elemen digeser → O(N)

```python
import time

lst = list(range(100_000))

# Append di akhir — cepat (O(1))
t = time.time()
lst.append(999)
print(f"Append: {time.time()-t:.6f}s")

# Insert di tengah — lebih lambat
t = time.time()
lst.insert(50_000, 999)
print(f"Insert tengah: {time.time()-t:.6f}s")

# Insert di awal — paling lambat (O(n))
t = time.time()
lst.insert(0, 999)
print(f"Insert awal: {time.time()-t:.6f}s")
```

Hal yang sama berlaku untuk `del lst[0]` atau `lst.pop(0)` — menghapus di awal juga O(N) karena semua elemen yang tersisa harus digeser ke kiri.

### 7.4 Konsekuensi Praktis

Pemahaman ini mengubah cara menulis kode:

Jika kamu sering perlu menambahkan elemen di *akhir* dan memproses dari akhir — list adalah pilihan sempurna.

Jika kamu sering perlu menambahkan elemen di *awal* — pertimbangkan `collections.deque`, yang dirancang untuk operasi efisien di kedua ujung.

Jika kamu sering perlu menyisipkan di *tengah* — pertimbangkan struktur data yang berbeda, atau pertimbangkan ulang algoritma kamu.

> **Behavior Note:** `list.pop(0)` dan `list.insert(0, x)` adalah O(N), bukan O(1). Jika kamu menggunakannya di dalam loop besar, total biayanya menjadi O(N²) — performa yang bisa sangat buruk untuk data besar. Gunakan `collections.deque` untuk antrian yang butuh operasi di kedua ujung.

> **Mini Conclusion:** Insert di tengah atau di awal list adalah O(N) karena sifat kontinu array: semua elemen setelah posisi penyisipan harus digeser. Biaya ini bukan kegagalan desain — ia adalah trade-off dari keputusan menggunakan array kontinu, yang sekaligus memberikan O(1) akses berdasarkan indeks. Setiap keputusan arsitektur membawa trade-off yang menyertai.

---

### Mental Model Part II: Dynamic Array

```
Representasi Internal List:
┌──────────────────────────────────┐
│  METADATA LIST                   │
│  size     = N (jumlah elemen)    │
│  capacity = M (slot dialokasikan)│
│  pointer  = → [array internal]   │
└──────────────┬───────────────────┘
               │
               ▼
┌─────┬─────┬─────┬─────┬─────┬─────┬─────┐
│ ref │ ref │ ref │ ref │  -  │  -  │  -  │  Array internal
└─────┴─────┴─────┴─────┴─────┴─────┴─────┘
 ←────────────────────────→←───────────────→
        terisi (size=4)     kosong (slack=3)
 ←─────────────────────────────────────────→
              capacity = 7

Operasi dan Kompleksitasnya:
  append()      → O(1) amortized  (tulis ke slot kosong / resize jika penuh)
  pop()         → O(1) amortized  (hapus dari akhir)
  lst[i]        → O(1)            (hitung alamat langsung)
  insert(0, x)  → O(N)            (geser semua elemen)
  pop(0)        → O(N)            (geser semua elemen)
  x in lst      → O(N)            (cek satu per satu)
```

---

# Part III — Hash Tables (The Engine Behind Dict & Set)

> *Tujuan Part III: Membuka arsitektur internal dari dict dan set hingga ke level mekanis.*
> *Di Volume 5, kita melihat hash table dari luar — hash → slot → value.*
> *Di sini, kita membuka tutupnya: apa isi slot itu? Bagaimana Python menentukan slot mana?*
> *Bagaimana collision diselesaikan? Bagaimana tabel tumbuh?*
> *Ini adalah jantung dari mengapa dict dan set begitu cepat.*

---

## 8. Internal Layout of a Hash Table

### 8.1 Bukan Array Biasa: Array yang Jarang

Jika dynamic array (list) adalah meja kerja dengan slot yang terisi rapat-rapat berurutan, maka hash table adalah loker penyimpanan besar di stasiun kereta yang *sengaja dibiarkan banyak kosong*.

Di loker itu, tidak ada sistem "isi dari kiri ke kanan." Setiap barang ditempatkan di loker nomor tertentu berdasarkan rumus yang dihitung dari karakteristik barang itu sendiri. Loker 1, 5, 27, 48 mungkin terisi sementara semua loker di antaranya kosong.

Inilah **sparse array** — array yang sengaja jarang. Dan ini adalah fondasi fisik dari hash table Python.

### 8.2 Apa yang Ada di Dalam Setiap Slot

Sebuah dict Python secara internal memiliki array dari slot, di mana setiap slot dapat berada dalam salah satu dari tiga kondisi:

**Empty** (kosong): slot ini belum pernah terisi. Python menggunakannya ketika mencari apakah key tertentu ada — jika menemukan slot kosong, berarti key tidak ada.

**Active** (aktif): slot ini berisi pasangan key-value yang valid. Ia menyimpan tiga hal: hash value dari key, alamat objek key, dan alamat objek value.

**Dummy** (bekas): slot ini pernah terisi tapi sudah dihapus. Python menggunakannya sebagai penanda bahwa proses pencarian harus terus berlanjut (tidak berhenti seperti saat menemukan Empty).

```
Array internal dict (representasi konseptual):

Indeks:  0        1        2        3        4        5        6        7
       ┌────────┬────────┬────────┬────────┬────────┬────────┬────────┬────────┐
       │ EMPTY  │ ACTIVE │ EMPTY  │ DUMMY  │ ACTIVE │ EMPTY  │ ACTIVE │ EMPTY  │
       │        │hash=42 │        │(bekas) │hash=89 │        │hash=13 │        │
       │        │key=... │        │        │key=... │        │key=... │        │
       │        │val=... │        │        │val=... │        │val=... │        │
       └────────┴────────┴────────┴────────┴────────┴────────┴────────┴────────┘
```

### 8.3 Load Factor: Mengapa Tabel Harus Jarang

**Load factor** adalah rasio antara slot aktif terhadap total slot. Python menjaga load factor dict di bawah **2/3** (sekitar 66.7%).

Mengapa harus jarang? Karena efisiensi hash table sangat bergantung pada tersedianya slot kosong untuk proses pencarian. Semakin padat tabel, semakin sering terjadi collision, semakin lama pencarian berlangsung. Pada titik ekstrem (tabel penuh), hash table tidak lebih baik dari pencarian linear.

```
Load Factor dan Performa (konseptual):

Load Factor   Rata-rata Probes
0.50          1.5 probes per lookup
0.67          2.0 probes per lookup (batas Python)
0.80          3.0 probes per lookup
0.90          5.5 probes per lookup
0.99          50  probes per lookup (mendekati linear!)
```

Dengan batas 2/3, Python memastikan bahwa rata-rata setiap lookup hanya membutuhkan sekitar 2 kali "mengetuk slot" — jauh dari O(N).

> **Mini Conclusion:** Hash table adalah array yang sengaja jarang. Setiap slot menyimpan hash value, alamat key, dan alamat value — atau penanda kosong/dummy. Load factor dijaga di bawah 2/3 untuk memastikan selalu ada cukup slot kosong bagi proses pencarian berlangsung efisien. Kekompakan adalah musuh hash table.

---

## 9. Buckets and Lookup

### 9.1 Apa Itu Bucket

Istilah **bucket** (atau **slot**) mengacu pada satu entri dalam array internal hash table. Setiap bucket adalah "laci" potensial tempat sebuah pasangan key-value bisa berada.

Jumlah bucket selalu merupakan pangkat dua dalam CPython (4, 8, 16, 32, ...). Ini bukan kebetulan — pangkat dua memungkinkan perhitungan indeks yang lebih cepat melalui operasi bitwise.

### 9.2 Jalur Lookup: Dari Key ke Bucket

Saat kamu mengakses `d["nama"]`, Python melakukan serangkaian langkah yang sangat cepat:

**Langkah 1:** Hitung hash value dari key.

```
hash_value = hash("nama")
# misalnya: hash_value = 7342987612345
```

**Langkah 2:** Petakan hash ke indeks bucket menggunakan operasi modulo (atau bitwise AND untuk pangkat dua).

```
indeks = hash_value % jumlah_bucket
# misalnya: 7342987612345 % 8 = 1
# → cek bucket nomor 1
```

**Langkah 3:** Periksa bucket nomor 1.

- Jika *empty*: key tidak ada → `KeyError`
- Jika *active* dengan key yang sama: ditemukan → kembalikan value
- Jika *active* dengan key berbeda: collision → cari bucket berikutnya
- Jika *dummy*: ada entry yang dihapus → lanjutkan pencarian

```
Visualisasi lookup d["nama"]:

hash("nama") → 7342987612345 → % 8 → indeks = 1

Array internal:
     [0]      [1]       [2]
  ┌────────┬────────┬────────┐...
  │ EMPTY  │ ACTIVE │ EMPTY  │
  │        │hash=73.│        │
  │        │"nama"  │        │
  │        │"Brata" │        │
  └────────┴────────┴────────┘
                ▲
          Langsung ke sini!
          Cocok → kembalikan "Brata"
```

### 9.3 Mengapa Ini O(1)

Kunci dari O(1)-nya adalah bahwa **hash menghitung indeks, bukan mencari indeks**.

Dengan list, untuk menemukan elemen, kamu harus memeriksa satu per satu: cek indeks 0, lalu 1, lalu 2, ... — sampai ketemu atau habis.

Dengan dict, kamu langsung *menghitung* di mana seharusnya elemen itu berada berdasarkan hash-nya. Tidak ada pencarian bertahap. Tidak ada perbandingan satu per satu. Kamu pergi langsung ke "pintu yang benar" — atau paling tidak, sangat dekat dengannya.

Jumlah langkah tidak bergantung pada ukuran dict. Dict dengan 10 entri dan dict dengan 10 juta entri membutuhkan jumlah "ketukan pintu" yang hampir sama.

> **Mini Conclusion:** Bucket adalah slot dalam array internal hash table. Lookup bekerja dalam tiga langkah: hitung hash, petakan ke indeks, periksa bucket. Ini O(1) karena hash *menghitung* lokasi secara matematika — bukan *mencari* lokasi secara bertahap. Inilah perbedaan fundamental antara hash-based lookup dan linear search.

---

## 10. Collision Resolution

### 10.1 Ketika Dua Key Mengincar Slot yang Sama

Dengan hash value yang sangat besar dan bucket yang jumlahnya jauh lebih sedikit, tak terhindarkan: dua key yang berbeda bisa menghasilkan indeks yang sama. Inilah **collision**.

```
hash("alice") % 8 = 3
hash("charlie") % 8 = 3  ← collision! Keduanya ingin bucket [3]
```

Collision bukan kegagalan sistem — ia adalah realitas matematis yang tak bisa dihindari. Sistem yang baik menangani collision dengan elegan, bukan menghindarinya.

### 10.2 Open Addressing: Python's Strategy

Python menggunakan strategi **open addressing** untuk menangani collision: ketika bucket yang diinginkan terisi oleh key yang berbeda, Python *mencari bucket lain* dalam array yang sama. Tidak ada struktur data terpisah (seperti linked list) yang digunakan.

Proses mencari bucket berikutnya disebut **probing**. Python menggunakan probe sequence yang tidak sekadar linier — ia mencampur nilai hash saat ini untuk menghasilkan indeks berikutnya, yang membantu menyebarkan entri lebih merata dan menghindari *clustering* (penumpukan di area tertentu).

Secara konseptual, proses ini bisa divisualisasikan sebagai:

```
Collision di bucket [3] saat insert ("charlie", 45):

Coba bucket [3]: terisi oleh "alice" → bukan "charlie" → collision!
Coba bucket [7]: terisi oleh "bob"   → bukan "charlie" → collision lagi!
Coba bucket [2]: EMPTY               → tulis ("charlie", 45) di sini!

Array setelah insert:
     [0]   [1]   [2]         [3]       [4]   [5]   [6]      [7]
  ┌──────┬──────┬─────────┬──────────┬──────┬──────┬──────┬──────────┐
  │EMPTY │EMPTY │CHARLIE  │  ALICE   │EMPTY │EMPTY │EMPTY │  BOB     │
  │      │      │hash=...│  hash=...│      │      │      │  hash=... │
  │      │      │val=45  │  val=25  │      │      │      │  val=30   │
  └──────┴──────┴─────────┴──────────┴──────┴──────┴──────┴──────────┘
```

### 10.3 Lookup dengan Collision

Ketika mencari key yang mengalami collision saat penyisipan, Python mengulang probe sequence yang sama:

1. Hitung indeks awal dari hash.
2. Periksa bucket tersebut.
3. Jika empty: tidak ditemukan (berhenti).
4. Jika active dengan key yang sama: ditemukan!
5. Jika active dengan key berbeda: lanjutkan probe.
6. Jika dummy: lanjutkan probe (tidak berhenti — mungkin ada entry di baliknya).

```
Mencari d["charlie"]:

hash("charlie") % 8 = 3
Coba [3]: "alice" bukan "charlie" → lanjutkan
Coba [7]: "bob"   bukan "charlie" → lanjutkan
Coba [2]: "charlie" == "charlie"  → DITEMUKAN! return 45
```

### 10.4 Dummy Entries: Jejak Penghapusan

Ketika sebuah entry dihapus dari hash table, Python tidak langsung mengosongkan bucket tersebut. Jika dikosongkan, lookup untuk key lain yang melewati bucket ini (selama probing) akan berhenti terlalu dini — mengira key tidak ada padahal ada di slot berikutnya.

Sebaliknya, Python menandai bucket sebagai **Dummy** (bekas). Dummy berarti: "ada sesuatu yang pernah di sini, tapi sudah pergi. Jangan berhenti di sini saat mencari — terus lanjutkan probing."

Dummy slots ini adalah overhead yang tidak terhindarkan dari operasi delete. Mereka "memakan" kapasitas tanpa menyimpan data aktif. Inilah salah satu mengapa Python akan melakukan rehashing (bukan hanya resizing) secara berkala.

> **Mini Conclusion:** Python menggunakan open addressing untuk collision resolution: ketika bucket yang diinginkan terisi, Python mencari bucket berikutnya melalui probe sequence. Proses ini determinik — probe sequence yang sama selalu menghasilkan urutan bucket yang sama untuk key yang sama. Dummy entries diperlukan untuk menjaga kebenaran lookup setelah penghapusan.

---

## 11. Resizing and Rehashing

### 11.1 Kapan Tabel Harus Tumbuh

Berbeda dengan list yang hanya resize ketika penuh, hash table Python resize ketika load factor melewati batas 2/3. Ini terjadi lebih awal — bahkan ketika masih ada slot kosong.

```python
# Perkiraan kapan dict akan resize:
d = {}
# Kapasitas awal: 8 bucket
# Resize terjadi saat slot aktif > 8 * (2/3) ≈ 5 entri
# Kapasitas baru: 32 bucket
# Resize berikutnya saat slot aktif > 32 * (2/3) ≈ 21 entri
# dst.
```

Kapasitas selalu pangkat dua: 8 → 32 → 128 → 512 → ...
(Dict kecil bisa tumbuh hingga 4x per resize untuk menghindari rehashing berulang di fase awal; dict yang lebih besar cenderung mendekati 2x. Lompatan yang lebih agresif di awal adalah trade-off yang disengaja antara memori dan biaya rehashing.)

### 11.2 Mengapa Tidak Bisa Sekadar Menyalin

Inilah perbedaan krusial antara list resizing dan dict resizing:

Ketika list resize, Python menyalin alamat elemen ke array baru. Posisi elemen dalam array baru sama dengan posisi di array lama (elemen ke-0 tetap di indeks 0).

Ketika dict resize, Python **tidak bisa** mempertahankan posisi. Indeks bucket ditentukan oleh `hash(key) % jumlah_bucket`. Ketika `jumlah_bucket` berubah, indeks setiap key berubah. Semua entry harus ditempatkan ulang di posisi yang baru-dihitung.

Proses ini disebut **rehashing**.

### 11.3 Proses Rehashing

```
Dict sebelum resize (8 bucket, 5 entry = load 62.5% → mendekati batas):

     [0]   [1]   [2]   [3]   [4]   [5]   [6]   [7]
  ┌──────┬──────┬──────┬──────┬──────┬──────┬──────┬──────┐
  │ E=5  │EMPTY │ A=25 │DUMMY │ B=30 │EMPTY │ C=45 │EMPTY │
  └──────┴──────┴──────┴──────┴──────┴──────┴──────┴──────┘

Insert entry baru → melewati batas 2/3 → REHASH!

1. Alokasikan array baru: 32 bucket (semua empty)
2. Iterasi semua entry AKTIF dari array lama:
   - "E":25 → hash("E") % 32 = indeks_baru_E → tulis ke sana
   - "A":25 → hash("A") % 32 = indeks_baru_A → tulis ke sana
   - "B":30 → hash("B") % 32 = indeks_baru_B → tulis ke sana
   - "C":45 → hash("C") % 32 = indeks_baru_C → tulis ke sana
   (DUMMY tidak disalin — kesempatan untuk "membersihkan" sisa penghapusan)
3. Tulis entry baru ke indeks yang dihitung dalam array baru
4. Bebaskan array lama

Setelah rehash: 32 bucket, 5 entry aktif (load = 15.6% — jauh dari batas)
```

Perhatikan: rehashing juga menghilangkan semua dummy slots! Ini adalah "pembersihan" alami yang menjaga tabel tetap segar.

### 11.4 Biaya Amortized Rehashing

Seperti list resizing, rehashing adalah operasi O(N) yang jarang terjadi. Dan seperti list, amortized-nya tetap O(1) per operasi insert karena:

- Setiap rehashing melipatgandakan kapasitas.
- Antara dua rehashing berturut-turut, bisa ada ratusan insert murah.
- Total biaya tersebar rata ke banyak operasi.

> **Behavior Note:** Dummy entries tidak ikut rehashed — mereka *dibuang*. Ini berarti rehashing bukan hanya "resize" tetapi juga "defragmentasi" tabel dari sampah penghapusan. Sebuah dict yang sering melakukan delete lalu insert akan sesekali mengalami rehashing yang membersihkan dummy entries ini.

> **Mini Conclusion:** Hash table resize terjadi saat load factor melewati 2/3, *sebelum* tabel penuh. Karena indeks bucket bergantung pada ukuran tabel, resize harus diikuti rehashing: semua entry aktif dihitung ulang indeksnya dan ditulis ke tabel baru. Dummy entries dibuang dalam proses ini. Biaya O(N) dari rehashing tersebar rata sehingga amortized O(1) per insert.

---

### Mental Model Part III: Hash Table

```
Representasi Internal Dict:
┌──────────────────────────────────────┐
│  METADATA DICT                        │
│  used     = jumlah entri aktif       │
│  fill     = aktif + dummy            │
│  size     = jumlah bucket (pangkat 2)│
│  pointer  = → [array internal]       │
└──────────────┬───────────────────────┘
               │
               ▼
┌────────┬────────┬────────┬────────┬────────┐
│ EMPTY  │ ACTIVE │ DUMMY  │ ACTIVE │ EMPTY  │  ...  Array sparse
│        │ h,k,v  │(bekas) │ h,k,v  │        │       (pangkat 2)
└────────┴────────┴────────┴────────┴────────┘

Lookup path:
  hash(key) → indeks → cek bucket
  ├── EMPTY → tidak ada → KeyError
  ├── ACTIVE, key cocok → ditemukan!
  ├── ACTIVE, key beda → probe berikutnya
  └── DUMMY → probe berikutnya

Operasi dan Kompleksitasnya:
  d[key]        → O(1) avg  (hash → indeks → cek)
  d[key] = val  → O(1) avg  (hash → indeks → tulis)
  del d[key]    → O(1) avg  (tandai sebagai DUMMY)
  key in d      → O(1) avg  (hash → indeks → cek)
```

---

# Part IV — Performance as Emergent Behavior

> *Tujuan Part IV: Menghubungkan internal mechanics dengan angka-angka performa yang kita lihat.*
> *Big O bukan sesuatu yang "ditetapkan" dari atas — ia adalah konsekuensi yang muncul*
> *dari keputusan desain internal. Di sini kita melihat mengapa.*

---

## 12. Understanding Big O Through Reality

### 12.1 Big O Bukan Waktu — Ia adalah Laju Pertumbuhan

Ada salah paham yang sangat umum tentang notasi Big O: bahwa ia mengukur waktu eksekusi. Bukan. Big O mengukur **bagaimana waktu eksekusi tumbuh seiring bertambahnya ukuran input**.

O(1) tidak berarti "satu milidetik." Ia berarti "waktu eksekusi tidak tumbuh seiring bertambahnya N." O(N) tidak berarti "lambat." Ia berarti "waktu eksekusi tumbuh proporsional dengan N." O(N²) berarti "waktu eksekusi tumbuh dengan kuadrat N."

Sebuah operasi O(1) yang membutuhkan 100 langkah internal tetaplah O(1), karena jumlah langkahnya tidak berubah berapa pun besar N. Sebuah operasi O(N) dengan konstanta yang sangat kecil mungkin *terasa* cepat untuk N kecil — tapi akan mengalahkan O(1) jika N cukup besar? Tidak — karena O(1) tidak tumbuh sama sekali.

### 12.2 O(1) Berarti "Konstan" — Bukan "Satu Langkah"

Dari Part II dan III, kita sekarang tahu bahwa:

- `append()` adalah O(1) amortized: kadang 1 langkah, kadang N langkah (saat resize). Rata-rata konstan.
- `dict[key]` adalah O(1) average: biasanya 1-2 langkah (probe). Dalam kasus terburuk (kolisi ekstrem), lebih banyak. Tapi rata-rata konstan.

"O(1)" di sini berarti bahwa jumlah langkah tidak bergantung pada ukuran struktur data — bukan bahwa selalu tepat satu langkah.

### 12.3 Worst Case vs Average Case vs Amortized

Tiga cara berbeda menganalisis kompleksitas:

**Worst case**: skenario terburuk yang mungkin terjadi.

- Append: O(N) saat resize terjadi.
- Dict lookup: O(N) jika semua key collision ke satu bucket.

**Average case**: rata-rata di semua kemungkinan input.

- Append: O(1) amortized.
- Dict lookup: O(1) dengan hash function yang baik.

**Amortized**: rata-rata di rangkaian operasi (bukan per operasi individu).

- Append: O(1) amortized — biaya resize tersebar ke banyak append murah.

> **Mini Conclusion:** Big O mengukur laju pertumbuhan, bukan waktu mutlak. O(1) berarti konstan, bukan "satu langkah." Dict dan set menawarkan O(1) *average case* — asumsinya adalah hash function yang baik dan load factor yang terjaga. List `append` adalah O(1) *amortized* — biaya resize tersebar ke banyak operasi murah.

---

## 13. Why List, Dict, and Set Behave Differently

### 13.1 Peta Performa yang Lahir dari Desain Internal

Setiap perbedaan performa antara list, dict, dan set bukan kebetulan atau keputusan sewenang-wenang. Mereka adalah konsekuensi langsung dari pilihan implementasi internal.

```
Perbandingan Operasi Utama:

Operasi              List       Dict       Set
─────────────────────────────────────────────
Akses via indeks     O(1)       N/A        N/A
Akses via key        N/A        O(1) avg   N/A
Membership test      O(N)       O(1) avg   O(1) avg
Tambah elemen        O(1) am.   O(1) avg   O(1) avg
Hapus (di akhir)     O(1) am.   O(1) avg   O(1) avg
Hapus (di tengah)    O(N)       O(1) avg   O(1) avg
Insert di awal       O(N)       N/A        N/A
Iterasi              O(N)       O(N)       O(N)
```

### 13.2 Mengapa List Search Adalah O(N)

List adalah contiguous array. Tidak ada "jalan pintas" untuk menemukan nilai tertentu tanpa posisinya. Python harus memeriksa slot 0, lalu 1, lalu 2, ... sampai ketemu atau habis.

```python
lst = list(range(1_000_000))

# Mencari 999_999 → Python memeriksa 1 juta elemen
print(999_999 in lst)   # True, tapi membutuhkan ~1 juta perbandingan
```

Tidak ada yang bisa dilakukan Python untuk mempercepat ini — selama list tidak diurutkan dan tidak ada struktur tambahan, pencarian linier adalah satu-satunya cara.

### 13.3 Mengapa Dict/Set Lookup Adalah O(1) Average

Dict dan set menggunakan hash table. Hash menghitung *indeks secara langsung* dari nilai key. Tidak ada iterasi. Python melompat langsung (atau sangat dekat) ke lokasi yang tepat.

```python
st = set(range(1_000_000))

# Mencari 999_999 → Python menghitung indeks langsung
print(999_999 in st)    # True, hanya 1-2 langkah
```

### 13.4 Mengapa List Index Access Adalah O(1)

Ini mungkin terasa berlawanan dengan intuisi: jika list search lambat, kenapa index access cepat?

Karena ini pertanyaan yang berbeda! `lst[i]` tidak mencari nilai — ia menghitung *alamat langsung* dari elemen ke-i:

```
alamat_element_i = alamat_awal_array + i * ukuran_satu_pointer
```

Satu operasi aritmetika. Tidak peduli i = 0 atau i = 999_999. Inilah keistimewaan array kontinu: akses posisi adalah O(1) karena posisi langsung diterjemahkan ke alamat.

> **Mini Conclusion:** Perbedaan performa list vs dict/set bukan keberuntungan atau ketidakberuntungan. List memiliki O(1) akses posisi (karena array kontinu) tapi O(N) pencarian (karena tidak ada shortcut). Dict/set memiliki O(1) akses key (karena hash table) dan O(1) membership test. Pilih struktur berdasarkan operasi mana yang paling sering kamu butuhkan.

---

## 14. Hidden Costs

### 14.1 Overhead Memori Dynamic Array

List selalu mengalokasikan lebih banyak slot dari yang diisi. Ini berarti ada memori yang "terbuang" untuk kapasitas yang belum digunakan.

```python
import sys

# List dengan 100 elemen
lst = list(range(100))
print(sys.getsizeof(lst))   # ~920 bytes

# Tapi kita hanya membutuhkan 100 referensi
# 100 * 8 bytes (ukuran pointer 64-bit) = 800 bytes
# Selisih ~120 bytes adalah overhead metadata dan slack capacity
```

Overhead ini biasanya kecil dan dapat diterima, tetapi bisa signifikan jika kamu membuat jutaan list kecil.

### 14.2 Overhead Memori Hash Table

Dict jauh lebih boros memori dari list dengan data yang sama, karena harus menjaga load factor di bawah 2/3.

```python
# Dict dengan 100 entry
d = {i: i for i in range(100)}
print(sys.getsizeof(d))     # ~4700+ bytes

# List dengan 100 elemen
lst = list(range(100))
print(sys.getsizeof(lst))   # ~920 bytes
```

Untuk 100 entry, dict membutuhkan setidaknya 128 bucket (pangkat dua pertama yang bisa menampung 100 entry dengan load factor < 2/3). Setiap bucket menyimpan hash value, alamat key, dan alamat value — lebih banyak data per slot dibandingkan list.

### 14.3 Biaya Hash Computation

Ketika kamu melakukan `d["nama"]`, Python harus menghitung `hash("nama")`. Ini bukan gratis — ada komputasi yang terlibat.

Untuk tipe bawaan seperti `int` dan string yang pendek, Python bisa meng-*cache* hash value (menyimpannya agar tidak dihitung ulang). Ini adalah optimasi penting.

Untuk string: hash dihitung sekali dan disimpan dalam objek string. Setiap lookup berikutnya menggunakan nilai yang sudah tersimpan.

Untuk int: hash biasanya sama dengan nilainya (untuk int kecil), sehingga hampir tanpa biaya.

Untuk objek custom: hash dihitung setiap diperlukan, kecuali diimplementasikan dengan caching eksplisit.

### 14.4 Worst-Case Collision Degradation

Dalam kasus normal, load factor yang dijaga dan hash function yang baik memastikan rata-rata 1-2 probe per lookup. Tapi ada skenario patologis:

Jika semua key yang dimasukkan memiliki hash yang memetakan ke indeks yang sama, maka setiap lookup membutuhkan memeriksa semua entry — O(N). Ini bisa terjadi secara sengaja (serangan *hash collision attack*) atau secara kebetulan dengan hash function yang buruk.

Python telah mengimplementasikan **hash randomization** sejak Python 3.3: hash dari string (dan beberapa tipe lain) menggunakan seed acak yang berbeda setiap kali Python dijalankan. Ini membuat serangan hash collision jauh lebih sulit.

### 14.5 Cache Locality: Faktor Tersembunyi

Ada faktor performa yang sering diabaikan dalam analisis Big O: **cache locality** (lokalitas cache CPU).

CPU modern menggunakan *cache* — memori sangat cepat yang menyimpan salinan data yang baru-baru ini diakses. Mengakses data yang bersebelahan di memori (seperti iterasi array) sangat efisien karena CPU bisa memuat satu blok besar sekaligus.

List (contiguous array) memiliki *cache locality* yang sangat baik: elemen bersebelahan dalam memori, sehingga iterasi list sangat ramah cache.

Dict dan set (sparse array) memiliki cache locality yang lebih buruk: elemen tersebar dalam array yang jarang, dengan banyak slot kosong di antaranya. Ini bisa membuat iterasi dict terasa lebih lambat dari yang diperkirakan Big O saja.

> **Behavior Note:** Untuk operasi yang melibatkan iterasi *semua* elemen, list sering lebih cepat dari dict dalam praktik — meskipun keduanya O(N) secara teori — karena cache locality yang lebih baik. Big O adalah model; performa nyata juga dipengaruhi oleh hardware.

> **Mini Conclusion:** Di balik O(1) yang elegan, ada biaya tersembunyi: overhead memori dari slack capacity (list) dan sparse table (dict), biaya komputasi hash, dan potensi degradasi akibat collision. Cache locality membuat list lebih cepat dari dict untuk operasi iterasi meskipun keduanya O(N) secara teori. Performa nyata adalah fungsi dari banyak faktor di luar Big O.

---

# Part V — Design Trade-Offs

> *Tujuan Part V: Memahami bahwa setiap keputusan desain Python adalah pilihan sadar*
> *yang menukar satu hal untuk mendapatkan hal lain.*
> *Tidak ada struktur data yang sempurna — hanya struktur yang paling cocok untuk kebutuhan tertentu.*

---

## 15. Memory vs Speed

### 15.1 Ketegangan Fundamental

Ada satu ketegangan yang tak bisa dihindari dalam desain struktur data: **memori dan kecepatan sering bergerak berlawanan arah**.

Untuk lebih cepat, kamu perlu informasi tambahan (seperti slot kosong untuk hash table, atau kapasitas ekstra untuk list). Informasi tambahan memakan memori. Untuk hemat memori, kamu perlu menghilangkan informasi redundan — tapi itu sering memperlambat operasi.

Ini bukan kelemahan desainer — ini adalah batasan fisik yang harus dinavigasi.

### 15.2 Bagaimana List Menukar Memori untuk Kecepatan Append

List mempertahankan kapasitas ekstra — slot kosong yang belum terisi. Ini adalah memori yang "terbuang" dalam arti tidak menyimpan data apapun saat ini.

Imbalannya: `append` hampir selalu O(1). Tanpa slack capacity, setiap `append` harus me-resize — O(N) setiap saat. Kamu membayar sedikit memori ekstra untuk mendapatkan append yang sangat cepat.

```
List: Trade-off memori vs kecepatan append

Tanpa slack (selalu tepat-pas):
  append: selalu O(N) (resize setiap saat)
  memori: selalu minimal

Dengan slack (Python's choice):
  append: O(1) amortized
  memori: hingga ~12.5% lebih besar dari minimum
  
  ↑ Python memilih kecepatan dengan biaya memori kecil
```

### 15.3 Bagaimana Dict Menukar Memori untuk Kecepatan Lookup

Dict menjaga tabelnya di bawah 2/3 penuh. Artinya, setidaknya 1/3 dari seluruh bucket adalah slot kosong yang tidak menyimpan apapun.

Untuk dict dengan 100 entry, Python menggunakan setidaknya 128 bucket — menyia-nyiakan minimal 28 bucket. Untuk 1.000 entry: minimal 2.048 bucket, menyia-nyiakan sekitar 1.048 slot.

Imbalannya: lookup tetap O(1) average karena cukup banyak ruang kosong untuk menghindari collision yang parah. Kamu membayar banyak memori ekstra untuk mendapatkan lookup yang sangat cepat.

### 15.4 Set sebagai Dict yang Lebih Hemat

Set menggunakan mekanisme yang sama dengan dict, tapi hanya menyimpan key — tidak ada value. Untuk setiap data yang sama, set membutuhkan sekitar setengah memori dari dict (kasar-kasarnya — ada overhead lain yang tetap ada).

Ketika kamu hanya perlu "apakah X ada?" tanpa perlu menyimpan value terkait X, set adalah pilihan yang lebih hemat.

```python
# Perbandingan memori kasar:
import sys

data = list(range(1000))

d = {x: True for x in data}   # dict
s = set(data)                  # set

print(sys.getsizeof(d))   # lebih besar
print(sys.getsizeof(s))   # lebih kecil
# Set ~50-60% ukuran dict untuk data yang sama
```

> **Mini Conclusion:** List menukar memori untuk kecepatan append (slack capacity). Dict menukar memori untuk kecepatan lookup (sparse table). Set adalah versi dict yang lebih hemat ketika value tidak diperlukan. Setiap trade-off ini bukan kegagalan desain — ia adalah pilihan yang bijak untuk kasus penggunaan paling umum masing-masing struktur.

---

## 16. Mutability vs Stability

### 16.1 Mechanic yang Menjelaskan Hashability Rule

Di Volume 5, kita memahami *mengapa* mutable objects tidak bisa di-hash dari sudut pandang kontrak stabilitas. Di sini, kita melihatnya dari sudut pandang mekanis internal hash table.

Ketika kamu memasukkan pasangan (key, value) ke dict, Python:

1. Menghitung `hash(key)` = H
2. Memetakan H ke indeks bucket I
3. Menyimpan (H, key, value) di bucket I

Sekarang bayangkan key bisa bermutasi. Setelah mutasi, `hash(key_yang_berubah)` = H', di mana H' ≠ H. Ketika kamu mencoba mengambil kembali nilai dengan key yang sama, Python menghitung H' → memetakan ke indeks I' → mencari di I' → tidak menemukan apapun → `KeyError`.

Datanya masih ada, tersimpan rapi di bucket I. Tapi Python tidak akan pernah memeriksanya — karena hash yang berubah mengirim Python ke I' yang salah.

```
Bencana akibat mutable key (hipotetis):

Insert: key=[1,2], val=25
  hash([1,2]) = 777 → bucket [3]
  Simpan ([1,2], 25) di bucket [3]

key.append(3) → key sekarang [1,2,3]
  hash([1,2,3]) = 999 → bucket [7]

Lookup: d[[1,2,3]]
  Python menghitung hash([1,2,3]) = 999 → bucket [7]
  Bucket [7]: EMPTY → KeyError!

  Data di bucket [3] tidak pernah dicek. Tersesat selamanya.
```

### 16.2 Immutability sebagai Jaminan Internal

Objek immutable — `str`, `int`, `tuple` dengan elemen immutable — tidak bisa berubah nilainya. Ini berarti hash-nya tidak bisa berubah. Ini berarti bucket yang dihitung saat insert selalu sama dengan bucket yang dihitung saat lookup.

Ini bukan aturan yang dibuat-buat. Ini adalah *persyaratan mekanis* dari hash table yang benar.

Python menegakkan persyaratan ini dengan membuat semua mutable built-in types unhashable — bukan sebagai hukuman, tapi sebagai perlindungan: mencegah programmer tidak sengaja merusak hash table mereka sendiri.

### 16.3 Trade-Off yang Dipilih Python

Python bisa saja memilih pendekatan berbeda: izinkan mutable keys, tapi copy mereka saat dimasukkan ke dict. Ini akan membuat `{[1,2]: "value"}` bekerja.

Tapi ini menimbulkan pertanyaan lebih dalam: apakah `d[[1,2]] = 5` lalu `d[[1,2]]` — apakah kedua list itu "sama"? Jika Python meng-copy key, maka dua list yang berbeda tapi bernilai sama akan menjadi key yang berbeda. Ini mungkin menyebabkan lebih banyak kebingungan.

Python memilih konsistensi: jika nilai bisa berubah, ia tidak bisa menjadi key. Sederhana, prediktabel, aman.

> **Mini Conclusion:** Hashability rule adalah persyaratan mekanis dari hash table, bukan kebijakan sewenang-wenang. Mutable key merusak invariant "hash yang sama → bucket yang sama" yang harus dijaga. Python memilih untuk melarang mutable keys daripada meng-copy mereka, demi kesederhanaan dan prediktabilitas semantik.

---

## 17. Why Python Chose These Designs

### 17.1 Optimasi untuk Kasus Umum

Filosofi inti dari desain CPython adalah: **optimalkan untuk kasus yang paling sering terjadi dalam kode Python nyata**.

Untuk list, operasi yang paling umum adalah: tambah elemen di akhir (`append`), akses berdasarkan indeks, iterasi. Bukan: insert di tengah, atau pencarian nilai. Maka Python memilih dynamic array yang sangat efisien untuk operasi-operasi itu.

Untuk dict, operasi yang paling umum adalah: lookup berdasarkan key, insert, dan kadang delete. Hash table memberikan O(1) average untuk ketiganya.

### 17.2 Alternatif yang Tidak Dipilih dan Alasannya

**Mengapa list bukan linked list?** Linked list memberikan O(1) insert di posisi manapun (jika kamu sudah punya pointer ke lokasi). Tapi ia memberikan O(N) akses berdasarkan indeks dan cache locality yang buruk. Karena akses berdasarkan indeks dan iterasi adalah operasi list yang paling umum, Python memilih array kontinu.

**Mengapa dict bukan binary search tree?** BST memberikan O(log N) lookup yang terjamin dalam worst case (lebih baik dari O(N) worst case hash table). Tapi O(1) average hash table jauh lebih cepat dalam praktik. Dan BST membutuhkan key yang bisa diurutkan — hash table hanya membutuhkan key yang bisa di-hash, yang jauh lebih fleksibel (integer, string, tuple semuanya bisa).

**Mengapa dict bukan sorted array?** Sorted array memberikan O(log N) lookup melalui binary search. Tapi insert membutuhkan mempertahankan urutan — O(N). Hash table memberikan O(1) average untuk keduanya, dengan biaya memori yang lebih besar.

### 17.3 Filosofi Pragmatis

Python dirancang oleh Guido van Rossum dengan filosofi yang sangat pragmatis: Python harus berguna untuk *programmer nyata* dalam *tugas nyata*, bukan hanya elegan secara teori.

Pilihan dynamic array untuk list dan hash table untuk dict mencerminkan filosofi ini: berikan performa yang sangat baik untuk operasi yang paling umum, terima trade-off yang wajar untuk operasi yang kurang umum, dan jaga API tetap sederhana dan konsisten.

Hasilnya adalah bahasa yang terasa "cepat" untuk kode sehari-hari — bukan karena setiap operasi dioptimalkan secara absolut, tapi karena operasi yang paling sering digunakan memiliki performa yang sangat baik.

> **Mini Conclusion:** Python memilih dynamic array untuk list dan hash table untuk dict berdasarkan analisis realistis tentang operasi mana yang paling umum dalam kode Python nyata. Alternatif seperti linked list, BST, atau sorted array menawarkan trade-off yang berbeda — lebih baik dalam beberapa aspek, tapi lebih buruk untuk kasus yang paling sering terjadi.

---

# Part VI — Integration Layer

> *Tujuan Part VI: Merajut seluruh pemahaman dari Volume 4, 5, dan 6 menjadi satu gambaran utuh.*
> *Melihat di mana ujung-ujung volume ini bertemu dan membentuk satu ekosistem pemahaman.*

---

## 18. Connecting Volume 4, 5, and 6

### 18.1 Tiga Lapisan Pemahaman

Tiga volume terakhir dari seri ini membangun tiga lapisan pemahaman yang saling berlapis:

**Volume 4 — Behavioral Semantics** menjawab pertanyaan: *"Apa yang terjadi?"*

Ketika kamu memodifikasi list, side effect bisa menyebar ke semua variabel yang menunjuknya. Ketika kamu meneruskan list ke fungsi, fungsi itu bisa mengubah list asli. Objek mutable memiliki perilaku yang berbeda dari yang mungkin kamu perkirakan pertama kali.

**Volume 5 — Hashability & Identity** menjawab pertanyaan: *"Mengapa harus begitu?"*

Karena variabel adalah referensi ke objek, aliasing tak terhindarkan. Karena dict menggunakan hash untuk lookup, key harus stabil nilainya. Karena equality harus konsisten dengan hash, `__eq__` dan `__hash__` harus dipasangkan.

**Volume 6 — Internal Mechanics** menjawab pertanyaan: *"Bagaimana cara kerjanya?"*

Karena list adalah contiguous array yang pre-allocated, append amortized O(1) dan insert O(N). Karena dict adalah sparse array dengan slot yang diindeks oleh hash, lookup O(1) average. Karena hash table harus jarang, dict lebih boros memori dari list.

### 18.2 Rantai Kausal dari Bawah ke Atas

```
INTERNAL MECHANICS (V6)
  └── Dynamic array → append O(1) am., insert O(N)
  └── Hash table → lookup O(1) avg, memori boros
  └── Load factor → tabel harus jarang
        │
        ▼
HASHABILITY & IDENTITY (V5)
  └── Mutable key merusak hash table invariant
  └── Immutable → hash stabil → hashable
  └── Identity = alamat fisik (id())
        │
        ▼
BEHAVIORAL SEMANTICS (V4)
  └── Aliasing = dua label, satu alamat
  └── Mutation menyebar melalui referensi
  └── Immutable = tidak ada mutation yang menyebar
        │
        ▼
PERILAKU YANG KAMU AMATI (V1-V3)
  └── append() cepat
  └── dict lookup instan
  └── list tidak bisa jadi key dict
  └── tuple lebih aman untuk dibagikan
```

### 18.3 Satu Contoh yang Merangkum Ketiganya

```python
# Skenario yang menyentuh V4, V5, dan V6 sekaligus:

cache = {}

def simpan_hasil(tag_set, nilai):
    # V5: frozenset diperlukan karena set tidak hashable
    # V6: karena dict adalah hash table, key harus hashable
    kunci = frozenset(tag_set)
    cache[kunci] = nilai

def ambil_hasil(tag_set):
    kunci = frozenset(tag_set)
    return cache.get(kunci)   # V6: O(1) average, bukan O(N)

tag = {"python", "data", "tutorial"}
simpan_hasil(tag, "konten artikel A")

# V4: tag (set) bisa bermutasi tanpa memengaruhi kunci yang sudah tersimpan
tag.add("baru")

print(ambil_hasil({"python", "data", "tutorial"}))   # "konten artikel A"
print(ambil_hasil(tag))   # None — frozenset berbeda karena tag berubah
```

Dalam 15 baris ini:

- V6 menjelaskan mengapa `cache[kunci]` O(1): hash table
- V5 menjelaskan mengapa harus `frozenset`: stabilitas hash
- V4 menjelaskan mengapa mutasi `tag` tidak memengaruhi `kunci` yang sudah tersimpan: aliasing + immutability

---

## 19. Unified Mental Model

```
═══════════════════════════════════════════════════════════════
        UNIFIED MENTAL MODEL — PYTHON DATA STRUCTURES
═══════════════════════════════════════════════════════════════

LAYER FISIK (RAM):
  ┌─────────────────────────────────────────────────────────┐
  │  Setiap objek = blok memori beralamat                   │
  │  Setiap variabel = alamat objek (referensi)             │
  │  Aliasing = dua variabel, satu alamat                   │
  └─────────────────────────────────────────────────────────┘

LAYER STRUKTUR:
  ┌──────────────────────┐  ┌───────────────────────────────┐
  │   DYNAMIC ARRAY      │  │        HASH TABLE             │
  │   (list)             │  │        (dict, set)            │
  │                      │  │                               │
  │  [ref][ref][ref][  ] │  │  [ ][h,k,v][ ][dummy][h,k,v] │
  │   ─────size───         │  │  ──────────── sparse ───────│
  │   ───────capacity────  │  │   load factor < 2/3         │
  │                      │  │                               │
  │  ✓ O(1) index access │  │  ✓ O(1) avg key lookup       │
  │  ✓ O(1) am. append   │  │  ✓ O(1) avg insert/delete    │
  │  ✗ O(N) search       │  │  ✗ Memory boros              │
  │  ✗ O(N) mid-insert   │  │  ✗ Key harus hashable         │
  └──────────────────────┘  └───────────────────────────────┘

LAYER SEMANTIK:
  ┌─────────────────────────────────────────────────────────┐
  │  Mutable object → tidak hashable → tidak bisa jadi key  │
  │  Immutable object → hashable → bisa jadi key            │
  │  Mutation menyebar melalui aliasing                     │
  │  Reference count → manajemen memori otomatis            │
  └─────────────────────────────────────────────────────────┘

LAYER PERILAKU (yang terlihat oleh programmer):
  ┌─────────────────────────────────────────────────────────┐
  │  append() cepat (karena slack capacity)                 │
  │  insert(0, x) lambat (karena shift semua elemen)        │
  │  dict[key] instan (karena hash → indeks langsung)       │
  │  list tidak bisa jadi key (karena mutable → unstable)   │
  └─────────────────────────────────────────────────────────┘
```

Setiap perilaku yang terlihat di permukaan adalah konsekuensi yang bisa dilacak ke keputusan desain di lapisan yang lebih dalam. Tidak ada yang magic. Tidak ada yang sewenang-wenang.

---

## 20. Common Misconceptions Revisited

Setelah membangun pemahaman internal mechanics yang menyeluruh, kita siap untuk membongkar beberapa mitos yang paling umum beredar.

---

### Miskonsepsi 1: "List adalah Linked List"

**Realitanya:** List Python adalah *dynamic array* (contiguous array), bukan linked list.

Linked list menyimpan setiap elemen sebagai node terpisah yang saling terhubung via pointer. Setiap node bisa berada di mana saja dalam memori — tidak ada kewajiban bersebelahan.

```
Linked list (bukan Python list):
  [A] → [B] → [C] → [D] → None
   (tersebar di memori, terhubung via pointer)

Python list (dynamic array):
  ┌─────┬─────┬─────┬─────┐
  │ ref │ ref │ ref │ ref │  (bersebelahan dalam memori)
  └─────┴─────┴─────┴─────┘
```

Konsekuensi dari miskonsepsi ini: jika list adalah linked list, kamu mungkin mengira `insert(0, x)` cepat (O(1) di linked list jika punya head pointer). Faktanya O(N) karena Python harus menggeser semua elemen dalam array.

---

### Miskonsepsi 2: "Dict Mencari Data Satu per Satu"

**Realitanya:** Dict *tidak pernah* menelusuri entry satu per satu dalam kasus normal. Hash function menghitung indeks secara matematika, lompat langsung ke bucket yang tepat.

Orang yang percaya mitos ini mungkin takut menggunakan dict untuk data besar, atau mengira perlu "membantu" dict dengan menyortir key-nya. Keduanya tidak perlu — dict O(1) average *terlepas* dari jumlah entry.

---

### Miskonsepsi 3: "O(1) Berarti Satu Langkah"

**Realitanya:** O(1) berarti *konstan* — jumlah langkah tidak bergantung pada ukuran N. Bisa jadi 1 langkah, bisa jadi 100 langkah, bisa jadi 1000 langkah — asalkan jumlah itu tidak tumbuh seiring bertambahnya N.

`dict[key]` adalah O(1), tapi bisa membutuhkan beberapa probe jika ada collision. `list.append()` adalah O(1) amortized, tapi sesekali membutuhkan O(N) untuk resize. Keduanya tetap O(1) dalam klasifikasi — karena rata-ratanya konstan.

---

### Miskonsepsi 4: "Hash Selalu Unik"

**Realitanya:** Collision — dua objek berbeda menghasilkan hash yang sama — adalah *pasti terjadi* secara matematis (prinsip pigeonhole: infinitely many possible objects, finitely many hash values).

```python
# Collision bisa terjadi:
print(hash("a"))   # misalnya: 123456789
print(hash("b"))   # misalnya: 987654321
# Keduanya berbeda hash — normal.
# Tapi ada pasangan string lain yang bisa collision.

# Python menangani ini dengan collision resolution
# (open addressing + probing), bukan dengan menolak collision.
```

Yang penting bukan "hash selalu unik," melainkan "Python menangani collision dengan benar."

---

### Miskonsepsi 5: "Faster Selalu Lebih Baik"

**Realitanya:** Setiap struktur data menukar satu properti untuk mendapatkan properti lain. "Lebih cepat" selalu dalam konteks operasi tertentu — dan biasanya dibayar dengan sesuatu yang lain.

Dict lebih cepat dari list untuk lookup key → tapi lebih lambat untuk iterasi, dan lebih boros memori. Set lebih cepat dari list untuk membership test → tapi kehilangan urutan dan duplikat.

Pilih struktur berdasarkan *trade-off yang cocok untuk kebutuhanmu*, bukan berdasarkan "mana yang lebih cepat secara umum."

---

### Miskonsepsi 6: "Immutable Selalu Lebih Cepat"

**Realitanya:** Immutable tidak otomatis lebih cepat. Yang berbeda adalah *jenis* operasi yang masing-masing efisien.

Tuple (immutable) lebih hemat memori dari list (mutable) untuk data yang sama — karena tidak perlu menyimpan slack capacity. Iterasi tuple juga sedikit lebih cepat karena overhead lebih kecil.

Tapi membuat "perubahan" pada tuple membutuhkan membuat objek baru — lebih mahal dari mutasi in-place pada list untuk operasi yang memang ingin mengubah data.

```python
import sys

lst = [1, 2, 3, 4, 5]
tpl = (1, 2, 3, 4, 5)

print(sys.getsizeof(lst))   # lebih besar (ada overhead capacity)
print(sys.getsizeof(tpl))   # lebih kecil (ukuran fixed, no slack)
# Tuple memang lebih hemat — tapi bukan karena "immutable lebih cepat"
# Melainkan karena tidak perlu slack capacity
```

---

### Miskonsepsi 7: "Dictionary Tidak Pernah Collision"

**Realitanya:** Collision di dalam dict terjadi secara rutin dalam penggunaan normal. Yang tidak terlihat adalah bahwa Python menanganinya secara internal melalui probing — programmer tidak perlu melakukan apapun.

Collision baru menjadi masalah jika terjadi sangat banyak (karena hash function yang buruk atau serangan khusus), yang menyebabkan performa degradasi menuju O(N). Dalam penggunaan normal dengan built-in types dan hash function yang baik, collision sangat jarang dan ditangani dengan sangat efisien.

---

### Mental Model Common Misconceptions

```
Miskonsepsi          Realita                     Implikasi Praktis
─────────────────────────────────────────────────────────────────
"List = linked list" List = dynamic array         insert(0,x) adalah O(N)
"Dict = linear scan" Dict = hash table            lookup selalu O(1) avg
"O(1) = 1 langkah"   O(1) = konstan              ada constant factor
"Hash = unik"        Hash bisa collision          normal, ditangani Python
"Faster = selalu OK" Trade-off adalah nyata       pilih berdasarkan kebutuhan
"Immut = selalu fast" Immut = hemat memori        bukan "cepat" dalam semua aspek
"Dict no collision"  Collision rutin terjadi      transparan bagi programmer
```

---

## 21. Transition to the Next Volume

### 21.1 Apa yang Sudah Kita Bangun

Dalam enam volume ini, kita telah membangun pemahaman yang berlapis dan kohesif:

**Volume 1** memberi kita kerangka berpikir: 8 dimensi untuk mengevaluasi struktur data — organization style, mutability, ordering, indexing, uniqueness, hashability, internal mechanism, consequences.

**Volume 2** membedah setiap struktur konkret — list, tuple, dict, set, frozenset, string — dengan kerangka itu sebagai lensa.

**Volume 3** mengangkat abstraksi satu tingkat: Sequence, Mapping, dan Set-like bukan sekadar tipe, melainkan cara berpikir yang berbeda tentang data.

**Volume 4** memperlihatkan bahwa struktur data bukan wadah pasif — mereka adalah entitas dengan perilaku: mutasi, aliasing, side effects, copy semantics.

**Volume 5** mengungkap arsitektur pengenalan: bagaimana Python melacak identitas objek dan menggunakannya untuk lookup yang cepat melalui hashing.

**Volume 6** membuka mesin di baliknya: dynamic array, hash table, load factor, resizing, rehashing — semua keputusan desain konkret yang menghasilkan perilaku yang kita amati.

### 21.2 Pertanyaan yang Tersisa

Dengan fondasi internal mechanics ini, kita siap menghadapi pertanyaan-pertanyaan yang lebih dalam:

- Bagaimana struktur data standar ini bisa diperluas? Kapan kita perlu struktur yang lebih specialized?
- Bagaimana `collections.deque`, `OrderedDict`, `defaultdict`, dan `Counter` bekerja di atas fondasi yang sudah kita pahami?
- Bagaimana memilih struktur data yang tepat untuk algoritma yang lebih kompleks?
- Bagaimana menganalisis kompleksitas ruang dan waktu dari program secara keseluruhan, tidak hanya per operasi?
- Bagaimana cache locality memengaruhi performa nyata? Mengapa array kontinu (list) sering lebih cepat dalam iterasi dibanding hash table (dict/set) meskipun keduanya O(N) — dan bagaimana CPU cache bekerja di balik perbedaan itu?

### 21.3 Peta Konseptual Seri

```
Volume 1: Fondasi & Mental Model
    "Bagaimana seharusnya saya berpikir tentang struktur data?"
        │
        ▼
Volume 2: Concrete Data Structures
    "Apa saja struktur data konkret Python dan bagaimana mereka bekerja?"
        │
        ▼
Volume 3: Organization Styles
    "Bagaimana struktur data mengorganisasi informasi?"
        │
        ▼
Volume 4: Behavioral Semantics
    "Bagaimana objek berperilaku ketika berinteraksi?"
        │
        ▼
Volume 5: Hashability & Identity
    "Bagaimana Python mengenali dan menemukan objek?"
        │
        ▼
Volume 6: Internal Mechanics  ← KITA DI SINI
    "Mengapa perilaku itu muncul seperti yang kita lihat?"
        │
        ▼
Volume 7: ???
    "Kapan kita perlu melampaui struktur built-in?"
```

### 21.4 Kalimat Penutup

Setelah membaca Volume 6, kamu tidak lagi sekadar *menggunakan* struktur data Python. Kamu *memahami* mengapa mereka dirancang seperti itu.

Ketika kamu melihat `append()` dan tahu bahwa di baliknya ada array pre-allocated dengan slack capacity yang tumbuh secara amortized — kamu bisa menulis kode yang lebih baik.

Ketika kamu melihat dict lookup dan tahu bahwa di baliknya ada hash yang memetakan langsung ke bucket, dengan probing untuk collision — kamu tidak lagi takut menggunakan dict untuk data besar.

Ketika kamu melihat `TypeError: unhashable type: 'list'` — kamu tidak bingung. Kamu tahu persis mengapa: mutable key merusak invariant hash table.

Pemahaman bukan sekadar pengetahuan tentang "apa." Pemahaman adalah kemampuan menelusuri "mengapa" — dari perilaku yang terlihat di permukaan, ke keputusan desain di lapisan yang lebih dalam, ke prinsip fisik yang mendasarinya.

Dan itulah yang selalu menjadi tujuan seri ini.

---

> *"Sekarang saya tidak hanya tahu cara menggunakan struktur data Python.* *Saya memahami mengapa mereka dirancang seperti itu."*

---

*Volume ini adalah bagian dari seri Struktur Data Python.*
*[[V5 - Hashability & Identity|← Volume 5: Hashability & Identity]] | Volume 7: Extended Data Structures →*
# Beyond The Syntax — Python Data Structures
## Volume 5 — Hashability & Identity
### The Architecture of Recognition and Retrieval

---

> *Volume ini adalah bagian kelima dari seri Struktur Data Python.*
> *Volume 4 memperlihatkan bahwa objek Python adalah entitas hidup —*
> *mereka bermutasi, berbagi state, dan menyebar perubahan melalui referensi.*
> *Volume 5 ini menelusuri pertanyaan yang menggantung dari sana:*
> *jika objek bisa berubah kapan saja, bagaimana Python mengenalinya secara konsisten?*
> *Dan mengapa dict bisa menemukan data di antara jutaan entri hampir seketika,*
> *sementara list terasa begitu lambat?*
> *Jawabannya terletak pada dua sistem yang tersembunyi di balik setiap struktur data:*
> *Identity dan Hashing.*

---

## Daftar Isi

### Part I — The Identity System
1. [Identity vs Equality: Siapa vs Apa](#1-identity-vs-equality-siapa-vs-apa)
2. [The Reference Layer: Variabel sebagai Label](#2-the-reference-layer-variabel-sebagai-label)
3. [Identity in Practice: `is`, `id()`, dan Aliasing](#3-identity-in-practice-is-id-dan-aliasing)

### Part II — The Hashing Engine
4. [The Linear Lookup Problem](#4-the-linear-lookup-problem)
5. [Hash as Mathematical Fingerprint](#5-hash-as-mathematical-fingerprint)
6. [The Stability Imperative](#6-the-stability-imperative)

### Part III — Hashability & The Mutability Boundary
7. [What Makes an Object Hashable](#7-what-makes-an-object-hashable)
8. [List: The Unhashable Mutable](#8-list-the-unhashable-mutable)
9. [Tuple: The Conditionally Hashable](#9-tuple-the-conditionally-hashable)
10. [String, Integer, and the Immutable Guarantee](#10-string-integer-and-the-immutable-guarantee)

### Part IV — Hash Table Architecture
11. [How Dict Works: The Hash Table Revealed](#11-how-dict-works-the-hash-table-revealed)
12. [How Set Works: Dict Without Values](#12-how-set-works-dict-without-values)
13. [Hash Collision: When Fingerprints Collide](#13-hash-collision-when-fingerprints-collide)

### Part V — The Equality Contract
14. [The Inseparable Pair: `__eq__` and `__hash__`](#14-the-inseparable-pair-__eq__-and-__hash__)
15. [The Hash-Equality Law](#15-the-hash-equality-law)
16. [Python's Protective Defaults](#16-pythons-protective-defaults)

### Part VI — Mental Model & Integration
17. [The Recognition Architecture: A Unified View](#17-the-recognition-architecture-a-unified-view)
18. [Transition to Volume 6: Internal Mechanics](#18-transition-to-volume-6-internal-mechanics)

---

# Part I — The Identity System

> *Tujuan Part I: Memahami bagaimana Python membedakan "satu objek" dari "dua objek yang kebetulan sama."*
> *Sebelum kita bisa membahas bagaimana Python menemukan objek, kita harus memahami*
> *bagaimana Python mengenalinya terlebih dahulu.*

---

## 1. Identity vs Equality: Siapa vs Apa

### 1.1 The Fundamental Distinction

Ada dua pertanyaan yang terlihat mirip tapi menanyakan hal yang sangat berbeda tentang objek Python.

Pertama: *"Apakah isi dari dua objek ini identik?"*
Kedua: *"Apakah dua nama ini menunjuk pada satu objek yang sama di memori?"*

Python mengekspresikan perbedaan ini melalui dua operator yang sering disalahgunakan pemula: `==` dan `is`.

### 1.2 Equality: The Question of Value

Operator `==` bertanya tentang *nilai* — isi, konten, atau representasi data dari sebuah objek.

```python
a = [1, 2, 3]
b = [1, 2, 3]

print(a == b)  # True
```

Python memeriksa elemen demi elemen. Elemen pertama `a[0]` sama dengan `b[0]`. Elemen kedua sama. Ketiga sama. Karena setiap pasang elemen bernilai sama, Python menyimpulkan bahwa `a` dan `b` adalah *equal*.

Tapi equal bukan berarti *satu objek yang sama*.

### 1.3 Identity: The Question of Existence

Operator `is` bertanya tentang *identitas fisik* — apakah dua nama ini merujuk pada satu entitas yang persis sama di memori komputer?

```python
print(a is b)  # False
```

Mengapa `False`? Karena Python menciptakan dua objek list terpisah di dua lokasi memori yang berbeda. Mereka kebetulan berisi nilai yang identik, tapi secara fisik mereka adalah dua bangunan yang terpisah.

```
a ─────┐
       ▼
    [1, 2, 3]   ← objek pertama (di alamat 140123456789)

b ─────┐
       ▼
    [1, 2, 3]   ← objek kedua (di alamat 140123459999)
```

### 1.4 The `id()` Function: The Object's Fingerprint

Python menyediakan fungsi `id()` untuk membuat perbedaan ini terlihat secara konkret. Fungsi ini mengembalikan angka unik yang merepresentasikan alamat fisik objek di memori — semacam "nomor KTP" yang tidak bisa dipalsukan.

```python
print(id(a))  # 140123456789
print(id(b))  # 140123459999 — berbeda!
```

> **Behavior Note:**
> `is` pada dasarnya adalah shorthand untuk `id(a) == id(b)`. Jika identitas fisik sama, maka `is` mengembalikan `True`.

### 1.5 When Identity and Equality Diverge

Situasi yang paling membingungkan bagi pemula adalah ketika `==` dan `is` memberikan jawaban yang berbeda:

```python
# Dua list dengan isi identik
x = [1, 2]
y = [1, 2]

print(x == y)   # True  — isi sama
print(x is y)   # False — fisik berbeda
```

Ini bukan kontradiksi. Ini adalah dua dimensi evaluasi yang berbeda: satu menanyakan "apa", yang lain menanyakan "siapa".

> **Mini Conclusion:**
> `==` menanyakan kesamaan nilai. `is` menanyakan kesamaan identitas fisik. Dua objek bisa bernilai sama tanpa menjadi satu objek yang sama — dan memahami perbedaan ini adalah fondasi dari seluruh sistem referensi Python.

---

## 2. The Reference Layer: Variabel sebagai Label

### 2.1 The Box Myth

Sebelum kita melanjutkan, ada satu mitos yang harus dihancurkan: *variabel bukanlah kotak penyimpan data.*

Di bahasa seperti C, variabel memang terikat pada lokasi memori fisik yang tetap. Tapi di Python, variabel hanyalah **label nama yang diikatkan ke sebuah objek** yang sudah ada di memori.

```python
a = [1, 2, 3]
```

Python **tidak** membuat kotak bernama `a` lalu memasukkan list ke dalamnya. Yang dilakukan Python adalah:

1. Menciptakan objek list `[1, 2, 3]` di suatu tempat di memori.
2. Membuat label bernama `a` dan mengikatkannya ke objek tersebut.

```
a ─────┐
       ▼
    [1, 2, 3]
```

### 2.2 Assignment as Rebinding

Ketika kita menulis `b = a`, yang terjadi bukan "salin isi `a` ke `b`". Yang terjadi adalah: *"buat label `b` dan ikatkan ke objek yang sama dengan `a`."*

```python
b = a
```

```
a ─────┐
       │
       ▼
    [1, 2, 3]
       ▲
       │
b ─────┘
```

Fenomena ini disebut **aliasing** — satu objek memiliki banyak nama (alias).

> **Behavior Note:**
> Assignment (`=`) di Python tidak pernah menyalin data. Ia hanya membuat atau memindahkan label. Data itu sendiri tetap di tempatnya.

### 2.3 Why This Matters for Identity

Pemahaman bahwa variabel adalah label — bukan kotak — menjelaskan mengapa identity begitu penting. Ketika kita bertanya `a is b`, kita sebenarnya bertanya: *"apakah label `a` dan label `b` menunjuk ke objek yang sama di memori?"*

Jika ya, maka perubahan pada objek melalui salah satu label akan terlihat oleh label lain — karena mereka melihat objek yang persis sama.

> **Mini Conclusion:**
> Variabel adalah label, bukan kotak. Assignment hanya memindahkan label, bukan menyalin data. Aliasing adalah konsekuensi langsung dari sistem referensi ini — dan identity adalah cara Python melacak apakah dua label menunjuk ke satu objek yang sama.

---

## 3. Identity in Practice: `is`, `id()`, dan Aliasing

### 3.1 Aliasing and Shared State

Ketika aliasing bertemu dengan mutabilitas, kita mendapatkan fenomena yang dibahas di Volume 4: shared mutable state.

```python
a = [10, 20]
b = a  # aliasing — b menunjuk ke objek yang sama

b.append(30)

print(a)  # [10, 20, 30] — a ikut berubah!
print(a is b)  # True — masih satu objek yang sama
```

Identity menjelaskan *mengapa* ini terjadi: `a` dan `b` bukan dua list yang terpisah. Mereka adalah satu list dengan dua nama. Modifikasi melalui `b` adalah modifikasi pada objek yang sama yang juga dilihat oleh `a`.

### 3.2 Identity with Immutable Objects

Dengan objek immutable, aliasing tidak berbahaya — karena objeknya tidak bisa berubah sama sekali.

```python
x = "Python"
y = x  # aliasing

# y = y + "!"  — ini membuat string BARU, bukan mengubah yang lama
print(x)       # "Python" — tidak berubah
print(x is y)  # False — y sekarang menunjuk ke objek baru "Python!"
```

Identity tetap penting di sini, tapi konsekuensinya berbeda: immutable objects aman untuk di-alias karena tidak ada mutasi yang bisa menyebar.

### 3.3 The `None` Singleton

Ada satu kasus khusus di mana `is` adalah cara yang benar untuk membandingkan: dengan `None`.

```python
def proses(data=None):
    if data is None:
        data = []
    # ...
```

`None` di Python adalah *singleton* — hanya ada satu objek `None` di seluruh program. Karena itu, `data is None` adalah cara yang paling tepat dan paling cepat untuk memeriksa apakah sebuah variabel merujuk ke `None`.

> **Behavior Note:**
> Menggunakan `== None` akan bekerja dalam kebanyakan kasus, tapi `is None` adalah konvensi Python yang lebih tepat secara semantik dan lebih cepat secara performa.

### 3.4 Identity as Architectural Tool

Memahami identity bukan sekadar pengetahuan teoretis. Ia adalah alat arsitektural:

- **Debugging:** Ketika data berubah tanpa sebab yang jelas, cek identity. Apakah dua variabel yang seharusnya terpisah ternyata menunjuk ke objek yang sama?
- **Optimization:** `is` lebih cepat dari `==` karena tidak perlu memeriksa isi — hanya membandingkan alamat memori.
- **Design:** Memilih antara mutable dan immutable sering kali dimulai dari pertanyaan: "apakah saya ingin objek ini bisa di-alias dengan aman?"

> **Mini Conclusion:**
> Identity adalah fondasi dari seluruh sistem objek Python. Ia menjelaskan mengapa aliasing berbahaya untuk objek mutable tapi aman untuk objek immutable. Ia adalah alat debugging, optimasi, dan desain yang sering diabaikan — padahal ia bekerja di balik setiap operasi Python.

---

# Part II — The Hashing Engine

> *Tujuan Part II: Memahami mekanisme yang membuat pencarian di Python bisa secepat kilat.*
> *Kita akan mulai dari masalah yang dihadapi, lalu melihat bagaimana hashing*
> *menjadi solusi yang elegan — dan mengapa solusi itu bergantung pada stabilitas.*

---

## 4. The Linear Lookup Problem

### 4.1 The Cost of Ignorance

Bayangkan sebuah perpustakaan dengan satu juta buku yang disusun tanpa sistem apa pun — hanya ditumpuk secara acak di lantai. Untuk menemukan satu buku, petugas harus memeriksa setiap buku satu per satu dari ujung ke ujung.

Rata-rata, petugas harus memeriksa 500.000 buku.

Dalam notasi Big-O, ini adalah **O(n)** — waktu pencarian tumbuh secara linear seiring bertambahnya data. Setiap kali koleksi berlipat ganda, waktu pencarian juga berlipat ganda.

### 4.2 Python's Linear Structures

List di Python bekerja persis seperti perpustakaan tanpa sistem. Ketika kita mengecek keanggotaan:

```python
daftar = list(range(1_000_000))
print(999_999 in daftar)  # True, tapi lambat
```

Python tidak tahu di mana `999_999` berada. Ia harus memeriksa elemen demi elemen dari indeks 0:

```
Cek index 0: 0? Tidak.
Cek index 1: 1? Tidak.
Cek index 2: 2? Tidak.
...
Cek index 999_998: 999_998? Tidak.
Cek index 999_999: 999_999? Ya! Ditemukan.
```

Sejuta pemeriksaan untuk satu pencarian.

### 4.3 The Scaling Trap

Untuk data kecil, O(n) terasa cepat. Tapi skalanya brutal:

| Jumlah Elemen | Rata-rata Pemeriksaan |
|---------------|----------------------|
| 1,000         | 500                  |
| 1,000,000     | 500,000              |
| 1,000,000,000 | 500,000,000          |

Perpustakaan dengan satu miliar buku akan membutuhkan rata-rata 500 juta pemeriksaan per pencarian. Ini tidak skalabel.

### 4.4 The Need for a Shortcut

Masalahnya bukan bahwa list buruk — list tidak dirancang untuk pencarian cepat. List dirancang untuk urutan dan akses posisi. Tapi ada kelas masalah di mana pertanyaan "apakah X ada?" adalah yang paling sering diajukan.

Untuk masalah itu, kita butuh struktur yang berbeda — struktur yang tidak menelusuri data, tapi langsung "melompat" ke lokasinya.

> **Mini Conclusion:**
> Pencarian linear O(n) bekerja untuk data kecil tapi gagal total pada skala besar. List adalah struktur urutan, bukan struktur pencarian. Kebutuhan akan pencarian yang tidak bergantung pada ukuran data adalah fondasi dari seluruh mekanisme hashing.

---

## 5. Hash as Mathematical Fingerprint

### 5.1 The Genius of the Hash Function

Bayangkan perpustakaan kedua dengan sistem yang berbeda. Setiap judul buku dilewatkan ke sebuah "mesin formula" yang selalu menghasilkan nomor yang sama untuk judul yang sama.

- "Python Data Structures" → formula → `42`
- "Algorithms Unlocked" → formula → `157`
- "Clean Code" → formula → `89`

Nomor ini bukan posisi fisik buku di rak. Ini adalah *sidik jari matematis* — sebuah representasi ringkas yang unik (atau hampir unik) untuk setiap judul.

Ketika ada pengunjung mencari "Python Data Structures", petugas tidak berjalan dari rak ke rak. Ia menghitung formula, mendapat `42`, dan langsung melompat ke rak 42.

Satu langkah. Tidak peduli perpustakaan punya 10 atau 10 juta buku.

### 5.2 Hash in Python

Python mengekspos mekanisme ini melalui fungsi bawaan `hash()`:

```python
print(hash("alice"))     # 2547083320
print(hash(42))          # 42
print(hash((1, 2, 3)))   # 2528502973977326415
```

Fungsi `hash()` mengambil sebuah objek dan memerasnya melalui algoritma matematika untuk menghasilkan angka integer. Angka ini adalah *hash value* — sidik jari matematis dari objek tersebut.

### 5.3 The Determinism of Hash

Sifat paling penting dari hash adalah **determinisme**: nilai yang sama selalu menghasilkan hash yang sama, sepanjang waktu program berjalan.

```python
print(hash("alice") == hash("alice"))  # True, selalu
```

Tanpa determinisme, hash tidak bisa menjadi jalan pintas yang andal. Jika "alice" kadang menghasilkan `42` dan kadang `99`, maka sistem pencarian runtuh.

### 5.4 Hash is Not Identity

Ada tiga hal yang hash **bukan**:

1. **Bukan alamat memori.** Hash adalah sidik jari matematis, bukan lokasi fisik di RAM.
2. **Bukan enkripsi.** Hash tidak dirancang untuk disembunyikan — ia dirancang untuk diprediksi.
3. **Bukan ID unik.** Dua objek berbeda bisa memiliki hash yang sama (collision).

Hash adalah *shortcut untuk pencarian*, bukan identitas objek.

> **Mini Conclusion:**
> Hash adalah sidik jari matematis yang mengubah nilai objek menjadi angka deterministik. Ia adalah jalan pintas yang memungkinkan pencarian langsung ke lokasi — tanpa menelusuri data satu per satu. Tapi keefektifannya bergantung pada satu syarat mutlak: hash value tidak boleh berubah selama objek hidup.

---

## 6. The Stability Imperative

### 6.1 The Corruption Scenario

Untuk melihat mengapa stabilitas hash begitu krusial, bayangkan skenario hipotetis di mana Python mengizinkan objek mutable memiliki hash.

```python
# (Hipotetis — tidak valid di Python)
my_dict = {}
key = [1, 2]          # sebuah list
my_dict[key] = 25     # Python menghitung hash(key) = 42
                      # Python menyimpan 25 di "slot 42"
```

Beberapa saat kemudian:

```python
key.append(3)         # key sekarang [1, 2, 3]
                      # hash(key) berubah menjadi 99 (hipotetis)
```

Lalu kita mencoba mengambil data:

```python
print(my_dict[key])   # Python menghitung hash(key) = 99
                      # Python mencari di "slot 99" — kosong!
                      # Data di "slot 42" tidak akan pernah ditemukan
```

Data tidak hilang secara fisik. Tapi secara efektif, ia sudah *tersesat*. Hash yang berubah mematahkan jalan pintasnya sendiri.

### 6.2 The Stability Contract

Dari skenario di atas, muncul kontrak yang tidak bisa ditawar:

> **Hash Stability Contract:**
> Hash value dari sebuah objek harus tetap konstan dari awal objek diciptakan hingga ia dihancurkan. Jika hash berubah, seluruh sistem pencarian yang bergantung padanya akan rusak.

### 6.3 Immutability as the Only Guarantee

Bagaimana cara menjamin stabilitas hash? Cara paling kuat adalah membuat objek *tidak bisa berubah*.

Jika isi objek tidak bisa dimodifikasi, maka hash-nya pun tidak akan pernah berubah. Objek yang tidak bisa berubah — *immutable* — adalah satu-satunya jaminan stabilitas hash yang bisa diandalkan.

Ini bukan kebetulan. Ini adalah hubungan kausal:

```
Immutability → Stabilitas Isi → Stabilitas Hash → Keandalan Pencarian
```

> **Mini Conclusion:**
> Stabilitas hash adalah syarat mutlak. Jika hash berubah, data tersesat. Satu-satunya cara menjamin stabilitas adalah membuat objek immutable — karena objek yang tidak bisa berubah tidak akan pernah mematahkan jalan pintasnya sendiri. Inilah fondasi dari konsep hashability.

---

# Part III — Hashability & The Mutability Boundary

> *Tujuan Part III: Memahami konsep hashability secara konkret —*
> *mengapa beberapa objek bisa di-hash dan yang lain tidak,*
> *dan bagaimana Python menegakkan batasan ini.*

---

## 7. What Makes an Object Hashable

### 7.1 Definisi Hashable

Sebuah objek Python disebut **hashable** jika memenuhi dua syarat:

1. **Memiliki hash value yang stabil** — nilai yang dikembalikan oleh `hash()` tidak berubah selama objek hidup.
2. **Bisa dibandingkan dengan equality** — objek mendukung `==` untuk memeriksa kesamaan nilai.

Syarat pertama adalah yang paling krusial. Tanpa stabilitas hash, objek tidak bisa digunakan sebagai key `dict` atau elemen `set` — karena kedua struktur itu bergantung penuh pada hash untuk menemukan lokasi data.

### 7.2 The Hashability-Mutability Axis

Python membagi objeknya secara sistematis berdasarkan hubungan antara hashability dan mutability:

| Tipe | Mutable | Hashable | Bisa Jadi Key Dict? |
|------|---------|----------|---------------------|
| `list` | ✓ | ✗ | ✗ |
| `dict` | ✓ | ✗ | ✗ |
| `set` | ✓ | ✗ | ✗ |
| `tuple` | ✗ | ✓* | ✓* |
| `str` | ✗ | ✓ | ✓ |
| `int` | ✗ | ✓ | ✓ |
| `frozenset` | ✗ | ✓ | ✓ |

> * Tuple hashable hanya jika seluruh elemennya juga hashable.*

Pola yang terlihat jelas: **semua objek immutable adalah hashable. Semua objek mutable adalah unhashable.** Ini bukan kebetulan — ini adalah konsekuensi langsung dari kebutuhan stabilitas hash.

### 7.3 The `hash()` Gatekeeper

Fungsi `hash()` di Python bertindak sebagai *gatekeeper*. Ketika dipanggil dengan objek unhashable, ia langsung menolak:

```python
hash([1, 2, 3])
# TypeError: unhashable type: 'list'
```

Penolakan ini bukan keisengan. Ini adalah perlindungan — Python tidak mau membiarkan objek yang bisa berubah masuk ke sistem yang bergantung pada stabilitas.

> **Mini Conclusion:**
> Hashable berarti memiliki hash value yang stabil dan bisa dibandingkan dengan equality. Hubungan antara hashability dan mutability adalah hubungan kausal: immutable → hashable, mutable → unhashable. Fungsi `hash()` adalah gatekeeper yang menegakkan batasan ini.

---

## 8. List: The Unhashable Mutable

### 8.1 Why List Cannot Be Hashed

`list` adalah struktur data yang paling fleksibel di Python — dan fleksibilitas itulah yang membuatnya tidak bisa di-hash.

```python
my_list = [1, 2, 3]
my_list.append(4)       # isi berubah
my_list[0] = 99         # isi berubah lagi
# objek yang sama, tapi kondisi internal sudah berbeda total
```

Jika Python mengizinkan list memiliki hash, hash itu harus merepresentasikan isi list saat ini. Tapi isi itu bisa berubah kapan saja. Hash yang valid pagi ini bisa menjadi jalan pintas yang menyesatkan sore ini.

Python tidak mau mengambil risiko itu. Maka list dideklarasikan sebagai *unhashable* secara fundamental.

### 8.2 The Consequences of Unhashability

Konsekuensi langsung dari unhashability adalah larangan:

```python
d = {}
d[[1, 2]] = "nilai"     # TypeError: unhashable type: 'list'

s = set()
s.add([1, 2])            # TypeError: unhashable type: 'list'
```

List tidak bisa menjadi key `dict`, dan tidak bisa menjadi elemen `set`. Kedua struktur itu bergantung penuh pada hash table — dan list tidak bisa menjamin stabilitas hash yang mereka butuhkan.

### 8.3 The Protection Rationale

Ini bukan larangan yang dibuat secara sewenang-wenang. Ini adalah perlindungan dari kerusakan yang jauh lebih serius: data yang tersimpan tapi tidak bisa ditemukan kembali.

Bayangkan jika Python mengizinkan list sebagai key dict. Kamu menyimpan data dengan key `[1, 2]`. Lalu key itu berubah menjadi `[1, 2, 3]` — data yang tersimpan di bawah key lama sekarang tidak bisa diakses lagi. Kamu punya data yang "terkubur hidup-hidup" di dalam dict.

Python mencegah skenario ini dengan menolak list sejak awal.

> **Mini Conclusion:**
> List adalah unhashable karena mutable. Mutabilitasnya mematahkan syarat stabilitas hash. Konsekuensinya: list tidak bisa menjadi key dict atau elemen set. Ini bukan keterbatasan — ini adalah perlindungan arsitektural.

---

## 9. Tuple: The Conditionally Hashable

### 9.1 Tuple's Immutable Structure

`tuple` adalah *immutable* — setelah dibuat, struktur dan isinya terkunci. Python bisa menghitung hash tuple sekali dan mengandalkannya selamanya.

```python
my_tuple = (1, 2, 3)
print(hash(my_tuple))   # berhasil!
```

Ini membuka kapabilitas yang sangat berguna: tuple bisa menjadi key `dict`. Kita bisa menggunakan pasangan nilai, koordinat, atau kombinasi data apapun sebagai identifier:

```python
jarak = {
    ("Jakarta", "Bandung") : 150,
    ("Jakarta", "Surabaya"): 800,
    ("Bandung", "Cirebon") : 130,
}

print(jarak[("Jakarta", "Bandung")])  # 150
```

### 9.2 The Shallow Immutability Trap

Tapi ada nuansa penting yang sering menjadi jebakan.

Immutability tuple bersifat *shallow* — yang dikunci adalah referensi-referensi di dalamnya, bukan isi objek yang dirujuk. Jika sebuah tuple berisi list, list itu masih bisa bermutasi:

```python
sneaky = (1, 2, [3, 4])
sneaky[2].append(5)      # valid — list di dalam tuple bisa diubah
print(sneaky)            # (1, 2, [3, 4, 5])

sneaky[2] = [99]         # TypeError — referensi di tuple tidak bisa diganti
```

### 9.3 The Recursive Hashability Rule

Dan jika tuple berisi list, tuple itu pun gagal di-hash:

```python
hash((1, 2, [3, 4]))
# TypeError: unhashable type: 'list'
```

Python menelusuri isi tuple sampai ke lapisan terdalam. Jika ada satu elemen yang unhashable, keseluruhan tuple ikut gagal. Kestabilan hash tidak bisa bersifat setengah-setengah — seluruh rantai isi harus stabil.

> **Aturan:**
> Tuple hashable **hanya jika** seluruh isinya, sampai lapisan terdalam, juga hashable.

### 9.4 Tuple as Bridge Between Worlds

Tuple adalah satu-satunya built-in sequence yang bisa menjadi key dict. Ia menjadi jembatan antara dunia *berurutan* (sequence) dan dunia *berlabel* (mapping):

```python
# Koordinat sebagai key — tidak mungkin dengan list
cache = {}
cache[(3, 4)] = 5
cache[(5, 12)] = 13
```

Tanpa tuple, dunia mapping Python akan sangat terbatas.

> **Mini Conclusion:**
> Tuple adalah hashable karena immutable — tapi hashabilitynya bersyarat. Seluruh elemen harus hashable hingga ke lapisan terdalam. Tuple adalah jembatan antara sequence dan mapping, dan tanpanya, banyak pola desain Python tidak akan mungkin.

---

## 10. String, Integer, and the Immutable Guarantee

### 10.1 String: The Universal Key

`str` adalah sequence immutable yang paling umum digunakan sebagai key dict. Hampir semua dict yang merepresentasikan objek dengan atribut berlabel menggunakan string sebagai key:

```python
pengguna = {
    "nama": "Brata",
    "email": "brata@email.com",
    "kota": "Cirebon"
}
```

String immutable — karakternya tidak bisa diubah setelah dibuat. Setiap "modifikasi" menghasilkan string baru:

```python
teks = "Python"
# teks[0] = "J"  — TypeError!

teks_baru = "J" + teks[1:]   # string baru "Jython"
print(teks)                   # "Python" — asli tidak berubah
```

Karena immutable, string selalu hashable — dan karena selalu hashable, string selalu bisa menjadi key dict.

### 10.2 Integer: The Self-Hashing Number

`int` adalah tipe yang menarik: integer kecil biasanya hash ke dirinya sendiri.

```python
print(hash(42))   # 42
print(hash(7))    # 7
```

Ini bukan kebetulan. Integer adalah immutable — nilai `42` tidak akan pernah berubah menjadi `43`. Karena itu, integer bisa di-hash dengan sempurna, dan hash-nya bisa seefisien mungkin.

### 10.3 The Immutable → Hashable Pattern

Pola yang konsisten terlihat di semua tipe Python:

```
str    → immutable → hashable → key dict universal
tuple  → immutable → hashable* → key dict bersyarat
int    → immutable → hashable → key dict efisien
float  → immutable → hashable → key dict
frozenset → immutable → hashable → key dict

list   → mutable   → unhashable → tidak bisa jadi key
dict   → mutable   → unhashable → tidak bisa jadi key
set    → mutable   → unhashable → tidak bisa jadi key
```

> **Mini Conclusion:**
> String, integer, dan tipe immutable lainnya selalu hashable karena nilainya tidak bisa berubah. String adalah key dict yang paling universal. Integer bahkan hash ke dirinya sendiri — sebuah optimasi elegan yang mungkin karena immutabilitynya mutlak.

---

# Part IV — Hash Table Architecture

> *Tujuan Part IV: Memahami bagaimana dict dan set benar-benar bekerja di balik layar.*
> *Kita akan melihat hash table sebagai struktur data — bukan magic,*
> *tapi arsitektur yang elegan dengan trade-off yang jelas.*

---

## 11. How Dict Works: The Hash Table Revealed

### 11.1 The Hash Table Concept

Dict di Python dibangun di atas struktur data yang disebut **hash table** — sebuah array internal di mana posisi penyimpanan setiap data ditentukan oleh hash dari key-nya sendiri.

Bayangkan sebuah gudang dengan 1.000 laci bernomor 0 sampai 999. Setiap barang yang masuk dilewatkan ke mesin formula yang menghasilkan nomor laci. Barang itu kemudian disimpan di laci yang sesuai.

### 11.2 Insertion: From Key to Slot

Saat kita menulis:

```python
umur = {}
umur["alice"] = 25
```

Yang terjadi di balik layar:

1. Python menghitung `hash("alice")` → misalnya `89347589345`
2. Python memetakan hash ke indeks array internal → misalnya slot `42`
3. Python menyimpan pasangan `("alice", 25)` di slot `42`

```
hash("alice") = 89347589345
       │
       ▼
   slot 42
       │
       ▼
   ("alice", 25)
```

### 11.3 Lookup: The O(1) Leap

Saat kita mengakses:

```python
print(umur["alice"])
```

Yang terjadi:

1. Python menghitung `hash("alice")` → `89347589345` (sama seperti tadi)
2. Python memetakan ke slot `42` (sama seperti tadi)
3. Python langsung mengambil nilai dari slot `42` → `25`

Satu langkah. Tidak peduli dict berisi 10 atau 10 juta entri — karena hash selalu mengarah ke slot yang sama.

### 11.4 Why Dict Requires Hashable Keys

Dari mekanisme di atas, syarat hashable menjadi jelas:

- Jika key bisa berubah, hash-nya bisa berubah.
- Jika hash berubah, slot yang dihitung dari hash juga berubah.
- Jika slot berubah, data yang tersimpan di slot lama tidak akan pernah ditemukan lagi.

Python menegakkan syarat hashable dengan menolak mutable objects sebagai key — bukan karena keisengan, tapi karena hash table tidak bisa berfungsi tanpanya.

> **Mini Conclusion:**
> Dict adalah hash table yang memetakan key ke slot melalui hash. Insertion dan lookup adalah O(1) karena hash memberikan alamat langsung. Syarat hashable bukan keisengan — ia adalah fondasi mekanis dari seluruh sistem.

---

## 12. How Set Works: Dict Without Values

### 12.1 Set as Simplified Hash Table

`set` di Python bekerja dengan mekanisme yang persis sama dengan dict — pada dasarnya, set adalah "dict yang hanya menyimpan key, tanpa value".

```python
allowed_users = {"alice", "bob", "charlie"}
```

Di balik layar:
- `"alice"` → hash → slot X
- `"bob"` → hash → slot Y
- `"charlie"` → hash → slot Z

Tidak ada value yang disimpan — hanya keberadaan key itu sendiri yang menjadi informasi.

### 12.2 Membership Testing: The O(1) Miracle

Ketika mengeksekusi:

```python
if "bob" in allowed_users:
    print("Masuk!")
```

Python:
1. Menghitung `hash("bob")`
2. Melompat ke slot yang sesuai
3. Memeriksa apakah `"bob"` benar-benar ada di slot itu
4. Mengembalikan `True` atau `False`

Satu langkah. Hampir instan. Tidak peduli set berisi 10 atau 10 juta elemen.

### 12.3 The Performance Gap

Rasakan perbedaan konseptualnya:

```python
# Pencarian di list — O(n), linear
daftar = list(range(1_000_000))
print(999_999 in daftar)    # True, tapi Python memeriksa satu per satu

# Pencarian di set — O(1), konstan
himpunan = set(range(1_000_000))
print(999_999 in himpunan)   # True, hampir instan
```

Perbedaan ini bukan "optimasi kecil" — ini adalah perbedaan arsitektural. List menelusuri. Set melompat.

### 12.4 Why Set Requires Hashable Elements

Sama seperti dict, set membutuhkan elemen yang hashable:

```python
s = set()
s.add([1, 2])   # TypeError: unhashable type: 'list'
```

Tanpa stabilitas hash, set tidak bisa menentukan slot untuk menyimpan elemen — dan tanpa slot yang deterministik, membership testing tidak bisa O(1).

> **Mini Conclusion:**
> Set adalah hash table yang menyimpan key tanpa value. Membership testing O(1)-nya adalah konsekuensi langsung dari hashing — bukan magic. Elemen harus hashable karena set, seperti dict, adalah mesin yang berjalan di atas hash table.

---

## 13. Hash Collision: When Fingerprints Collide

### 13.1 The Mathematical Reality

Hash value adalah angka dalam rentang terbatas, tapi kemungkinan objek yang bisa dibuat tidak terbatas. Secara matematis, sangat mungkin dua objek berbeda menghasilkan hash yang sama.

Ini disebut **hash collision** — dua barang merebut satu slot yang sama.

```
hash("apple")  = 777  (fiktif)
hash("orange") = 777  (fiktif)

Slot 777: "apple" → collision → "orange" juga ingin masuk
```

### 13.2 Collision Resolution: Hash + Equality

Python tidak menyerah ketika collision terjadi. Ia menggunakan kombinasi hash dan equality:

1. Python menghitung hash dari key yang dicari.
2. Python melompat ke slot yang sesuai.
3. Jika slot kosong → key tidak ada.
4. Jika slot terisi → Python memeriksa apakah isi slot `==` key yang dicari.
   - Jika `True` → ditemukan!
   - Jika `False` → collision! Python mencari slot berikutnya.

```
Flow Lookup:
  hash(key) → slot
       │
       ▼
   slot kosong? → key tidak ada
       │
       ▼
   isi slot == key? (equality check)
       │
   ┌──┴──┐
   │     │
  True  False
   │     │
   ▼     ▼
Ditemukan!  Cari slot berikutnya
```

### 13.3 Why Equality Matters

Dari mekanisme collision resolution, muncul pemahaman baru: hash table membutuhkan **dua hal** dari setiap key:

1. **Hash yang stabil** — untuk menemukan lokasi yang tepat.
2. **Equality yang benar** — untuk mengonfirmasi bahwa yang ditemukan memang yang dicari.

Tanpa equality, collision tidak bisa diselesaikan. Dua objek dengan hash yang sama tapi nilai berbeda akan saling menimpa.

### 13.4 The Load Factor Trade-Off

Hash table membutuhkan slot kosong untuk mengakomodasi collision. Python menjaga *load factor* (rasio elemen terhadap slot) di bawah batas tertentu — biasanya sekitar 2/3.

Ketika load factor mendekati batas, Python mengalokasikan array baru yang lebih besar dan menyebar ulang semua elemen. Proses ini tersembunyi dari programmer, tapi ia menjelaskan mengapa dict lebih boros memori dari list: kamu membayar slot kosong sebagai asuransi collision.

> **Mini Conclusion:**
> Collision adalah realitas matematis dari hashing. Python mengatasinya dengan menggabungkan hash dan equality. Hash menemukan lokasi. Equality mengonfirmasi identitas. Dan load factor adalah trade-off: kamu membayar memori ekstra untuk menjaga kecepatan pencarian.

---

# Part V — The Equality Contract

> *Tujuan Part V: Memahami hubungan yang tidak bisa dipisahkan*
> *antara equality dan hash — dan bagaimana Python melindungi integritasnya.*

---

## 14. The Inseparable Pair: `__eq__` and `__hash__`

### 14.1 Magic Methods Behind the Scenes

Ketika kita menulis `a == b`, Python sebenarnya memanggil `a.__eq__(b)`. Ketika kita menulis `hash(a)`, Python memanggil `a.__hash__()`.

Kedua method ini adalah "magic methods" — method spesial yang mendefinisikan perilaku fundamental objek Python.

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __eq__(self, other):
        return self.x == other.x and self.y == other.y

    def __hash__(self):
        return hash((self.x, self.y))
```

### 14.2 The Contract Between Them

`__eq__` dan `__hash__` bukan method yang independen. Mereka terikat oleh kontrak yang tidak bisa dilanggar:

> **Hash-Equality Contract:**
> Jika `a == b` bernilai `True`, maka `hash(a)` wajib sama dengan `hash(b)`.

Kontrak ini bukan saran — ini adalah syarat mekanis. Jika dilanggar, hash table akan rusak.

### 14.3 Why the Contract Exists

Bayangkan jika kontrak dilanggar:

```python
# (Hipotetis — hasil buruk)
a = Point(1, 2)
b = Point(1, 2)

print(a == b)      # True — nilai sama
print(hash(a))     # 100
print(hash(b))     # 200 — hash berbeda! (kontrak dilanggar)

s = {a, b}
print(len(s))      # 2 — padahal seharusnya 1!
```

Dua objek yang dianggap "sama" oleh equality disimpan di slot yang berbeda karena hash berbeda. Set tidak bisa mengenali bahwa mereka sebenarnya duplikat.

> **Mini Conclusion:**
> `__eq__` dan `__hash__` adalah pasangan yang tidak bisa dipisahkan. Equality menentukan "apakah dua objek bernilai sama." Hash menentukan "di mana objek disimpan." Jika equality mengatakan "sama" tapi hash mengatakan "berbeda", seluruh sistem pencarian runtuh.

---

## 15. The Hash-Equality Law

### 15.1 The Law Stated

Dari kontrak di atas, muncul hukum yang harus dipahami oleh setiap programmer Python:

> **The Hash-Equality Law:**
> Jika `a == b` adalah `True`, maka `hash(a) == hash(b)` wajib `True`.
> Sebaliknya tidak berlaku: dua objek dengan hash sama belum tentu equal.

### 15.2 Python's Built-In Compliance

Python sendiri menjaga hukum ini dengan konsisten di semua tipe bawaan:

```python
# Integer 1 dan float 1.0 dianggap sama nilainya
print(1 == 1.0)             # True
print(hash(1) == hash(1.0)) # True — hash identik
```

Angka `1` dan `1.0` adalah tipe yang berbeda (`int` vs `float`), tapi karena nilai matematisnya sama, Python memastikan hash mereka pun sama — sehingga keduanya bisa saling "menemukan" satu sama lain di dalam dict atau set.

### 15.3 The Converse is Not True

Hukum hash-equality hanya berlaku satu arah:

```python
print(hash("apple") == hash("orange"))  # Mungkin False, mungkin True (collision)
print("apple" == "orange")               # Selalu False
```

Dua objek bisa memiliki hash yang sama (collision) tanpa bernilai sama. Itulah mengapa equality check masih diperlukan setelah hash menemukan slot.

> **Mini Conclusion:**
> Hash-equality law adalah tulang punggung integritas dict dan set. Python menjaganya di semua tipe bawaan. Tapi hukum ini hanya berlaku satu arah: equal → hash sama, tapi hash sama tidak menjamin equal.

---

## 16. Python's Protective Defaults

### 16.1 The Automatic Unhashing

Jika kamu menulis class sendiri dan mendefinisikan `__eq__` tanpa mendefinisikan `__hash__`, Python secara otomatis akan membuat class itu **unhashable**.

```python
class Person:
    def __init__(self, name):
        self.name = name

    def __eq__(self, other):
        return self.name == other.name
    # __hash__ tidak didefinisikan

p = Person("Alice")
hash(p)  # TypeError: unhashable type: 'Person'
```

Ini bukan kebetulan — ini adalah **perlindungan paksa**. Python tidak mau membiarkan situasi di mana equality sudah didefinisikan tapi hash dibiarkan tidak konsisten.

### 16.2 The Explicit Override

Jika kamu memang ingin classmu hashable, kamu harus mendefinisikan `__hash__` secara eksplisit — dan kamu bertanggung jawab untuk memastikan konsistensinya:

```python
class Person:
    def __init__(self, name):
        self.name = name

    def __eq__(self, other):
        return self.name == other.name

    def __hash__(self):
        return hash(self.name)

p = Person("Alice")
print(hash(p))  # Berhasil!
```

### 16.3 The Mutable Object Protection

Python juga melindungi dari skenario di mana objek mutable di-hash:

```python
class MutablePoint:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __eq__(self, other):
        return self.x == other.x and self.y == other.y

    def __hash__(self):
        return hash((self.x, self.y))

mp = MutablePoint(1, 2)
d = {mp: "value"}

mp.x = 100  # Objek berubah!
# Sekarang hash(mp) berbeda dari saat dimasukkan ke dict
# Data di dict tidak bisa ditemukan lagi
```

Meskipun Python mengizinkan definisi `__hash__` untuk objek mutable, melakukannya adalah **anti-pattern berbahaya**. Python tidak bisa mencegahnya secara otomatis, tapi programmer yang berpengalaman tahu untuk tidak melakukannya.

> **Mini Conclusion:**
> Python adalah bahasa yang protektif. Ia membuat class unhashable secara default jika equality didefinisikan tanpa hash. Ia memaksa programmer untuk secara eksplisit memikirkan hubungan antara equality dan hash. Tapi perlindungan terakhir — tidak meng-hash objek mutable — adalah tanggung jawab programmer.

---

# Part VI — Mental Model & Integration

> *Tujuan Part VI: Mengintegrasikan seluruh konsep dari Volume 5*
> *menjadi satu sistem pemahaman yang kohesif — dan menyiapkan transisi*
> *ke Volume 6, di mana kita akan membuka kotak hitam internal mechanics.*

---

## 17. The Recognition Architecture: A Unified View

### 17.1 The Three Pillars

Volume 5 membangun tiga pilar yang saling terkait:

**Pilar 1: Identity System**
- `is` vs `==`
- `id()` sebagai alamat fisik
- Aliasing dan shared state
- Variabel sebagai label, bukan kotak

**Pilar 2: Hashing Engine**
- Hash sebagai sidik jari matematis
- Determinisme dan stabilitas
- Linear lookup problem → hash shortcut
- Collision dan resolution

**Pilar 3: Equality Contract**
- `__eq__` dan `__hash__` sebagai pasangan
- Hash-equality law
- Python's protective defaults
- Mutable object anti-pattern

### 17.2 The Causal Chain

Ketiga pilar ini membentuk rantai kausal:

```
Identity System
      │
      ▼
  Variabel = Label
      │
      ▼
  Aliasing → Shared State
      │
      ▼
  Mutability → State Change
      │
      ▼
  Need for Stable Recognition
      │
      ▼
  Hashing Engine
      │
      ▼
  Hash Stability Requirement
      │
      ▼
  Immutability → Hashability
      │
      ▼
  Hash Table Architecture (dict, set)
      │
      ▼
  Equality Contract for Integrity
```

Setiap konsep muncul karena konsep sebelumnya membutuhkannya.

### 17.3 The Architectural Map

```
┌─────────────────────────────────────────────────────────────┐
│                    THE RECOGNITION ARCHITECTURE              │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  IDENTITY LAYER                                              │
│  ├─ is vs ==                                                 │
│  ├─ id()                                                     │
│  └─ Aliasing & Shared State                                  │
│                                                              │
│  HASHING LAYER                                               │
│  ├─ hash() as mathematical fingerprint                       │
│  ├─ Stability imperative                                     │
│  └─ Hash table (dict, set)                                   │
│                                                              │
│  CONTRACT LAYER                                              │
│  ├─ __eq__ & __hash__                                        │
│  ├─ Hash-equality law                                        │
│  └─ Python's protective defaults                             │
│                                                              │
│  BOUNDARY                                                    │
│  ├─ Immutable → Hashable → Key Dict / Element Set            │
│  └─ Mutable → Unhashable → No Key Dict / No Element Set      │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 17.4 The Decision Framework

Setelah Volume 5, memilih struktur data tidak lagi sekadar "yang familiar". Ia menjadi keputusan arsitektural yang didasarkan pada pemahaman mekanisme internal:

| Pertanyaan | Implikasi |
|-----------|-----------|
| Apakah data perlu di-hash? | Pilih immutable (tuple, str, frozenset) |
| Apakah data akan berubah? | Pilih mutable (list, dict, set), tapi terima unhashability |
| Apakah pencarian cepat penting? | Gunakan dict/set dengan key/elemen hashable |
| Apakah urutan penting? | Gunakan sequence, tapi terima O(n) lookup |

> **Mini Conclusion:**
> Identity, hashing, dan equality contract membentuk satu sistem pengenalan dan pengambilan yang terintegrasi. Memahami ketiganya sebagai satu kesatuan — bukan konsep terpisah — adalah kunci untuk benar-benar mengerti bagaimana Python mengelola objeknya.

---

## 18. Transition to Volume 6: Internal Mechanics

### 18.1 What We Have Seen

Di Volume 5, kita telah memahami:

- **Mengapa** beberapa objek harus hashable (stabilitas hash)
- **Bagaimana** hash table memberi kecepatan pada dict dan set
- **Mengapa** equality dan hash harus bergandengan
- **Bagaimana** Python melindungi integritas sistemnya

Tapi kita masih melihat semua ini dari *luar* — dari sisi perilaku.

### 18.2 What Lies Beneath

Ada lapisan yang lebih dalam yang belum kita sentuh:

- Bagaimana hash table *benar-benar* dibangun di dalam memori CPython?
- Bagaimana Python mengelola pertumbuhan array internal saat data bertambah?
- Bagaimana dynamic array di balik list menangani resizing?
- Bagaimana keputusan implementasi tingkat rendah ini — yang tidak pernah kita tulis sendiri — berdampak pada performa kode sehari-hari?

### 18.3 The Next Frontier

Di **Volume 6 — Internal Mechanics**, kita akan membuka kotak hitam itu. Kita akan melihat:

- **Memory layout** dari hash table dan dynamic array
- **Resizing strategies** dan trade-offnya
- **Cache locality** dan bagaimana ia mempengaruhi kecepatan
- **The CPython implementation** dari struktur data yang kita gunakan setiap hari

Bukan untuk menjadi ahli C — tapi untuk memahami *hukum fisika* yang mendikte batasan dan kekuatan struktur data Python.

> **Closing Thought:**
> Volume 5 telah memperlihatkan bahwa kecepatan dict dan set bukan magic — ia adalah konsekuensi dari arsitektur yang elegan. Tapi arsitektur itu dibangun dari batu bata konkret: byte memori, pointer, dan algoritma resizing. Di Volume 6, kita akan melihat batu bata itu satu per satu.

---

*Volume ini adalah bagian dari seri Struktur Data Python.*
*← Volume 4: Behavioral Semantics | Volume 6: Internal Mechanics →*


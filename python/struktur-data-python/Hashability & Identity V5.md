# Beyond The Syntax — Python Data Structures
## Volume 5 — Hashability & Identity

Di Volume 4, kita telah belajar bahwa *mutable object* (seperti `list` atau `dict`) memiliki perilaku internal yang bisa berubah sepanjang waktu. Kita bisa menambah, menghapus, atau memodifikasi isinya.

Sekarang, kita melihat konsekuensi lanjutan dari behavior tersebut. Jika sebuah object bisa berubah, bagaimana Python bisa melacak identitasnya dengan pasti? Bagaimana Python bisa dengan sekejap menemukan sebuah kunci di dalam `dict` yang berisi jutaan item, sementara pencarian di dalam `list` terasa sangat lambat?

Jawaban dari pertanyaan-pertanyaan ini bermuara pada dua konsep paling fundamental di balik layar Python: **Identity** dan **Hashing**.

Volume ini bukan tentang performa tingkat rendah, bukan pula tentang *memory allocator*. Volume ini adalah tentang membangun intuition yang tepat mengenai bagaimana Python "mengenali" object-object di dalam sistemnya, dan kenapa beberapa struktur data dirancang menjadi seperti yang Anda lihat sekarang.

Mari kita mulai perjalanan ini.

---

### 1. Identity vs Equality (Siapa vs Apa)

Bayangkan Anda dan teman Anda masing-masing membeli mobil Honda Civic merah di hari yang sama dengan spesifikasi yang persis sama. 

Secara fisik, apakah mobil Anda sama dengan mobil teman Anda? Tentu saja tidak. Kalian tidak bisa mengemudikan mobil yang sama secara bersamaan. Fisiknya ada dua.
Namun, secara spesifikasi dan nilai, apakah mobil kalian sama? Ya, sama persis.

Di Python, ini adalah akar dari perbedaan antara **Equality** dan **Identity**.

#### Kesamaan Nilai (`==`)

Operator `==` (equality) bertanya: *"Apakah isi atau nilai dari kedua object ini sama?"*

```python
a = [1, 2]
b = [1, 2]

print(a == b)  # Output: True
```

Python memeriksa ke dalam `a` dan `b`, membandingkan elemen pertama dan kedua. Karena isinya persis sama, ia menjawab `True`.

#### Identitas Fisik (`is`)

Operator `is` (identity) bertanya: *"Apakah kedua object ini adalah satu entitas fisik yang sama di dalam memori komputer?"*

```python
a = [1, 2]
b = [1, 2]

print(a is b)  # Output: False
```

Mengapa `False`? Karena Python membuat dua "kotak" list terpisah di memori. 

```text
a ─────┐
       ▼
    [1, 2]   (Kotak List Pertama)

b ─────┐
       ▼
    [1, 2]   (Kotak List Kedua)
```

Untuk membuktikan bahwa mereka ada di tempat yang berbeda, Python menyediakan fungsi `id()`. Fungsi `id()` mengembalikan semacam nomor KTP unik atau "alamat" dari sebuah object.

```python
print(id(a))  # Contoh: 140123456789
print(id(b))  # Contoh: 140123459999 (Nomornya berbeda!)
```

> **Behavior Note:**  
> `is` membandingkan *identity* (apakah `id()` mereka sama), bukan isi object. Sedangkan `==` membandingkan isi object.

Kenapa pemisahan ini penting? Karena di Volume 4 kita tahu list adalah mutable. Jika Anda mengubah `a`, Anda tidak ingin `b` ikut berubah tanpa sepengetahuan Anda. Memiliki identitas yang terpisah adalah cara Python menjaga object Anda tidak saling tumpang tindih.

---

### 2. Apa Itu Hashing?

Setelah memahami identitas, mari kita bahas tentang pencarian.

Bayangkan Anda bekerja di perpustakaan raksasa dengan jutaan buku. Jika ada pengunjung yang mencari buku "Python Data Structures", dan Anda harus membaca judul buku di rak satu per satu dari ujung ke ujung (pencarian linear), Anda akan kehabisan waktu.

Namun, bagaimana jika perpustakaan memiliki sistem jenius: setiap judul buku dilewatkan ke sebuah "mesin formula". Mesin itu akan selalu mengubah judul "Python Data Structures" menjadi sebuah angka pasti, misalnya `bucket 42`. 

Angka `42` ini langsung memberi tahu Anda: buku itu **pasti** ada di lorong 4, rak ke-2. Anda bisa langsung melompat ke sana tanpa membaca buku-buku lain.

Itulah inti dari **Hashing**.
Hashing adalah proses mengambil sebuah object (seperti string atau angka) dan memerasnya melalui algoritma matematika untuk menghasilkan sebuah angka unik yang mewakili object tersebut. Angka ini disebut *hash value*.

Fungsi bawaan `hash()` di Python melakukan hal ini. Ketika Anda memanggil `hash(obj)`, ia meminta *hash value* dari object tersebut.

```python
# Meminta hash value dari string
print(hash("alice"))  
# Output: 89347589345 (angka besar yang acak tapi konsisten)
```

```text
hash("alice")
      │
      ▼
  89347589345 (Bucket number)
      │
      ▼
   Lokasi Data
```

> **Common Misconception:**  
> Banyak pemula mengira *hash* adalah *memory address* (lokasi fisik di RAM) atau semacam *enkripsi*. Padahal bukan. Hash hanyalah "sidik jari matematis" yang dipakai sebagai *shortcut* atau jalan pintas untuk pencarian cepat.

---

### 3. Hashability: Kenapa Stabilitas Itu Krusial

Kita tahu hash memberi kita *shortcut* untuk menemukan barang. Namun, sistem ini memiliki satu syarat mutlak agar tidak runtuh: **Hash Consistency**.

Jika hari ini "alice" diubah menjadi angka `42`, maka besok, minggu depan, atau selama program itu hidup, "alice" **harus** tetap menghasilkan angka `42`. Jika nilainya berubah-ubah, Anda akan kehilangan jejak di mana data Anda disimpan (*lookup corruption*).

Di sinilah muncul konsep **Hashable**.

Sebuah object dikatakan *hashable* jika object tersebut memiliki *hash value* yang stabil dari awal ia diciptakan hingga ia dihancurkan.

Karena kestabilan ini sangat penting, object tersebut idealnya **tidak boleh berubah isinya (immutable)**.

```python
# String adalah immutable, ia hashable
hash("halo") 

# Angka adalah immutable, ia hashable
hash(100)
```

Lalu, bagaimana dengan object mutable yang bisa berubah sewaktu-waktu? Di sinilah aturan ketat Python berlaku.

---

### 4. Kenapa List Tidak Hashable?

Ini adalah salah satu pertanyaan wawancara paling sering ditanyakan: *Kenapa list tidak bisa di-hash?*

Mari kita gunakan analogi. Bayangkan *hash value* adalah label pengiriman paket. Label itu dicetak berdasarkan isi paket. 
Isi paket: "Baju". Label: "Rak B". 

Bagaimana jika paket Anda adalah sebuah kotak *list* yang isinya bisa dikeluarkan atau diganti kapan saja? 
Anda memasukkan "Baju", label mencetak "Rak B". Anda taruh paketnya di Rak B.
Lalu seseorang membuka paket tersebut, membuang "Baju", dan memasukkan "Batu".

Sekarang isinya "Batu". Jika dicetak ulang, labelnya seharusnya "Rak X". Tapi paket itu saat ini tertinggal dan tersesat di "Rak B". Sistem penyimpanan Anda rusak parah.

```text
Flow Mental Model:
List bisa diubah (Mutable)
    ↓
Jika list diizinkan punya hash, hash-nya akan berubah mengikuti isi
    ↓
Jika dipakai sebagai Dictionary Key, lokasi tersesat
    ↓
Dictionary rusak
    ↓
Python melarang keras list menjadi hashable
```

Mari kita buktikan di Python:

```python
my_list = [1, 2, 3]

# Meminta hash dari list
hash(my_list)
# TypeError: unhashable type: 'list'
```

Python langsung menolak. Python tidak akan mengambil risiko merusak integritas *dictionary* atau *set*. Jika struktur internal sebuah object bisa bermutasi, ia **unhashable**.

---

### 5. Kenapa Tuple Bisa Hashable?

Di Volume 2, kita mempelajari `tuple`. Secara sekilas, tuple sangat mirip dengan list, namun ia **immutable** (tidak bisa diubah).

Karena Python tahu bahwa ukuran dan elemen di dalam tuple telah "terkunci" dan tidak akan berubah, Python menjamin bahwa *hash value* dari tuple akan stabil.

```python
my_tuple = (1, 2, 3)

# Meminta hash dari tuple
print(hash(my_tuple))  # Berhasil!
```

**Namun, ada jebakan besar di sini!**

> **Common Misconception:**  
> Immutable tidak otomatis berarti selalu hashable.

Ingat bahwa tuple bisa berisi object apa saja, termasuk list! Jika sebuah tuple berisi object yang mutable, maka immutabilitas tuple tersebut terkompromi. Wadahnya (tuple) memang tidak bisa diganti, tetapi *isinya* bisa.

```python
# Tuple ini mengikat sebuah list
sneaky_tuple = (1, 2, [3, 4])

# Coba kita hash
hash(sneaky_tuple)
# TypeError: unhashable type: 'list'
```

> **Behavior Note:**  
> Tuple hanya hashable jika **seluruh isinya** juga hashable hingga ke akar-akarnya. Jika ada satu saja elemen yang bisa bermutasi, stabilitas hash-nya batal.

---

### 6. Bagaimana Dict dan Set Bekerja Menggunakan Hashing

Sekarang kita mengerti mengapa Python sangat cerewet soal hashability. Kenapa ini begitu krusial? Karena dua struktur data terkuat di Python bergantung penuh pada konsep ini: **Dictionary** (`dict`) dan **Set** (`set`).

#### Dictionary (Key Lookup)
Sebuah `dict` beroperasi menggunakan struktur data yang disebut **Hash Table**.

Ketika Anda menambahkan data:
```python
user_ages = {"alice": 25}
```

Python mengambil key `"alice"`, meminta `hash("alice")`, dan mengubahnya menjadi lokasi spesifik di memori (misalnya *bucket 5*). Python menaruh angka `25` di bucket tersebut.

Ketika Anda memanggil data:
```python
print(user_ages["alice"])
```

Python tidak mencari dari daftar awal. Ia langsung melakukan *hash* pada `"alice"`, mendapatkan *bucket 5*, dan langsung mengambil `25`. Pencarian terjadi seketika (O(1)), tidak peduli dictionary Anda berisi 10 atau 1 juta data.

Ini menjelaskan aturan paling kaku dari `dict`: **Key dari sebuah dict HARUS hashable.**

#### Set (Membership Testing)
`set` adalah versi lebih sederhana dari dictionary. Jika `dict` adalah kumpulan pasang "Key dan Value", maka `set` hanyalah "Kumpulan Key" tanpa value.

```python
allowed_users = {"alice", "bob", "charlie"}

if "bob" in allowed_users:
    print("Masuk!")
```

Ketika mengeksekusi `"bob" in allowed_users`, Python men-hash `"bob"`, melompat ke lokasinya, dan memastikan ia ada di sana.
Sama seperti `dict` key, **semua anggota `set` HARUS hashable**. List tidak boleh masuk ke dalam set.

---

### 7. Hash Collision: Saat Tabrakan Terjadi

Di dunia nyata, memori komputer Anda terbatas, namun kemungkinan object yang bisa Anda buat tidak terbatas.

Secara matematis, sangat mungkin dua object yang berbeda isinya menghasilkan *hash value* yang persis sama. Ini disebut **Hash Collision** (Tabrakan Hash).

Misalnya (sebagai ilustrasi fiktif):
`hash("apple")` = `777`
`hash("orange")` = `777`

Jika `dict` hanya mengandalkan hash, maka saat Anda menyimpan `"orange"`, nilai dari `"apple"` akan tertimpa. 

Bagaimana Python mengatasi ini? Di sinilah **Equality** (kesamaan) kembali bermain!

Saat Python mencari `"orange"` dan melihat *bucket 777* sudah ditempati, Python akan mengambil object di *bucket 777* tersebut (yaitu `"apple"`) dan bertanya menggunakan equality:
*"Hei, apakah 'apple' `==` 'orange'?"*

Karena jawabannya `False`, Python menyadari ini adalah tabrakan (dua barang berebut rak yang sama). Python kemudian akan mencari rak kosong terdekat lainnya untuk `"orange"`.

```text
Flow Mental Model (Lookup):
Dapatkan hash dari Key
    ↓
Lompat ke Bucket
    ↓
Apakah bucket kosong? → Data tidak ada
    ↓
Jika ada isi, apakah isinya == Key? (Equality Check)
    ↓
Jika True → Ditemukan!
Jika False → Terjadi tabrakan, cek bucket berikutnya
```

Ini membuktikan bahwa pencarian `dict` dan `set` **membutuhkan perpaduan hash yang stabil dan equality yang jelas**.

---

### 8. `__hash__` dan `__eq__` (Keterkaitan Erat)

Sebagai penutup, jika kelak Anda membuat tipe object sendiri menggunakan *Class*, Python mengizinkan Anda mendefinisikan *behavior* hashing ini melalui magic methods:
*   `__eq__(self, other)`: Menentukan aturan kapan dua object dianggap sama nilainya.
*   `__hash__(self)`: Menentukan algoritma hash object Anda.

Ada satu hukum tidak tertulis yang dijaga ketat oleh Python:
**Jika dua object bernilai sama (`a == b` adalah True), maka mereka WAJIB memiliki hash yang sama.**

Jika Anda mendefinisikan `__eq__` di class Anda tapi lupa mendefinisikan `__hash__`, Python secara otomatis akan membuat class Anda menjadi *unhashable*. Ini adalah perlindungan keamanan agar Anda tidak membuat object yang terlihat identik tapi bisa tersesat di beda bucket saat dimasukkan ke dalam `dict`.

---

### Mental Model Akhir

Setelah menelusuri Volume 5, inilah intisari dari cara berpikir seorang engineer Python terkait *Identity* dan *Hashability*:

1.  **Identity bukan sekadar Equality:** Dua barang kembar tidak sama dengan satu barang yang sama. (`is` vs `==`).
2.  **Hashing adalah Jalan Pintas:** Hash adalah sidik jari matematis yang memberi tahu lokasi instan suatu data tanpa pencarian linear panjang.
3.  **Hashability butuh Kestabilan (Immutability):** Jika sebuah object bisa bermutasi, sidik jarinya bisa berubah. Object mutable tidak boleh memiliki hash.
4.  **Dictionary dan Set adalah Mahakarya Hashing:** Kecepatan kilat `dict` dan `set` sepenuhnya bergantung pada keandalan *Hash Table*, yang mensyaratkan setiap elemen atau kunci bersifat *hashable*.

Di Volume 4, Anda melihat perilaku object yang bermutasi. Di Volume 5, Anda memahami mengapa mutasi memiliki batasan dan konsekuensi, terutama terkait stabilitas identitas di ekosistem Python.

Di volume selanjutnya (Volume 6), kita akan menyingkap lebih dalam lagi: Bagaimana struktur-struktur memori, *Hash Table*, dan mekanika tingkat bawah (internal mechanics) Python benar-benar bekerja secara harfiah.

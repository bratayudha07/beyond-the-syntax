# Struktur Data Python — Volume 4

## Behavior, Identity, and Object Semantics

---

_Di volume-volume sebelumnya, kita membangun cara melihat struktur data — dari bentuk konkretnya, cara mereka mengorganisasi informasi, sampai klasifikasi konseptual yang menghubungkan semuanya ke dalam satu gambaran utuh._

_Volume 4 masuk ke lapisan yang berbeda. Di sini, pertanyaannya tidak lagi tentang bentuk atau organisasi, melainkan tentang bagaimana Python benar-benar memperlakukan objek yang ia pegang — bagaimana mereka dirujuk, diubah, dibagikan, dan disalin._

_Ini adalah titik di mana "struktur data" berubah menjadi "pemahaman tentang sistem objek Python." Sebuah sistem yang, begitu dipahami, tidak hanya masuk akal — ia terasa tak terelakkan._

---

## Sebelum Kita Mulai

Ada kelas bug Python yang tidak terlihat seperti bug. Program berjalan. Tidak ada error. Tidak ada traceback. Tapi datanya salah — di tempat yang tidak pernah kamu sentuh.

```python
a = [1, 2, 3]
b = a
b.append(99)
print(a)   # [1, 2, 3, 99] — tapi kamu tidak menyentuh a
```

```python
def tambah(item, wadah=[]):
    wadah.append(item)
    return wadah

print(tambah("apel"))    # ["apel"]
print(tambah("mangga"))  # ["apel", "mangga"] — dari mana "apel" datang?
```

```python
data = ([1, 2], [3, 4])   # tuple, seharusnya immutable
data[0].append(99)
print(data)               # ([1, 2, 99], [3, 4])
```

Tidak satu pun dari ini adalah bug Python. Semua adalah konsekuensi yang sangat logis dari cara Python merancang sistem objeknya. Dan ketika kamu benar-benar memahami sistem itu, semua behavior ini tidak hanya masuk akal — mereka terasa tak terhindarkan, karena mereka mengalir secara alami dari prinsip-prinsip dasarnya.

---

## Daftar Isi

1. [Object Identity — Nama Bukan Kotak](#%20Bab%201:%20Object%20Identity%20%20-Nama%20Bukan%20Kotak)
2. [Mutability — Stabilitas State sebagai Prinsip Desain](#Bab%202%20Mutability%20%E2%80%94%20Stabilitas%20State%20sebagai%20Prinsip%20Desain)
3. [Hashing dan Stabilitas — Prasyarat yang Tidak Bisa Ditawar](#bab-3-hashing-dan-stabilitas--prasyarat-yang-tidak-bisa-ditawar)
4. [Copying dan Nested Structures — Di Mana Reference Bersembunyi](https://claude.ai/chat/6de16075-fc0c-45d5-b8cf-e560abde1f5d#bab-4-copying-dan-nested-structures--di-mana-reference-bersembunyi)
5. [Behavior Emergence — Ketika Semua Sistem Bertemu](https://claude.ai/chat/6de16075-fc0c-45d5-b8cf-e560abde1f5d#bab-5-behavior-emergence--ketika-semua-sistem-bertemu)
6. [Mental Model Akhir](https://claude.ai/chat/6de16075-fc0c-45d5-b8cf-e560abde1f5d#bab-6-mental-model-akhir)

---

# Bab 1: Object Identity — Nama Bukan Kotak

## Masalah Fundamental

Hampir semua orang yang belajar pemrograman pertama kali menerima model yang sama: _variabel adalah kotak_. `x = 5` artinya menaruh angka `5` ke dalam kotak bernama `x`. `y = x` artinya menyalin isi kotak `x` ke kotak baru bernama `y`.

Model kotak ini cukup berguna di awal. Ia memvisualisasikan memori sebagai lemari penyimpanan — rapi, intuitif, mudah dipahami. Tapi ia salah secara fundamental di Python, dan kesalahannya tidak langsung terasa. Ia bersembunyi di balik sintaks yang tampak familiar, menunggu sampai program cukup kompleks sehingga bug mulai muncul dari arah yang tidak terduga.

## Intuisi: Sticky Note, Bukan Kotak

Python tidak membuat kotak untuk setiap variabel. Model yang lebih akurat adalah _sticky note_.

Ketika kamu menulis `x = [1, 2, 3]`, Python melakukan dua hal:

1. **Menciptakan** objek list `[1, 2, 3]` di suatu tempat di memori.
2. **Menempelkan** sticky note bertuliskan `x` pada objek itu.

Perhatikan: sticky note `x` tidak _menyimpan_ list tersebut — ia _menunjuk_ ke list itu. List hidup di memorinya sendiri; `x` hanyalah sebuah label yang ditempelkan padanya.

Ketika kamu menulis `y = x`, Python tidak membuat salinan list. Ia tidak mengalokasikan memori baru. Ia hanya membuat **sticky note baru bertuliskan `y`** dan menempelkannya pada _objek yang sama_. Sekarang ada dua nama untuk satu objek yang sama.

---

## `id()` dan Object Identity

Setiap objek Python punya _identitas_ — sebuah integer unik yang tidak dimiliki objek lain selama objek itu hidup. Fungsi `id()` memperlihatkan identitas ini.

```python
a = [1, 2, 3]
b = [1, 2, 3]   # list baru dengan konten yang sama
c = a            # sticky note baru pada objek yang sama

print(id(a))   # misal: 140234001
print(id(b))   # misal: 140234888 — BERBEDA: dua objek berbeda
print(id(c))   # misal: 140234001 — SAMA dengan a: satu objek, dua nama
```

`a` dan `b` memiliki nilai yang sama persis — `[1, 2, 3]` — tetapi identitasnya berbeda. Dua objek terpisah di memori, kebetulan berisi konten yang identik. `a` dan `c`, sebaliknya, adalah dua nama untuk _satu_ objek yang sama.

---

## `==` vs `is` — Dua Pertanyaan yang Berbeda

`==` membandingkan _nilai_. `is` membandingkan _identitas_. Keduanya menjawab pertanyaan yang berbeda, dan mencampurkannya adalah sumber kebingungan yang umum.

```python
a = [1, 2, 3]
b = [1, 2, 3]
c = a

print(a == b)   # True  — konten identik
print(a == c)   # True  — konten identik
print(a is b)   # False — dua objek berbeda
print(a is c)   # True  — satu objek, dua nama
```

`a == b` bertanya: _"Apakah nilai kalian sama?"_ — seperti menanyakan apakah dua orang memiliki nama yang identik. `a is b` bertanya: _"Apakah kalian satu entitas yang sama?"_ — seperti menanyakan apakah dua nama merujuk ke satu orang yang sama.

**Implikasi praktis:** gunakan `is` untuk mengecek `None`, bukan `==`. `None` adalah singleton di Python — ada tepat satu objek `None` dalam seluruh program. `if x is None` bukan hanya lebih idiomatik, tapi juga secara teknis lebih tepat: kamu tidak membandingkan nilai, kamu menanyakan identitas.

---

## Aliasing — Konsekuensi dari Reference Model

_Aliasing_ adalah kondisi di mana satu objek memiliki lebih dari satu nama. Ini bukan anomali — ini adalah konsekuensi alami dari model referensi Python. Tapi konsekuensinya dalam, dan semakin besar objeknya, semakin besar dampak yang tidak terduga.

```python
daftar_asli = ["senin", "selasa", "rabu"]
alias       = daftar_asli          # dua nama, satu objek

alias.append("kamis")

print(daftar_asli)          # ["senin", "selasa", "rabu", "kamis"]
print(daftar_asli is alias) # True — memang satu objek
```

Ini bukan "daftar_asli ikut berubah" — ini adalah _"daftar_asli tidak pernah berbeda dari aliаs sejak awal."_ Keduanya adalah nama untuk objek yang persis sama. Tidak ada "milik daftar_asli" yang terpisah dari "milik aliаs."

Aliasing menjadi masalah ketika dua nama berbagi objek yang sama _tanpa kamu sadari_. Paling sering terjadi saat melempar objek mutable ke fungsi:

```python
def bersihkan(koleksi):
    koleksi.clear()
    koleksi.append("sudah bersih")

data = [1, 2, 3, 4, 5]
bersihkan(data)
print(data)   # ["sudah bersih"] — data asli ikut berubah
```

Fungsi tidak menerima _salinan_ list. Ia menerima _referensi ke list yang sama_. Modifikasi di dalam fungsi langsung mempengaruhi objek aslinya.

---

## Behavioral Observation

Mari kita verifikasi pemahaman ini dengan eksperimen langsung:

```python
# Eksperimen: assignment ulang vs modifikasi
x = [1, 2, 3]
id_awal = id(x)

x = x + [4]               # membuat LIST BARU, x diarahkan ke sana
print(id(x) == id_awal)   # False — objek baru

y = [1, 2, 3]
id_awal_y = id(y)
y.append(4)               # modifikasi in-place — objek yang sama
print(id(y) == id_awal_y) # True — tidak ada objek baru

# Fungsi menerima referensi, bukan salinan
def tes(lst):
    print(f"  id di dalam fungsi: {id(lst)}")
    lst.append(99)

z = [1, 2, 3]
print(f"id sebelum: {id(z)}")
tes(z)
print(f"id setelah: {id(z)}")
print(z)   # [1, 2, 3, 99]
# Ketiga baris id() menunjukkan angka yang sama
```

---

## Konsekuensi

Aliasing bukan hal yang "salah" — ia adalah fitur desain yang sangat efisien. Kamu bisa melempar objek besar ke fungsi tanpa biaya menyalin. Fungsi memodifikasi objek yang sama, dan hasilnya langsung terlihat di pemanggil. Tidak ada duplikasi memori, tidak ada overhead.

Tapi efisiensi ini datang dengan harga: dua bagian program yang tampak tidak terhubung ternyata berbagi objek yang sama. Perubahan di satu tempat muncul sebagai gejala misterius di tempat yang tidak kamu sentuh.

Setiap kali kamu menulis `b = a` untuk objek mutable, tanyakan: _apakah ini aliasing yang disengaja, atau saya sebenarnya butuh dua objek terpisah?_ Jawabannya menentukan apakah baris berikutnya akan menjadi clean code atau debugging session yang panjang.

---

# Bab 2: Mutability — Stabilitas State sebagai Prinsip Desain

## Masalah Fundamental

Volume-volume sebelumnya mendefinisikan mutability secara fungsional: objek mutable bisa diubah, objek immutable tidak. Definisi itu benar, tapi dangkal untuk level ini. Ia menjawab _apa_ yang terjadi, tapi tidak _mengapa_.

Pertanyaan yang lebih dalam: _mengapa_ beberapa objek dibuat immutable? Python bisa saja membuat string mutable secara teknis — tidak ada hambatan bahasa yang mencegahnya. Tapi ada alasan sistemik yang mendalam mengapa immutability string adalah keputusan desain yang sangat baik. Dan memahaminya bukan hanya menambah wawasan — ia mengubah cara kamu melihat seluruh ekosistem Python.

## Intuisi: Nilai vs Tempat

Bayangkan dua jenis entitas.

Yang pertama adalah angka `42`. Dimanapun `42` muncul, ia adalah `42`. Tidak ada operasi di dunia yang bisa mengubah `42` menjadi `43` — yang bisa kamu lakukan hanyalah berhenti menunjuk ke `42` dan mulai menunjuk ke `43`. Angka `42` tetap `42` selamanya, tidak tersentuh, tidak berubah. Inilah immutable.

Yang kedua adalah sebuah kotak. Kamu bisa mengeluarkan isinya, memasukkan isi baru. Kotaknya tetap kotak yang sama, tapi apa yang _di dalamnya_ bisa berubah. Inilah mutable.

Objek immutable seperti integer, string, atau tuple adalah seperti angka `42`: nilainya tetap selamanya. Tidak ada yang bisa mengubah string `"halo"` menjadi `"halo dunia"` — yang terjadi adalah variabelmu diarahkan ke objek baru `"halo dunia"`. Objek `"halo"` tetap ada, utuh, tidak tersentuh.

Objek mutable seperti list atau dict adalah seperti kotak: `[1, 2, 3]` bisa berubah menjadi `[1, 2, 3, 4]` di _tempat yang sama_. Identitas objek tidak berubah. Sticky note tetap menempel pada objek yang sama. Tapi _state_ di dalamnya berubah.

---

## In-Place Modification vs Object Replacement

Ini adalah perbedaan paling fundamental tentang mutability.

```python
# MUTABLE — in-place modification:
lst = [1, 2, 3]
print(id(lst))      # angka A
lst.append(4)
print(id(lst))      # angka A — SAMA. objek tidak diganti, state-nya yang berubah.

# IMMUTABLE — object replacement:
s = "halo"
print(id(s))        # angka B
s = s + " dunia"
print(id(s))        # angka C — BERBEDA. variabel menunjuk ke objek baru.

# Integer:
n = 100
print(id(n))        # angka D
n = n + 1
print(id(n))        # angka E — BERBEDA.
# Objek integer '100' tidak berubah menjadi '101'.
# Yang berubah adalah ke mana variabel 'n' menunjuk.
```

Perhatikan kalimat terakhir: _"yang berubah adalah ke mana variabel menunjuk."_ Ini adalah kebenaran yang dalam tentang immutability: **immutable object tidak berubah — yang berubah adalah referensi variabelmu**. Nilai `100` tetap `100` di memori sampai garbage collector membersihkannya. Variabel `n` sekarang hanya menunjuk ke tempat lain.

---

## Mengapa Python Membuat String Immutable

String bisa saja dibuat mutable — tidak ada hambatan teknis yang mencegahnya. Tapi Python memilih immutability karena tiga alasan sistemik. Ketiganya bukan keputusan terpisah — mereka adalah konsekuensi yang saling terkait dari satu prinsip.

**Alasan 1: String Interning**

Karena string tidak bisa berubah, Python bisa aman berbagi satu objek yang sama di banyak variabel tanpa risiko:

```python
a = "Python"
b = "Python"
print(a is b)   # True (biasanya) — satu objek di memori, bukan dua salinan
```

Ini tidak mungkin untuk objek mutable. Jika dua variabel berbagi satu list dan satu memodifikasinya, yang lain ikut terpengaruh secara tak terduga. Interning memungkinkan Python menghemat memori secara signifikan — terutama dalam program yang banyak menggunakan string literal yang sama berulang kali.

**Alasan 2: Hashability**

Karena immutable, string bisa di-hash dengan aman. Ini membuatnya bisa menjadi key dictionary — peran yang sangat krusial. Hampir seluruh dict dalam program Python bergantung pada string hashable sebagai key-nya. Coba bayangkan jika string berubah setelah menjadi key — seluruh mekanisme lookup akan runtuh. Mekanisme ini akan kita bahas lebih dalam di bab berikutnya.

**Alasan 3: Safety Guarantee**

```python
def format_nama(nama):
    return nama.title()   # membuat string BARU, tidak mengubah yang asli

asli = "brata yudha"
hasil = format_nama(asli)
print(asli)    # "brata yudha" — dijamin tidak berubah
print(hasil)   # "Brata Yudha"
```

Kamu bisa melempar string ke fungsi apapun tanpa perlu berpikir dua kali. String tidak bisa "bocor" menjadi modifikasi tidak disengaja. Tidak ada defensive copy yang diperlukan. Keamanan ini bukan aksidental — ia adalah konsekuensi desain yang dipertahankan dengan konsisten.

---

## Konsekuensi Mutability terhadap Safety

Berbagi referensi ke objek mutable antara beberapa bagian program berarti semua bagian itu bisa saling mempengaruhi tanpa sadar. Efek ini merambat diam-diam:

```python
konfigurasi_default = {"debug": False, "port": 8080}

def setup_dev(config):
    config["debug"] = True    # memodifikasi dict yang diterima

setup_dev(konfigurasi_default)
print(konfigurasi_default)   # {"debug": True, "port": 8080}
# konfigurasi_default tidak seharusnya berubah — tapi ia berubah
```

Ini adalah kebocoran state yang halus. Fungsi `setup_dev` tidak seharusnya mengubah konfigurasi global — tapi ia melakukannya, tanpa ada yang memberi tahu kamu.

Immutable objects memberikan jaminan sebaliknya: ketika kamu memberikan string atau integer ke fungsi, kamu tahu _dengan pasti_ bahwa nilainya tidak akan berubah di tangan penerima. Jaminan ini membuat reasoning tentang kode jauh lebih mudah — satu variabel yang tidak perlu diwaspadai berarti satu sumber kebingungan yang hilang.

---

## Behavioral Observation

Perbedaan antara in-place modifikasi dan object replacement menjadi jelas ketika kita mengamati aliasing secara langsung:

```python
# Observasi: in-place vs replacement
a = [1, 2, 3]
b = a                    # aliasing
b.append(99)
print(a)                 # [1, 2, 3, 99] — satu objek

x = "halo"
y = x
x = x.upper()            # string baru dibuat
print(y)                 # "halo" — tidak terpengaruh sama sekali
print(x is y)            # False — dua objek berbeda

# Immutable object sharing aman:
p = 42
q = 42
print(p is q)            # True — satu objek, aman karena tidak bisa diubah
```

---

# Bab 3: Hashing dan Stabilitas — Prasyarat yang Tidak Bisa Ditawar

Dari pembahasan tentang mutability, muncullah pertanyaan lanjutan yang tak terhindarkan: jika immutability memberikan stabilitas, apa sebenarnya konsekuensi _praktis_ dari stabilitas itu? Jawabannya adalah _hashability_ — dan dari hashability inilah muncul salah satu batasan desain Python yang paling sering disalahpahami.

## Masalah Fundamental

Mengapa list tidak bisa menjadi key dictionary? Mengapa set tidak hashable? Mengapa tuple _kadang_ hashable dan kadang tidak?

Pertanyaan-pertanyaan ini, sebelum dipahami, terasa seperti aturan sewenang-wenang — batasan yang dipasang tanpa alasan jelas, sekadar "memang begitu aturannya." Tapi setelah kamu memahami mekanisme di baliknya, keputusan-keputusan ini tidak hanya masuk akal — mereka terasa tak terelakkan. Mereka bukan aturan; mereka adalah konsekuensi.

## Intuisi: Sistem Loker dengan Nomor Nama

Bayangkan sebuah sistem loker yang cerdas. Setiap loker tidak punya nomor tetap — nomornya dihasilkan dari _nama penyewa_. Fungsi `hash(nama) → nomor_loker` memastikan setiap nama selalu mendapat loker yang sama.

Sistem ini bekerja karena satu asumsi fundamental: **nama penyewa tidak berubah**. Kamu datang sebagai "Ali" → sistem menghitung hash → loker 47. Besok kamu datang lagi sebagai "Ali" → hash yang sama → loker 47. Barangmu ada di sana.

Tapi bayangkan nama bisa berubah. Hari ini kamu "Ali" → loker 47. Besok namamu menjadi "Ahmad" → loker 23. Barangmu masih di loker 47, tapi sistem mengirimmu ke loker 23. Data tidak pernah ditemukan. Bukan karena sistem rusak — tapi karena premisnya (nama stabil) dilanggar.

Hash bekerja persis dengan cara ini. Nilai objek diubah menjadi nomor slot di hash table — struktur internal dict dan set. Kalau nilai objek berubah setelah disimpan sebagai key, slot yang dihitung saat pengambilan akan berbeda. Data yang disimpan menjadi tidak bisa ditemukan kembali — bukan hilang, tapi tersesat di dalam sistem yang tidak bisa lagi menjangkaunya.

---

## Apa Itu Hash

_Hash_ adalah fungsi deterministik yang mengubah input (nilai objek) menjadi integer. Deterministik artinya input yang sama selalu menghasilkan output yang sama.

Python menggunakan hash ini untuk menentukan di mana data disimpan dalam hash table — struktur internal dict dan set.

```python
print(hash("Python"))    # angka integer tertentu
print(hash("Python"))    # angka yang SAMA — deterministik
print(hash(42))          # angka lain
print(hash((1, 2, 3)))   # tuple hashable menghasilkan hash
```

Saat `d["Python"] = 3.12`:

1. Python menghitung `hash("Python")` → dapat slot `X`
2. Python menyimpan `3.12` di slot `X`

Saat `d["Python"]` diakses kembali:

1. Python menghitung `hash("Python")` lagi → dapat slot `X` yang sama
2. Python pergi ke slot `X` → menemukan `3.12`

Sistem ini hanya bekerja kalau hash-nya konsisten antara penyimpanan dan pengambilan. Dan itu hanya mungkin kalau nilai objek tidak berubah di antaranya.

---

## Kenapa List Tidak Hashable

Bayangkan jika list bisa menjadi key dict. Skenario berikut akan terjadi:

```python
# Skenario hipotetis yang tidak bisa terjadi di Python:
d = {}
key = [1, 2, 3]
d[key] = "nilai"     # disimpan di slot hash([1,2,3])

key.append(4)         # key sekarang [1,2,3,4]
d[key]                # hash([1,2,3,4]) → slot BERBEDA!
                      # "nilai" tidak pernah ditemukan — tersesat selamanya
```

Ini bukan sekadar "perilaku tidak ideal" — ini adalah kerusakan fundamental pada mekanisme lookup. Data tidak hilang secara fisik (masih ada di slot lama), tapi sistem kehilangan kemampuan untuk menemukannya kembali. Hash table menjadi korup, dan tidak ada error yang memberitahumu.

Python mencegah skenario ini dengan tegas:

```python
lst = [1, 2, 3]
hash(lst)   # TypeError: unhashable type: 'list'
```

Python menolak di awal — sebelum ada kesempatan untuk merusak data. Lebih baik error yang jelas daripada data yang diam-diam tidak bisa ditemukan.

---

## Kenapa Tuple Kadang Hashable Kadang Tidak

Hash tuple dihitung dari _kombinasi hash semua elemennya_. Kalau salah satu elemen tidak bisa di-hash, hash tuple itu sendiri tidak bisa dihitung.

```python
t1 = (1, 2, 3)        # semua elemen immutable → hashable
t2 = (1, [2, 3])      # mengandung list mutable → tidak hashable
t3 = (1, (2, [3]))    # nested: tuple → tuple → list → tidak hashable

print(hash(t1))        # bekerja

try:
    hash(t2)
except TypeError as e:
    print(f"TypeError: {e}")   # unhashable type: 'list'

try:
    hash(t3)
except TypeError as e:
    print(f"TypeError: {e}")   # unhashable type: 'list'
```

Hashability bukan hanya tentang tipe di permukaan — ia tentang _seluruh grafnya secara rekursif_. Kalau ada satu node mutable di dalam struktur, seluruh struktur itu tidak hashable.

---

## Kenapa Frozenset Ada

Set adalah mutable — isinya bisa berubah setelah dibuat. Jika hash set dihitung saat pembuatan dan isinya kemudian berubah, hash tersebut menjadi tidak valid. Inilah mengapa set tidak hashable:

```python
s = {1, 2, 3}
hash(s)   # TypeError: unhashable type: 'set'
```

Tapi kebutuhan akan _"himpunan yang bisa di-hash"_ tetap ada. Di sinilah frozenset hadir — versi immutable dari set, stabil, hashable, dan bisa menjadi key dictionary atau elemen di dalam set lain.

```python
fs = frozenset([1, 2, 3])
print(hash(fs))   # bekerja

# Kasus nyata: kombinasi preferensi sebagai key cache
cache = {}
pref_a = frozenset(["rock", "jazz", "indie"])
pref_b = frozenset(["pop", "jazz"])

cache[pref_a] = ["rekomendasi_1", "rekomendasi_2"]
cache[pref_b] = ["rekomendasi_3"]

# Urutan elemen tidak penting saat lookup — set tidak ordered
print(cache[frozenset(["jazz", "rock", "indie"])])
# ["rekomendasi_1", "rekomendasi_2"]
```

Perhatikan pola yang muncul: hubungan `set ↔ frozenset` mencerminkan hubungan `list ↔ tuple`. Python secara konsisten menyediakan pasangan mutable/immutable untuk setiap tipe koleksi yang membutuhkannya. Pola ini bukan kebetulan — ia ada karena kebutuhan yang sama muncul berulang kali: seringkali kamu membutuhkan versi immutable (dan karenanya hashable) dari suatu struktur, tepat di saat versi mutablenya lebih natural untuk dibangun.

---

## Behavioral Observation

Mari kita petakan batas hashability dengan jelas:

```python
# Peta hashability

# Hashable:
for obj in [42, 3.14, "Python", True, None, (1, 2), frozenset([1])]:
    print(f"hash({obj!r:25}) = {hash(obj)}")

print()

# Tidak Hashable:
for obj in [[1, 2], {1, 2}, {"a": 1}, (1, [2, 3])]:
    try:
        hash(obj)
    except TypeError as e:
        print(f"hash({obj!r:25}) → TypeError")
```

```
hash(42)                   = 42
hash(3.14)                 = ...
hash('Python')             = ...
hash(True)                 = 1
hash(None)                 = 0
hash((1, 2))               = ...
hash(frozenset({1}))       = ...

hash([1, 2])               → TypeError
hash({1, 2})               → TypeError
hash({'a': 1})             → TypeError
hash((1, [2, 3]))          → TypeError
```

---

# Bab 4: Copying dan Nested Structures — Di Mana Reference Bersembunyi

Setelah memahami bahwa stabilitas hash mensyaratkan immutability, kita dihadapkan pada pertanyaan praktis: bagaimana cara _bekerja_ dengan data ketika mutability adalah kebutuhan, bukan kelemahan? Jawabannya ada pada mekanisme copying — dan di situlah reference semantics menunjukkan konsekuensinya yang paling nyata.

## Masalah Fundamental

Kamu memiliki dict berisi list. Kamu menyalinnya. Kamu memodifikasi salinannya. Dan list di dalam dict aslimu ikut berubah.

Ini bukan bug. Ini adalah shallow copy bekerja persis seperti yang dirancang. Tapi "bekerja sesuai rancangan" tidak selalu berarti "bekerja seperti yang kamu harapkan" — dan jarak antara keduanya adalah tempat bug-bug halus lahir.

## Intuisi: Peta vs Wilayah

Ada perbedaan mendasar antara menyalin sebuah peta dan mereplikasi seluruh wilayah yang dipetakan.

Ketika kamu menyalin peta, kamu mendapat selembar kertas baru dengan gambar yang sama. Tapi gambar itu masih _merujuk_ ke lokasi yang sama di dunia nyata. Jika sebuah gedung di wilayah itu hancur, gambar gedung di kedua peta — asli dan salinan — sama-sama tidak lagi akurat. Keduanya menunjuk ke realitas yang sama.

_Shallow copy_ bekerja seperti menyalin peta: container baru, tapi referensi ke konten yang sama. _Deep copy_ bekerja seperti mereplikasi seluruh wilayah: container baru, dan semua yang ada di dalamnya juga diciptakan ulang secara independen. Tidak ada hubungan yang tersisa antara asli dan salinan.

---

## Tiga Tingkat "Salinan"

Satu hal penting untuk dipahami sejak awal: tidak semua hal yang terlihat seperti menyalin sebenarnya menyalin. Python memiliki tiga tingkat perilaku, dan masing-masing memberikan jaminan yang berbeda.

### Level 1 — Assignment: Bukan Copy Sama Sekali

```python
a = [1, 2, 3]
b = a        # aliasing, bukan copy
```

`b = a` hanya menempelkan sticky note baru pada objek yang sudah ada. Tidak ada data yang disalin. Tidak ada objek baru yang diciptakan. `b` dan `a` adalah dua nama untuk satu objek — modifikasi melalui nama manapun akan terlihat di keduanya.

### Level 2 — Shallow Copy: Container Baru, Referensi Lama

Shallow copy membuat _container baru_, tapi elemen-elemennya masih merujuk ke objek yang sama.

```python
import copy

original = [[1, 2], [3, 4], [5, 6]]
shallow  = copy.copy(original)      # juga bisa: original.copy() atau original[:]

print(original is shallow)          # False — container yang berbeda
print(original[0] is shallow[0])    # True  — elemen yang SAMA
```

Untuk struktur datar (flat) — di mana semua elemen adalah immutable — shallow copy sudah cukup:

```python
a = [1, 2, 3]
b = a.copy()
b.append(99)
print(a)   # [1, 2, 3] — tidak terpengaruh
# Integer immutable, berbagi referensi aman
```

Tapi untuk nested mutable structures, shallow copy menghasilkan kejutan:

```python
asli    = [[1, 2], [3, 4]]
salinan = asli.copy()

salinan[0].append(99)   # memodifikasi sub-list via salinan

print(asli)     # [[1, 2, 99], [3, 4]] — sub-list asli ikut berubah!
print(salinan)  # [[1, 2, 99], [3, 4]]
```

Mengapa? `salinan[0]` adalah sticky note yang menunjuk ke objek _yang sama_ dengan `asli[0]` — sub-list `[1, 2]`. Shallow copy hanya menyalin lapisan terluar. Lapisan dalamnya tetap berbagi referensi.

Cara memvisualkan ini:

<div style="font-family: system-ui, -apple-system, sans-serif; background: #1a1a1d; border: 1px solid #333; border-radius: 14px; padding: 28px; color: #f5f5f7; max-width: 680px; margin: 30px auto; box-shadow: 0 8px 24px rgba(0,0,0,0.35);">
<div style="font-size: 0.75rem; font-weight: 700; text-transform: uppercase; letter-spacing: 0.14em; color: #6b7280; margin-bottom: 14px; border-bottom: 1px solid #2a2a2d; padding-bottom: 10px;">Visualisasi: Efek Shallow Copy</div>
<div style="background: rgba(43,43,46,0.5); padding: 20px; border-radius: 10px; border: 1px solid #2a2a2d; margin-bottom: 18px;">
<div style="display: flex; flex-direction: column; gap: 20px; font-family: 'SF Mono', 'Fira Code', monospace; font-size: 0.9rem;">
<div style="display: flex; align-items: center; gap: 12px;">
<div style="width: 70px; color: #a78bfa; font-weight: 600; text-align: right; font-size: 0.85rem;">asli</div>
<div style="color: #6b7280;">➔</div>
<div style="background: #2b2b2e; padding: 8px 14px; border-radius: 6px; border: 1px solid #444; display: flex; gap: 14px;">
<span style="color: #4a9e8d; border-bottom: 2px solid #4a9e8d;">[ref_A]</span>
<span style="color: #b16d7a; border-bottom: 2px solid #b16d7a;">[ref_B]</span>
</div>
</div>
<div style="display: flex; justify-content: center; gap: 56px; margin: -8px 0; color: #6b7280; font-size: 1.1rem; padding-left: 36px;">
<div style="display: flex; flex-direction: column; align-items: center;"><div>▲</div><div style="height: 16px; border-left: 2px dashed rgba(74,158,141,0.4);"></div><div>▼</div></div>
<div style="display: flex; flex-direction: column; align-items: center;"><div>▲</div><div style="height: 16px; border-left: 2px dashed rgba(177,109,122,0.4);"></div><div>▼</div></div>
</div>
<div style="display: flex; align-items: center; gap: 12px;">
<div style="width: 70px; color: #a78bfa; font-weight: 600; text-align: right; font-size: 0.85rem;">salinan</div>
<div style="color: #6b7280;">➔</div>
<div style="background: #2b2b2e; padding: 8px 14px; border-radius: 6px; border: 1px solid #444; display: flex; gap: 14px;">
<span style="color: #4a9e8d; border-bottom: 2px solid #4a9e8d;">[ref_A]</span>
<span style="color: #b16d7a; border-bottom: 2px solid #b16d7a;">[ref_B]</span>
</div>
</div>
</div>
</div>
<div style="display: flex; gap: 36px; justify-content: center; margin-bottom: 18px; font-family: 'SF Mono', 'Fira Code', monospace; font-size: 0.85rem; background: rgba(0,0,0,0.2); padding: 10px 14px; border-radius: 8px;">
<div><span style="color: #4a9e8d;">• ref_A</span> ➔ <span style="background: #2b2b2e; padding: 2px 6px; border-radius: 4px; color: #e1e1e1;">[1, 2]</span></div>
<div><span style="color: #b16d7a;">• ref_B</span> ➔ <span style="background: #2b2b2e; padding: 2px 6px; border-radius: 4px; color: #e1e1e1;">[3, 4]</span></div>
</div>
<div style="border-left: 3px solid #7c3aed; padding-left: 14px; font-size: 0.88rem; line-height: 1.6; color: #a1a1aa;">
<p style="margin: 0 0 4px 0;"><strong style="color: #e1e1e1;">Prinsip:</strong> Kontainer luar (<code style="font-family: monospace; background: #2b2b2e; padding: 1px 4px; border-radius: 3px; color: #a78bfa; font-size: 0.85rem;">asli</code> &amp; <code style="font-family: monospace; background: #2b2b2e; padding: 1px 4px; border-radius: 3px; color: #a78bfa; font-size: 0.85rem;">salinan</code>) memiliki alamat memori yang berbeda.</p>
<p style="margin: 0;"><strong style="color: #b16d7a;">Konsekuensi:</strong> Elemen di dalamnya (<code style="font-family: monospace; background: #2b2b2e; padding: 1px 4px; border-radius: 3px; color: #4a9e8d; font-size: 0.85rem;">ref_A</code> &amp; <code style="font-family: monospace; background: #2b2b2e; padding: 1px 4px; border-radius: 3px; color: #b16d7a; font-size: 0.85rem;">ref_B</code>) tetap menunjuk ke objek yang sama. Modifikasi melalui variabel manapun akan mengubah keduanya.</p>
</div>
</div>

### Level 3 — Deep Copy: Independensi Penuh

```python
import copy

asli  = [[1, 2], [3, 4]]
dalam = copy.deepcopy(asli)

dalam[0].append(99)

print(asli)    # [[1, 2], [3, 4]] — tidak terpengaruh sama sekali
print(dalam)   # [[1, 2, 99], [3, 4]]

print(asli[0] is dalam[0])   # False — dua objek yang benar-benar berbeda
```

Deep copy membuat salinan rekursif — setiap objek di setiap level ikut disalin ulang. Container baru, dan semua referensi di dalamnya juga menunjuk ke objek-objek baru.

---

## Default Mutable Argument — Jebakan Klasik

Ini adalah salah satu jebakan Python yang paling terkenal — dan paling tidak intuitif, bahkan untuk programmer berpengalaman. Ia menggabungkan reference semantics dengan evaluasi default argument menjadi satu perilaku yang tampak seperti bug:

```python
def tambah_ke_log(pesan, log=[]):   # default dievaluasi SEKALI saat definisi
    log.append(pesan)
    return log

print(tambah_ke_log("error 1"))   # ["error 1"]
print(tambah_ke_log("error 2"))   # ["error 1", "error 2"]
print(tambah_ke_log("error 3"))   # ["error 1", "error 2", "error 3"]
```

Akar penyebabnya: argumen default `[]` dievaluasi _sekali — saat fungsi didefinisikan_, bukan setiap kali fungsi dipanggil. Objek list itu hidup di atribut `__defaults__` fungsi dan tidak pernah diganti. Setiap pemanggilan tanpa argumen `log` akan menggunakan _objek yang sama_ — dan setiap pemanggilan memodifikasinya.

```python
print(tambah_ke_log.__defaults__)   # ([...],) — selalu objek yang sama
```

Ini bukan bug Python. Ini adalah konsekuensi yang sangat konsisten dari reference semantics-nya: default argument adalah referensi ke objek, dan objek mutable bisa berubah. Jika Python mengevaluasi ulang default setiap pemanggilan, ia harus menyimpan _cara membuat objek_, bukan _objek itu sendiri_ — dan itu berarti sistem yang lebih kompleks hanya untuk kasus tepi ini.

Pola idiomatik untuk menghindari jebakan ini:

```python
def tambah_ke_log(pesan, log=None):
    if log is None:
        log = []       # buat objek baru setiap pemanggilan
    log.append(pesan)
    return log
```

Dengan pola ini, setiap pemanggilan tanpa argumen `log` mendapatkan objek list baru yang independen.

---

## Behavioral Observation

Untuk melihat perbedaan ketiga level copying secara berdampingan:

```python
import copy

original = [1, [10, 20], {"x": 100}]

alias   = original
shallow = copy.copy(original)
dalam   = copy.deepcopy(original)

# Modifikasi elemen primitif di level teratas:
original[0] = 999
print(alias[0])    # 999 — satu objek
print(shallow[0])  # 1   — container berbeda, tidak terpengaruh
print(dalam[0])    # 1   — sepenuhnya independen

# Modifikasi nested list:
original[1].append(30)
print(alias[1])    # [10, 20, 30] — satu objek
print(shallow[1])  # [10, 20, 30] — MASIH TERPENGARUH (shared reference)
print(dalam[1])    # [10, 20]     — tidak terpengaruh

# Modifikasi nested dict:
original[2]["y"] = 200
print(alias[2])    # {"x": 100, "y": 200}
print(shallow[2])  # {"x": 100, "y": 200} — MASIH TERPENGARUH
print(dalam[2])    # {"x": 100}            — tidak terpengaruh
```

Shallow copy melindungi dari perubahan di level teratas container — tapi tidak dari perubahan di dalam objek yang dirujuk.

---

# Bab 5: Behavior Emergence — Ketika Semua Sistem Bertemu

## Masalah Fundamental

Bab-bab sebelumnya membahas identity, mutability, hashing, dan copying seolah-olah mereka adalah topik yang terpisah — masing-masing dengan aturan dan konsekuensinya sendiri. Tapi di Python nyata, mereka tidak pernah beroperasi sendiri-sendiri. Mereka berinteraksi terus-menerus — dan dari interaksi inilah lahir behavior-behavior yang paling membingungkan.

Sejauh ini kita sudah melihat setiap prinsip secara terisolasi. Sekarang saatnya melihat bagaimana mereka saling mempengaruhi dalam kode nyata, dan bagaimana dari interaksi sederhana muncullah kompleksitas yang tak terduga.

Bab ini adalah sintesis. Kita akan mengamati perilaku-perilaku yang tampak paling ajaib, lalu menelusuri dengan tepat dari mana mereka muncul dalam sistem. Setiap fenomena yang kita lihat akan kita bongkar hingga ke akar penyebabnya — dan kita akan melihat bahwa tidak ada yang ajaib sama sekali.

---

## `+=` — Satu Syntax, Dua Behavior

`+=` bukan satu operator dengan satu cara kerja yang tetap. Ia adalah shorthand sintaksis yang didelegasikan ke metode internal objek — dan metode itu berbeda tergantung tipe objeknya.

```python
# Pada list (mutable):
a = [1, 2, 3]
b = a
a += [4, 5]       # memanggil a.__iadd__([4,5]) — in-place!

print(a)           # [1, 2, 3, 4, 5]
print(b)           # [1, 2, 3, 4, 5] — b melihatnya karena satu objek
print(a is b)      # True — masih objek yang sama

# Pada string (immutable):
x = "halo"
y = x
x += " dunia"     # ekuivalen dengan x = x + " dunia" — objek BARU

print(x)           # "halo dunia"
print(y)           # "halo" — y tidak terpengaruh
print(x is y)      # False — objek yang berbeda sekarang

# Pada tuple (immutable):
p = (1, 2)
q = p
p += (3, 4)       # ekuivalen dengan p = p + (3,4) — tuple BARU

print(p)           # (1, 2, 3, 4)
print(q)           # (1, 2) — tidak terpengaruh
print(p is q)      # False
```

Mekanismenya: Python pertama-tama mencari metode `__iadd__` (in-place addition). Jika tipe objek mendefinisikannya (seperti list), `+=` memodifikasi objek yang sama dan mengembalikannya. Jika tidak (seperti string dan tuple), Python _fallback_ ke `__add__` — yang selalu membuat objek baru.

Satu syntax, dua semantics — sepenuhnya ditentukan oleh ada atau tidaknya `__iadd__`, yang pada gilirannya ditentukan oleh mutability objek. Tidak ada ambiguitas dalam sistem — hanya ambiguitas dalam ekspektasi kita.

---

## Tuple yang "Berubah"

```python
t = ([1, 2], [3, 4])
t[0].append(99)
print(t)   # ([1, 2, 99], [3, 4])
```

Tuple tidak berubah. Mari kita buktikan: `t[0]` masih menunjuk ke _objek yang sama_ dengan sebelum — sub-list `[1, 2]` yang sama, di alamat memori yang sama. Yang berubah adalah _isi dari objek yang dirujuk_, bukan tuple itu sendiri.

Ini adalah nuansa penting yang sering terlewatkan:

Immutability tuple adalah jaminan tentang _referensi_ — bukan tentang _nilai_. Kamu tidak bisa mengubah _ke mana_ slot-slot tuple menunjuk. Tapi jika objek yang dirujuk itu sendiri mutable, Python tidak bisa — dan tidak akan — mencegahnya berubah dari dalam.

```python
t = ([1, 2], [3, 4])
t[0] = [10, 20]         # TypeError: tidak bisa mengganti referensi
t[0].append(99)         # valid: memodifikasi objek yang dirujuk
```

Dua baris ini menunjukkan batas immutability tuple dengan sempurna: baris pertama mencoba mengganti _ke mana_ slot menunjuk — ditolak. Baris kedua memodifikasi _isi_ dari objek yang sudah dirujuk — diizinkan. Tuple tetap setia pada jaminannya: ia tidak pernah membiarkanmu mengganti referensi-referensinya. Tapi ia tidak punya kendali atas apa yang terjadi di dalam objek-objek yang dirujuknya.

---

## Nested Mutation Leakage

Ini adalah kombinasi dari aliasing, mutability, dan cara pembuatan nested structure yang menghasilkan bug yang sangat sulit dilacak.

`list multiplication` — `[obj] * n` — tidak membuat n objek baru. Ia membuat n referensi ke _objek yang sama_. Ini efisien, tapi mematikan jika objeknya mutable.

```python
# Pattern berbahaya: list multiplication untuk membuat matrix
matrix = [[0] * 3] * 3    # JANGAN LAKUKAN INI
```

`[[0] * 3] * 3` menghasilkan tiga _referensi ke objek yang sama_ — bukan tiga list terpisah. Mengubah `matrix[0]` berarti mengubah objek yang sama yang dirujuk oleh ketiga elemen.

```
matrix → [ ref_A, ref_A, ref_A ]
               ↓
           [0, 0, 0]   (satu objek, tiga referensi)
```

```python
# Cara yang benar: list comprehension
matrix = [[0] * 3 for _ in range(3)]

matrix[0][1] = 99
print(matrix)
# [[0, 99, 0], [0, 0, 0], [0, 0, 0]] — hanya baris pertama
```

List comprehension membuat _objek baru_ untuk setiap iterasi. `[0] * 3` dieksekusi tiga kali — tiga objek berbeda, tiga referensi berbeda.

---

## Shared References dalam Nested Structure

```python
default_config = {
    "mode"   : "production",
    "flags"  : ["read", "write"],
    "limits" : {"max_retry": 3}
}

# Shallow copy: container dict baru, tapi value masih shared
dev_config = default_config.copy()
dev_config["mode"] = "development"       # aman: string immutable, referensi diganti

dev_config["flags"].append("debug")      # TIDAK aman: list yang sama!
dev_config["limits"]["max_retry"] = 10   # TIDAK aman: dict yang sama!

print(default_config["mode"])    # "production" — tidak terpengaruh ✓
print(default_config["flags"])   # ["read", "write", "debug"] — terpengaruh ✗
print(default_config["limits"])  # {"max_retry": 10} — terpengaruh ✗
```

Perhatikan pola yang instruktif di sini: `dev_config["mode"] = "development"` _aman_ — string immutable, dan penugasan ini mengganti _referensi_ di `dev_config`, bukan mengubah string itu sendiri. Tapi `dev_config["flags"].append("debug")` memodifikasi _objek yang sama_ yang juga dirujuk oleh `default_config["flags"]`. Begitu pula dengan `limits`.

```python
import copy
dev_config = copy.deepcopy(default_config)   # isolasi penuh
```

---

## Semua Bersumber dari Satu Keputusan

Semua perilaku "aneh" yang kita amati di bab ini bukanlah anomali yang terisolasi. Mereka adalah manifestasi berbeda dari satu keputusan desain fundamental Python:

> **Variabel Python adalah referensi ke objek, bukan wadah nilai.**

Dari satu keputusan ini, semuanya mengalir secara logis:

- **Assignment** adalah aliasing — sticky note baru pada objek yang sudah ada
- **`+=`** berperilaku berbeda pada mutable vs immutable — tergantung ada tidaknya `__iadd__`
- **Shallow copy** hanya menyalin container — sub-objek tetap berbagi referensi
- **Default mutable argument** — satu objek yang sama digunakan di semua pemanggilan
- **Tuple "berubah"** — bukan tuple-nya, tapi isi objek yang dirujuk
- **List multiplication** `[obj] * n` — n referensi ke objek yang sama, bukan n objek

Semua konsisten. Tidak ada yang kebetulan. Tidak ada bug dalam sistem — hanya gap antara model mental kita dan model Python. Menutup gap itu berarti memahami satu prinsip, dan dari prinsip itu, semuanya menjadi jelas.

---

# Bab 6: Mental Model Akhir

Kita telah menelusuri setiap prinsip: identity, mutability, hashing, copying. Kita telah melihat bagaimana mereka berinteraksi dan dari interaksi itu muncul perilaku-perilaku yang tadinya membingungkan. Sekarang saatnya menyatukan semuanya ke dalam satu model mental yang koheren.

## Python's Object Model

<div style="font-family: system-ui, -apple-system, sans-serif; background: #1a1a1d; border: 1px solid #333; border-radius: 14px; padding: 28px; color: #f5f5f7; max-width: 680px; margin: 30px auto; box-shadow: 0 8px 24px rgba(0,0,0,0.35);">
<div style="text-align: center; margin-bottom: 24px; border-bottom: 1px solid #333; padding-bottom: 14px;">
<h3 style="margin: 0; color: #a78bfa; font-size: 1.2rem; font-weight: 600; letter-spacing: 0.02em;">Konsep Variabel &amp; Objek</h3>
<p style="margin: 8px 0 0 0; font-size: 0.9rem; color: #a1a1aa; font-style: italic;">"Setiap variabel adalah <strong style="color: #c4b5fd;">REFERENSI</strong> ke objek, bukan objek itu sendiri."</p>
</div>
<div style="display: flex; align-items: center; justify-content: center; gap: 20px; margin-bottom: 32px; background: rgba(43,43,46,0.5); padding: 16px 20px; border-radius: 10px; border: 1px solid #2a2a2d;">
<div style="background: #2b2b2e; padding: 8px 18px; border-radius: 6px; font-weight: 600; border: 1px solid #444; color: #e1e1e1; font-size: 0.95rem;">nama</div>
<div style="font-size: 1.4rem; color: #a78bfa;">➔</div>
<div style="border: 2px dashed #4a4a4e; border-radius: 10px; padding: 14px 22px; min-width: 190px; background: rgba(124,58,237,0.04);">
<div style="text-align: center; font-weight: 600; margin-bottom: 8px; letter-spacing: 0.05em; color: #a78bfa; font-size: 0.85rem;">[ OBJEK ]</div>
<div style="font-size: 0.85rem; font-family: 'SF Mono', 'Fira Code', monospace; line-height: 1.8;">
<div style="color: #d4d4d8;">• <span style="color: #a78bfa;">id</span> <span style="color: #6b7280; font-size: 0.8rem;">(identitas unik)</span></div>
<div style="color: #d4d4d8;">• <span style="color: #4a9e8d;">type</span> <span style="color: #6b7280; font-size: 0.8rem;">(tipe data)</span></div>
<div style="color: #d4d4d8;">• <span style="color: #b16d7a;">value</span> <span style="color: #6b7280; font-size: 0.8rem;">(nilai dalam memori)</span></div>
</div>
</div>
</div>
<div style="display: grid; grid-template-columns: 1fr 1fr; gap: 18px;">
<div style="background: rgba(52,211,153,0.04); border: 1px solid rgba(52,211,153,0.25); border-radius: 10px; padding: 16px;">
<h4 style="margin: 0 0 6px 0; color: #34d399; font-size: 0.95rem; font-weight: 600; letter-spacing: 0.03em;">MUTABLE</h4>
<p style="font-size: 0.8rem; margin: 0 0 14px 0; color: #a1a1aa; line-height: 1.4;">Value bisa berubah, <span style="text-decoration: underline; text-decoration-color: #34d399;">ID tetap sama</span>.</p>
<div style="font-family: 'SF Mono', 'Fira Code', monospace; font-size: 0.8rem; background: rgba(0,0,0,0.2); padding: 12px; border-radius: 8px; border: 1px solid #2a2a2d; line-height: 1.7;">
<div style="color: #6b7280;"># Inisialisasi</div>
<div style="color: #e1e1e1;">a ➔ <span style="color: #34d399;">[list: 1,2,3]</span></div>
<div style="margin: 5px 0; color: #6b7280;">a.append(4)</div>
<div style="color: #6b7280;"># Objek yang sama dimodifikasi</div>
<div style="color: #e1e1e1;">a ➔ <span style="color: #34d399;">[list: 1,2,3,4]</span></div>
</div>
</div>
<div style="background: rgba(248,113,113,0.04); border: 1px solid rgba(248,113,113,0.25); border-radius: 10px; padding: 16px;">
<h4 style="margin: 0 0 6px 0; color: #f87171; font-size: 0.95rem; font-weight: 600; letter-spacing: 0.03em;">IMMUTABLE</h4>
<p style="font-size: 0.8rem; margin: 0 0 14px 0; color: #a1a1aa; line-height: 1.4;">Value tidak bisa berubah. <span style="text-decoration: underline; text-decoration-color: #f87171;">Objek baru</span> dibuat.</p>
<div style="font-family: 'SF Mono', 'Fira Code', monospace; font-size: 0.8rem; background: rgba(0,0,0,0.2); padding: 12px; border-radius: 8px; border: 1px solid #2a2a2d; line-height: 1.7;">
<div style="color: #6b7280;"># Inisialisasi</div>
<div style="color: #e1e1e1;">s ➔ <span style="color: #f87171;">[str: "halo"]</span></div>
<div style="margin: 5px 0; color: #6b7280;">s = s + "!"</div>
<div style="color: #6b7280;"># Membuat OBJEK BARU</div>
<div style="color: #e1e1e1;">s ➔ <span style="color: #f87171;">[str: "halo!"]</span></div>
<div style="font-size: 0.7rem; margin-top: 6px; color: #6b7280; border-top: 1px dashed #2a2a2d; padding-top: 6px;">✗ <span style="text-decoration: line-through;">[str: "halo"]</span> ditinggal di memori</div>
</div>
</div>
</div>
</div>

---

## Tabel Konsekuensi Sistemik

| Aspek | Mutable (list, dict, set) | Immutable (tuple, str, frozenset, int) |
|---|---|---|
| **Modifikasi** | In-place — `id` tetap | Object replacement — `id` baru |
| **Aliasing** | Berbahaya jika tidak disadari | Aman — tidak bisa bocor |
| **Shallow copy** | Seringkali tidak cukup | Selalu cukup |
| **Hashable** | Tidak | Ya (jika seluruh graf hashable) |
| **Key dict / elemen set** | Tidak bisa | Bisa |
| **Default argument** | Jangan gunakan langsung | Aman sebagai default |
| **Berbagi antar fungsi** | Efisien tapi berisiko | Aman sepenuhnya |

---

## Rantai Logis dari Satu Prinsip

<div style="font-family: system-ui, -apple-system, sans-serif; background: #1a1a1d; border: 1px solid #333; border-radius: 14px; padding: 28px; color: #f5f5f7; max-width: 720px; margin: 30px auto; box-shadow: 0 8px 24px rgba(0,0,0,0.35); position: relative; overflow: hidden;">
<div style="position: absolute; top: -80px; left: -80px; width: 300px; height: 300px; background: radial-gradient(circle, rgba(128,90,213,0.04) 0%, transparent 70%); pointer-events: none;"></div>
<div style="position: absolute; bottom: -80px; right: -80px; width: 300px; height: 300px; background: radial-gradient(circle, rgba(107,114,128,0.04) 0%, transparent 70%); pointer-events: none;"></div>
<div style="background: linear-gradient(135deg, #7c3aed, #a78bfa); color: #fff; padding: 14px 20px; border-radius: 10px; font-weight: 700; font-size: 1.15rem; text-align: center; margin-bottom: 24px; box-shadow: 0 4px 12px rgba(124,58,237,0.25); letter-spacing: 0.02em;">Variabel = Referensi ke Objek</div>
<div style="display: flex; flex-direction: column; gap: 16px;">
<div style="background: rgba(43,43,46,0.5); border-left: 4px solid #7c3aed; border-radius: 4px 10px 10px 4px; padding: 16px 18px; border: 1px solid #333;">
<h4 style="margin: 0 0 6px 0; color: #a78bfa; font-size: 1rem; font-weight: 600;">1. Assignment = Aliasing <span style="font-weight: 400; color: #6b7280; font-size: 0.85rem; margin-left: 6px;">(Bukan Salinan)</span></h4>
<div style="font-size: 0.9rem; padding-left: 16px; border-left: 2px dashed #3a3a3d; margin-top: 8px; color: #a1a1aa; line-height: 1.5;">➔ <strong style="color: #e1e1e1;">Mutation via satu nama</strong> akan terlihat dari semua nama lain yang merujuk ke objek yang sama.</div>
</div>
<div style="background: rgba(43,43,46,0.5); border-left: 4px solid #34d399; border-radius: 4px 10px 10px 4px; padding: 16px 18px; border: 1px solid #333;">
<h4 style="margin: 0 0 6px 0; color: #34d399; font-size: 1rem; font-weight: 600;">2. MUTABLE <span style="font-weight: 400; color: #6b7280; font-size: 0.85rem; margin-left: 6px;">(State berubah, ID tetap)</span></h4>
<ul style="margin: 8px 0 0 0; padding-left: 20px; font-size: 0.9rem; line-height: 1.6; color: #d4d4d8; list-style: none;">
<li style="padding-left: 16px; position: relative; margin-bottom: 4px;"><span style="position: absolute; left: 0; top: 0; color: #34d399;">•</span> <strong style="color: #34d399;">In-place:</strong> Objek diubah langsung di memori — container baru tidak dibuat.</li>
<li style="padding-left: 16px; position: relative; margin-bottom: 4px;"><span style="position: absolute; left: 0; top: 0; color: #34d399;">•</span> <strong style="color: #34d399;">Aliasing berisiko</strong> — nested structures berbagi referensi diam-diam.</li>
<li style="padding-left: 16px; position: relative; margin-bottom: 4px;"><span style="position: absolute; left: 0; top: 0; color: #34d399;">•</span> <strong style="color: #34d399;">Shallow copy</strong> tidak memutus hubungan referensi ke sub-objek.</li>
<li style="padding-left: 16px; position: relative; color: #f87171;"><span style="position: absolute; left: 0; top: 0; color: #f87171;">•</span> <strong style="color: #f87171;">Tidak hashable</strong> — tidak bisa jadi key dict atau elemen set.</li>
</ul>
</div>
<div style="background: rgba(43,43,46,0.5); border-left: 4px solid #f87171; border-radius: 4px 10px 10px 4px; padding: 16px 18px; border: 1px solid #333;">
<h4 style="margin: 0 0 6px 0; color: #f87171; font-size: 1rem; font-weight: 600;">3. IMMUTABLE <span style="font-weight: 400; color: #6b7280; font-size: 0.85rem; margin-left: 6px;">(State tidak bisa berubah)</span></h4>
<ul style="margin: 8px 0 0 0; padding-left: 20px; font-size: 0.9rem; line-height: 1.6; color: #d4d4d8; list-style: none;">
<li style="padding-left: 16px; position: relative; margin-bottom: 4px;"><span style="position: absolute; left: 0; top: 0; color: #f87171;">•</span> <strong style="color: #f87171;">"Perubahan"</strong> = variabel dialihkan ke objek baru di memori.</li>
<li style="padding-left: 16px; position: relative; margin-bottom: 4px;"><span style="position: absolute; left: 0; top: 0; color: #34d399;">•</span> <strong style="color: #34d399;">Aman untuk sharing</strong> — tidak perlu defensive copy.</li>
<li style="padding-left: 16px; position: relative; margin-bottom: 4px;"><span style="position: absolute; left: 0; top: 0; color: #34d399;">•</span> <strong style="color: #34d399;">Hash stabil</strong> → bisa jadi key dict (jika seluruh graf komponen hashable).</li>
<li style="padding-left: 16px; position: relative; color: #34d399;"><span style="position: absolute; left: 0; top: 0; color: #34d399;">•</span> <strong style="color: #34d399;">Bisa digunakan</strong> sebagai key <code style="font-family: monospace; color: #e1e1e1; background: #2b2b2e; padding: 1px 5px; border-radius: 3px; font-size: 0.85rem;">dict</code> atau elemen <code style="font-family: monospace; color: #e1e1e1; background: #2b2b2e; padding: 1px 5px; border-radius: 3px; font-size: 0.85rem;">set</code>.</li>
</ul>
</div>
</div>
</div>

---

## Tiga Pertanyaan untuk Dibawa Selamanya

Ketika kamu menulis atau membaca kode Python dan sesuatu terasa tidak jelas — sebuah variabel berubah tanpa alasan, sebuah struktur data tiba-tiba berbeda dari ekspektasi — tiga pertanyaan ini akan mengembalikanmu ke jalur yang benar:

**Pertama: Berapa banyak nama menunjuk ke objek yang sama?** Gunakan `is` untuk mengecek. Jika dua nama menunjuk ke objek mutable yang sama, memodifikasi salah satu mempengaruhi keduanya. Pertanyaan ini mengungkap aliasing tersembunyi.

**Kedua: Jika objek ini dimodifikasi, siapa lagi yang akan melihatnya?** Telusuri semua pemegang referensi: fungsi yang menerimanya sebagai argumen? Variabel lain di scope yang sama? Struktur data yang menyimpannya? Pertanyaan ini mengungkap footprint aliasing.

**Ketiga: Apakah "salinan" ini benar-benar terpisah?** Untuk nested mutable structures, `.copy()` seringkali hanya memotong lapisan permukaan. Tanyakan: apakah sub-objek juga perlu independen? Jika ya, `copy.deepcopy()` adalah jalan keluarnya.

---

## Kata Penutup

Ada sebuah momen dalam belajar Python di mana semuanya tiba-tiba _klik_. Bukan momen ketika kamu menghafal semua method, atau ketika kamu berhasil menulis program yang berjalan tanpa error.

Tapi momen ketika kamu melihat baris kode seperti `b = a` dan langsung tahu — tanpa perlu berpikir panjang — bahwa itu adalah aliasing, bukan copy; bahwa memodifikasi `b` akan mempengaruhi `a`; bahwa jika kamu tidak ingin itu terjadi, kamu perlu `.copy()` atau `deepcopy()` tergantung struktur datanya.

Momen itu adalah ketika model mental sudah sepenuhnya terinternalisasi. Bukan sebagai aturan yang dihafal, tapi sebagai _intuisi_ yang mengalir alami dari pemahaman tentang sistem.

Python tidak berperilaku aneh. Python berperilaku _sangat konsisten_ — konsisten dengan model objeknya. Dan model itu, sekali dipahami, tidak hanya masuk akal. Ia menjadi bagian dari cara kamu berpikir tentang kode.

---

_← Volume 3: Organization Style | Volume 5: Abstract Data Types & Collections Module →_

---
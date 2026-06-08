# Beyond The Syntax — Python Data Structures
## Volume 2 — Concrete Data Structures

---

> *Volume ini adalah bagian kedua dari seri Struktur Data Python.*
> *Volume 1 membangun fondasi klasifikasi dan cara berpikir.*
> *Volume 2 ini menyelami keenam struktur data konkret Python —*
> *bukan untuk menghafal method, tetapi untuk memahami sistem desain*
> *yang menyatukan mereka dalam satu filosofi bahasa.*

---

## Cara Membaca Volume Ini

Ada dua cara memahami struktur data. Cara pertama: menghafal tabel method dan kompleksitas algoritmik. Cara kedua: memahami *logika keberadaan* setiap struktur — mengapa ia lahir, apa yang ia korbankan untuk mendapatkan kekuatannya, dan bagaimana ia berhubungan dengan struktur lain dalam satu ekosistem.

Volume ini mengambil jalan kedua. Setiap bab dimulai dari masalah ontologis: *data seperti apa yang tidak bisa diwakili dengan baik oleh struktur lain?* Dari situ, kita membangun model mental yang kuat, lalu menggunakan kode kecil sebagai *alat observasi* — bukan tutorial, bukan demo syntax, tetapi eksperimen untuk melihat sifat fundamental bersinar.

Yang kita kejar bukan kecepatan menghafal. Yang kita kejar adalah *kecepatan berpikir*: saat kamu melihat suatu masalah, kamu langsung merasakan struktur mana yang secara ontologis cocok, tanpa perlu membandingkan method satu per satu.

---

## Daftar Isi

1. [[#Bab 1: List — Urutan yang Hidup|List — Urutan yang Hidup]]
2. [[#Bab 2: Tuple — Paraf yang Membeku|Tuple — Paraf yang Membeku]]
3. [[#Bab 3: Dict — Arsip Berlabel|Dict — Arsip Berlabel]]
4. [[#Bab 4: Set — Logika Keanggotaan|Set — Logika Keanggotaan]]
5. [[#Bab 5: Frozenset — Himpunan yang Bisa Dijadikan Kunci|Frozenset — Himpunan yang Bisa Dijadikan Kunci]]
6. [[#Bab 6: String — Teks sebagai Sequence Immutable|String — Teks sebagai Sequence Immutable]]
7. [[#Bab 7: Sistem Satu Kesatuan: Enam Struktur, Tiga Gaya Berpikir|Sistem Satu Kesatuan: Enam Struktur, Tiga Gaya Berpikir]]

---

# Bab 1: List — Urutan yang Hidup

## Kenapa Struktur Ini Ada

Semua program, pada titik tertentu, menghadapi pertanyaan yang sama: *bagaimana menampung sekumpulan data yang jumlahnya tidak diketahui saat program ditulis, yang urutannya membawa makna semantik, dan yang isinya akan bertransformasi seiring eksekusi?*

Bayangkan sebuah antrian di loket. Orang datang, orang pergi, seseorang menyelip di tengah karena prioritas, urutan bergeser. Data ini tidak statis — ia *hidup*. Ia bernapas, tumbuh, menyusut, berubah bentuk. Menyimpannya dalam wadah berukuran tetap adalah penyangkalan terhadap realitas data yang sebenarnya.

List hadir sebagai jawaban ontologis untuk kebutuhan ini. Ia bukan sekadar "array yang bisa diubah" — ia adalah representasi Python terhadap gagasan *koleksi dinamis yang berpikir dalam posisi*.

## Intuisi: Gulungan Pergamen yang Terus Memanjang

Bayangkan sebuah gulungan pergamen ajaib. Kamu menulis baris pertama di posisi 0, baris kedua di posisi 1, dan seterusnya. Gulungan ini tidak pernah kehabisan ruang — saat kamu butuh baris baru di akhir, ia memanjang sendiri. Saat kamu butuh menyisipkan baris di tengah, ia membuat celah dengan menggeser semua baris di bawahnya. Setiap baris selalu punya nomor posisinya, dan nomor itu adalah cara utama kamu berkomunikasi dengannya.

```
[0] belajar Python
[1] olahraga
[2] baca buku
[3] memasak       ← baru ditambahkan, gulungan memanjang
```

Yang membuat gulungan ini istimewa bukan hanya kemampuan memanjang. Yang istimewa adalah bahwa *posisi adalah identitas*. Mengubah urutan berarti mengubah data itu sendiri. "Olahraga" di posisi 1 adalah fakta yang berbeda dari "olahraga" di posisi 3.

---

## Karakteristik Fundamental

### 1. Organization Style: Sequence

List adalah *sequence* — struktur yang mengorganisir data dalam urutan linear, di mana posisi elemen adalah bagian tak terpisahkan dari maknanya. Sequence berpikir dalam dimensi *posisi*: elemen pertama, elemen kedua, elemen terakhir. Ini adalah gaya organisasi yang paling alami bagi manusia karena mencerminkan pengalaman temporal dan spasial: antrian, daftar, urutan langkah.

Ketika kamu memilih list, kamu secara implisit menyatakan: "urutan di sini bukan kebetulan — urutan adalah data."

### 2. Mutability: Modifikasi sebagai Perubahan Identitas Internal

*Mutable* berarti objek bisa berubah setelah dibuat. Tapi dalam Python, ini memiliki dimensi yang lebih dalam karena semua variabel adalah referensi. Ketika kamu memodifikasi list, kamu tidak mengganti objek — kamu mengubah *isi* objek yang sama di memori.

```python
daftar = ["senin", "selasa", "rabu"]
print(id(daftar))        # 140234001234000

daftar.append("kamis")
daftar[0] = "SENIN"

print(id(daftar))        # 140234001234000 — identitas sama
print(daftar)            # ['SENIN', 'selasa', 'rabu', 'kamis']
```

Identitas (`id`) tidak berubah. Yang berubah adalah *state* internal. Ini adalah esensi mutable: modifikasi terjadi *in-place*. Konsekuensinya adalah fenomena *aliasing* — dua nama merujuk ke satu objek yang sama, dan perubahan melalui satu nama terlihat oleh nama lain:

```python
a = [1, 2, 3]
b = a          # b merujuk ke objek yang sama

b.append(99)
print(a)       # [1, 2, 3, 99] — a ikut berubah
print(a is b)  # True
```

Aliasing bukan bug — ia adalah konsekuensi langsung dari mutability yang bertemu dengan sistem referensi Python. Ia menjadi bug hanya ketika programmer lupa bahwa list adalah objek bersama, bukan nilai terisolasi.

Mengapa Python mendesain list sebagai mutable? Karena data nyata berubah. Log aktivitas, buffer pembacaan, antrian proses — semua ini adalah entitas yang secara ontologis dinamis. Membuat salinan baru setiap kali ada perubahan kecil akan mengkhianati realitas data yang hidup.

### 3. Ordering: Urutan sebagai Kontrak Semantik

List adalah *ordered* — dan ini bukan sekadar "Python mengingat urutan". Ini adalah *kontrak perilaku*: dua list dengan elemen identik tapi urutan berbeda adalah dua data yang berbeda.

```python
langkah_a = ["nyalakan komputer", "buka browser", "login"]
langkah_b = ["login", "nyalakan komputer", "buka browser"]

print(langkah_a == langkah_b)   # False
```

Ordered berarti iterasi bersifat deterministik: elemen selalu muncul dari posisi terendah ke tertinggi. Ini memungkinkan pola berpikir seperti "proses semua item dari awal sampai akhir" tanpa kekhawatiran urutan acak.

### 4. Indexing / Keying: Akses melalui Posisi Numerik

Karena ordered, setiap elemen mendapat indeks integer dimulai dari `0`. Indeks adalah mekanisme akses utama list — bukan nama, bukan kunci, tetapi posisi.

```python
buah = ["apel", "mangga", "jeruk", "durian"]

print(buah[0])    # "apel" — elemen pertama
print(buah[-1])   # "durian" — indeks negatif menghitung dari belakang
```

Indeks negatif (`-1`, `-2`, ...) adalah desain elegan yang menunjukkan pemahaman Python tentang pola penggunaan: programmer sering butuh elemen terakhir tanpa tahu panjang list. `-1` adalah abstraksi yang mengatakan "terakhir" secara universal.

Slicing (`[mulai:stop:langkah]`) adalah ekstensi dari indexing — kemampuan mengambil sub-sequence dalam satu operasi. Yang penting untuk dipahami: slicing selalu menghasilkan *objek baru*. Ia tidak memotong list asli, melainkan menyalin bagian yang diminta.

```python
angka = [10, 20, 30, 40, 50, 60]
print(angka[1:4])    # [20, 30, 40] — list baru
print(angka[::-1])   # [60, 50, 40, 30, 20, 10] — list baru yang terbalik
```

### 5. Uniqueness: Duplikat Dibolehkan

List tidak memiliki konsep keunikan. Elemen yang sama bisa muncul berulang, dan setiap kemunculan adalah entitas terpisah karena berada di posisi berbeda:

```python
absen = ["Ali", "Budi", "Ali", "Citra", "Budi", "Ali"]
print(len(absen))          # 6 — semua dihitung
print(absen.count("Ali"))  # 3 — Ali muncul tiga kali di posisi berbeda
```

Ini bukan kelemahan — ini adalah kebebasan. Data nyata seringkali punya pengulangan yang bermakna: dua transaksi dengan nilai sama, kata yang muncul berulang dalam teks, pengukuran sensor yang kebetulan identik. List membiarkan realitas ini terekspresi tanpa distorsi.

### 6. Hashability: Tidak Bisa Dihash

Karena mutable, list tidak bisa di-hash. Hash membutuhkan stabilitas nilai — jika hash dari suatu objek berubah setelah ia dimasukkan ke dict atau set, struktur tersebut tidak akan pernah menemukannya lagi. Karena list bisa berubah kapan saja, Python secara prinsip menolak untuk menghashnya.

```python
hash([1, 2, 3])   # TypeError: unhashable type: 'list'
```

Konsekuensi konseptual: list tidak bisa menjadi key dictionary, tidak bisa menjadi elemen set, dan tidak bisa digunakan dalam konteks apa pun yang membutuhkan stabilitas nilai.

### 7. Internal Mechanism: Dynamic Array

Di balik layar, list diimplementasikan sebagai *dynamic array* — array berkontinu di memori yang ukurannya bisa berubah. Ketika kapasitas penuh, Python mengalokasikan array baru yang lebih besar (biasanya 1.125x atau 2x untuk list kecil, strategi yang lebih halus untuk list besar) dan menyalin semua referensi elemen ke sana.

Desain ini menjelaskan dua perilaku kritis:

Pertama, `append()` di akhir sangat cepat rata-rata — *O(1) amortized* — karena alokasi ulang jarang terjadi. Kedua, `insert()` atau `delete()` di tengah sangat lambat — *O(n)* — karena semua elemen setelah titik modifikasi harus digeser untuk menjaga kontinuitas array.

```python
besar = list(range(100_000))

# Cepat: tambah di akhir, tidak ada elemen yang perlu digeser
besar.append(999)

# Lambat: sisipkan di posisi 0, 100.000 elemen harus bergeser ke kanan
besar.insert(0, -1)
```

Memahami internal mechanism ini mengubah cara kamu menulis kode: kamu tidak lagi "menghindari insert di awal" karena "aturan performa", tetapi karena kamu memahami bahwa insert di awal bertentangan dengan mekanisme kontinuitas array.

### 8. Consequences

Ketujuh karakteristik di atas menghasilkan konsekuensi sistemik:

- **Aliasing sebagai default**: Menugaskan list ke variabel baru tidak menyalin — ia membuat alias. Ini memudahkan berbagi data tetapi membutuhkan kesadaran tentang kepemilikan bersama.
- **Slicing sebagai salinan dangkal**: `[:]` membuat salinan baru, tapi hanya salinan referensi. Jika elemennya objek mutable, salinan dan asli masih berbagi objek internal yang sama.
- **Pencarian linear**: `x in list` harus memeriksa elemen satu per satu dari awal — O(n) — karena tidak ada mekanisme selain posisi untuk menemukan data.

---

## Consequences & Behavior

```python
# Behavior 1: Modifikasi in-place dan identitas yang bertahan
antrean = ["A", "B", "C"]
print(id(antrean))
antrean.append("D")
antrean.insert(1, "X")
print(id(antrean))       # Sama — objek tidak diganti
print(antrean)           # ['A', 'X', 'B', 'C', 'D']

# Behavior 2: Aliasing — dua nama, satu objek, satu nasib
def tambah_item(lst, item):
    lst.append(item)     # memodifikasi objek asli yang direferensikan

belanja = ["susu"]
tambah_item(belanja, "telur")
print(belanja)           # ['susu', 'telur'] — berubah!

# Behavior 3: Slicing membuat salinan baru (tapi dangkal)
asli = [1, 2, [3, 4]]
salinan = asli[:]
salinan[0] = 99
salinan[2].append(5)
print(asli)              # [1, 2, [3, 4, 5]] — elemen nested ikut berubah!

# Behavior 4: Perbandingan mempertimbangkan urutan sebagai identitas
print([1, 2, 3] == [1, 2, 3])   # True
print([1, 2, 3] == [3, 2, 1])   # False — urutan berbeda = identitas berbeda
```

---

## Relationship to Other Structures

List berbagi *organization style* dengan tuple dan string: ketiganya adalah sequence. Mereka berpikir dalam posisi, mendukung indexing, slicing, iterasi berurutan, dan operator `in` dengan semantik pencarian linear (untuk list; untuk tuple dan string juga linear).

Tapi list adalah satu-satunya sequence *mutable* di antara ketiga itu. Tuple dan string adalah sequence *immutable*. Perbedaan mutability ini adalah garis pemisah yang paling dalam: ia membuat list tidak bisa di-hash, sementara tuple dan string bisa.

Hubungan list-tuple adalah hubungan komplementer yang paling simetris di Python: pilih list ketika data secara ontologis perlu berubah (antrian, koleksi hasil, buffer); pilih tuple ketika data adalah fakta final yang seharusnya stabil (koordinat, tanggal, record).

List juga berdiri di sisi berlawanan dari dict dan set. Di mana list berpikir "posisi ke-N", dict berpikir "label X", dan set berpikir "apakah X ada?" Tiga gaya berpikir ini mencakup hampir semua kebutuhan pengorganisasian data.

---

## Trade-Off

```
Yang diberikan:
  ✓ Fleksibilitas ontologis penuh — data bisa tumbuh, menyusut, berubah
  ✓ Urutan sebagai kontrak — posisi bermakna dan terjaga
  ✓ Akses indeks langsung — O(1) untuk read/write berbasis posisi
  ✓ Ekspresivitas slicing untuk manipulasi sub-sequence
  ✓ Kebebasan duplikat — merepresentasikan realitas data yang berulang
  ✓ Tipe campuran — satu list bisa menampung string, integer, objek

Yang dikorbankan:
  ✗ Tidak bisa di-hash — tidak bisa menjadi key dict atau elemen set
  ✗ Pencarian tanpa indeks lambat — O(n) linear search
  ✗ Insert/delete di tengah lambat — O(n) karena pergeseran array
  ✗ Aliasing bisa menjadi sumber bug halus jika tidak disadari
  ✗ Overhead memori lebih besar dari array typed karena dynamic resizing
```

---

> **List** adalah tipe data *sequence* yang bersifat *mutable*, *ordered*, dan *indexed*. Diimplementasikan sebagai *dynamic array*, list mendukung modifikasi in-place dengan identitas objek yang bertahan, namun memerlukan kesadaran tentang aliasing. List adalah wadah untuk data yang urutannya bermakna dan isinya secara ontologis hidup — berubah seiring waktu.

---

# Bab 2: Tuple — Paraf yang Membeku

## Kenapa Struktur Ini Ada

Ada kategori data di dunia nyata yang secara konseptual *seharusnya tidak berubah*. Koordinat GPS sebuah kota, tanggal lahir seseorang, nilai RGB dari warna, hasil pengukuran sensor pada satu waktu. Data-data ini bukan hanya *kebetulan* stabil — mereka *dimaksudkan* untuk stabil. Mengubahnya bukan edit, tetapi kesalahan.

Jika kita menyimpan data semacam ini dalam list, kita kehilangan jaminan ontologis. List terbuka untuk perubahan — sifatnya yang demikian berarti tidak ada yang mencegah kode lain, atau bahkan kita sendiri di masa depan, secara tidak sengaja merusak fakta yang seharusnya final.

Tuple hadir untuk memberikan *jaminan arsitektural*: data ini telah diparaf, tidak bisa diubah, dan kamu bisa mengandalkan stabilitasnya dalam semua konteks — sebagai key dictionary, sebagai elemen set, sebagai argumen fungsi yang aman.

## Intuisi: Dokumen yang Sudah Ditandatangani

Bayangkan sebuah dokumen resmi yang telah ditandatangani dan diparaf. Isinya mungkin terdiri dari beberapa bagian — nama, tanggal, nilai — dan setiap bagian punya posisinya sendiri. Dokumen ini bisa kamu baca, kamu salin, kamu gunakan sebagai referensi. Tapi kamu tidak bisa mengubah isinya tanpa membuat dokumen baru. Paraf itu adalah jaminan integritas.

Tuple adalah dokumen semacam itu dalam dunia Python. Ketika programmer lain melihat tuple, mereka membaca sinyal semantik yang jelas: "data ini adalah fakta final."

```python
# Koordinat: fakta geografis yang tidak berubah
bandung = (6.9175, 107.6191)

# Daftar belanja: entitas dinamis yang memang perlu berubah
belanja = ["susu", "telur", "roti"]
```

Memilih tuple untuk koordinat bukan optimasi — itu adalah *deklarasi intent*. Kamu mengatakan kepada kode, kepada rekan kerja, dan kepada dirimu di masa depan: "ini adalah konstanta."

---

## Karakteristik Fundamental

### 1. Organization Style: Sequence

Seperti list, tuple adalah *sequence*. Ia mengorganisir data dalam urutan linear, di mana posisi membawa makna. Tuple berpikir dalam posisi — elemen pertama, kedua, ketiga — dengan cara yang identik dengan list dari sudut pandang akses.

### 2. Mutability: Immutable — Beku pada Struktur

*Immutable* berarti struktur tuple tidak bisa diubah setelah dibuat. Tidak ada penugasan ke indeks, tidak ada `append()`, tidak ada operasi apapun yang memodifikasi tuple yang sudah ada.

```python
titik = (3, 5)
titik[0] = 10    # TypeError: 'tuple' object does not support item assignment
```

Tapi ada nuansa filosofis yang sering disalahpahami: immutability tuple adalah *shallow*, bukan *deep*. Yang dibekukan adalah *referensi* di dalam tuple — bukan *isi* objek yang dirujuk. Jika tuple berisi list, list itu sendiri masih bisa dimutasi:

```python
data = ([1, 2], [3, 4])
data[0].append(99)         # Valid — list di dalam tuple berubah
print(data)                # ([1, 2, 99], [3, 4])

data[0] = [10, 20]         # TypeError — referensi di tuple tidak bisa diganti
```

Perbedaan ini mengungkapkan filosofi Python tentang immutability: tuple menjamin *struktur* — susunan referensi — tidak berubah. Ia tidak menjamin *rekursif immutability* kecuali semua elemennya juga immutable. Ini adalah desain yang pragmatis: menjamin struktur tetap sudah cukup untuk hashability dan keamanan referensi, tanpa memaksakan biaya rekursif yang mahal.

Mengapa immutability itu berharga?

**Keamanan referensial**: Kamu bisa melempar tuple ke fungsi mana pun dan yakin bahwa fungsi itu tidak akan merusak struktur data asli. Ia mungkin merusak objek mutable di dalamnya, tapi susunan elemen tuple tidak akan berubah.

**Stabilitas hash**: Karena struktur tidak berubah, hash dari tuple bisa dihitung sekali dan diandalkan selamanya. Ini membuka pintu ke penggunaan sebagai key dictionary.

### 3. Ordering: Terurut dan Tetap

Seperti list, tuple menjaga urutan elemen secara ketat. Tapi dalam tuple, urutan memiliki nuansa tambahan: karena tidak bisa diubah, urutan adalah *kontrak permanen*. Jika tuple merepresentasikan `(tahun, bulan, hari)`, posisi 0 selalu tahun, posisi 1 selalu bulan, posisi 2 selalu hari — dan ini tidak akan pernah berubah secara tidak sengaja.

```python
tanggal = (2024, 8, 17)
# tahun=2024, bulan=8, hari=17 — posisi adalah semantik
```

### 4. Indexing / Keying: Akses Posisi seperti List

Tuple mendukung indexing dan slicing dengan semantik yang identik dengan list:

```python
rgb = (255, 128, 0)
r = rgb[0]        # 255
sub = rgb[:2]     # (255, 128) — tuple baru
```

### 5. Uniqueness: Duplikat Dibolehkan

Tuple tidak menuntut keunikan. `(1, 1, 1)` adalah tuple yang valid. Keunikan bukan bagian dari kontrak tuple — itu adalah domain set.

### 6. Hashability: Kunci yang Membuka Dunia Mapping

Ini adalah karakteristik paling transformatif dari tuple. *Hashable* berarti objek bisa diubah menjadi integer hash yang stabil, yang digunakan Python untuk menyimpan dan menemukan objek dalam hash table (dict dan set).

Syarat hashability adalah stabilitas nilai. Karena tuple immutable, ia memenuhi syarat ini — dengan catatan: *semua elemennya juga harus hashable*.

```python
# Tuple bisa menjadi key dictionary
jarak = {
    ("Jakarta", "Bandung")  : 150,
    ("Jakarta", "Surabaya") : 800,
}

print(jarak[("Jakarta", "Bandung")])   # 150

# Tuple dengan elemen tidak hashable juga tidak hashable
t1 = (1, 2, 3)        # hashable
t2 = (1, [2, 3])      # tidak hashable — mengandung list
hash(t2)              # TypeError: unhashable type: 'list'
```

Kemampuan ini membuat tuple menjadi jembatan antara dunia sequence dan dunia mapping: kamu bisa menggunakan pasangan nilai, koordinat multidimensi, atau state komposit sebagai key dictionary. Tanpa tuple, dunia mapping Python akan sangat terbatas.

### 7. Internal Mechanism: Fixed-Size Array

Tuple diimplementasikan sebagai array berkontinu dengan ukuran tetap. Karena ukurannya tidak berubah, Python bisa mengalokasikan memori yang lebih kompak dibandingkan list (yang harus menyimpan ruang tambahan untuk pertumbuhan). Tuple juga tidak perlu menyimpan kapasitas ekstra atau logika resizing.

Konsekuensinya: tuple lebih hemat memori dari list dengan elemen yang sama, dan iterasi tuple sedikit lebih cepat karena tidak ada overhead kapasitas dinamis.

### 8. Consequences

Immutability tuple menghasilkan pola-pola desain yang unik:

- **Unpacking**: Karena urutan tetap dan tidak berubah, Python bisa menawarkan *unpacking* — penugasan elemen tuple ke variabel terpisah dalam satu operasi. Ini adalah cara menamai posisi-posisi tuple tanpa harus mengingat indeks numerik.

```python
titik = (4, 7)
x, y = titik           # unpacking — menamai posisi 0 dan 1

def statistik(data):
    return min(data), max(data), sum(data) / len(data)

nilai_min, nilai_max, rata = statistik([85, 92, 78, 95])
```

- **Return value multi-part**: Tuple adalah mekanisme Python untuk mengembalikan beberapa nilai dari fungsi secara bersih. Fungsi mengembalikan satu tuple; caller meng-unpack-nya.

- **Memory efficiency**: Program yang membuat banyak koleksi kecil yang tidak berubah (seperti koordinat pixel) akan lebih hemat memori dengan tuple.

---

## Consequences & Behavior

```python
# Behavior 1: Immutability — modifikasi langsung ditolak
koordinat = (6.9175, 107.6191)
try:
    koordinat[0] = 7.0
except TypeError as e:
    print(f"Error: {e}")
# 'tuple' object does not support item assignment

# Behavior 2: Shallow immutability — isi mutable bisa berubah
campur = (1, [10, 20], "halo")
campur[1].append(30)
print(campur)            # (1, [10, 20, 30], 'halo')
# tapi: campur[1] = [99]  # TypeError — referensi tidak bisa diganti

# Behavior 3: Unpacking sebagai cara menamai posisi
nama, umur, kota = ("Brata", 20, "Cirebon")
print(f"{nama}, {umur} tahun, dari {kota}")

# Behavior 4: Sebagai key dictionary — jembatan ke dunia mapping
cache = {}
cache[(3, 4)]  = 5       # koordinat sebagai key
cache[(5, 12)] = 13
print(cache[(3, 4)])     # 5

# Behavior 5: Perbandingan memori — lebih hemat dari list
import sys
lst = [1, 2, 3, 4, 5]
tpl = (1, 2, 3, 4, 5)
print(sys.getsizeof(lst))   # lebih besar
print(sys.getsizeof(tpl))   # lebih kecil
```

---

## Relationship to Other Structures

Tuple dan list adalah *pasangan simetris* dalam sistem Python: mereka berbagi organization style yang identik (sequence, ordered, indexed), tetapi berbeda pada satu sumbu fundamental — *mutability*.

- **list**: sequence mutable — tidak hashable — untuk data hidup
- **tuple**: sequence immutable — hashable (jika elemen hashable) — untuk data final

Hubungan ini adalah cermin dari hubungan set-frozenset. Python secara sistematis menawarkan pasangan untuk setiap organization style utama: satu versi mutable untuk fleksibilitas, satu versi immutable untuk stabilitas dan hashability.

Tuple juga lebih dekat ke *string* daripada ke *list* dari sudut mutability. Keduanya adalah sequence immutable yang hashable. Perbedaannya: string adalah sequence *terspesialisasi* untuk teks Unicode, sementara tuple adalah sequence *generik* untuk tipe campuran.

---

## Trade-Off

```
Yang diberikan:
  ✓ Immutable — jaminan struktural terhadap perubahan tidak disengaja
  ✓ Hashable — bisa menjadi key dict atau elemen set (jika elemen hashable)
  ✓ Lebih hemat memori dari list karena ukuran tetap
  ✓ Sinyal semantik yang kuat: "data ini final"
  ✓ Unpacking yang elegan untuk menamai posisi
  ✓ Aman untuk dibagikan antar fungsi dan thread

Yang dikorbankan:
  ✗ Tidak bisa dimodifikasi — tidak cocok untuk koleksi yang dinamis
  ✗ Tidak punya method penambahan/penghapusan elemen
  ✗ Shallow immutability — elemen mutable di dalamnya masih bisa berubah
  ✗ Tidak bisa digunakan sebagai akumulator hasil iterasi yang bertambah
```

---

> **Tuple** adalah tipe data *sequence* yang bersifat *immutable*, *ordered*, dan *indexed*. Karena immutable, tuple bersifat *hashable* (syarat: semua elemen hashable) dan dapat digunakan sebagai key dictionary. Tuple adalah wadah untuk data yang secara ontologis adalah satu unit tetap — fakta, koordinat, record, hasil fungsi multi-nilai — yang tidak seharusnya berubah sepanjang hidup program.

---

# Bab 3: Dict — Arsip Berlabel

## Kenapa Struktur Ini Ada

Bayangkan kamu menyimpan data mahasiswa menggunakan list:

```python
mhs = ["2024001", "Brata Yudha", "Informatika", 3.85, "Cirebon"]
```

Ini bekerja secara mekanis, tapi gagal secara semantik. `mhs[3]` adalah IPK — tapi kamu harus *mengingat* bahwa posisi 3 adalah IPK. Jika struktur berubah (misalnya ditambahkan field `angkatan` di posisi 2), semua kode yang mengakses posisi 2, 3, 4 harus diperbaiki. Satu perubahan struktur merusak semua dependensi.

Masalah fundamentalnya: data ini punya *nama alami* untuk setiap field, tapi list memaksamu mengganti nama itu dengan nomor posisi yang rapuh. Kamu berpikir dalam "NIM", "nama", "IPK" — tapi wadahnya memaksamu berpikir dalam "0", "1", "3".

Dict hadir untuk menyelesaikan ketidakcocokan ontologis ini. Ia berkata: *akses data dengan nama yang ia miliki, bukan dengan nomor yang kamu tetapkan*.

```python
mhs = {
    "nim"    : "2024001",
    "nama"   : "Brata Yudha",
    "jurusan": "Informatika",
    "ipk"    : 3.85,
    "kota"   : "Cirebon",
}
print(mhs["ipk"])    # 3.85 — akses via nama, bukan via posisi
```

## Intuisi: Lemari Arsip Berlabel

Bayangkan lemari arsip dengan banyak laci. Setiap laci punya label di luarnya: "faktur 2023", "kontrak aktif", "korespondensi". Kamu tidak pernah membuka laci nomor 3 — kamu membuka laci berlabel "kontrak aktif". Label adalah cara alami manusia mengorganisir informasi: nama mengidentifikasi isi.

```
┌──────────────┬──────────────────────┐
│     Key      │        Value         │
├──────────────┼──────────────────────┤
│  "nama"      │  "Brata Yudha"       │
│  "ipk"       │  3.85                │
│  "jurusan"   │  "Informatika"       │
│  "kota"      │  "Cirebon"           │
└──────────────┴──────────────────────┘
         Akses via label, bukan posisi
```

Key adalah label. Value adalah isi laci. Dan setiap label dalam satu lemari harus unik — tidak mungkin ada dua laci berlabel identik.

---

## Karakteristik Fundamental

### 1. Organization Style: Mapping

Dict adalah *mapping* — struktur yang mengorganisir data sebagai *relasi* antara identifier (key) dan nilai yang terkait (value). Ini adalah gaya berpikir yang fundamental berbeda dari sequence.

Di mana sequence bertanya "apa di posisi ke-N?", mapping bertanya "apa nilai dari label X?". Sequence berpikir dalam *dimensi spasial linear*; mapping berpikir dalam *dimensi asosiatif*. Keduanya adalah cara mengorganisir data, tetapi untuk ontologi yang berbeda: sequence untuk urutan, mapping untuk atribut berlabel.

### 2. Mutability: Mutable, tapi Key adalah Konstanta

Dict bersifat mutable — kamu bisa menambah pasangan key-value baru, mengubah value untuk key yang sudah ada, atau menghapus key. Tapi ada batasan filosofis: *key itu sendiri, setelah masuk, tidak bisa berubah* (karena key harus hashable).

```python
profil = {"nama": "Brata"}
profil["email"] = "brata@email.com"   # menambah pasangan baru
profil["nama"] = "Brata Yudha"        # mengubah value
del profil["email"]                   # menghapus key
```

Value boleh apa saja — list, dict lain, fungsi, `None`. Value boleh berulang: dua key berbeda boleh menunjuk ke value yang sama. Tapi key harus unik dalam satu dict. Jika key yang sama dimasukkan dua kali, yang terakhir menimpa yang sebelumnya:

```python
d = {"a": 1, "b": 2, "a": 3}
print(d)   # {'a': 3, 'b': 2} — "a" hanya ada sekali
```

Ini bukan bug — ini adalah konsekuensi dari mekanisme hash table: satu key menempati satu slot, dan slot itu hanya bisa menyimpan satu value.

### 3. Ordering: Insertion Ordered, Bukan Sequence Ordered

Sejak Python 3.7, dict menjamin urutan iterasi mengikuti urutan penyisipan. Ini adalah properti yang berguna dan bisa diandalkan.

Tapi ada jebakan konseptual yang fatal jika tidak dipahami: *insertion ordered tidak membuat dict menjadi sequence*. Urutan penyisipan adalah detail implementasi yang menentukan iterasi, bukan mekanisme akses. Kamu tidak bisa mengakses "elemen ke-0" dari dict — karena dict tidak berpikir dalam posisi.

```python
d = {}
d["z"] = 26
d["a"] = 1
d["m"] = 13

print(list(d.keys()))   # ['z', 'a', 'm'] — urutan penyisipan

# Tapi ini tidak bisa:
d[0]   # KeyError — 0 adalah key, bukan indeks posisi
```

Dict mengingat *kapan* key dimasukkan untuk keperluan iterasi, tapi mekanisme aksesnya tetap *apa* key-nya. Ini seperti buku telepon yang tersusun kronologis berdasarkan waktu pendaftaran: kamu tahu siapa yang daftar pertama, tapi kamu tetap mencari berdasarkan nama, bukan nomor urut pendaftaran.

### 4. Indexing / Keying: Akses via Key, Bukan Posisi

Mekanisme akses dict adalah *keying* — menggunakan key sebagai identifier untuk menemukan value. Key adalah "nama" data, bukan "alamat" data.

```python
konfig = {"tema": "dark", "font_size": 14}
print(konfig["tema"])        # akses via key
print(konfig.get("bahasa"))  # None — key tidak ada, tidak error
```

Tidak ada konsep "elemen ke-N" dalam dict. Jika kamu butuh mengakses berdasarkan posisi, kamu telah memilih struktur yang salah — atau kamu sedang memaksakan paradigma sequence ke dalam paradigma mapping.

### 5. Uniqueness: Key Unik, Value Bebas

Uniqueness dalam dict bersifat *asimetris*: key harus unik (karena key adalah identifier), tapi value tidak perlu unik. Ini mencerminkan realitas dunia nyata: dua orang berbeda (key berbeda) bisa tinggal di kota yang sama (value sama).

### 6. Hashability: Key Harus Hashable — Kontrak yang Tidak Bisa Ditawar

Ini adalah syarat paling keras dict. *Semua key harus hashable*. Mengapa? Karena dict diimplementasikan sebagai hash table, dan hash table bekerja dengan menghitung hash dari key untuk menentukan di mana value akan disimpan.

Jika key bisa berubah (mutable), hash-nya bisa berubah — dan dict tidak akan pernah menemukan value-nya lagi. Seluruh sistem lookup runtuh. Oleh karena itu, list tidak bisa menjadi key dict: list mutable, hash-nya tidak stabil, dan Python secara prinsip menolaknya.

```python
d = {}
d["nama"] = "Brata"       # ✓ string hashable
d[42]     = "angka"       # ✓ integer hashable
d[(1, 2)] = "titik"       # ✓ tuple hashable

d[[1, 2]] = "poin"        # ✗ TypeError: unhashable type: 'list'
```

Syarat ini menjelaskan mengapa tuple begitu penting: tuple adalah satu-satunya built-in sequence yang bisa menjadi key dict, menjadikannya jembatan antara dunia berurutan dan dunia berlabel.

### 7. Internal Mechanism: Hash Table

Di balik layar, dict adalah *hash table* — array sparse di mana posisi penyimpanan value ditentukan oleh hash dari key. Python menggunakan teknik *open addressing dengan probing* untuk menangani collision (ketika dua key memiliki hash yang sama atau berbeda tetapi memetakan ke slot yang sama).

Mekanisme ini menjelaskan mengapa lookup, penyisipan, dan penghapusan dict adalah *O(1) rata-rata*: hash memberikan "alamat perkiraan" langsung ke lokasi data, tanpa perlu memeriksa elemen satu per satu.

Tapi O(1) ini adalah *konsekuensi* dari pilihan desain hash table, bukan alasan utama dict ada. Alasan utama adalah akses berbasis nama; kecepatan adalah bonus arsitektural.

Konsekuensi mekanisme lainnya: dict lebih boros memori dari list. Hash table membutuhkan *load factor* (biasanya dijaga di bawah 2/3) dan slot kosong untuk mengakomodasi collision. Kamu membayar memori ekstra untuk mendapatkan kecepatan lookup.

### 8. Consequences

- **Lookup O(1)**: Kecepatan konstan adalah konsekuensi hash table. Tapi ini rata-rata — dalam kasus collision ekstrem (serangan hash collision yang sengaja dibuat), bisa degradasi ke O(n).
- **Key stability**: Setelah key dimasukkan, nilai hash-nya harus tetap. Ini adalah kontrak tak tertulis yang Python tidak bisa periksa secara runtime untuk objek custom, tetapi diharapkan oleh programmer.
- **No positional access**: Dict sengaja tidak menyediakan akses posisi karena itu akan mengkhianati paradigma mapping.

---

## Consequences & Behavior

```python
# Behavior 1: Akses via key — eksplisit dan langsung
profil = {"nama": "Brata", "kota": "Cirebon", "ipk": 3.85}
print(profil["nama"])              # "Brata"
print(profil.get("email"))         # None — aman jika key tidak ada
print(profil.get("email", "-"))    # "-" — default value

# Behavior 2: Key unik — yang terakhir menang
d = {"a": 1, "a": 2, "a": 3}
print(d)   # {'a': 3}

# Behavior 3: Modifikasi setelah dibuat
profil["email"] = "brata@email.com"
profil["ipk"]   = 3.90
del profil["kota"]
print(profil)

# Behavior 4: Iterasi mengikuti urutan penyisipan (3.7+)
d = {}
d["z"] = 26
d["a"] = 1
d["m"] = 13
print(list(d.keys()))   # ['z', 'a', 'm']

# Behavior 5: 'in' mengecek KEY, bukan value
print("nama" in profil)              # True
print("brata@email.com" in profil)   # False — ini adalah value, bukan key
```

---

## Relationship to Other Structures

Dict adalah *mapping* — organization style yang fundamental berbeda dari *sequence* (list, tuple, string). Di mana sequence berpikir "posisi ke-N", dict berpikir "label X". Keduanya adalah cara mengorganisir data, tetapi untuk masalah yang berbeda: sequence untuk urutan, mapping untuk atribut.

Hubungan paling erat dict adalah dengan *set*. Keduanya menggunakan hash table. Keduanya membutuhkan elemen/key yang hashable. Dalam implementasi CPython, set pada dasarnya adalah "dict tanpa value" — keduanya berbagi kode inti. Ini bukan sekadar implementasi; ini adalah refleksi bahwa set dan dict adalah dua ekspresi dari satu mekanisme dasar: hash-based storage.

Dict juga berdiri sebagai pelengkap list dalam banyak pola Python: list menyimpan urutan item yang seragam (semua mahasiswa), dict menyimpan atribut dari satu entitas (detail satu mahasiswa). List of dicts adalah pola universal yang mencerminkan pemisahan antara "koleksi" (list) dan "atribut" (dict).

---

## Trade-Off

```
Yang diberikan:
  ✓ Lookup sangat cepat — O(1) rata-rata karena hash table
  ✓ Akses berbasis nama — kode lebih ekspresif dan tahan perubahan struktur
  ✓ Key-value relationship — merepresentasikan relasi data secara natural
  ✓ Insertion ordered sejak Python 3.7 — iterasi deterministik

Yang dikorbankan:
  ✗ Key harus hashable — list dan dict lain tidak bisa jadi key
  ✗ Lebih boros memori dari list karena overhead hash table
  ✗ Tidak ada akses berbasis posisi — tidak cocok untuk data berurutan
  ✗ Tidak ada operasi "elemen ke-N" — memaksakan sequence paradigm gagal
```

---

> **Dict** adalah tipe data *mapping* yang menyimpan pasangan *key-value*. Diimplementasikan sebagai *hash table*, dict mendukung lookup, penyisipan, dan penghapusan dalam O(1) rata-rata. Key harus hashable dan unik. Sejak Python 3.7, dict menjaga urutan penyisipan — tapi ini adalah properti iterasi, bukan mekanisme akses. Dict adalah wadah untuk data yang memiliki label alami dan diakses via nama, bukan via posisi.

---

# Bab 4: Set — Logika Keanggotaan

## Kenapa Struktur Ini Ada

Ada kelas masalah di dunia pemrograman di mana pertanyaan yang paling sering diajukan bukan "berapa posisinya?" atau "apa nilai dari label ini?" — melainkan: *"apakah X ada di sini?"*

Bayangkan log akses server dengan ribuan baris, banyak di antaranya berulang karena pengguna yang sama mengakses berkali-kali. Pertanyaannya: *berapa pengguna unik yang mengakses hari ini?* Atau: *apakah pengguna ini sudah pernah muncul?*

Dengan list, kamu harus menulis loop yang memeriksa setiap elemen — O(n) setiap pengecekan. Dengan set, keberadaan adalah sifat bawaan:

```python
log = ["ali", "budi", "ali", "citra", "budi", "dani", "ali"]

# List: manual, lambat, dan tidak ekspresif
unik_list = []
for u in log:
    if u not in unik_list:   # O(n) setiap kali
        unik_list.append(u)

# Set: keunikan adalah aturan fundamentalnya
unik_set = set(log)
print(len(unik_set))    # 5 — ali, budi, citra, dani
```

Set ada karena ada domain data di mana **keunikan bukan kebetulan, melainkan hukum**. Dan di domain itu, pertanyaan fundamental adalah keanggotaan — bukan posisi, bukan label.

## Intuisi: Kotak yang Menolak Duplikat dan Tidak Peduli Urutan

Bayangkan sebuah kotak ajaib dengan dua aturan keras:

1. **Aturan Keunikan**: Ia hanya menyimpan satu salinan dari setiap item. Melempar "apel" dua kali tidak membuatnya punya dua apel — ia tetap punya satu.
2. **Aturan Tanpa Posisi**: Ia tidak peduli item mana yang masuk pertama atau terakhir. Ia tidak punya "posisi 0" atau "posisi terakhir". Yang ia tahu hanya: "apel ada", "mangga ada", "jeruk ada".

```
Memasukkan: "apel", "mangga", "apel", "jeruk", "mangga"

Isi kotak:  { "apel", "mangga", "jeruk" }
            └── keunikan otomatis
            └── urutan tidak relevan
```

Dan kotak ini sangat cepat menjawab satu pertanyaan: "Apakah X ada di dalamnya?" Ia tidak perlu membuka dan memeriksa satu per satu. Ia langsung tahu — hampir instan — karena ia menggunakan mekanisme hash untuk menemukan item.

---

## Karakteristik Fundamental

### 1. Organization Style: Set-Like

Set adalah *set-like structure* — gaya organisasi yang berpikir dalam *keanggotaan* (membership). Di mana sequence berpikir "posisi" dan mapping berpikir "label", set-like berpikir "keberadaan".

Set tidak memiliki konsep urutan. Ia tidak memiliki konsep posisi. Ia tidak memiliki konsep label. Yang ia miliki adalah koleksi elemen di mana pertanyaan yang paling natural adalah: *apakah elemen ini termasuk anggota?*

Ini adalah paradigma ketiga dalam sistem Python, bersanding dengan sequence dan mapping.

### 2. Mutability: Mutable, tapi Elemennya Terbatas

Set bersifat mutable — kamu bisa menambah elemen (`add`), menghapus elemen (`remove`, `discard`), dan melakukan operasi update in-place. Tapi ada batasan yang lebih ketat dari list: *elemen yang ditambahkan harus hashable*.

```python
s = set()
s.add(42)           # ✓ integer hashable
s.add("halo")       # ✓ string hashable
s.add((1, 2))       # ✓ tuple hashable

s.add([1, 2])       # ✗ TypeError: unhashable type: 'list'
```

Batasan ini bukan keisengan — ia adalah konsekuensi langsung dari mekanisme internal set. Karena set menggunakan hash table untuk menentukan keberadaan elemen, elemen harus bisa di-hash. List tidak bisa di-hash karena mutable; oleh karena itu, list tidak bisa menjadi elemen set.

### 3. Ordering: Tidak Terurut — Bukan "Acak", tetapi "Di Luar Kontrak"

Set adalah *unordered*. Ini bukan sekadar "Python tidak menjamin urutan" — ini adalah *penolakan terhadap urutan sebagai konsep*. Urutan bukan bagian dari identitas set. Dua set dengan elemen yang sama dalam urutan berbeda adalah set yang identik:

```python
a = {1, 2, 3}
b = {3, 2, 1}
print(a == b)   # True — urutan tidak relevan
```

Ketika kamu iterasi set, urutan yang kamu lihat adalah artefak implementasi hash table — bergantung pada nilai hash elemen dan urutan penyisipan. Ia bisa berubah antar run program, antar versi Python, atau bahkan antar objek set yang identik.

Ini bukan kekurangan yang "perlu ditolerir". Ini adalah *trade-off arsitektural yang sengaja*: dengan melepaskan urutan, set mendapatkan kecepatan keanggotaan yang luar biasa.

### 4. Indexing / Keying: Tidak Ada — Hanya Keanggotaan

Set tidak mendukung indexing. Tidak ada `s[0]`, tidak ada `s[-1]`, tidak ada slicing. Konsep "posisi ke-N" tidak ada dalam dunia set.

```python
s = {"a", "b", "c"}
s[0]    # TypeError: 'set' object is not subscriptable
```

Yang ada hanya keanggotaan: `x in s`. Ini adalah cara berkomunikasi yang satu-satunya dengan set: bukan "berikan saya elemen ke-N", melainkan "apakah X ada di sini?"

### 5. Uniqueness: Hukum Fundamental

Keunikan bukan fitur tambahan dalam set — ia adalah *definisi*. Set tidak bisa memiliki duplikat secara ontologis. Jika kamu memasukkan elemen yang sudah ada, set mengabaikannya dengan tenang:

```python
s = {3, 1, 4, 1, 5, 9, 2, 6, 5, 3}
print(s)       # {1, 2, 3, 4, 5, 6, 9} — duplikat otomatis hilang
print(len(s))  # 7, bukan 10

s.add(3)       # sudah ada — tidak ada yang berubah
s.add(7)       # belum ada — ditambahkan
```

Mengapa set menghapus duplikat secara otomatis? Karena ia menggunakan hash table. Saat elemen ditambahkan, Python menghitung hash-nya, menemukan slot yang sesuai, dan mengecek apakah slot itu sudah terisi oleh nilai yang sama (dengan equality check). Jika sudah ada, tidak ada yang ditambahkan. Jika belum, elemen baru masuk.

Keunikan ini adalah konsekuensi mekanis dari hash table, bukan aturan yang diterapkan di atasnya.

### 6. Hashability: Elemen Harus Hashable; Set Sendiri Tidak

Semua elemen set harus hashable — alasan mekanismenya sudah dijelaskan di atas. Tapi set itu sendiri *tidak hashable* karena mutable. Kamu tidak bisa memiliki set yang berisi set lain:

```python
a = {1, 2, 3}
b = {4, 5, 6}
c = {a, b}      # TypeError: unhashable type: 'set'
```

Ini adalah konsekuensi langsung: mutable → tidak hashable. Set mutable, sehingga tidak bisa di-hash, sehingga tidak bisa menjadi elemen set lain atau key dict.

Solusinya — yang akan kita bahas di bab berikutnya — adalah frozenset.

### 7. Internal Mechanism: Hash Table

Set diimplementasikan sebagai hash table — array sparse di mana keberadaan elemen ditentukan oleh hash-nya. Ketika kamu mengecek `x in s`, Python menghitung `hash(x)`, memetakannya ke indeks array, dan memeriksa apakah slot tersebut berisi `x` (dengan equality check untuk menangani collision).

Ini menjelaskan dua hal:

Pertama, **keanggotaan O(1) rata-rata**. Hash memberikan alamat langsung; tidak peduli set berisi 10 atau 10 juta elemen, kecepatan pengecekan hampir konstan.

Kedua, **kenapa elemen harus hashable**. Tanpa hash, tidak ada cara menentukan slot. Tanpa stabilitas hash (yang membutuhkan immutability), slot bisa "kehilangan" elemen setelah elemen berubah.

### 8. Consequences

- **O(1) membership testing**: Kecepatan ini adalah konsekuensi hash table, bukan magic. Ia membuat set menjadi struktur pilihan untuk pengecekan keberadaan.
- **Tidak ada duplikat**: Data yang masuk ke set secara otomatis "dibersihkan" dari redundansi.
- **Tidak bisa di-hash**: Karena mutable, set tidak bisa menjadi key dict atau elemen set lain.
- **Tidak ada urutan**: Operasi yang bergantung pada urutan (seperti "ambil elemen pertama" atau "urutkan tanpa sorted()") tidak ada.

---

## Consequences & Behavior

```python
# Behavior 1: Keunikan otomatis — duplikat dilarang oleh desain
s = {1, 2, 2, 3, 3, 3}
print(s)           # {1, 2, 3}
print(len(s))      # 3

# Behavior 2: Unordered — urutan adalah artefak, bukan kontrak
buah = {"apel", "mangga", "jeruk"}
print(buah)        # bisa {'jeruk', 'apel', 'mangga'} atau lainnya

# Behavior 3: Tidak ada indexing
try:
    print(buah[0])
except TypeError as e:
    print(f"Error: {e}")
# 'set' object is not subscriptable

# Behavior 4: Membership testing — O(1), hampir instan
besar = set(range(1_000_000))
print(999_999 in besar)    # True — hampir instan
print(2_000_000 in besar)  # False — hampir instan

# Behavior 5: set kosong — HARUS pakai set(), bukan {}
kosong = {}       # <class 'dict'> — ini dict!
kosong = set()    # <class 'set'> — ini yang benar

# Behavior 6: Elemen harus hashable
try:
    s = {[1, 2], [3, 4]}
except TypeError as e:
    print(f"Error: {e}")
# unhashable type: 'list'
```

---

## Relationship to Other Structures

Set adalah *pasangan simetris* dari frozenset — persis seperti list adalah pasangan dari tuple. Hubungan ini adalah cermin yang konsisten di seluruh sistem Python:

- **list** ↔ **tuple**: sequence mutable vs sequence immutable
- **set** ↔ **frozenset**: set-like mutable vs set-like immutable

Set juga berkerabat sangat dekat dengan **dict**. Keduanya menggunakan hash table. Keduanya membutuhkan hashable elements. Dalam implementasi CPython, set bisa dilihat sebagai "dict yang hanya menyimpan key, tanpa value". Ini bukan sekadar analogi — keduanya memang berbagi kode inti.

Perbedaan fundamental set dengan sequence: sequence berpikir posisi, set berpikir keberadaan. Kamu tidak bisa mengkonversi masalah "urutan bermakna" ke set tanpa kehilangan informasi. Sebaliknya, kamu tidak bisa menggunakan sequence untuk membership testing yang efisien tanpa kehilangan performa.

---

## Trade-Off

```
Yang diberikan:
  ✓ Keunikan otomatis — tidak perlu cek manual, tidak perlu algoritma deduplikasi
  ✓ Membership testing O(1) rata-rata — sangat cepat untuk pengecekan keberadaan
  ✓ Operasi himpunan matematis built-in (union, intersection, difference)
  ✓ Efisien untuk deduplikasi koleksi besar

Yang dikorbankan:
  ✗ Tidak terurut — urutan iterasi tidak bisa diandalkan
  ✗ Tidak ada indexing — tidak bisa "ambil elemen ke-N"
  ✗ Elemen harus hashable — list dan dict tidak bisa jadi elemen
  ✗ Set sendiri tidak hashable — tidak bisa jadi key dict atau elemen set lain
  ✗ Overhead memori hash table — seperti dict, lebih boros dari list
```

---

> **Set** adalah tipe data *set-like* yang menyimpan elemen-elemen unik tanpa urutan tertentu. Diimplementasikan sebagai *hash table*, set mendukung membership testing, penyisipan, dan penghapusan dalam O(1) rata-rata. Elemen harus hashable. Set bersifat mutable, tidak terurut, dan berorientasi pada keanggotaan — bukan pada posisi atau label. Set adalah wadah untuk data di mana pertanyaan fundamental adalah "apakah X ada?", bukan "X ada di posisi berapa?" atau "apa nilai dari X?".

---

# Bab 5: Frozenset — Himpunan yang Bisa Dijadikan Kunci

## Kenapa Struktur Ini Ada

Set sangat berguna untuk logika keanggotaan. Tapi ia punya satu keterbatasan yang sering terasa: karena mutable, set tidak bisa di-hash. Dan karena tidak bisa di-hash, set tidak bisa menjadi key dictionary atau elemen dalam set lain.

Bayangkan kamu membangun sistem rekomendasi yang menyimpan hasil kalkulasi dalam cache. Key cache-nya adalah kombinasi preferensi pengguna — misalnya set genre musik yang disukai. Masalahnya: set tidak bisa menjadi key dict.

```python
cache = {}
preferensi = {"pop", "jazz", "indie"}

cache[preferensi] = ["lagu_a", "lagu_b"]   # TypeError!
# unhashable type: 'set'
```

Frozenset hadir sebagai solusi: versi set yang *membeku* — immutable — dan karena itu *hashable*.

```python
preferensi_frozen = frozenset({"pop", "jazz", "indie"})
cache[preferensi_frozen] = ["lagu_a", "lagu_b"]   # ✓
print(cache[preferensi_frozen])   # ['lagu_a', 'lagu_b']
```

## Intuisi: Hubungan Simetris yang Konsisten

Hubungan frozenset dengan set adalah hubungan yang *identik* dengan hubungan tuple terhadap list. Python secara sistematis menawarkan pasangan untuk setiap organization style utama:

```
Sequence:
  list      →  mutable, tidak hashable, fleksibel
  tuple     →  immutable, hashable, stabil

Set-like:
  set       →  mutable, tidak hashable, fleksibel
  frozenset →  immutable, hashable, stabil
```

Frozenset adalah "tuple-nya set". Ia punya semua sifat set — keunikan elemen, tidak terurut, berbasis hash — tapi tidak bisa dimodifikasi setelah dibuat. Pembekuan ini memungkinkannya untuk di-hash.

---

## Karakteristik Fundamental

### 1. Organization Style: Set-Like

Seperti set, frozenset adalah struktur set-like yang berpikir dalam keanggotaan. Ia tidak memiliki urutan, tidak memiliki posisi, dan tidak memiliki label. Yang ada hanya koleksi elemen unik dan kemampuan mengecek keberadaan.

### 2. Mutability: Immutable — Set yang Membeku

Setelah frozenset dibuat, isinya tidak bisa berubah. Tidak ada `add()`, tidak ada `remove()`, tidak ada `discard()`. Struktur himpunan tersebut dibekukan pada keadaan terakhirnya.

```python
fs = frozenset([1, 2, 3, 4, 5])
fs.add(6)      # AttributeError
fs.remove(1)   # AttributeError
```

Tapi operasi baca tetap tersedia penuh: membership testing, iterasi, ukuran, dan semua operasi himpunan non-mutating.

### 3. Ordering: Tidak Terurut

Seperti set, frozenset tidak menjaga urutan. Iterasi menghasilkan elemen dalam urutan yang tidak terjamin, sama seperti set.

### 4. Indexing / Keying: Tidak Ada

Tidak ada konsep posisi dalam frozenset. Tidak ada `fs[0]`. Akses hanya melalui logika keanggotaan: `x in fs`.

### 5. Uniqueness: Ya — Sama seperti Set

Frozenset menjaga keunikan elemen dengan cara yang identik dengan set. Duplikat tidak mungkin ada.

### 6. Hashability: Hashable — Kunci Utama Keberadaannya

Ini adalah karakteristik yang membedakan frozenset dari set. Karena immutable dan semua elemennya hashable (syarat yang diwarisi dari set), frozenset bisa di-hash.

```python
fs = frozenset([1, 2, 3])
print(hash(fs))   # bekerja — frozenset hashable
```

Hashability ini membuka dua kemampuan transformatif:

**Pertama**, frozenset bisa menjadi key dictionary:

```python
izin = {
    frozenset(["baca", "tulis", "hapus", "admin"]): "Administrator",
    frozenset(["baca", "tulis"])                  : "Editor",
    frozenset(["baca"])                            : "Viewer",
}

hak = frozenset(["baca", "tulis"])
print(izin[hak])   # "Editor"
```

**Kedua**, frozenset bisa menjadi elemen dalam set lain — memungkinkan "set of sets":

```python
kelompok = {
    frozenset(["Ali", "Budi"]),
    frozenset(["Citra", "Dani"]),
    frozenset(["Ali", "Budi"]),    # duplikat — dibuang
}
print(len(kelompok))   # 2
```

Tanpa frozenset, tidak mungkin memiliki koleksi himpunan dalam Python.

### 7. Internal Mechanism: Hash Table Read-Only

Frozenset menggunakan hash table yang identik dengan set — tapi dalam mode read-only. Struktur internalnya tidak bisa dialokasikan ulang, tidak bisa ditambah, tidak bisa dikurangi setelah dibuat.

Karena tidak ada operasi modifikasi, frozenset bisa menghitung hash-nya sekali dan menyimpannya (cache hash). Ini membuat operasi hash frozenset lebih efisien dari tuple dalam beberapa kasus, karena tuple harus menghitung hash dari semua elemen setiap kali `hash()` dipanggil (meskipun CPython juga meng-cache hash tuple).

### 8. Consequences

- **Stabilitas hash**: Frozenset bisa digunakan sebagai key dict dengan aman — hash-nya tidak akan pernah berubah.
- **Set operations return new frozenset**: Union, intersection, difference menghasilkan frozenset baru, tanpa memodifikasi operand.
- **Set of sets**: Hanya frozenset yang memungkinkan hierarki koleksi himpunan.

---

## Consequences & Behavior

```python
# Behavior 1: Frozenset tidak bisa dimodifikasi
fs = frozenset([10, 20, 30])
try:
    fs.add(40)
except AttributeError as e:
    print(f"Error: {e}")
# 'frozenset' object has no attribute 'add'

# Behavior 2: Hashable — bisa jadi key dict
cache = {}
key1 = frozenset(["a", "b", "c"])
key2 = frozenset(["x", "y"])
cache[key1] = "hasil abc"
cache[key2] = "hasil xy"
print(cache[frozenset(["b", "a", "c"])])   # "hasil abc" — urutan tidak penting!

# Behavior 3: Bisa menjadi elemen set — set of sets
koleksi = set()
koleksi.add(frozenset([1, 2]))
koleksi.add(frozenset([3, 4]))
koleksi.add(frozenset([1, 2]))   # duplikat — tidak ditambahkan
print(len(koleksi))   # 2

# Behavior 4: Operasi himpunan menghasilkan frozenset baru
x = frozenset([1, 2, 3])
y = frozenset([2, 3, 4])
z = x | y
print(type(z))   # <class 'frozenset'>
print(x)         # frozenset({1, 2, 3}) — x tidak berubah
```

---

## Relationship to Other Structures

Frozenset adalah "tuple-nya set" — hubungan ini simetris dan konsisten dengan pasangan list-tuple. Dalam sistem besar Python:

- **list** ↔ **tuple**: sequence, beda mutability
- **set** ↔ **frozenset**: set-like, beda mutability

Kapan pun kamu butuh himpunan yang tidak berubah — untuk dijadikan key dict, untuk disimpan dalam set lain, atau sekadar untuk merepresentasikan himpunan fakta yang stabil — frozenset adalah satu-satunya pilihan built-in.

Frozenset juga menunjukkan bahwa **immutability → hashability** adalah hukum universal di Python: tuple (immutable sequence) hashable, frozenset (immutable set-like) hashable, string (immutable sequence) hashable. Set, list, dict (mutable) tidak hashable.

---

## Trade-Off

```
Yang diberikan:
  ✓ Hashable — bisa jadi key dict atau elemen set
  ✓ Immutable — aman dari perubahan tidak sengaja, thread-safe
  ✓ Semua operasi himpunan tersedia (hasilnya frozenset baru)
  ✓ Semantik "himpunan final" — data yang stabil

Yang dikorbankan:
  ✗ Tidak bisa dimodifikasi — tidak cocok untuk himpunan yang dinamis
  ✗ Lebih jarang dibutuhkan dibanding set biasa
  ✗ Syntax pembuatan kurang intuitif: frozenset([...]) vs {...}
  ✗ Tidak bisa digunakan sebagai akumulator hasil yang bertambah
```

---

> **Frozenset** adalah versi *immutable* dari set di Python. Seperti set, ia menyimpan elemen unik tanpa urutan tertentu dan mendukung operasi himpunan. Karena immutable, frozenset bersifat *hashable* dan dapat digunakan sebagai key dictionary atau elemen dalam set. Frozenset adalah solusi untuk masalah "set of sets" dan semua konteks di mana himpunan perlu diperlakukan sebagai nilai stabil — bukan sebagai koleksi yang terus berubah.

---

# Bab 6: String — Teks sebagai Sequence Immutable

## Kenapa Struktur Ini Ada

String sering direduksi menjadi "tipe primitif untuk teks" — seolah-olah ia hanya sekadar wadah untuk karakter. Tapi dalam Python, string adalah *objek penuh* dengan organization style yang jelas: ia adalah *sequence immutable of Unicode characters*.

Memahami string sebagai struktur data — bukan sekadar tipe primitif — menjelaskan banyak perilaku yang tampak aneh jika dilihat dari lensa "teks biasa": kenapa `s[0] = 'H'` tidak bisa? Kenapa `s + "!"` menghasilkan string baru? Kenapa string bisa menjadi key dictionary? Jawabannya semua berakar dari satu keputusan desain arsitektural: **string adalah sequence yang tidak bisa diubah**.

String hadir karena teks adalah data yang secara universal *berurutan* (karakter ke-0, ke-1, ke-2...) dan *seharusnya stabil* (mengubah "Python" menjadi "python" seharusnya menghasilkan teks baru, bukan merusak teks lama).

## Intuisi: Teks yang Sudah Tercetak

Bayangkan string seperti teks yang sudah tercetak di kertas. "Halo" adalah urutan `H`, `a`, `l`, `o` — dan urutan itu adalah fakta yang sudah final. Kamu bisa membacanya, memfotokopinya, menulis catatan di margin fotokopi — tapi kamu tidak bisa mengubah teks tercetak itu sendiri.

Berbeda dengan list yang seperti papan tulis, string seperti dokumen resmi. Setiap "modifikasi" — kapitalisasi, penggantian, pemotongan — menghasilkan dokumen baru, sementara dokumen asli tetap tak tersentuh di arsip.

---

## Karakteristik Fundamental

### 1. Organization Style: Sequence (Terspesialisasi untuk Teks)

String adalah *sequence* — ia mengorganisir karakter dalam urutan linear, di mana posisi bermakna. Karakter ke-0 adalah `H`, ke-1 adalah `a`, dan seterusnya. String berbagi antarmuka sequence dengan list dan tuple: indexing, slicing, iterasi, dan operator `in`.

Tapi string adalah sequence *terspesialisasi*: ia dikhususkan untuk menyimpan karakter Unicode. Ia tidak bisa menampung integer atau objek bebas — hanya karakter. Ini adalah batasan yang membuatnya optimal untuk manipulasi teks.

### 2. Mutability: Immutable — Karakter Tidak Bisa Diubah

String tidak mendukung penugasan ke indeks, tidak mendukung `append()`, tidak mendukung operasi modifikasi apapun. Setiap operasi yang "mengubah" string selalu menghasilkan string *baru*.

```python
teks = "Python"
teks[0] = "p"       # TypeError: 'str' object does not support item assignment

teks_baru = "p" + teks[1:]
print(teks_baru)    # 'python' — string baru
print(teks)         # 'Python' — string asli tidak berubah
```

Immutability string bukan keterbatasan — ia adalah *keputusan desain multi-dimensi* yang menghasilkan keuntungan sistemik.

### 3. Ordering: Terurut — Urutan Karakter adalah Identitas

Seperti semua sequence, string menjaga urutan dengan ketat. "Python" dan "nohtyP" adalah dua string yang berbeda, meskipun mengandung karakter yang sama. Urutan adalah identitas.

### 4. Indexing / Keying: Akses Posisi Karakter

String mendukung indexing dan slicing dengan semantik identik dengan list dan tuple:

```python
kode = "INF2024"
print(kode[0])      # 'I'
print(kode[:3])     # 'INF'
print(kode[::-1])   # '4202FNI' — dibalik
```

### 5. Uniqueness: Duplikat Karakter Dibolehkan

String tidak menuntut keunikan karakter. `"aaabbb"` adalah string yang valid. Keunikan adalah domain set, bukan sequence.

### 6. Hashability: Hashable — Immutable Membukanya

Karena immutable, string bisa di-hash. Ini menjadikan string tipe data yang paling umum digunakan sebagai key dictionary — hampir semua dict yang merepresentasikan objek dengan atribut berlabel menggunakan string sebagai key:

```python
pengguna = {"nama": "Brata", "email": "brata@mail.com"}
# "nama" dan "email" adalah string — hashable, stabil, sempurna sebagai key
```

### 7. Internal Mechanism: Array Kompak Unicode

String diimplementasikan sebagai array kontinu yang kompak dari karakter Unicode. Python menggunakan representasi internal yang fleksibel (1, 2, atau 4 byte per karakter tergantung pada konten) untuk mengoptimalkan memori.

Karena immutable, Python bisa melakukan *string interning* — menyimpan satu salinan string literal yang sama dan membuat banyak variabel merujuk ke objek yang sama:

```python
a = "Python"
b = "Python"
print(a is b)   # True — seringkali objek yang sama di memori
```

Interning menghemat memori secara signifikan untuk program yang banyak menggunakan string literal yang sama.

### 8. Consequences

Immutability string menghasilkan konsekuensi yang luas:

- **Thread safety**: String bisa dibagi antar thread tanpa mekanisme sinkronisasi karena tidak bisa berubah.
- **Keamanan fungsional**: Kamu bisa melempar string ke fungsi mana pun tanpa khawatir fungsi itu merusak data asli.
- **Overhead concatenation**: Menggabungkan string dengan `+` dalam loop berulang kali membuat objek baru setiap iterasi — tidak efisien. Solusinya adalah `join()`, yang mengumpulkan semua bagian dan menggabungnya sekali.
- **Polimorfisme sequence**: Fungsi yang bekerja pada sequence (len, slicing, iterasi) bekerja untuk string, list, dan tuple secara seragam.

---

## Consequences & Behavior

```python
# Behavior 1: Immutability — identitas bertahan, konten tidak berubah
s = "halo"
print(id(s))
s = s + " dunia"     # membuat objek BARU, bukan memodifikasi yang lama
print(id(s))         # berbeda!
print(s)             # "halo dunia"

# Behavior 2: Method selalu mengembalikan string baru
original = "  Halo, Python!  "
bersih = original.strip()
print(original)      # "  Halo, Python!  " — tidak berubah
print(bersih)      # "Halo, Python!" — string baru

# Behavior 3: String sebagai sequence — berbagi antarmuka dengan list/tuple
def panjang(seq):
    return len(seq)

print(panjang("Python"))      # 6
print(panjang([1, 2, 3]))     # 3
print(panjang((10, 20)))      # 2

# Behavior 4: Unpacking bekerja pada string
a, b, c = "xyz"
print(a, b, c)   # x y z

# Behavior 5: Concatenation dalam loop — cara yang salah vs benar
# SALAH dan LAMBAT:
hasil = ""
for i in range(1000):
    hasil += str(i)    # membuat string baru setiap iterasi!

# BENAR dan CEPAT:
bagian = [str(i) for i in range(1000)]
hasil = "".join(bagian)   # gabung sekali, satu objek baru

# Behavior 6: Hashable — key dictionary yang universal
frekuensi = {}
for kata in "python adalah python".split():
    frekuensi[kata] = frekuensi.get(kata, 0) + 1
print(frekuensi)   # {'python': 2, 'adalah': 1}
```

---

## Relationship to Other Structures

String adalah *sequence immutable* — dari sudut mutability, ia berada di kubu yang sama dengan tuple, berlawanan dengan list. String, tuple, dan list membentuk spektrum sequence:

- **list**: sequence mutable, generik, tidak hashable
- **tuple**: sequence immutable, generik, hashable (jika elemen hashable)
- **string**: sequence immutable, terspesialisasi untuk teks, hashable

String lebih dekat ke tuple daripada ke list. Perbedaan utama: tuple adalah sequence generik untuk tipe campuran, string adalah sequence terspesialisasi untuk karakter Unicode dengan puluhan method manipulasi teks.

String juga adalah bukti bahwa **immutability → hashability** adalah hukum konsisten di Python: string immutable, sehingga hashable, sehingga bisa menjadi key dict — persis seperti tuple dan frozenset.

---

## Trade-Off

```
Yang diberikan:
  ✓ Immutable — aman, thread-safe, bisa di-intern untuk hemat memori
  ✓ Hashable — key dictionary yang paling umum dan natural
  ✓ Kaya method teks (strip, split, join, replace, ...)
  ✓ Mendukung Unicode penuh — karakter dari semua bahasa
  ✓ Polimorfisme dengan sequence lain (list, tuple)

Yang dikorbankan:
  ✗ "Perubahan" selalu membuat string baru — overhead untuk operasi berulang
  ✗ Concatenation dengan += dalam loop sangat tidak efisien
  ✗ Tidak bisa mengubah karakter langsung — harus membuat string baru
  ✗ Hanya bisa menyimpan karakter — tidak bisa menampung objek bebas
```

---

> **String** (`str`) adalah tipe data *sequence* yang bersifat *immutable* dan menyimpan urutan karakter Unicode. Karena immutable, string bersifat *hashable* dan dapat digunakan sebagai key dictionary. String mendukung indexing, slicing, iterasi, dan berbagai method manipulasi teks — semuanya menghasilkan string baru tanpa memodifikasi yang lama. String adalah sequence terspesialisasi untuk teks, berbagi paradigma posisi dengan list dan tuple, tetapi berbagi paradigma immutability dengan tuple.

---

# Bab 7: Sistem Satu Kesatuan: Enam Struktur, Tiga Gaya Berpikir

## Bukan Enam Pulau Terpisah

Setelah menyelami keenam struktur secara individual, saatnya melihat mereka sebagai satu sistem yang saling terhubung. Mereka bukan enam alat yang kebetulan ada di Python — mereka adalah solusi yang dirancang secara filosofis untuk tiga cara berpikir tentang data, dengan dua sumbu fundamental: **organization style** dan **mutability**.

## Peta Arsitektur

```
                         SEQUENCE
                    (berpikir dalam posisi)
                           │
              ┌────────────┼────────────┐
              │            │            │
            list         tuple         str
         (mutable)   (immutable)   (immutable)
         unhashable    hashable      hashable
              │            │            │
              └────────────┴────────────┘
                           │
                    hashable ──────────────┐
                                           │
                    MAPPING              SET-LIKE
               (berpikir dalam key)   (berpikir dalam keanggotaan)
                    │                     │
                  dict              ┌─────┴─────┐
               (mutable)           set       frozenset
            (insertion-ordered)  (mutable)  (immutable)
             unhashable          unhashable   hashable
```

Tiga pola besar terlihat di peta ini:

**Pola 1: Mutability membentuk pasangan simetris**
- `list` ↔ `tuple`: sequence mutable vs immutable
- `set` ↔ `frozenset`: set-like mutable vs immutable

Dalam setiap pasangan, keduanya berbagi organization style yang identik, berbeda hanya pada satu sumbu: kemampuan berubah.

**Pola 2: Immutability membuka hashability**
- `tuple` → hashable (jika elemen hashable)
- `frozenset` → hashable
- `str` → hashable

Semua struktur immutable di Python bersifat hashable. Semua struktur mutable tidak hashable. Ini bukan kebetulan — ini adalah konsekuensi mekanis dari stabilitas nilai yang dibutuhkan oleh hash table.

**Pola 3: Organization style menentukan cara akses**
- **Sequence** → diakses via posisi: "apa di indeks ke-N?"
- **Mapping** → diakses via key: "apa nilai dari label X?"
- **Set-like** → diakses via keanggotaan: "apakah X ada di sini?"

Tiga pertanyaan ini mencakup hampir semua pola akses data dalam pemrograman.

## Tabel Perbandingan Menyeluruh

| Struktur    | Gaya Organisasi | Mutable | Terurut   | Indexed | Unik | Hashable |
| ----------- | --------------- | ------- | --------- | ------- | ---- | -------- |
| `list`      | Sequence        | ✓       | ✓         | ✓       | ✗    | ✗        |
| `tuple`     | Sequence        | ✗       | ✓         | ✓       | ✗    | ✓*       |
| `dict`      | Mapping         | ✓       | Insertion | ✗ (key) | Key  | ✗        |
| `set`       | Set-like        | ✓       | ✗         | ✗       | ✓    | ✗        |
| `frozenset` | Set-like        | ✗       | ✗         | ✗       | ✓    | ✓        |
| `str`       | Sequence        | ✗       | ✓         | ✓       | ✗    | ✓        |

> *\* Tuple hashable hanya jika semua elemennya juga hashable.*

## Panduan Memilih: Tiga Pertanyaan Ontologis

Ketika menghadapi masalah dan perlu memilih struktur data, tiga pertanyaan ini akan mengantarmu ke jawaban yang tepat hampir setiap saat:

**Pertanyaan 1: Apakah posisi/urutan membawa makna semantik?**
- Ya, dan data akan berubah → **list**
- Ya, dan data adalah fakta final → **tuple** (atau **str** jika teks)

**Pertanyaan 2: Apakah data memiliki label alami untuk setiap field?**
- Ya → **dict**

**Pertanyaan 3: Apakah keunikan adalah hukum, dan pertanyaan utamanya adalah keanggotaan?**
- Ya, dan koleksi akan berubah → **set**
- Ya, dan koleksi final / perlu di-hash → **frozenset**

**Kasus khusus:**
- Data teks → **str**
- Koleksi yang perlu menjadi key dict → **tuple** (jika berurutan) atau **frozenset** (jika himpunan)
- Data berurutan yang perlu di-hash → **tuple**

## Satu Contoh, Enam Struktur Bersatu

Mari lihat bagaimana keenam struktur ini bekerja bersama secara natural dalam satu konteks:

```python
# Sistem pengelolaan artikel blog

# str — judul dan isi: teks yang berurutan dan final
judul = "Memahami Struktur Data Python"
isi   = "Python menyediakan berbagai struktur data yang kuat..."

# list — daftar artikel: urutan bermakna, dinamis
artikel_terbaru = [
    "Belajar Python dari Nol",
    "Memahami Struktur Data Python",
    "Tips Debugging Python"
]

# tuple — metadata: fakta yang tidak seharusnya berubah
metadata = ("2024-08-17", "Brata Yudha", "published")
tanggal, penulis, status = metadata   # unpacking

# dict — detail artikel: atribut berlabel
artikel = {
    "judul"  : judul,
    "penulis": penulis,
    "isi"    : isi,
    "views"  : 1250,
}

# set — tag: unik, sering dicek keberadaannya
tag = {"Python", "struktur-data", "pemrograman", "tutorial"}

# frozenset — kombinasi tag sebagai key cache: perlu hashable
cache_rekomendasi = {}
tag_frozen = frozenset(tag)
cache_rekomendasi[tag_frozen] = ["artikel_x", "artikel_y"]

# Semua bekerja bersama secara natural:
print(f"Artikel: {artikel['judul']}")
print(f"Oleh: {artikel['penulis']}, {tanggal}")
print(f"Tag: {', '.join(sorted(tag))}")
print(f"Posisi: {artikel_terbaru.index(judul) + 1}")
print(f"Ada tag 'Python'? {'Python' in tag}")
```

Perhatikan bahwa setiap pilihan struktur adalah *deklarasi intent* tentang sifat data:
- `metadata` adalah tuple karena fakta yang tidak berubah
- `tag` adalah set karena keunikan adalah hukum
- `artikel` adalah dict karena punya atribut berlabel
- `tag_frozen` adalah frozenset karena perlu dijadikan key

---

## Penutup: Berpikir dalam Python

Enam struktur data ini adalah fondasi dari hampir semua kode Python. Mereka bukan sekadar alat — mereka adalah *cara berpikir*.

Ketika kamu memilih `tuple` bukan karena "lebih hemat memori" tapi karena "data ini adalah fakta final", ketika kamu memilih `set` bukan karena "bisa menghapus duplikat" tapi karena "pertanyaan utamanya adalah keanggotaan", ketika kamu memilih `dict` bukan karena "bisa nyimpan apapun" tapi karena "data ini punya nama alami" — di saat itulah kamu tidak hanya menggunakan Python, tetapi *berpikir dalam Python*.

Struktur data Python adalah sistem yang simetris, elegan, dan filosofis. Memahami hubungan antar mereka — list dengan tuple, set dengan frozenset, sequence dengan mapping dengan set-like, mutable dengan immutable, hashable dengan unhashable — berarti memahami bahasa itu sendiri.

---

*Volume ini adalah bagian dari seri Struktur Data Python.*
*[[V1 - Fondasi & Mental Model|← Volume 1: Fondasi & Mental Model]] | [[V3 - Organization Styles|Volume 3: Organization Styles →]]*

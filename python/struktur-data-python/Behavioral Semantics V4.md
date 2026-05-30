# Beyond The Syntax — Python Data Structures
## Volume 4 — Behavioral Semantics

Dalam dua volume sebelumnya, kita telah melihat wujud nyata struktur data (*Volume 2 — Concrete Structures*) dan bagaimana mereka mengorganisasikan informasi ke dalam pola yang bermakna (*Volume 3 — Organization Styles*). Kita telah melihat Python sebagai sebuah gudang penyimpanan yang sangat fleksibel.

Namun, menganggap struktur data sekadar sebagai "wadah" atau "gudang" adalah sebuah jebakan. Struktur data di Python tidak pasif. Mereka adalah entitas yang hidup. Mereka berinteraksi, merespons instruksi, dan yang paling penting: **mereka bisa berubah**.

Volume ini didedikasikan untuk memahami *Behavioral Semantics*—lapisan di mana kita berhenti melihat bentuk, dan mulai melihat perilaku. Bagaimana sebuah *object* berubah? Apa konsekuensi dari perubahan itu terhadap bagian program Anda yang lain? Mengapa *bug* misterius sering kali muncul bukan dari kesalahan logika, melainkan dari kesalahpahaman tentang perilaku *object*?

Mari kita bongkar cara berpikir Python.

---

### 1. Introduction — Data Structures Are Behaviors

Bayangkan Anda membeli sebuah rumah (sebagai analogi dari sebuah struktur data). Bentuk rumah itu (tipe datanya) dan susunan ruangannya (organisasi) sudah jelas. Namun, bagaimana perilaku rumah itu jika Anda mulai membongkar dinding atau menambahkan lantai baru? Apakah rumah tetangga yang menempel pada dinding Anda akan ikut runtuh?

Di Python, struktur data mendefinisikan *aturan main* tentang bagaimana perubahan dapat dilakukan. Aturan ini menentukan apakah sebuah data boleh dimodifikasi (*mutation*), bagaimana modifikasi tersebut menyebar (*side effects*), dan bagaimana berbagai komponen dalam program Anda berbagi informasi yang sama (*shared state*).

Memahami *behavior* ini adalah kunci transisi dari seorang pengguna sintaks menjadi seorang *engineer* yang mendesain perangkat lunak yang tangguh dan dapat diprediksi.

---

### 2. Mutable vs Immutable (Transisi State vs Terkunci)

Perbedaan paling mendasar dalam perilaku *object* Python terletak pada satu pertanyaan sederhana: **Apakah object ini diizinkan untuk berubah setelah ia diciptakan?**

Jawaban dari pertanyaan ini membagi seluruh ekosistem Python ke dalam dua kubu besar: **Mutable** dan **Immutable**.

#### Mutability (Bisa Berubah)
Sebuah *object* disebut *mutable* jika Anda bisa memodifikasi "isi" atau keadaan internalnya (*state*) tanpa menghancurkan *object* itu sendiri.
Contoh: `list`, `dict`, `set`.

```python
# Membuat list (mutable)
keranjang = ["apel", "jeruk"]

# Kita mengubah state-nya
keranjang.append("mangga")
```
*Behavioral Reasoning:* `keranjang` sekarang memiliki *state* yang baru, tetapi secara fisik, ia adalah keranjang yang persis sama dengan yang kita buat di awal.

#### Immutability (Terkunci)
Sebuah *object* disebut *immutable* jika *state* internalnya dikunci permanen sejak detik pertama ia lahir. Jika Anda ingin data yang berbeda, Anda tidak bisa memodifikasinya; Anda harus menciptakan *object* yang benar-benar baru.
Contoh: `tuple`, `str`, `int`.

```python
kata = "Python"
# kata[0] = "J"  <-- TypeError! Python akan menolak mutasi ini
```
*Behavioral Reasoning:* Teks `"Python"` tidak akan pernah bisa diubah menjadi `"Jython"`. String adalah pahatan batu. Ia stabil secara permanen.

> **Intuisi:**
> Mutabilitas adalah kemampuan sebuah *object* untuk melakukan transisi *state* (perubahan kondisi) seiring berjalannya waktu.

---

### 3. Object References (Variabel Bukanlah Kotak)

Sebelum kita melihat konsekuensi dari mutasi, kita harus menghancurkan satu mitos terbesar di kalangan programmer pemula: *Variabel bukanlah sebuah kotak penyimpan data.*

Di bahasa seperti C kuno, variabel memang seperti kotak yang terikat pada lokasi memori fisik. Namun di Python, variabel hanyalah **label nama (reference) yang diikatkan ke sebuah object.**

Lihat contoh sederhana ini:
```python
a = [1, 2, 3]
```

Python **tidak** membuat kotak bernama `a` lalu memasukkan list ke dalamnya.
Yang dilakukan Python adalah:
1. Menciptakan sebuah *object* list `[1, 2, 3]` di suatu tempat di memori.
2. Mengambil seutas tali (reference), menempelkan label bernama `a` di ujungnya, dan mengikat ujung satunya ke list tersebut.

Pemahaman bahwa variabel hanyalah "label" adalah fondasi mutlak untuk memahami anomali perilaku di Python.

---

### 4. Shared References & Aliasing

Karena variabel hanyalah label nama, apa yang terjadi jika kita menggunakan operator `=`?

```python
a = [10, 20]
b = a
```

**Assignment `=` TIDAK pernah menyalin data.** 
Instruksi `b = a` berarti: *"Hai Python, ambil label nama `b`, lalu ikatkan pada object yang saat ini sedang ditunjuk oleh label `a`."*

Secara visual, inilah yang terjadi di memori:
```text
a ─────┐
       ▼
    [10, 20]
       ▲
b ─────┘
```

Fenomena di mana dua atau lebih variabel menunjuk ke *object* fisik yang persis sama disebut **Aliasing** (satu benda memiliki banyak alias/nama).

*Aliasing* sangat efisien secara memori, namun ini adalah pintu gerbang menuju kompleksitas perilaku.

---

### 5. State and Behavioral Change

Sekarang, mari kita gabungkan konsep **Mutability** (bisa berubah) dan **Aliasing** (berbagi referensi).

Ketika sebuah *object* memiliki banyak label nama, dan *object* itu kebetulan bersifat *mutable* (seperti `list`), kita berhadapan dengan situasi **Shared Mutable State** (kondisi bisa berubah yang digunakan bersama).

> *Shared mutable state* diakui secara luas dalam *computer science* sebagai salah satu akar terbesar dari *bug* perangkat lunak yang sulit dilacak.

Perhatikan skenario ini:
```python
a = [10, 20]
b = a

# Kita memutasi object melalui jalur 'b'
b.append(30)

# Apa yang terjadi pada 'a'?
print(a)  # Output: [10, 20, 30]
```

**Kenapa behavior ini muncul?**
Karena `a` dan `b` menunjuk pada *object* yang sama. Ketika Anda meminta `b` untuk bermutasi, *state* dari *object* fisik tersebut berubah. Karena `a` kebetulan sedang melihat ke arah *object* yang sama, `a` secara otomatis akan mencerminkan *state* yang baru.

Ini bukan *magic*, dan ini bukan kesalahan Python. Ini adalah konsekuensi logis mutlak dari sistem referensi (*reference system*). 

---

### 6. Mutation and Side Effects (Efek Samping)

Perilaku berbagi *state* ini menjadi sangat berbahaya ketika kita melewati batas antar wilayah dalam program, misalnya saat memasukkan variabel ke dalam sebuah fungsi.

```python
def format_data_user(user_dict):
    # Mengubah huruf menjadi kapital
    user_dict['nama'] = user_dict['nama'].upper()
    return user_dict

data_saya = {'nama': 'budi'}

# Memanggil fungsi
hasil = format_data_user(data_saya)

print(data_saya) 
# Output: {'nama': 'BUDI'} (Data asli ikut berubah!)
```

**Behavioral Consequences:**
Di Python, argumen diteruskan ke fungsi menggunakan mekanisme *Pass-by-Object-Reference*. Artinya, variabel lokal `user_dict` di dalam fungsi akan diikat ke *object* yang persis sama dengan `data_saya` di luar fungsi.

Ketika fungsi tersebut memutasi *dictionary* itu, mutasi tersebut bocor keluar dari fungsi. Inilah yang disebut **Side Effect** (efek samping). 

Sebuah fungsi dengan *side effect* akan mengubah keadaan dunia di luar dirinya. Jika arsitektur program Anda tidak hati-hati, fungsi yang berniat baik ini bisa menghancurkan *state* data yang sedang dipakai oleh bagian program lain.

---

### 7. Copy Semantics: Assignment ≠ Copy

Lalu, bagaimana jika kita memang *ingin* menyalin datanya agar mutasinya tidak menular? Karena `b = a` hanya menyalin referensi (label nama), kita butuh cara eksplisit untuk menduplikasi *object* fisiknya.

Di sinilah **Copy Semantics** berperan. Kita harus memaksa Python membuat *object* fisik yang benar-benar baru.

```python
# Membuat salinan nyata untuk list sederhana
asli = [1, 2, 3]
salinan = asli.copy()  # atau list(asli)

salinan.append(4)
print(asli)     # [1, 2, 3] (Aman!)
print(salinan)  # [1, 2, 3, 4]
```
Kini, perubahan di `salinan` tidak menyentuh `asli` karena mereka adalah dua bangunan yang terpisah.

---

### 8. Shallow Copy vs Deep Copy (Jebakan Berlapis)

Masalah belum selesai. Struktur data dunia nyata sering kali berlapis (*nested*), seperti list di dalam list, atau list di dalam dictionary.

Menyalin *object* bersarang memunculkan dilema: **Shallow Copy** (Salinan Dangkal) vs **Deep Copy** (Salinan Dalam).

#### Shallow Copy
Ketika Anda menggunakan `.copy()` atau rutin standar penyalinan, Python melakukan *Shallow Copy*. Python akan membuat "amplop" terluar yang baru, tetapi "isi suratnya" tidak disalin ulang—Python hanya menempelkan referensi ke *object* isi yang sudah ada.

```python
import copy

pabrik = ["Mobil", ["Mesin A", "Roda B"]]
# Melakukan Shallow Copy
cabang_pabrik = copy.copy(pabrik)

# Mengubah daftar item terluar (Aman)
cabang_pabrik.append("Motor")

# Memodifikasi object yang ada di DILAM list (Bahaya!)
cabang_pabrik[1][0] = "Mesin Rusak"

print(pabrik[1])        # ['Mesin Rusak', 'Roda B']
print(cabang_pabrik[1]) # ['Mesin Rusak', 'Roda B']
```

**Analogi Mental:**
Shallow copy ibarat menyalin map/folder di komputer. Foldernya sendiri adalah folder baru, tetapi isi (*shortcut*) di dalamnya masih mengarah ke *file* asli yang persis sama. Jika Anda menghapus *file* dari dalam folder salinan, *file* aslinya ikut lenyap.

#### Deep Copy
Untuk menduplikasi total secara aman, gunakan `deepcopy`. Algoritma ini akan menelusuri struktur data sampai ke akar-akarnya, dan menyalin ulang setiap *mutable object* yang ia temukan.

```python
cabang_pabrik_aman = copy.deepcopy(pabrik)
```
*Deep copy* sangat aman, mencegah semua risiko *side effect* pada lapisan dalam. Namun, konsekuensinya ia sangat mahal secara performa dan memori, ibarat membangun gedung duplikat dari awal hingga batu bata terakhir.

---

### 9. Immutability as Stability

Mengingat betapa berisikonya *shared mutable state* dan betapa rumitnya menyalin data, kita mulai melihat keindahan sesungguhnya dari **Immutable Objects** (seperti `tuple`).

Kenapa kita butuh *tuple* padahal kita punya *list* yang jauh lebih fleksibel? Jawabannya adalah **Stabilitas Perilaku**.

Ketika sebuah *object* bersifat *immutable*, Anda bisa dengan tenang membagikan referensinya (*aliasing*) ke ratusan variabel dan puluhan fungsi. Anda dijamin 100% bebas dari ancaman *hidden mutation* dan *side effects*.

```python
def baca_data(data_tuple):
    # Tidak peduli sekeras apa pun fungsi ini mencoba,
    # ia tidak akan bisa memutasi data_tuple.
    pass
```

Jika aplikasi Anda bersifat besar dan multithreading (banyak proses berjalan bersamaan), *immutability* adalah penyelamat hidup karena menghilangkan risiko bahwa *thread* lain secara diam-diam memodifikasi data yang sedang Anda baca. *Immutability* membawa kedamaian dan prediksi yang pasti.

---

### 10. Mutable Default Argument Trap

Ada satu jebakan perilaku mutasi di Python yang telah memakan korban jutaan *programmer* pemula. Jebakan ini merupakan kombinasi langsung dari cara Python menangani *state* dan referensi fungsi.

Perhatikan kode yang terlihat tidak berbahaya ini:
```python
def tambah_barang(item, keranjang=[]):
    keranjang.append(item)
    return keranjang

print(tambah_barang("Apel"))   # Output: ['Apel']
print(tambah_barang("Jeruk"))  # Output: ['Apel', 'Jeruk'] <-- Tunggu, kenapa apelnya ikut?!
```

**Behavioral Reasoning:**
Banyak orang mengira parameter `keranjang=[]` akan menciptakan list baru setiap kali fungsi dipanggil. 
Faktanya: Nilai *default argument* di Python dievaluasi **hanya satu kali**, yaitu pada detik saat fungsi tersebut pertama kali dibaca/didefinisikan oleh Python.

List kosong `[]` itu lahir sekali dan terus hidup di memori. Setiap kali Anda memanggil fungsi tanpa memberikan parameter kedua, fungsi tersebut akan mengakses list tua yang persis sama. Karena operasi `append` memutasi *state* list tersebut, *state*-nya terus menumpuk di setiap pemanggilan.

**Mental Model untuk Mencegahnya:**
Jangan pernah menjadikan *mutable object* sebagai nilai *default*. Gunakan nilai statis dan bangun *object* di dalam fungsi.

```python
def tambah_barang_aman(item, keranjang=None):
    if keranjang is None:
        keranjang = []  # List benar-benar baru di setiap pemanggilan
    keranjang.append(item)
    return keranjang
```

---

### 11. Mental Model Akhir

Volume 4 telah mengubah cara kita memandang variabel dan data di Python. Ini adalah pondasi mental model *engineer* Anda:

1.  **Variabel adalah Referensi:** Jangan memikirkan kotak penyimpanan. Pikirkan label nama dan utas tali yang menunjuk pada *object* hidup.
2.  **Object Memiliki State:** Modifikasi adalah transisi *state*, bukan sekadar menulis ulang teks.
3.  **Aliasing dan Shared State:** Beberapa nama yang menunjuk pada *object* yang sama menciptakan jembatan tak kasat mata. Jika *object* itu bisa bermutasi (*mutable*), jembatan itu mengalirkan perubahan ke semua sisi secara simultan.
4.  **Copy vs Assignment:** Pemahaman membedakan "=" dengan ".copy()" adalah tameng pertama Anda terhadap *side effects*.
5.  **Stabilitas Immutability:** Kunci *state* Anda (gunakan *tuple*, *frozenset*) kapanpun modifikasi tidak diperlukan, demi stabilitas arsitektur kode.

Object Python tidak pasif; mereka memiliki perilaku dan konsekuensi ruang lingkup (*scope consequence*).

---

### 12. Transition to Volume 5

Kita telah memahami bahwa identitas fisik sebuah *object* (lokasi referensinya) dan kemampuannya untuk bermutasi sangatlah sentral dalam perilaku Python. 

Namun, fakta ini membawa satu pertanyaan desain yang sangat besar bagi sang pencipta bahasa Python:
> Jika sebuah *object* (seperti `list`) isinya bisa berubah-ubah dan bertumbuh kapan saja, **bagaimana caranya Python bisa memastikan dan melacak identitas object tersebut secara absolut?**

Jika kita ingin membuat sistem pencarian instan—seperti menemukan kunci di dalam *dictionary* yang isinya jutaan data—kita butuh jaminan mutlak. Kita butuh sebuah identitas yang *konsisten*. 

Bagaimana Python menjembatani sifat mutabilitas ini dengan kebutuhan sistem pencarian berkecepatan tinggi? Jawaban teknis dan filosofis atas pertanyaan ini menanti di volume kita selanjutnya:

---

*Volume ini adalah bagian dari seri Struktur Data Python.*
*← Volume 1: Fondasi & Mental Model | Volume 3: Organization Style & Abstraksi →*

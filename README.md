# Deney Sonu Teslimatı

Sistem Programlama ve Veri Yapıları bakış açısıyla veri tabanlarındaki performansı öne çıkaran hususlar nelerdir?

Aşağıda kutucuk (checkbox) ile gösterilen maddelerden en az birini seçtiğiniz açık kaynak kodlu bir VT kaynak kodları üzerinde göstererek açıklayınız. Açıklama bölümüne kısaca metninizi yazıp, kod üzerinde gösterim videonuzun linkini en altta belirtilen kutucuğa yerleştiriniz.

- [X]  Seçtiğiniz konu/konuları bu şekilde işaretleyiniz. **!**
    
---

# 1. Sistem Perspektifi (Operating System, Disk, Input/Output)

### Disk Erişimi

- [ ]  **Blok bazlı disk erişimi** → block_id + offset
- [ ]  Rastgele erişim

### VT için Page (Sayfa) Anlamı

- [ ]  VT hangisini kullanır? **Satır/ Sayfa** okuması

---

### Buffer Pool

- [x]  Veritabanları, Sık kullanılan sayfaları bellekte (RAM) kopyalar mı (caching) ?

- [x]  LRU / CLOCK gibi algoritmaları
- [ ]  Diske yapılan I/O nasıl minimize ederler?

# 2. Veri Yapıları Perspektifi

- [ ]  B+ Tree Veri Yapıları VT' lerde nasıl kullanılır?
- [ ]  VT' lerde hangi veri yapıları hangi amaçlarla kullanılır?
- [ ]  Clustered vs Non-Clustered Index Kavramı
- [ ]  InnoDB satırı diskte nasıl durur?
- [ ]  LSM-tree (LevelDB, RocksDB) farkı
- [ ]  PostgreSQL heap + index ayrımı

DB diske yazarken:

- [ ]  WAL (Write Ahead Log) İlkesi
- [ ]  Log disk (fsync vs write) sistem çağrıları farkı

---

# Özet Tablo

| Kavram      | Bellek          | Disk / DB      |
| ----------- | --------------- | -------------- |
| Adresleme   | Pointer         | Page + Offset  |
| Hız         | O(1)            | Page IO        |
| PK          | Yok             | Index anahtarı |
| Veri yapısı | Array / Pointer | B+Tree         |
| Cache       | CPU cache       | Buffer Pool    |

---

# Video [Linki](https://www.youtube.com/watch?v=Nw1OvCtKPII&t=2635s) 
Ekran kaydı. 2-3 dk. açık kaynak V.T. kodu üzerinde konunun gösterimi. Video kendini tanıtma ile başlamalıdır (Numara, İsim, Soyisim, Teknik İlgi Alanları). 

---

# Açıklama (Ort. 600 kelime)

### Konu: PostgreSQL Üzerinde Buffer Pool (Önbellek Havuzu) İncelemesi

**1. Giriş ve Teorik Altyapı:**
Veritabanı yönetim sistemlerinde (DBMS) disk erişimi (I/O), bellek (RAM) erişimine göre çok daha yavaş ve maliyetli bir işlemdir. Bu nedenle, sık kullanılan veri sayfalarının (pages) sürekli diskten okunması yerine, RAM üzerinde ayrılmış özel bir alanda tutulması gerekir. Bu alana **Buffer Pool (Önbellek Havuzu)** denir.

Teorik olarak Buffer Pool, diskin yavaşlığını maskelemek için bir "cache" (önbellek) katmanı gibi çalışır. Bir veri istendiğinde veritabanı önce Buffer Pool'a bakar (Logical Read). Eğer veri oradaysa diske gitmeden milisaniyeler içinde yanıt döner (Buffer Hit). Eğer yoksa, veri diskten okunur ve Buffer Pool'a yüklenir (Physical Read). Buffer Pool dolduğunda ise, hangi sayfanın atılacağına (eviction) karar vermek için algoritmalar (LRU, Clock Sweep vb.) kullanılır.

**2. Açık Kaynak Kod İncelemesi (PostgreSQL):**
Bu raporda, açık kaynak veritabanı olan **PostgreSQL**'in kaynak kodlarını inceledim. PostgreSQL'de Buffer Pool yönetimi, temel olarak `src/backend/storage/buffer/` dizini altındaki dosyalarda yapılmaktadır.

* **Yönetim Merkezi (`bufmgr.c`):** İncelediğim `bufmgr.c` dosyası, Buffer Yöneticisinin ana arayüzüdür. Burada bulunan `ReadBuffer` ve `ReadBufferExtended` fonksiyonları sistemin giriş kapısıdır. Bir sorgu veri istediğinde bu fonksiyonlar çağrılır; sayfanın bellekte olup olmadığı burada kontrol edilir.
* **Sayfa Değiştirme Algoritması (Clock Sweep):** PostgreSQL, standart bir LRU (Least Recently Used) yerine, daha performanslı olan "Clock Sweep" algoritmasını kullanır. Kodlarda gördüğümüz `StrategyGetBuffer` fonksiyonu (`freelist.c` ile bağlantılı çalışır), bellekte yer açmak gerektiğinde "victim" (kurban) sayfanın seçilmesinden sorumludur. `BufferDescriptors` dizisi üzerinde dönerek kullanım sıklığına (usage count) göre hangi sayfanın atılacağına karar verir.

Bu yapı sayesinde PostgreSQL, disk I/O işlemlerini minimize ederek yüksek performans sağlar.


## VT Üzerinde Gösterilen Kaynak Kodları

PostgreSQL Buffer Manager (bufmgr.c): [Github Linki](https://github.com/postgres/postgres/blob/master/src/backend/storage/buffer/bufmgr.c)
PostgreSQL Buffer Strategy (freelist.c): [Github Linki](https://github.com/postgres/postgres/blob/master/src/backend/storage/buffer/freelist.c)

Açıklama [Linki](https://...) \
Açıklama [Linki](https://...) \
Açıklama [Linki](https://...) \
... \
...

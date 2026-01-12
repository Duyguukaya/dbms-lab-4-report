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

### Konu: PostgreSQL Üzerinde Buffer Pool (Önbellek Havuzu) ve Bellek Yönetimi Mimarisi

**1. Giriş: Sistem Programlama Perspektifi ve Disk/Bellek Darboğazı**
Veritabanı Yönetim Sistemleri (DBMS) tasarımındaki en temel zorluk, bellek hiyerarşisindeki hız farklarıdır. Modern bilgisayar mimarilerinde CPU (İşlemci) ve RAM (Ana Bellek) arasındaki veri alışverişi nanosaniyeler mertebesinde gerçekleşirken, Disk (HDD/SSD) erişimi milisaniyeler veya mikrosaniyeler sürer. Bu durum, disk I/O işlemlerinin sistemin genel performansı üzerinde ciddi bir darboğaz (bottleneck) oluşturmasına neden olur. Sistem programlama açısından bakıldığında, her veri isteğinde diske `read()` sistem çağrısı yapmak maliyetlidir. Bu sorunu çözmek için PostgreSQL, İşletim Sisteminin "Page Cache" yapısına benzer ancak veritabanı ihtiyaçlarına özelleşmiş **Buffer Pool (Önbellek Havuzu)** mekanizmasını kullanır.

PostgreSQL verileri diskte varsayılan olarak 8KB’lık "Page" (Sayfa) blokları halinde saklar. Buffer Pool, bu sayfaların RAM üzerinde kopyalarının tutulduğu paylaşımlı bir bellek alanıdır. Bir sorgu çalıştırıldığında, sistem önce istenen sayfanın bu havuzda olup olmadığını kontrol eder (**Logical Read**). Eğer veri bellekteyse (Buffer Hit), disk erişimine gerek kalmadan işlemci hızında yanıt verilir. Veri bellekte yoksa, diskten okunup havuza yüklenir (**Physical Read**). Temel amaç, fiziksel okumaları minimize ederek "Hit Ratio" oranını maksimize etmektir.

**2. Algoritma ve Veri Yapıları: LRU Yerine Neden Clock Sweep?**
Buffer Pool sınırlı bir boyuta sahiptir (Shared Buffers). Havuz dolduğunda, yeni gelen verilere yer açmak için bellekteki bir sayfanın atılmasına (eviction) karar verilmelidir. Teorik veri yapıları derslerinde bu işlem için genellikle **LRU (Least Recently Used - En Az Son Kullanılan)** algoritması önerilir. Ancak LRU, her veri erişiminde bellekteki bir "Çift Bağlı Liste" (Doubly Linked List) yapısının güncellenmesini gerektirir. Çok kullanıcılı (concurrency) ve yüksek işlem hacimli sistemlerde, bu listenin sürekli kilitlenmesi (Locking) ciddi performans kayıplarına ("Lock Contention") yol açar.

Bu nedenle PostgreSQL, LRU'nun bir yaklaşımı olan **Clock Sweep (Saat Algoritması)** kullanır.
* **Veri Yapısı:** Bufferlar dairesel bir dizi (Circular Buffer) gibi düşünülür.
* **Çalışma Mantığı:** Her sayfanın bir "Usage Count" (Kullanım Sayacı) değeri vardır. Algoritma bir saat ibresi gibi sayfalar üzerinde döner. İbre bir sayfaya geldiğinde, eğer `usage_count > 0` ise sayacı azaltır ve sayfayı bellekte tutarak bir sonrakine geçer (ona ikinci bir şans verir). Eğer `usage_count == 0` ise, o sayfa "Victim" (Kurban) seçilir ve üzerine yeni veri yazılır. Bu yöntem, O(1) karmaşıklığına yakın çalışır ve kilitlenme sorunlarını minimize eder.

**3. Açık Kaynak Kod İncelemesi (PostgreSQL Source Code)**
Bu çalışmada, GitHub üzerindeki PostgreSQL kaynak kodlarının `src/backend/storage/buffer/` dizini incelenmiştir.

* **Buffer Yöneticisi (`bufmgr.c`):** Sistemin kalbi burasıdır. `ReadBufferExtended` fonksiyonu, tüm okuma isteklerinin giriş kapısıdır. Bu fonksiyon içinde çağrılan `BufferAlloc`, istenen sayfa bellekte yoksa yeni bir yer ayırmakla görevlidir. Eğer yer yoksa, tahliye algoritmasını tetikler.
* **Tahliye Stratejisi (`freelist.c`):** Clock Sweep algoritmasının kodlandığı yerdir. `StrategyGetBuffer` fonksiyonu incelendiğinde, kodun `BufferDescriptors` dizisi üzerinde bir döngü kurduğu ve `usage_count` değişkenini kontrol ederek "kurban" sayfayı aradığı görülmektedir. Bu, teorik algoritmanın C diliyle pratik uygulamasıdır.
* **Kirli Sayfalar (Dirty Pages):** Eğer bellekteki bir sayfada güncelleme yapıldıysa, bu sayfa "Dirty" olarak işaretlenir. Bu sayfa bellekten atılmadan önce, veri bütünlüğü için `FlushBuffer` fonksiyonu ile mutlaka diske yazılmalıdır. Ayrıca, Write Ahead Log (WAL) protokolü gereği, veri diske yazılmadan önce log kaydının alınması da bu kod bloklarında yönetilir.

Sonuç olarak; PostgreSQL'in Buffer Pool yapısı, disk yavaşlığını maskeleyerek veritabanı performansını optimize eden, gelişmiş veri yapıları ve algoritmalar kullanan karmaşık bir sistem programlama örneğidir.


## VT Üzerinde Gösterilen Kaynak Kodları

PostgreSQL Buffer Manager (bufmgr.c): [Github Linki](https://github.com/postgres/postgres/blob/master/src/backend/storage/buffer/bufmgr.c)
PostgreSQL Buffer Strategy (freelist.c): [Github Linki](https://github.com/postgres/postgres/blob/master/src/backend/storage/buffer/freelist.c)

Açıklama [Linki](https://...) \
Açıklama [Linki](https://...) \
Açıklama [Linki](https://...) \
... \
...

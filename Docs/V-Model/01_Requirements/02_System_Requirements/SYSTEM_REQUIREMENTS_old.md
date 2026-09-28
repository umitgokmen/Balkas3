# Balcas Geliştirilmiş Sistem Gereksinimleri

## 1. Belgenin amacı

Bu belge, `CURRENT_SYSTEM_REQUIREMENTS.md` içindeki mevcut ürün kapsamını temel alarak Balcas sisteminin daha açık, tutarlı, doğrulanabilir ve yeniden geliştirmeye elverişli gereksinimlerini tanımlar.

Bu belge:

- mevcut belgenin yerine geçmez; onunla birlikte kullanılır;
- kaynak kodun nasıl çalıştığından çok ürünün ne yapması gerektiğine odaklanır;
- mevcut sistemdeki bariz çelişkileri gereksinim olarak tekrar etmez;
- kabul kriterlerini ve hata davranışlarını açıkça tanımlar;
- doğrulanmamış iş kararlarını görünür biçimde işaretler;
- belirli bir UI framework'ü, programlama dili veya görüntü işleme kütüphanesi dayatmaz.

Belge statüsü: **İnceleme taslağı**  
Temel belge: `CURRENT_SYSTEM_REQUIREMENTS.md`  
Hedef okuyucular: operasyon, proses, otomasyon/PLC, görüntü işleme, yazılım ve test ekipleri

## 2. Gereksinim dili

| Terim | Anlam |
|---|---|
| ZORUNLU | Ürünün kabul edilmesi için sağlanması gerekir. |
| ÖNERİLEN | Güçlü biçimde beklenir; uygulanmayacaksa gerekçe kaydedilir. |
| OPSİYONEL | Faydalıdır ancak ilk teslimat için zorunlu değildir. |
| ONAY GEREKLİ | Mevcut koddan güvenilir biçimde çıkarılamayan iş kararıdır. |

Her gereksinimin bir kimliği ve önceliği vardır. Kabul kriterleri, gereksinimin nasıl doğrulanacağını tarif eder.

## 3. Ürün özeti

Balcas, bir tomruğa ait çok kameralı 3B veriyi analiz ederek tomruğun geometrik özelliklerini ve kabul durumunu belirleyen, PLC ile koordineli çalışan bir endüstriyel denetim sistemidir.

Sistem aşağıdaki temel sonucu üretir:

```text
PLC denetim isteği
        ↓
Eşzamanlı 3B kamera edinimi
        ↓
Ortak koordinat sisteminde nokta bulutu
        ↓
Tomruk izolasyonu ve kalite kontrolü
        ↓
Uzunluk, çap, yön, flare ve sehim ölçümü
        ↓
Kabul / Ret / Ölçüm Hatası
        ↓
PLC sonucu + operatör görünümü + izlenebilir kayıt
```

## 4. Hedefler ve kapsam dışı konular

### 4.1 Hedefler

| Kimlik | Hedef |
|---|---|
| OBJ-001 | Her PLC tetiklemesinde tek ve izlenebilir bir tomruk denetimi gerçekleştirmek. |
| OBJ-002 | Geçerli 3B veriden uzunluk, çap, yön, flare ve sehim değerleri üretmek. |
| OBJ-003 | Geçersiz veya yetersiz ölçümü yanlışlıkla kabul edilmiş ürün olarak yayınlamamak. |
| OBJ-004 | PLC ile deterministik bir istek/sonuç/onay el sıkışması yürütmek. |
| OBJ-005 | Operatöre sistem, kamera, PLC ve denetim durumunu anlaşılır biçimde göstermek. |
| OBJ-006 | Denetimin tekrar analiz edilebilmesi için yeterli log ve isteğe bağlı ham veri saklamak. |
| OBJ-007 | Kayıtlı veriler üzerinde üretimden bağımsız simülasyon ve regresyon testi yapabilmek. |

### 4.2 Kapsam dışı

Aşağıdakiler ayrıca onaylanmadıkça bu gereksinim setinin kapsamı dışındadır:

- PLC makine hareketlerinin kontrolü;
- tomruğun hatta fiziksel olarak yönlendirilmesi veya ayrılması;
- kullanıcı/rol yönetimi;
- bulut tabanlı görüntü işleme;
- yapay zekâ ile tomruk sınıflandırma;
- kamera kalibrasyonunun otomatik gerçekleştirilmesi;
- üretim planlama veya ERP entegrasyonu.

## 5. Terimler ve sonuç modeli

| Terim | Tanım |
|---|---|
| Denetim | Tek PLC isteğine veya tek kullanıcı simülasyon komutuna karşılık gelen uçtan uca işlem. |
| Etkin kamera | Ayarlarda denetime dahil edilen kamera. |
| Dilim | Tomruk ekseni boyunca belirli bir pencere içinde bulunan 3B veri ve buna uydurulan geometrik model. |
| Aykırı dilim | Veri kalitesi veya geometrik tutarlılık kontrolünü geçemeyen dilim. |
| Flare | Tomruğun dip ucuna yakın, yarıçapın belirgin biçimde arttığı bölge. |
| Plan A | Silindir dilimleri üzerinden çalışan birincil sehim ölçümü. |
| Plan B | Birincil yöntemi doğrulamak veya gelecekte yedeklemek için kullanılan ikincil ölçüm yöntemi. |

### 5.1 Zorunlu sonuç durumları

Sistem, boolean bir sonuçtan bağımsız olarak aşağıdaki üç mantıksal durumu ayırt etmelidir:

| Durum | Tanım | PLC Outcome eşlemesi |
|---|---|---|
| Accepted | Ölçüm geçerlidir ve bütün kabul kuralları sağlanmıştır. | `true` |
| Rejected | Ölçüm geçerlidir fakat en az bir ürün kabul kuralı sağlanmamıştır. | `false` |
| MeasurementError | Güvenilir karar vermeye yetecek ölçüm üretilememiştir. | `false` |

`MeasurementError`, fiziksel bir ret ile aynı şey değildir. PLC arayüzü yalnızca boolean sonucu destekliyorsa ikisi de güvenli davranış olarak `Outcome=false` yayınlanır; operatör ekranı ve loglar gerçek nedeni ayırt etmelidir.

## 6. Uçtan uca operasyon gereksinimleri

### 6.1 Sistem başlatma

| Kimlik | Öncelik | Gereksinim |
|---|---|---|
| SYS-START-001 | ZORUNLU | Sistem başlarken son geçerli kalıcı ayarları yüklemelidir. |
| SYS-START-002 | ZORUNLU | Ayar dosyası yoksa belgelenmiş varsayılan ayarlar kullanılmalı ve operatöre bilgi mesajı gösterilmelidir. |
| SYS-START-003 | ZORUNLU | Ayar dosyası okunamıyor veya geçersizse sistem bunu sessizce yok saymamalı; varsayılanlarla çalıştığını belirgin biçimde göstermeli ve loglamalıdır. |
| SYS-START-004 | ZORUNLU | Sistem etkin gerçek kameraların her birine bağlanmayı denemelidir. Devre dışı kameranın bağlantısı denetim hazır durumunu etkilememelidir. |
| SYS-START-005 | ZORUNLU | PLC kanalı kullanıma hazır olmadan sistem `Ready/Waiting` durumu yayınlamamalıdır. |
| SYS-START-006 | ZORUNLU | Kamera, PLC veya ayar başlatma hatası uygulamanın kapanmasına neden olursa hata kalıcı logda bulunmalıdır. |
| SYS-START-007 | ÖNERİLEN | Başlangıç ekranında uygulama sürümü, çalışma modu, seçilen PLC protokolü ve ayar kaynağı gösterilmelidir. |

### 6.2 Hazır olma koşulu

| Kimlik | Öncelik | Gereksinim |
|---|---|---|
| SYS-READY-001 | ZORUNLU | Gerçek çalışma modunda en az bir kamera etkin olmalıdır. |
| SYS-READY-002 | ZORUNLU | Bütün etkin kameralar bağlı değilse sistem ölçüm için hazır sayılmamalıdır. |
| SYS-READY-003 | ZORUNLU | Simülasyon modunda seçili veri seti bulunamıyor veya gerekli kamera dosyaları eksikse sistem ölçüm için hazır sayılmamalıdır. |
| SYS-READY-004 | ZORUNLU | Ayarlar doğrulama hatası içeriyorsa sistem denetim kabul etmemeli ve hatalı alanları operatöre göstermelidir. |
| SYS-READY-005 | ZORUNLU | Hazır olmayan sistem, PLC tetiklemesini başarılı ölçüm gibi yanıtlamamalıdır; hata sonucu üretmelidir. |

### 6.3 PLC denetim çevrimi

| Kimlik | Öncelik | Gereksinim |
|---|---|---|
| PLC-FLOW-001 | ZORUNLU | Sistem `Dormant`, `Waiting`, `Working` ve `Holding` durumlarını desteklemelidir. |
| PLC-FLOW-002 | ZORUNLU | Başlatma başarıyla tamamlandığında sistem `Waiting` durumuna geçmelidir. |
| PLC-FLOW-003 | ZORUNLU | Yalnızca `Waiting`, `Inspect=true` ve `ResultACK=false` birlikteyken yeni denetim başlatılmalıdır. |
| PLC-FLOW-004 | ZORUNLU | Kabul edilen her tetik tek bir denetim oluşturmalı; sistem önceki istekten sonra `Inspect=false` değerini görmeden yeni bir denetim için yeniden kurulmamalıdır. Böylece Inspect sinyali yüksek kalırsa aynı tomruk için ikinci denetim başlamamalıdır. |
| PLC-FLOW-005 | ZORUNLU | Denetim başlatıldığında durum `Working` olmalıdır. |
| PLC-FLOW-006 | ZORUNLU | Denetim tamamlanmadan hiçbir nihai sonuç alanı geçerli olarak sunulmamalıdır. |
| PLC-FLOW-007 | ZORUNLU | Sonuç alanları eksiksiz yazıldıktan sonra durum `Holding` olmalıdır. |
| PLC-FLOW-008 | ZORUNLU | Sistem `Holding` durumunda PLC onayını beklemeli ve sonuç değerlerini değiştirmemelidir. |
| PLC-FLOW-009 | ZORUNLU | `ResultACK=true` alındığında bütün sonuç alanları temizlenmeli ve sistem `Waiting` durumuna dönmelidir. |
| PLC-FLOW-010 | ZORUNLU | PLC bağlantı hatasında sistem hatayı loglamalı, operatöre göstermeli ve kontrollü yeniden bağlantı denemelidir. |
| PLC-FLOW-011 | ZORUNLU | Sistem heartbeat değerini çevrimsel olarak izlemeli/yansıtmalı; heartbeat kaybı hazır durumunu düşürmelidir. |
| PLC-FLOW-012 | ZORUNLU | Uygulama kontrollü biçimde durdurulduğunda mümkünse `Dormant` durumu yayınlanmalıdır. |

### 6.4 Denetim kimliği ve izlenebilirlik

| Kimlik | Öncelik | Gereksinim |
|---|---|---|
| TRACE-001 | ZORUNLU | Her denetim sistem genelinde benzersiz bir kimliğe sahip olmalıdır. |
| TRACE-002 | ZORUNLU | Kimlik en az tarih, saat ve milisaniye hassasiyetini içermelidir. |
| TRACE-003 | ZORUNLU | Simülasyon sonuçları üretim sonuçlarından açıkça ayırt edilmelidir. |
| TRACE-004 | ZORUNLU | Denetim kimliği; sonuç, detay logu, özet kaydı ve saklanan ham veride aynı olmalıdır. |
| TRACE-005 | ÖNERİLEN | Kimlik oluşturma işlemi klasör adı uzunluğu veya karakterleri nedeniyle başarısız olmamalıdır. |

## 7. Kamera ve veri edinim gereksinimleri

### 7.1 Kamera yönetimi

| Kimlik | Öncelik | Gereksinim |
|---|---|---|
| CAM-001 | ZORUNLU | Sistem üç bağımsız 3B kamera kanalını desteklemelidir. |
| CAM-002 | ZORUNLU | Her kamera ayrı ayrı etkinleştirilebilir/devre dışı bırakılabilir olmalıdır. |
| CAM-003 | ZORUNLU | Kamera bağlantı durumu `Bağlı`, `Bağlı Değil`, `Devre Dışı` ve `Hata` olarak ayırt edilmelidir. |
| CAM-004 | ZORUNLU | Etkin her kameranın cihaz kimliği yapılandırılabilir olmalıdır. |
| CAM-005 | ZORUNLU | Kamera bağlantısı koptuğunda sistem kontrollü yeniden bağlantı denemeli; denetim sırasında koparsa ilgili denetim `MeasurementError` olmalıdır. |
| CAM-006 | ZORUNLU | Etkin kameraların çekimi aynı denetim bağlamında paralel başlatılabilmelidir. |
| CAM-007 | ZORUNLU | Kamera 1/3 ve kamera 2 için ayrı bekleme süreleri uygulanabilmelidir. |
| CAM-008 | ZORUNLU | Her etkin kamera çekimi belirli bir zaman aşımına sahip olmalıdır. |
| CAM-009 | ZORUNLU | Etkin kameradan null, boş veya okunamayan veri gelirse sistem bunu boş modelle gizlememeli; denetimi ölçüm hatası olarak işaretlemelidir. |
| CAM-010 | ÖNERİLEN | Kamera parametrelerinin uygulanıp uygulanmadığı başlangıç logunda ayrı ayrı raporlanmalıdır. |

### 7.2 Kamera verisi

| Kimlik | Öncelik | Gereksinim |
|---|---|---|
| CAM-DATA-001 | ZORUNLU | Her kamera çekimi en az X, Y ve Z nokta bulutu bileşenlerini sağlamalıdır. |
| CAM-DATA-002 | ÖNERİLEN | Mevcutsa renk/doku görüntüsü aynı denetim ve kamera kimliğiyle ilişkilendirilmelidir. |
| CAM-DATA-003 | ZORUNLU | Giriş verisinin koordinat birimi açıkça tanımlanmalı ve bütün algoritma boyunca tek birim kullanılmalıdır. Hedef birim milimetredir. |
| CAM-DATA-004 | ZORUNLU | Her kameranın ana dönüşümü ve ince hizalama dönüşümü versiyonlanmış ayarlardan alınmalıdır. |
| CAM-DATA-005 | ZORUNLU | Dönüşüm matrisleri kullanılmadan önce boyut ve sayısal geçerlilik açısından doğrulanmalıdır. |

## 8. Simülasyon ve tekrar oynatma

| Kimlik | Öncelik | Gereksinim |
|---|---|---|
| SIM-001 | ZORUNLU | Simülasyon modu gerçek kameraya erişmeden kayıtlı 3B verilerle aynı ölçüm hattını çalıştırmalıdır. |
| SIM-002 | ZORUNLU | Bir simülasyon veri seti, etkin her kamera için açıkça eşleştirilebilen bir dosya içermelidir. |
| SIM-003 | ZORUNLU | Eksik veya bozuk simülasyon dosyası o veri seti için `MeasurementError` üretmelidir; toplu çalışmanın kalan veri setleri işlenebilmelidir. |
| SIM-004 | ZORUNLU | Tek veri seti çalıştırma ve bir klasördeki bütün veri setlerini toplu çalıştırma ayrı komutlar olmalıdır. |
| SIM-005 | ZORUNLU | Toplu çalışma her veri seti için sonuç durumu, ölçümler, süre ve hata nedenini raporlamalıdır. |
| SIM-006 | ZORUNLU | Toplu çalışma sırasında geçici ayar değişiklikleri işlem başarılı veya hatalı bitse de eski değerlerine geri alınmalıdır. |
| SIM-007 | ÖNERİLEN | Toplu çalışma özeti makine tarafından okunabilir CSV veya JSON biçiminde dışa aktarılabilmelidir. |
| SIM-008 | ZORUNLU | Simülasyon çalışması üretim sayacı ve üretim özetiyle karıştırılmamalıdır veya kayıtlarda açıkça `Simulation` olarak işaretlenmelidir. |

## 9. 3B işleme ve ölçüm gereksinimleri

### 9.1 Ön işleme

| Kimlik | Öncelik | Gereksinim |
|---|---|---|
| PROC-001 | ZORUNLU | Etkin kameralardan gelen nokta bulutları kalibrasyon dönüşümleriyle ortak koordinat sistemine taşınmalıdır. |
| PROC-002 | ZORUNLU | Her kamera için X, Y ve Z kırpma sınırları uygulanabilmelidir. |
| PROC-003 | ZORUNLU | Kırpma sonunda hiçbir geçerli nokta kalmazsa denetim `MeasurementError` olmalıdır. |
| PROC-004 | ZORUNLU | Geçerli kamera nokta bulutları tek bir tomruk aday modelinde birleştirilmelidir. |
| PROC-005 | ZORUNLU | Gürültü/küçük bileşenler yapılandırılabilir fiziksel boyut eşiğiyle elenmelidir. |
| PROC-006 | ZORUNLU | Eleme sonunda tomruk adayı bulunamazsa denetim `MeasurementError` olmalıdır. |
| PROC-007 | ZORUNLU | İşleme yoğunluğunu kontrol etmek için yapılandırılabilir örnekleme mesafesi uygulanabilmelidir. |
| PROC-008 | ZORUNLU | Ölçümden önce tomruğun iki ucundan yapılandırılabilir trim mesafesi çıkarılmalıdır. |
| PROC-009 | ZORUNLU | İki uç trim toplamı ölçülen ham uzunluğa eşit veya büyükse ayar/veri hatası üretilmelidir. |

### 9.2 Dilim oluşturma ve geometrik model

| Kimlik | Öncelik | Gereksinim |
|---|---|---|
| SLICE-001 | ZORUNLU | Tomruk, ekseni boyunca yapılandırılabilir pencere genişliği ve adımıyla dilimlenmelidir. |
| SLICE-002 | ZORUNLU | Her dilim için silindirik model uydurma denenmelidir. |
| SLICE-003 | ZORUNLU | Başarılı dilimlerden merkez, eksen yönü, yarıçap, nokta sayısı ve eksen konumu elde edilmelidir. |
| SLICE-004 | ZORUNLU | Model uydurulamayan dilim geçersiz sayılmalı ve nedeni teşhis verisine eklenmelidir. |
| SLICE-005 | ZORUNLU | Dilimlerin veri kalite durumu; `Valid`, `LowPointCount`, `AngleOutlier`, `RadiusOutlier`, `CenterOutlier`, `Flare` veya `FitFailed` olarak raporlanabilmelidir. |
| SLICE-006 | ZORUNLU | Aykırı değer kuralları, hareketli medyan penceresi ve fiziksel sınırlar yapılandırılabilir olmalıdır. |
| SLICE-007 | ZORUNLU | Yarıçap/çap terimleri ayar, arayüz, log ve kod sözleşmelerinde birbirine karıştırılmamalıdır. |

### 9.3 Ölçüm güvenilirliği

| Kimlik | Öncelik | Gereksinim |
|---|---|---|
| QUALITY-001 | ZORUNLU | Tomruk, uzunluğu boyunca yapılandırılabilir sayıda sektöre ayrılmalıdır. |
| QUALITY-002 | ZORUNLU | Her sektördeki geçerli dilim sayısı hesaplanmalıdır. |
| QUALITY-003 | ZORUNLU | Gerekli minimum sektör kapsaması sağlanmazsa Plan A sonucu geçersiz kabul edilmelidir. |
| QUALITY-004 | ZORUNLU | Geçersiz Plan A sonucu fiziksel sehim değeriymiş gibi `998`, `999` vb. özel sayı kullanılarak raporlanmamalıdır. |
| QUALITY-005 | ZORUNLU | Ölçüm hatası, hata kodu ve açıklamayla ayrı bir sonuç alanı olarak tutulmalıdır. |
| QUALITY-006 | ZORUNLU | Ölçüm güvenilir değilse ürün `Accepted` olarak yayınlanmamalıdır. |

### 9.4 Yön ve flare

| Kimlik | Öncelik | Gereksinim |
|---|---|---|
| ORIENT-001 | ZORUNLU | Sistem `Unknown`, `ButtFirst` ve `ButtLast` yönlerini desteklemelidir. |
| ORIENT-002 | ZORUNLU | Yön hesabı geçerli yarıçap eğilimine dayanmalı ve flare nedeniyle bozulmamalıdır. |
| ORIENT-003 | ZORUNLU | İkinci yön yöntemi ilk ve ikinci yarının temsilî çaplarını karşılaştırmalıdır. |
| ORIENT-004 | ZORUNLU | Yön için yeterli veri yoksa tahmin yapılmamalı, `Unknown` dönülmelidir. |
| FLARE-001 | ZORUNLU | Flare araması yalnızca yönün gösterdiği dip uç bölgesinde yapılmalıdır. |
| FLARE-002 | ZORUNLU | Flare başlangıcı yapılandırılabilir yarıçap değişim eşiğiyle belirlenmelidir. |
| FLARE-003 | ZORUNLU | Flare bölgesi sehim ve normal gövde çapı hesabından çıkarılmalıdır. |
| FLARE-004 | ZORUNLU | Flare bulunup bulunmadığı sonuçta bağımsız boolean alan olarak sunulmalıdır. |

### 9.5 Uzunluk ve çap

| Kimlik | Öncelik | Gereksinim |
|---|---|---|
| DIM-001 | ZORUNLU | Uzunluk, ortak koordinat sisteminde tomruk ekseni boyunca ölçülmeli ve milimetre cinsinden raporlanmalıdır. |
| DIM-002 | ZORUNLU | Minimum, medyan ve maksimum gövde çapları flare ve aykırı dilimler çıkarıldıktan sonra geçerli yarıçapların iki katından hesaplanmalıdır. |
| DIM-003 | ZORUNLU | `DiameterMid` terimi medyan çap anlamına gelmelidir. Eğer fiziksel orta noktadaki çap isteniyorsa ayrı `DiameterAtMidpoint` alanı kullanılmalıdır. |
| DIM-004 | ZORUNLU | Birinci ve ikinci yarı çapları yarıçap değil, çap olarak raporlanmalıdır. |
| DIM-005 | ZORUNLU | Bütün çap değerleri milimetre cinsinden ve negatif olmayan tamsayı olarak PLC'ye yayınlanmalıdır. |

### 9.6 Sehim

| Kimlik | Öncelik | Gereksinim |
|---|---|---|
| DEF-001 | ZORUNLU | Sehim, flare ve aykırı dilimler çıkarıldıktan sonra geçerli tomruk gövdesinden hesaplanmalıdır. |
| DEF-002 | ZORUNLU | Tomruk, tam 360°'yi kapsayacak açılarda değerlendirilmelidir. |
| DEF-003 | ZORUNLU | Açı adımı pozitif olmalı ve 360°'yi kalansız bölmelidir. |
| DEF-004 | ZORUNLU | Her açıda referans doğru tomruğun iki geçerli uç ölçümüne göre kurulmalıdır. |
| DEF-005 | ZORUNLU | Her açıdaki mutlak maksimum sapma hesaplanmalıdır; pozitif ve negatif sapmaların nasıl birleştirileceği tek bir belgelenmiş formülle tanımlanmalıdır. |
| DEF-006 | ZORUNLU | Nihai sehim, değerlendirilen açılar içindeki en büyük geçerli sehim olmalıdır. |
| DEF-007 | ZORUNLU | Uygulanan herhangi bir düzeltme katsayısı, ham ve düzeltilmiş değerlerle birlikte loglanmalıdır. |
| DEF-008 | ZORUNLU | Operatör ekranında nihai sehim, bulunduğu açı ve ilgili dilim/konum gösterilmelidir. |
| DEF-009 | ONAY GEREKLİ | Mevcut `DefCoeff=0.7` düzeltmesinin fiziksel gerekçesi, uygulanacağı aralık ve kalibrasyon yöntemi proses sahibi tarafından onaylanmalıdır. |

### 9.7 Plan B'nin rolü

| Kimlik | Öncelik | Gereksinim |
|---|---|---|
| PLANB-001 | ZORUNLU | Plan B sonucu Plan A'dan ayrı hesaplanmalı ve ayrı isimle loglanmalıdır. |
| PLANB-002 | ZORUNLU | Plan B nihai sonuçta kullanılmıyorsa sistem bunu “seçildi” olarak saymamalıdır. |
| PLANB-003 | ZORUNLU | Plan B'nin yalnızca teşhis, doğrulama veya otomatik yedek yöntem rollerinden hangisine sahip olduğu yapılandırma değil, onaylı iş kuralı olmalıdır. |
| PLANB-004 | ONAY GEREKLİ | Plan A geçersiz olduğunda Plan B ile kabul/ret kararı verilip verilmeyeceği proses sahibi tarafından belirlenmelidir. Bu karar verilene kadar Plan A geçersizliği `MeasurementError` üretmelidir. |

## 10. Kabul ve ret kuralları

### 10.1 Önerilen karar kuralı

| Kimlik | Öncelik | Gereksinim |
|---|---|---|
| DECISION-001 | ZORUNLU | İzin verilen maksimum sehim `LengthMm × MaxDeflectionPerMeterMm / 1000` olarak hesaplanmalıdır. |
| DECISION-002 | ZORUNLU | Ölçüm geçerliyse ve ölçülen sehim izin verilen değere eşit veya küçükse sonuç `Accepted` olmalıdır. |
| DECISION-003 | ZORUNLU | Ölçüm geçerliyse ve ölçülen sehim izin verilen değerden büyükse sonuç `Rejected` olmalıdır. |
| DECISION-004 | ZORUNLU | Ölçüm geçersizse sonuç `MeasurementError` olmalı ve PLC `Outcome=false` almalıdır. |
| DECISION-005 | ZORUNLU | Flare ve yön bilgisi, ayrıca tanımlanmış bir ret kuralı olmadıkça tek başına kabul/ret sonucunu değiştirmemelidir. |
| DECISION-006 | ONAY GEREKLİ | Eşik değerine tam eşit sehim için kabul davranışı proses sahibi tarafından onaylanmalıdır. Bu belgede önerilen davranış kabuldür. |

Karar tablosu:

| Ölçüm geçerli mi? | Sehim sınır içinde mi? | Sonuç |
|---|---|---|
| Hayır | Uygulanamaz | MeasurementError / PLC Outcome=false |
| Evet | Evet | Accepted / PLC Outcome=true |
| Evet | Hayır | Rejected / PLC Outcome=false |

## 11. Sonuç veri sözleşmesi

### 11.1 Mantıksal sonuç modeli

| Alan | Tip | Birim | Zorunluluk |
|---|---|---|---|
| InspectionId | String | - | Her sonuçta zorunlu |
| ResultStatus | Enum | Accepted/Rejected/MeasurementError | Her sonuçta zorunlu |
| Outcome | Boolean | - | PLC uyumluluğu için zorunlu |
| ErrorCode | Integer/Enum | - | MeasurementError durumunda zorunlu |
| ErrorMessage | String | - | MeasurementError durumunda zorunlu |
| Length | Integer | mm | Geçerli ölçümde zorunlu |
| Deflection | Integer | mm | Geçerli ölçümde zorunlu |
| AllowedDeflection | Integer | mm | Geçerli ölçümde zorunlu |
| DeflectionAngle | Integer | derece | Geçerli ölçümde zorunlu |
| DiameterMin | Integer | mm | Geçerli ölçümde zorunlu |
| DiameterMedian | Integer | mm | Geçerli ölçümde zorunlu |
| DiameterMax | Integer | mm | Geçerli ölçümde zorunlu |
| FirstHalfDiameter | Integer | mm | Yeterli veri varsa zorunlu |
| SecondHalfDiameter | Integer | mm | Yeterli veri varsa zorunlu |
| Orientation | Enum | Unknown/ButtFirst/ButtLast | Her sonuçta zorunlu |
| OrientationSecondary | Enum | Unknown/ButtFirst/ButtLast | Her sonuçta zorunlu |
| FlareExists | Boolean | - | Her sonuçta zorunlu |
| MeasurementDuration | Integer | ms | Her sonuçta zorunlu |
| AlgorithmVersion | String | - | Her sonuçta zorunlu |
| SettingsVersion | String/hash | - | Her sonuçta zorunlu |

### 11.2 Mevcut PLC sözleşmesiyle uyumluluk

İlk uyumlu teslimat aşağıdaki mevcut alanları korumalıdır:

- Status
- Heartbeat IN/OUT
- Inspect
- ResultACK
- Outcome
- Length
- Deflection
- DiameterMin/Mid/Max
- Orientation/Orientation2
- FlareExist
- FirstHalfDiameter/SecondHalfDiameter

| Kimlik | Öncelik | Gereksinim |
|---|---|---|
| PLC-DATA-001 | ZORUNLU | Mevcut EtherNet/IP tag adları ve Modbus register adresleri, PLC tarafında birlikte onaylanmış bir versiyon değişikliği olmadan değiştirilmemelidir. |
| PLC-DATA-002 | ZORUNLU | PLC'ye yazılan her sayısal ölçümün birimi sözleşmede belirtilmelidir. |
| PLC-DATA-003 | ZORUNLU | Int32 alanların byte/register sırası entegrasyon testiyle doğrulanmalıdır. |
| PLC-DATA-004 | ÖNERİLEN | `ErrorCode` ve sonuç türü için PLC sözleşmesine yeni alan eklenmelidir. |
| PLC-DATA-005 | ZORUNLU | Yeni alan eklenemiyorsa ölçüm hatası `Outcome=false` ve belgelenmiş güvenli sayısal değerlerle temsil edilmeli; özel değerler fiziksel ölçüm gibi gösterilmemelidir. |

## 12. Kullanıcı arayüzü gereksinimleri

### 12.1 Ana görünüm

| Kimlik | Öncelik | Gereksinim |
|---|---|---|
| UI-001 | ZORUNLU | Ana görünüm üç kamera kanalının durumunu göstermelidir. |
| UI-002 | ZORUNLU | PLC bağlantısı, Inspect, ACK ve iş akışı durumu gösterilmelidir. |
| UI-003 | ZORUNLU | Çalışma modu `Production` veya `Simulation` olarak belirgin biçimde gösterilmelidir. |
| UI-004 | ZORUNLU | Son denetimin kimliği, sonucu, sehim, izin verilen sehim, uzunluk, çaplar, yön ve flare bilgisi gösterilmelidir. |
| UI-005 | ZORUNLU | Accepted yeşil, Rejected kırmızı, MeasurementError farklı ve açık bir hata rengi/metniyle gösterilmelidir; yalnızca renge güvenilmemelidir. |
| UI-006 | ZORUNLU | Toplam tetik, tamamlanan, kabul, ret ve ölçüm hatası sayaçları ayrı gösterilmelidir. |
| UI-007 | ZORUNLU | Oranların paydası ve kapsamı açık olmalıdır; simülasyon ve üretim sayaçları karıştırılmamalıdır. |
| UI-008 | ÖNERİLEN | Sayaçların sıfırlanma zamanı veya oturum başlangıcı gösterilmelidir. |

### 12.2 Operatör komutları

| Kimlik | Öncelik | Gereksinim |
|---|---|---|
| UI-CMD-001 | ZORUNLU | Canlı tek denetim, tek simülasyon veri seti ve toplu simülasyon farklı isimli komutlar olmalıdır. |
| UI-CMD-002 | ZORUNLU | Üretim modunda manuel canlı denetim yalnızca proses güvenliği tarafından izin veriliyorsa etkin olmalıdır. |
| UI-CMD-003 | ZORUNLU | `Continue` komutu yalnızca bekleyen teşhis görselleştirmesini devam ettirmeli; denetim veya PLC akışını belirsiz biçimde değiştirmemelidir. |
| UI-CMD-004 | ZORUNLU | Ham veriyi yalnızca sonraki denetim için saklama seçeneği açıkça gösterilmelidir. |

### 12.3 Ayarlar

| Kimlik | Öncelik | Gereksinim |
|---|---|---|
| UI-SET-001 | ZORUNLU | Ayarlar anlamlı gruplara ayrılmalıdır: PLC, kamera, kalibrasyon, 3B işleme, karar, kayıt ve simülasyon. |
| UI-SET-002 | ZORUNLU | Her sayısal ayarın birimi ve geçerli aralığı arayüzde gösterilmelidir. |
| UI-SET-003 | ZORUNLU | Geçersiz ayar kaydedilmemeli; alan bazında hata açıklaması gösterilmelidir. |
| UI-SET-004 | ZORUNLU | Ayar değişikliklerinin hemen mi yoksa sonraki denetimde/yeniden başlatmada mı geçerli olduğu belirtilmelidir. |
| UI-SET-005 | ZORUNLU | Devam eden denetim, başlangıçta aldığı değişmez ayar snapshot'ıyla tamamlanmalıdır. |
| UI-SET-006 | ÖNERİLEN | Kalibrasyon matrisleri günlük operatör ekranından ayrı, kontrollü bir uzman görünümünde düzenlenmelidir. |
| UI-SET-007 | ÖNERİLEN | Ayarlar dışa aktarılabilir, içe aktarılabilir ve fabrika varsayılanlarına döndürülebilir olmalıdır. |

### 12.4 Teşhis

| Kimlik | Öncelik | Gereksinim |
|---|---|---|
| UI-DIAG-001 | ZORUNLU | Dilim bazında nokta sayısı, açı, yarıçap, merkez ve geçerlilik nedeni incelenebilmelidir. |
| UI-DIAG-002 | ZORUNLU | Ham, dönüştürülmüş, kırpılmış ve ölçülmüş 3B modeller isteğe bağlı gösterilebilmelidir. |
| UI-DIAG-003 | ZORUNLU | Görselleştirme kapalı olduğunda denetim kullanıcı etkileşimi beklememelidir. |
| UI-DIAG-004 | ZORUNLU | Görselleştirme bekleme süresi yapılandırılabilir olmalı ve süre sonunda denetim tanımlı biçimde devam etmeli veya iptal edilmelidir. |
| UI-DIAG-005 | ÖNERİLEN | Seçilen sehim açısında referans doğru, en yüksek sapma ve ilgili dilim görsel olarak işaretlenmelidir. |

## 13. Ayar gereksinimleri ve doğrulama kuralları

### 13.1 Birimler

| Ayar grubu | Birim |
|---|---|
| Uzunluk, çap, yarıçap, trim, örnekleme, merkez sapması | mm |
| Sehim ve metre başına sehim | mm, mm/m |
| Rotasyon ve açı sapması | derece |
| Kamera bekleme ve zaman aşımı | ms |
| Flare bölgesi ve göreli sapma | 0-1 oran veya açıkça `%`; tek gösterim seçilmelidir |

### 13.2 Asgari doğrulamalar

| Kimlik | Öncelik | Gereksinim |
|---|---|---|
| CFG-001 | ZORUNLU | PLC IP adresi geçerli IPv4/hostname olmalıdır. |
| CFG-002 | ZORUNLU | Etkin kameraların cihaz kimliği boş olmamalıdır. |
| CFG-003 | ZORUNLU | Kamera bekleme süreleri ve zaman aşımları sıfır veya pozitif olmalıdır. |
| CFG-004 | ZORUNLU | Tarama adımı ve pencere genişliği pozitif, tarama adımı pencere genişliğine eşit veya küçük olmalıdır. |
| CFG-005 | ZORUNLU | Trim uzunluğu sıfır veya pozitif olmalıdır. |
| CFG-006 | ZORUNLU | Sektör sayısı ve medyan pencere boyutu en az bir olmalıdır. |
| CFG-007 | ZORUNLU | Dönüş açı adımı `0 < angle ≤ 180` koşulunu sağlamalı ve 360'ı kalansız bölmelidir. |
| CFG-008 | ZORUNLU | Flare bölgesi oranı 0 ile 1 arasında olmalıdır. |
| CFG-009 | ZORUNLU | Düzeltme katsayısı pozitif olmalı; önerilen üst sınır 1'dir. |
| CFG-010 | ZORUNLU | Her kırpma ekseninde minimum maksimumdan küçük olmalıdır. |
| CFG-011 | ZORUNLU | İnce rotasyon/öteleme dizileri tam üç, ana dönüşüm matrisleri tam on iki sayısal değer içermelidir. |
| CFG-012 | ZORUNLU | Ayar kaydı atomik olmalı; yarım veya bozuk dosya önceki geçerli ayarı kaybettirmemelidir. |
| CFG-013 | ZORUNLU | Her kaydedilmiş ayar seti versiyon ve değiştirilme zamanı içermelidir. |

Mevcut varsayılan değerlerin başlangıç adayı olarak kullanılması için `CURRENT_SYSTEM_REQUIREMENTS.md` bölüm 6 esas alınır. Varsayılanlar proses doğrulaması yapılmadan kalibrasyon değeri olarak kabul edilmemelidir.

## 14. Kayıt, log ve veri saklama

| Kimlik | Öncelik | Gereksinim |
|---|---|---|
| LOG-001 | ZORUNLU | Her denetim başlangıç/bitiş zamanı, kimlik, mod, sonuç, ölçümler, hata ve toplam süreyle loglanmalıdır. |
| LOG-002 | ZORUNLU | Kamera edinimi, ön işleme, dilimleme ve ölçüm aşamalarının süreleri ayrı izlenebilmelidir. |
| LOG-003 | ZORUNLU | Log satırlarında uygulama, algoritma ve ayar versiyonu bulunmalıdır. |
| LOG-004 | ZORUNLU | Log yazma hatası denetim sürecini çökertmemeli; mümkün olan başka bir kanaldan görünür olmalıdır. |
| LOG-005 | ZORUNLU | Saatlik/günlük özet, Accepted, Rejected ve MeasurementError sonuçlarını ayırt etmelidir. |
| LOG-006 | ZORUNLU | Ham veri saklama tetikleri ayrı ayrı yapılandırılabilmelidir: manuel, ret, ölçüm hatası ve özel teşhis. |
| LOG-007 | ZORUNLU | Kaydedilen ham veri; denetim kimliği, kamera numarası, çekim zamanı, sonuç ve ayar versiyonuyla ilişkilendirilmelidir. |
| LOG-008 | ZORUNLU | Dosya ve klasör yolları yapılandırılabilir olmalı; varsayılan olarak mevcut `C:\Envisage\Balcas` yapısı desteklenmelidir. |
| LOG-009 | ZORUNLU | Kayıt hedefi erişilemezse denetim sonucu kaybolmamalı; hata operatöre ve loga yansıtılmalıdır. |
| LOG-010 | ONAY GEREKLİ | Ham veri ve logların saklama süresi ile disk doluluk politikası operasyon/IT tarafından belirlenmelidir. |

## 15. Hata yönetimi

### 15.1 Hata sınıfları

| Kod grubu | Örnekler | Sonuç davranışı |
|---|---|---|
| ConfigurationError | Geçersiz ayar, eksik kamera kimliği | Denetim başlamaz, MeasurementError |
| CameraError | Bağlantı, timeout, boş veri | MeasurementError |
| PlcCommunicationError | Okuma/yazma, heartbeat kaybı | Ready düşer; güvenli yeniden bağlantı |
| InputQualityError | Tomruk bulunamadı, yetersiz sektör | MeasurementError |
| ProcessingError | Beklenmeyen algoritma exception'ı | MeasurementError |
| StorageError | Log/ham veri yazılamadı | Denetim sonucu korunur; ayrı operasyon hatası |

| Kimlik | Öncelik | Gereksinim |
|---|---|---|
| ERR-001 | ZORUNLU | Her hata kararlı bir hata koduna ve insan tarafından okunabilir açıklamaya sahip olmalıdır. |
| ERR-002 | ZORUNLU | Hata kodları fiziksel ölçüm alanlarında sihirli sayı olarak taşınmamalıdır. |
| ERR-003 | ZORUNLU | Beklenmeyen exception uygulamanın sonraki denetimleri kabul etmesini sessizce engellememelidir. |
| ERR-004 | ZORUNLU | Yeniden deneme yalnızca tekrar edilmesi güvenli işlemlerde yapılmalıdır. Aynı PLC tetiklemesi için ikinci sonuç üretilmemelidir. |
| ERR-005 | ZORUNLU | Kritik hata sonrasında sistemin `Waiting`, `Holding`, `Dormant` veya `Fault` durumlarından hangisinde olduğu açık olmalıdır. |
| ERR-006 | ÖNERİLEN | Tekrarlanan hatalarda log selini önleyen oran sınırlaması uygulanmalıdır; ilk hata ve değişen hata ayrıntıları kaybolmamalıdır. |

## 16. Fonksiyonel olmayan gereksinimler

### 16.1 Performans

| Kimlik | Öncelik | Gereksinim |
|---|---|---|
| NFR-PERF-001 | ZORUNLU | Denetim süresi kamera edinimi ve hesaplama olarak ayrı ölçülmelidir. |
| NFR-PERF-002 | ONAY GEREKLİ | PLC tetiklemesinden `Holding` durumuna kadar izin verilen maksimum süre saha çevrim süresine göre belirlenmelidir. |
| NFR-PERF-003 | ZORUNLU | Kullanıcı arayüzü kamera çekimi, 3B işleme ve dosya yazma sırasında yanıt verebilir kalmalıdır. |
| NFR-PERF-004 | ÖNERİLEN | Toplu simülasyon, görselleştirme kapalıyken kullanıcı etkileşimi beklememelidir. |

### 16.2 Güvenilirlik ve emniyetli davranış

| Kimlik | Öncelik | Gereksinim |
|---|---|---|
| NFR-REL-001 | ZORUNLU | Sistem güvenilir ölçüm üretemediğinde varsayılan davranış kabul değil hata/ret yönünde olmalıdır. |
| NFR-REL-002 | ZORUNLU | Sonuç alanları ile durum alanı PLC tarafından yarım sonuç okunmasını önleyen sırada yazılmalıdır. |
| NFR-REL-003 | ZORUNLU | Bir denetimin kaynakları tamamlandığında veya hata verdiğinde serbest bırakılmalıdır. |
| NFR-REL-004 | ZORUNLU | Uzun süreli çalışma sırasında kamera, 3B nesne veya görev kaynaklarının sınırsız büyümediği dayanıklılık testiyle doğrulanmalıdır. |
| NFR-REL-005 | ZORUNLU | Uygulama beklenmeyen biçimde kapanıp yeniden başladığında PLC ile tutarlı duruma dönebilmelidir. |

### 16.3 Bakım ve test edilebilirlik

| Kimlik | Öncelik | Gereksinim |
|---|---|---|
| NFR-MAINT-001 | ZORUNLU | Kamera, PLC, ayar depolama ve ölçüm motoru bağımsız test edilebilir sözleşmelere sahip olmalıdır. |
| NFR-MAINT-002 | ZORUNLU | Kabul/ret kuralı 3B görüntüleme veya kullanıcı arayüzüne bağımlı olmadan birim testine tabi tutulabilmelidir. |
| NFR-MAINT-003 | ZORUNLU | Kayıtlı referans veri setleriyle regresyon testi çalıştırılabilmelidir. |
| NFR-MAINT-004 | ZORUNLU | Aynı veri seti, algoritma ve ayarla tolerans dahilinde aynı sonucu üretmelidir. |
| NFR-MAINT-005 | ZORUNLU | Her yayın benzersiz uygulama ve algoritma sürümüne sahip olmalıdır. |

### 16.4 Güvenlik ve işletim

| Kimlik | Öncelik | Gereksinim |
|---|---|---|
| NFR-OPS-001 | ZORUNLU | PLC, kamera, log ve ayar bağlantı bilgileri kaynak kod değiştirilmeden yapılandırılabilmelidir. |
| NFR-OPS-002 | ZORUNLU | Merkezi log sunucusu erişilemez olduğunda yerel denetim çalışmaya devam etmelidir. |
| NFR-OPS-003 | ÖNERİLEN | Ayar değişiklikleri kullanıcı/istasyon, zaman ve önceki/yeni değerlerle denetlenebilir biçimde kaydedilmelidir. |
| NFR-OPS-004 | ÖNERİLEN | Kalibrasyon ve kritik karar ayarları yetkisiz değişikliğe karşı korunmalıdır. |

## 17. Kabul senaryoları

### AC-001 — Normal kabul

**Verilen:** Sistem hazır, PLC bağlantısı sağlıklı, bütün etkin kameralar bağlı ve geçerli bir tomruk verisi mevcut.  
**Ne zaman:** PLC `Inspect=true` ve `ResultACK=false` yapar.  
**O zaman:** Sistem bir kez denetim yapar, ölçümleri yayınlar, sehim sınır içindeyse `Outcome=true` ve `Holding` durumuna geçer.

### AC-002 — Geometrik ret

**Verilen:** Ölçüm geçerli ve izin verilen sehim 75 mm.  
**Ne zaman:** Ölçülen düzeltilmiş sehim 76 mm olur.  
**O zaman:** Sonuç `Rejected`, PLC Outcome `false` olmalı; ölçülen ve izin verilen değerler loglanmalıdır.

### AC-003 — Kamera verisi eksik

**Verilen:** Kamera 2 etkindir.  
**Ne zaman:** Kamera 2 timeout veya boş veri üretir.  
**O zaman:** Sistem boş modelle devam edip kabul üretmemeli; sonuç `MeasurementError`, PLC Outcome `false` olmalı ve kamera hata kodu kaydedilmelidir.

### AC-004 — Yetersiz sektör kapsaması

**Verilen:** Plan A için gerekli minimum sektör kapsaması sağlanmamıştır.  
**Ne zaman:** Ölçüm kararı oluşturulur.  
**O zaman:** Fiziksel sehim alanına 998 yazılarak kabul üretilmemeli; Plan B yedek karar olarak onaylanmamışsa sonuç `MeasurementError` olmalıdır.

### AC-005 — PLC onayı

**Verilen:** Sistem `Holding` durumunda geçerli sonuçları tutmaktadır.  
**Ne zaman:** PLC `ResultACK=true` yapar.  
**O zaman:** Sonuç alanları temizlenmeli ve sistem yalnızca temizleme tamamlandıktan sonra `Waiting` durumuna geçmelidir.

### AC-006 — Yüksek kalan Inspect

**Verilen:** Bir denetim tamamlanmış, Inspect sinyali hâlâ true ve ResultACK false.  
**Ne zaman:** Sistem Holding durumundadır.  
**O zaman:** İkinci denetim başlamamalıdır.

### AC-007 — Tam açı taraması

**Verilen:** Açı adımı 90°'dir.  
**Ne zaman:** Sehim hesabı yapılır.  
**O zaman:** 0°, 90°, 180° ve 270° açıları tam bir kez değerlendirilmelidir.

### AC-008 — Ayar doğrulama

**Verilen:** Tarama adımı 0 veya dönüş açısı 360'ı kalansız bölmüyor.  
**Ne zaman:** Kullanıcı ayarları kaydetmeye çalışır.  
**O zaman:** Ayar kaydedilmemeli ve ilgili alanlarda açıklayıcı hata gösterilmelidir.

### AC-009 — Toplu simülasyonda bozuk veri

**Verilen:** Bir klasörde on simülasyon veri seti ve bunlardan birinde eksik kamera dosyası vardır.  
**Ne zaman:** Toplu simülasyon çalıştırılır.  
**O zaman:** Eksik veri seti hata olarak raporlanmalı, diğer dokuz veri seti çalıştırılmalı ve özet on satır içermelidir.

### AC-010 — Ayar snapshot'ı

**Verilen:** Bir denetim Working durumunda ve sehim sınırı 15 mm/m'dir.  
**Ne zaman:** Operatör kayıtlı ayarı 20 mm/m yapar.  
**O zaman:** Devam eden denetim 15 mm/m ile tamamlanmalı; yeni değer sonraki denetimde kullanılmalıdır.

### AC-011 — Ölçüm birimleri

**Verilen:** Referans objenin doğrulanmış uzunluğu ve çapları bilinmektedir.  
**Ne zaman:** Referans veri işlenir.  
**O zaman:** UI, log ve PLC değerleri aynı mm biriminde ve belirlenmiş tolerans içinde olmalıdır.

### AC-012 — Yeniden başlatma

**Verilen:** Uygulama bir önceki çalışmada beklenmeyen biçimde kapanmıştır.  
**Ne zaman:** Uygulama yeniden başlatılır.  
**O zaman:** Eski sonuçlar yeni sonuç gibi yayınlanmamalı, PLC ile temiz ve tanımlı bir başlangıç durumu kurulmalıdır.

## 18. Doğrulama yaklaşımı

| Gereksinim alanı | Asgari doğrulama yöntemi |
|---|---|
| Kabul/ret kuralları | Saf birim testleri ve sınır değer testleri |
| PLC durum makinesi | Sahte PLC ile otomatik entegrasyon testleri |
| EtherNet/IP sözleşmesi | Gerçek PLC veya onaylı emulator ile tag testi |
| Modbus register düzeni | Ham register okuma/yazma entegrasyon testi |
| Kamera hata yolları | Timeout, kopma, boş veri ve kısmi kamera testleri |
| 3B algoritma | Etiketlenmiş referans `.om3` veri setleriyle regresyon |
| Birimler ve kalibrasyon | Bilinen ölçülü fiziksel referans obje testi |
| Uzun süreli çalışma | Tekrarlı denetim ve bellek/kaynak dayanıklılık testi |
| Log ve saklama | Disk yok/dolu/erişilemez hata senaryoları |
| UI | Kritik operatör akışları için otomatik veya prosedürel kabul testi |

## 19. Onay bekleyen kararlar

Bu kararlar sonuçlandırılmadan doküman “onaylı gereksinim” statüsüne alınmamalıdır.

| Kimlik | Karar | Önerilen başlangıç |
|---|---|---|
| D-001 | Ölçüm hatası PLC'de nasıl temsil edilecek? | Outcome=false + ayrı ErrorCode |
| D-002 | Plan A geçersizse Plan B karar verebilir mi? | Hayır; doğrulanana kadar MeasurementError |
| D-003 | Eşiğe tam eşit sehim kabul mü? | Evet |
| D-004 | Sehim formülü yalnızca çukur yönünü mü, iki yönlü mutlak sapmayı mı kullanacak? | İki yönlü maksimum mutlak sapma |
| D-005 | `DefCoeff=0.7` bilimsel olarak gerekli mi? | Kalibrasyon testiyle doğrulanana kadar karar bekliyor |
| D-006 | En az kaç etkin kamera üretim ölçümü için yeterli? | Saha kapsama testiyle belirlenmeli |
| D-007 | Bir sektörde gerekli minimum geçerli dilim sayısı nedir? | Referans veriyle belirlenmeli |
| D-008 | Maksimum izin verilen uçtan uca denetim süresi nedir? | Hat çevrim süresinden türetilmeli |
| D-009 | Flare tek başına ret nedeni mi? | Mevcut davranışa uygun olarak hayır |
| D-010 | Yön bilinmiyorsa ürün kararı etkilenir mi? | Proses sahibi belirlemeli |
| D-011 | Ham veri ve log saklama süresi nedir? | IT/operasyon belirlemeli |
| D-012 | Mevcut sabit kamera parametrelerinin hangileri kullanıcı ayarı olmalı? | Kamera uzmanı belirlemeli |

## 20. Mevcut dokümana izlenebilirlik

| Bu belgedeki alan | Mevcut belge bölümü |
|---|---|
| PLC akışı ve sözleşmesi | Bölüm 4.2, 4.3, 4.12 |
| Kamera ve simülasyon | Bölüm 4.5, 4.6 |
| Ön işleme ve dilimleme | Bölüm 4.7, 4.8 |
| Yön ve flare | Bölüm 4.9 |
| Sehim ve sonuç | Bölüm 4.10, 4.11 |
| UI ve görselleştirme | Bölüm 4.13, 4.14 |
| Log ve saklama | Bölüm 5 |
| Varsayılan ayarlar | Bölüm 6 |
| Mevcut çelişkiler | Bölüm 8 |

## 21. Belge onay tablosu

| Rol | İsim | Tarih | Durum |
|---|---|---|---|
| Operasyon sahibi |  |  | Bekliyor |
| Proses/kalite sahibi |  |  | Bekliyor |
| PLC/otomasyon sahibi |  |  | Bekliyor |
| Kamera/görüntü işleme sahibi |  |  | Bekliyor |
| Yazılım sorumlusu |  |  | Bekliyor |
| Test/kabul sorumlusu |  |  | Bekliyor |

# Balcas Müşteri Gereksinimleri Dokümanı

| Belge alanı | Değer |
|---|---|
| Belge kodu | BLC-MGD-001 |
| Belge türü | Müşteri Gereksinimleri Dokümanı (MGD / URS) |
| Sürüm | 0.4 |
| Durum | Müşteri inceleme taslağı |
| Tarih | 28.09.2026 |
| Hazırlayan |  |
| Müşteri | Balcas |


## 1. Amaç

## 2. İş ihtiyacı ve hedefler

Balcas, üretim hattındaki her tomruğun üç boyutlu verisini otomatik olarak inceleyen ve sonucu PLC'ye ileten güvenilir bir kalite kontrol çözümüne ihtiyaç duymaktadır.

Sistemin iş hedefleri şunlardır:

- Her geçerli hat tetiklemesi için yalnızca bir tomruk denetimi yapmak.
- Tomruğun uzunluk, çap, yön, dip şişkinliği (flare) ve sehim bilgilerini üretmek.
- Geçerli ölçüm ile ürün reddini birbirinden ayırmak.
- Güvenilir ölçüm üretilemediğinde durumu “Ölçüm Hatası” olarak kaydetmek; üretim maliyeti kararı gereği PLC'ye `Outcome=true` göndererek tomruğun hatta kabul yönünde ilerlemesini sağlamak.
- Sonuçları operatöre anlaşılır biçimde göstermek ve PLC'ye güvenli biçimde aktarmak.
- Üretim sorunlarının incelenebilmesi için denetimleri izlenebilir şekilde kaydetmek.
- Kayıtlı verilerle üretim hattından bağımsız test ve tekrar analiz yapabilmek.

## 3. Kapsam

### 3.1 Kapsam dahilinde

- Üç bağımsız 3B kamera kanalından veri alınması
- Kamera verilerinin ortak koordinat sisteminde birleştirilmesi
- Tomruğun tespit edilmesi ve ölçüm verisinin kalite kontrolü
- Uzunluk, çap, yön, flare ve sehim ölçümleri
- Kabul, ret ve ölçüm hatası kararının oluşturulması
- PLC ile tetik, durum, sonuç ve onay bilgi alışverişi
- Operatör ekranları, ayarlar ve teşhis görünümü
- Denetim sonuçlarının, hataların ve seçili ham verilerin kaydedilmesi
- Tekli ve toplu kayıtlı veri simülasyonu

### 3.2 Kapsam dışında

Aşağıdaki işler ayrıca yazılı olarak kapsam içine alınmadıkça bu teslimata dahil değildir:

- Üretim hattındaki mekanik hareketlerin PLC adına kontrol edilmesi
- Tomruğun fiziksel olarak yönlendirilmesi, durdurulması veya ayrılması
- Modbus haberleşmesi ve Modbus register entegrasyonu
- ERP/MES ya da bulut sistemi entegrasyonu
- Yapay zekâ tabanlı kalite sınıflandırması
- Otomatik kamera kalibrasyonu
- Kurumsal kullanıcı ve rol yönetimi
- Kamera, PLC, bilgisayar, ağ ve mekanik montaj tedariki

## 4. Paydaşlar ve kullanıcılar

| Rol | Temel beklenti / sorumluluk |
|---|---|
| Operatör | Sistemin durumunu izlemek, hataları anlamak ve izin verilen komutları kullanmak |
| Proses ve kalite sahibi | Ölçüm, tolerans ve kabul/ret kurallarını onaylamak |
| Otomasyon/PLC ekibi | PLC veri sözleşmesini ve el sıkışma akışını onaylamak |
| Kamera/kalibrasyon uzmanı | Kamera yerleşimi, kalibrasyon ve veri kalitesini doğrulamak |
| Bakım/IT | Bilgisayar, ağ, depolama ve yedekleme ortamını işletmek |
| Yazılım tedarikçisi | Onaylı gereksinimleri uygulamak, doğrulamak ve teslim etmek |

## 5. Öncelik ve uygunluk dili

| Terim | Açıklama |
|---|---|
| Zorunlu | Sistem kabulü için karşılanmalıdır. |
| Önerilen | İşletim kalitesini artırır; uygulanmaması halinde gerekçe yazılmalıdır. |
| Opsiyonel | Ayrı planlama veya sonraki faz kapsamında değerlendirilebilir. |
| Karar bekliyor | Müşteri veya ilgili proses sahibi tarafından netleştirilmelidir. |

Bir gereksinimin karşılandığı; test, inceleme, gösterim veya ölçüm yöntemlerinden uygun olanıyla kanıtlanacaktır.

## 6. Genel çalışma senaryosu

1. Sistem açılır, onaylı ayarları yükler ve PLC ile etkin kameralara bağlanır.
2. Gerekli bağlantılar ve ayarlar geçerliyse sistem “Hazır” durumuna geçer.
3. PLC yeni tomruk için denetim isteği gönderir.
4. Sistem etkin kameralardan 3B veriyi alır ve tek bir denetim kaydı oluşturur.
5. Verinin yeterliliği kontrol edilir; geçerliyse ölçümler ve ürün kararı hesaplanır.
6. Sonuç PLC'ye gönderilir ve operatör ekranında gösterilir.
7. Sistem, PLC'nin sonucu aldığını onaylamasına kadar sonucu değiştirmeden tutar.
8. Onaydan sonra sonuç alanları temizlenir ve sistem bir sonraki tomruk için hazır olur.

## 7. Müşteri gereksinimleri

### 7.1 Çalıştırma ve hazır olma

| Kimlik | Öncelik | Gereksinim | Doğrulama |
|---|---|---|---|
| MGR-OPS-001 | Zorunlu | Sistem açıldığında son geçerli ve onaylı ayarları yüklemelidir. | Test |
| MGR-OPS-002 | Zorunlu | Ayarlar eksik, bozuk veya geçersizse sistem operatörü bilgilendirmeli ve güvenilir ölçüm yapıyormuş gibi davranmamalıdır. | Test |
| MGR-OPS-003 | Zorunlu | Üretim modunda bütün etkin kameralar ve PLC haberleşmesi hazır olmadan sistem “Hazır” göstermemelidir. Varsayılan üretim yapılandırmasında üç kamera da etkin olmalıdır. | Test |
| MGR-OPS-004 | Zorunlu | Sistem çalışma modunu “Üretim” veya “Simülasyon” olarak açıkça göstermelidir. | Gösterim |
| MGR-OPS-005 | Zorunlu | Kontrollü kapatma ve beklenmeyen yeniden başlatma sonrasında sistem PLC ile güvenli, tanımlı bir başlangıç durumuna dönmelidir. | Test |

### 7.2 Denetim ve ölçüm

| Kimlik | Öncelik | Gereksinim | Doğrulama |
|---|---|---|---|
| MGR-INS-001 | Zorunlu | Her geçerli PLC tetiklemesi yalnızca bir denetim oluşturmalıdır. Uzun süre aktif kalan bir tetik ikinci denetime neden olmamalıdır. | Test |
| MGR-INS-002 | Zorunlu | Her denetim benzersiz bir kimliğe sahip olmalı; ekranda, loglarda, sonuçlarda ve saklanan verilerde aynı kimlik kullanılmalıdır. | İnceleme/Test |
| MGR-INS-003 | Zorunlu | Sistem üç bağımsız 3B kamera kanalını desteklemeli ve etkin kameraların verisini aynı denetim kapsamında toplamalıdır. Üç kamera varsayılan olarak etkin olmalı; test amacıyla her kamera ayrı ayrı devre dışı bırakılabilmelidir. | Test |
| MGR-INS-004 | Zorunlu | Sistem tomruğun uzunluğunu milimetre cinsinden ölçmelidir. | Referans obje testi |
| MGR-INS-005 | Zorunlu | Sistem minimum, medyan ve maksimum gövde çaplarını milimetre cinsinden ölçmelidir. | Referans obje testi |
| MGR-INS-006 | Zorunlu | Sistem tomruğun yönünü “dip önde”, “dip arkada” veya “bilinmiyor” olarak bildirmelidir. | Etiketli veri testi |
| MGR-INS-007 | Zorunlu | Sistem flare bulunup bulunmadığını ayrı bir sonuç olarak bildirmelidir. | Etiketli veri testi |
| MGR-INS-008 | Zorunlu | Sistem tomruk sehimini 360° çevresinde değerlendirmeli; ayarlardan “Tek yönlü”, “İki yönlü” veya “Her ikisi” hesap modu seçilebilmelidir. Seçilen moda ait sehim değeri/değerleri, ilgili açı ve konumla birlikte bildirilmelidir. | Referans veri testi/Gösterim |
| MGR-INS-009 | Zorunlu | Flare bölgesi ve güvenilir olmadığı belirlenen ölçüm parçaları normal gövde çapı ve sehim hesabını bozmamalıdır. | Referans veri testi |
| MGR-INS-010 | Zorunlu | Ölçümün geçerli sayılması için tanımlı sektörlerin her birinde en az bir geçerli veri bulunmalıdır. Herhangi bir sektörde hiç geçerli veri yoksa sistem sayısal bir ölçüm uydurmamalı ve sonucu “Ölçüm Hatası” olarak vermelidir. | Hata senaryosu testi |
| MGR-INS-011 | Zorunlu | Aynı referans obje/veri, aynı koşullar ve aynı ayarlarla tekrar ölçüldüğünde uzunluk, çap ve sehim sonuçlarının her biri için elde edilen en büyük ve en küçük değer arasındaki fark en fazla 10 mm olmalıdır. | Tekrarlanabilirlik testi |
| MGR-INS-012 | Zorunlu | Sehim ölçümü ve ürün kararı, denetim başlangıcında seçili olan onaylı hesap moduyla tamamlanmalıdır; ölçüm sırasında otomatik olarak başka bir moda geçilmemelidir. | Tasarım incelemesi/Test |

### 7.3 Karar kuralları

| Kimlik | Öncelik | Gereksinim | Doğrulama |
|---|---|---|---|
| MGR-DEC-001 | Zorunlu | Sistem üç farklı sonucu ayırmalıdır: “Kabul”, “Ret” ve “Ölçüm Hatası”. | Test |
| MGR-DEC-002 | Zorunlu | İzin verilen sehim, tomruk uzunluğu ile müşteri tarafından onaylanan metre başına sehim sınırından hesaplanmalıdır. | Hesap kontrolü |
| MGR-DEC-003 | Zorunlu | Geçerli ölçümde sehim izin verilen değere eşit veya küçükse sonuç “Kabul” olmalıdır. | Sınır değer testi |
| MGR-DEC-004 | Zorunlu | Geçerli ölçümde sehim izin verilen değerden büyükse sonuç “Ret” olmalıdır. | Sınır değer testi |
| MGR-DEC-005 | Zorunlu | Güvenilir karar üretilemeyen durumda iç sonuç “Ölçüm Hatası” olmalı; üretim kaybını azaltmak için PLC'ye `Outcome=true` gönderilerek tomruk kabul yönünde ilerletilmelidir. Ölçüm hatası, geçerli bir ölçüm kabulü gibi raporlanmamalı ve hata ayrıntıları kaydedilmelidir. | Hata senaryosu testi |
| MGR-DEC-006 | Zorunlu | Flare bulunması tek başına ret nedeni olmamalıdır. | Kural incelemesi/Test |
| MGR-DEC-007 | Zorunlu | Nihai sehim, ölçüm yönteminin ürettiği değer olmalı; sonuca herhangi bir sehim düzeltme katsayısı uygulanmamalıdır. | Hesap kontrolü/Test |
| MGR-DEC-008 | Zorunlu | Tomruk yönünün belirlenememesi kabul/ret sonucunu etkilememeli; yön “Bilinmiyor” olarak kaydedilmelidir. | Kural incelemesi/Test |
| MGR-DEC-009 | Zorunlu | Sehim hesap modu “Her ikisi” seçildiğinde tek yönlü ve iki yönlü sonuçlar ayrı ayrı raporlanmalı; kabul/ret kararında bu iki değerden büyük olan nihai sehim olarak kullanılmalıdır. | Hesap kontrolü/Test |

### 7.4 PLC entegrasyonu

| Kimlik | Öncelik | Gereksinim | Doğrulama |
|---|---|---|---|
| MGR-PLC-001 | Zorunlu | Sistem; bekleme, çalışma, sonucu tutma ve devre dışı durumlarını PLC ile belirlenmiş sözleşmeye göre yönetmelidir. | Entegrasyon testi |
| MGR-PLC-002 | Zorunlu | Ölçüm tamamlanmadan PLC'ye geçerli nihai sonuç sunulmamalıdır. | Entegrasyon testi |
| MGR-PLC-003 | Zorunlu | Sistem sonuçları PLC onayı gelene kadar değiştirmeden tutmalıdır. | Entegrasyon testi |
| MGR-PLC-004 | Zorunlu | PLC onayı alındıktan sonra sonuçlar temizlenmeli ve sistem yeni denetim için hazırlanmalıdır. | Entegrasyon testi |
| MGR-PLC-005 | Zorunlu | PLC bağlantısı veya heartbeat kaybolduğunda “Hazır” durumu kaldırılmalı, hata gösterilmeli ve kontrollü yeniden bağlantı denenmelidir. | Hata senaryosu testi |
| MGR-PLC-006 | Zorunlu | Mevcut PLC etiketleri, veri tipleri ve veri birimleri iki tarafça onaylanan bir arayüz değişikliği olmadan değiştirilmemelidir. Modbus bu entegrasyonun parçası değildir. | Arayüz incelemesi |
| MGR-PLC-007 | Zorunlu | “Ölçüm Hatası” durumunda PLC `Outcome=true` almalıdır. Sistem, buna rağmen gerçek sonucu operatör ekranında ve kayıtlarda “Ölçüm Hatası” olarak korumalıdır. | Entegrasyon testi |

### 7.5 Operatör arayüzü

| Kimlik | Öncelik | Gereksinim | Doğrulama |
|---|---|---|---|
| MGR-UI-001 | Zorunlu | Ana ekran her kamera kanalının ve PLC bağlantısının durumunu göstermelidir. | Gösterim |
| MGR-UI-002 | Zorunlu | Ana ekran son denetimin kimliğini, sonucunu, uzunluğunu, çaplarını, sehimini, izin verilen sehimi, yönünü ve flare bilgisini göstermelidir. | Gösterim |
| MGR-UI-003 | Zorunlu | Kabul, ret ve ölçüm hatası yalnızca renkle değil, açık metinle de birbirinden ayrılmalıdır. | Gösterim |
| MGR-UI-004 | Zorunlu | Toplam tetik, tamamlanan denetim, kabul, ret ve ölçüm hatası sayaçları ayrı gösterilmelidir. | Test |
| MGR-UI-005 | Zorunlu | Üretim ve simülasyon sonuçları ile sayaçları birbirine karıştırılmamalıdır. | Test |
| MGR-UI-006 | Zorunlu | Geçersiz ayarlar kaydedilmemeli; ilgili alan ve hata nedeni operatöre gösterilmelidir. | Test |
| MGR-UI-007 | Zorunlu | Devam eden denetim, denetim başlangıcındaki ayarlarla tamamlanmalı; sonradan yapılan değişiklik bir sonraki denetimde uygulanmalıdır. | Test |
| MGR-UI-008 | Önerilen | Yetkili kullanıcılar ham, işlenmiş ve ölçülmüş 3B veriyi teşhis amacıyla görüntüleyebilmelidir. | Gösterim |
| MGR-UI-009 | Zorunlu | Operatör arayüzünde manuel denetim komutu bulunmalıdır. Komut yalnızca sistem yeni bir denetim başlatmaya hazırken etkin olmalı; manuel başlatılan denetim kayıtlarda açıkça işaretlenmeli ve aynı tomruk için PLC tetiklemesiyle ikinci bir denetim oluşturulmamalıdır. | Gösterim/Test |

### 7.6 Simülasyon, kayıt ve raporlama

| Kimlik | Öncelik | Gereksinim | Doğrulama |
|---|---|---|---|
| MGR-DATA-001 | Zorunlu | Sistem, gerçek kameraya ihtiyaç duymadan kayıtlı 3B veri üzerinde aynı ölçüm ve karar sürecini çalıştırabilmelidir. | Test |
| MGR-DATA-002 | Zorunlu | Tek veri seti ve bir klasördeki bütün veri setleri ayrı komutlarla çalıştırılabilmelidir. | Gösterim |
| MGR-DATA-003 | Zorunlu | Toplu simülasyonda bozuk bir veri seti diğer veri setlerinin işlenmesini engellememelidir. | Test |
| MGR-DATA-004 | Zorunlu | Toplu simülasyon özeti her veri seti için sonuç, ölçümler, süre ve hata nedenini içermelidir. | Çıktı incelemesi |
| MGR-DATA-005 | Önerilen | Toplu simülasyon özeti CSV veya JSON biçiminde dışa aktarılabilmelidir. | Gösterim |
| MGR-DATA-006 | Zorunlu | Her denetim için başlangıç/bitiş zamanı, kimlik, çalışma modu, sonuç, ölçümler, hata, süre ve yazılım/ayar sürümü kaydedilmelidir. | Kayıt incelemesi |
| MGR-DATA-007 | Zorunlu | Ham verinin manuel, ret, ölçüm hatası veya teşhis amacıyla saklanması ayrı ayrı yapılandırılabilmelidir. | Test |
| MGR-DATA-008 | Zorunlu | Kayıt hedefi geçici olarak erişilemez olduğunda ölçüm sonucu kaybolmamalı; operatöre depolama hatası bildirilmelidir. | Hata senaryosu testi |
| MGR-DATA-009 | Zorunlu | Uygulamanın log veya ham verileri süreye bağlı olarak otomatik silmesi gerekmemektedir. Saklama, arşivleme, yedekleme, disk kotası ve silme işlemleri uygulama dışında IT/operasyon tarafından yönetilebilmelidir. | Tasarım incelemesi/Gösterim |

## 8. Fonksiyonel olmayan gereksinimler

| Kimlik | Öncelik | Gereksinim | Doğrulama |
|---|---|---|---|
| MGR-NFR-001 | Zorunlu | PLC tetiklemesinden sonucun hazır olmasına kadar geçen süre ölçülmeli ve kaydedilmelidir. | Kayıt incelemesi |
| MGR-NFR-002 | Zorunlu | Sabit bir azami uçtan uca denetim süresi kabul koşulu değildir. İstenirse ayarlardan bir uyarı süresi tanımlanabilmeli veya süre denetimi devre dışı bırakılabilmelidir. Tanımlı süre aşılırsa olay kaydedilmeli ve operatöre uyarı verilmelidir; süre aşımı tek başına ürün sonucunu değiştirmemelidir. | Performans testi/Gösterim |
| MGR-NFR-003 | Zorunlu | Kullanıcı arayüzü kamera çekimi, 3B işleme ve dosya yazımı sırasında yanıt verebilir kalmalıdır. | Performans testi |
| MGR-NFR-004 | Zorunlu | Beklenmeyen bir hata sonraki denetimlerin sessizce durmasına yol açmamalı; durum ve hata nedeni görünür olmalıdır. | Dayanıklılık testi |
| MGR-NFR-005 | Zorunlu | Kamera veya veri kalitesi nedeniyle ölçüm üretilemezse hata görünür ve izlenebilir olmalı; müşteri iş kuralı gereği hat akışı için `Outcome=true` gönderilirken iç sonuç “Ölçüm Hatası” olarak korunmalıdır. | Hata senaryosu testi |
| MGR-NFR-006 | Zorunlu | Uzun süreli çalışmada bellek ve cihaz kaynaklarının kontrolsüz büyümediği dayanıklılık testiyle gösterilmelidir. | Uzun süreli test |
| MGR-NFR-007 | Zorunlu | Bütün fiziksel uzunluk ve çap sonuçlarında temel birim milimetre olmalıdır. | İnceleme/Test |
| MGR-NFR-008 | Zorunlu | Her yazılım teslimatı benzersiz uygulama ve ölçüm algoritması sürümüne sahip olmalıdır. | Sürüm incelemesi |
| MGR-NFR-009 | Önerilen | Kritik kalibrasyon ve karar ayarları yetkisiz değişikliğe karşı korunmalı, değişiklikler denetlenebilir olmalıdır. | Güvenlik incelemesi |
| MGR-NFR-010 | Zorunlu | PLC, kamera, kayıt yolu ve ayarlar kaynak kod değiştirilmeden yapılandırılabilmelidir. | Gösterim |

## 9. Asgari müşteri kabul senaryoları

| Kimlik | Senaryo | Beklenen sonuç |
|---|---|---|
| MKA-001 | Sistem hazırken geçerli ve sınır içindeki bir tomruk denetlenir. | Bir denetim oluşur; ölçümler yayınlanır; sonuç “Kabul” olur ve PLC onayına kadar tutulur. |
| MKA-002 | Geçerli ölçümde sehim izin verilen sınırı aşar. | Sonuç “Ret” olur; ölçülen ve izin verilen değerler kaydedilir. |
| MKA-003 | Etkin kameralardan biri zaman aşımına uğrar veya boş veri verir. | İç sonuç “Ölçüm Hatası” olur, kamera hatası kaydedilir ve PLC'ye `Outcome=true` gönderilir. |
| MKA-004 | Tanımlı sektörlerden en az birinde hiç geçerli veri bulunmaz. | Fiziksel ölçüm yerine özel/sihirli sayı kullanılmaz; iç sonuç “Ölçüm Hatası” olur ve PLC'ye `Outcome=true` gönderilir. Her sektörde en az bir geçerli veri bulunduğunda bu kapsam koşulu sağlanmış sayılır. |
| MKA-005 | PLC sonucu onaylar. | Sonuç alanları temizlenir ve sistem yeni denetim için “Hazır” durumuna döner. |
| MKA-006 | PLC tetik sinyali yüksek kalır. | Aynı tomruk için ikinci denetim başlamaz. |
| MKA-007 | Sehim, onaylı sınıra tam eşittir. | Sonuç “Kabul” olur. |
| MKA-008 | Geçersiz bir ayar kaydedilmeye çalışılır. | Kayıt reddedilir ve açıklayıcı alan hatası gösterilir. |
| MKA-009 | On veri setinden biri bozukken toplu simülasyon çalıştırılır. | Bozuk veri hata olarak raporlanır; diğer dokuz veri işlenir ve özet on kayıt içerir. |
| MKA-010 | Çalışan denetim sırasında karar ayarı değiştirilir. | Mevcut denetim eski ayarla, sonraki denetim yeni ayarla tamamlanır. |
| MKA-011 | Aynı referans obje/veri, aynı koşullar ve aynı ayarlarla tekrarlı olarak işlenir. | Ekran, kayıt ve PLC değerleri aynı birimdedir; uzunluk, çap ve sehim sonuçlarının her biri için elde edilen en büyük ve en küçük değer arasındaki fark 10 mm'yi aşmaz. |
| MKA-012 | Uygulama beklenmeyen kapanma sonrasında yeniden açılır. | Eski sonuç yeni sonuç gibi yayınlanmaz; PLC ile güvenli başlangıç durumu kurulur. |
| MKA-013 | Test yapılandırmasında kameralardan biri veya ikisi devre dışı bırakılır. | Sistem yalnızca etkin kameraları bekler, devre dışı kameraları açıkça gösterir ve test denetimini etkin kamera verileriyle çalıştırır. |
| MKA-014 | Geçerli ölçümde flare bulunur ve sehim sınır içindedir. | Flare kaydedilir; tek başına ret oluşturmaz ve sonuç “Kabul” olur. |
| MKA-015 | Yön belirlenemez ve diğer ölçümler geçerlidir. | Yön “Bilinmiyor” kaydedilir; yön belirsizliği kabul/ret sonucunu değiştirmez. |
| MKA-016 | Sehim hesap modu sırasıyla “Tek yönlü”, “İki yönlü” ve “Her ikisi” seçilerek bilinen sehimli referans veri işlenir. | Seçilen moda ait değerler ayrı ve doğru etiketlerle raporlanır; “Her ikisi” modunda büyük değer nihai sehim olarak kullanılır ve hiçbir sonuca düzeltme katsayısı uygulanmaz. |
| MKA-017 | Denetim süresi uyarısı etkin ve tanımlı süre aşılır. | Süre aşımı kaydedilir ve operatöre uyarı verilir; ürün sonucu yalnızca ölçüm ve karar kurallarına göre belirlenir. Süre denetimi kapatıldığında süre uyarısı oluşmaz. |
| MKA-018 | Saklanan log ve ham verilerin harici IT/operasyon politikasıyla arşivlenmesi veya silinmesi gerekir. | Uygulama süreye bağlı otomatik silme yapmaz; veriler uygulama dışında yönetilebilir. |
| MKA-019 | Sistem yeni denetim için hazırken operatör manuel denetim komutunu verir. | Yalnızca bir denetim başlar, kaynak türü “Manuel” olarak kaydedilir ve aynı tomruk için eşzamanlı PLC tetiklemesi ikinci bir denetim başlatmaz. |

Nihai fabrika kabul testi (FAT) ve saha kabul testi (SAT) prosedürleri, bu senaryolar ile onaylanan sayısal hedeflerden türetilecektir.

## 10. Teslimatlar

Asgari teslimat kapsamı aşağıdakileri içerir:

- Çalıştırılabilir Balcas denetim uygulaması ve sürüm bilgisi
- Onaylı varsayılan yapılandırma ve ayar açıklamaları
- PLC arayüz/veri sözleşmesi
- Operatör kullanım talimatı
- Kurulum, yedekleme ve geri yükleme talimatı
- Hata kodları ve sorun giderme listesi
- FAT/SAT test prosedürü ve test sonuçları
- Onaylı referans veri setleri ve regresyon test özeti
- Sürüm notları ve bilinen kısıtlar

## 11. Müşteri girdileri ve sorumlulukları

Müşteri veya müşterinin yetkilendirdiği taraf aşağıdaki girdileri sağlamalı ve onaylamalıdır:

- İsteğe bağlı denetim süresi uyarısının kullanılacağı durumlarda uyarı süresi
- Ölçüm aralıkları ve 10 mm tekrarlanabilirlik parametresinin doğrulanacağı referanslar
- Metre başına izin verilen azami sehim
- Üç kameranın yerleşimi, görüş alanı ve kalibrasyon referansları
- PLC protokolü, etiket listesi ve veri tipleri
- Kabul/ret iş kuralları ve hata halinde hat davranışı
- Referans tomruklar/veri setleri ile beklenen sonuçlar
- Ham veri ve loglar için uygulama dışında yürütülecek disk kotası, arşivleme, yedekleme ve silme politikası
- Üretim ağ erişimi, kullanıcı yetkileri ve siber güvenlik kuralları
- FAT ve SAT ortamı, test zamanı ve kabul yetkilileri

Bu girdiler sağlanmadan ilgili gereksinimin nihai doğrulaması yapılamaz.


## 13. İzlenebilirlik ve değişiklik yönetimi

Her müşteri gereksinimi benzersiz `MGR-*` kimliğiyle takip edilir. Sistem gereksinimleri, tasarım maddeleri ve kabul testleri ilgili müşteri gereksinimi kimliğine referans vermelidir.

Onaydan sonra yapılacak kapsam, iş kuralı, arayüz veya kabul ölçütü değişiklikleri yazılı değişiklik talebiyle yönetilir. Değişiklik talebi en az etkilenen gereksinimleri, maliyet/takvim etkisini, doğrulama ihtiyacını ve onaylayan tarafları içermelidir.

## 14. Varsayımlar ve bağımlılıklar

- Tomruk, kamera görüş alanına mekanik olarak uygun ve ölçüm sırasında yeterince kararlı biçimde sunulur.
- Kamera, PLC, ağ ve bilgisayar donanımı üretim koşullarına uygun ve çalışır durumdadır.
- Kalibrasyonun yapılması ve periyodik doğrulanması için uygun fiziksel referanslar sağlanır.
- PLC arayüzü ve hat sıralaması otomasyon ekibiyle birlikte test edilebilir.
- Ölçüm algoritmasının kabulü için müşteri tarafından doğrulanmış örnekler sağlanır.
- Ortam ışığı, titreşim, kirlenme, sıcaklık ve ağ yükü gibi saha koşulları üzerinde mutabık kalınan çalışma aralığında tutulur.

## Ek A — Revizyon geçmişi


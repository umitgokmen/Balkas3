# Balcas Müşteri Gereksinimleri Dokümanı

| Belge alanı | Değer |
|---|---|
| Belge kodu | BLC-MGD-001 |
| Belge türü | Müşteri Gereksinimleri Dokümanı (MGD / URS) |
| Sürüm | 0.1 |
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
- Güvenilir ölçüm üretilemediğinde ürünü yanlışlıkla kabul etmemek.
- Sonuçları operatöre anlaşılır biçimde göstermek ve PLC'ye güvenli biçimde aktarmak.
- Üretim sorunlarının incelenebilmesi için denetimleri izlenebilir şekilde kaydetmek.
- Kayıtlı verilerle üretim hattından bağımsız test ve tekrar analiz yapabilmek.

## 3. Kapsam

### 3.1 Kapsam dahilinde

- Üç adede kadar 3B kamera kanalından veri alınması
- Kamera verilerinin ortak koordinat sisteminde birleştirilmesi
- Tomruğun tespit edilmesi ve ölçüm verisinin kalite kontrolü
- Uzunluk, çap, yön, flare ve sehim ölçümleri
- Kabul, ret ve ölçüm hatası kararının oluşturulması
- PLC ile tetik, durum, sonuç ve onay bilgi alışverişi
- Operatör ekranları, ayarlar ve teşhis görünümü
- Denetim sonuçlarının, hataların ve seçili ham verilerin kaydedilmesi
- Tekli ve toplu kayıtlı veri simülasyonu

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
| MGR-OPS-003 | Zorunlu | Üretim modunda bütün etkin kameralar ve PLC haberleşmesi hazır olmadan sistem “Hazır” göstermemelidir. | Test |
| MGR-OPS-004 | Zorunlu | Sistem çalışma modunu “Üretim” veya “Simülasyon” olarak açıkça göstermelidir. | Gösterim |
| MGR-OPS-005 | Zorunlu | Kontrollü kapatma ve beklenmeyen yeniden başlatma sonrasında sistem PLC ile güvenli, tanımlı bir başlangıç durumuna dönmelidir. | Test |

### 7.2 Denetim ve ölçüm

| Kimlik | Öncelik | Gereksinim | Doğrulama |
|---|---|---|---|
| MGR-INS-001 | Zorunlu | Her geçerli PLC tetiklemesi yalnızca bir denetim oluşturmalıdır. Uzun süre aktif kalan bir tetik ikinci denetime neden olmamalıdır. | Test |
| MGR-INS-002 | Zorunlu | Her denetim benzersiz bir kimliğe sahip olmalı; ekranda, loglarda, sonuçlarda ve saklanan verilerde aynı kimlik kullanılmalıdır. | İnceleme/Test |
| MGR-INS-003 | Zorunlu | Sistem, yapılandırılmış etkin kameraların 3B verisini aynı denetim kapsamında toplamalıdır. | Test |
| MGR-INS-004 | Zorunlu | Sistem tomruğun uzunluğunu milimetre cinsinden ölçmelidir. | Referans obje testi |
| MGR-INS-005 | Zorunlu | Sistem minimum, medyan ve maksimum gövde çaplarını milimetre cinsinden ölçmelidir. | Referans obje testi |
| MGR-INS-006 | Zorunlu | Sistem tomruğun yönünü “dip önde”, “dip arkada” veya “bilinmiyor” olarak bildirmelidir. | Etiketli veri testi |
| MGR-INS-007 | Zorunlu | Sistem flare bulunup bulunmadığını ayrı bir sonuç olarak bildirmelidir. | Etiketli veri testi |
| MGR-INS-008 | Zorunlu | Sistem tomruk sehimini 360° çevresinde değerlendirerek nihai sehimi, ilgili açıyı ve konumu bildirmelidir. | Referans veri testi |
| MGR-INS-009 | Zorunlu | Flare bölgesi ve güvenilir olmadığı belirlenen ölçüm parçaları normal gövde çapı ve sehim hesabını bozmamalıdır. | Referans veri testi |
| MGR-INS-010 | Zorunlu | Ölçüm için gereken veri kapsamı sağlanmıyorsa sistem sayısal bir ölçüm uydurmamalı ve sonucu “Ölçüm Hatası” olarak vermelidir. | Hata senaryosu testi |
| MGR-INS-011 | Zorunlu | Aynı kayıtlı veri, aynı algoritma sürümü ve aynı ayarlarla işlendiğinde tanımlı tolerans içinde aynı sonucu vermelidir. | Regresyon testi |

### 7.3 Karar kuralları

| Kimlik | Öncelik | Gereksinim | Doğrulama |
|---|---|---|---|
| MGR-DEC-001 | Zorunlu | Sistem üç farklı sonucu ayırmalıdır: “Kabul”, “Ret” ve “Ölçüm Hatası”. | Test |
| MGR-DEC-002 | Zorunlu | İzin verilen sehim, tomruk uzunluğu ile müşteri tarafından onaylanan metre başına sehim sınırından hesaplanmalıdır. | Hesap kontrolü |
| MGR-DEC-003 | Zorunlu | Geçerli ölçümde sehim izin verilen değere eşit veya küçükse sonuç “Kabul” olmalıdır. | Sınır değer testi |
| MGR-DEC-004 | Zorunlu | Geçerli ölçümde sehim izin verilen değerden büyükse sonuç “Ret” olmalıdır. | Sınır değer testi |
| MGR-DEC-005 | Zorunlu | Güvenilir karar üretilemeyen durumda sonuç “Ölçüm Hatası” olmalı ve PLC'ye kabul sonucu gönderilmemelidir. | Hata senaryosu testi |
| MGR-DEC-006 | Zorunlu | Flare veya yön bilgisi, müşteri tarafından ayrıca onaylanmış bir kural bulunmadıkça tek başına ret nedeni olmamalıdır. | Kural incelemesi/Test |
| MGR-DEC-007 | Zorunlu | Ham ve düzeltilmiş sehim ile kullanılan düzeltme katsayısı sonuç kaydında izlenebilir olmalıdır. | Kayıt incelemesi |

### 7.4 PLC entegrasyonu

| Kimlik | Öncelik | Gereksinim | Doğrulama |
|---|---|---|---|
| MGR-PLC-001 | Zorunlu | Sistem; bekleme, çalışma, sonucu tutma ve devre dışı durumlarını PLC ile belirlenmiş sözleşmeye göre yönetmelidir. | Entegrasyon testi |
| MGR-PLC-002 | Zorunlu | Ölçüm tamamlanmadan PLC'ye geçerli nihai sonuç sunulmamalıdır. | Entegrasyon testi |
| MGR-PLC-003 | Zorunlu | Sistem sonuçları PLC onayı gelene kadar değiştirmeden tutmalıdır. | Entegrasyon testi |
| MGR-PLC-004 | Zorunlu | PLC onayı alındıktan sonra sonuçlar temizlenmeli ve sistem yeni denetim için hazırlanmalıdır. | Entegrasyon testi |
| MGR-PLC-005 | Zorunlu | PLC bağlantısı veya heartbeat kaybolduğunda “Hazır” durumu kaldırılmalı, hata gösterilmeli ve kontrollü yeniden bağlantı denenmelidir. | Hata senaryosu testi |
| MGR-PLC-006 | Zorunlu | Mevcut PLC etiketleri/register adresleri ve veri birimleri, iki tarafça onaylanan bir arayüz değişikliği olmadan değiştirilmemelidir. | Arayüz incelemesi |
| MGR-PLC-007 | Önerilen | PLC arayüzü, “Ret” ile “Ölçüm Hatası” durumlarını ayrı bir sonuç türü veya hata koduyla ayırt edebilmelidir. | Entegrasyon testi |

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

## 8. Fonksiyonel olmayan gereksinimler

| Kimlik | Öncelik | Gereksinim | Doğrulama |
|---|---|---|---|
| MGR-NFR-001 | Zorunlu | PLC tetiklemesinden sonucun hazır olmasına kadar geçen süre ölçülmeli ve kaydedilmelidir. | Kayıt incelemesi |
| MGR-NFR-002 | Karar bekliyor | Uçtan uca azami denetim süresi müşteri tarafından hat çevrim süresine göre belirlenecektir. | Performans testi |
| MGR-NFR-003 | Zorunlu | Kullanıcı arayüzü kamera çekimi, 3B işleme ve dosya yazımı sırasında yanıt verebilir kalmalıdır. | Performans testi |
| MGR-NFR-004 | Zorunlu | Beklenmeyen bir hata sonraki denetimlerin sessizce durmasına yol açmamalı; durum ve hata nedeni görünür olmalıdır. | Dayanıklılık testi |
| MGR-NFR-005 | Zorunlu | Kamera, PLC veya veri kalitesi hatası halinde varsayılan davranış güvenli olmalı; sistem yanlış kabul üretmemelidir. | Hata senaryosu testi |
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
| MKA-003 | Etkin kameralardan biri zaman aşımına uğrar veya boş veri verir. | Sonuç “Ölçüm Hatası” olur; PLC'ye kabul gönderilmez ve kamera hatası kaydedilir. |
| MKA-004 | Ölçüm için gerekli veri kapsamı oluşmaz. | Fiziksel ölçüm yerine özel/sihirli sayı kullanılmaz; sonuç “Ölçüm Hatası” olur. |
| MKA-005 | PLC sonucu onaylar. | Sonuç alanları temizlenir ve sistem yeni denetim için “Hazır” durumuna döner. |
| MKA-006 | PLC tetik sinyali yüksek kalır. | Aynı tomruk için ikinci denetim başlamaz. |
| MKA-007 | Sehim, onaylı sınıra tam eşittir. | Sonuç, müşteri kararı değişmediği sürece “Kabul” olur. |
| MKA-008 | Geçersiz bir ayar kaydedilmeye çalışılır. | Kayıt reddedilir ve açıklayıcı alan hatası gösterilir. |
| MKA-009 | On veri setinden biri bozukken toplu simülasyon çalıştırılır. | Bozuk veri hata olarak raporlanır; diğer dokuz veri işlenir ve özet on kayıt içerir. |
| MKA-010 | Çalışan denetim sırasında karar ayarı değiştirilir. | Mevcut denetim eski ayarla, sonraki denetim yeni ayarla tamamlanır. |
| MKA-011 | Bilinen ölçülü referans obje işlenir. | Ekran, kayıt ve PLC değerleri aynı birimde ve onaylı tolerans içindedir. |
| MKA-012 | Uygulama beklenmeyen kapanma sonrasında yeniden açılır. | Eski sonuç yeni sonuç gibi yayınlanmaz; PLC ile güvenli başlangıç durumu kurulur. |

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

- Üretim hattı çevrim süresi ve azami sonuç süresi
- Ölçüm aralıkları, doğruluk ve tekrarlanabilirlik toleransları
- Metre başına izin verilen azami sehim
- Kamera sayısı, yerleşimi, görüş alanı ve kalibrasyon referansları
- PLC protokolü, etiket/register listesi, veri tipleri ve byte sırası
- Kabul/ret iş kuralları ve hata halinde hat davranışı
- Referans tomruklar/veri setleri ile beklenen sonuçlar
- Ham veri ve log saklama süresi, disk kotası, arşivleme ve yedekleme politikası
- Üretim ağ erişimi, kullanıcı yetkileri ve siber güvenlik kuralları
- FAT ve SAT ortamı, test zamanı ve kabul yetkilileri

Bu girdiler sağlanmadan ilgili gereksinimin nihai doğrulaması yapılamaz.

## 12. Müşteri kararı bekleyen konular

| Kimlik | Karar | Önerilen başlangıç yaklaşımı | Karar / tarih |
|---|---|---|---|
| MK-001 | Ölçüm hatası PLC'de ayrı hata koduyla gösterilecek mi? | `Outcome=false` ve ayrı `ErrorCode` |  |
| MK-002 | Birincil sehim yöntemi geçersizse ikincil yöntem karar verebilir mi? | Doğrulanana kadar hayır; “Ölçüm Hatası” |  |
| MK-003 | Sehim sınıra tam eşitse sonuç ne olmalı? | Kabul |  |
| MK-004 | Sehim hesabında tek yönlü mü, iki yönlü mutlak sapma mı kullanılacak? | En büyük iki yönlü mutlak sapma |  |
| MK-005 | Sehim düzeltme katsayısının değeri ve fiziksel dayanağı nedir? | Kalibrasyon testiyle belirlenmeli |  |
| MK-006 | Üretimde güvenilir ölçüm için gereken asgari etkin kamera sayısı nedir? | Saha kapsama testiyle belirlenmeli |  |
| MK-007 | Ölçüm geçerliliği için gereken asgari veri/dilim kapsamı nedir? | Referans veriyle belirlenmeli |  |
| MK-008 | Azami uçtan uca denetim süresi nedir? | Hat çevrim süresinden türetilmeli |  |
| MK-009 | Flare tek başına ret nedeni midir? | Hayır |  |
| MK-010 | Yön belirlenemezse kabul/ret sonucu etkilenir mi? | Proses sahibi belirlemeli |  |
| MK-011 | Log ve ham veriler ne kadar süre saklanacaktır? | IT/operasyon belirlemeli |  |
| MK-012 | Ölçüm doğruluğu ve tekrarlanabilirlik toleransları nedir? | Referans obje ve saha testiyle belirlenmeli |  |
| MK-013 | Üretimde manuel denetim komutuna izin verilecek mi? | Proses güvenlik incelemesiyle belirlenmeli |  |

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

| Sürüm | Tarih | Değişiklik | Hazırlayan |
|---|---|---|---|
| 0.1 | 28.09.2026 | İlk müşteri inceleme taslağı |  |

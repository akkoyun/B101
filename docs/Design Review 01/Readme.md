# B100 / B101BA Modem Kartı Design Review Değerlendirme Raporu

**Doküman:** Telit Cinterion Design Review Report  
**Rapor No:** DR-35567 Rev 1  
**Tarih:** 2026-04-08  
**Modem:** Telit-Cinterion LE910C1-EUX  
**İncelenen Tasarım:** B101BA_00.00.01  
**Hazırlayan:** Mehmet Günce Akkoyun için teknik değerlendirme notu

---

## 1. Yönetici Özeti

Telit tarafından yapılan design review sonucunda kart genel olarak çalışabilir görünse de, özellikle **modem footprint hatası**, **reserved pin bağlantıları**, **seviye çevirici besleme tercihi**, **güç kesintisinde modem güvenli kapatma mekanizması** ve **yüksek manyetik alan ortamında bobin seçimi** konularında revizyon ihtiyacı vardır.

En kritik konu PCB layout tarafındadır: Telit, **modem footprint’inin yanlış olduğunu** belirtmiştir. Bu doğrudan üretilebilirlik, lehimleme güvenilirliği, modem oturumu ve uzun dönem saha güvenilirliği açısından **kritik hata** olarak ele alınmalıdır.

İkinci kritik konu, **K4 ve M6 pinlerinin reserved olması ve bağlanmaması gerektiğidir**. Bu pinlerin devrede herhangi bir nete bağlı olması ileride modem davranışı, sertifikasyon veya yeni modem revizyonları açısından risk oluşturabilir.

Üçüncü önemli konu ise **voltaj çevirici VCCB hattında VIO_1V8 yerine VAUX/PWRMON kullanılması önerisidir**. Telit bunu latch-up riskini azaltmak için önermiştir.

Bu rapora göre B100/B101BA modem kartı için revizyon yapılmadan seri üretime geçilmemelidir.

---

## 2. Telit Review Sonuç Özeti

| İnceleme Alanı | Durum | Yorum |
|---|---:|---|
| Power Supply | Warning + Tip | L3 bobini shielded olmalı, fast shutdown öneriliyor |
| SIM Pins | OK | SIM bağlantılarında problem görülmemiş |
| Digital Pins | Error + Warning | Reserved pin bağlantıları ve seviye çevirici besleme riski var |
| RF Aspects - Schematic | Warning | 33 pF RF/harmonik filtreleme kapasitörleri öneriliyor |
| Audio | N/A | Kullanılmıyor |
| PCB General Placement | Error | Modem footprint yanlış |
| Antenna Waveguide | OK | RF hattı 50 Ohm karakteristik empedansa uygun |
| RF Layout | OK | RF waveguide hesaplaması uygun |
| Audio Layout | N/A | Kullanılmıyor |

---

## 3. Kritik Bulgular

## 3.1 Modem Footprint Hatalı

**Seviye:** Kritik / Error  
**Alan:** PCB Layout  
**Telit yorumu:** Modemin footprint’i yanlış, doğru footprint şekli raporda verilmiş.

### Risk

Bu madde tasarımdaki en ciddi bulgudur. Footprint hatası aşağıdaki problemlere yol açabilir:

- Modemin PCB üzerine mekanik olarak hatalı oturması
- Pad hizalama problemleri
- Lehim köprüsü veya açık devre riski
- Reflow sonrası gizli lehim hataları
- Sahada titreşim/ısı döngüsü sonrası bağlantı kopmaları
- Üretim veriminin düşmesi
- Modem üretici tavsiyesine uyumsuzluk nedeniyle sertifikasyon veya onay sürecinde sorun

### Yapılacak İşlem

- Telit’in verdiği footprint çizimi referans alınarak mevcut footprint yeniden kontrol edilmeli.
- Library footprint’i güncellenmeli.
- Pad ölçüleri, keep-out/inhibit wiring bölgeleri, package outline ve mekanik hizalama tekrar çizilmeli.
- Güncellenmiş footprint ile PCB tekrar route edilmeli.
- Eski footprint kullanılan tüm projeler kontrol edilmeli.

### Öncelik

**P0 - Üretim öncesi mutlaka düzeltilmeli.**

---

## 3.2 K4 ve M6 Reserved Pinleri Bağlı Olmamalı

**Seviye:** Kritik / Error  
**Alan:** Schematic / Digital Pins  
**Telit yorumu:** K4 ve M6 pinleri reserved olup bağlanmamalıdır.

### Risk

Reserved pinlerin bağlanması şu riskleri doğurur:

- Modem iç yapısında bilinmeyen fonksiyonlara müdahale
- Farklı modem donanım revizyonlarında uyumsuzluk
- Sertifikasyon sırasında üretici tavsiyesine aykırı bağlantı
- Uyku/uyanma, boot, RF veya debug davranışlarında öngörülemeyen sorunlar
- Saha koşullarında rastgele modem kilitlenmesi veya kararsızlık

### Yapılacak İşlem

- K4 ve M6 pinleri şemada **NC** bırakılmalı.
- Bu pinlere bağlı net, test point, pull-up, pull-down veya başka bağlantı varsa kaldırılmalı.
- PCB üzerinde bu pinlerden çıkan izler silinmeli.
- ERC/DRC için bu pinler “Do Not Connect” olarak işaretlenmeli.

### Öncelik

**P0 - Mutlaka düzeltilmeli.**

---

## 3.3 Voltage Translator VCCB Hattında VIO_1V8 Yerine VAUX/PWRMON Kullanılmalı

**Seviye:** Warning  
**Alan:** Schematic / Communication Interfaces  
**Telit yorumu:** Module latch-up riskini önlemek için voltage translator’ın VCCB hattında VIO_1V8 yerine VAUX/PWRMON kullanılmalı.

### Teknik Değerlendirme

Bu uyarı önemlidir. Modem ile MCU veya başka dijital devreler arasında seviye çevirici kullanılıyorsa, modem kapalıyken veya power state geçişindeyken IO hatlarından modeme ters besleme gitmesi latch-up riskini artırabilir.

VAUX/PWRMON kullanımı, modem IO domain aktif olduğunda seviye çeviricinin de aktif olmasını sağlar. Böylece modem kapalıyken dış dünyadan IO pinleri üzerinden modem içine kaçak besleme verilmesi engellenir.

### Risk

- Modem latch-up
- Boot sırasında kararsız davranış
- Modem kapanmış görünürken IO üzerinden kısmi beslenme
- Akım tüketiminde anormal artış
- Modem reset ihtiyacı
- Uzun vadede modem hasarı

### Yapılacak İşlem

- Voltage translator VCCB hattı mevcutta VIO_1V8’e bağlıysa bağlantı revize edilmeli.
- VCCB, Telit önerisine uygun şekilde VAUX/PWRMON sinyalinden beslenmeli.
- Translator yön kontrolü, enable pini ve pull-up/pull-down yapısı tekrar kontrol edilmeli.
- Modem kapalıyken MCU tarafının modem IO hattına enerji basmadığı doğrulanmalı.
- Güç sıralaması test edilmeli.

### Test Önerisi

- Modem kapalı, MCU açık senaryosu
- MCU kapalı, modem açık senaryosu
- Her ikisi aynı anda açılıyor senaryosu
- Ani güç kesintisi sonrası tekrar açılma senaryosu
- Sleep/wakeup senaryosu

### Öncelik

**P1 - Revizyon kartında düzeltilmeli.**

---

## 3.4 L3 Bobini Shielded Tip ile Değiştirilmeli

**Seviye:** Warning  
**Alan:** Power Supply  
**Telit yorumu:** Uygulama yüksek manyetik alana yakın çalışacağı için L3 bobini shielded tip ile değiştirilmelidir.

### Teknik Değerlendirme

Kartın motor kontrol panosu, kontaktör, sürücü, soft starter, kompanzasyon ve yüksek akım hatlarına yakın çalışacağı düşünülürse bu uyarı doğru ve önemlidir. Açık tip bobinler harici manyetik alanlardan etkilenebilir, aynı zamanda kendileri de çevreye daha fazla EMI yayabilir.

Shielded indüktör kullanmak özellikle modem besleme hattında daha stabil davranış sağlar.

### Alternatif Shielded İndüktör Önerileri

    Not: L3 için kesin değer mevcut şemadaki regülatör topolojisine, switching frekansına, hedef çıkış akımına ve mevcut endüktans değerine göre netleştirilmelidir. Aşağıdaki tablo, modem besleme hattında kullanılabilecek **shielded power inductor** alternatiflerini kısa liste olarak verir. Seçim yapılırken mevcut L3 değeriyle aynı endüktans değeri, daha yüksek saturation current ve düşük DCR önceliklendirilmelidir.
 
| Üretici | Seri / Parça Ailesi | Örnek Parça | Paket | Avantaj | Not |
|---|---|---|---|---|---|
| Würth Elektronik | WE-PD / WE-LHMI | 7447709xxx / 744373xxxx | SMD shielded | Kolay bulunur, güçlü datasheet, endüstriyel kullanım için güvenilir | Avrupa tedarik zinciri ve prototip için iyi seçenek |
| Coilcraft | XAL / XFL / MSS | XAL40xx / XFL40xx / MSSxxxx | SMD shielded | Düşük DCR, yüksek saturation current, kaliteli manyetik yapı | Performans odaklı seçim için iyi ama maliyet daha yüksek olabilir |
| Bourns | SRN / SRP | SRN6045 / SRP5030 / SRP7030 | SMD semi-shielded / shielded | Yaygın bulunabilir, maliyet/performans dengesi iyi | SRP serisi daha güçlü shielded yapı için tercih edilebilir |
| TDK | SPM / VLS | SPM5030 / VLS6045 | SMD shielded | Kompakt, kaliteli, otomotiv/endüstriyel seçenekleri var | Stok ve fiyat kontrolü gerekli |
| Murata | LQH / DFE | DFE252012 / DFE322512 / LQHxx | SMD shielded | Kompakt tasarım, güvenilir marka | Yüksek akım gerekiyorsa paket boyutu dikkatli seçilmeli |
| Taiyo Yuden | NR / NRS | NR6045 / NRS6045 | SMD shielded | Uygun maliyetli, yaygın Asya tedariki | Seri üretim maliyeti için değerlendirilebilir |
| Sunlord | SWPA / WPN | SWPA6045 / SWPA7030 | SMD shielded | Çin tedarikinde ekonomik alternatif | Uzun dönem kalite ve sıcaklık performansı ayrıca doğrulanmalı |
| Chilisin / Yageo | SCDS / SLF / çeşitli power serileri | Değere göre seçilmeli | SMD shielded | Ekonomik ve bulunabilir alternatifler | Datasheet parametreleri dikkatli karşılaştırılmalı |

#### Seçim Kriterleri

- Mevcut L3 ile **aynı endüktans değeri** korunmalı.
- Saturation current değeri, modem TX burst ve regülatör peak current ihtiyacının üzerinde olmalı.
- RMS current değeri sürekli çalışma akımını güvenli karşılamalı.
- DCR düşük olmalı; aksi halde ısınma ve verim kaybı artar.
- Shielded yapı tercih edilmeli; semi-shielded parça ancak yer/maliyet zorunluluğu varsa düşünülmeli.
- Çalışma sıcaklığı en az **-40 °C / +85 °C**, tercihen **+105 °C** veya üzeri olmalı.
- Aynı footprint içinde ikinci kaynak alternatifi belirlenmeli.
- Prototipte modem TX anında bobin sıcaklığı, çıkış ripple ve voltaj çökmesi ölçülmeli.

#### Öncelikli Kısa Liste

| Öncelik | Öneri | Gerekçe |
|---:|---|---|
| 1 | Würth Elektronik WE-PD / WE-LHMI | Dokümantasyon, erişilebilirlik ve endüstriyel güvenilirlik açısından güçlü seçenek |
| 2 | Coilcraft XAL / XFL | Elektriksel performans ve düşük kayıp açısından güçlü seçenek |
| 3 | Bourns SRP serisi | Maliyet/performans dengesi iyi, yaygın bulunabilir |
| 4 | TDK SPM serisi | Kompakt ve kaliteli alternatif |
| 5 | Taiyo Yuden NR/NRS veya Sunlord SWPA | Seri üretimde maliyet optimizasyonu için değerlendirilebilir |

### Risk

- Bobin saturasyonu
- Regülatör çıkışında ripple artışı
- Modem TX burst sırasında voltaj çökmesi
- EMI/EMC problemleri
- RF performansında düşüş
- Saha koşullarında rastgele modem resetleri

### Yapılacak İşlem

- L3 mevcut parça numarası kontrol edilmeli.
- Aynı endüktans değerinde, uygun saturation current değerine sahip shielded indüktör seçilmeli.
- RMS current, saturation current, DCR ve sıcaklık yükselmesi kontrol edilmeli.
- Modem TX burst sırasında besleme hattı osiloskop ile izlenmeli.
- EMI açısından yakın alan probu ile karşılaştırmalı test yapılmalı.

### Öncelik

**P1 - Saha güvenilirliği için düzeltilmeli.**

---

## 3.5 Fast Shutdown Özelliği Eklenmeli

**Seviye:** Tip / Strong Recommendation  
**Alan:** Power Supply  
**Telit yorumu:** Ani güç kesintisi ihtimali olan sistemlerde modem memory corruption riskini önlemek için fast-shutdown özelliği uygulanmalı.

### Teknik Değerlendirme

Bu cihaz saha koşullarında motor panosunda çalışacağı için ani enerji kesintisi, kontaktör kaynaklı düşümler, pano içi bakım sırasında kapatma veya akü/hat dalgalanması gibi durumlar beklenebilir.

Modemin dosya sistemi, NVM alanı veya internal memory yapısı ani enerji kesintilerinden etkilenebilir. Bu nedenle fast shutdown mekanizması özellikle endüstriyel IoT uygulamalarında ihmal edilmemelidir.

### Risk

- Modem memory corruption
- SIM/network registration problemleri
- Modem boot loop
- FOTA veya konfigürasyon verisi bozulması
- Sahada cihazın bağlantısız kalması

### Yapılacak İşlem

- Telit LE910C1-EUX Hardware User Guide içindeki FAST_SHUTDOWN bölümü incelenmeli.
- Modem ON/OFF, RESET, PWRMON ve fast shutdown ilişkisi netleştirilmeli.
- Ana besleme düşüşü algılanacaksa MCU veya supervisor üzerinden shutdown tetiklenmeli.
- Yeterli hold-up kapasitesi veya batarya/power-path desteği ile modem shutdown süresi garanti edilmeli.
- Firmware tarafında “power fail detected” event’i oluşturulmalı.

### Öncelik

**P1 - Endüstriyel saha ürünü için uygulanması güçlü şekilde önerilir.**

---

## 3.6 RF/Harmonik Filtreleme İçin 33 pF Kapasitör Footprintleri Eklenmeli

**Seviye:** Warning  
**Alan:** RF Aspects / Schematic  
**Telit yorumu:** Harmoniklerden kaçınmak için modem kartında ve yakındaki kartlarda belirli noktalara GND’ye 33 pF kapasitörler konumlandırılması öneriliyor.

### Telit’in Önerdiği Filtreleme Noktaları

- Input/output connector üzerindeki power source ve sinyal hatları
- Power supply output pad’leri
- Komponentlerin power supply input pad’leri
- LED gibi forward conduction çalışan diyotların anot/katot hatları
- Transistor base hatları
- Phototransistor ve opto-isolator giriş/çıkış hatları

### Teknik Değerlendirme

Telit burada doğrudan EMI/RF harmonik bastırma amacıyla küçük değerli COG/NPO kapasitör öneriyor. 33 pF değeri özellikle 0402/0603 paketlerde yaklaşık 1 GHz civarında self-resonance davranışı nedeniyle önerilmiş. 0201 paketlerde ise 10–20 pF değerlerin 1 GHz civarına daha uygun olduğu belirtilmiş.

Burada kritik nokta şu: Bu kapasitörler rastgele her hatta takılmamalı; sinyal bütünlüğü, rise time, UART/SPI/I2C hızları, open-drain yapılar ve sürücü empedansları dikkate alınmalı. Ama PCB üzerinde opsiyonel footprint bırakmak doğru yaklaşımdır.

### Yapılacak İşlem

- Kritik hatlara opsiyonel “DNP” 33 pF COG/NPO kapasitör footprintleri eklenmeli.
- Kapasitörler filtrelenecek component pad’ine çok yakın yerleştirilmeli.
- Aynı path ve aynı layer üzerinde konumlandırılmalı.
- GND dönüş yolu kısa ve düşük empedanslı olmalı.
- High-speed veya timing-critical hatlarda kapasitör değeri testle doğrulanmalı.
- 0402/0603 için 33 pF, 0201 için 10–20 pF opsiyonları değerlendirilmeli.

### Öncelik

**P2 - EMC/sertifikasyon hazırlığı için önerilir.**

---

## 3.7 SIM Hatları Uygun

**Seviye:** OK  
**Alan:** SIM Pins  
**Telit yorumu:** SIM pin bağlantıları uygun.

### Teknik Değerlendirme

Telit SIM tarafında hata belirtmemiştir. Buna rağmen üretim öncesi şu kontroller yapılmalıdır:

- SIM ESD koruması
- SIM hat uzunlukları
- SIM clock hattı yakınındaki agresif sinyaller
- SIM kart soketi mekanik dayanımı
- SIM detect kullanılıyorsa logic seviyesi
- GND referans sürekliliği

### Öncelik

**P3 - Mevcut yapı korunabilir, üretim öncesi rutin kontrol yeterli.**

---

## 3.8 RF Waveguide 50 Ohm Uygun

**Seviye:** OK  
**Alan:** RF Layout  
**Telit yorumu:** RF waveguide required 50 Ohm characteristic impedance değerine uygun.

### Teknik Değerlendirme

RF hattı açısından önemli bir olumlu bulgudur. Telit’in RF waveguide hesaplamasında 50 Ohm empedans uygun görülmüş. Bu nedenle RF hattının genişlik/boşluk/stack-up yapısı korunmalıdır.

### Yapılacak İşlem

- RF hattı revizyon sırasında değiştirilmemeli.
- Footprint düzeltmesi RF hattını etkiliyorsa empedans yeniden hesaplanmalı.
- Üreticiye stack-up net verilmeli.
- RF hattında solder mask, via geçişleri ve ground stitching korunmalı.
- Anten konnektörü çevresindeki GND yapısı bozulmamalı.

### Öncelik

**P3 - Mevcut yapı korunmalı.**

---

# 4. Revizyon İçin Yapılacaklar Listesi

## 4.1 P0 - Kritik / Üretim Öncesi Zorunlu

- [ ] LE910C1-EUX modem footprint’i Telit’in önerdiği footprint’e göre yeniden çizilecek.
- [ ] Mevcut PCB’de modem footprint pad ölçüleri, package outline ve keep-out/inhibit wiring bölgeleri kontrol edilecek.
- [ ] Revize PCB üretime gitmeden önce Telit footprint çizimiyle birebir karşılaştırılacak.

---

## 4.2 P1 - Güvenilirlik İçin Güçlü Revizyon Önerileri

- [ ] Modem kapalıyken MCU tarafından IO hatları üzerinden modeme ters besleme gitmediği doğrulanacak.
- [ ] L3 bobini shielded tip indüktör ile değiştirilecek.
- [ ] Yeni L3 için saturation current, RMS current, DCR ve sıcaklık değerleri kontrol edilecek.
- [ ] Fast shutdown özelliği Telit Hardware User Guide’a göre incelenecek.
- [ ] Ani güç kesintisi senaryosu için modem güvenli kapanma akışı tasarlanacak.
- [ ] Hold-up süresi için bulk capacitor / batarya / power-path yeterliliği hesaplanacak.
- [ ] Power fail algılama ve modem shutdown firmware akışı tasarlanacak.
- [ ] L3 bobini shielded tip indüktör ile değiştirilecek.
- [ ] Yeni L3 için saturation current, RMS current, DCR ve sıcaklık değerleri kontrol edilecek.
- [ ] L3 için Würth, Coilcraft, Bourns, TDK ve ekonomik seri üretim alternatifleri içinden en az iki pin-to-pin uyumlu kaynak belirlenecek.

---

## 4.3 P2 - EMC / Sertifikasyon Hazırlığı

- [ ] Power source giriş/çıkış hatlarına opsiyonel 33 pF COG/NPO kapasitör footprintleri eklenecek.
- [ ] Input/output connector sinyal hatlarına opsiyonel RF filtre kapasitör footprintleri eklenecek.
- [ ] Power supply output pad’lerine yakın 33 pF opsiyonları değerlendirilecek.
- [ ] LED D6 anot/katot hattı için opsiyonel 33 pF filtre footprint’i eklenecek.
- [ ] Transistor base hatlarına opsiyonel 33 pF filtre footprint’i eklenecek.
- [ ] Opto-isolator ve phototransistor hatları varsa opsiyonel filtre footprintleri eklenecek.
- [ ] 0402/0603 için 33 pF, 0201 için 10–20 pF alternatifleri BOM opsiyonuna eklenecek.
- [ ] Bu kapasitörler varsayılan olarak DNP bırakılacak, EMC testine göre takılacak.

---

# 5. Test Planı

## 5.1 Power Testleri

- [ ] Modem TX burst sırasında modem beslemesi osiloskop ile ölçülecek.
- [ ] Ani enerji kesintisi testi yapılacak.
- [ ] Fast shutdown tetiklenme testi yapılacak.
- [ ] Modem açık/kapalı power sequencing testi yapılacak.
- [ ] MCU açık, modem kapalı senaryosunda IO leakage ölçülecek.
- [ ] Modem açık, MCU kapalı senaryosunda IO leakage ölçülecek.

---

## 5.2 Digital Interface Testleri

- [ ] UART/SPI/I2C hangi arayüz kullanılıyorsa logic analyzer ile kontrol edilecek.
- [ ] Voltage translator VCCB değişimi sonrası haberleşme stabilitesi test edilecek.
- [ ] Sleep/wakeup sonrası haberleşme tekrar başlıyor mu kontrol edilecek.
- [ ] Power cycle sonrası modem kararlı boot ediyor mu kontrol edilecek.

---

## 5.3 RF Testleri

- [ ] Anten bağlantısı sonrası RSSI/RSRP/RSRQ/SINR değerleri ölçülecek.
- [ ] RF hattı revizyon sonrası VNA ile mümkünse kontrol edilecek.
- [ ] Yakın alan EMI taraması yapılacak.
- [ ] 33 pF DNP kapasitörlerin EMC testinde etkisi karşılaştırılacak.
- [ ] Modem TX sırasında yakındaki dijital hatlarda parazit gözlemlenecek.

---

## 5.4 Üretim Testleri

- [ ] Modem footprint revizyonundan sonra ilk PCB’de X-ray veya mikroskop kontrolü yapılacak.
- [ ] Reflow sonrası modem pad lehim kalitesi kontrol edilecek.
- [ ] Reserved pinlerin gerçekten bağlantısız olduğu netlist üzerinden doğrulanacak.
- [ ] RF hattı continuity ve short kontrolü yapılacak.
- [ ] SIM kart algılama ve network registration testi yapılacak.

---

# 6. Revizyon Notları

## 6.1 Schematic Revizyon Notları

| Madde | Aksiyon |
|---|---|
| K4 / M6 | NC yapılacak |
| VCCB | VIO_1V8 yerine VAUX/PWRMON |
| Fast shutdown | Telit HUG’a göre eklenecek |
| L3 | Shielded indüktör ile değiştirilecek; Würth / Coilcraft / Bourns / TDK ve seri üretim alternatifi karşılaştırılacak |
| 33 pF filtreler | Kritik hatlara DNP footprint olarak eklenecek |

---

## 6.2 PCB Revizyon Notları

| Madde | Aksiyon |
|---|---|
| Modem footprint | Telit önerisine göre düzeltilecek |
| RF line | 50 Ohm yapı korunacak |
| GND stitching | RF hattında korunacak |
| 33 pF filtreler | Pad’e çok yakın yerleştirilecek |
| Keep-out alanları | Telit footprint’e göre yeniden kontrol edilecek |

---

# 7. Mühendislik Yorumu

Telit’in raporu kısa ama kritik maddeler içeriyor. En önemli nokta, review sonucunda RF hattının 50 Ohm olarak uygun bulunması ve SIM tarafında problem görülmemesidir. Bu iyi haber.

Ancak modem footprint hatası ciddi bir konu. Bu hata düzeltilmeden PCB üretimine devam etmek doğru olmaz. Eğer mevcut karttan prototip üretildiyse, bu kartlar sadece sınırlı fonksiyon testi için kullanılmalı; saha pilotu veya seri üretim adayı olarak değerlendirilmemelidir.

Reserved pinlerin bağlı olması da hafife alınmamalı. Bugün çalışıyor gibi görünse bile modem üreticisi bu pinleri ileride farklı amaçla kullanabilir veya pin iç yapısı revizyonla değişebilir. Endüstriyel sahada çalışacak bir modem kartında bu tür belirsizlik kabul edilmemelidir.

Voltage translator beslemesiyle ilgili öneri de sahada rastgele modem kilitlenmelerini önlemek açısından önemlidir. Bu tarz latch-up problemleri laboratuvarda kolay yakalanmaz; asıl problem sahada, özellikle motor panosu gibi gürültülü ortamlarda ortaya çıkar.

Sonuç olarak bu design review, kartın tamamen kötü olduğunu göstermiyor; fakat seri üretim öncesi mutlaka yapılması gereken birkaç net düzeltme olduğunu gösteriyor.

---

# 8. Kapanış Kararı

Bu kart için önerilen karar:

**Revizyon gerekli. Mevcut tasarım doğrudan seri üretime çıkarılmamalı.**

Revizyon sonrası aşağıdaki üç kontrol tamamlanmadan üretim onayı verilmemelidir:

1. Telit footprint uyumluluk kontrolü
2. Reserved pinlerin tamamen NC doğrulaması
3. Power sequencing / latch-up / fast shutdown testleri

Bu üç madde kapatıldıktan sonra kart, saha pilotu için yeniden değerlendirilebilir.
# B100 / B101BA Modem Kartı Design Review Revizyon Raporu

**Kaynak Doküman:** Telit Cinterion Design Review Report  
**Rapor No:** DR-35567 Rev 1  
**Tarih:** 2026-04-08  
**Modem:** Telit-Cinterion LE910C1-EUX  
**Tasarım:** B101BA_00.00.01  
**Kapsam:** Telit design review bulguları, yapılan düzeltmeler ve açık kalan doğrulama adımları

---

## 1. Yönetici Özeti

Telit tarafından yapılan design review sonucunda B100 / B101BA modem kartı için birkaç kritik düzeltme önerilmiştir. Review’de öne çıkan başlıklar şunlardır:

- Modem footprint’inin Telit önerisiyle uyumlu hale getirilmesi
- K4 ve M6 reserved pinlerinin bağlantısız bırakılması
- Voltage translator beslemesinde VIO_1V8 yerine PWRMON/VAUX kullanılması
- L3 indüktörün shielded/kalkanlı olduğunun netleştirilmesi
- Ani güç kesintisi senaryosu için fast shutdown mekanizmasının değerlendirilmesi
- RF/harmonik bastırma için opsiyonel 33 pF filtre kapasitörlerinin değerlendirilmesi

İlk revizyon çalışması kapsamında kritik donanım maddeleri kapatılmıştır. Kart artık doğrudan “hatalı tasarım” durumundan çıkmış, **revizyon sonrası doğrulama/test aşamasına** geçmiştir.

Seri üretim kararı verilmeden önce güç sıralaması, IO leakage, modem TX burst besleme stabilitesi ve temel RF/EMC davranışı doğrulanmalıdır.

---

## 2. Telit Bulguları ve Yapılan İşlemler

| No | Telit Bulgusu | Öncelik | Yapılan İşlem | Durum |
|---:|---|---|---|---|
| 1 | Modem footprint hatalı | P0 | LE910C1-EUX footprint’i Telit önerisine göre düzeltildi | Tamamlandı |
| 2 | K4 ve M6 pinleri reserved, bağlanmamalı | P0 | K4 ve M6 pinleri NC hale getirildi | Tamamlandı |
| 3 | Voltage translator VCCB hattında VIO_1V8 yerine PWRMON/VAUX kullanılmalı | P1 | 1V8 besleme kaldırılarak PWRMON/VAUX besleme yapıldı | Tamamlandı |
| 4 | L3 yüksek manyetik alan için shielded olmalı | P1 | L3’ün shielded/kalkanlı olduğu şematikte netleştirildi | Tamamlandı |
| 5 | Ani güç kesintisi için fast shutdown öneriliyor | P1 | Donanım/firmware akışı ayrıca değerlendirilecek | Açık |
| 6 | RF/harmonik azaltma için 33 pF filtreler öneriliyor | P2 | Opsiyonel DNP filtre footprintleri değerlendirilecek | Açık / Opsiyonel |
| 7 | SIM pinleri | P3 | Telit tarafından uygun bulundu | OK |
| 8 | RF waveguide 50 Ohm | P3 | Telit tarafından uygun bulundu; yapı korunacak | OK |

---

## 3. Kapatılan Kritik Maddeler

### 3.1 Modem Footprint Düzeltmesi

Telit, PCB layout incelemesinde modem footprint’inin doğru olmadığını belirtmiştir. Bu madde üretilebilirlik, lehim kalitesi ve saha güvenilirliği açısından en kritik bulgudur.

**Yapılan işlem:**

- LE910C1-EUX modem footprint’i Telit önerisine göre güncellendi.
- Pad geometrileri ve mekanik yerleşim kontrol edildi.
- Yeni footprint’e göre PCB yerleşimi revize edildi.

**Durum:** Tamamlandı.

---

### 3.2 Reserved Pin Düzeltmesi

Telit, K4 ve M6 pinlerinin reserved olduğunu ve bağlanmaması gerektiğini belirtmiştir.

**Yapılan işlem:**

- K4 pini NC hale getirildi.
- M6 pini NC hale getirildi.
- Reserved pinlerin sinyal, test point, pull-up veya pull-down bağlantısı olmaması sağlandı.

**Durum:** Tamamlandı.

---

### 3.3 Voltage Translator Besleme Revizyonu

Telit, latch-up riskini azaltmak için voltage translator VCCB hattının VIO_1V8 yerine PWRMON/VAUX ile beslenmesini önermiştir.

**Yapılan işlem:**

- VCCB hattındaki 1V8 besleme kaldırıldı.
- Voltage translator beslemesi PWRMON/VAUX yapısına alındı.

**Beklenen fayda:**

- Modem kapalıyken IO hatlarından ters besleme riski azalır.
- Modem power state ile dijital arayüz seviyesi daha uyumlu hale gelir.
- Latch-up riski düşer.

**Durum:** Tamamlandı.

---

### 3.4 L3 İndüktör / Shielded Bilgisi

Telit, uygulamanın yüksek manyetik alana yakın çalışacağı gerekçesiyle L3 indüktörün shielded tip olmasını önermiştir.

Mevcut L3 parçası:

```text
Murata LQH3NPN2R2MMEL
2.2 µH
Shielded / Magnetic Resin Shielded
```

**Yapılan işlem:**

- L3’ün shielded/kalkanlı olduğu şematik üzerinde açık şekilde belirtildi.
- BOM açıklamasında shielded power inductor bilgisi netleştirildi.

**Mühendislik notu:**

Mevcut parça shielded olsa da modem besleme hattı için daha yüksek akım kapasiteli ve daha düşük DCR değerine sahip alternatifler ileride değerlendirilebilir. Bu değişiklik zorunlu değil; daha çok saha güvenilirliği ve EMI dayanımı için iyileştirme adayıdır.

Önerilen alternatif kısa liste:

| Öncelik | Parça | Değer | Not |
|---:|---|---:|---|
| 1 | Bourns SRP5030TA-2R2M | 2.2 µH | Fiyat/kalite dengesi iyi, daha yüksek akım kapasitesi |
| 2 | Würth 74437346022 | 2.2 µH | Daha güçlü ve düşük DCR’li seçenek |
| 3 | Coilcraft XEL4030-222MEC | 2.2 µH | Performans odaklı alternatif |

**Durum:** Şematik açıklama düzeltmesi tamamlandı. Alternatif komponent geçişi opsiyonel iyileştirme olarak bırakıldı.

---

## 4. Açık Kalan Maddeler

### 4.1 Fast Shutdown Değerlendirmesi

Telit, ani güç kesintisi olan sistemlerde modem memory corruption riskini azaltmak için fast shutdown özelliğinin değerlendirilmesini önermiştir.

Bu konu henüz kapatılmamıştır.

**Yapılacaklar:**

- Telit LE910C1-EUX Hardware User Guide içindeki FAST_SHUTDOWN bölümü incelenecek.
- Ani güç kesintisinde modem güvenli kapanma akışı belirlenecek.
- Gerekirse MCU/supervisor üzerinden power fail algılama ve shutdown tetikleme yapısı tasarlanacak.
- Hold-up kapasitesi veya batarya/power-path süresi yeterlilik açısından değerlendirilecek.

---

### 4.2 Opsiyonel 33 pF RF/Harmonik Filtreleri

Telit, harmonik bastırma için bazı güç ve sinyal hatlarında GND’ye 33 pF kapasitör footprintleri bırakılmasını önermiştir.

Bu konu EMC hazırlığı için faydalıdır; ancak tüm hatlara kontrolsüz uygulanmamalıdır.

**Yapılacaklar:**

- Kritik güç ve IO hatları belirlenecek.
- Gerekli görülen noktalara DNP 33 pF COG/NPO footprint bırakılacak.
- EMC test sonucuna göre bu kapasitörler takılacak veya boş bırakılacak.

---

## 5. Güncel Karar

Revizyon sonrası Telit design review’de belirtilen ana kritik maddeler kapatılmıştır.

**Kapatılan başlıca maddeler:**

1. Modem footprint düzeltildi.
2. K4 ve M6 reserved pinleri NC hale getirildi.
3. Voltage translator beslemesi PWRMON/VAUX yapısına alındı.
4. L3’ün shielded/kalkanlı olduğu şematik üzerinde netleştirildi.

Bu aşamada kart, **revize prototip üretim ve doğrulama testleri** için uygundur.

Seri üretim kararı ise ancak aşağıdaki testlerden sonra verilmelidir:

- Power sequencing testi
- IO leakage testi
- Modem TX burst besleme testi
- SIM/network registration testi
- Temel RF/EMI gözlemi

---

## 6. Sonuç

Telit review sonucunda ortaya çıkan kritik tasarım riskleri büyük ölçüde giderilmiştir. Mevcut durumda ana risk artık şematik/PCB hatasından çok, revizyon sonrası doğrulama testlerinin tamamlanmasıdır.

Bu nedenle önerilen karar:

```text
Revize tasarım prototip üretime alınabilir.
Seri üretim kararı, doğrulama testleri tamamlandıktan sonra verilmelidir.
```
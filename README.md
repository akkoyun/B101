# B101BA-PCIe

B101BA-PCIe, Altium Designer ile geliştirilmiş bir PCIe donanım kartı tasarım projesidir. Bu repo; şematikler, PCB yerleşimi, üretim çıktıları ve proje dosyalarını tek yerde toplamak için yayınlanmaktadır.

## Repo Amacı

Bu depo, kartın tasarım dosyalarını şeffaf biçimde paylaşmak ve proje yapısını açık şekilde korumak için tutulur. Yazılım ağırlıklı bir repo değil; doğrudan **donanım tasarım deposu** olarak düşünülmelidir.

## İçerik

Repo içinde başlıca şu dosya ve klasörler yer alır:

- `B101BA.PrjPcb` — Altium proje dosyası
- `B101BA.PcbDoc` — PCB yerleşim dosyası
- `Schematics/` — şematik sayfaları
- `B101BA.BomDoc` — BOM dokümanı
- `Job File.OutJob` — üretim / çıktı alma ayarları
- `Output/` — üretim veya dışa aktarılmış çıktı dosyaları
- `History/` — Altium geçmiş / revizyon izleri
- `B101BA.PCBDwf` — PCB dokümantasyon çıktısı

## Gereksinimler

Projeyi tam olarak açmak ve düzenlemek için aşağıdaki araçlar önerilir:

- **Altium Designer**
- PCB üretim ve inceleme için uygun CAM / Gerber görüntüleyici araçlar

Sadece repo yapısını incelemek için Altium zorunlu değildir; ancak tasarımı düzenlemek için gerekir.

## Kullanım Senaryoları

Bu repo şu amaçlarla kullanılabilir:

- kartın şematik ve PCB yapısını incelemek
- tasarım yaklaşımını görmek
- üretim çıktıları üzerinden üretim hazırlığı yapmak
- projeyi çatallayıp kendi varyantını geliştirmek

## Notlar

- Bu repo donanım tasarım dosyaları içerdiği için, içeriklerin büyük bölümü Altium ekosistemine aittir.
- `Output/` klasöründeki dosyaların kapsamı kullanılan üretim akışına göre değişebilir.
- Repo adında `B101`, dosya yapısında ise `B101BA` geçmektedir; dokümantasyon ve proje adlandırması bu isim etrafında standardize edilebilir.

## Açık Kaynak Sayfası

Projeye ait genel tanıtım sayfası:

<https://www.akkoyun.net/acik-kaynak/B101BA-PCIe>

## Lisans

Bu repo için lisans bilgisi henüz açıkça tanımlanmamış görünüyor. Açık kaynak kullanım sınırlarını netleştirmek için bir `LICENSE` dosyası eklenmesi önerilir.

# Web ve Ürünler — Yayına Hazırlık

**Oluşturulma:** 25 Ekim 2026  
**Kapsam:** Kyrox web sitesi, Fair-CRM ve Fair-Stand.  
**Amaç:** Kullanıcıyla kararlaştırılan işleri basitten zora, tek tek tamamlamak.

## Nasıl ilerleyeceğiz?

- Her seferinde bir madde ele alınır; sonuç kontrol edilmeden tamamlandı işareti konmaz.
- Liste bir iş planıdır; maddelerin mevcut sistemde kesinlikle eksik olduğu anlamına gelmez. Önce güncel kod ve gerektiğinde canlı ortam doğrulanır.
- Tamamlanan maddenin altına tarih, yapılan değişiklik ve kısa doğrulama kanıtı eklenir. Uygulanmayan madde gerekçesiyle belirtilir.
- Bu dosya çapraz ürün takip listesidir. Ürünlerin teknik yol haritaları ve ortak standartlar kendi kaynaklarında kalır; burada yeniden kopyalanmaz.
- Başlangıç incelemesi yerel kod ve belgelere dayanır. Canlı form teslimi, güncel mobil kullanım ve performans henüz bu çalışma kapsamında doğrulanmadı.

**Güncel durum:** Web paketi ve gerçek Elikon sahneli V3, kullanıcının açık yayın talebiyle 25 Ekim 2026'da yayımlandı. 08–10 tamamlandı. CRM/Stand uygulama depolarındaki önceki yerel değişiklikler henüz gönderilmedi veya dağıtılmadı; kalan kutular bu yüzden açık. Sonraki ana başlık 11; uygulama değişikliklerinin toplu kabulü ayrıca bekliyor. Madde başına ayrı dal/commit/push açılmaz.

## Basitten zora iş listesi

- [ ] **01 — Fair-Stand faviconu:** Bağımsız giriş ve CRM içinde açılan sürümün simgesini kontrol et; eksik tanımı marka varlıklarıyla tamamla.
- [ ] **02 — Sekme başlıkları:** CRM ve Stand başlıklarını ürün adı ve marka ile tutarlı hale getir.
- [ ] **03 — Web ürün bağlantıları:** CRM ve Stand kartlarının doğru hedefe ulaştığını kontrol et.
- [ ] **04 — CRM bulunamayan sayfa ekranı:** Bilinmeyen adreslerin sessizce dashboard'a düşmesi yerine anlaşılır hata ve dönüş bağlantısı sun.
- [ ] **05 — Görsel alt metinleri:** Anlam taşıyan görselleri açıklayıcı metin, dekoratif görselleri uygun boş alt metin açısından kontrol et.
  - **Canlı WordPress görselleri — sorumlu: Codex.** 25 Ekim 2026'da canlı medya API'si kontrol edildi: Ortam Kütüphanesi 0 kayıt döndü; düzenlenecek yüklenmiş görsel yok. Tema görselleri ve V3 proje görselleri kaynakta alt metin taşır. Yeni medya eklenirse içerik incelemesi Codex'in işidir; kullanıcıya devredilmez. CRM marka alt metinleri uygulama deposunda hâlâ yereldedir.
- [ ] **06 — Kırık bağlantılar:** Web ve uygulamalarda iç bağlantıları, dosyaları ve dış hedefleri tara; bulunan sorunları düzelt.
- [ ] **07 — İndeksleme politikası:** Herkese açık tanıtım sayfalarıyla giriş gerektiren ekranları ayır; uygun robots/noindex davranışını doğrula. İndeksleme ayarlarını erişim güvenliği yerine kullanma.
- [x] **08 — Sık sorulan sorular:** Altı soru-cevap, panel yönetimi, aynı içerikten FAQPage ve llms.txt; ana sayfada yayımlandı.
- [x] **09 — Fair-CRM tanıtım sayfası:** Mevcut özellikler, kullanım akışı ve giriş/iletişim bağlantılarıyla `/fuar-crm/` yayımlandı. Gerçek olmayan ekran görüntüsü eklenmedi.
- [x] **10 — Fair-Stand tanıtım sayfası:** Mevcut V2 tasarımı korunarak gerçek Elikon sahneli V3, `/fuar-stand/` adresinde yayımlandı.
- [ ] **11 — Tanıtım sayfalarının SEO ve paylaşımı:** Başlık, açıklama, canonical, paylaşım görseli ve site haritası kapsamını doğrula.
- [ ] **12 — Çerez ve dış servis envanteri:** Oturum, depolama, font, analiz ve diğer dış bağlantıların fiilî kullanımını çıkar.
- [ ] **13 — Gizlilik ve kullanım metinleri:** Envantere ve gerçek veri akışlarına göre gizlilik, veri kullanımı ve kullanım koşulları içeriklerini tamamla; ilgili ekranlardan erişilebilir yap.
- [ ] **14 — Çerez tercihleri:** Envanter sonucuna göre gereken tercih yönetimini uygula; gerekli olmayan servislerin tercih öncesi davranışını doğrula.
- [ ] **15 — Ölçümleme:** Amaç ve veri kapsamını belirleyerek ziyaret ve ürün kullanım ölçümünü kur; gerçek olaylarla doğrula.
- [ ] **16 — Web formunun canlı doğrulaması:** Talebin kaydı, yönetim panelinde görünmesi, e-posta teslimi ve hata durumlarını doğrula.
- [ ] **17 — Mobil kullanım:** Web, CRM ve Stand'ın güncel sürümünü dar ekran ve dokunmatik kullanımda kontrol et; taşma ve erişilemeyen işlemleri düzelt.
- [ ] **18 — Erişilebilirlik:** Klavye, görünür odak, form etiketleri, hata bildirimleri, kontrast ve ekran okuyucu akışlarını kontrol et.
- [ ] **19 — Performans:** Web, CRM ve 3D Stand için ölçüm al; belirlenen darboğazları iyileştir ve aynı koşullarda tekrar ölç.
- [ ] **20 — Kritik akışlar:** CRM kayıt işlemleri ve Stand oluşturma/kaydetme/yeniden açma/malzeme listesi akışlarını başarı ve hata senaryolarıyla uçtan uca doğrula.
- [ ] **21 — Stand teknik kapanışları:** Proje sürümleri, ZIP/GLB hata yolları, malzeme listesi ve yayın kontrollerini mevcut kanonik bulgu defterinden takip ederek kapat. Ayrı bir bulgu listesi üretme.

## İlgili kaynaklar

- [Web deposu ve kurulum bilgileri](https://github.com/kyrox-studio/kyrox-web/blob/main/README.md)
- [Fair-CRM yol haritası](../projects/fair-crm/ROADMAP.md)
- [Fair-Stand açık iş haritası](../projects/fair-stand/OPEN_WORK_INVENTORY.md)
- [Fair-Stand kanonik bulgu defteri](https://github.com/kyrox-studio/kyrox-fair-stand/blob/main/audit/FINDINGS.md)
- [Ortak güvenlik programı](../standards/security/SECURITY_PROGRAM.md)
- [Ortak arayüz ve erişilebilirlik standardı](../standards/ui/UI_FOUNDATION_STANDARD.md)

## Tamamlanan işler

**25 Ekim 2026 — Web yayını:** [Web PR #10](https://github.com/kyrox-studio/kyrox-web/pull/10) üç CI kontrolü yeşilken ana dala birleştirildi. Sunucu tam olarak `42a6c816662ebe370ee3c536f4a52489c7a733d3` commit'ine ilerletildi. Önce WordPress DB ve eski tema yedeği `/opt/kyrox/backups/web-v3-20261025-01a172db` altında alındı. Mevcut sayfalar korunarak yalnız eksik Fuar CRM / Stand Konfiguratör sayfaları oluşturuldu; diğer uygulama servisleri dağıtılmadı.

- Canlı adresler: [Stand V3](https://kyrox.studio/fuar-stand/), [Fuar CRM tanıtımı](https://kyrox.studio/fuar-crm/), [ana sayfa SSS](https://kyrox.studio/#sss).
- Elikon: kayıtlı 9 × 5 m projenin 29 öğesi gerçek ürün renderer'ıyla kuruldu; 29/29 doğrulandı. Yalnız görsel sahne dışa aktarıldı. Ham proje/katalog, müşteri/organizasyon kimliği, oturum ve fiyat verisi yayımlanmadı. Görüntüleyicide düzenleme veya kayıt API'si yoktur.
- Geometri sadeleştirilmeden Meshopt/WebP ile 16,55 MB → 1,65 MB; Three.js aynı ürün sürümünden kullanılır. Model kaynak atıfları sayfadan erişilebilir.
- Chromium yerel + canlı: ilk yüklemede 3D isteği yok; gerçek dönüş, zoom, klavye, dokunmatik dönüş, mobilde taşma olmaması, Elikon/GVN karşılaştırması doğrulandı. Yerelde ağ hatası → kapak + yeniden deneme → hazır sahne kontrolü geçti. Canlı sayfa/JS/model hatası yok.
- PHP 8.4 ile 22 dosya, 6 statik sayfanın bağlantı/JSON-LD/site haritası kontrolleri ve üç GitHub CI kontrolü başarılı. İki ürün adresi canlı WordPress sitemap ve llms.txt içinde; Stand canonical, OG görseli ve indekslenebilirlik doğrulandı.
- Orijinal Desktop/projeler klasöründeki V1/V2 korunarak `v3.html` + `assets/v3/` taşınabilir kopyası eklendi. Yayın kaynaklarının sahibi `kyrox-web` reposudur.
- Reklam kampanyası veya piksel/dönüşüm ölçümü kurulmadı; ölçümleme 15. maddede açık kalır.

## İlk yerel uygulama kaydı — 25 Ekim 2026

Aşağıdaki maddeler uygulama sırasındaki tarihî ara durumlardır; güncel yayın durumu yukarıdadır.

- **01:** Marka deposundaki SVG/ICO faviconları Stand'a aynen eklendi; dosya adları CRM faviconunu ezmeyecek şekilde seçildi. İlk birim testleri ve derleme geçti; 245 senaryoluk tam tarayıcı koşusu kullanıcının toplu kontrol isteğiyle durduruldu. Tam koşu geçti diye raporlanmaz.
- **02:** Başlık kararı: `Kyrox Studio | Fuar CRM` ve `Kyrox Studio | Stand Konfiguratör`. CRM içindeki Stand editörü de ürün başlığını kullanır. CRM alt sayfalarının mevcut sayfa adı eki korunur.
- **03:** Stand kartı `/stand-projects` hedefine bağlandı; aynı hedefe girişten sonra dönüş için sabit izinli hedef eklendi. WordPress eski başlangıç Stand kartının adresi veri değiştirmeden çözülür; özel adresler korunur.
- **04:** CRM'de genel tanınmayan adres için bulunamayan sayfa ekranı ve ana sayfaya dönüş eklendi. Eski bölüm içi yönlendirmeler ve canlı sunucu HTTP durumları ayrıca kontrol edilecek.
- **05:** CRM marka görsellerinin alt metinleri tamamlandı; web iş kartlarının Ortam Kütüphanesi alt metnini zorla boşaltması kaldırıldı. Canlı WordPress medya incelemesi Codex'in açık işidir.
- **06 — İlk bağlantı taraması:** Yerel web derlendi; 4 sayfalık bağlantı/JSON-LD/site haritası kontrolü geçti. Kontrolcü göreli bağlantıları, aynı origin mutlak adresleri ve bölüm kimliklerini de denetleyecek şekilde genişletildi; olmayan bir bölüm bağlantısının reddedildiği negatif kontrolle doğrulandı.
  - Canlı tarama: `kyrox.studio` ana sayfa, blog listesi/yazısı, bağlı CSS/JS/font/görseller, CRM giriş HTML'i ve onun statik dosyaları, Standlar giriş HTML'i, Instagram, LinkedIn, harita ve WhatsApp dahil 34 benzersiz hedef istendi. 32 hedef HTTP 200 verdi. Google Fonts'a ait iki kök adres `preconnect` bağlantısıdır; kök GET yanıtları 404 olsa da sayfa/dosya hedefi olmadıkları için kırık bağlantı değildir. Gerçek Google Fonts CSS isteği 200 verdi. Kontrol edilen bölüm bağlantılarında eksik kimlik bulunmadı.
  - Ayrıca CRM'nin üç marka PNG'si ile `/login`, `/signup`, `/forgot-password`, `/settings/security` giriş HTML'leri HTTP 200 döndü. SPA giriş HTML'inin 200 dönmesi uygulama ekranının çalıştığını kanıtlamaz; oturumlu gezinme ve dinamik Stand katalog/model dosyalarının tamamı henüz doğrulanmadı.
  - Canlı Stand kartı hâlâ CRM kök adresini kullanıyor; 03'te hazırlanan yerel düzeltme henüz yayımlanmadı. Canlıda hiçbir içerik veya sunucu ayarı değiştirilmedi.
- **07:** Canlı web ana sayfasında noindex yok; robots.txt düz metin ve WordPress site haritası XML olarak HTTP 200 dönüyor. Canlı CRM'de robots meta/X-Robots-Tag yok ve `/robots.txt` SPA HTML'i dönüyor. Yerelde CRM ve bağımsız Stand giriş HTML'lerine `noindex, nofollow`, her iki ürüne aynı içerikte gerçek `robots.txt` eklendi. Taramaya izin verilmesi botların noindex işaretini okuyabilmesi içindir; erişim kontrolü değildir. Web'de özel bot gruplarına da `/wp-admin/` kısıtı eklendi. Yerel politika değişiklikleri henüz canlıya uygulanmadı; toplu doğrulama bekliyor.
- **08:** Ana sayfaya altı soru-cevap içeren yerel SSS bölümü eklendi: CRM hedef kitlesi ve takip kapsamı, Stand tasarımı ve kayıt, uygulamalara erişim, iletişim. Klavyeyle kullanılabilen doğal `details/summary` yapısı, mobil stiller, WordPress Özelleştir alanları, aynı görünür cevaplardan `FAQPage` ve llms.txt üretimi hazırlandı. Başlangıç metinlerini tema ve statik önizleme aynı JSON dosyasından okur. Fiyat/ücretsiz deneme vaadi yok. Tema varlık sürümü 1.3.0; toplu PHP/derleme/görsel kontrol ve canlı yayın bekliyor.
- **09:** `/fuar-crm/` herkese açık ürün sayfası yerelde hazırlandı. Müşteriler, fuarlar, görüşmeler, görevler, teklifler ve müşteriyle ilişkili stand projeleri anlatılır; kullanım adımları, uygulama girişi ve iletişim CTA'ları vardır. Gerçek olmayan ekran görüntüsü, fiyat veya performans iddiası eklenmedi. WordPress başlık/özet/içerik alanlarını yönetir; yalnız eksik başlangıç sayfası kurulur, mevcut içerik korunur. Ana sayfanın varsayılan CRM kartı yayımlanmış tanıtım sayfasına bağlanır; sayfa henüz yoksa eski uygulama hedefi korunur. Statik önizleme, sitemap ve ürün listesi de güncellendi. PHP/derleme/mobil-görsel kontroller ve canlı kurulum toplu aşamada bekliyor.
- **10 — Mevcut tasarım referansı:** Kullanıcı `C:/Users/hinthorozuTUF/Desktop/projeler/kyrox_fair_stand_v19/` klasörünü verdi. `index.html` ilk sürüm, `v2.html` ayrılmış CSS/JS kullanan ikinci sürüm; Elikon/GVN tasarım-gerçek uygulama karşılaştırmaları ve dört proje görseli mevcut. V2'nin 3D sahnesi kodla üretilmiş temsili standdır; gerçek kayıtlı proje görüntüleyicisi değildir. `V3-DEVAM-PLANI.md` gerçek proje görüntüleyicisi için geçmiş plandır; V3 dosyası bulunmadı ve plan kullanıcıdan bağımsız yeni uygulama talimatı sayılmaz. Kaynak klasöre dokunulmadı; web entegrasyonu henüz yapılmadı. Mevcut CTA'lar CRM köküne gider, entegrasyonda Standlar hedefiyle güncellenecek.

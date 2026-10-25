# Fair Stand — Kullanılabilirlik Yol Haritası

**Kod karşılaştırması:** 2026-10-25, Fair Stand `main` / `ac18bc1a`.
**Hedef:** Daha önce çizim yapmamış bir kullanıcının hızla adapte olması.
**Durum:** Mod/ayarlar, metreyle kurulum, Geçmiş, ortak Özellikler paneli ve tek kart/varyant akışı mevcut. Yapıyı bütün olarak düzenleme tamamlanmış değil.
**Takip:** `[x]` mevcut kod davranışını; `[ ]` açık işi gösterir. Bu güncelleme belge/kod karşılaştırmasıdır; test veya canlı kabul koşusu yapılmadı. Geçmiş teslimat/doğrulama kayıtları: [PROJECT_STATUS.md](PROJECT_STATUS.md). Rakip gözlemleri: [COMPETITOR_ANALYSIS.md](COMPETITOR_ANALYSIS.md).

---

## Teşhis

### Bizde neden zor?
1. **Yardım bir başvuru kitabı.** Uzun başvuru metni; "şimdi şunu yap" diyen ilk adım rehberi yok.
2. **Katalog üretim parçalarıyla dolu.** Dikme, profil, kısa panel, kutu blok satış yapan birinin karşısına çıkıyor.
3. **Teknik dil.** Üretim listesinde `profile_388` gibi kodlar, "strip", "reçete" gibi terimler.
4. **Önemli işler artık tek tıkla açılan Özellikler panelinde.** Eski sağ tık keşfedilebilirlik sorunu için uygulama var; son görsel hiyerarşi ve erişilebilirlik kabulü açık.
5. **Fiyat yok.** Kullanıcı tasarladığı standın ne tuttuğunu görmüyor; maliyet CRM tarafında.
6. **Geri dönüş zor.** Stand kurulduktan sonra duvar yüksekliği değiştirilemiyor (B.16).

### Rakipte neden kolay?
- Her adımda tek soru soran sihirbaz (etkinlik → ölçü → yapı → başlangıç → editör).
- Az, satışa dönük ürün; her kartta fiyat, anlık sepet toplamı.
- Hazır şablonla başlama.
- Logo ve yazı için ayrı bölümler.
- Panele tek tık = seç, ikinci tık = seçenekler (dokunmatikte de çalışır).
- Salon ortamı, sinematik sunum.

### Bizim avantajımız (korunacak)
- Gerçek üretim listesi (BOM) ve birleşim kuralları.
- Geri al / ileri al, otomatik kayıt, revizyonlar.
- Panel bazında tasarım, CRM bağlantısı, canlı paylaşım.
- Rakipte Maxima sisteminde duvara yazı bile yok.

---

## 🔴 ACİL — yeni kullanıcıyı doğrudan engelleyenler

| # | İş | Neden | Büyüklük |
|---|---|---|---|
| 1 | **Hazır stand şablonları**: "Sırt duvar 3×2 + banko + TV", "L stand 4×3 + depo" gibi 5–10 tasarım; sihirbaz sonunda "boş başla / şablondan başla". | Çizim bilmeyen için en hızlı yol hazır standı değiştirmek; rakibin en güçlü yanı. | 2–3 gün (grup şablonu altyapısı var) |
| 2 | **Basit mod**: satış kullanıcısına yapısal kartları (dikme, profil, kısa panel, kutu blok) gizle; yapısal parçalar otomatik gelsin. Ustaya "Gelişmiş" anahtarı. | Teknik parçalar satış kullanıcısına hâlâ sunuluyor; tek kart/varyant düzeni otomatik yapı çözme değildir. | 1 gün |
| 3 | **Teklif / fiyat özeti**: sahnede canlı toplam ("Bu stand ≈ X ₺") ve satır özeti; maliyet CRM'de var. | Satış konuşmasının merkezi. | 2–3 gün |
| 4 | **Kurulu standda duvar yüksekliğini değiştirme** (B.16). | Kullanıcı fikir değiştirir; şu an baştan kurmak gerekiyor. | 1–2 gün |
| 5 | **Performans**: tam yeniden çizim yerine yerinde güncelleme; ekran kartına göre otomatik kalite (PERFORMANCE_ROADMAP P1–P3). | Yavaş his "bozuk" algısı yaratıyor. | 3–5 gün |

## İlk kullanılabilirlik paketinin mevcut durumu

| # | İş | Durum |
|---|---|---|
| 1 | **Uyarıları görünür yap**: sağ altta üst üste dizilen bildirimler (hata kırmızı, başarı yeşil, × ile kapanır, aynı mesaj tekrarlanmaz). Yalnız yerleştirme değil **bütün hatalar**: kaydetme, açma, görsel, revizyon, render, otomatik kayıt, yerleşim, 3B model yüklenemedi, çizilemeyen modül, eksik görsel. Hata için `alert` penceresi kalmadı; "Render hazır" artık görünür. | ✅ Kodda mevcut |
| 2 | **Üretim listesinde teknik kodları gizle**: ekranda "8 × Profil 388 · adet"; kod ipucunda. "Birleşik leaf toplam" → "Toplam parça listesi", "Alt item yok" → "Parça yok". Kopyalanan üretim metni kodu taşır (üretim ekibi için). | ✅ Kodda mevcut |
| 3 | **Rehberin başına "5 Dakikada İlk Standın"**: 5 numaralı adım (kur, ekle, yerleştir, giydir, kaydet/göster); eski uzun metin "Hızlı Başlangıç Ayrıntıları"na indi. | ✅ Kodda mevcut |
| 4 | **Stand özeti başlığı**: sahnenin sol altında "L Stand Sol · 4 × 3 m · duvar 3,5 m". | ✅ Kodda mevcut |
| 5 | **Sağ tık işleri seçim çubuğunda** | ✅ Tek tık / sağ tık / uzun basma ortak Özellikler panelini açıyor; ⋯ kaldırıldı, temel seçim çubuğu korundu. |
| 6 | **İlk kullanım turu** | ✔ Zaten vardı: ilk sahnede 3 ipucu (katalogdan ekle, çift tıkla seç, Kaydet), bir kez gösterilir. |
| 7 | **Katalog kartına açıklama ipucu** | ➜ Orta vadeye taşındı: DB'de açıklama alanı + admin ekranı gerekiyor (kodda Item'a özel metin kuralı yasak). |
| 8 | **Boş proje kartı** | ✔ Gerek yok: yeni projede sihirbaz zaten açılıyor. |

**Pakete eklenen (kullanıcı isteği):**
- **Profil bağlantısı**: "Dik bağlantı yalnız dikme olan noktaya yapılır" kuralı **profiller için kaldırıldı**. Profil başka bir profilin her noktasına dik bağlanır (iki profilin arasına, ortasına; yaklaşınca 10 cm adımla yapışır), uç uca eklenir; birbirinin ortasından geçen iki profil (X) çakışır. Duvar panelleri için kural aynen geçerli. BOM'a profil T aparatı eklenmedi (gerekirse ayrı karar). ✅ Kodda mevcut
- Kontrol: profiller zeminden tavana Z'de ↑/↓ ile taşınabiliyor — **zaten vardı**, değiştirilmedi.

**Önceden yapılanlar (aynı gün):** ölçü soran kartlarda ölçüsüz başlık ("Profil", "Dikme", "Panel Tek Sıra"), kapasite uyarısı kaldırıldı, profil 346,5 eklendi ve sahne profilleri temizlendi (#330), panel yan payı 2 cm (#331).

## 🟡 ORTA VADE

| # | İş | Not |
|---|---|---|
| 1 | Kenar çubuğunu 4 sekmeye indirmek: **Stand / Duvar & Panel / Mobilya / Teklif**. | Şu an çok sayıda katlanır bölüm. |
| 2 | **Seçime göre özellik paneli** | Tek tıkla açılma uygulandı; ikinci tık gerekmiyor. Görsel kabul için [ayrı plan](SECIM_OZELLIK_PANELI_YOL_HARITASI.md). |
| 3 | **Logo ve yazı aracı**: duvara metin (yazı tipi, renk, boyut), yerleşen yazılar listesi. | Rakipte Maxima'da yok — fark yaratır. |
| 4 | **Sunum modu**: salon ortamı, dönen kamera, hazır açılar, müşteriye PDF (görsel + ölçü + teklif). | Onayı hızlandırır. |
| 5 | **Müşteri onay akışı**: canlı paylaşım linkinde "Onayla / Not bırak". | Altyapı var. |
| 6 | **Sahnesiz ürünler**: sarf, hizmet, montaj için "listeye ekle". | Teklif bütünlüğü. |
| 7 | **"Projeyi kontrol et"**: çakışma, açık kenar, eksik dikme, stand dışı modül; tıklayınca sahnede gösterir. | Acemi hatasını sistem yakalar. |
| 8 | **Kısa video / GIF eğitimler** rehber bölümlerine. | |
| 9 | Ertelenen teknik işler: `scene3d.js` / `main.js` bölme, `z_frame` taşıma. | Kullanıcı görmez, yukarıdakileri hızlandırır. |
| 10 | **Katalog kartı açıklaması** (Hızlı listeden taşındı): DB'de Item açıklama alanı + admin ekranı + kart ipucu. | ~1 gün |

---

## Önerilen sıra
1. ~~**Hızlı** listenin tamamı tek paket~~ — yapıldı (2026-10-23).
2. **Acil 1 + 2** (şablonlar + basit mod) birlikte — "çizim bilmeyen kullanıcı" sorununun asıl çözümü.
3. **Acil 3** (fiyat özeti), sonra **Acil 4–5**.
4. **Orta vade** sırası satış ekibinin geri bildirimine göre.

---

## Basit Mod — Uygulamanın Tamamı İçin Çalışma Planı

**Tarih:** 2026-10-23
**Durum:** Kısmen uygulandı; aşağıdaki işaretler mevcut kodla karşılaştırıldı.
**Kapsam:** Yalnız Item veya katalog filtreleme değil; başlangıç, sahne, gezinme, seçim, yapı düzenleme, tasarım, kayıt ve müşteriye sunum akışının tamamı.
**Hedef:** Satış kullanıcısı üretim parçalarını ve teknik kontrolleri öğrenmeden stand tasarlayabilsin. Usta kullanıcı aynı projede Gelişmiş moda geçebilsin.

### 1. Mesafeler, ölçüler ve sahne görünümü

- [x] Taşırken öğeden standın dört kenarına çizilen mesafe çizgileri ve etiketleri basit modda varsayılan gizlensin.
- [x] Seçili öğenin genişlik / derinlik / yükseklik ölçüleri ve zemine / tavana uzaklıkları varsayılan gizlensin.
- [x] Yerleştirme sırasında görünen zemin ızgarasının mevcut davranışı korunsun; basit mod veya ölçü kontrolü ızgarayı gizlemesin.
- [x] İsteyen kullanıcı Gelişmiş moddaki **Ayarlar → Ölçüleri göster** ayarıyla ölçü yardımcılarını açıp kapatabilsin. Bu kontrol zemin ızgarasını etkilemesin (2026-10-23 kullanıcı kararı).
- [x] Standın genel ölçüsü ve mevcut stand özeti görünür kalsın.
- [x] Görsel yardımcıların gizlenmesi kenara yapışmayı, otomatik hizalamayı ve geçersiz yerleşim denetimlerini kapatmasın.

**Mevcut kanıt:** `kyrox-fair-stand/src/scene3d.js` içinde ölçüler ayrı `measure-overlay` katmanında üretiliyor. Zemin ızgarası da ayrı yönetiliyor. Basit / Gelişmiş mod ve bağımsız ölçü görünürlüğü uygulandı; taşıma ızgarası ayrı yönetiliyor.

### 2. Başlangıç ve menü düzeni

- [x] Mevcut yeni proje sihirbazı korunsun; ölçüler kullanıcıya **Genişlik / Derinlik** ve metre cinsinden sunulsun. İç hesapların birimi bu nedenle değiştirilmesin.
- [x] Depo, alınlık ve zemin seçenekleri anlaşılır ifadelerle sunulsun.
- [ ] Sol menü **Stand / Ürün ekle / Tasarım** etrafında sadeleştirilsin. **Ertelendi (2026-10-23): Şu an uygulanmayacak; yapılacaklar listesinde kalacak.**
- [x] Kaydet ve ekranı paylaş işlemleri kolay erişilen üst alanda bulunsun.
- [x] Gruplar, Şablonlarım ve Standartlar yalnız **Gelişmiş** modda erişilebilir olsun. **Ayarlar** bölümü de yalnız Gelişmiş modda görünür.

**Mevcut kanıt:** `src/newProjectWizard.js` üç adımlı sihirbaz içeriyor. `src/configuratorMarkup.js` içinde Proje, Stand Tipi, Modül ekle, Gruplar, Şablonlarım, Seçili yüzey ve Standartlar bölümleri bulunuyor.

### 3. Seçim ve düzenleme

**Mevcut uygulama ve kalan kabul:** [Seçime Göre Özellik Paneli Yol Haritası](SECIM_OZELLIK_PANELI_YOL_HARITASI.md). Tek / çift tık araç çubuğu ve Çoklu panel / Çoklu modül düğmeleri korunacak. Tek tık, sağ tık ve uzun basma ortak Özellikler panelini açıyor; ⋯ kaldırıldı. Görsel tasarımın son kabulü açık. Aşağıdaki eski sadeleştirme önerileri araç çubuğunun kaldırılması olarak yorumlanmayacak.

- [x] Tek tık / dokunma ile seçim ve seçilene uygun görünür işlemler mevcut; temel kullanım çift tık veya sağ tık öğrenmeyi gerektirmesin.
- [x] Taşıma sahnede sürüklemeyle; Döndür / Kopyala / Sil seçim çubuğuyla sunulur.
- [x] Çoklu panel / Çoklu modül düğmeleri kullanıcı kararıyla seçim çubuğunda korunur; grup ve kilit işlemleri ortak Özellikler panelindedir.
- [ ] Panel yüzeyi ile bütün yapının seçimi açıkça ayırt edilsin; seçim sadeleşirken kamera hareketi, sürükleme ve dokunmatik etkileşim birlikte değerlendirilsin.

**Kod:** `src/selectionToolbar.js`, `src/moduleContextMenu.js`, `src/selectionSurfaceSummary.js` ve `src/main.js`. Tek üst mod seçici vardır; bağımsız eski sağ tık menüsü yerine ortak panel kullanılır. Panel/yüzey ve bütün öğe bağlamları ayrı tutulur; son görsel kabul diğer yol haritasında izlenir.

### 4. Renk, görsel ve zemin

- [ ] İlk görünümde renk paleti, görsel yükleme ve **Doldur / Sığdır / Kaldır** yeterli olsun.
- [ ] Renk alanının son sadeleştirmesi: panelde renk kutusu ve HEX mevcut; RGB / CMYK panel CSS'inde gizli. HEX dahil son ayrıntı hiyerarşisi henüz tamamlanmış sayılmasın.
- [x] Basit modda önce standın tamamına zemin kaplaması seçilsin.
- [x] Bölgesel zemin için Alan X/Y, alan genişliği/derinliği ve sahnede zemin alanı çizme işlemleri Gelişmiş moda taşınsın.
- [x] Gelişmişte hazırlanmış bölgesel zeminler basit moda geçince korunsun.

### 5. Yapıyı bütün olarak düzenleme

- [ ] Kullanıcı **duvar / depo / alınlık** gibi tamamlanmış yapılar üzerinden çalışabilsin.
- [ ] Dikme, profil, kısa panel ve kutu blok gibi teknik katalog kartlarının basit modda görünürlüğü değerlendirilsin. Görünürlük kararı fiziksel parça veya BOM kaydını kaldırmasın.
- [ ] Duvar genişliği değiştirildiğinde gerekli panel, dikme ve profilleri sistemin çözmesi için bütün yapı düzenleme akışı tasarlansın.
- [ ] Otomatik üretilecek parçalar mevcut Item, birleşim ve BOM kurallarından türetilsin; yalnız kart gizlemek otomatik yapı kurma tamamlandı sayılmasın.
- [ ] Duvarı uzatma, kısaltma veya silmenin bağlı görsellere, kaplamalara, aksesuarlara ve diğer yapılara etkisi uygulama öncesinde çıkarılsın.
- [ ] Kurulu standın duvar yüksekliğini değiştirme işi Acil 4 / B.16 ile birlikte takip edilsin; mevcut yetenek olarak sunulmasın.

**Mevcut kanıt:** `src/automaticWall.js`, `src/autoDepot.js` ve mevcut alınlık akışı başlangıç altyapısı sağlıyor. Her elle oluşturulan yapının otomatik tamamlanması veya bütün olarak yeniden boyutlandırılması hazır kabul edilmiyor.

### 6. Kayıt, yardım ve müşteriye sunum

- [x] Kayıt aralığı seçimi Gelişmiş → Ayarlar bölümüne taşındı; mevcut kayıt durumu ve kayıt davranışı korundu.
- [x] Revizyonlar gerektiğinde açılan **Geçmiş** bölümünde erişilebilir olsun.
- [x] Paylaşım düğmesi **Ekranı paylaş** olarak adlandırıldı.
- [ ] **Render Al → Görsel oluştur** adlandırması henüz uygulanmadı; düğme Render Al.
- [x] Üretim listesi mevcut son düzende üst banttadır; Ayrıntılar bölümü kaldırıldı. Önceki ayrıntılara taşıma düzeni güncel değildir; BOM hesabı ve erişim korunur.
- [ ] Uzun rehberin yanında yapılan işe uygun kısa ipuçları gösterilsin; tam rehber erişilebilir kalsın.
- [ ] Canlı fiyat / teklif özeti Acil 3 ile ayrı takip edilsin; basit modda hazırmış gibi gösterilmesin.

### 7. Basit / Gelişmiş geçişinin kuralları

- [x] İki mod aynı proje verisi üzerinde çalışsın; ayrı proje biçimi veya ayrı hesaplama modeli oluşmasın.
- [x] Mod değiştirmek parçaları silmesin, sahneyi yeniden kurmasın veya tasarımı değiştirmesin.
- [x] Gelişmişte hazırlanmış projenin yapısı Basit moda geçince korunsun. Mod geçişi yalnız kontrolleri ve yardımcı gösterimleri değiştirir; öğeleri veya bölgesel zeminleri kaldırmaz. Bu madde bilinen bir veri kaybı hatası değil, korunması gereken genel kuraldır.
- [ ] Basit modda desteklenen düzenlemeler açıkça sunulsun; bütün yapı düzenleme ve teknik parçaların otomatik çözülmesi gibi henüz uygulanmamış işler hazır kabul edilmesin.
- [ ] Basit modda gizlenen işlemler klavye kısayolları ve sağ tık menüsü dahil bütün giriş yollarıyla birlikte değerlendirilsin.
- [x] Mod seçimi yetki sistemiyle karıştırılmasın; bir kontrolün gizlenmesi güvenlik yetkisi sayılmasın.
- [x] **Karar (2026-10-23):** Tek Basit / Gelişmiş seçici sol menünün üstünde yer alır. Basit varsayılan moddur. Ölçüler Basitte kapalı, Gelişmişte açık başlar; kullanıcı Gelişmiş → Ayarlar bölümünden değiştirebilir; Ayarlar Basit modda görünmez. Tercihler oturum boyunca mod başına ayrı korunur; sayfa yeniden açılınca varsayılanlara döner.

### Önerilen uygulama sırası

1. **İlk paket — ekran ve etkileşim sadeleştirme:** mesafe / ölçü görünürlüğü (ızgara korunacak), menü düzeni, renk paneli ve görünür seçim işlemleri.
2. **İkinci paket — günlük kullanım:** başlangıç dili, zemin seçenekleri, kayıt / geçmiş, yardım ve müşteriye sunum adlandırmaları.
3. **Üçüncü paket — bütün yapı düzenleme:** duvar, depo ve alınlık üzerinden işlem; teknik parçaların otomatik çözülmesi ve bağlı öğelerin korunması.

**Takip ilkesi:** Her paket uygulamadan önce mevcut davranış ve etkileriyle somutlaştırılacak. Bu listedeki maddeler uygulanıp doğrulanmadan tamamlandı olarak işaretlenmeyecek.


### Kod dayanakları

| Alan | Kaynak |
|---|---|
| Mod varsayılanları, bağımsız ölçü tercihi | `src/editorViewSettings.js`, `src/main.js`, `src/style.css` |
| Ortak açılır/kapanır bölümler, yalnız Gelişmiş ayarları | `src/configuratorMarkup.js`, `src/style.css` |
| Herşeyi Kilitle | `src/main.js`, `src/moduleLock.js`; tüm mevcut öğeleri kilitler, kaldırılınca öğe/grup kilitlerini açar; grup üyeliği kalır. Yeni öğeleri sürekli kilitleme politikası değildir. |
| Metre girişi, 0,5 m adımı; içeride cm | `src/standSizeInput.js`, `src/standSetup.js`, `src/newProjectWizard.js` |
| Tek mavi / ortak turuncu kesik seçim sınırı, taşıma ızgarası | `src/scene3d.js`, `src/selectionBoundsFrame.js` |
| Bölgesel zemin, alan düşümü ve kayıt | `src/floorArea.js`, `src/main.js`, `src/scene3d.js` |
| Bildirimler | `src/toastNotifications.js`, `src/toastNotifications.css`: en fazla iki bildirim, alttan 12 px + güvenli alan |
| Kılavuz | `src/helpGuide.js`: Gelişmiş > Ayarlar altında kilit, ölçü ve otomatik kayıt açıklamaları |

Kaynak adları uygulama deposuna göredir. Grup dış sınır çerçevesi mevcut; **grubun toplam ölçü etiketleri ve grup aç/kapat henüz eklenmedi**.

### Yeni değerlendirmeler — 2026-10-25

Bunlar uygulanmış özellik değildir; önceki analizin ve kullanıcı kararlarının kaydıdır.

- [ ] **Kurulu projenin duvar yüksekliğini değiştirme:** mevcut kurulum yüksekliği seçeneği yeni sahne içindir. Mevcut duvar/depo/alınlık, bağlı aksesuar, renk ve görselleri koruyan yerinde değişiklik akışı eksik. Kaybolacak veya sığmayacak hedeflerin davranışı uygulamadan önce kesinleştirilecek.
- [ ] **Kurulu projenin genişlik/derinliğini değiştirme:** mevcut Sahneyi Oluştur yolu yeni sahne kurar; projeyi yerinde yeniden boyutlandırma sayılmaz. Sınır duvarları, serbest öğeler, depo, alınlık, bağlı görseller ve küçültmede dışarıda kalan öğeler birlikte ele alınacak. Sabit fiziksel mobilyalar kendiliğinden ölçeklenmeyecek.
- [ ] **Bölgesel zemini sahneye bırakarak ekleme ve sürükleyerek taşıma:** kaplama seçilip ana zemine bırakılacak; konumu sahnede değiştirilecek. X/Y girişi yerine sahne etkileşimi hedefleniyor. Mevcut tek dikdörtgen `stand.floorArea`, alan çizme ve kaplama seçimi altyapıdır; mevcut alanı öğe gibi taşıma henüz yok. Çoklu alan veya çokgen bu taleple tamamlanmış sayılmaz.
- **Kesin zemin hesabı:** Eskisi gibi alan paylaşımı korunacak. 100 m² standın 20 m²'si parke ise **20 m² parke + 80 m² halı** hesaplanacak; ayrıca 100 m² ana kaplama yazılmayacak. `resolveSplitFloorBomLines` mevcut m² hesabı zaten bu davranışı sağlar.
- **Ekran görüntüsü:** Web uygulamasında işletim sistemi veya başka cihazla ekran görüntüsü almayı kesin engelleyen özellik yoktur. Render indirme ve ekran paylaşımı mevcut. Filigran / indirme yetkisi yalnız değerlendirilen alternatiflerdir; uygulama kararı verilmedi.

### Kalan işler ve sınırlar

- Hazır tam stand şablonları, canlı fiyat/teklif ve orta vade listesi ayrı işleri olarak kalır; mevcut grup şablonları bunları tamamlamaz.
- Sol menüyü Stand / Ürün ekle / Tasarım çevresinde yeniden kurma kullanıcı kararıyla **ertelendi**.
- Yapısal kartları gizlemek ve bütün yapıyı otomatik çözmek tamamlanmadı.
- Ortak özellik panelinin son görsel düzen / erişilebilirlik kabulü [kendi yol haritasında](SECIM_OZELLIK_PANELI_YOL_HARITASI.md) izlenir.
- Mod değiştirmek standı değiştirmez: ölçüler, öğeler, yerleşimler, grup/kilit, yüzeyler ve bölgesel zemin korunur. Yalnız arayüz ve yardımcı gösterimler değişir; tamamlanmamış alan çizimi iptal edilebilir.
- Bu belge düzeltmesi kaynak kod, proje, katalog veya sunucu verisi değiştirmez. Mevcut test kapsamı: `e2e/editor-mode-settings.spec.mjs`, `e2e/new-project-wizard.spec.mjs`, `e2e/floor-area.spec.mjs`, `e2e/selection-property-panel.spec.mjs`, `e2e/selection-toolbar.spec.mjs`, `e2e/module-group.spec.mjs`.

# Fair Stand — Kullanılabilirlik Yol Haritası

**Tarih:** 2026-10-23
**Kanonik konum:** kyrox-platform/projects/fair-stand/KULLANILABILIRLIK_YOL_HARITASI.md. Teşhis ve ara doğrulama kayıtları ilk incelemenin durumudur; son teslimat kaydı güncel durumdur.
**Hedef:** Daha önce çizim yapmamış bir kullanıcının hızla adapte olması.
**İncelenen:** standbuilder.app (rakip), depodaki rakip analizi (`kyrox-platform/projects/fair-stand/COMPETITOR_ANALYSIS.md`, 2026-10-20 karşılaştırması hâlâ geçerli), yardım rehberi (21 bölüm, ~45 KB), katalog (55 görünür kart), sağ tık menüsü, üretim listesi.

---

## Teşhis

### Bizde neden zor?
1. **Yardım bir başvuru kitabı.** 21 bölüm, ~45 KB metin; "şimdi şunu yap" diyen ilk adım rehberi yok.
2. **Katalog üretim parçalarıyla dolu.** Dikme, profil, kısa panel, kutu blok satış yapan birinin karşısına çıkıyor.
3. **Teknik dil.** Üretim listesinde `profile_388` gibi kodlar, "strip", "reçete" gibi terimler.
4. **Önemli işler sağ tıkta.** Panel ekleme, boyutlandırma sağ tıkta saklı; bilmeyen bulamaz, tablette uzun basma gerekir.
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
| 2 | **Basit mod**: satış kullanıcısına yapısal kartları (dikme, profil, kısa panel, kutu blok) gizle; yapısal parçalar otomatik gelsin. Ustaya "Gelişmiş" anahtarı. | 55 kartın yarısı yeni kullanıcı için anlamsız. | 1 gün |
| 3 | **Teklif / fiyat özeti**: sahnede canlı toplam ("Bu stand ≈ X ₺") ve satır özeti; maliyet CRM'de var. | Satış konuşmasının merkezi. | 2–3 gün |
| 4 | **Kurulu standda duvar yüksekliğini değiştirme** (B.16). | Kullanıcı fikir değiştirir; şu an baştan kurmak gerekiyor. | 1–2 gün |
| 5 | **Performans**: tam yeniden çizim yerine yerinde güncelleme; ekran kartına göre otomatik kalite (PERFORMANCE_ROADMAP P1–P3). | Yavaş his "bozuk" algısı yaratıyor. | 3–5 gün |

## 🟢 HIZLI YAPALIM — durum (2026-10-23)

| # | İş | Durum |
|---|---|---|
| 1 | **Uyarıları görünür yap**: sağ altta üst üste dizilen bildirimler (hata kırmızı, başarı yeşil, × ile kapanır, aynı mesaj tekrarlanmaz). Yalnız yerleştirme değil **bütün hatalar**: kaydetme, açma, görsel, revizyon, render, otomatik kayıt, yerleşim, 3B model yüklenemedi, çizilemeyen modül, eksik görsel. Hata için `alert` penceresi kalmadı; "Render hazır" artık görünür. | ✅ Yapıldı (PR bekliyor) |
| 2 | **Üretim listesinde teknik kodları gizle**: ekranda "8 × Profil 388 · adet"; kod ipucunda. "Birleşik leaf toplam" → "Toplam parça listesi", "Alt item yok" → "Parça yok". Kopyalanan üretim metni kodu taşır (üretim ekibi için). | ✅ Yapıldı (PR bekliyor) |
| 3 | **Rehberin başına "5 Dakikada İlk Standın"**: 5 numaralı adım (kur, ekle, yerleştir, giydir, kaydet/göster); eski uzun metin "Hızlı Başlangıç Ayrıntıları"na indi. | ✅ Yapıldı (PR bekliyor) |
| 4 | **Stand özeti başlığı**: sahnenin sol altında "L Stand Sol · 4 × 3 m · duvar 3,5 m". | ✅ Yapıldı (PR bekliyor) |
| 5 | **Sağ tık işleri seçim çubuğunda** | ✔ Zaten vardı: seçim çubuğundaki **⋯** sağ tık menüsünü açıyor (tablette de). |
| 6 | **İlk kullanım turu** | ✔ Zaten vardı: ilk sahnede 3 ipucu (katalogdan ekle, çift tıkla seç, Kaydet), bir kez gösterilir. |
| 7 | **Katalog kartına açıklama ipucu** | ➜ Orta vadeye taşındı: DB'de açıklama alanı + admin ekranı gerekiyor (kodda Item'a özel metin kuralı yasak). |
| 8 | **Boş proje kartı** | ✔ Gerek yok: yeni projede sihirbaz zaten açılıyor. |

**Pakete eklenen (kullanıcı isteği):**
- **Profil bağlantısı**: "Dik bağlantı yalnız dikme olan noktaya yapılır" kuralı **profiller için kaldırıldı**. Profil başka bir profilin her noktasına dik bağlanır (iki profilin arasına, ortasına; yaklaşınca 10 cm adımla yapışır), uç uca eklenir; birbirinin ortasından geçen iki profil (X) çakışır. Duvar panelleri için kural aynen geçerli. BOM'a profil T aparatı eklenmedi (gerekirse ayrı karar). ✅ Yapıldı (PR bekliyor)
- Kontrol: profiller zeminden tavana Z'de ↑/↓ ile taşınabiliyor — **zaten vardı**, değiştirilmedi.

**Önceden yapılanlar (aynı gün):** ölçü soran kartlarda ölçüsüz başlık ("Profil", "Dikme", "Panel Tek Sıra"), kapasite uyarısı kaldırıldı, profil 346,5 eklendi ve sahne profilleri temizlendi (#330), panel yan payı 2 cm (#331).

## 🟡 ORTA VADE

| # | İş | Not |
|---|---|---|
| 1 | Kenar çubuğunu 4 sekmeye indirmek: **Stand / Duvar & Panel / Mobilya / Teklif**. | Şu an çok sayıda katlanır bölüm. |
| 2 | **Tek tık seç, ikinci tık seçenekler** (panel işlemleri). | Dokunmatik ve yeni kullanıcı dostu. |
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
**Durum:** Öneri / yapılacaklar. Bu bölümün yazılması uygulama değişikliklerinin yapıldığı anlamına gelmez.
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

- [ ] Tek tık / dokunma ile seçim ve seçilene uygun görünür işlemler tasarlansın; temel kullanım çift tık veya sağ tık öğrenmeyi gerektirmesin.
- [ ] Temel işlemler **Taşı / Döndür / Kopyala / Sil** olarak anlaşılır biçimde sunulsun.
- [ ] Çoklu panel / çoklu modül seçimi, gruplama ve kilitleme ikinci seviyeye taşınsın.
- [ ] Panel yüzeyi ile bütün yapının seçimi açıkça ayırt edilsin; seçim sadeleşirken kamera hareketi, sürükleme ve dokunmatik etkileşim birlikte değerlendirilsin.

**Güncel durum (2026-10-23):** Uygulamanın tamamını yöneten tek Basit / Gelişmiş seçici sol menünün üstündedir. `src/selectionToolbar.js` seçim işlemlerini sunar. `src/moduleContextMenu.js` içindeki Gelişmiş, sağ tık menüsünün ayrıntı bölümüdür; ikinci bir uygulama modu değildir. Önceki envanter notu üst mod seçici eklenmeden önceki durumu anlatıyordu.

### 4. Renk, görsel ve zemin

- [ ] İlk görünümde renk paleti, görsel yükleme ve **Doldur / Sığdır / Kaldır** yeterli olsun.
- [ ] HEX, RGB ve CMYK alanları ayrıntılara taşınsın.
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
- [ ] **Render Al → Görsel oluştur**, **Canlı Paylaş → Ekranı paylaş** adlandırmaları değerlendirilsin; paylaşımın mevcut işlevi korunarak açık anlatılsın.
- [x] Üretim listesi ana işlem sırasından ayrıntılara taşınsın; hesaplama ve erişim devam etsin.
- [ ] Uzun rehberin yanında yapılan işe uygun kısa ipuçları gösterilsin; tam rehber erişilebilir kalsın.
- [ ] Canlı fiyat / teklif özeti Acil 3 ile ayrı takip edilsin; basit modda hazırmış gibi gösterilmesin.

### 7. Basit / Gelişmiş geçişinin kuralları

- [x] İki mod aynı proje verisi üzerinde çalışsın; ayrı proje biçimi veya ayrı hesaplama modeli oluşmasın.
- [x] Mod değiştirmek parçaları silmesin, sahneyi yeniden kurmasın veya tasarımı değiştirmesin.
- [ ] Gelişmişte hazırlanmış projenin yapısı basit modda eksiksiz görüntülensin; desteklenen düzenlemeler açıkça sunulsun.
- [ ] Basit modda gizlenen işlemler klavye kısayolları ve sağ tık menüsü dahil bütün giriş yollarıyla birlikte değerlendirilsin.
- [x] Mod seçimi yetki sistemiyle karıştırılmasın; bir kontrolün gizlenmesi güvenlik yetkisi sayılmasın.
- [x] **Karar (2026-10-23):** Tek Basit / Gelişmiş seçici sol menünün üstünde yer alır. Basit varsayılan moddur. Ölçüler Basitte kapalı, Gelişmişte açık başlar; kullanıcı Gelişmiş → Ayarlar bölümünden değiştirebilir; Ayarlar Basit modda görünmez. Tercihler oturum boyunca mod başına ayrı korunur; sayfa yeniden açılınca varsayılanlara döner.

### Önerilen uygulama sırası

1. **İlk paket — ekran ve etkileşim sadeleştirme:** mesafe / ölçü görünürlüğü (ızgara korunacak), menü düzeni, renk paneli ve görünür seçim işlemleri.
2. **İkinci paket — günlük kullanım:** başlangıç dili, zemin seçenekleri, kayıt / geçmiş, yardım ve müşteriye sunum adlandırmaları.
3. **Üçüncü paket — bütün yapı düzenleme:** duvar, depo ve alınlık üzerinden işlem; teknik parçaların otomatik çözülmesi ve bağlı öğelerin korunması.

**Takip ilkesi:** Her paket uygulamadan önce mevcut davranış ve etkileriyle somutlaştırılacak. Bu listedeki maddeler uygulanıp doğrulanmadan tamamlandı olarak işaretlenmeyecek.


### Ortak menü ve mod tercihleri — 2026-10-23 uygulaması

- Sol menünün üstünde, referans görseldeki gibi tek **Basit / Gelişmiş** mod seçici.
- İki modda ortak **açılır/kapanır bölüm şablonu**: aynı başlık, ok, içerik aralığı ve klavye davranışı; bölümler bağımsız açılıp kapanır.
- Ortak **Ayarlar** bölümünde **Ölçüleri göster** ayarı. Mod varsayılanları kullanıcı ayarını kilitlemez.
- Gelişmişte ölçüyü kapatan kullanıcı başka moda geçip geri döndüğünde aynı oturumda kapalı bulur; Basitteki tercih ayrıdır.
- Zemin ızgarası moddan ve ölçü ayarından bağımsız, mevcut yerleştirme davranışıyla korunur.
- Bu aşama katalog filtreleme, yapı otomasyonu ve diğer sadeleştirme maddelerinin tamamlandığı anlamına gelmez.
- Durum: Yerel uygulama doğrulandı — 1.455 birim testi, 228 Chromium E2E testi, derleme ve değişiklik sözleşmesi kontrolü başarılı. Diğer basit mod maddeleri açık; bu kayıt yayın veya PR CI onayı anlamına gelmez.


### Taşıma ızgarası ve grup ölçüsü — 2026-10-23 takip

- **Kesin kural:** Sahnede bir öğe veya grup taşınıyorsa ızgara görünür; taşıma bittiğinde kapanır. Katalogdan yerleştirme ve kopya önizlemesi de bu kapsamdadır. Mod ve ölçü tercihi bu kuralı etkilemez.
- **Bulunan eski hata:** Izgara yalnız placement ghost görünürlüğüne bağlıydı; grup taşıması gerçek üyeleri doğrudan hareket ettirdiğinden ızgara açılmıyordu. Ortak görünürlük denetimine etkin sürükleme de eklendi.
- Görünüm panelindeki gereksiz ızgara açıklaması kullanıcı isteğiyle kaldırıldı.
- [ ] **Grup ölçüsü önerisi:** Grubun dış sınırlarından toplam genişlik / derinlik / yükseklik; taşırken stand kenarlarına uzaklık. Mevcut Ölçüleri göster tercihine bağlı olacak. Henüz uygulanmadı.

- Izgara düzeltmesinin yerel doğrulaması tamamlandı: 1.455 birim testi, 228 Chromium E2E testi, derleme ve değişiklik sözleşmesi kontrolü başarılı. Grup ölçüsü maddesi açık kalıyor.


### Genel ayar alanı — 2026-10-23

- Kullanıcı kararı: Bölümün adı **Ayarlar**; yalnız görünüm değil, editörün genel kullanıcı tercihleri burada toplanacak.
- **Ölçüleri göster** ve **Otomatik kayıt** aynı ortak açılır/kapanır bölümde. Otomatik kayıt kontrolü Proje bölümünden buraya taşındı; kayıt davranışı, seçenekleri ve kalıcı tercihi değişmedi.
- Sonraki ileri düzey kullanıcı ayarları aynı şablona eklenecek; bu aşamada yeni bir yetki kısıtı oluşturulmadı.

- Ayarlar alanının doğrulaması: 1.455 birim testi, derleme ve sözleşme kontrolü başarılı. 228 tarayıcı senaryosunun 224'ü toplu koşuda geçti; çalışma sırasında sayfa yenilenmesinden etkilenen dört senaryo tek işçiyle tekrar çalıştırılıp başarılı doğrulandı.


### Başlangıç ve menü düzeni — 2026-10-23 kullanıcı kararları

- Sihirbazda Genişlik / Derinlik metreyle girilecek; 0,5 m adımları ve 4,5 / 6,5 gibi virgüllü değerler desteklenecek. Canonical hesap ve proje birimi cm kalır.
- Depo, alınlık ve zemin ifadeleri sadeleştirilecek; zemin seçimi sihirbaza eklenecek.
- Sol menünün Stand / Ürün ekle / Tasarım etrafında sadeleştirilmesi iptal edilmedi; kullanıcı bu maddeyi şimdilik bekletiyor. Mevcut uygulama bu düzeni değiştirmiyor.
- Kaydet ve Ekranı paylaş üst araç alanından erişilecek.
- Ayarlar, Gruplar, Şablonlarım ve Standartlar yalnız Gelişmiş modda görünecek. Ayarlar bölümünün Basitte de görünmesi yönündeki önceki karar bu kararla güncellenmiştir; gizlemek proje verisini silmez.
- Durum: Yerel uygulama doğrulandı — 1.457 birim testi, 228 Chromium E2E testi, derleme ve değişiklik sözleşmesi kontrolü başarılı. Menü sadeleştirme maddesi yapılacaklar listesinde bekliyor.


### Metre gösterimi — tüm kurulum yüzeyleri (2026-10-23)

- Genişlik ve derinlik yalnız sihirbazda değil, **sol menüde de metre** olarak girilir. 4,5 ve 6,5 gibi virgüllü değerler desteklenir.
- **Duvar yüksekliği** seçenekleri hem sol menüde hem sihirbazda metre gösterir (ör. 3,5 m). Canonical seçenek değerleri ve proje kaydı cm kalır.
- Ortak giriş/gösterim sahibi: `src/standSizeInput.js`. Mevcut proje açılınca kayıtlı cm ölçüleri metre biçiminde gösterilir; ölçü veya BOM yeniden ölçeklenmez.
- Sol menüdeki 50 cm / 5000 cm açıklaması, 0,5 m adımları ve 50 × 50 m sınırı olarak güncellendi.
- Hedefli kayıt/açma, sihirbaz ve duvar yüksekliği akışları: 27 tarayıcı testi geçti. Genel doğrulama tamamlandı: 1.458 birim testi, derleme ve sözleşme kontrolü başarılı. 229 tarayıcı senaryosundan 228'i toplu koşuda geçti; yerel bağlantı kesilmesi (ECONNRESET) nedeniyle yarıda kalan tek senaryo tekrar çalıştırılıp başarılı doğrulandı.


### Herşeyi Kilitle — 2026-10-23

- Gelişmiş → Ayarlar altına **Herşeyi Kilitle** kutusu eklendi. Mevcut Ctrl+A → Kilitle / Kilidi aç işlemiyle aynı öğe kapsamını kullanır.
- İşaretlenince bütün mevcut öğeler kilitlenir; kaldırılınca öğelerin ve grupların kilitleri açılır. Grup üyeliği korunur.
- Tam kilitte işaretli, kısmi kilitte ara durum, boş sahnede pasif. Başka bir işlemle değişen gerçek kilit durumu kutuya yansır.
- Yeni öğeler için sürekli otomatik kilit politikası oluşturmaz; geri al/ileri al ve proje kaydı mevcut kilit davranışıyla çalışır.
- Doğrulama tamamlandı: 1.459 birim testi ve 230 Chromium tarayıcı testi geçti; derleme ve değişiklik sözleşmesi başarılı.

### Kılavuz ve teslimat — 2026-10-23

- Kılavuzda tek açılır/kapanır **Gelişmiş > Ayarlar** bölümü bulunur. Herşeyi Kilitle, Ölçüleri göster ve Otomatik kayıt burada ayrı alt başlıklarla açıklanır.
- Kilitlerin projeyle kaydedilmesi, ölçü tercihinin yalnız oturumda hatırlanması ve otomatik kayıt süresinin tarayıcıda saklanması ayrı ayrı belirtilir.
- Uygulama commit’i: `401bd238`, dal: `feat/basit-mod-olcu-gorunumu`; GitHub’a gönderildi. Main’e merge veya canlı deploy yapılmadı.
- Son tam yerel doğrulama: **1.459 birim testi, 230 Chromium E2E testi, build ve contract:verify başarılı**. Önceki bölümlerdeki daha düşük sayılar ara doğrulama kayıtlarıdır.
- Açık işler: grup dış sınır ölçüleri; Stand / Ürün ekle / Tasarım menü düzeni (ertelendi); yapı bütününü düzenleme ve teknik parçaların otomatik çözülmesi; diğer işaretlenmemiş sadeleştirme maddeleri.
### Bölgesel zemin ve genel mod kuralı — 2026-10-23

- **Genel kullanıcı kuralı:** Basit / Gelişmiş geçişi yalnız arayüzü ve yardımcı gösterimleri değiştirir; standı veya tasarım verisini değiştirmez. Ölçü, öğe, yerleşim, grup, kilit, yüzey ve zeminler korunur. Uygulama sözleşmesi: `kyrox-fair-stand/PROJECT_RULES.md`.
- Tüm standın baz kaplaması iki modda da seçilebilir. Alan X/Y, genişlik/derinlik, alan kaplaması ve ekle/çiz/sil kontrolleri yalnız Gelişmişte görünür.
- Mevcut bölgesel zemin Basitte sahnede, BOM ve proje kaydında korunur; Gelişmişe dönünce aynı alan düzenlenir. Mod değişiminde yalnız tamamlanmamış çizim önizlemesi iptal edilir.
- Uygulandı ve doğrulandı: 1.459 birim testi, build ve contract:verify başarılı. 230 Chromium senaryosunun 225 tanesi ilk koşuda geçti, 2 tanesi aynı koşuda yeniden denemede geçti; kalan 3 senaryo tek işçiyle tekrar çalıştırılıp geçti. Bölgesel zemin senaryosu tam koşuda başarılı; iki yönlü mod geçişi, görüntü, stand/modules kayıt eşitliği ve Basitte kayıt/açma denetlendi. Henüz commit/push yapılmadı.
### Geçmiş ve üretim ayrıntıları — 2026-10-23

- Proje içindeki revizyon listesi **Geçmiş** başlığından açılır; başlangıçta kapalıdır, kayıt yoksa görünmez. Seçili revizyon, eski kaydı açma ve son revizyona dönme mevcut akışla çalışır.
- Üretim Listesi üst banttan **Ayrıntılar → Üretim Listesi** girişine taşındı. Ayrıntılar başlangıçta kapalıdır; iki modda da erişilir. Sahne yokken düğme pasiftir.
- Mevcut BOM hesabı, liste paneli, metin indirme ve ayrı pencere korunur. Bu bölümleri açıp kapatmak stand verisini değiştirmez.
- Kılavuz güncellendi. Son tam doğrulama: 1.459 birim testi, 230/230 Chromium E2E testi (6,4 dk), build ve contract:verify başarılı. Üst bantla seçim çubuğunun çakışması da giderildi; Kaydet erişimi gerçek tarayıcı akışında doğrulandı. Uygulama yerel; commit/push yapılmadı.

### GitHub teslimatı — 2026-10-23

- İlk mod/ayar/metre paketi `401bd238`: [PR #334](https://github.com/kyrox-studio/kyrox-fair-stand/pull/334) main dalına birleştirildi.
- Bölgesel zemin, Geçmiş ve üretim ayrıntıları paketi `48fcb9c4`: [PR #335](https://github.com/kyrox-studio/kyrox-fair-stand/pull/335). Yerel 1.459 birim ve 230/230 Chromium E2E başarılı; GitHub CI ve otomatik birleştirme takip ediliyor.
- Canlı deploy yapılmadı. Önceki commit/push bekleme notları ara çalışma kayıtlarıdır; bu teslimat kaydı onları günceller.

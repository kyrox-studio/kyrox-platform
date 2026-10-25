# Fair Stand — Seçime Göre Özellik Paneli Yol Haritası

**Kod karşılaştırması:** 2026-10-25, Fair Stand `main` / `ac18bc1a` (PR #340 ve #341 sonrası).
**Durum:** Ortak panel ve seçim girişleri kodda mevcut. Canlı varyant seçimi ve desteklenen düzenlenebilir ölçü alanları da panelde. Görsel düzen / erişilebilirlik kabulü açık.
**Okuma kuralı:** `[x]` kodda mevcut davranışı, `[ ]` kalan doğrulama veya uygulamayı gösterir. Bu belge düzenlemesinde testler yeniden çalıştırılmadı; geçmiş koşu sonuçları [PROJECT_STATUS.md](PROJECT_STATUS.md) içindedir. Kodda bulunması, bütün görsel veya dokunmatik kabul senaryolarının kapandığı anlamına gelmez.

**Sıralama:** Önce işlev ve mevcut davranışların korunması; görsel düzenleme en son.

## Kesin kapsam

Sağ tık işlemleri ve seçimin desteklediği özellikler ortak bir panelde sunulacak. Her ürüne aynı ayarlar gösterilmeyecek. Item, seçilen yüzey, mevcut değerler, kilit ve işlem yetkisi birlikte değerlendirilecek.

**Mevcut tek/çift tık araç çubuğu kaldırılmayacak:** Döndür, Kopyala, Sil, Çoklu panel / Çoklu modül korunacak. Kullanıcının sonraki kararıyla ⋯ kaldırıldı; panel tek tıkla açılır. Çubuk ile panel ayrı erişim yüzeyleridir; aynı işlemi ayrı iş kurallarıyla gerçekleştirmeyecekler.

İlk adım normal tık, çift tık, sağ tık ve çoklu seçim yollarının tamamını kapsar. Yalnız tek öğede açılan bir panel, ilk adım tamamlandı sayılmaz.

## Mevcut uygulama dayanağı

Karşılaştırma mevcut kaynak kod ve depodaki regresyon senaryoları üzerinden yapıldı; katalog/Ekle çalışması artık bekleyen yerel değişiklik olarak anlatılmıyor.

| Mevcut sahip | Panel çalışmasındaki rolü |
|---|---|
| `src/scene3d.js` | Seçim, hit testi, panel / modül ayrımı, `getSelectionContext`, `buildModuleContext`, sağ tık, uzun basma ve çoklu seçim |
| `src/selectionToolbar.js` | Korunacak seçim araç çubuğu, Çoklu panel / Çoklu modül; ⋯ kaldırıldı |
| `src/moduleContextMenu.js` | Ortak Özellikler paneli, komut uygunluğu, varyant/ölçü alanları ve Ekle penceresi |
| `src/itemCapabilities.js` | Item yüzey yetenekleri: renk, görsel, cam, Lightbox ve Mesh |
| `src/main.js` | Mevcut işlem bağlantıları, renk / görsel kontrolleri, ürün özel işlemleri ve uygulama yetkileri |
| `src/moduleGroups.js`, `src/moduleLock.js` | Grup ve kilit kuralları |
| `src/rectSelection.js`, `src/surfaceStateBinding.js` | Çoklu panel geometrisi ve yüzey durumunun doğru kayda yazılması |

Item adı, kart başlığı veya model dosya adı yetenek kaynağı yapılmayacak. Mevcut Item ve yüzey verileri kullanılacak; eksik yetenek yeni ürün kuralı uydurularak tamamlanmayacak.

## 1. Adım — İşlevsel panel ve bütün seçim girişleri

Bu adımda sade, mevcut ortak bileşenleri kullanan bir panel yeterlidir. Son renk, boşluk, ikon ve yerleşim tasarımı bu adımın şartı değildir.

### 1.1 Korunacak etkileşimler

| Giriş | Seçim davranışı | Araç çubuğu / ortak panel |
|---|---|---|
| Yüzeye normal tık / dokunma | Mevcut panel / yüzey seçimi; renk ve görselin hedefi aynı yüzey kalır | Mevcut çubuk kalır. Ortak panel açılır; açıksa yeni seçime göre güncellenir. |
| Yüzey hedefi olmayan öğeye normal tık | Mevcut öğe seçimi korunur | Desteklenen öğe işlemleri ve doğru özellik bağlamı kullanılır. |
| Çift tık / çift dokunma | Öğenin tamamını seçer | Mevcut çubuk kalır; Çoklu modül davranışı korunur. Panel öğe bağlamına geçer. |
| Sağ tık | Tek seçimde mevcut hedef belirleme kuralı korunur | Ayrı eski menü yerine ortak panel açılır; sağ tıklanan yüzey / öğe ve işlem kapsamı açık gösterilir. |
| Çubuktaki ⋯ | Kullanıcı kararıyla kaldırıldı | Yerine öğeye/yüzeye tek tık paneli açar. |
| Dokunmatik uzun basma | Mevcut sağ tık eşdeğeri | Aynı panel açılır; ardından gelen tarayıcı olayı ikinci kez işlem başlatmaz. |
| Ctrl/Cmd + tık | Seçili olana göre panel veya modül çoğulu | Çoklu panel ve çoklu modül birbirine dönüşmez. |
| Çoklu panel düğmesi | Mevcut dokunma / parmak taraması ve tek seferlik mod | Seçim korunur; panel toplu yüzey bağlamını gösterir. |
| Çoklu modül düğmesi | Öğeye dokunarak ekleme / çıkarma | Panel çoklu öğe bağlamını gösterir. |
| Boş alana tık / Esc | Mevcut seçim temizleme ve işlem iptali öncelikleri | Panel kapanır; seçim yoksa eski öğenin işlemleri aktif kalmaz. |
| Boş alana sağ tık | Mevcut seçimi gizlice başka öğeye çevirmeme | Hedef yoksa sahte öğe bağlamı üretilmez; mevcut kapanma davranışı korunur. |

**Açılma kararı (kullanıcı kararı, 2026-10-24):** Panel öğeye / yüzeye **tek tıkla** (dokunmada tek dokunuşla) açılır; referans BoothDesign'daki gibi sahnenin sağında, seçimin başlığıyla. Sağ tık ve uzun basma aynı paneli açmaya devam eder. Sürükleme, kamera hareketi, sağ tuşla çevirme ve kopya yerleştirme paneli açmaz; boş alana tık paneli kapatır. Mevcut araç çubuğu (Döndür, Kopyala, Sil, Çoklu panel / modül) korunur; ⋯ kullanıcı kararıyla kaldırılmıştır.

- [x] Tüm girişler aynı güncel seçim bağlamına bağlansın.
- [x] Tek tık / çift tık ayrımı, sürükleme eşiği ve kamera hareketi bozulmasın.
- [x] Panel üzerindeki tık, yazı girişi ve kaydırma sahne seçimini veya kamerayı tetiklemesin.
- [x] Çoklu öğe seçimi varken sağ tık seçimi tek öğeye düşürmesin; komşuya kayan sağ tık için mevcut koruma taşınsın.
- [x] Çoklu panel içinde sağ tık toplu yüzey hedefini kaybetmesin. Seçim dışındaki sağ tık davranışı mevcut kuralla karşılaştırılıp ayrıca test edilsin; panel geçişi gerekçesiyle sessizce değiştirilmesin.

### 1.2 Tek öğe / yüzey yetenekleri

- [x] Panel başlığında gerçek öğe adı, varsa ölçüsü ve seçilen yüzey gösterilsin; öğe ile yüzey birbirine karıştırılmasın.
- [x] Renk destekleniyorsa renk kartelası / mevcut renk kontrolleri gösterilsin.
- [x] Görsel destekleniyorsa mevcut arşiv / yükleme, Doldur, Sığdır, Kaldır ve uygun görsel ölçülendirme işlemleri gösterilsin.
- [x] Cam, Metal Separatör, Lightbox, Mesh ve bunların mevcut dönüşüm / birlikte kullanım kuralları korunsun.
- [x] Raf altı ışığı, Lightbox ışığı, arka yüze panel, dikme boyu, profil boyu ve boyutlandırma gibi mevcut özel işlemler yalnız geçerli hedefte gösterilsin.
- [x] TV gibi çok yüzeyli öğelerde gövde yeteneği ekrana, ekran yeteneği bütün ürüne genellenmesin; mevcut yüzey ayrımı gösterilsin.
- [x] Item'ın desteklemediği özellik gösterilmesin. Desteklediği ama o anda kullanılamayan işlem, mevcut durumu koruyarak anlaşılır gerekçeyle pasif gösterilsin.

### 1.3 Sağ tık komutlarının eksiksiz taşınması

| İşlem ailesi | Korunacak mevcut işlevler |
|---|---|
| Temel öğe | Kilitle / kilidi aç, döndür, sil; çubuktaki kopyalama aynı mevcut akışı sürdürür |
| Ekle | Mevcut Ekle… kataloğu, tek öğe seçimi, ölçü soruları ve hedefe göre yerleştirme |
| Ürün özel | Mevcut boyutlandırma, görsel ölçülendirme, raf ışığı, arka panel, dikme / profil boyu |
| Yüzey | Cam, Metal Separatör, Lightbox, Mesh ve uygun aydınlatma |
| Çoklu öğe | Seçilenleri kilitle, kilidi aç, sil ve mevcut grup seçenekleri |
| Grup | Oluştur, gruba ekle, adlandır, kilitle, çoğalt, üyeyi çıkar, çöz, seçilen grupları çöz ve grubu sil; mevcut uygunluk koşullarıyla |

- [x] Mevcut komut aileleri ortak panelde: temel öğe, Ekle…, özel yüzey, ürün ölçüsü, çoklu seçim ve grup. Kod sahibi `src/moduleContextMenu.js`; bu liste gelecekte eklenecek komutların kendiliğinden kabulü değildir.
- [x] Her panel kontrolü mevcut işlem fonksiyonuna yönlensin. Yeni menü için paralel boyama, silme, ekleme, kilit veya BOM mantığı yazılmasın.
- [x] Çubuktaki Döndür / Kopyala / Sil düğmeleri doğrudan çalışmaya devam etsin; kullanıcı bu işlemler için panel açmaya zorlanmasın.
- [x] İlk adımın sonunda sağ tık eski bağımsız menü yerine ortak paneli açsın. Geçiş sırasında eski menü karşılaştırma için geçici tutulabilir; kalıcı iki farklı işlem sistemi oluşmasın.

### 1.4 Çoklu panel

- [x] Başlıkta panel sayısı ve gerekiyorsa kaç öğeye ait olduğu gösterilsin.
- [x] Ctrl/Cmd ile mevcut dikdörtgen seçimi, farklı modüllere yayılan seçim, köşe ve ön / arka yüz kuralları korunsun.
- [x] Banko gibi yüzleri tek tek seçilen ürünlerde mevcut özel seçim davranışı korunsun; her yüzey dikdörtgen panel gibi ele alınmasın.
- [x] Parmak taraması / dokunma ile Çoklu panel modu aynı seçim kümesini panele iletsin; parmak kalkınca modun kapanması seçimi silmesin.
- [x] Ortak özellikler bütün seçimin yeteneklerinden hesaplanarak sunulsun; yalnız son tıklanan panelin yeteneğine göre kontrol açılmasın.
- [x] Panel açılınca ilk öğenin değeri diğerlerine kendiliğinden uygulanmasın.
- [x] Her hedefin kabul etmediği toplu işlem gizlensin veya açık gerekçeyle pasif olsun; kullanıcıdan habersiz yalnız bazı yüzeylere uygulanmasın.
- [x] Görsel / Lightbox / Mesh / Metal Separatör için mevcut aynı düzlem, tam dikdörtgen ve birleşim doğrulamaları aynen kullanılsın. Geçersiz seçimde neden belirtilsin.
- [x] Birleşik görselin hedefleri ve yüzey kaydı korunsun; kopya gruptaki düzenleme kaynak gruba yazılmasın.

### 1.5 Çoklu modül ve grup

- [x] Çoklu öğe seçimi ile isimli grup ayrı bağlamlar olarak gösterilsin. Birden fazla öğe seçmek kendiliğinden grup oluşturmasın.
- [x] Öğenin tamamı seçiliyken Ctrl/Cmd + tık ve Çoklu modül düğmesiyle ekle / çıkar davranışı korunsun.
- [x] Başlıkta öğe sayısı, varsa grup bilgisi ve kilitli öğe sayısı gösterilsin.
- [x] Mevcut toplu kilitleme, kilit açma, silme ve grup işlemleri taşınsın; desteklenmeyen toplu döndürme, kopyalama veya boyutlandırma bu çalışma adına icat edilmesin.
- [x] Seçimde kilitli öğe varsa mevcut atomik toplu silme kuralı korunsun; kilitsiz parçaların bir kısmı sessizce silinmesin.
- [x] Grup kilidi ve öğenin kendi kilidi ayrımı korunsun. Kilit, mevcut kurallar izin veriyorsa renk / görsel işlemlerini gereksiz yere kapatmasın.
- [x] Grup üyesi yüzey düzenlemesi ile bütün gruba uygulanacak işlem açıkça ayırt edilsin.
- [x] Tek öğenin mavi, grup / çoklu öğenin ortak turuncu kesik çerçevesi korunsun. Çoklu panelin yüzey seçimi göstergeleriyle karıştırılmasın.
- [x] Grup aç / kapat ve grup toplam ölçüleri bu geçişte eklenmedi; ayrı kapsamda kalıyor.

### 1.6 Panelin durumu ve güvenli işlem hedefi

- [x] Seçim değişimi, kopyalama modu, silme, grup çözme, geri al / ileri al ve proje açma sonrası panel güncel seçimden beslensin.
- [x] Eski mesh / öğe referansına yazılmasın; gecikmeli görsel yükleme veya ölçü penceresinin hedefi kimlik üzerinden doğrulansın. Hedef yoksa işlem uygulanmasın.
- [x] Panelin açılması, kapanması veya Basit / Gelişmiş geçişi tasarım verisine ve undo geçmişine yazmasın.
- [x] Kopyalama sırasında Kopyalamayı bitir kontrolü ve Esc önceliği korunsun. Panelin kapanması kopya yerleştirme hedefini değiştirmesin.
- [x] Metin alanında Delete, Ctrl+A, Ctrl+Z gibi kısayollar sahneyi etkilemesin; ölçü / görsel pencereleri ve Esc iptali çakışmasın.
- [x] Item yeteneği, kilit ve etkili kullanıcı yetkisi ayrı değerlendirilsin. Basit / Gelişmiş modu yetkilendirme yerine kullanılmasın.

### İşlevsel uygulama ve kalan kabul ayrımı

Normal tık / çift tık araç çubuğu korunmuş; tek tık, sağ tık ve uzun basma ortak paneli kullanıyor. Tek yüzey, tek öğe, çoklu panel, çoklu modül ve grup bağlamlarının kod yolları mevcut. Aşağıdaki açık doğrulamalar ve 3. adımın görsel/erişilebilirlik kabulü ayrıca kapatılmalı; kaynak incelemesi bunların yerine geçmez.

## Sonradan eklenen mevcut işlevler

- [x] Ürün ailesinin desteklediği varyantlar panelde seçilir; DB/katalog verisi kullanılır (`describeModuleVariants`, `describeVariantChoices`, 0086).
- [x] Kutu blok, ışıklı strafor ve tül ölçüleri; dikme/profil boy seçenekleri; uygun görselin ölçüsü mevcut sınırlarla panelde düzenlenir (`describeDimensionFields`). Bu, her ürün için serbest boyutlandırma değildir.
- [x] Vitrin gövdesi renk kontrolü sunta Item yeteneği üzerinden açılır; cam raflara aynı renk uygulanmaz (0085).
- [x] Seçim açıklaması korunur; uzun içerik panel içinde kayar. Editör yeniden açıldığında eski seçim çubuğu temizlenir.

**Kullanıcı kararları:** Çoklu panel için fazladan işlem hedefi satırı ve Lightbox/Mesh için yeni “Karışık renk” işi bu yol haritasından çıkarıldı. Genel sabit ürün ölçülerini salt bilgi olarak ekleme denemesi iptal edildi; mevcut düzenlenebilir ölçülerle karıştırılmayacak ve açık iş sayılmayacak.

## 2. Adım — İşlevsel kabul ve geçiş temizliği

- [x] Eski menü / renk-görsel alanı ile yeni panel arasında işlev karşılaştırması tamamlanıp artık kullanılmayan arayüz ve olay bağları temizlensin; çalışan kontroller kopyalanıp iki ayrı kaynak bırakılmasın.
- [x] Mevcut katalog Ekle… akışı, 0082/0083 ile yürüyen ürün ailesi ölçü seçimi, desteklenen ekleme hedefleri ve hata bildirimleri korunmuş olsun. Bu migration'lar bu panel işi için yeniden oluşturulmasın.
- [x] Basit / Gelişmişte aynı seçim bağlamı ve desteklenen işlemler doğrulansın. Görsel hiyerarşi veya hangi kontrolün ayrıntılarda duracağı son tasarımda belirlensin; ürünün yeteneği moda göre değişmesin.
- [ ] Büyük seçimlerde ve sahne sürüklemesinde panel güncellemesi gereksiz sahne kurulumuna veya her karede tam arayüz üretimine neden olmasın.
- [ ] Renk, görsel, özel yüzey, ölçü, silme ve grup işlemlerinin mevcut undo / redo, autosave, revizyon ve kayıt-açma akışlarıyla eşliği doğrulansın.
- [x] Panel görünürlüğü ve genişliği proje şemasına veya BOM'a eklenmiyor. Yetki/organization modeli bu arayüz çalışmasıyla değişmiyor; katalog yetenek düzeltmeleri ayrı 0084/0085 migration'larıyla yapıldı.

### Test matrisi

| Senaryo | Beklenen kanıt |
|---|---|
| Normal / çift tık | Aynı hedef seçilir ve panel açılır; mevcut çubuk ve Çoklu panel / modül ayrımı kalır. |
| Sağ tık / uzun basma | Aynı seçimde aynı işlem listesi ve aynı sonuç; çift tetikleme yok. |
| Renk / görsel almayan ürün | Desteklenmeyen kontrol görünmez; mevcut ürün ve yüzey kuralları kullanılır. |
| Tek yüzey / çok yüzeyli öğe | İşlem doğru yüzeye yazılır; gövde ve ekran karışmaz. |
| Çoklu panel | Geçerli dikdörtgen işlem görür; geçersiz düzlem / birleşim nedeni açıklanır. |
| Karma yetenek | Panel açmak state değiştirmez; sessiz kısmi uygulama olmaz. |
| Çoklu modül / grup | Seçim sağ tıkla dağılmaz; grup üyeliği, kilit ve atomik silme korunur. |
| Kopya görsel grubu | Kopyanın değişikliği kaynağa yazılmaz. |
| Panel odağı / iki parmak / sürükleme | Panel kullanımı sahneyi yanlışlıkla taşımaz; kamera ve pinch davranışları korunur. |
| Geri al / kayıt-açma / mod değişimi | Tasarım ve desteklenen özellikler kaybolmaz; panel doğru bağlamla güncellenir. |

**Doğrulama:** Ortak bağlam ve action uygunluğu için anlamlı unit testleri; gerçek Chromium'da tek tık, çift tık, sağ tık, çoklu panel, çoklu modül, grup ve dokunmatik E2E. Uygulama deposunun change-contract, tam unit, build ve tam E2E kapıları geçmeden işlevsel kabul verilmez. Yeni panelin insan/AI kullanım açıklaması kılavuzda güncellenir.

İlgili mevcut regresyon başlangıçları: `e2e/double-click-select.spec.mjs`, `e2e/selection-toolbar.spec.mjs`, `e2e/module-group.spec.mjs`, `e2e/module-lock.spec.mjs`, `e2e/color-image-capability.spec.mjs`, `e2e/grouped-image-direction.spec.mjs`, `e2e/panel-sweep.spec.mjs`; dikdörtgen seçim için `test/rectSelection.test.js`. Ek kanıtlar: `e2e/selection-property-panel.spec.mjs`, `e2e/variant-snap-fit.spec.mjs`, `e2e/shelf-width-variant.spec.mjs` ve `test/selectionSurfaceSummary.test.js`. Test dosyasının bulunması bu incelemede yeni bir başarılı koşu iddiası değildir.

## 3. Adım — Görsel düzenleme

**Mevcut:** Sağda kaydırılabilir kart, başlık/alt başlık, kilit notu, ikonlar, renk/görsel alanları ve dar ekran CSS'i var. Aşağıdaki maddeler bu temelin son görsel ve erişilebilirlik kabulüdür; sıfırdan başlanacak işler değildir.

- [ ] Panelin konumu ve boyutu gerçek kullanım üzerinden belirlensin; masaüstünde sağ panel ve mobilde alttan açılan panel değerlendirilsin.
- [ ] Seçili öğe / yüzey / grup başlığı, bölüm sırası, boş durum ve kilit açıklaması tasarlansın.
- [ ] Renk kartelası, görsel alanı, ölçü seçenekleri, Ekle… ve grup işlemleri ortak bileşenlerle düzenlensin.
- [ ] Basit / Gelişmiş ayrıntı hiyerarşisi son haline getirilsin; mevcut seçim araç çubuğu ve çoklu seçim düğmeleri korunsun.
- [ ] Sahne, seçili öğe, Kaydet, görünüm küpü ve seçim çubuğu örtüşmeleri masaüstü / dar ekran / tablette kontrol edilsin.
- [ ] Klavye odağı, ekran okuyucu etiketleri, kaydırma, dokunma hedefleri ve açık / kapalı panel davranışı doğrulansın.

## Kapsam dışında

Yeni Item / BOM kuralı, yeni ürün özelliği, serbest ölçü sistemi, yeni grup aç/kapat davranışı, grup toplam ölçüleri, otomatik yapı çözme, katalog / sol menüyü baştan düzenleme ve backend yetki modeli değişikliği bu planın kapsamı değildir. Mevcut yeteneği görünür kılmak, o ürün için yeni bir yetenek üretmek anlamına gelmez.

Bu belge güncel yol haritasıdır; uygulama ve doğrulama durumu üstteki kayıt ile PROJECT_STATUS.md üzerinden izlenir.

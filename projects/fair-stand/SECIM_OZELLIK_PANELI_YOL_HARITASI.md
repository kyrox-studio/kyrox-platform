# Fair Stand — Seçime Göre Özellik Paneli Yol Haritası

**Tarih:** 2026-10-24  
**Durum:** Kullanıcı kararlarına göre hazırlanmış uygulama planı; panel henüz uygulanmadı.  
**Sıralama:** Önce işlev ve mevcut davranışların korunması; görsel düzenleme en son.

## Kesin kapsam

Sağ tık işlemleri ve seçimin desteklediği özellikler ortak bir panelde sunulacak. Her ürüne aynı ayarlar gösterilmeyecek. Item, seçilen yüzey, mevcut değerler, kilit ve işlem yetkisi birlikte değerlendirilecek.

**Mevcut tek/çift tık araç çubuğu kaldırılmayacak:** Döndür, Kopyala, Sil, Çoklu panel / Çoklu modül ve ⋯ korunacak. Çubuk ile panel ayrı erişim yüzeyleridir; aynı işlemi ayrı iş kurallarıyla gerçekleştirmeyecekler.

İlk adım normal tık, çift tık, sağ tık ve çoklu seçim yollarının tamamını kapsar. Yalnız tek öğede açılan bir panel, ilk adım tamamlandı sayılmaz.

## Mevcut uygulama dayanağı

2026-10-24 yerel çalışma ağacı okundu. Katalog / Ekle penceresine ilişkin devam eden yerel değişiklikler var; bu belge onları inceleyip kabul eden bir audit veya canlı yayın kaydı değildir. Uygulama başında güncel taban tekrar doğrulanacak.

| Mevcut sahip | Panel çalışmasındaki rolü |
|---|---|
| `src/scene3d.js` | Seçim, hit testi, panel / modül ayrımı, `getSelectionContext`, `buildModuleContext`, sağ tık, uzun basma ve çoklu seçim |
| `src/selectionToolbar.js` | Korunacak seçim araç çubuğu, Çoklu panel / Çoklu modül ve ⋯ girişi |
| `src/moduleContextMenu.js` | Taşınacak sağ tık komutları, görünürlük koşulları ve mevcut Ekle penceresi |
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
| Yüzeye normal tık / dokunma | Mevcut panel / yüzey seçimi; renk ve görselin hedefi aynı yüzey kalır | Mevcut çubuk kalır. Panel açıksa yeni seçime göre güncellenir. |
| Yüzey hedefi olmayan öğeye normal tık | Mevcut öğe seçimi korunur | Desteklenen öğe işlemleri ve doğru özellik bağlamı kullanılır. |
| Çift tık / çift dokunma | Öğenin tamamını seçer | Mevcut çubuk kalır; Çoklu modül davranışı korunur. Panel açıksa öğe bağlamına geçer. |
| Sağ tık | Tek seçimde mevcut hedef belirleme kuralı korunur | Ayrı eski menü yerine ortak panel açılır; sağ tıklanan yüzey / öğe ve işlem kapsamı açık gösterilir. |
| Çubuktaki ⋯ | Mevcut seçimden bağlam alınır | Sağ tıkla aynı ortak panel açılır; yeni hit testiyle seçimin hedefi değiştirilmez. |
| Dokunmatik uzun basma | Mevcut sağ tık eşdeğeri | Aynı panel açılır; ardından gelen tarayıcı olayı ikinci kez işlem başlatmaz. |
| Ctrl/Cmd + tık | Seçili olana göre panel veya modül çoğulu | Çoklu panel ve çoklu modül birbirine dönüşmez. |
| Çoklu panel düğmesi | Mevcut dokunma / parmak taraması ve tek seferlik mod | Seçim korunur; panel toplu yüzey bağlamını gösterir. |
| Çoklu modül düğmesi | Öğeye dokunarak ekleme / çıkarma | Panel çoklu öğe bağlamını gösterir. |
| Boş alana tık / Esc | Mevcut seçim temizleme ve işlem iptali öncelikleri | Seçim yoksa eski öğenin işlemleri aktif kalmaz. |
| Boş alana sağ tık | Mevcut seçimi gizlice başka öğeye çevirmeme | Hedef yoksa sahte öğe bağlamı üretilmez; mevcut kapanma davranışı korunur. |

**Açılma kararı:** İlk sürümde sağ tık, ⋯ ve uzun basma ortak panelin kesin açılma yollarıdır. Normal / çift tık mevcut çubuğu korur ve açık paneli günceller. Kapalı panelin her normal / çift tıkta otomatik açılması ayrı bir kullanım kararıdır; araç çubuğunun kaldırılması anlamına gelmez ve bu planda kendiliğinden uygulanmaz.

- [ ] Tüm girişler aynı güncel seçim bağlamına bağlansın.
- [ ] Tek tık / çift tık ayrımı, sürükleme eşiği ve kamera hareketi bozulmasın.
- [ ] Panel üzerindeki tık, yazı girişi ve kaydırma sahne seçimini veya kamerayı tetiklemesin.
- [ ] Çoklu öğe seçimi varken sağ tık seçimi tek öğeye düşürmesin; komşuya kayan sağ tık için mevcut koruma taşınsın.
- [ ] Çoklu panel içinde sağ tık / ⋯ toplu yüzey hedefini kaybetmesin. Seçim dışındaki sağ tık davranışı mevcut kuralla karşılaştırılıp ayrıca test edilsin; panel geçişi gerekçesiyle sessizce değiştirilmesin.

### 1.2 Tek öğe / yüzey yetenekleri

- [ ] Panel başlığında gerçek öğe adı, varsa ölçüsü ve seçilen yüzey gösterilsin; öğe ile yüzey birbirine karıştırılmasın.
- [ ] Renk destekleniyorsa renk kartelası / mevcut renk kontrolleri gösterilsin.
- [ ] Görsel destekleniyorsa mevcut arşiv / yükleme, Doldur, Sığdır, Kaldır ve uygun görsel ölçülendirme işlemleri gösterilsin.
- [ ] Cam, Metal Separatör, Lightbox, Mesh ve bunların mevcut dönüşüm / birlikte kullanım kuralları korunsun.
- [ ] Raf altı ışığı, Lightbox ışığı, arka yüze panel, dikme boyu, profil boyu ve boyutlandırma gibi mevcut özel işlemler yalnız geçerli hedefte gösterilsin.
- [ ] TV gibi çok yüzeyli öğelerde gövde yeteneği ekrana, ekran yeteneği bütün ürüne genellenmesin; mevcut yüzey ayrımı gösterilsin.
- [ ] Sabit ölçü bilgi olarak gösterilsin; yalnız mevcut ürün kurallarının değiştirmeye izin verdiği ölçüler düzenlenebilsin. Yeni serbest ölçü veya kaydırıcı kuralı eklenmesin.
- [ ] Item'ın desteklemediği özellik gösterilmesin. Desteklediği ama o anda kullanılamayan işlem, mevcut durumu koruyarak anlaşılır gerekçeyle pasif gösterilsin.

### 1.3 Sağ tık komutlarının eksiksiz taşınması

| İşlem ailesi | Korunacak mevcut işlevler |
|---|---|
| Temel öğe | Kilitle / kilidi aç, döndür, sil; çubuktaki kopyalama aynı mevcut akışı sürdürür |
| Ekle | Mevcut Ekle… kataloğu, tek öğe seçimi, ölçü soruları ve hedefe göre yerleştirme |
| Ürün özel | Mevcut boyutlandırma, görsel ölçülendirme, raf ışığı, arka panel, dikme / profil boyu |
| Yüzey | Cam, Metal Separatör, Lightbox, Mesh ve uygun aydınlatma |
| Çoklu öğe | Seçilenleri kilitle, kilidi aç, sil ve mevcut grup seçenekleri |
| Grup | Oluştur, gruba ekle, adlandır, kilitle, çoğalt, üyeyi çıkar, çöz, seçilen grupları çöz ve grubu sil; mevcut uygunluk koşullarıyla |

- [ ] Uygulama başlangıcında sağ tıkta bulunan bütün action'lar listeye karşı kontrol edilsin; yerel Ekle çalışmasındaki son değişiklikler envantere dahil edilsin.
- [ ] Her panel kontrolü mevcut işlem fonksiyonuna yönlensin. Yeni menü için paralel boyama, silme, ekleme, kilit veya BOM mantığı yazılmasın.
- [ ] Çubuktaki Döndür / Kopyala / Sil düğmeleri doğrudan çalışmaya devam etsin; kullanıcı bu işlemler için panel açmaya zorlanmasın.
- [ ] İşlem kapsamı başlık ve düğme adıyla belli olsun: yüzey işlemi, öğeyi silme ve grubu silme aynı işlem gibi sunulmasın.
- [ ] İlk adımın sonunda sağ tık ve ⋯ eski bağımsız menü yerine ortak paneli açsın. Geçiş sırasında eski menü karşılaştırma için geçici tutulabilir; kalıcı iki farklı işlem sistemi oluşmasın.

### 1.4 Çoklu panel

- [ ] Başlıkta panel sayısı ve gerekiyorsa kaç öğeye ait olduğu gösterilsin.
- [ ] Ctrl/Cmd ile mevcut dikdörtgen seçimi, farklı modüllere yayılan seçim, köşe ve ön / arka yüz kuralları korunsun.
- [ ] Banko gibi yüzleri tek tek seçilen ürünlerde mevcut özel seçim davranışı korunsun; her yüzey dikdörtgen panel gibi ele alınmasın.
- [ ] Parmak taraması / dokunma ile Çoklu panel modu aynı seçim kümesini panele iletsin; parmak kalkınca modun kapanması seçimi silmesin.
- [ ] Ortak özellikler bütün seçimin yeteneklerinden hesaplanarak sunulsun; yalnız son tıklanan panelin yeteneğine göre kontrol açılmasın.
- [ ] Farklı değerlerde ortak alan Karışık olarak gösterilsin; panel açılınca ilk öğenin değeri diğerlerine kendiliğinden uygulanmasın.
- [ ] Her hedefin kabul etmediği toplu işlem gizlensin veya açık gerekçeyle pasif olsun; kullanıcıdan habersiz yalnız bazı yüzeylere uygulanmasın.
- [ ] Görsel / Lightbox / Mesh / Metal Separatör için mevcut aynı düzlem, tam dikdörtgen ve birleşim doğrulamaları aynen kullanılsın. Geçersiz seçimde neden belirtilsin.
- [ ] Birleşik görselin hedefleri ve yüzey kaydı korunsun; kopya gruptaki düzenleme kaynak gruba yazılmasın.

### 1.5 Çoklu modül ve grup

- [ ] Çoklu öğe seçimi ile isimli grup ayrı bağlamlar olarak gösterilsin. Birden fazla öğe seçmek kendiliğinden grup oluşturmasın.
- [ ] Öğenin tamamı seçiliyken Ctrl/Cmd + tık ve Çoklu modül düğmesiyle ekle / çıkar davranışı korunsun.
- [ ] Başlıkta öğe sayısı, varsa grup bilgisi ve kilitli öğe sayısı gösterilsin.
- [ ] Mevcut toplu kilitleme, kilit açma, silme ve grup işlemleri taşınsın; desteklenmeyen toplu döndürme, kopyalama veya boyutlandırma bu çalışma adına icat edilmesin.
- [ ] Seçimde kilitli öğe varsa mevcut atomik toplu silme kuralı korunsun; kilitsiz parçaların bir kısmı sessizce silinmesin.
- [ ] Grup kilidi ve öğenin kendi kilidi ayrımı korunsun. Kilit, mevcut kurallar izin veriyorsa renk / görsel işlemlerini gereksiz yere kapatmasın.
- [ ] Grup üyesi yüzey düzenlemesi ile bütün gruba uygulanacak işlem açıkça ayırt edilsin.
- [ ] Tek öğenin mavi, grup / çoklu öğenin ortak turuncu kesik çerçevesi korunsun. Çoklu panelin yüzey seçimi göstergeleriyle karıştırılmasın.
- [ ] Bekleyen Grup aç / kapat ve grup toplam ölçüleri bu geçişte yapılmış kabul edilmesin; ayrı kapsamda kalsın.

### 1.6 Panelin durumu ve güvenli işlem hedefi

- [ ] Seçim değişimi, kopyalama modu, silme, grup çözme, geri al / ileri al ve proje açma sonrası panel güncel seçimden beslensin.
- [ ] Eski mesh / öğe referansına yazılmasın; gecikmeli görsel yükleme veya ölçü penceresinin hedefi kimlik üzerinden doğrulansın. Hedef yoksa işlem uygulanmasın.
- [ ] Panelin açılması, kapanması veya Basit / Gelişmiş geçişi tasarım verisine ve undo geçmişine yazmasın.
- [ ] Kopyalama sırasında Kopyalamayı bitir kontrolü ve Esc önceliği korunsun. Panelin kapanması kopya yerleştirme hedefini değiştirmesin.
- [ ] Metin alanında Delete, Ctrl+A, Ctrl+Z gibi kısayollar sahneyi etkilemesin; ölçü / görsel pencereleri ve Esc iptali çakışmasın.
- [ ] Item yeteneği, kilit ve etkili kullanıcı yetkisi ayrı değerlendirilsin. Basit / Gelişmiş modu yetkilendirme yerine kullanılmasın.

### 1. Adım kabulü

Normal tık / çift tık araç çubuğu yerinde kalacak; sağ tık, ⋯ ve uzun basma aynı paneli açacak. Tek yüzey, tek öğe, çoklu panel, çoklu modül ve grup bağlamları mevcut işlemlerle çalışmadan bu adım kapanmayacak. Ürün yetenekleri ve toplu işlem uygunluğu ilk adımın parçasıdır; görsel aşamaya ertelenmeyecek.

## 2. Adım — İşlevsel kabul ve geçiş temizliği

- [ ] Eski menü / renk-görsel alanı ile yeni panel arasında işlev karşılaştırması tamamlanıp artık kullanılmayan arayüz ve olay bağları temizlensin; çalışan kontroller kopyalanıp iki ayrı kaynak bırakılmasın.
- [ ] Mevcut katalog Ekle… akışı, 0082/0083 ile yürüyen ürün ailesi ölçü seçimi, desteklenen ekleme hedefleri ve hata bildirimleri korunmuş olsun. Bu migration'lar bu panel işi için yeniden oluşturulmasın.
- [ ] Basit / Gelişmişte aynı seçim bağlamı ve desteklenen işlemler doğrulansın. Görsel hiyerarşi veya hangi kontrolün ayrıntılarda duracağı son tasarımda belirlensin; ürünün yeteneği moda göre değişmesin.
- [ ] Büyük seçimlerde ve sahne sürüklemesinde panel güncellemesi gereksiz sahne kurulumuna veya her karede tam arayüz üretimine neden olmasın.
- [ ] Renk, görsel, özel yüzey, ölçü, silme ve grup işlemlerinin mevcut undo / redo, autosave, revizyon ve kayıt-açma akışlarıyla eşliği doğrulansın.
- [ ] Panel görünürlüğü ve genişliği proje şemasına veya BOM'a eklenmesin. Item master, organization kapsamı ve backend yetkileri değişmesin.

### Test matrisi

| Senaryo | Beklenen kanıt |
|---|---|
| Normal / çift tık | Aynı hedef seçilir; mevcut çubuk ve Çoklu panel / modül ayrımı kalır. |
| Sağ tık / ⋯ / uzun basma | Aynı seçimde aynı işlem listesi ve aynı sonuç; çift tetikleme yok. |
| Renk / görsel almayan ürün | Desteklenmeyen kontrol görünmez; mevcut ürün ve yüzey kuralları kullanılır. |
| Tek yüzey / çok yüzeyli öğe | İşlem doğru yüzeye yazılır; gövde ve ekran karışmaz. |
| Çoklu panel | Geçerli dikdörtgen işlem görür; geçersiz düzlem / birleşim nedeni açıklanır. |
| Karma değer ve yetenek | Karışık değer doğru görünür; panel açmak state değiştirmez, sessiz kısmi uygulama olmaz. |
| Çoklu modül / grup | Seçim sağ tıkla dağılmaz; grup üyeliği, kilit ve atomik silme korunur. |
| Kopya görsel grubu | Kopyanın değişikliği kaynağa yazılmaz. |
| Panel odağı / iki parmak / sürükleme | Panel kullanımı sahneyi yanlışlıkla taşımaz; kamera ve pinch davranışları korunur. |
| Geri al / kayıt-açma / mod değişimi | Tasarım ve desteklenen özellikler kaybolmaz; panel doğru bağlamla güncellenir. |

**Doğrulama:** Ortak bağlam ve action uygunluğu için anlamlı unit testleri; gerçek Chromium'da tek tık, çift tık, sağ tık, çoklu panel, çoklu modül, grup ve dokunmatik E2E. Uygulama deposunun change-contract, tam unit, build ve tam E2E kapıları geçmeden işlevsel kabul verilmez. Yeni panelin insan/AI kullanım açıklaması kılavuzda güncellenir.

İlgili mevcut regresyon başlangıçları: `e2e/double-click-select.spec.mjs`, `e2e/selection-toolbar.spec.mjs`, `e2e/module-group.spec.mjs`, `e2e/module-lock.spec.mjs`, `e2e/color-image-capability.spec.mjs`, `e2e/grouped-image-direction.spec.mjs`, `e2e/panel-sweep.spec.mjs`; dikdörtgen seçim için `test/rectSelection.test.js`. Uygulama başında son dosya / kapsam değişiklikleri tekrar doğrulanacak.

## 3. Adım — Görsel düzenleme

**Ön koşul:** İlk iki adımın işlevsel kabulü tamamlanmış olacak.

- [ ] Panelin konumu ve boyutu gerçek kullanım üzerinden belirlensin; masaüstünde sağ panel ve mobilde alttan açılan panel değerlendirilsin.
- [ ] Seçili öğe / yüzey / grup başlığı, bölüm sırası, boş durum, kilit açıklaması ve Karışık değer sunumu tasarlansın.
- [ ] Renk kartelası, görsel alanı, ölçü seçenekleri, Ekle… ve grup işlemleri ortak bileşenlerle düzenlensin.
- [ ] Basit / Gelişmiş ayrıntı hiyerarşisi son haline getirilsin; mevcut seçim araç çubuğu ve çoklu seçim düğmeleri korunsun.
- [ ] Sahne, seçili öğe, Kaydet, görünüm küpü ve seçim çubuğu örtüşmeleri masaüstü / dar ekran / tablette kontrol edilsin.
- [ ] Klavye odağı, ekran okuyucu etiketleri, kaydırma, dokunma hedefleri ve açık / kapalı panel davranışı doğrulansın.

## Kapsam dışında

Yeni Item / BOM kuralı, yeni ürün özelliği, serbest ölçü sistemi, yeni grup aç/kapat davranışı, grup toplam ölçüleri, otomatik yapı çözme, katalog / sol menüyü baştan düzenleme ve backend yetki modeli değişikliği bu planın kapsamı değildir. Mevcut yeteneği görünür kılmak, o ürün için yeni bir yetenek üretmek anlamına gelmez.

Bu belge yalnız yol haritasıdır. Bu plan hazırlanırken uygulama kodu değiştirilmedi; hiçbir faz uygulanmış veya test edilmiş sayılmıyor.

# Kilit, Geri Alma, Gruplama ve Şablonlar — Yol Haritası

> **Durum:**
>
> | Faz | Durum |
> |---|---|
> | Faz 1 — Kilit | Uygulandı: `kyrox-fair-stand` PR #278, CI yeşil, `main`'de |
> | Faz 2 — Geri al / İleri al | Uygulandı: `kyrox-fair-stand` PR #281, CI yeşil, `main`'de |
> | Faz 3 — Grup (öğeden bağımsız) | Uygulandı: `kyrox-fair-stand` PR #284 |
> | Faz 3b — Grup aç / kapat (3ds Max gibi) | Açık karar, kod yok |
> | Faz 4 — Çok duvarlı grup ve grup döndürme | Uygulandı: çok duvarlı grup Faz 3 ile, grup döndürme `kyrox-fair-stand` PR #285 |
> | Faz 5 — Kurumsal şablon | Uygulandı: `kyrox-fair-stand` PR #286 (migration `0067`, API + Şablonlarım kartı) |
> | Faz 6 | İlk iki madde Faz 3 ve Faz 4 ile karşılandı; opsiyonel maddeler plan |
>
> **Grup işi kapandı (2026-10-18):** Faz 1–5 uygulandı ve yayında. Faz 3b (grup aç / kapat) ve Faz 6 opsiyonel maddeleri (şablon görselleri, rafın duvarı takip etmesi) yeni bir karar verilene kadar açık bekler.
>
> Her faz ayrı PR'dır ve kodlamadan önce kullanıcı onayı alınır (`kyrox-fair-stand/AGENTS.md` kod öncesi checkpoint).
>
> Belgenin yeri: Fair Stand plan belgeleri `kyrox-platform/projects/fair-stand/` altında tutulur (`AGENTS.md` sert kural). Aşağıdaki dosya yolları `kyrox-fair-stand` deposuna göredir.
>
> Her faz `SYSTEM_CHANGE_GATE.md` + `SYSTEM_IMPACT_SWEEP.md` zincirinden geçer: `.github/change-contract.json` beyanı → `npm run contract:verify` → targeted regression → `npm test` → `python -m pytest` → `npm run build` → `npm run e2e`.
>
> Kilit, grup ve şablon **fiziksel ürün değildir**: BOM'a, üretime ve maliyete kendileri girmez, `itemKey` taşımaz (`ITEM_CONTRACT.md` kapsamı dışı; açık ürün kararı). Şablondan sahneye konan parçalar normal Item'dır ve BOM'a bugünkü gibi girer.

## 1. Neden

| # | Durum | Örnek | Çözüm | Faz |
|---|---|---|---|---|
| A | Yanlışlıkla kaydırıp yapıyı bozmak | Depo (kapı dahil 5 duvar parçası), duvara konan TV, yere konan box | **Kilit** | 1 |
| B | Yapılan bir hatayı düzeltmek | Yanlış taşıma, yanlış silme, yanlış renk | **Geri al / İleri al** | 2 |
| C | Bağlı parçaların ayrı ayrı oynaması | `wall_200_350` + 3 raf + baza; duvar + lambalar; masa + 3 sandalye | **Grup** (çapalı) | 3 |
| D | Grubun da yanlışlıkla oynamaması | Kurulmuş depo grubu | **Grup kilidi** | 3 |
| E | Grubun içinde ne olduğunu görmek | "Duvar raflı grup" → hangi parçalar? | **Grup listesi paneli** | 3 |
| F | Kurulu grubu bir daha kullanmak | Masa + 3 sandalyeyi çoğaltmak | **Grup çoğaltma** | 3 |
| G | Bir yapıyı bütün olarak taşımak | Depoyu standın başka yerine almak | **Çok duvarlı grup** | 4 |
| H | Sık kullanılan seti her projede hazır bulmak | Organizasyonun "raflı TV'li duvar" seti | **Kurumsal şablon** | 5 |

## 2. Bugünkü durum (koda göre)

- **Seçim tek öğelik.** `selectedModuleId` tek değer (`src/scene3d.js`). Ctrl/Cmd+tık öğe değil **panel yüzeyi** seçer (görsel/renk için; `selectRectangleTo`). Öğeler için çoklu seçim yok.
- **Taşıma tek öğelik.** `dragSession` tek `moduleState` taşır. Bırakınca yalnız plan içindeki yerleşimler yazılır (`finishPlacementDrag`). Duvar zincirinde `planContinuousModuleMove` komşuları kaydırabilir (`src/moduleMove.js`, `wallReflow.js`). Serbest öğe `validatePlacementAgainstModules` ile doğrulanır (`src/modulePlacement.js`). Ok tuşları seçili tek öğeyi taşır.
- **Raf duvara bağlı değil.** Raf bırakılırken duvar bulunup panel birleşimine oturtulur (`snapPanelSeamOverlayPlacement`). Ama yalnız raf kendi mutlak `placement`'ını saklar; "hangi duvara ait" alanı yoktur. Duvar taşınınca raf yerinde kalır.
- **Yerleşim türleri** (canlı `fair_stand_item_type_scene_behavior`):
  - `flat-panel`, `door`, `panel`, `profile`, `connector` → `wall`
  - `shelf`, `tv`, `illuminated-foam` → `wall-overlay`
  - `led-floodlight` → `top`
  - `base`, `box-block`, `counter`, `chair`, `table-*`, `sofa-*`, bitkiler → `free`
- **Duvar yerleşimi iki türlü.** Kenar zinciri (`wallId`: `back` / `left` / `right`; taşıma komşuları kaydırır) ve serbest sıra (`wallId: 'free'`; standın içindeki duvarlar, zincir hesabına girmez).
- **Admin önizlemedeki kilit grubu.** Ürün 3D önizlemesinde (Admin) parçalar `lock_group_id` ile katı grup olarak kilitlenir; "Kilitle", "Gruba ekle", "Gruptan çıkar / Kilidi aç" komutları vardır (`src/itemAdminAssemblyLock.js`, `docs/refactor/ITEM_3D_PREVIEW.md`). Bu **Item montajı** içindir, sahne/proje kilidi değildir; ikisi karıştırılmaz. Katı grup hareket mantığı Faz 3'te yeniden kullanılmak üzere incelenir. Kullanıcıya görünen adlar tutarlı seçilir.
- **Benzer mevcut kavram.** Lightbox/Mesh kumaşı (`fabricGroupId`) birden fazla modülü bağlar ve onları **taşımayı engeller** (`getFabricMoveLock`). Bu kuralı bozmayız.
- **Proje state'i.**
  - Sahne `main.js` içindeki `currentModules` dizisinden kurulur (`rebuildWall` → `scene3d.buildWall`). Stand kurulumu `rebuildSceneFromSetup` ile, proje yükleme `restoreProject` ile yapılır.
  - Anlık görüntü `buildProjectSnapshot()` ile alınır. Otomatik kayıt bu görüntünün imzasını izler (`getProjectStateSignature`, `src/autosaveController.js`).
  - Proje biçimi `{ id, version: 1, modules, stand }` (`src/projectImportValidation.js`). `normalizeModuleItemState` modülü yerinde düzeltir, bilinmeyen alanı silmez.
- **Çoğaltma.** `duplicateModuleState` modülü JSON olarak **bütünüyle kopyalar**, yeni kimlik verir (`src/designState.js`). Duvar zincirinde `duplicateContextModule` yanına ekleme planı kullanır (`src/main.js`).
- **Görseller proje başınadır.** `fair_stand_project_assets` (`organization_id` + `project_id`).
- **Organizasyon.** Projeler, proje görselleri ve maliyet kalemleri organizasyona bağlı (`organization_id`). Item kataloğu (`fair_stand_items`) **ortak**, organizasyona ait değil.
- **Geri alma (Ctrl+Z) yok.**

## 3. Kavramlar

- **Toplu seçim** — Shift+tık ile birden fazla öğe seçmek. Kalıcı değildir; toplu kilitleme ve grup kurmanın ön adımıdır. Ctrl, panel yüzey seçimi olarak kalır.
- **Kilit** — öğenin bulunduğu yere sabitlenmesi. Kilitli öğe taşınmaz, döndürülmez, silinmez. Görünüm (renk, görsel, TV içeriği) değiştirilebilir.
- **Geri al / İleri al** — proje tasarımındaki her değişikliğin adım adım geri alınması ve yinelenmesi.
- **Grup** — öğeden bağımsız, isimli, kalıcı öğe kümesi. Her türlü öğe girer; grup bütün olarak aynı kaydırmayla taşınır ve çoğaltılır. (Faz 3 uygulamasında eski "çapa" tanımı kullanıcı kararıyla kaldırıldı.)
- **Grup kilidi** — grubun bütün olarak kilitlenmesi.
- **Kurumsal şablon** — bir grubun organizasyon kataloğuna kaydedilmiş hali. Sahneye konunca **normal parçalara ve yeni bir gruba açılır**; şablon bir Item değildir.

## 4. Fazlar

### Faz 1 — Kilit (A)

> **Durum: uygulandı** (`kyrox-fair-stand` PR #278).
> - Kaynak `src/moduleLock.js`.
> - Testler `test/moduleLock.test.js` ve `e2e/module-lock.spec.mjs`.
> - Uygulamada verilen kararlar:
>   - Kilit Sil ve Boyutlandır'ı pasifleştirir.
>   - "Tüm özellikleri kaldır" kilidi korur ve kilitli Işıklı Straforu silmez.
>   - Sahnede kilit simgesi yok; kilit menü başlığında ve seçim bilgisinde "Kilitli" olarak görünür.
>   - Stand kurulumu "Yeni proje oluşturulacak" akışı olduğu ve eski kayıtlı projeyi koruduğu için ek kilit uyarısı eklenmedi.

**Amaç.** Depo, TV, box gibi yerleşmiş parçaların yanlışlıkla bozulmasını bitirmek.

**Davranış**
- Sağ tık → **Kilitle** / **Kilidi aç**.
- Shift+tık ile toplu seçim → **Seçilenleri kilitle / kilidini aç**.
- Kilitli öğe:
  - Sürüklenmez. Sürüklenmeye çalışılırsa "Kilitli — taşımak için önce kilidi açın" uyarısı çıkar; kamera dönüşü normal çalışır.
  - Ok tuşları ve döndürme kısayolu çalışmaz.
  - Silinemez: Sil tuşu ve menü uyarı verir.
  - Seçilebilir; renk, görsel ve TV içeriği değiştirilebilir.
- **Dolaylı hareketi de engeller.** Duvar zincirinde ekleme, çoğaltma ya da başka öğenin taşınması kilitli bir öğeyi kaydıracaksa işlem reddedilir ve hangi kilitli öğenin engellediği yazılır. Kontrol, plan (`placements`) uygulanmadan önceki ortak noktada yapılır.
- Seçiliyken menüde ve durum çubuğunda "Kilitli" görünür. Sahnede kilit simgesi gösterilip gösterilmeyeceği uygulama sırasında karar verilir.

**Veri.** Modül state'ine `locked: true` (yoksa kilitsiz). Proje kaydı, yeniden yükleme, ZIP dışa/içe aktarma ve revizyonlarda korunur. Eski projeler değişmeden açılır. `duplicateModuleState` her şeyi kopyaladığı için çoğaltmada `locked` açıkça temizlenir; kopya kilitsiz doğar.

**Açık kontroller.** Otomatik duvar / otomatik depo / "sahneyi sıfırla" kilitli öğeyle karşılaşınca ne yapar? Sunucu payload'ı alanı aynen saklıyor mu?

**Dokunulacak yüzeyler.**
- `src/scene3d.js`: sürükleme, ok ve döndürme girişleri; toplu seçim.
- `src/moduleContextMenu.js`.
- `src/main.js`: silme, çoğaltma, ekleme yolları.
- Plan uygulayan yerleşim yolları.
- `src/designState.js`: çoğaltmada kilidin temizlenmesi.
- `src/helpGuide.js`.
- Yeni `src/moduleLock.js` ve onun `systemChangeContract.js` path eşlemesi.

Kesin liste `contract:verify` discovery'si ile çıkarılır.

**Testler.**
- Unit: taşıma, döndürme ve silme reddi; reflow planı kilitli öğeyi kaydırıyorsa ret; çoğaltma kilitsiz; kayıt round-trip.
- E2E (`e2e/module-lock.spec.mjs`): kilitle → sürükle → yerinde → yenile → hâlâ kilitli → kilidi aç → taşınabilir.

**Kabul.** Kilitli öğe hiçbir yolla (sürükleme, ok, döndürme, sil, zincir kaydırması) yer değiştirmez; kilit kayıttan sonra korunur; CI zinciri yeşil.

### Faz 2 — Geri al / İleri al (B)

> **Durum: uygulandı** (`kyrox-fair-stand` PR #281).
> - Kaynak `src/designHistory.js`.
> - Testler `test/designHistory.test.js` ve `e2e/design-undo.spec.mjs`.
> - Uygulamada verilen kararlar:
>   - Değişiklik, tasarım imzası (stand + modüller) ile algılanır: `pointerup` / `keyup` / `change` sonrası ve saniyede bir.
>   - Ctrl/Cmd+Z, Ctrl/Cmd+Y, Ctrl/Cmd+Shift+Z; üst bantta iki düğme.
>   - Son 50 adım; geçmiş kaydedilmez.
>   - Görsel kütüphanesinden görsel silinince geçmiş sıfırlanır.
>   - Geçmiş revizyon görünümünde kapalı.
>   - Stand ölçüsü ve tipi proje içinde değişmediği için geri yükleme yalnız modülleri ve zemini (tip, renk, zemin alanı) yeniden çizer.

**Amaç.** Tasarımda yapılan her değişikliği geri alabilmek. Gruplamadan önce gelir; grup işlerinde yapılan hata da geri alınabilsin.

**Davranış**
- **Ctrl+Z** geri al; **Ctrl+Y** ve **Ctrl+Shift+Z** ileri al. Araç çubuğunda iki düğme bulunur, geri alınacak adım yoksa pasiftir.
- Geri alınabilen işler: taşıma, döndürme, ekleme, silme, çoğaltma, renk, görsel yerleşimi, Lightbox kumaşı, kilit, grup işlemleri, zemin alanı, stand değişiklikleri.
- Her bırakma ya da menü komutu **tek adımdır**. Sürükleme sırasındaki ara konumlar adım sayılmaz.
- Metin kutusunda yazarken Ctrl+Z tarayıcının kendi metin geri almasıdır; tasarımı etkilemez.

**Teknik yaklaşım: anlık görüntü yığını**
- Değişiklikten önceki tasarım (`stand`, `modules`, ileride `groups`) `buildProjectSnapshot` temelinde kopyalanıp yığına eklenir.
- Geri almada o görüntü `currentModules` ve stand'e yazılır, sahne mevcut yollarla (`rebuildWall`, gerekirse `rebuildSceneFromSetup`) yeniden kurulur.
- Değişiklik tespiti, otomatik kaydın kullandığı imza mantığıyla yapılır: kullanıcı eylemi bittiğinde imza değiştiyse önceki görüntü yığına girer.
- Sınır: son 50 adım (uygulamada ölçülüp kesinleşir).
- Geçmiş **kaydedilmez**. Proje değişince, revizyon açılınca ve sayfa yenilenince sıfırlanır.

**Riskler / kararlar**
- **Görsel silme.** Görsel yöneticisinden bir görselin silinmesi proje varlığını siler (`deleteProjectAsset`). Eski bir adımda o görsel kullanılıyorsa geri alındığında görsel boş görünür. Öneri: görsel silme geri alma kapsamı dışında, ayrı ve onaylı bir işlem olarak kalır.
- **Otomatik kayıt.** Geri alma da bir değişikliktir; otomatik kayıt onu normal şekilde kaydeder.
- **Bellek.** Her adım tasarımın JSON kopyasıdır. Proje boyutuna göre ölçülür; gerekirse sınır düşürülür.

**Dokunulacak yüzeyler.** Yeni `src/designHistory.js` (saf yığın: ekle / geri / ileri / sınır / sıfırla; testli), `src/main.js` (yakalama noktaları, geri yükleme, araç çubuğu), `src/viewKeyboardShortcuts.js` (kısayollar), `index.html` (düğmeler), `src/helpGuide.js`.

**Testler.**
- Unit: yığın davranışı; sınır; yeni değişiklikte ileri dalın silinmesi.
- E2E (`e2e/design-undo.spec.mjs`): taşı → Ctrl+Z → eski yerinde → Ctrl+Y → yeni yerinde; sil → Ctrl+Z → geri geldi.

**Kabul.** Listelenen bütün işler tek tek geri alınır ve yinelenir; proje değişince geçmiş sıfırlanır; CI zinciri yeşil.

### Faz 3 — Grup: öğeden bağımsız küme, birlikte taşıma, liste paneli, çoğaltma (C, D, E, F)

> **Durum: uygulandı** (`kyrox-fair-stand` PR #284). İlk plandaki "tek çapa / grupta en fazla 1 duvar" tanımı, uygulamadan önce kullanıcı kararıyla aşağıdaki öğeden bağımsız tanımla değiştirildi.

**Amaç.** İstenen öğeler (duvar paneli, profil, lamba, logo, TV, görselli kutu blok, masa, sandalye, banko…) tek bir isimli küme olsun; birlikte taşınsın, çoğaltılsın ve kilitlenip düzeni korunsun.

**Kararlar (kullanıcı, 2026-10-18)**
- Grup öğeden bağımsızdır; her türlü öğe ve karışık küme gruplanır. Duvar sınırı yoktur.
- Gruplanınca stand kenarına bağlı (`back` / `left` / `right`) üyeler serbest öğeye çevrilir; konum ve açı aynı kalır, eski kenar `fromWallId` olarak hatırlanır. Grup çözülünce ya da üye çıkarılınca öğe hâlâ aynı kenar çizgisinde ve aynı açıdaysa kenara geri bağlanır.
- Grup bütün olarak her yöne taşınır (sürükleme, ok tuşu; 10 cm adım). **Tek kısıt:** hiçbir üye aktif stand alanının dışına taşamaz. Başka öğelerle çakışma ve tür kuralları (ör. TV'nin duvarda durması) aranmaz; zincir itme yoktur. Stand dışına çıkan adım uygulanmaz, grup kenarda durur.
- İçinde ne olursa olsun her grup çoğaltılabilir. Kopya "… (kopya)" adıyla yeni, kilitsiz bir gruptur; grubun sağına, soluna, önüne ya da arkasına ilk boşluğa konur, yer yoksa işlem reddedilir.
- Grubu silmek onaylıdır: "Grubun tamamı silinecek (N parça)". Delete tuşu yalnız seçili parçayı siler.
- Tık bugünkü gibi panel seçer (boyama bozulmaz); grubun diğer üyelerinde çerçeve görünür. Alt+tık yok; grup işlemleri sağ tık menüsünde ve Gruplar kartında.
- Bir öğe aynı anda tek grupta olur. Başka gruptaki öğe eklenmek istenirse sorulur: "… grubundan çıkarılıp … grubuna eklensin mi?" Hayır → hiçbir şey değişmez.
- Grupta 1 parça kalırsa grup kendiliğinden çözülür.
- Gruptaki öğede Shift+R (döndürme) reddedilir; grup döndürme sonraki faza kalır.

**Grup kilidi.** Grup kilitliyse bütün üyeler Faz 1 kurallarıyla kilitli sayılır (taşınmaz, döndürülmez, boyutlanmaz, silinmez). Kilitli gruptaki tek parçanın kilidi açılamaz. Kilitliyken adlandırma ve çoğaltma yapılabilir; çıkarma, çözme ve silme yapılamaz. Kendi kilidi olan bir üye grubu taşıtmaz.

**Veri.** Her üyede `group: { id, name, locked, fromWallId? }`. Sunucu proje payload'u yalnız `{stand, modules}` tuttuğu için ayrı bir `groups` alanı kullanılmadı; şema, migration ve proje sürümü değişmez. Kayıt, revizyon, içe/dışa aktarma ve geri al bu alanı kendiliğinden kapsar. İçe aktarma alanın biçimini ve gruplar arası tutarlılığı doğrular. Tek öğe kopyası gruba girmez; "Tüm özellikleri kaldır" grubu ve öğe kilidini korur.

**Bilinçli sonuç.** Kenardan ayrılan duvar parçaları duvar kapasitesi hesabına ve otomatik duvar lambası dizisine girmez.

**Dokunulan yüzeyler.** Yeni `src/moduleGroups.js` (saf mantık) ve `src/moduleGroupPanel.js` (Gruplar kartı); `src/moduleLock.js` (`isModuleLocked` grup kilidini sayar); `src/scene3d.js` (grup sürükleme, ok tuşu, Shift+R reddi, üye çerçevesi, `selectModulesById`); `src/moduleContextMenu.js` (grup bölümü); `src/main.js`; `src/designState.js`; `src/projectImportValidation.js`; `src/featureContracts.js` (`module-group`); `index.html` ve `src/configuratorMarkup.js`; `src/helpGuide.js`.

**Testler.** Unit `test/moduleGroups.test.js` ve `test/moduleGroupPanel.test.js`; E2E `e2e/module-group.spec.mjs`: iki kutu blok gruplanır, kartta 2 parça görünür, birlikte taşınır, stand dışına taşmaz, kilitliyken oynamaz, çoğaltılır, onaylı silinir, çözülür.

**Seçim rengi.** Shift+tık toplu seçim ve grup üyesi çerçevesi Kyrox bakırıdır (`#C47A3A`); tek öğe ve panel seçimi mavi kalır.

### Faz 3b — Grup aç / kapat (açık karar, kullanıcı notu 2026-10-18)

Faz 3 sonrası kullanıcının sorduğu, henüz karara bağlanmamış iki konu. 3ds Max'teki **Group > Open / Close** davranışı örnek alınır.

**1. Gruptaki bir öğeye tıklanınca bütün grup mu seçilsin?**
- **Bugün (Faz 3):** Tık yalnız tıklanan paneli seçer; grubun diğer üyeleri bakır çerçeveyle gösterilir. Sürükleme ve ok tuşu bütün grubu taşır.
- **Seçenek:** Tık bütün grubu seçer (3ds Max'te kapalı grup tek nesne gibi seçilir). Gruptaki tek öğeye ulaşmak için grup **açılır**.

**2. Gruptaki öğenin özellikleri (renk, görsel, cam, kumaş, ölçü) değiştirilebilsin mi?**
- **Bugün (Faz 3):** Değiştirilebiliyor; grup yalnız taşıma, kilit, çoğaltma ve silmeyi birlikte yapıyor.
- **Seçenek (3ds Max gibi):** Kapalı grupta öğe düzenlenemez. Sağ tık veya Gruplar kartından **Grubu aç** denir; grup açıkken öğeler tek tek seçilir, düzenlenir ve taşınır. **Grubu kapat** ile grup yeniden tek parça gibi davranır. Açık grup kesik bakır çerçeveyle gösterilir; aynı anda tek grup açık olur.

**Karar verilmesi gerekenler:** (a) Tıkın varsayılanı (panel mi, bütün grup mu); (b) kapalı grupta görünüm değişikliğine (renk/görsel) izin verilip verilmeyeceği; (c) açık grupta tek öğe taşımanın diğer üyelerle göreli konumu bozmasına izin verilip verilmeyeceği; (d) aç/kapat durumunun kaydedilip kaydedilmeyeceği (öneri: kaydedilmez, proje açılınca gruplar kapalı başlar); (e) kilitli grubun açılıp açılamayacağı (öneri: açılamaz, önce kilit açılır).

### Faz 4 — Çok duvarlı gruplar: serbest sıra (G)

> **Durum:** Faz 3'ün öğeden bağımsız tanımıyla büyük ölçüde karşılandı: grupta duvar sınırı yok, gruplanınca kenar parçaları serbest öğe olur, grup katı cisim gibi taşınır ve çoğaltılır. Açık kalan: grubu toptan döndürme.

**Amaç.** Depo gibi standın içinde duran, birden fazla duvar parçasından (kapı dahil) kurulu yapıyı bütün olarak taşımak ve çoğaltmak.

**Kapsam.** Gruptaki bütün duvar parçaları serbest sırada (`wallId: 'free'`) ise birden fazla duvar parçasına izin verilir. Grup katı cisim gibi taşınır: aynı öteleme, üyeler arası bağlantı korunur. Doğrulama her üye için grup dışı öğelere karşı yapılır.

**Ön koşul.** Kullanıcının gerçek depo kaydında duvar parçalarının `wallId` değerleri incelenir. Kenar zincirine dokunan parça varsa o grup Faz 6'ya kalır.

**Testler.** Unit (çok duvarlı katı taşıma, dış çakışmada ret) ve E2E (5 parçalı depo grubunu taşı; parçalar arası mesafe değişmez).

### Faz 5 — Kurumsal şablonlar (H)

> **Durum:** Uygulandı (`kyrox-fair-stand`). Migration `0067_group_templates`, API `/api/v1/fair-stand/group-templates`, kenar çubuğu **Şablonlarım**, Gruplar kartında **Şablon olarak kaydet**.

**Amaç.** Bir grubu organizasyonun kendi kataloğuna kaydedip başka projelerde tek hamlede kullanmak.

**Neden Item değil, şablon**
- Item kataloğu (`fair_stand_items`) ortaktır, organizasyona ait değildir.
- Bir Item'ın BOM reçetesi, birimi ve davranış ailesi olmalıdır (`ITEM_CONTRACT.md`). Grubu Item yapmak BOM'u ikiye katlama ve sözleşme dışı paralel model riski taşır.
- **Şablon** ise "hangi parçalar, nasıl dizilmiş, hangi yüzey ataması" bilgisini saklar. Sahneye konunca normal parçalara açılır. Her parça zaten bir Item olduğu için BOM ve maliyet bugünkü gibi doğru çıkar.

**Uygulanan davranış (kullanıcı kararları, 2026-10-18)**
- Gruplar kartı → **Şablon olarak kaydet** → isim.
- Kenar çubuğunda **Şablonlarım**: **Yerleştir** / **Adlandır** / **Sil**. Güncelleme yok; silip yeniden kaydedilir.
- Yerleştirince parçalar yeni kimliklerle eklenir, şablon adıyla **yeni kilitsiz grup** kurulur; stand ortasına (yer yoksa ilk boş yere) konur, sonra sürüklenir.
- Katalogda olmayan `itemKey` atlanır ve yazılır.
- Yetkiler proje yetkileridir (read / create|update / delete). Oturum yoksa (CRM dışı) şablonlar kapalıdır.
- **Korunan:** ölçü, göreli konum/açı, renk, cam panel, metal separatör.
- **Korunmayan (5a):** görseller, Lightbox, Mesh. (Opsiyonel 5b: şablon görsellerinin taşınması — yapılmadı.)

**Veri (sunucu)**
- Tablo `fair_stand_group_templates`: `id`, `organization_id`, `name`, `member_count`, `payload` (JSON), `created_by`, tarihler. `description` kolonu yok.
- `organization_id` yalnız auth context’ten; başka org 404.
- Canonical envanter: `docs/refactor/DATABASE.md`, `docs/refactor/FAIR_STAND_DB_KULLANIM_KILAVUZU.md` §5.22.

**Kod yüzeyleri.**
- Backend: `0067_group_templates.py`, `group_templates.py`, `group_template_routes.py`, pytest.
- Frontend: `src/groupTemplates.js`, `groupTemplateRemote.js`, `groupTemplatePanel.js`, `main.js`, `moduleGroupPanel.js`.
- Test: `test/groupTemplates.test.js`, `e2e/group-template.spec.mjs`.

### Faz 6 — Gelişmiş

- ~~Kenar duvar zincirine (`back` / `left` / `right`) bağlı birden fazla duvar parçasının grup olarak taşınması.~~ **Faz 3 ile karşılandı:** gruplanınca kenar parçaları serbest öğeye çevrilir ve grup her yöne taşınır. Fark: zincir/reflow ile komşu paneller itilmez (kullanıcı kararı).
- ~~Grubu döndürme.~~ **Faz 4 ile uygulandı** (`kyrox-fair-stand` PR #285). Kararlar (kullanıcı, 2026-10-18):
  - Gruptaki öğe seçiliyken yalnız **Shift+R**; sağ tık ve Gruplar kartında düğme yok.
  - Adım **45°**; yön tek öğe Shift+R ile aynı. Gruptaki 90° adımlı duvar, kapı ve panel de 45° açıya geçebilir.
  - Merkez üye merkezlerinin ortalamasıdır; dönüşte sabit kaldığı için art arda döndürmede grup kaymaz (8 × 45° başa döner).
  - Her üyenin konumu ve açısı birlikte döner. Tek kısıt stand alanı: sığmazsa işlem yapılmaz. Kilitli grup ya da kendi kilidi olan üye döndürmeyi engeller.
- (Opsiyonel, ayrı karar) Rafların gruplamadan bağımsız olarak bağlı oldukları duvarı her zaman takip etmesi: raf state'ine kalıcı "ev sahibi duvar" bağı. Duvar ve raf aynı gruptaysa Faz 3 ile zaten birlikte taşınır.
- (Opsiyonel) Şablon görsellerinin şablonla birlikte taşınması (5b).

## 5. Kararlar

| Konu | Karar | Durum |
|---|---|---|
| Faz sırası | Kilit → Geri al → Grup → Çok duvarlı grup → Şablon → Gelişmiş | Öneri |
| Toplu seçim tuşu | Shift+tık (Ctrl panel seçiminde kalır) | Karar (Faz 1) |
| Gruptan tek öğe seçmek | Alt+tık yok; tık panel seçer, grup işlemleri sağ tıkta ve Gruplar kartında | Karar (Faz 3) |
| Kilitli öğe silinebilir mi | Hayır, önce kilit açılır | Karar (Faz 1) |
| Grup seçiliyken Sil | Grubu sil komutu bütün grubu onayla siler; Delete tuşu yalnız seçili parçayı | Karar (Faz 3) |
| Çoğaltılan öğe / grup | Kilitsiz doğar; grup kopyası yeni, kilitsiz gruptur; içinde ne olursa olsun çoğaltılır | Karar (Faz 1, Faz 3) |
| Grup kapsamı (Faz 3) | Öğeden bağımsız; duvar sınırı yok; kenar parçaları serbest öğeye çevrilir | Karar (Faz 3) |
| Grup taşıma kısıtı | Yalnız aktif stand alanı; çakışma ve tür kuralları aranmaz | Karar (Faz 3) |
| Öğe birden fazla grupta | Hayır; başka gruba eklemek sorularak taşır | Karar (Faz 3) |
| Geri alma geçmişi | Kaydedilmez, son 50 adım | Karar (Faz 2) |
| Görsel silme geri alınır mı | Hayır; görsel silinince geçmiş sıfırlanır | Karar (Faz 2) |
| Şablon Item mı | Hayır, sahnede normal parçalara açılan şablon | Karar (Faz 5) |
| Şablonda renk / cam / metal | Korunur | Karar (Faz 5) |
| Şablonda görseller / Lightbox / Mesh | 5a: yok; 5b opsiyonel | Karar (Faz 5) / 5b açık |
| Kilit / grup / şablon BOM'a girer mi | Hayır (parçalar girer) | Ürün kararı |

## 6. Riskler

- `src/scene3d.js` çok büyük (~350 KB). Seçim ve sürükleme değişiklikleri mevcut kaynak-şekli testlerine dokunabilir (`test/moduleSelectionState.test.js`, `test/moduleSelectionVisual.test.js`, `test/ctrlMultiSelect.test.js`, `test/scene3dPublicContract.test.js`). Discovery çıktısına göre gözden geçirilir.
- Kilit kontrolü yalnız sürüklemeye konursa zincir kaydırması, ekleme, çoğaltma ya da otomatik planlayıcılar kilidi delebilir. Kontrol, plan uygulanmadan önceki ortak noktaya konmalıdır.
- **Shift tuşu çakışması.** Kamerayı yöneten hazır Three.js kontrolünde Shift + sol tıkla boş alanı sürüklemek kamerayı döndürmek yerine kaydırır (pan). Bu varsayılan davranıştır ve uygulamadan önce bu projede doğrulanacaktır. Çoklu seçim de Shift+tık kullanacağı için kural şudur:
  - **Shift+tık bir parçanın üstündeyse:** Parça seçime eklenir ya da çıkarılır; kamera kımıldamaz. Tıklama kamera kontrolünden önce yakalanır; bugün Ctrl+tık için yapılanın aynısıdır (`test/ctrlMultiSelect.test.js`).
  - **Shift+tık / sürükleme boş alandaysa:** Bugünkü kamera davranışı aynen korunur.
- Geri alma yakalama noktası kaçırılırsa bir işlem geri alınamaz; değişiklik tespiti tek noktadan (imza) yapılmalıdır.
- Şablonlar organizasyonlar arası veri sızıntısına açık bir yüzeydir; `organization_id` izolasyonu backend testleriyle korunur.

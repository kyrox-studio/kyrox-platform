# FAIR CRM Yol Haritası

**FAIR CRM'in tek iş listesi.** Açık, ertelenmiş veya karar bekleyen her iş burada bir satırdır. Bugünkü durum: [PROJECT_STATUS.md](PROJECT_STATUS.md); tamamlanan işlerin geçmişi: [CHANGELOG.md](CHANGELOG.md). Satırların durumu 2026-10-28'de koda göre doğrulandı (FAIR CRM `main` `122fdf9f`).

Kurallar:

- Yeni iş önce buraya satır olarak eklenir. Biten iş buradan silinir; [PROJECT_STATUS.md](PROJECT_STATUS.md) ve [CHANGELOG.md](CHANGELOG.md)'ye yazılır.
- Ayrıntı belgeleri tasarımı ve kabul ölçütünü tutar; kendi iş listesi tutmaz.
- Ekosistem SaaS sıralaması [SAAS_ROADMAP.md](../../ecosystem/SAAS_ROADMAP.md) içindedir ve ikinci bir CRM listesi değildir. Somut bir SaaS işi CRM'de aktif olunca kapsam ve sahiplik onayıyla buraya satır olarak alınır. Her esaslı CRM değişikliği ortak SaaS-impact, tenant-isolation, yetki kapsamı ve runtime kabul kapılarından geçer.
- Güvenlik bulguları yalnız [SECURITY_PROGRAM.md](../../standards/security/SECURITY_PROGRAM.md) içindedir.
- Eski sprint numarasından ya da bayat durum metninden yeni sıra çıkarılmaz; sıra yalnız açık ürün kararıyla burada değişir.

## 1. Ürün kalitesi

| İş | Durum | Ayrıntı |
|---|---|---|
| Yetkiye bağlı arayüz tutarlılığı | Aktif. Navigasyon, doğrudan rota, liste araç çubuğu, satır/detay işlemleri, oluştur/düzenle/sil ve gönder/çalıştır/geri yükle gibi CRUD dışı işlemler etkili yetkiye göre gösterilir. Arayüzde gizlemek güvenlik değildir; iş, arayüz ile backend reddi aynı sonucu verince biter | [CRUD_UI_AUTHORIZATION_STANDARD.md](../../standards/ui/CRUD_UI_AUTHORIZATION_STANDARD.md) |
| Teklif ve maliyet kataloğu belge eşitlemesi | Kod Temmuz belgelerinin ilerisinde. Mevcut kod, migration, yetki, test ve UI/API durumu [PROJECT_STATUS.md](PROJECT_STATUS.md)'ye yazılır; bayat plan notlarından yeniden tasarlanmaz | — |

## 2. Planlanan ürün davranışı

| İş | Durum | Ayrıntı |
|---|---|---|
| Uygulama içi bildirim merkezi | Faz 1 uygulaması ve yerel doğrulama tamamlandı. Zamanlanmış görev sonuçları ve ileri tarihli takip hatırlatmaları açık. | [Bildirim merkezi](#uygulama-içi-bildirim-merkezi) |
| Operasyon: duraklat ve iptal ayrımı | `RunStatus.PAUSED` tanımlı ama duraklatma / sürdürme use case'i ve uç noktası yok; "Durdur" iptal eder. Hedef: "Durdur" `running → paused`, "Devam Ettir" kaldığı yerden sürdürür (ilerleme, log ve ara durum korunur), ayrı "İptal Et" `cancelled` yapar. Backend yaşam döngüsü, handler yeteneği, UI, yetki ve gerçek runtime kabulü birlikte | — |
| Müşteriyi pasife alma | Yok. Neden (seçilir veya yazılır), not ve müşteri aktivite kaydı; gönderim öncesi kontrol "Customer pasif mi?" adımını da içerir. Müşteri/Contact e-posta izni, MailerSend unsubscribe/spam webhook'u ve merkezi `EmailConsentPolicy` mevcut | [archive: EMAIL_COMMUNICATION_PREFERENCES](../../archive/fair-crm/backlog/EMAIL_COMMUNICATION_PREFERENCES.md) |
| Görevler Faz 3 | Faz 2 dışında bırakılanlar: kişi / lead / katılımcı bazlı çalışma listesi, satır bazlı kullanıcı atama, otomatik görev oluşturma ve auto-reopen, `follow_up_at` üzerinden görev/bildirim/takvim, Excel export, AI not özeti, kampanya otomasyonu, otomatik arama entegrasyonu | [WORKLIST_PHASE2.md](todo/WORKLIST_PHASE2.md) § 7 |
| WhatsApp ve SMS gönderimi | Kodda gönderim yok (WhatsApp yalnız aktivite tipi) | [TODO_MODULE_DECISIONS.md](todo/TODO_MODULE_DECISIONS.md) |
| Adapter adım yapısı ve kaldığı yerden devam | Kodda yok: adapter bazlı adım tanımı, Operation Engine adım / checkpoint / resume yok | [ADAPTER_WORKFLOW_STEP_ARCHITECTURE.md](operations/ADAPTER_WORKFLOW_STEP_ARCHITECTURE.md) |
| Adapter Builder (URL'den adapter üretimi) | Vizyon; kodda yok | [ADAPTER_BUILDER_V2.md](scraper/ADAPTER_BUILDER_V2.md) |
| Otomasyon sihirbazları: tekil e-posta (3.4), veri temizleme (3.5), hatırlatma (3.7) | Handler ve operasyon tipi yok | [AUTOMATION_ARCHITECTURE.md](operations/AUTOMATION_ARCHITECTURE.md) |
| Maliyet kataloğu: USD → TL güncel kur | Kur sağlayıcısı seçilmedi; ekranda "Kur bekleniyor". Maliyet hesabı ve teklif sonraki faz | [COST_CATALOG.md](decisions/COST_CATALOG.md) |

### Uygulama içi bildirim merkezi

**Faz 1, 2026-11-15.** CRM üst çubuğundaki zil okunmamış bildirim sayısını ve en yeni beş okunmamış bildirimi gösterir. **Tüm bildirimler** kişisel geçmişi açar. Liste sayfalanır ve Tümü / Okunmamış / Okunmuş filtrelerini sunar. Bildirim okundu işaretlendiğinde zilden düşer ve geçmişte kalır.

Super Admin, **Admin → System → Bildirimler** sayfasında geçmiş gönderimleri, gönderim tarihini ve alıcı sayısını görür. **Bildirim Gönder** formu başlık ve mesajı tüm aktif kullanıcılara ulaştırır. Gönderim mevcut audit servisine kaydedilir.

Core testleri kişisel kayıt kapsamını, alıcı seçimini, çift teslimatın önlenmesini, okundu durumunu, filtre ve sayfalamayı doğrular. CRM testleri izinleri, istemci sorgularını ve zil önizlemesini kapsar. Yerel tarayıcıda gerçek Core oturumuyla önizleme sorgusu ve 60 örnek bildirimle filtre, sayfalama ve okundu akışı doğrulandı. Dağıtımda CRM arayüzünden önce Core API ve 0079 migration uygulanır.

**Mevcut durum.** Core'da `in_app` kanalı, alıcıya göre kayıt ve yinelenen gönderimi önleyen anahtar var. `/system/notifications/in-app` Super Admin'e özgü; CRM'de TOBB akışı bunu kullanıyor. Faz 1 kişisel gelen kutusunu ve duyuru geçmişini bu kanala bağladı. Görev listesindeki `follow_up_at` için mevcut Faz 2 davranışı yalnız tarih kaydıdır; hatırlatma işi Faz 3 kapsamındadır.

| Kaynak | Ne zaman görünür | Alıcı |
|---|---|---|
| Super Admin duyurusu | Başlık ve mesaj önizlenip gönderim onaylanınca. Örnek: “Yeni zemin seçenekleri yayında.” | Gönderim anındaki aktif kullanıcılar. Toplu gönderim platform yetkisi gerektirir. |
| Zamanlanmış görev ve otomasyon | Başarı, son hata, kısmi sonuç, iptal veya kullanıcı müdahalesi gerektiğinde. Normal ilerleme ve yeniden deneme ayrıntıları görev ekranında kalır. | Atanan kişi veya tanımlı alıcılar. Bunlar yoksa organizasyondaki tüm kullanıcılar. Platform bakım işlerinde tanımlı operasyon sorumluları. |
| Aktivite ve görev takibi | İleri tarihli görev veya aktivitenin hatırlatma zamanı geldiğinde. Mevcut takip tarihi ayrı bir otomatik görev yaratmadan bildirim üretebilir. | Atanan kişi. Atama yoksa organizasyondaki tüm kullanıcılar. |
| Müdahale gerektiren sistem olayı | Örneğin planlı yedekleme başarısız olduğunda, toplu gönderim kısmen tamamlandığında veya bir aktarım onay beklediğinde. | Olayı görebilen ve işlem yapabilecek kullanıcı veya operasyon sorumlusu. |

**Kullanıcı akışı.** CRM üst çubuğunda zil ve okunmamış sayısı. Zilde son beş okunmamış bildirim ve “Tüm bildirimler” bağlantısı. Kişisel geçmiş sayfalanır ve okunma durumuna göre filtrelenir. Duyuru, hatırlatma ve görev sonucu filtreleri ile tümünü okundu işaretleme sonraki kapsamdadır. Görev veya aktivite bildirimi yetki denetiminden geçen detay ekranına gider. Super Admin ekranında başlık ve mesaj alanları vardır. Geçmiş, gönderim tarihini ve alıcı sayısını gösterir. Gönderen audit kaydında izlenir.

**Sahiplik ve güvenlik.** Core mevcut bildirim kanalını genişletir ve kullanıcıya özel teslimat, okundu durumu, sayfalama ve tekrar denemeyi yönetir. CRM, görev/aktivite olayının zamanına ve alıcılarına karar verir. Zamanlama için mevcut iş altyapısı kullanılır. Üreticiler doğrulanmış bir olay türü, kaynak kayıt, organization kapsamı ve tekrarsız olay kimliğiyle ortak kanalı çağırır. İş sonucu kaydedildikten sonra bildirim teslimatı güvenilir biçimde denenir; bildirim hatası kaynak işin sonucunu değiştirmez. Aynı olay ve kullanıcı için yeniden deneme ikinci mesaj üretmez.

Atamasız organizasyon bildirimini organizasyondaki tüm kullanıcılar görür. Kaynak kaydı açma yetkisi olmayan kullanıcıya genel özet gösterilir. Hassas içerik ve detay bağlantısı için kaynak erişimi sunucuda doğrulanır. Liste, okunmamış sayısı, okundu işlemleri ve detay bağlantısı her kullanıcının kendi kayıtlarıyla sınırlıdır. Yetki sonradan kalkarsa hassas içerik gösterilmez. Mesajlarda kullanıcıya anlaşılır özet ve işlem bağlantısı yer alır. Teknik hata, kişisel veri ve sır içeren ayrıntılar kendi yetkili kayıtlarında kalır. Görev iptal, tamamlanma, tarih değişikliği veya yeniden atamada bekleyen hatırlatma güncellenir. Saat dilimi, tekrar deneme ve geciken işler kabul testlerinde kapsanır. Kullanıcı/şirket silme ve saklama davranışı mevcut lifecycle kararlarıyla eşleştirilir.

**Teslim sırası.**

1. Ortak kutu ve Super Admin duyurusu. Core kullanıcı bazlı liste/okundu ve duyuru teslimatını sağlar. CRM zil, kutu ve gönderim ekranını bağlar. Mevcut TOBB bildirim akışı için regresyon testi yazılır.
2. Zamanlanmış görevler. Bir gerçek CRM otomasyonu başarı, son hata ve müdahale bildirimiyle bağlanır. Alıcı, detay bağlantısı ve tekrar deneme kanıtlandıktan sonra diğer görev türleri aynı yolu kullanır.
3. İleri tarihli aktivite ve görev. Hatırlatma zamanı, saat dilimi, yeniden atama, tarih değişikliği, iptal, tamamlanma ve eşzamanlı işleme doğrulanır.
4. Diğer kullanıcıya dönük sistem olayları. Her kaynak için olay, alıcı, yetki, önem ve detay hedefi tanımlanır. Sık tekrar eden hatalar gruplanır.

**Kabul.** Gerçek Core oturumu ile Super Admin duyurusu iki ayrı şirketten aktif kullanıcılara ulaşır. Normal kullanıcı toplu gönderim yapamaz. Bir kullanıcının okundu işlemi diğerini etkilemez. Atamasız görev bildirimi organizasyonun tüm kullanıcılarına görünür. Kaynak erişimi olmayan kullanıcı yalnız genel özeti görür. Diğer şirketin olayına liste, sayaç, doğrudan kimlik veya detay bağlantısıyla erişilemez. İptal edilen hatırlatma gitmez. İş veya bildirim yeniden denemesi çift kayıt üretmez. Güvenlik etkileri, testler ve kanıtlar [ortak güvenlik programına](../../standards/security/SECURITY_PROGRAM.md) göre izlenir.

**Uygulama öncesi kararlar.** Hatırlatma varsayılanı (tam saatinde veya önceden), rutin başarı bildirimlerinin varsayılan görünürlüğü ve saklama süresi.

## 3. Mail gönderim kuyruğu

Mevcut: `fair-crm-mail-worker` systemd servisi kuyruğu sürekli işler; takılan `sending` kurtarma, sağlayıcı hata politikası (`error_policy.py`: hesabı pasife al, otomatik yeniden dene), `Retry-After`, MSO `provider_status` ve Mail Operasyonları ekranında iptal mevcut.

| İş | Durum |
|---|---|
| SMTP hesabı ve organizasyon bazlı gönderim hız sınırı | Kodda yok |
| Hesap kapasitesine ve bağlantı durumuna göre dağıtım | Kodda yok |
| Geçici hatalar için zamanlı yeniden deneme / backoff motoru | Kodda yok; bugün yeniden deneme hata politikasıyla anında ya da kullanıcının "Tekrar Gönder"i ile |
| Sağlayıcıya özel politika (batch boyutu, eşzamanlılık, sessiz saatler) | Kodda yok |
| Kuyruk izleme ekranı (kuyruk derinliği, takılan `sending`, başarı/hata oranı, worker gecikmesi) | Kodda yok |
| Zamanlamanın Core `platform_jobs` ile yapılması | Kodda yok; bugün ayrı systemd worker. CRM Core Python modülünü import etmez (ADR-009) |
| MailerSend ile küçük gerçek toplu gönderim kabulü | Kayıt yok. Alıcı çözümleme, izin kontrolü, MSO, worker, API gönderimi, `external_message_id`, `provider_status`, yeniden deneme ve UI özeti birlikte doğrulanır |

## 4. Performans

| İş | Durum | Ayrıntı |
|---|---|---|
| Veritabanı index stratejisi | Migration `0023` mevcut; index sonrası ölçüm tablosu boş, her yeni tablo/sorgu için index incelemesi süreci | [INFRASTRUCTURE_PERFORMANCE_ROADMAP.md](performance/INFRASTRUCTURE_PERFORMANCE_ROADMAP.md) |
| Müşteri iletişim tabloları performans incelemesi | Ertelendi; tetik eşiği belgede | [CUSTOMER_COMMUNICATION_PERFORMANCE.md](performance/CUSTOMER_COMMUNICATION_PERFORMANCE.md) |

## 5. Operasyon

| İş | Durum | Ayrıntı |
|---|---|---|
| Arka plan işi ortak alanları (BACKGROUND_JOB_STANDARD §2) | İptal ucu ve `job_scope` bütün işlerde var (migration `0099`). Ad farkları ve eksik ilerleme / heartbeat / özet alanları tablo tablo standarttaki listede; alanların doldurulması işçi kodu ister | [BACKGROUND_JOB_STANDARD.md](operations/BACKGROUND_JOB_STANDARD.md) § 14 |
| GitHub'ın eski `refs/pull/*` ve önbelleği silmesi | Depo private ve geçmiş 2026-10-27'de yeniden yazıldı; Support talebi açık, yanıt kullanıcıda | [HISTORY_REWRITE_2026-10-27](ops/HISTORY_REWRITE_2026-10-27.md) |

## Kapanış sınırı: organizasyon yaşam döngüsü

P0.2 OL08 zorunlu dışa aktarma yolu 2026-09-11'de tamamlandı ([P0_2_OL_08_IMPLEMENTATION.md](../../ecosystem/P0_2_OL_08_IMPLEMENTATION.md)). **Yeni bir OL08/OL09 runtime işi sıralanmış veya onaylanmış değildir.** `not_required` yetkisi, yeni kanıt sınıfı, sağlayıcı yaşam döngüsü vakası veya yedek/geri yükleme yükümlülüğü önce açık bir politika/ürün kararıyla buraya satır olarak alınır.

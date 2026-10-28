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
| Operasyon: duraklat ve iptal ayrımı | `RunStatus.PAUSED` tanımlı ama duraklatma / sürdürme use case'i ve uç noktası yok; "Durdur" iptal eder. Hedef: "Durdur" `running → paused`, "Devam Ettir" kaldığı yerden sürdürür (ilerleme, log ve ara durum korunur), ayrı "İptal Et" `cancelled` yapar. Backend yaşam döngüsü, handler yeteneği, UI, yetki ve gerçek runtime kabulü birlikte | — |
| Müşteriyi pasife alma | Yok. Neden (seçilir veya yazılır), not ve müşteri aktivite kaydı; gönderim öncesi kontrol "Customer pasif mi?" adımını da içerir. Müşteri/Contact e-posta izni, MailerSend unsubscribe/spam webhook'u ve merkezi `EmailConsentPolicy` mevcut | [archive: EMAIL_COMMUNICATION_PREFERENCES](../../archive/fair-crm/backlog/EMAIL_COMMUNICATION_PREFERENCES.md) |
| Görevler Faz 3 | Faz 2 dışında bırakılanlar: kişi / lead / katılımcı bazlı çalışma listesi, satır bazlı kullanıcı atama, otomatik görev oluşturma ve auto-reopen, `follow_up_at` üzerinden görev/bildirim/takvim, Excel export, AI not özeti, kampanya otomasyonu, otomatik arama entegrasyonu | [WORKLIST_PHASE2.md](todo/WORKLIST_PHASE2.md) § 7 |
| WhatsApp ve SMS gönderimi | Kodda gönderim yok (WhatsApp yalnız aktivite tipi) | [TODO_MODULE_DECISIONS.md](todo/TODO_MODULE_DECISIONS.md) |

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
| GitHub'ın eski `refs/pull/*` ve önbelleği silmesi | Depo private ve geçmiş 2026-10-27'de yeniden yazıldı; Support talebi açık, yanıt kullanıcıda | [HISTORY_REWRITE_2026-10-27](ops/HISTORY_REWRITE_2026-10-27.md) |

## Kapanış sınırı: organizasyon yaşam döngüsü

P0.2 OL08 zorunlu dışa aktarma yolu 2026-09-11'de tamamlandı ([P0_2_OL_08_IMPLEMENTATION.md](../../ecosystem/P0_2_OL_08_IMPLEMENTATION.md)). **Yeni bir OL08/OL09 runtime işi sıralanmış veya onaylanmış değildir.** `not_required` yetkisi, yeni kanıt sınıfı, sağlayıcı yaşam döngüsü vakası veya yedek/geri yükleme yükümlülüğü önce açık bir politika/ürün kararıyla buraya satır olarak alınır.

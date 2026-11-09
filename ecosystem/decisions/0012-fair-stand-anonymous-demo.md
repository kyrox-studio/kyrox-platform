# ADR-0012: Fair Stand oturumsuz demo sayfası

- **Durum:** Accepted
- **Tarih:** 2026-11-09
- **Karar verenler:** KYROX ekosistem sahipleri (kullanıcı onayı 2026-11-09)

## Bağlam

Fair Stand editörünün bütün API'leri Core oturumu ve yetki ister ([ADR-0008](0008-fair-stand-independent-service.md): tek giriş Core JWT, ikinci login yok). Yeni bir kullanıcının satın almadan önce editörü denemesi isteniyor. Deneyen kişi bütün çizimi yapıp ekran görüntüsüyle ücretsiz kullanmamalı; kayıt, dışa aktarma, render ve gerçek Üretim Listesi demoda olmamalı.

Daha önce demo, misafir ya da guest kararı yok. [ADR-0009](0009-demo-recorder-repository.md) (silinen video kayıt reposu) bu konuyla ilgili değildir.

## Karar

1. **Yer:** CRM içinde oturumsuz `/stand-demo` sayfası (`fuar.kyrox.studio/stand-demo`). Giriş kontrolünden önce açılır, canlı izleme sayfası (`/stand/watch/:token`) ile aynı yöntem. Editör `mountFairStand(el, { demo: true })` ile açılır; Core başlığı, proje, müşteri ve yetki bilgisi geçirilmez.
2. **İkinci login değildir.** Demo bir kimlik ya da rol oluşturmaz. Demo çerezi yalnız `/api/v1/fair-stand/demo/*` uçlarında geçerlidir; proje, şablon, maliyet, canlı paylaşım ve admin uçları Core JWT istemeye devam eder.
3. **Katalog:** `GET /api/v1/fair-stand/demo/catalog/bootstrap` görünür ve gizli bütün aktif Item'ları döner (kullanıcı kararı: tüm görünür item'lar). Sunucu yalnız BOM / maliyet için kullanılan alanları çıkarır: `lengthParts`, `isCostEnabled`. `composition` sahne, snap ve montaj için de okunduğu için çıkarılamaz; birleşik Item reçetesi ağ yanıtında okunabilir. Bu kabul edilen risktir.
4. **Kilitli işlemler** basınca "Demo sayfasında bu işlem için oturum açmanız gerekiyor." der: Farklı Kaydet, Dışarı Aktar, İçe Aktar, Aç, Sil, Kaydet, Render Al, Ekranı paylaş, Şablon olarak kaydet, proje adı Değiştir. Otomatik kayıt kapalı ve seçilemez; Geçmiş gizli. Açık kalanlar: Gelişmiş mod (kullanıcı kararı), görsel yükleme (tarayıcıda kalır), Sahne Işığı, Geri/İleri al.
5. **Üretim Listesi** açılır ama gerçek liste hiç hesaplanmaz: "Demo görüntülemede gerçek Üretim Listesi görülemez." notu ve sabit, açıkça "Örnek" adlı satırlar gösterilir. Satırlar sahneye göre değişmez; metin indirme yoktur.
6. **Süre sınırı sunucudadır:** aynı tarayıcı 24 saatte **25 dakika** aktif kullanım (kullanıcı kararı), aynı IP bunun 3 katı. Kalan süre iki sayacın küçüğüdür. Süre yalnız 30 saniyede bir gelen canlılık sinyaliyle işler; bir sinyal en çok 45 saniye sayar, gizli sekme sinyal göndermez. Süre dolunca sinyal ve katalog 403 döner, editör kilit ekranıyla kapanır. Sahnede filigran vardır: Kyrox Studio logosu ve altında "DEMO" (düz, soluk; marka standardı gereği logo döndürülmez).
7. **Kimlik:** tarayıcı, sunucunun verdiği rastgele `fs_demo_id` çereziyle tanınır (HttpOnly, SameSite=Strict, yol `/api/v1/fair-stand/demo`, 7 gün). IP Nginx'in yazdığı `X-Real-IP` başlığından alınır; başlık yalnız güvenilen vekilden (`127.0.0.1`) gelirse okunur. IPv6 için /64 ön eki sayılır. Ham IP ve çerez saklanmaz, yalnız `FAIR_STAND_DEMO_SECRET` ile HMAC-SHA256 özeti tutulur; 48 saat görülmeyen satır silinir. Tarayıcı parmak izi (canvas/WebGL) kullanılmaz: kolay atlatılır ve aydınlatma/rıza gerektirir.
8. **Açma-kapama:** `FAIR_STAND_DEMO_ENABLED=true` ve boş olmayan `FAIR_STAND_DEMO_SECRET` birlikte gerekir; biri yoksa demo uçları 404 döner (fail-closed). Süre `FAIR_STAND_DEMO_MINUTES`, IP katı `FAIR_STAND_DEMO_IP_MULTIPLIER` ile değişir.
9. **Hız sınırı:** Nginx demo uçlarına IP başına dakikada 30 istek (patlama 20) uygular; kural CRM `scripts/server/nginx_performance.py` ile her deploy'da eksikse eklenir.

## Bilinen atlatma yolları

- VPN, mobil veri ya da modem yenileme ile IP değiştirip gizli pencere açan kişi yeni süre alır.
- Tarayıcıda JavaScript'i değiştiren kişi, yüklenmiş sahneyi kilit ekranı olmadan kullanmaya devam edebilir; yeni katalog alamaz.
- 3D model ve dokular (`/models/`, `/textures/`) bugün de oturumsuz indirilebilir.
- Ekran görüntüsü engellenemez. Zararı azaltanlar: kayıt, dışa aktarma ve render yok; gerçek Üretim Listesi yok; filigran ve süre sınırı var.

## Sonuçlar

- Fair Stand Alembic `0094_demo_usage`: `fair_stand_demo_usage` tablosu.
- Yeni çerez `fs_demo_id`, web çerez envanterine ([WEB_STORAGE_AND_SERVICES.md](../WEB_STORAGE_AND_SERVICES.md)) eklenir.
- Canlıya alma: önce Stand (migration + `.env` değerleri), sonra CRM (rota + Nginx kuralı).

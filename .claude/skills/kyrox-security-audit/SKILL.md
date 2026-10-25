---
name: kyrox-security-audit
description: KYROX ekosisteminin (kyrox-core, kyrox-fair-crm, kyrox-fair-stand, kyrox-web, sunucu) uçtan uca güvenlik denetimi ve karar uyum taraması. Platform belgelerini, ADR'leri ve SECURITY_PROGRAM.md'yi okur, yüzey envanterini çıkarır, 6 kapsamı paralel alt ajanlarla tarar, belge/ADR ile kod ve canlı yapılandırma arasındaki farkları bulur ve sonucu kanıt türleri ayrılmış biçimde SECURITY_PROGRAM.md'ye işler. Kullanıcı "güvenlik denetimi", "uçtan uca tarama", "açıkları bul", "kararlarla kodu karşılaştır", "ADR'ye uyuyor mu", "belgelerde yazanla kod aynı mı", "eksikleri bul", "SEC kontrollerini eşleştir", "neyi bozmuşuz", "Cursor/ChatGPT döneminde aldığımız kararlar uygulanmış mı" gibi bir şey söylediğinde veya bir değişikliğin KYROX kararlarına/güvenlik kurallarına uyup uymadığını sorduğunda bu skill'i kullan — kullanıcı "skill" demese bile.
---

# KYROX güvenlik ve karar uyum denetimi

Bu skill, KYROX'ta güvenlik açıklarını ve **yazılı kararlar ile gerçek sistem arasındaki uyumsuzlukları** kanıtla bulmak için tekrar çalıştırılabilir prosedürdür. Tek kayıt `standards/security/SECURITY_PROGRAM.md`'dir; yeni rapor dosyası üretilmez.

Neden böyle: KYROX'ta kararlar zaman içinde farklı araçlarla (Cursor, ChatGPT, Claude) alındı ve belgelere dağıldı. Bir denetim yalnız koda bakarsa "yeni davranış" sanılan şeylerin aslında eski karar olduğunu kaçırır; yalnız belgeye bakarsa kodun kararı uygulamadığını kaçırır. Bu yüzden iki yönü ayrı adımlarda karşılaştırırız ve her iddiayı kanıt türüyle etiketleriz.

## 0. Önce kapsamı ve sınırları netleştir

Kullanıcı açıkça istemedikçe tam denetim pahalıdır (6 alt ajan, uzun testler). Başlamadan:
- Hedef: tüm sistem mi, tek depo/değişiklik mi, yalnız karar uyumu (adım 4) mu? Dar istekte yalnız ilgili adımları çalıştır.
- Kaynak SHA'ları kaydet: her depo için `git fetch` + `git rev-parse origin/main`. Canlı sürümü salt okunur oku (`references/rules.md` §Canlı).
- Diğer oturumların açık işini bul: her depoda `git worktree list`, `gh pr list --state open`. Onlara dokunma; kendi worktree'lerini `Kyrox/.tmp/<benzersiz-ad>` altında aç.
- [SECURITY_PROGRAM.md](../../../standards/security/SECURITY_PROGRAM.md) içindeki mevcut kimlikleri (E-, F-, R-) listele: yeni bulgu mevcut kimliği tekrar üretmemeli, yalnız yeni bulgu veya durum değişikliği eklenmeli.

Sınırları her alt ajana aynen ver: [references/rules.md](references/rules.md).

## 1. Kararları ve kuralları oku

Okuma sırası ve öncelik [AGENTS.md](../../../AGENTS.md)'deki gibidir. Asgari:
- `ecosystem/decisions/` (ADR'ler, `ADR_INDEX.md`), `ecosystem/DOCUMENT_GOVERNANCE.md`, `standards/` (özellikle security, quality, ui).
- `projects/*/` — `PROJECT_STATUS.md`, `CHANGELOG.md`, `ROADMAP.md`, `integrations/` rehberleri, proje `decisions/`.
- `archive/*/designs` — tarihsel karar bağlamı için okunur ama **canlı kural değildir** (AGENTS.md Precedence). Archive'daki bir karar canlı belgede yoksa bu bir "belgede eksik" bulgusudur, otomatik kural değildir.
- Kod depolarındaki makine sözleşmeleri: CRM `.kyrox/features/*.json`, Stand `.github/change-contract.json` ve `SYSTEM_*.md`, Core `.kyrox/security-controls.json`.

Öncelik: ADR > shared standard > proje belgesi > entegrasyon rehberi > archive > sohbet geçmişi. Çelişki varsa hangisinin canlı olduğunu yaz.

## 2. Yüzey envanteri

Örnekleyerek değil, sayarak çıkar ve SECURITY_PROGRAM'daki envanterle karşılaştır (değişen sayılar da bulgudur):
- API route'ları (FastAPI `app.routes` programatik sayım; kimliksiz / bearer / permission / servis token grupları).
- Dosya yükleme/indirme, export/import, arka plan işleri ve worker'lar.
- Sunucunun dışarı istek attığı yollar (httpx/requests/Playwright/SMTP).
- DB'ler, roller, sunucu servisleri, nginx yolları, cron/timer'lar, deploy/seed betikleri.

## 3. Altı kapsam — paralel alt ajanlar

Her kapsam için bir alt ajan başlat (aynı mesajda, arka planda). Her birine: [references/rules.md](references/rules.md) yolunu, kendi worktree yolunu, kaynak SHA'yı, rapor dosyası yolunu (scratchpad) ve [references/scopes.md](references/scopes.md)'deki ilgili bölümü ver. Rapor biçimi [references/finding-format.md](references/finding-format.md).

1. Core kimlik / oturum / parola kurtarma / yetki yükseltme
2. CRM organization izolasyonu
3. Stand proje/revizyon/görsel/paylaşım + dosya yükleme
4. DB ve sunucu (canlı salt okunur)
5. Tarayıcı ve dış bağlantılar (XSS, CSRF, cookie, CORS, başlık, SSRF, rate limit)
6. Tedarik zinciri ve toparlanma (bağımlılık, CI, sır taraması, izole restore)

Ajanlar oturum limitine takılırsa yeniden başlatma; `SendMessage` ile kaldıkları yerden devam ettir. Bir ajanın canlı yapılandırma gözlemi başka bir ajanın kod sorusunu doğuruyorsa (ör. canlıda token eksik → kod bu durumda ne yapıyor?) soruyu ilgili ajana ilet.

## 4. Karar ↔ kod uyum taraması (ayrı adım)

Bu adım güvenlik taramasından ayrıdır, çünkü amacı "kod güvenli mi" değil "kod kararlarımızı uyguluyor mu"dur. Bir alt ajan veya kendin yap:

- Adım 1'deki her uygulanabilir kararı tek satıra indir: `K-<proje>-<n> | kaynak dosya + bölüm | karar ne söylüyor | kodda nerede (dosya+sembol) | canlıda nasıl | durum`.
- Durum değerleri: **UYUYOR** (kod + test kanıtı), **ÇELİŞİYOR** (kod kararın tersini yapıyor), **KISMİ** (bazı yollar uyguluyor), **BELGEDE EKSİK** (kod/canlı bir davranış var ama hiçbir canlı belgede karar yok), **BELGELER ÇELİŞİYOR** (iki canlı belge farklı söylüyor), **DOĞRULANMADI**.
- Bir davranışı "yeni" veya "karar gerektiriyor" diye raporlamadan önce adım 1'deki kaynaklarda grep'le (anahtar kelime + Türkçe/İngilizce eşanlamlılar). Karar zaten varsa bulgu "kod kararı uygulamıyor" olur, kullanıcıya yeniden sorulmaz.
- Yalnız gerçekten belirsiz veya çelişkili kararları "karar bekliyor" listesine koy; her biri için seçenekleri ve önerini yaz.

## 5. Kayıt ve teslim

- Bulguları SECURITY_PROGRAM.md'ye mevcut düzene uyarak ekle: yeni E- kanıt bölümü, yeni F-/R-/K- satırları, matris ve düzeltme sırası güncellemesi. Her bulguda kod / izole / canlı kanıtı ayrı sütunlarda, kaynak SHA, test/komut, ortam, sonuç ve eksik doğrulama olsun ([references/finding-format.md](references/finding-format.md)).
- "Kod var", "test geçti (izole)", "canlıda doğrulandı", "kabul edildi" ayrı durumlardır; kanıtsız PASS/DONE yazma. Kontrolleri, testleri veya kapıları gevşetme.
- Platform deposunda ayrı worktree + dal + PR; kullanıcı tercihine göre auto-merge. Platform CI yalnız Markdown kabul eder ve depo içi linklerin çözülmesini ister.
- Kod düzeltmesi bu skill'in işi değildir; kullanıcı ayrıca isterse her depo için ayrı dal/PR ve mevcut sıkı testlerle yapılır.
- Kullanıcıya kısa Türkçe özet: en kritik yeni bulgular (öncelik sırasıyla), karar bekleyenler, doğrulanamayanlar ve nedeni.

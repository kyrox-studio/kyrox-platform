# Fair CRM git geçmişi yeniden yazımı — 2026-10-27

`kyrox-fair-crm` geçmişinde gerçek firma/kişi verisi (ad, telefon, e-posta) içeren betik çıktıları vardı. Depo public iken bu dosyalar okunabiliyordu; 2026-10-27'de depo private yapıldı ve geçmiş `git filter-repo --invert-paths` ile yeniden yazıldı.

## Çıkarılan yollar

`scripts/legacy/reports/`, `scripts/legacy/migration_reports/`, `scripts/legacy/cleaned/`, `scripts/maintenance/reports/`, `frontend/reports/`, `backend/data/scraper-handoff/`, `backend/foodist_*`, `backend/detail_*.html`, `backend/parser_test.txt` (geçmişte 951 yol). `scripts/legacy/cleaned/*.json` o güne kadar `main`'de duruyordu; yeniden yazımla `main`'den de çıktı. Hepsi `.gitignore`'dadır ve betiklerin yeniden üretilebilir çıktısıdır.

## Sonuç

- Commit sayısı aynı (1747); yeni `main` `04412027` (eski `02b85bb6`). Tek ağaç farkı `scripts/legacy/cleaned/` dosyalarıdır.
- Depo paketi 99 MB → 15 MB. Merge olmuş 5 uzak dal silindi.
- Sunucu `/opt/kyrox/kyrox-fair-crm` eski `c92247b8`'in karşılığı `33b4f47e`'ye alındı (kod aynı; servis yeniden başlatılmadı). Sonraki deploy ileri sarma ile çalışır.

## Eski SHA'lar

Bu tarihten önceki belgelerde geçen CRM commit SHA'ları eski geçmişe aittir. Karşılıkları: [HISTORY_REWRITE_2026-10-27_commit-map.tsv](HISTORY_REWRITE_2026-10-27_commit-map.tsv) (`old` → `new`).

## Açık

GitHub, eski PR'ların `refs/pull/*/head` referanslarını ve önbelleği kendisi tutar; bunları yalnız GitHub Support siler. Talep kullanıcı tarafından gönderilir.

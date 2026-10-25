# Bulgu ve kanıt biçimi

SECURITY_PROGRAM.md §3 durum modeli ve kanıt kaydı alanları geçerlidir; burada yalnız uygulama özeti var.

## Alt ajan raporu (scratchpad)
1. **Yüzey envanteri** — sayılarla, dosya yollarıyla.
2. **Kontrol matrisi** — yüzey × SEC-xx → DOĞRULANMADI / KOD VAR / TEST GEÇTİ (izole) / CANLIDA DOĞRULANDI / BULGU AÇIK / N/A + gerekçe.
3. **Kanıtlanmış bulgular** — aşağıdaki satır biçimiyle.
4. **Doğrulanmayı bekleyen riskler** — neden doğrulanamadı, nasıl doğrulanır.
5. **Kapsam dışı** ve nedeni.
6. **Öneriler** — (a) güvenli, kapsamı net; (b) ürün/mimari karar gerektiren.

## Bulgu satırı
| Kimlik / SEC | Akış | Kod | İzole kanıt | Canlı yapılandırma | Risk / durum | Eksik doğrulama |

- **Kod:** dosya + sembol (+ satır), kaynak SHA.
- **İzole kanıt:** test adı / komut, ortam (yerel SQLite, geçici Postgres, mock'lu mu), sonuç (passed/failed sayısı).
- **Canlı:** yalnız salt okunur gözlem; değer yerine anahtar adı/hash.
- **Risk:** P0/P1/P2; durum: BULGU AÇIK, DÜZELTME PR'DA, CANLIDA DEPLOY, KABUL EDİLDİ (canlı).

## Karar uyum satırı (adım 4)
| Kimlik | Kaynak (dosya + bölüm) | Karar | Kodda | Canlıda | Durum | Not / öneri |

Kimlik öneki `K-` (ör. `K-CORE-03`). Durum: UYUYOR, ÇELİŞİYOR, KISMİ, BELGEDE EKSİK, BELGELER ÇELİŞİYOR, DOĞRULANMADI.

## Kimlik çakışması
Yeni bulguya SECURITY_PROGRAM'da kullanılmamış bir sonraki numarayı ver; mevcut bir bulgunun durumu değişiyorsa yeni satır açma, mevcut satırı güncelle ve değişikliği yeni E- kanıt bölümünde açıkla.

# Denetim kuralları (her alt ajana aynen verilir)

Bu kurallar 2026-10-25 denetiminde (SECURITY_PROGRAM E-013) kullanılan sınırlardır. Amaç: denetimin kendisinin veri kaybına, hesap kilitlenmesine, sır sızıntısına veya başka oturumların işini bozmasına yol açmaması.

## Çalışma alanı
- Yalnız `Kyrox/.tmp/<benzersiz-ad>` altında `origin/main`'den açılmış ayrık (detached) worktree'lerde incele ve test et. Ana kopyalara ve başka oturumların worktree'lerine dokunma (`git worktree list` ile kontrol et).
- Commit, push, PR, dal oluşturma yalnız koordinatörün işidir. Worktree'ye geçici test eklersen bitince geri al; kalıcı olmayan test betikleri scratchpad'de durur.
- Ana kopyalardaki `.venv`'leri yalnız yorumlayıcı olarak kullan; paket kurmak gerekiyorsa scratchpad'de ayrı venv aç. Windows'ta yeni venv'e `tzdata` gerekebilir.
- CRM backend testleri `backend/` dizininden koşulur (CI gibi); kökten koşmak göreli yol hatası verir.
- Testler yerel/izole: SQLite/geçici DB veya ayrı adlı yerel Docker konteyneri (bitince sil). Canlı DB'ye bağlanma.
- `scripts/dev-start.ps1` gibi yerel şifre sıfırlayan betikleri sormadan çalıştırma.

## Canlı ortam
- `ssh kyrox` yalnız PowerShell aracından çalışır. Çok satırlı komut: scratchpad'e LF satır sonlu, BOM'suz `.sh` yaz, `cmd /c "ssh kyrox bash -s < dosya.sh"`.
- Yalnız salt okunur yapılandırma/metadata: dosya izni (`stat`), `ss -tlnp`, `systemctl show`, nginx yapılandırması, `pg_roles`/`pg_trigger` gibi katalog sorguları, sertifika. Değiştirme, yeniden başlatma, deploy, silme yok.
- Sırları ve kişisel verileri çıktıya yazma: `.env` değerlerini basma; anahtar adlarını (`cut -d= -f1`) veya sha256 ilk 8 karakterini kullan. DB'de satır verisi okuma.
- Okunabilirlik testi içerik okumadan: `runuser -u nobody -- test -r` ve `head -c 0`.
- Dış HTTP: yalnız tekil GET/HEAD/OPTIONS. Canlıda rate-limit, yük, upload, restore veya istismar denemesi yok. Veriyi değiştirebilecek bir uca (ör. purge) geçerli kimlikle istek atma; var olmayan rastgele kimlik ve ret yolları dışında deneme yapma.

## Rapor
- Rapor dosyası scratchpad'de; biçim `finding-format.md`.
- Örnek birkaç uca bakıp tüm sisteme PASS verme; sayarak envanter çıkar.
- Mock'lu testleri gerçek zincir kanıtı sayma; hangisinin mock olduğunu yaz.
- Kanıtsız bulgu yazma; şüpheyi "doğrulanmayı bekleyen risk" olarak ayır.

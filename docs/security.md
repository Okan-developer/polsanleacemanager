# Security

> Bootstrap'ta belirlendi (2026-10-19). Ajanlar her plan ve review'da bu kurallara uyar.

## Secrets
- Secrets never enter the repo, specs, prompts, or chat. `.env` is gitignored; provide `.env.example`.
- Agents never print secret values, even when debugging.
- .NET'e özel: geliştirmede `dotnet user-secrets`, sunucuda ortam değişkeni ya da kurumsal secret
  store kullanılır. `appsettings*.json` içinde bağlantı parolası ya da OIDC client secret olmaz.

## Input & output
- Her dış girdi (API isteği, Blazor formu) sınırda doğrulanır: Application katmanındaki komut
  doğrulayıcıları tek doğruluk kaynağıdır; UI doğrulaması yalnızca kullanıcı kolaylığıdır.
- Veritabanı erişimi yalnızca EF Core/parametreli sorgularla yapılır; string birleştirerek SQL yasaktır.
- Blazor varsayılan HTML encoding'i kapatılmaz; `MarkupString` kullanıcı verisiyle kullanılmaz.
- Hata yanıtları ProblemDetails'tır ve yalnızca stabil hata kodu + Türkçe mesaj içerir; stack trace,
  SQL, iç tip adları ve kişisel veri asla sızmaz (`docs/conventions.md`, Error handling).
- **KVKK — kişisel veri**: doğum tarihi, izin gerekçesi ve ret gerekçesi kişisel veridir. Loglara,
  telemetriye ve hata mesajlarına **yazılmaz** (loglarda yalnızca ID'ler); yalnızca gereken role gösterilir.
- **Sağlık verisi yok**: ilk sürümde sağlık raporu ya da belge eki saklanmaz (KVKK özel nitelikli veri).
  Eklenmesi yeni bir spec ve güvenlik değerlendirmesi gerektirir.

## AuthN / AuthZ
- **Kimlik doğrulama**: OpenID Connect ile kurumsal IdP (Microsoft Entra ID; şirket AD FS ise aynı
  protokol). Uygulama parola saklamaz. Çalışan, IdP kimliği (`oid`/`sub`) ile Organization modülünde eşlenir.
- **Sahte kimlik şeması** yalnızca `Development` ve test ortamlarında kayıtlıdır; `Production`'da
  kaydı başlangıçta hata verir (fail-closed).
- **Varsayılan-ret**: fallback authorization policy = kimliği doğrulanmış kullanıcı; anonim uç yalnızca
  health check. Yeni bir uç açıkça yetki ister.
- **Roller**: `Employee`, `Manager`, `HR`. Yönetici rolü, Organization'daki doğrudan yönetici ilişkisinden türetilir.
- **Kaynak düzeyinde yetki Application katmanında** zorlanır (UI'da gizlemek yetki değildir):
  çalışan yalnızca kendi taleplerini/bakiyesini; yönetici yalnızca doğrudan ekibinin taleplerini; İK herkesi görür.
  Kimse kendi talebini onaylayamaz (BR-9). Yetkisiz erişim `Forbidden`, var olmayan ya da görülemeyen kaynak `NotFound` döner.
- HTTPS zorunlu, HSTS açık; Blazor/API için antiforgery açık; çerezler `Secure`, `HttpOnly`, `SameSite=Lax`.

## Dependencies
- Yeni NuGet paketi **plan aşamasında** gerekçesiyle önerilir; insan plan onayıyla birlikte onaylar.
  Plan dışı paket eklemek yasaktır.
- Lisans beyaz listesi: MIT, Apache-2.0, BSD-2/3-Clause, MS-PL. Ücretli ya da ticari lisanslı paket
  yok (ör. FluentAssertions 8+ bu yüzden kullanılmaz; yerine AwesomeAssertions).
- Bakımı süren (son 12 ayda sürüm) ve bilinen High/Critical zafiyeti olmayan paket seçilir.
- Versiyonlar CPM ile sabitlenir; `scripts/check` tam profilde `dotnet list package --vulnerable
  --include-transitive` çalıştırır ve High/Critical bulgu check'i kırmızı yapar.

## Review lens
Security is a mandatory dimension of every independent review (see `prompts/review.md`), not a
separate afterthought phase.

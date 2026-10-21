# Conventions

> Bootstrap'ta belirlendi (2026-10-19). Buradaki her kural ya araçla zorlanır (tercih edilen) ya da
> review'da kontrol edilir. Temenni buraya yazılmaz.

## Language
<!-- Set at bootstrap (step 0) and mirrored in AGENTS.md. Changing it later is allowed — edit both
     places; existing documents are not translated retroactively. -->
- Chat language (interviews, sessions): Turkish (`tr`)
- Document language (docs/, specs, plans, ADRs, review and verify reports): Turkish (`tr`)
- Always English (protocol, not prose): ANEW core files, template headings and field labels,
  `Status:` values (Draft / Approved / In progress / Shipped), `Approved by / on:`, `Source:`.
- Code identifiers, branch names, commit messages: English. Alan terimlerinin Türkçe↔İngilizce
  eşlemesi `docs/domain.md` sözlüğündedir. Kullanıcıya görünen metinler Türkçedir (`.resx`).
- Agent messages (handoff summaries, refusals, `Next:` lines): chat language; command names, file
  paths and protocol fields verbatim.

## Language & framework versions
- .NET SDK 10 (LTS), `global.json` ile sabitlenir (`rollForward: latestFeature`); C# 14.
- ASP.NET Core 10 + Blazor Web App; EF Core 10 + Npgsql; PostgreSQL 18.
- xUnit v3, AwesomeAssertions, NetArchTest, Testcontainers.PostgreSql, Microsoft.Playwright.
- Tüm paket versiyonları **Central Package Management** ile `Directory.Packages.props`'ta sabitlenir;
  csproj'larda versiyon yazılmaz. Ortak derleme ayarları `Directory.Build.props`'tadır.
- Yeni paket ekleme kuralı: `docs/security.md` → Dependencies.

## Naming
- Projeler: `PolsanLeaveManager.<Module>`, `PolsanLeaveManager.<Module>.Contracts`; testler
  `PolsanLeaveManager.<Module>.Tests` (`docs/architecture.md` çözüm düzeni).
- .NET standart isimlendirmesi (PascalCase tipler/üyeler, `_camelCase` private alanlar, `I` önekli arayüzler, async metotlar `Async` sonekli).
- Test metodu: `Method_Scenario_Expected` (ör. `Submit_WhenBalanceInsufficient_ReturnsInsufficientBalance`);
  iş kuralını test eden metot BR numarasını trait olarak taşır: `[Trait("Rule", "BR-6")]`.
- Hata kodu: `<module>.<snake_case>` (ör. `leave.insufficient_balance`, `leave.overlapping_request`).
- DB şeması = modül adının küçük harfi (`organization`, `leave`, `calendar`).
- Dal ve commit isimlendirmesi: `docs/git.md`.

## Error handling
- **Beklenen hata** (iş kuralı ihlali, doğrulama, bulunamadı, yetkisiz): exception değil,
  `Result<T>` / `Result` döner; `Error` stabil bir kod ve tür taşır (`Validation`, `NotFound`, `Forbidden`, `Conflict`).
- API'de `Result` hatası **ProblemDetails (RFC 9457)** olur: `type`/`title` + `extensions.code` = hata kodu;
  HTTP durumu hata türünden türetilir (400/403/404/409). Blazor aynı kodu `.resx`'ten Türkçe mesaja çevirir.
- **Beklenmeyen hata**: exception fırlar, yalnızca global handler (`IExceptionHandler`) yakalar,
  loglar ve kullanıcıya trace id içeren genel bir mesaj döner. Stack trace, SQL, iç tip adları ve
  kişisel veri kullanıcıya asla sızmaz.
- `catch (Exception)` ile hatayı yutmak yasaktır; yalnızca global handler ve dış sistem sınırları yakalar.

## Data rules
- İzin süresi, hak ve bakiye: `decimal`, 0,5'in katı. `float`/`double` yasak.
- İzin tarihleri: `DateOnly` (izin bir takvim günüdür, saat dilimi taşımaz).
- Denetim zaman damgaları (`CreatedAt`, `DecidedAt` …): `DateTimeOffset`, **UTC** saklanır;
  arayüzde `Europe/Istanbul` gösterilir. Kod `DateTime.Now`/`UtcNow` yerine enjekte edilen `TimeProvider` kullanır.
- Kimlikler: `Guid` v7 (`Guid.CreateVersion7()`, zamana göre sıralı); veritabanı üretmez.
- Bakiye bir sütunda tutulmaz; hareket defterinden türetilir. Defter satırları güncellenmez/silinmez (`docs/domain.md`).
- Bir modül yalnızca kendi şemasına yazar ve okur.

## Enforced by tooling
İlk spec (çözüm iskeleti) bunları kurar; o zamana kadar `scripts/check` greenfield no-op'tur.
- Derleyici: `Nullable=enable`, `TreatWarningsAsErrors=true`, `AnalysisLevel=latest-recommended` (`Directory.Build.props`).
- Biçim ve stil: `.editorconfig` + `dotnet format --verify-no-changes` (check `format` adımı).
- Modül sınırları A1–A6: `PolsanLeaveManager.Architecture.Tests` (check `test unit+architecture` adımı).
- Modül iç tiplerinin `internal` olması: derleyici.
- Paket versiyonları: CPM (`ManagePackageVersionsCentrally=true`, csproj'da `Version=` olursa derleme hatası).
- Zafiyetli paket (High/Critical): check `vulnerable packages` adımı (tam profil).

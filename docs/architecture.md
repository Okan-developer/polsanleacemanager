# Architecture

> Bootstrap'ta karar verilen mimari (2026-10-19). Ajanlar plan yapmadan önce bu dosyayı okur.

## System overview
PolsanLeaveManager, Polsan çalışanlarının izin talebi oluşturduğu, yöneticilerin onayladığı ve
izin bakiyesinin hareket defterinden (ledger) takip edildiği kurum içi bir web uygulamasıdır.
Stil: **modüler monolit**. Tek bir ASP.NET Core (.NET 10) host olarak deploy edilir; içeride
sınırları mimari testlerle korunan iş modülleri bulunur. Arayüz aynı host'ta Blazor Web App,
veri PostgreSQL'de durur; her modülün kendi şeması ve DbContext'i vardır. Gerekçe: bu ölçekte
mikroservis gereksiz operasyon yükü getirir, sınırlar ise ileride modülü ayırmayı mümkün kılar
(`docs/decisions/0001-modular-monolith-dotnet-postgres.md`).

Çözüm düzeni (ilk spec kurar):
```
PolsanLeaveManager.slnx
src/PolsanLeaveManager.SharedKernel/          Result<T>, Error, ID üretimi — hiçbir şeye bağımlı değil
src/Modules/<Module>/PolsanLeaveManager.<Module>/            iç: Domain/ Application/ Infrastructure/
src/Modules/<Module>/PolsanLeaveManager.<Module>.Contracts/  diğer modüllerin göreceği tek yüzey
src/PolsanLeaveManager.Web/                   host: Blazor UI + HTTP API + kimlik + modül kaydı
tests/PolsanLeaveManager.<Module>.Tests/      Unit + Integration
tests/PolsanLeaveManager.Architecture.Tests/  yasak bağımlılık testleri
tests/PolsanLeaveManager.E2E.Tests/           Playwright
```

## Modules / components and ownership

| Module | Single responsibility | Owns |
|---|---|---|
| Organization | Çalışanı ve organizasyon yapısını tanımak | Çalışan, işe giriş tarihi, doğum tarihi, doğrudan yönetici ilişkisi, rol ataması (şema `organization`) |
| Leave | İzin hakkını hesaplamak ve izin talebinin yaşam döngüsünü yönetmek | İzin türü, hak tablosu, izin talebi, onay kararı, bakiye hareket defteri (şema `leave`) |
| Calendar | Bir tarih aralığında kaç iş günü olduğunu söylemek | Resmi tatil takvimi (arife yarım günleri dahil), çalışma haftası tanımı (şema `calendar`) |
| Web | Kullanıcıya ve HTTP istemcilerine yüzey sunmak; modülleri bir araya getirmek | İş verisi sahibi değil. Kimlik doğrulama, Blazor sayfaları, API uçları, DI kaydı |
| SharedKernel | Ortak teknik yapı taşları | `Result<T>`, `Error` (stabil hata kodu), Guid v7 üretimi. İş kuralı yok |

## Communication rules
- Modüller arası çağrı **yalnızca senkron** ve **yalnızca hedef modülün `*.Contracts` projesindeki
  arayüzlerle** yapılır (ör. `Leave` → `ICalendarService.CountWorkingDays(...)`,
  `Leave` → `IEmployeeDirectory.GetManagerOf(...)`). İlk sürümde mesaj kuyruğu ya da event bus yok.
- Bağımlılık yönü tek yönlüdür ve döngü yoktur: `Leave → Organization.Contracts`,
  `Leave → Calendar.Contracts`. Organization ve Calendar başka modüle bağımlı değildir.
- Bir işlem tek bir modülün DbContext'i içinde biter. Modüller arası dağıtık transaction yoktur.
- Web, modüllere yalnızca Contracts üzerinden erişir: sayfalar ve API uçları ince kalır,
  doğrulama ve yetki kontrolü modülün Application katmanındadır.

## Forbidden dependencies (make them testable)
`tests/PolsanLeaveManager.Architecture.Tests` (NetArchTest, `Category=Architecture`, fast profil) her birini doğrular:
- **A1** — Bir modül başka bir modülün yalnızca `*.Contracts` assembly'sine referans verir; iç
  tiplerine, DbContext'ine ya da şemasına/tablolarına asla erişmez.
- **A2** — Bir modülün `*.Domain` namespace'i `Microsoft.EntityFrameworkCore`, `Microsoft.AspNetCore`
  ve `Microsoft.AspNetCore.Components` (Blazor) tiplerine bağımlı değildir.
- **A3** — Web iş kuralı içermez: Web'in modül projesinden kullanabileceği **tek** tip, DI kaydı
  için public `<Module>ModuleRegistration` sınıfıdır (`Add<Module>Module()`); geri kalan her erişim
  `*.Contracts` üzerindendir. Modül iç tipleri (`*.Domain`, `*.Application`, `*.Infrastructure`)
  `internal`'dır, bu yüzden derleyici de bu kuralı zorlar.
- **A4** — Hiçbir modül ve hiçbir `*.Contracts` projesi Web'e referans vermez.
- **A5** — `*.Contracts` yalnızca SharedKernel'e bağımlıdır (EF Core, ASP.NET yok); SharedKernel hiçbir proje bağımlılığı taşımaz.
- **A6** — Modüller arası referans grafiğinde döngü yoktur.

## Deliberately out of scope
İlk sürümde bilinçli olarak yapılmayanlar:
- Bordro entegrasyonu, Outlook/takvim senkronizasyonu, e-posta/anlık bildirim.
- Çok kademeli onay (yönetici + İK), vekâlet/devir.
- İK raporları ve ekip izin takvimi görünümü (sonraki sürüm adayı).
- Sağlık raporu ya da belge eki saklama (KVKK özel nitelikli veri, `docs/security.md`).
- Cumartesi çalışılan lokasyonlar ve lokasyon bazlı çalışma haftası.
- Mobil (Flutter) istemci, çoklu şirket/tenant.

# ADR 0001 — .NET 10 üzerinde modüler monolit, PostgreSQL ve Blazor

- Status: Accepted
- Date: 2026-10-19

## Context
PolsanLeaveManager, kurum içi bir izin yönetim sistemidir. İlk sürümün kapsamı çekirdek izin
akışıdır: talep → doğrudan yönetici onayı → hareket defterinden türetilen bakiye. Proje eğitim
amaçlı başlıyor ve küçük bir ekip tarafından geliştirilecek. Polsan'ın mevcut platformu .NET'tir
(`PolsanPlatform`); makinelerde .NET 10 SDK kurulu, Docker Desktop mevcut. Kod GitHub'da
(`Okan-developer/polsanleacemanager`) tutuluyor. Kararlar bootstrap mülakatında insan tarafından verildi.

## Decision
Sistemi **.NET 10 (LTS) üzerinde tek bir ASP.NET Core host olarak deploy edilen modüler monolit**
olarak kuruyoruz. Modüller Organization, Leave ve Calendar'dır; her birinin kendi PostgreSQL şeması
ve DbContext'i vardır, birbirleriyle yalnızca `*.Contracts` arayüzleri üzerinden senkron konuşurlar.
Arayüz aynı host'ta Blazor Web App'tir. Sınırlar NetArchTest ile `scripts/check` içinde zorlanır
(`docs/architecture.md`, A1–A6).

## Consequences
- **Kazanç:** tek deploy, tek dil ve tek build; kurum içi .NET bilgisi yeniden kullanılır.
  Modül sınırları testle korunduğu için ileride bir modül ayrılabilir.
- **Kazanç:** PostgreSQL ücretsizdir, Testcontainers ile entegrasyon testleri gerçek veritabanında koşar.
- **Maliyet:** modül başına iki proje (iç + Contracts) ve mimari testler, düz bir katmanlı yapıya göre tören ekler.
- **Maliyet:** tam profil check Docker gerektirir; Docker'sız makinede yalnızca `--fast` yeşil olabilir.
- **Maliyet:** Polsan'ın diğer projelerinden farklı olarak SQL Server değil PostgreSQL; DBA/operasyon desteği teyit edilmeli.

## Alternatives considered
- **Klasik katmanlı yapı (PolsanPlatform gibi):** şirket içinde tanıdık, ama iş alanı sınırları
  bulanıklaşır ve modül ayrımı test edilemez.
- **Clean Architecture (Domain/Application/Infrastructure/Api projeleri):** bu ölçek için fazla tören;
  modül içi klasörler + `internal` görünürlük aynı korumayı daha ucuza verir.
- **Mikroservisler:** ekip ve kapsam için gereksiz operasyon yükü, dağıtık tutarlılık sorunu.
- **SQL Server:** kurum standardı olsaydı tercih edilirdi; lisans ve container kolaylığı PostgreSQL lehine.
- **Ayrı SPA (React) ya da Flutter istemci:** ikinci bir toolchain ve build; ilk sürümde değer katmıyor.

## Revisit triggers
- Bir modülün bağımsız ölçeklenme ya da ayrı ekip ihtiyacı doğarsa (modülü servise ayırma).
- Kurum, veritabanı olarak SQL Server'ı zorunlu kılarsa.
- Mobil istemci (Flutter) kapsamına girerse: API sözleşmesinin ayrı bir yüzey olarak ele alınması.
- Çok kademeli onay ya da bildirim gibi modüller arası asenkron akış ihtiyacı doğarsa (event/outbox).

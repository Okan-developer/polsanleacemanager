# Domain

> Bootstrap'ta belirlenen ortak dil (2026-10-19). Koddaki karşılık İngilizce tanımlayıcıdır;
> bir terim burada yoksa ajan kendi anlamını uydurur — önce buraya ekleyin.

## Ubiquitous language

| Term | Meaning | Notes / not to be confused with |
|---|---|---|
| Çalışan (`Employee`) | Sistemde izin talep eden kişi. İşe giriş tarihi ve doğum tarihi vardır. | Kullanıcı hesabı değil; kimlik IdP'dendir, Organization modülü çalışanla eşler. |
| Yönetici (`Manager`) | Bir çalışanın **doğrudan** yöneticisi; ekibinin taleplerini onaylar. | Üst zincirdeki yöneticiler onaycı değildir. |
| İK (`HR`) | İzin türlerini, hak tablosunu ve resmi tatilleri yöneten rol; yedek onaycı. | Bir departman değil, bir **rol**. |
| İzin türü (`LeaveType`) | İK'nın tanımladığı izin kategorisi (ör. Yıllık izin). Bakiye tüketir ya da tüketmez. | Yıllık izin (`Annual`) ilk ve zorunlu türdür. |
| Kıdem (`Seniority`) | İşe giriş tarihinden bugüne tamamlanmış **tam yıl** sayısı. | Yaş değil. |
| İzin hakkı (`Entitlement`) | Bir hak yılı için kazanılan yıllık izin günü (BR-1…BR-4). | Bakiye değil — bakiye kullanılanlar düşülmüş toplamdır. |
| Hak tablosu (`EntitlementPolicy`) | Kıdem aralığına göre gün sayısı. Varsayılanı 4857 s. İş Kanunu md.53. | İK artırabilir, yasal minimumun altına indiremez. |
| Hareket defteri (`LeaveLedger`) | Bakiyeyi değiştiren her olayın değişmez (append-only) kaydı. | Bakiye alanı **tutulmaz**; hareketlerden türetilir. |
| Hareket (`LedgerEntry`) | `Accrual` (hak ediş, +), `Usage` (kullanım, −), `Reversal` (iade, +), `Adjustment` (İK düzeltmesi, ±, gerekçeli). | — |
| Bakiye (`Balance`) | Bir çalışanın bir izin türündeki hareketlerinin toplamı. | — |
| Kullanılabilir bakiye (`AvailableBalance`) | Bakiye − **bekleyen** taleplerin süreleri toplamı. | Talep kontrolü buna göre yapılır (BR-6). |
| İzin talebi (`LeaveRequest`) | Çalışanın bir tarih aralığı için açtığı izin isteği. | Onaylanmadan bakiyeden düşmez. |
| Talep durumu (`LeaveRequestStatus`) | `Pending` (Beklemede), `Approved` (Onaylandı), `Rejected` (Reddedildi), `Cancelled` (İptal edildi). | Geçişler: aşağıdaki değişmezler. |
| Onaycı (`Approver`) | Bir talebi onaylamaya/reddetmeye yetkili kişi (BR-9). | — |
| İş günü (`WorkingDay`) | Pazartesi–Cuma arası, resmi tatil olmayan gün. | Takvim günü değil. |
| Resmi tatil (`PublicHoliday`) | Calendar modülündeki tatil günü; tam gün ya da yarım gün (arife öğleden sonrası). | — |
| İzin süresi (`Duration`) | Talebin tükettiği iş günü; 0,5 adımlıdır (`decimal`). | — |
| Yarım gün (`HalfDay`) | Sabah ya da öğleden sonra; 0,5 iş günü. | — |

## Business rules
- **BR-1** — Yıllık izin hakkı kıdeme göre: 1 ≤ kıdem < 5 → 14 gün; 5 ≤ kıdem < 15 → 20 gün; kıdem ≥ 15 → 26 gün (iş günü).
- **BR-2** — Hak ediş anında 18 yaşından küçük ya da 50 yaşından büyük çalışanın hakkı en az 20 gündür.
- **BR-3** — İK hak tablosunu artırabilir; BR-1/BR-2'nin yasal minimumlarının altına indiren değişiklik reddedilir.
- **BR-4** — Hak, her işe giriş yıldönümünde bir `Accrual` hareketi olarak yazılır. İlk yıl dolmadan yıllık izin hakkı yoktur.
- **BR-5** — İzin süresi = aralıktaki iş günleri (Pzt–Cum) − resmi tatiller; yarım günlük tatil 0,5 düşer. Talep başında ve sonunda yarım gün seçilebilir; en küçük birim 0,5'tir.
- **BR-6** — Bakiye tüketen türde talep, **kullanılabilir bakiyeyi** aşamaz; aşarsa `leave.insufficient_balance` ile reddedilir.
- **BR-7** — Süresi 0 olan talep (ör. tamamı hafta sonu/tatil) ve başlangıcı bitişinden sonra olan talep geçersizdir.
- **BR-8** — Çalışanın `Pending` ya da `Approved` bir talebiyle çakışan yeni talep reddedilir.
- **BR-9** — Onaycı, talep sahibinin doğrudan yöneticisidir; yöneticisi yoksa İK'dır. Kimse kendi talebini onaylayamaz/reddedemez.
- **BR-10** — Onay, talep süresi kadar bir `Usage` hareketi yazar; ret hareket yazmaz. Ret gerekçesi zorunludur.
- **BR-11** — Onaylanmış talep, izin başlangıç gününden **önce** iptal edilebilir; iptal, `Usage` kadar `Reversal` yazar. Başlamış izin iptal edilemez.
- **BR-12** — Bekleyen talep, sahibi tarafından geri çekilebilir (`Cancelled`); hareket yazılmaz.
- **BR-13** — Bakiye hiçbir zaman negatif olamaz (avans izin yoktur).
- **BR-14** — Kullanılmayan yıllık izin süresiz devreder; hak yanmaz.

## Key domain invariants
- Bakiye = hareket defterindeki hareketlerin toplamıdır; defter **append-only**'dir. Hata düzeltme yeni bir hareketle (`Reversal`/`Adjustment`) yapılır, hareket silinmez/güncellenmez.
- Hiçbir işlem bakiyeyi negatife düşüremez (BR-13).
- Durum geçişleri yalnızca: `Pending → Approved | Rejected | Cancelled`; `Approved → Cancelled` (başlangıçtan önce). `Rejected` ve `Cancelled` terminaldir.
- Yasal minimum hak (BR-1, BR-2) hiçbir yapılandırmayla delinemez.
- İzin süresi ve bakiye `decimal`'dır ve 0,5'in katıdır.

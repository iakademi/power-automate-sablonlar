# Onay Akışı

## Amaç

SharePoint listesine eklenen bir talep kaydını yöneticiye onaya göndermek, sonucu listeye yazmak ve talep sahibine e-posta ile bildirmek.

## Ön Koşullar

- SharePoint listesi `Talepler`; sütunlar: `Title` (Konu), `Aciklama` (Çok satırlı metin), `Tutar` (Sayı), `Yonetici` (Kişi), `Durum` (Seçim: Beklemede, Onaylandı, Reddedildi), `YoneticiYorumu` (Çok satırlı metin)
- `Durum` sütununun varsayılan değeri `Beklemede`
- Akışı çalıştıran hesabın Onaylar ve Office 365 Outlook bağlayıcılarını kullanma yetkisi

## Akış Özeti

```text
Öğe oluşturulduğunda
   -> Onay başlat ve bekle
   -> Koşul: Sonuç = Approve?
        Evet -> Öğeyi güncelle (Onaylandı) + e-posta
        Hayır -> Öğeyi güncelle (Reddedildi) + e-posta
```

## Adımlar

1. **Tetikleyici:** `SharePoint - Bir öğe oluşturulduğunda` (When an item is created). Site adresi ve liste adı: `Talepler`.

2. **Eylem:** `Onaylar - Başlat ve onay için bekle` (Start and wait for an approval)
   - Onay türü: `Onayla/Reddet - İlk yanıt verir` (Approve/Reject - First to respond)
   - Başlık: `Talep onayı: @{triggerOutputs()?['body/Title']}`
   - Atanan: `@{triggerOutputs()?['body/Yonetici/Email']}`
   - Ayrıntılar (Markdown desteklenir):
     ```text
     **Talep sahibi:** @{triggerOutputs()?['body/Author/DisplayName']}
     **Tutar:** @{formatNumber(float(triggerOutputs()?['body/Tutar']), 'N2', 'tr-TR')} TL
     **Açıklama:** @{triggerOutputs()?['body/Aciklama']}
     ```
   - Öğe bağlantısı: `@{triggerOutputs()?['body/{Link}']}`
   - Öğe bağlantısı açıklaması: `Talebi görüntüle`

3. **Eylem:** `Koşul` (Condition)
   - Sol: `Sonuç` (Outcome, önceki onay eyleminden dinamik içerik)
   - Operatör: `eşittir`
   - Sağ: `Approve`

4. **Evet dalı:**
   - `SharePoint - Öğeyi güncelle`: `Id` = tetikleyici `ID`; `Durum Değeri` = `Onaylandı`; `YoneticiYorumu` = onay eyleminden `Yanıtlar Yorumlar` (Responses Comments) veya şu ifade:
     ```text
     first(outputs('Onay_baslat_ve_bekle')?['body/responses'])?['comments']
     ```
   - `Office 365 Outlook - E-posta gönder (V2)`:
     - Kime: `@{triggerOutputs()?['body/Author/Email']}`
     - Konu: `Talebiniz onaylandı: @{triggerOutputs()?['body/Title']}`
     - Gövde: `Talebiniz @{first(outputs('Onay_baslat_ve_bekle')?['body/responses'])?['responder']?['displayName']} tarafından onaylandı.`

5. **Hayır dalı:** Aynı yapı; `Durum Değeri` = `Reddedildi`, konu `Talebiniz reddedildi`.

## Zaman Aşımı

Onay bekleyen akış varsayılan olarak 30 gün sonra zaman aşımına uğrar. Bunu kısaltmak için onay eyleminde `...` > `Ayarlar` > `Zaman aşımı` alanına ISO 8601 süre yazın, örneğin 3 gün için `P3D`. Zaman aşımı sonrası akış başarısız olmasın diye onay eyleminden sonra gelen eylemlerin çalışma koşulunu (**Run after**) `başarısız oldu` ve `zaman aşımına uğradı` için de açın veya onayı bir `Scope` içine alıp hata dalında listeyi `Beklemede` tutun ([en-iyi-uygulamalar.md](../en-iyi-uygulamalar.md) içindeki try-catch desenine bakın).

## Tutara Göre Çok Aşamalı Onay

Koşula ek bir yol ekleyerek yüksek tutarlarda ikinci onaylayıcıyı devreye alabilirsiniz:

```text
@greater(float(triggerOutputs()?['body/Tutar']), 10000)
```

İfade `true` ise ikinci bir `Başlat ve onay için bekle` eylemi (ör. bir üst yönetici) çalışır.

## Test

1. Listeye bir kayıt ekleyin.
2. Yöneticinin e-postasındaki veya Teams `Onaylar` uygulamasındaki isteği yanıtlayın.
3. Listede `Durum` değerinin güncellendiğini ve talep sahibine e-posta geldiğini doğrulayın.

## Sık Sorunlar

| Sorun | Neden | Çözüm |
| --- | --- | --- |
| `Yonetici/Email` boş | Kişi sütunu için alt alan seçilmedi | İfadeyi `.../Yonetici/Email` olarak yazın |
| Akış sürekli tetikleniyor | Aynı akışta öğe güncellenince "oluşturulduğunda" değil "değiştirildiğinde" tetikleyicisi kullanıldı | Tetikleyiciyi "oluşturulduğunda" yapın veya tetikleyici koşulu ekleyin |
| Onay kimseye ulaşmıyor | Atanan adresi hatalı | `Atanan` alanına tam e-posta adresi (UPN) yazın |

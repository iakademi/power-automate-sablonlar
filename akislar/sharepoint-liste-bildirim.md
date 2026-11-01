# SharePoint Liste Bildirimi

## Amaç

SharePoint listesinde bir öğe oluşturulduğunda veya durumu değiştiğinde ilgili kişilere ayrıntılı bir e-posta göndermek. Gereksiz bildirimi önlemek için yalnızca `Durum` alanı değiştiğinde çalışır.

## Ön Koşullar

- SharePoint listesi `Gorevler`; sütunlar: `Title`, `Atanan` (Kişi), `Durum` (Seçim: Yeni, Devam ediyor, Tamamlandı), `SonTarih` (Tarih), `Oncelik` (Seçim)
- Sürüm geçmişinin listede açık olması (`Liste ayarları > Sürüm oluşturma ayarları`): önceki değer karşılaştırması için gerekir

## Akış Özeti

```text
Öğe oluşturulduğunda veya değiştirildiğinde
   -> Önceki sürümün durumunu al
   -> Durum değiştiyse
        -> Atanan kişiye HTML e-posta
```

## Adımlar

1. **Tetikleyici:** `SharePoint - Bir öğe oluşturulduğunda veya değiştirildiğinde` (When an item is created or modified)

2. **Eylem:** `SharePoint - Sürüm geçmişini al` (Get changes for an item or a file / "Get changes" eylemi)
   - Kimlik: `ID`
   - Since (Başlangıç): `Tetikleyici Penceresi Başlangıç Belirteci` (Trigger Window Start Token)
   - Eylem adı: `Degisiklikleri_al`

   Bu eylem hangi sütunların değiştiğini `Has Column Changed: Durum` gibi boolean çıktılarla verir.

3. **Eylem:** `Koşul`
   - Sol: dinamik içerik `Has Column Changed: Durum`
   - Sağ: `true`

4. **Evet dalı:**
   - `Değişkeni başlat`: `varTarih` (Metin), değer:
     ```text
     formatDateTime(triggerOutputs()?['body/SonTarih'], 'dd.MM.yyyy', 'tr-TR')
     ```
   - `Office 365 Outlook - E-posta gönder (V2)`:
     - Kime: `@{triggerOutputs()?['body/Atanan/Email']}`
     - Konu: `[@{triggerOutputs()?['body/Durum/Value']}] @{triggerOutputs()?['body/Title']}`
     - Gövde (HTML kodunu görüntüle `</>` ile):
       ```html
       <p>Merhaba @{triggerOutputs()?['body/Atanan/DisplayName']},</p>
       <p><b>@{triggerOutputs()?['body/Title']}</b> görevinin durumu
       <b>@{triggerOutputs()?['body/Durum/Value']}</b> olarak güncellendi.</p>
       <ul>
         <li>Son tarih: @{variables('varTarih')}</li>
         <li>Öncelik: @{coalesce(triggerOutputs()?['body/Oncelik/Value'], 'Belirtilmedi')}</li>
         <li>Değiştiren: @{triggerOutputs()?['body/Editor/DisplayName']}</li>
       </ul>
       <p><a href="@{triggerOutputs()?['body/{Link}']}">Görevi aç</a></p>
       ```

## Tetikleyici Koşulu ile Gereksiz Çalışmayı Azaltma

Tetikleyicide `...` > `Ayarlar` > `Tetikleyici koşulları` alanına şu ifadeyi ekleyerek yalnızca tamamlanmamış görevlerde çalıştırabilirsiniz:

```text
@not(equals(triggerOutputs()?['body/Durum/Value'], 'Tamamlandı'))
```

## Gecikmiş Görevler İçin Zamanlanmış Varyant

Her sabah 08:30 (Türkiye saati) vadesi geçmiş görevleri hatırlatmak için:

1. Tetikleyici: `Zamanlanmış bulut akışı` (Recurrence), sıklık günlük, saat dilimi `(UTC+03:00) İstanbul`, saat `08:30`.
2. `SharePoint - Öğeleri al`: Filtre sorgusu:
   ```text
   Durum ne 'Tamamlandı' and SonTarih lt '@{formatDateTime(utcNow(), 'yyyy-MM-dd')}'
   ```
3. `Koşul`: `length(outputs('Ogeleri_al')?['body/value'])` büyüktür `0`.
4. `Create HTML table` ile liste oluşturun ve tek bir özet e-postada gönderin (her görev için ayrı e-posta göndermek yerine).

## Test

1. Listede bir görevin durumunu `Yeni` -> `Devam ediyor` yapın.
2. Çalıştırma geçmişinde koşulun `Evet` dalına gittiğini kontrol edin.
3. Başka bir sütunu (ör. `Title`) değiştirin; e-posta gönderilmemelidir.

## Sık Sorunlar

| Sorun | Çözüm |
| --- | --- |
| Her değişiklikte e-posta gidiyor | `Has Column Changed` koşulunu kullanın |
| Atanan kişi boş geliyor | Önce `empty(triggerOutputs()?['body/Atanan/Email'])` ile kontrol edin |
| Tarih bir gün geri görünüyor | SharePoint tarihleri UTC saklar; `convertTimeZone(..., 'UTC', 'Turkey Standard Time')` kullanın |

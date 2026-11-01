# Forms Yanıtlarını Excel'e Aktarma

## Amaç

Microsoft Forms'ta gönderilen her yanıtı OneDrive veya SharePoint'teki Excel tablosuna yeni satır olarak eklemek ve katılımcıya teşekkür e-postası göndermek.

## Ön Koşullar

- Bir Microsoft Forms formu, örnek soru adları: `Ad Soyad`, `E-posta`, `Eğitim`, `Not`
- OneDrive for Business veya SharePoint'te bir Excel dosyası; içinde **tablo olarak biçimlendirilmiş** (`Ekle > Tablo`) bir aralık. Tablo adı: `Katilimcilar`. Sütun başlıkları: `Tarih`, `AdSoyad`, `Eposta`, `Egitim`, `Not`

Excel bağlayıcısı yalnızca **tablolarla** çalışır; düz aralıkta satır eklenemez.

## Akış Özeti

```text
Yeni yanıt gönderildiğinde
   -> Yanıt ayrıntılarını al
   -> Tabloya satır ekle
   -> Teşekkür e-postası
```

## Adımlar

1. **Tetikleyici:** `Microsoft Forms - Yeni bir yanıt gönderildiğinde`; Form Kimliği: ilgili formu seçin.

2. **Eylem:** `Microsoft Forms - Yanıt ayrıntılarını al`
   - Form Kimliği: aynı form
   - Yanıt Kimliği: tetikleyiciden `Yanıt Kimliği` (List of response notifications Response Id)
   - Eylem adı: `Yanit_ayrintilari`

3. **Eylem:** `Excel Online (Business) - Tabloya satır ekle`
   - Konum: OneDrive for Business
   - Belge Kitaplığı: OneDrive
   - Dosya: `/Formlar/Katilimcilar.xlsx`
   - Tablo: `Katilimcilar`
   - Sütun eşlemeleri:

     | Sütun | Değer |
     | --- | --- |
     | Tarih | `formatDateTime(convertTimeZone(utcNow(), 'UTC', 'Turkey Standard Time'), 'dd.MM.yyyy HH:mm')` |
     | AdSoyad | `Ad Soyad` (Forms dinamik içeriği) |
     | Eposta | `E-posta` (Forms dinamik içeriği) |
     | Egitim | `Eğitim` (Forms dinamik içeriği) |
     | Not | `coalesce(outputs('Yanit_ayrintilari')?['body/rxxxxxxxxxx'], '')` |

   Not: Forms soru alanları ham halde `r` ile başlayan kimliklerle gelir. Dinamik içerik listesinden soru başlığını seçmek bu kimliği otomatik yazar.

4. **Eylem:** `Office 365 Outlook - E-posta gönder (V2)`
   - Kime: yanıttaki `E-posta` alanı
   - Konu: `Kaydınız alındı`
   - Gövde: `Merhaba @{outputs('Yanit_ayrintilari')?['body/responder']}, formu doldurduğunuz için teşekkür ederiz.`

   Yanıtlayıcı anonim ise `responder` alanı boştur; bu durumda `E-posta` sorusunu zorunlu yapın.

## Çok Seçenekli (Onay Kutusu) Soruları

Birden fazla seçim sorusu, JSON dizisi metni olarak gelir (ör. `["Excel","Power BI"]`). Excel'e okunabilir yazmak için:

```text
join(json(outputs('Yanit_ayrintilari')?['body/rYYYYYYYY']), ', ')
```

## Dosya Yükleme Sorusu

Dosya yükleme sorusu bir JSON metni döndürür. Dosya bağlantısını almak için `Parse JSON` eylemi kullanın:

```json
{
  "type": "array",
  "items": {
    "type": "object",
    "properties": {
      "name": { "type": "string" },
      "link": { "type": "string" },
      "id": { "type": "string" },
      "size": { "type": "number" }
    }
  }
}
```

Ardından `first(body('Parse_JSON'))?['link']` ifadesiyle ilk dosya bağlantısını Excel'e yazabilirsiniz.

## Test

1. Formu doldurup gönderin.
2. Akışın çalıştırma geçmişinde üç eylemin de yeşil olduğunu doğrulayın.
3. Excel tablosunda yeni satırı ve gelen e-postayı kontrol edin.

## Sık Sorunlar

| Sorun | Çözüm |
| --- | --- |
| Tablo listede görünmüyor | Dosyayı kaydedin; aralığı `Tablo` olarak biçimlendirin; eylemde dosya yolunu yeniden seçin |
| Sütunlar eşleşmiyor | Excel sütun başlıklarını değiştirdiyseniz eylemi silip yeniden ekleyin |
| Çift satır ekleniyor | Aynı formu dinleyen birden fazla akışı kontrol edin |
| Satır ekleme çok yavaş | Excel dosyasını başka biri açıksa kilitlenebilir; `Yeniden dene` ilkesini açın |

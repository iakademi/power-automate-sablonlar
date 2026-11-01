# E-posta Eklerini Kaydetme

## Amaç

Belirli bir konu başlığını içeren, eki olan gelen e-postaların eklerini SharePoint belge kitaplığına (veya OneDrive'a) kaydetmek; dosya adlarını tarih ekleyerek benzersiz hale getirmek.

## Ön Koşullar

- Office 365 Outlook hesabı
- SharePoint'te `Faturalar` adlı bir belge kitaplığı (veya OneDrive klasörü)
- İzin verilen uzantılar: `.pdf`, `.xlsx`, `.docx`

## Akış Özeti

```text
Yeni e-posta geldiğinde (eki olan, konusu filtreli)
   -> Her ek için (Apply to each)
        -> Uzantı izinli mi?
             Evet -> Dosya oluştur
```

## Adımlar

1. **Tetikleyici:** `Office 365 Outlook - Yeni bir e-posta geldiğinde (V3)`
   - Klasör: `Gelen Kutusu`
   - Ekleri Ekle: `Evet`
   - Yalnızca Ekli Olanlar: `Evet`
   - Konu Filtresi: `Fatura`
   - Ekleri Satır İçi Dahil Et: `Hayır` (imza resimlerini dışarıda bırakmak için)

2. **Eylem:** `Her birine uygula` (Apply to each), eylem adı: `Her_ek_icin`
   - Önceki adımlardan çıktı seç: tetikleyici `Ekler` (Attachments)

3. **Her_ek_icin içinde** `Koşul` (Condition), gelişmiş modda:
   ```text
   @or(
     endsWith(toLower(items('Her_ek_icin')?['name']), '.pdf'),
     endsWith(toLower(items('Her_ek_icin')?['name']), '.xlsx'),
     endsWith(toLower(items('Her_ek_icin')?['name']), '.docx')
   )
   ```

4. **Evet dalı:** `SharePoint - Dosya oluştur` (Create file)
   - Site Adresi: ilgili site
   - Klasör Yolu: `/Faturalar/@{formatDateTime(utcNow(), 'yyyy')}/@{formatDateTime(utcNow(), 'MM')}`  
     (Klasörler önceden yoksa `SharePoint - Klasör oluştur` ile oluşturun veya yalnızca `/Faturalar` kullanın)
   - Dosya Adı:
     ```text
     concat(formatDateTime(utcNow(), 'yyyyMMdd_HHmmss'), '_', items('Her_ek_icin')?['name'])
     ```
   - Dosya İçeriği: `items('Her_ek_icin')?['contentBytes']`

   Dosya adındaki yasak karakterleri (`\ / : * ? " < > |`) temizlemek için `replace` zinciri kullanın:
   ```text
   replace(replace(items('Her_ek_icin')?['name'], '/', '-'), ':', '-')
   ```

5. **Hayır dalı (isteğe bağlı):** Desteklenmeyen uzantı için kendinize Teams/e-posta bildirimi gönderin.

6. **Döngüden sonra (isteğe bağlı):** `Office 365 Outlook - E-postayı okundu olarak işaretle (V3)` ile mesajı `Message Id` üzerinden işaretleyin; işlenmiş mesajların izini sürmek kolaylaşır.

## Aynı Ad Çakışması

Zaman damgası öneki aynı saniyede gelen aynı adlı iki dosyayı da ayırmak için yeterli olmayabilir. Daha güvenli yöntem `guid()` ile kısa benzersiz son ek eklemektir:

```text
concat(substring(guid(), 0, 8), '_', items('Her_ek_icin')?['name'])
```

## Test

1. Konusunda "Fatura" geçen ve PDF eki olan bir e-postayı kendinize gönderin.
2. Akış çalıştıktan sonra SharePoint kitaplığında dosyanın oluştuğunu doğrulayın.
3. `.zip` ekli bir e-posta gönderin; kaydedilmediğini doğrulayın.

## Sık Sorunlar

| Sorun | Çözüm |
| --- | --- |
| Dosya bozuk açılıyor | `contentBytes` kullandığınızdan emin olun (içerik zaten Base64 ikili olarak gelir; ek `base64ToBinary` gerekmez) |
| İmza resimleri de kaydediliyor | `Ekleri Satır İçi Dahil Et` ayarını `Hayır` yapın |
| Akış çok sık çalışıyor | Konu ve gönderen filtresi ekleyin; `Kime` veya `Kimden` alanını kısıtlayın |
| Büyük dosyalar eksik | E-posta eki bağlayıcı sınırları vardır; çok büyük dosyalar için SharePoint bağlantısı isteyin |

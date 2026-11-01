# Teams Bildirimi

## Amaç

Bir olay gerçekleştiğinde (ör. yeni destek talebi) Microsoft Teams kanalına başlık, ayrıntı ve düğmeler içeren bir Uyarlanabilir Kart (Adaptive Card) göndermek; yüksek öncelikli kayıtlarda ilgili kişiyi etiketlemek.

## Ön Koşullar

- Kanal gönderme yetkisi olan bir Teams ekibi ve kanalı
- SharePoint listesi `DestekTalepleri`; sütunlar: `Title`, `Oncelik` (Seçim: Düşük, Orta, Yüksek), `Sorumlu` (Kişi), `Aciklama`

## Akış Özeti

```text
Öğe oluşturulduğunda
   -> Öncelik yüksek mi?
        Evet -> Kullanıcıyı etiketle + kart gönder (@mention)
        Hayır -> Basit mesaj gönder
```

## Yöntem 1: Basit Mesaj

1. **Tetikleyici:** `SharePoint - Bir öğe oluşturulduğunda`
2. **Eylem:** `Microsoft Teams - Bir kanalda ileti gönder` (Post message in a chat or channel)
   - Gönderen: `Akış botu` (Flow bot)
   - Gönderilecek yer: `Kanal`
   - Ekip ve Kanal: hedef kanal
   - İleti:
     ```html
     <b>Yeni destek talebi:</b> @{triggerOutputs()?['body/Title']}<br>
     Öncelik: @{triggerOutputs()?['body/Oncelik/Value']}<br>
     <a href="@{triggerOutputs()?['body/{Link}']}">Talebi aç</a>
     ```

## Yöntem 2: Kişiyi Etiketleme (@mention)

1. **Eylem:** `Microsoft Teams - Kullanıcı için etiket oluştur` (Get an @mention token for a user)
   - Kullanıcı: `@{triggerOutputs()?['body/Sorumlu/Email']}`
   - Eylem adı: `Etiket_olustur`
2. **Eylem:** `Kanalda ileti gönder`; ileti içine dinamik içerik olarak `Etiket` (Mention token) ekleyin:
   ```html
   @{outputs('Etiket_olustur')?['body/atMention']} yeni bir yüksek öncelikli talep atandı:
   <b>@{triggerOutputs()?['body/Title']}</b>
   ```

## Yöntem 3: Uyarlanabilir Kart

**Eylem:** `Microsoft Teams - Kanalda uyarlanabilir kart gönder` (Post adaptive card in a chat or channel)

Kart JSON'u (tasarım için [adaptivecards.io/designer](https://adaptivecards.io/designer) kullanılabilir):

```json
{
  "type": "AdaptiveCard",
  "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
  "version": "1.4",
  "body": [
    {
      "type": "TextBlock",
      "text": "Yeni Destek Talebi",
      "weight": "Bolder",
      "size": "Large"
    },
    {
      "type": "FactSet",
      "facts": [
        { "title": "Konu", "value": "@{triggerOutputs()?['body/Title']}" },
        { "title": "Öncelik", "value": "@{triggerOutputs()?['body/Oncelik/Value']}" },
        { "title": "Sorumlu", "value": "@{coalesce(triggerOutputs()?['body/Sorumlu/DisplayName'], 'Atanmadı')}" }
      ]
    },
    {
      "type": "TextBlock",
      "text": "@{coalesce(triggerOutputs()?['body/Aciklama'], 'Açıklama girilmedi')}",
      "wrap": true
    }
  ],
  "actions": [
    {
      "type": "Action.OpenUrl",
      "title": "Talebi aç",
      "url": "@{triggerOutputs()?['body/{Link}']}"
    }
  ]
}
```

Düğmeye basıldığında akışın devam etmesini istiyorsanız `Uyarlanabilir kart gönder ve yanıt bekle` (Post adaptive card and wait for a response) eylemini ve `Action.Submit` kullanın; yanıt, sonraki adımlarda dinamik içerik olarak gelir.

## Koşul: Yalnızca Yüksek Öncelik

Tetikleyici koşulu ile gereksiz akış çalıştırmalarını önleyin:

```text
@equals(triggerOutputs()?['body/Oncelik/Value'], 'Yüksek')
```

## Zamanlanmış Özet Bildirimi

Her iş günü 09:00'da açık talep sayısını bildirmek için:

1. Tetikleyici: `Recurrence`; sıklık: Hafta; haftanın günleri Pazartesi-Cuma; saat `09:00`; saat dilimi `(UTC+03:00) İstanbul`.
2. `SharePoint - Öğeleri al`: Filtre sorgusu `Durum ne 'Kapalı'`.
3. `Kanalda ileti gönder`:
   ```text
   concat('Açık talep sayısı: ', string(length(outputs('Ogeleri_al')?['body/value'])))
   ```

## Test

1. Yüksek öncelikli bir talep ekleyin; kanalda kartın göründüğünü ve düğmenin bağlantıyı açtığını doğrulayın.
2. Düşük öncelikli talep ekleyin; tetikleyici koşulu nedeniyle akışın çalışmadığını doğrulayın.

## Sık Sorunlar

| Sorun | Çözüm |
| --- | --- |
| İleti "Akış botu" adıyla geliyor | Gönderen olarak `Kullanıcı` seçeneği, akış sahibi adına gönderir |
| Kart görünmüyor | JSON'daki `@{...}` tırnakları ve virgülleri kontrol edin; kart şemasını tasarımcıda doğrulayın |
| Etiket çalışmıyor | `Email` değeri kiracıdaki gerçek bir kullanıcıya ait olmalıdır |
| Kanal listede yok | Akış sahibinin ekibin üyesi olduğundan emin olun |

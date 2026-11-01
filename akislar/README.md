# Akış Tasarımları

Her dosya, tek bir senaryoyu tetikleyiciden son adıma kadar tarif eder. Standart şablon:

1. Amaç
2. Ön koşullar
3. Akış özeti (şema)
4. Adımlar (eylem adı, ayarlar, ifadeler)
5. Test
6. Sık sorunlar

| Dosya | Senaryo | Tetikleyici |
| --- | --- | --- |
| [onay-akisi.md](onay-akisi.md) | Talep onayı ve sonuç bildirimi | SharePoint: Bir öğe oluşturulduğunda |
| [sharepoint-liste-bildirim.md](sharepoint-liste-bildirim.md) | Liste değişikliğinde e-posta bildirimi | SharePoint: Bir öğe oluşturulduğunda veya değiştirildiğinde |
| [forms-excel.md](forms-excel.md) | Microsoft Forms yanıtlarını Excel tablosuna yazma | Forms: Yeni yanıt gönderildiğinde |
| [eposta-ek-kaydetme.md](eposta-ek-kaydetme.md) | Gelen e-posta eklerini klasöre kaydetme | Outlook: Yeni e-posta geldiğinde (V3) |
| [teams-bildirim.md](teams-bildirim.md) | Teams kanalına zengin kart bildirimi | Zamanlanmış veya olay tabanlı |

Eylem adlarını İngilizce arayüzdeki karşılıklarıyla verilmiştir; Türkçe arayüzde ad farklı görünebilir. Akış içindeki eylemlere, ifadelerde kolay başvurmak için anlamlı adlar verin (bkz. [en-iyi-uygulamalar.md](../en-iyi-uygulamalar.md)).

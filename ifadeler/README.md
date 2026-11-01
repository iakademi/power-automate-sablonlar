# İfadeler (Expressions)

Power Automate ifadeleri, Azure Logic Apps'in Workflow Definition Language dilini kullanır. Bir alana `İfade` sekmesinden yazılır; dinamik içerikle birlikte `@{...}` biçiminde metin içine gömülebilir.

| Dosya | İçerik |
| --- | --- |
| [tarih-ve-saat.md](tarih-ve-saat.md) | `utcNow`, `formatDateTime`, `addDays`, saat dilimi dönüşümü |
| [metin-ve-mantik.md](metin-ve-mantik.md) | `concat`, `replace`, `substring`, `if`, `coalesce`, `and`/`or` |
| [koleksiyon-json-xml.md](koleksiyon-json-xml.md) | `length`, `first`, `union`, `json`, `xml`, `xpath`, `body`, `outputs` |

## Genel Kurallar

- Metin değerleri **tek tırnak** içinde yazılır: `'Merhaba'`. Metin içinde tek tırnak gerekirse ikiye katlayın: `'Ali''nin'`.
- Fonksiyon adları büyük/küçük harf duyarlı değildir ancak okunabilirlik için camelCase yazılır.
- Önceki bir eylemin çıktısı: `outputs('Eylem_Adi')?['body/alan']`. `?` işareti alan yoksa hata yerine `null` döndürür.
- Eylem adındaki boşluklar ifadede alt çizgi (`_`) olur: "Get items" eylemi `outputs('Get_items')`.
- Tetikleyici çıktısı: `triggerOutputs()?['body/alan']` veya `triggerBody()?['alan']`.

# Koleksiyon, JSON ve XML İfadeleri

## Dizi (koleksiyon) işlemleri

| Amaç | İfade | Açıklama |
| --- | --- | --- |
| Eleman sayısı | `length(body('Get_items')?['value'])` | Liste eleman sayısı |
| İlk eleman | `first(body('Get_items')?['value'])` | Boş listede `null` |
| Son eleman | `last(body('Get_items')?['value'])` | |
| Belirli eleman | `body('Get_items')?['value']?[0]?['Title']` | Dizin 0'dan başlar |
| Dizi oluşturma | `createArray('a', 'b', 'c')` | |
| Birleşim | `union(variables('Liste1'), variables('Liste2'))` | Yinelenenler tek kalır |
| Kesişim | `intersection(variables('Liste1'), variables('Liste2'))` | |
| İlk N | `take(body('Get_items')?['value'], 5)` | |
| İlk N'i atla | `skip(body('Get_items')?['value'], 5)` | Sayfalama için |
| Ters çevirme | `reverse(variables('Liste'))` | |
| Sıralama | `sort(variables('SayiListesi'))` | Nesne dizisinde: `sort(dizi, 'AlanAdi')` |
| İçeriyor mu | `contains(variables('Liste'), 'Ankara')` | |
| Boş mu | `empty(body('Get_items')?['value'])` | |

## Nesne işlemleri

| Amaç | İfade |
| --- | --- |
| Alan okuma | `items('Her_birine_uygula')?['Ad']` |
| İç içe alan | `items('Her_birine_uygula')?['Yonetici']?['DisplayName']` |
| Alan yoksa varsayılan | `coalesce(items('Her_birine_uygula')?['Not'], '-')` |
| Nesne ekleme | `addProperty(variables('Nesne'), 'Durum', 'Yeni')` |
| Nesne güncelleme | `setProperty(variables('Nesne'), 'Durum', 'Kapalı')` |
| Alan silme | `removeProperty(variables('Nesne'), 'Gecici')` |

## Önceki eylem çıktıları

| İfade | Anlamı |
| --- | --- |
| `body('Eylem_Adi')` | Eylemin gövde çıktısı |
| `outputs('Eylem_Adi')` | Eylemin tüm çıktısı (durum kodu, başlıklar, gövde) |
| `outputs('Eylem_Adi')?['statusCode']` | HTTP durum kodu |
| `actions('Eylem_Adi')?['status']` | `Succeeded`, `Failed`, `Skipped`, `TimedOut` |
| `result('Scope_Adi')` | Scope içindeki eylemlerin sonuç listesi (hata yönetiminde önemli) |
| `workflow()?['run']?['name']` | Çalıştırma kimliği |
| `workflow()?['tags']?['flowDisplayName']` | Akışın görünen adı |

## JSON

### Metinden JSON nesnesine

```text
json('{"ad":"Ayşe","yas":30}')
```

Sonra alanlara şöyle erişilir:

```text
json(outputs('Compose_Metin'))?['ad']
```

### Parse JSON eylemi için örnek şema

`HTTP` yanıtından gelen şu gövde için:

```json
{ "id": 1, "ad": "Örnek", "aktif": true, "etiketler": ["a", "b"] }
```

Şema:

```json
{
  "type": "object",
  "properties": {
    "id": { "type": "integer" },
    "ad": { "type": "string" },
    "aktif": { "type": "boolean" },
    "etiketler": { "type": "array", "items": { "type": "string" } }
  }
}
```

### Nesneyi metne çevirme

```text
string(outputs('Compose_Nesne'))
```

## Dizi seçme ve filtreleme

Veri İşlemleri eylemlerinin ifade karşılıkları:

| Amaç | Eylem | Not |
| --- | --- | --- |
| Alan seçme/dönüştürme | `Select` | `From`: dizi; `Map`: `item()?['Ad']` |
| Süzme | `Filter array` | `item()?['Durum']` eşittir `Açık` |
| Tabloya çevirme | `Create HTML table` | E-posta gövdesi için |
| Birleştirme | `Join` | Diziyi ayırıcıyla metne çevirir |

`Filter array` gelişmiş modunda:

```text
@and(equals(item()?['Durum'], 'Açık'), greater(item()?['Oncelik'], 2))
```

## Tekil değerler (distinct)

Power Automate'in tek adımlı `distinct` fonksiyonu yoktur; kolay yöntem `union` ile kendi kendine birleştirmektir:

```text
union(body('Select_Sehirler'), body('Select_Sehirler'))
```

## XML ve xpath

### Metni XML'e çevirme

```text
xml(json('{"siparis":{"no":"A-100","tutar":250}}'))
```

Sonuç:

```xml
<siparis><no>A-100</no><tutar>250</tutar></siparis>
```

### xpath ile değer okuma

Elde edilen XML (`outputs('Compose_Xml')`) için:

```text
xpath(xml(outputs('Compose_Xml')), 'string(/siparis/no)')
```

Sonuç: `A-100`

Eleman sayısı:

```text
xpath(xml(outputs('Compose_Xml')), 'count(/liste/oge)')
```

Birden çok düğüm (dizi döner):

```text
xpath(xml(outputs('Compose_Xml')), '/liste/oge/ad/text()')
```

## Base64 ve URI

| Amaç | İfade |
| --- | --- |
| Base64'e çevir | `base64(outputs('Dosya_Icerigi'))` |
| Base64'ten metne | `base64ToString(variables('Kodlu'))` |
| URL kodlama | `encodeUriComponent('Ürün adı & kod')` |
| URL kod çözme | `decodeUriComponent('%C3%9Cr%C3%BCn')` |
| Dosya içeriği ikili | `base64ToBinary(variables('Kodlu'))` |
| Metni ikiliye | `binary('Merhaba')` |

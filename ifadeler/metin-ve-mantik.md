# Metin ve Mantık İfadeleri

## Metin işlemleri

| Amaç | İfade | Sonuç |
| --- | --- | --- |
| Birleştirme | `concat('Merhaba ', triggerBody()?['Ad'])` | `Merhaba Ayşe` |
| Büyük harf | `toUpper('rapor')` | `RAPOR` |
| Küçük harf | `toLower('RAPOR')` | `rapor` |
| Boşluk temizle | `trim('  metin  ')` | `metin` |
| Parça alma | `substring('Power Automate', 0, 5)` | `Power` |
| Değiştirme | `replace('rapor_2026.xlsx', '_', '-')` | `rapor-2026.xlsx` |
| Uzunluk | `length('Merhaba')` | `7` |
| İçeriyor mu | `contains('Fatura 2026', 'Fatura')` | `true` |
| Başlıyor mu | `startsWith('Fatura 2026', 'Fatura')` | `true` |
| Bitiyor mu | `endsWith('rapor.pdf', '.pdf')` | `true` |
| Bölme | `split('a;b;c', ';')` | `['a','b','c']` |
| Dizi birleştirme | `join(createArray('a','b','c'), ', ')` | `a, b, c` |
| Sondan N karakter | `substring(variables('Metin'), sub(length(variables('Metin')), 4), 4)` | son 4 karakter |
| Konum bulma | `indexOf('rapor.pdf', '.')` | `5` |
| Son ayırıcıdan sonrası | `last(split('klasor/alt/dosya.pdf', '/'))` | `dosya.pdf` |

## Dosya uzantısı ve adı

```text
last(split(triggerOutputs()?['body/Name'], '.'))
```

Uzantısız ad:

```text
substring(triggerOutputs()?['body/Name'], 0, lastIndexOf(triggerOutputs()?['body/Name'], '.'))
```

## Türkçe karakter notu

`toUpper('ikinci')` sonucu `IKINCI` olur (Türkçe `İ` değil). Türkçe duyarlı büyük harf gerekiyorsa önce `replace(..., 'i', 'İ')` uygulayıp ardından `toUpper` kullanın.

## Boş değer ve varsayılan değer

| Amaç | İfade |
| --- | --- |
| İlk dolu değeri al | `coalesce(triggerBody()?['Telefon'], triggerBody()?['Cep'], 'Belirtilmedi')` |
| Boş mu | `empty(triggerBody()?['Aciklama'])` |
| Null mu | `equals(triggerBody()?['Alan'], null)` |
| Boşsa varsayılan | `if(empty(triggerBody()?['Aciklama']), 'Açıklama yok', triggerBody()?['Aciklama'])` |

`coalesce`, `null` olmayan ilk değeri döndürür; boş metin (`''`) null değildir. Boş metni de yakalamak için `if(empty(...), ..., ...)` kullanın.

## Koşullar

| Amaç | İfade |
| --- | --- |
| Eşit mi | `equals(triggerBody()?['Durum'], 'Onaylandı')` |
| Büyük mü | `greater(triggerBody()?['Tutar'], 1000)` |
| Büyük veya eşit | `greaterOrEquals(int(triggerBody()?['Adet']), 5)` |
| Ve | `and(greater(variables('Tutar'), 100), equals(variables('Bolge'), 'Marmara'))` |
| Veya | `or(equals(variables('Durum'), 'Acil'), equals(variables('Durum'), 'Kritik'))` |
| Değil | `not(contains(variables('Eposta'), '@'))` |
| Koşullu değer | `if(greater(variables('Tutar'), 5000), 'Yönetici onayı', 'Otomatik onay')` |

## Sayı işlemleri

| Amaç | İfade |
| --- | --- |
| Toplama / çıkarma | `add(1, 2)`, `sub(10, 4)` |
| Çarpma / bölme | `mul(5, 3)`, `div(10, 4)` |
| Mod | `mod(10, 3)` |
| Yuvarlama | `div(round(mul(1234.5678, 100)), 100)` |
| Metni sayıya çevirme | `int('42')`, `float('3.14')` |
| Sayıyı metne çevirme | `string(42)` |
| Para biçimi | `formatNumber(1234567.891, 'N2', 'tr-TR')` -> `1.234.567,89` |
| Rastgele sayı | `rand(1, 100)` |
| Benzersiz kimlik | `guid()` |

## Örnek: Şablon metin üretme

```text
concat(
  'Sayın ', triggerBody()?['AdSoyad'], ',', decodeUriComponent('%0A'),
  formatNumber(float(triggerBody()?['Tutar']), 'N2', 'tr-TR'), ' TL tutarındaki talebiniz alınmıştır.'
)
```

`decodeUriComponent('%0A')` yeni satır karakteri üretir.

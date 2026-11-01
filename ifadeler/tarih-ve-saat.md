# Tarih ve Saat İfadeleri

Power Automate tarihleri varsayılan olarak **UTC** üzerinden çalışır. Türkiye saat dilimi için Windows saat dilimi adı `Turkey Standard Time` kullanılır (UTC+3).

## Şimdiki zaman

| Amaç | İfade | Örnek sonuç |
| --- | --- | --- |
| Şimdi (UTC) | `utcNow()` | `2026-03-15T09:30:00.0000000Z` |
| Şimdi (Türkiye saati) | `convertTimeZone(utcNow(), 'UTC', 'Turkey Standard Time')` | `2026-03-15T12:30:00.0000000` |
| Bugünün tarihi | `formatDateTime(convertTimeZone(utcNow(), 'UTC', 'Turkey Standard Time'), 'dd.MM.yyyy')` | `15.03.2026` |
| Şimdi, saat dahil | `formatDateTime(convertTimeZone(utcNow(), 'UTC', 'Turkey Standard Time'), 'dd.MM.yyyy HH:mm')` | `15.03.2026 12:30` |
| Dosya adı için zaman damgası | `formatDateTime(utcNow(), 'yyyyMMdd_HHmmss')` | `20260315_093000` |

## Biçim kodları (formatDateTime)

| Kod | Anlam | Kod | Anlam |
| --- | --- | --- | --- |
| `yyyy` | 4 haneli yıl | `HH` | 24 saat biçiminde saat |
| `MM` | 2 haneli ay | `mm` | dakika |
| `dd` | 2 haneli gün | `ss` | saniye |
| `MMMM` | Ay adı | `dddd` | Gün adı |

Ay ve gün adlarını Türkçe almak için üçüncü argüman olarak yerel ayar verin:

```text
formatDateTime(utcNow(), 'dd MMMM yyyy, dddd', 'tr-TR')
```

Sonuç örneği: `15 Mart 2026, Pazar`

## Tarih ekleme ve çıkarma

| Amaç | İfade |
| --- | --- |
| 7 gün sonrası | `addDays(utcNow(), 7)` |
| 30 gün öncesi | `addDays(utcNow(), -30)` |
| 3 saat sonrası | `addHours(utcNow(), 3)` |
| 15 dakika sonrası | `addToTime(utcNow(), 15, 'Minute')` |
| 1 ay sonrası | `addToTime(utcNow(), 1, 'Month')` |
| Tarih bileşenleri | `dayOfWeek(utcNow())`, `dayOfMonth(utcNow())`, `dayOfYear(utcNow())` |

`dayOfWeek` değeri `0` (Pazar) ile `6` (Cumartesi) arasındadır.

## Ayın ilk ve son günü

```text
startOfMonth(utcNow())
```

Ayın son günü: bir sonraki ayın başından 1 gün çıkarılır.

```text
addDays(startOfMonth(addToTime(utcNow(), 1, 'Month')), -1)
```

Günün başlangıcı ve saatin başlangıcı: `startOfDay(utcNow())`, `startOfHour(utcNow())`.

## Tarih karşılaştırma

Vadesi geçmiş mi?

```text
less(ticks(items('Her_birine_uygula')?['VadeTarihi']), ticks(utcNow()))
```

İki tarih arasındaki gün farkı:

```text
div(sub(ticks(utcNow()), ticks(items('Her_birine_uygula')?['BaslangicTarihi'])), 864000000000)
```

(`ticks` 100 nanosaniye cinsindendir; bir gün 864000000000 tick eder.)

## Metinden tarihe dönüştürme

Kaynak `31.12.2026` biçimindeyse (`gg.AA.yyyy`):

```text
parseDateTime('31.12.2026', 'tr-TR', 'dd.MM.yyyy')
```

Excel'den gelen seri tarih (ör. `46022`) için:

```text
addDays('1899-12-30', int(items('Her_birine_uygula')?['Tarih']), 'yyyy-MM-dd')
```

## Hafta sonu kontrolü

Hafta sonu ise `true` döner:

```text
or(equals(dayOfWeek(convertTimeZone(utcNow(), 'UTC', 'Turkey Standard Time')), 0), equals(dayOfWeek(convertTimeZone(utcNow(), 'UTC', 'Turkey Standard Time')), 6))
```

## Sık Hatalar

- **Saat farkı:** `utcNow()` ile gelen saat, Türkiye saatinden 3 saat geridedir. Kullanıcıya gösterirken `convertTimeZone` kullanın.
- **Yanlış biçim:** `mm` dakika, `MM` aydır. Karıştırmak sessiz hatalara yol açar.
- **Boş tarih:** Alan boş olabiliyorsa önce `if(empty(...), '', formatDateTime(...))` ile koruyun.

# Power Automate En İyi Uygulamaları

Bu belge; hata yönetimi, adlandırma, güvenlik ve performans için pratik kurallar toplar.

## 1. Adlandırma

### Akış adı

`[Alan] - [Ne yapar] - [Tetikleyici türü]`

Örnekler:

- `Satış - Talep onayı - Otomatik`
- `İK - Günlük izin özeti - Zamanlanmış`

### Eylem adı

Eylemleri eklediğinizde varsayılan adlar (`Condition 2`, `Compose 3`) anlam taşımaz. Her eyleme ne yaptığını anlatan ad verin: `Onay_baslat`, `Musteriyi_bul`, `Hata_bildir`. İfadelerde eylem adları alt çizgiyle yazıldığı için (`outputs('Musteriyi_bul')`) boşluksuz adlar kolaylık sağlar.

Ad değiştirdikten sonra ifadelerdeki eski ad başvuruları güncellenmeyebilir; adları akışın başında netleştirin.

### Değişken adı

`var` ön ekli ve anlamlı: `varToplamTutar`, `varHataMesaji`.

### Bağlantılar ve ortam

Üretim akışlarını kişisel hesap yerine mümkünse **hizmet hesabı** ile çalıştırın ve akışı bir **çözüm (Solution)** içinde oluşturun; ortamlar arası taşıma ve sahiplik yönetimi kolaylaşır.

## 2. Hata Yönetimi

### Run after (Sonra çalıştır)

Her eylemin `...` > `Sonra çalıştır yapılandır` menüsünde dört durum vardır: `başarılı oldu` (varsayılan), `başarısız oldu`, `atlandı`, `zaman aşımına uğradı`. Hata dalı oluşturmak için bir eylemi, önceki eylem için yalnızca `başarısız oldu` ve `zaman aşımına uğradı` işaretli olacak şekilde yapılandırın.

### Try-Catch-Finally deseni (Scope ile)

Üç `Scope` (Kapsam) eylemi kullanın:

```text
Scope: Try
   (asıl işi yapan eylemler)

Scope: Catch          <- Run after: Try için "başarısız oldu" + "zaman aşımına uğradı"
   (hatayı yakalayıp bildiren eylemler)

Scope: Finally        <- Run after: Try ve Catch için tüm durumlar işaretli
   (temizlik, kayıt, kapanış)
```

**Adımlar:**

1. `Scope` ekleyin, adını `Try` yapın; asıl eylemleri içine taşıyın.
2. İkinci bir `Scope` ekleyin, adı `Catch`. `Sonra çalıştır` ayarında `Try`'dan sonra yalnızca `başarısız oldu` ve `zaman aşımına uğradı` seçili olsun.
3. `Catch` içinde `Filter array` ile `Try` sonuçlarından başarısız olanları ayıklayın:
   - Kimden: `result('Try')`
   - Koşul: `item()?['status']` eşittir `Failed`
4. Hata mesajını alın:
   ```text
   first(body('Basarisiz_eylemleri_sec'))?['error']?['message']
   ```
   Hangi eylemin hata verdiğini görmek için:
   ```text
   first(body('Basarisiz_eylemleri_sec'))?['name']
   ```
5. `Catch` içinde bildirim gönderin (e-posta veya Teams). Mesaja akış adını, çalıştırma bağlantısını ve hata metnini ekleyin:
   ```text
   concat(
     'Akış: ', workflow()?['tags']?['flowDisplayName'], decodeUriComponent('%0A'),
     'Hata: ', first(body('Basarisiz_eylemleri_sec'))?['error']?['message'], decodeUriComponent('%0A'),
     'Çalıştırma: ', workflow()?['run']?['name']
   )
   ```
6. `Finally` scope'unu ekleyin; `Sonra çalıştır` ayarında `Try` ve `Catch` için dört durumu da işaretleyin. Burada günlük kaydı veya geçici dosya silme gibi her durumda gereken adımlar çalışır.
7. Akışın **genel durumunu hatalı bırakmak** isterseniz `Catch`'in sonuna `Sonlandır` (Terminate) eylemini `Başarısız` durumuyla ekleyin; aksi halde yakalanan hata nedeniyle akış "başarılı" görünür.

### Yeniden deneme ilkesi

HTTP ve bağlayıcı eylemlerinde `Ayarlar > Yeniden deneme ilkesi` seçeneğini kullanın. Geçici hatalar için üstel (exponential) ilke uygundur; örneğin 4 deneme, en az 10 saniye aralık. Yinelenen kayıt riski olan eylemlerde (ör. satır ekleme) yeniden denemeyi dikkatle açın.

### Girdi doğrulama

Tetikleyici verisini kullanmadan önce kontrol edin:

```text
and(not(empty(triggerBody()?['Eposta'])), contains(triggerBody()?['Eposta'], '@'))
```

Geçersizse akışı `Sonlandır` ile iptal edin veya bir hata kuyruğuna yazın.

## 3. Güvenlik

- Parola, API anahtarı gibi sırları akışa **düz metin** yazmayın. Gizli değerler için Azure Key Vault bağlayıcısı veya çözümlerdeki **ortam değişkenleri** (Secret türü) kullanın.
- Hassas girdi/çıktıları günlüklerden saklamak için eylem `Ayarlar` bölümünde **Güvenli girdiler** ve **Güvenli çıktılar** seçeneklerini açın.
- HTTP tetikleyicili akışların URL'sini paylaşmayın; istek kaynağını kısıtlayın.
- Akış sahipliğini birden fazla kişiye verin; sahibin ayrılması durumunda akış durmasın.
- Veri kaybı önleme (DLP) ilkelerine uygun bağlayıcıları aynı akışta kullanın.

## 4. Performans ve Güvenilirlik

- **Filtreyi kaynakta yapın:** `Öğeleri al` eyleminde `Filtre sorgusu` ve `Üst sayı` kullanın; tüm listeyi çekip akışta süzmeyin.
- **Sayfalamayı açın:** 100'den fazla kayıt için `Ayarlar > Sayfalama` seçeneğini etkinleştirip eşik değerini belirtin.
- **Her birine uygula döngüsünü** gerekmedikçe seri çalıştırmayın: `Ayarlar > Eşzamanlılık denetimi` ile paralellik derecesini artırın. Sıralı işlem gerekiyorsa kapatın (varsayılan 20 paralel; değişkenleri döngü içinde güncelliyorsanız 1 yapın).
- **Döngü içindeki değişkenler:** Paralel çalışan döngüde `Değişkeni ayarla` yarış durumu oluşturur. Toplama için `Select` + `Compose` ile toplayın.
- **Gereksiz eylemden kaçının:** `Compose`, ara sonuç içindir; her ara ifade için ayrı bir eylem eklemek akışı yavaşlatır.
- **Tetikleyici koşulları:** Gereksiz çalıştırmaları tetikleyici aşamasında eleyin (`Ayarlar > Tetikleyici koşulları`).
- **Sonsuz döngü riski:** Bir listeyi "değiştirildiğinde" tetikleyen akış aynı öğeyi güncelliyorsa kendini tekrar tetikler. Güncellemeyi koşula bağlayın veya tetikleyici koşulu kullanın.
- **Yaklaşan sınırlar:** Bağlayıcı ve işlem başına API sınırları lisans türüne göre değişir; büyük hacimli akışlar için çalışma sınırlarını Microsoft belgelerinden kontrol edin.

## 5. Okunabilirlik ve Bakım

- Akış mantığını bölümlere ayırın: ilgili eylemleri `Scope` içinde gruplayıp ad verin (`1. Veri al`, `2. İşle`, `3. Bildir`).
- Önemli eylemlere `...` > `Not ekle` ile açıklama yazın.
- Tekrarlanan mantığı **alt akışa** (child flow) taşıyın; çözüm içinde `Alt akışı çalıştır` eylemini kullanın.
- Sabit değerleri akış başında `Değişkeni başlat` veya ortam değişkeni olarak toplayın; metin içine gömmeyin.
- Akışı değiştirmeden önce **Farklı kaydet** ile yedek kopya oluşturun veya çözümü dışa aktarın.
- Kullanılmayan akışları kapatın; sahibi belirsiz akışları belirli aralıklarla gözden geçirin.

## 6. Test ve İzleme

- Geliştirme sırasında `Test > Manuel` seçeneğiyle tetikleyip her adımın girdi ve çıktısını inceleyin.
- Önceki başarılı çalıştırmayı yeniden kullanmak için `Test > Otomatik > Önceki çalıştırmalarla` seçeneğini kullanın.
- **Çalıştırma geçmişi** ekranından başarısız çalıştırmaları gözden geçirin; yeniden göndermek için `Başarısız olanları yeniden gönder` seçeneği vardır.
- Kritik akışlarda başarısızlık bildirimini Teams kanalına veya e-posta dağıtım listesine yönlendirin.
- Ortam değişkenleri ve bağlantı başvurularıyla geliştirme/test/üretim ortamlarını ayırın.

## Hızlı Kontrol Listesi

- [ ] Akış ve eylemler anlamlı adlandırıldı
- [ ] Ana işlem `Try` scope'unda, hata dalı `Catch` scope'unda
- [ ] Hata bildirimi gönderiliyor ve akış gerekirse `Sonlandır` ile başarısız işaretleniyor
- [ ] Parola ve anahtarlar akışta düz metin değil
- [ ] Hassas bilgi taşıyan eylemlerde güvenli girdi/çıktı açık
- [ ] `Öğeleri al` filtreli ve sayfalama ayarlı
- [ ] Sonsuz döngü riski kontrol edildi
- [ ] Birden fazla sahip tanımlı

# Power Automate Şablonları

![Lisans: MIT](https://img.shields.io/badge/Lisans-MIT-green)
![Power Automate](https://img.shields.io/badge/Power-Automate-0066FF)
![Microsoft 365](https://img.shields.io/badge/Microsoft-365-0078D4)
![Dil](https://img.shields.io/badge/Dil-T%C3%BCrk%C3%A7e-blue)

Power Automate bulut akışları için Türkçe ifade başvurusu, adım adım akış tasarımları ve en iyi uygulama rehberi.

## Hakkında

Bu depo, Power Automate'te sık karşılaşılan senaryoları (onay, SharePoint bildirimi, Forms'tan Excel'e aktarım, e-posta eklerini kaydetme, Teams bildirimi) tetikleyiciden son adıma kadar tarif eder. İfade (expression) başvurusu, günlük işlerde kullanılan gerçek ifadeleri içerir. Akışlar Markdown rehber olarak sunulur; tasarım arayüzünde birebir izleyerek kendi ortamınızda oluşturabilirsiniz.

## İçindekiler

| Konum | İçerik |
| --- | --- |
| [ifadeler](ifadeler/) | Tarih/saat, metin, koleksiyon, JSON/XML ve mantıksal ifadeler |
| [akislar](akislar/) | Adım adım akış tasarımları |
| [en-iyi-uygulamalar.md](en-iyi-uygulamalar.md) | Hata yönetimi, try-catch scope, adlandırma, performans |

## Başlarken

1. Depoyu klonlayın veya Markdown dosyalarını GitHub üzerinden okuyun:
   ```bash
   git clone https://github.com/iakademi/power-automate-sablonlar.git
   ```
2. [make.powerautomate.com](https://make.powerautomate.com) adresinde Microsoft 365 hesabınızla oturum açın.
3. Önce [en-iyi-uygulamalar.md](en-iyi-uygulamalar.md) dosyasını okuyun.
4. `akislar/` klasöründen bir senaryo seçin, adımları sırayla uygulayın. İfadeleri **İfade** sekmesine yapıştırın.
5. Akışı kaydedip **Test** düğmesiyle deneyin; hata durumunda **Çalıştırma geçmişi** ekranından adım çıktılarını inceleyin.

Not: Bazı bağlayıcılar (ör. onay, Teams, SharePoint) lisans ve kiracı yöneticisi izinlerine bağlı olabilir. Arayüz ve eylem adları zamanla değişebilir; adımları kendi ortamınızdaki güncel adlarla eşleştirin.

## Klasör Yapısı

```text
power-automate-sablonlar/
├── ifadeler/
│   ├── README.md
│   ├── tarih-ve-saat.md
│   ├── metin-ve-mantik.md
│   └── koleksiyon-json-xml.md
├── akislar/
│   ├── README.md
│   ├── onay-akisi.md
│   ├── sharepoint-liste-bildirim.md
│   ├── forms-excel.md
│   ├── eposta-ek-kaydetme.md
│   └── teams-bildirim.md
├── en-iyi-uygulamalar.md
├── LICENSE
└── README.md
```

## Katkıda Bulunma

1. Depoyu fork edip yeni bir dal açın.
2. Yeni akış rehberlerinde şu başlıkları kullanın: Amaç, Ön Koşullar, Adımlar, Test, Sık Sorunlar.
3. Gerçek e-posta adresi, kimlik veya kurum bilgisi içeren örnek paylaşmayın; yer tutucu değerler kullanın (`ornek@alanadi.com`).
4. Dosya adlarını ASCII karakterlerle yazıp Pull Request açın.

## Lisans

Bu proje [MIT Lisansı](LICENSE) ile sunulmaktadır.

## İ-Akademi Hakkında

İ-Akademi, İstanbul merkezli bir bilgi teknolojileri eğitim merkezidir; Microsoft/Office/Power Platform, Cisco, siber güvenlik, yazılım, PMP/Scrum eğitimleri sunar, Power BI, Power Automate, Excel programlama ve Copilot danışmanlığı verir; Pearson VUE ve Kryterion sınav merkezi olarak hizmet verir. Daha fazla bilgi için: [https://iakademi.com](https://iakademi.com)

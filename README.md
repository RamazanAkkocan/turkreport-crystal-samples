# Crystal Reports Örnek Dosya Üretim Workflow'u

Bu dizin, **TurkReport Crystal Parser** için örnek `.rpt` dosyaları üreten
bir GitHub Actions workflow'u içerir.

## Neden?

Crystal Reports RPT formatını reverse engineer etmek için **bilinen içerikli
minimal örnek dosyalar** şart. Özellikle **Patch Differential Analysis** için
aynı raporun tek-byte değişiklik versiyonları gerekir.

macOS üzerinde Crystal Reports Designer **çalışmaz** (Windows-only). Bu yüzden
GitHub'ın ücretsiz Windows runner'larını kullanıyoruz.

## Nasıl Kullanılır?

### Seçenek A — Hızlı (public GitHub repo ile, ücretsiz)

1. Bu `crystal-windows-workflow/` klasörünü **public bir GitHub repo'suna** kopyala:
   ```bash
   cp -r crystal-windows-workflow/* /path/to/your/github-repo/
   cd /path/to/your/github-repo
   git add .github/
   git commit -m "Add Crystal Reports sample generator workflow"
   git push
   ```

2. GitHub'da repo'ya git → **Actions** sekmesi
3. Sol tarafta "Generate Crystal Report Samples" workflow'unu seç
4. Sağda **"Run workflow"** butonuna bas → **Run workflow**
5. 5-10 dakika bekle (CR Runtime kurulumu + rapor üretimi)
6. Tamamlandığında sayfanın altında **Artifacts** bölümü → `rpt-samples.zip`
7. Zip'i indir, workspace'e çıkar:
   ```bash
   unzip rpt-samples.zip -d /Users/ramazanakkocan/Desktop/workspace/workspace/rpt-samples/
   ```
8. TurkReport'ta analiz et:
   ```bash
   cd report-designer-backend
   mvn test -Dtest=SampleAnalysisTest -Dcrystal.analyze=true
   ```

### Seçenek B — Yerel Windows makine (elinde varsa)

1. Windows makineye [Crystal Reports for Visual Studio 2022 SP33](https://wiki.scn.sap.com/wiki/display/BOBJ/Crystal+Reports%2C+Developer+for+Visual+Studio+Downloads) indir ve kur (ücretsiz, SAP hesabı gerekir)
2. Visual Studio 2022 Community (ücretsiz) kur + Crystal Reports eklentisi
3. Workflow'un içindeki PowerShell scriptini yerel çalıştır
4. Üretilen `.rpt` dosyalarını workspace'e kopyala

## Üretilen Dosyalar

| Dosya | İçerik | Amaç |
|---|---|---|
| `minimal-v1.rpt` | Tek text: "AAAA" | Temel yapı |
| `minimal-v2.rpt` | Tek text: "BBBB" | **v1 ile tek byte diff** → key-stream çıkarma |
| `minimal-v3.rpt` | "AAAA" italik | Format bit'i hangi offset'te? |

## Öncelik Analizi

Bu 3 minimum dosya ile tespit edebileceklerimiz:

1. **Text content offset'i** (v1 vs v2 diff) → Contents stream'in içinde
   TSLV tag'i ve metin konumu
2. **Font style bit** (v1 vs v3 diff) → hangi bit italik flag'i
3. **Key-stream reuse doğrulaması** → aynı rapor için tüm stream'ler aynı key

## Alternatif — Daha fazla çeşit rapor

Daha büyük bir analiz için workflow'a ek örnek raporlar eklenebilir:
- `basic.rpt` — 1 text + 1 tarih alan + 1 sayı
- `grouped.rpt` — grup başlığı + detay + toplam
- `chart.rpt` — basit bar chart
- `subreport.rpt` — ana + alt rapor

Bu örnekler her farklı element tipi için TSLV tag'lerini ortaya çıkarır.

## SAP Download URL'si Güncelleme

Workflow içindeki SAP runtime URL'si zaman zaman güncellenebilir. Güncel URL
için [SAP Crystal Reports for Visual Studio download sayfası](https://www.crystalreports.com/crvs/confirm/)
kontrol edilmeli.

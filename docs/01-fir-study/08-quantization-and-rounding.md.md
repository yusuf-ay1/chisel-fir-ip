# Quantization & Rounding

> scaling, kuantizasyon, rounding gibi bitwidth konularına dair her şey.

```text
├── 1. Bit Growth (Genişleme) Yönetimi
│   ├── Full Precision (Guard bits)
│   │   └── Her toplama/çarpmada bit sayısını büyüt, hiç kırpma yapma — en sona kadar taşı
│   ├── Fixed-point Scaling (kırpmalı)
│   │   └── Her aşamada belirli bitwidth'e sabitle, ara sonuçları kuantize et
│   └── Kurallar
│       ├── Adder: max(a,b) bit + 1 (carry/overflow için)
│       └── Multiplier: a_bits + b_bits (tam çarpım, kırpma yapılmazsa lossless)
│
├── 2. Scaling Stratejileri
│   ├── Fixed Scaling (Static)
│   │   └── Tasarım zamanında sabit shift miktarı — basit ama worst-case'e göre konservatif
│   ├── Block Floating Point (BFP)
│   │   └── Bir blok/frame içindeki tüm örnekler ortak exponent paylaşır; dinamik shift ile MSB'ye hizalanır
│   ├── Scale Factor Tracking (per-stage scaling)
│   │   └── CIC/FFT gibi çok kademeli yapılarda her stage'in max kazancına göre ayrı shift — Hogenauer pruning (CIC için)
│   └── Automatic Gain Control benzeri dinamik scaling
│       └── Çalışma zamanında sinyal seviyesine göre adaptif shift 
│
├── 3. Rounding Teknikleri
│   ├── Truncation (Round toward -∞ / kırpma)
│   │   └── En basit, en ucuz (sadece bit at); DC bias oluşturur (negatif yönlü hata)
│   ├── Round Half Up (Round toward +∞ / convergent değil)
│   │   └── LSB'nin altına 0.5 ekleyip kırp; +yönlü DC bias
│   ├── Round Half Away From Zero
│   │   └── İşarete göre simetrik yuvarlama; DC bias yok ama donanımda ekstra sign-check gerekir
│   ├── Round Half to Even (Banker's Rounding / Convergent Rounding)
│   │   └── Tam ortadaki (.5) değerlerde en yakın çift sayıya yuvarlar; istatistiksel bias'ı minimize eder, IEEE-754 default'u
│   ├── Round Half to Odd
│   │   └── Nadiren kullanılır, sadece belirli overflow senaryolarında (round-to-even'ın ürettiği tekrarlı pattern'i kırmak için)
│   ├── Stochastic Rounding
│   │   └── Yuvarlama yönünü rastgele (olasılıksal, hata büyüklüğüyle orantılı) seç — ML/quantization'da bias'ı sıfırlar ama donanımda RNG gerektirir
│   └── Jamming / Von Neumann Rounding
│       └── LSB'yi her zaman 1 yap (kırpılan bitlerden bağımsız) — DC bias'ı simetrik dağıtır, çok ucuz donanım
│
├── 4. Overflow / Saturation
│   ├── Wraparound (modular)
│   │   └── Overflow'da bit sarar (2's complement rollover) — en ucuz ama katastrofik hata (büyük genlik sıçraması) riski
│   ├── Saturation (clipping)
│   │   └── Overflow'da min/max değere kilitle — sinyal işleme için genelde tercih edilir (yumuşak bozulma)
│   └── Guard Bits ile Overflow Önleme
│       └── Yeterli MSB guard biti bırakarak overflow ihtimalini tasarım zamanında ele
│
├── 5. Hata Metrikleri (Error / Noise Metrics)
│   ├── Model: kuantizasyon hatası, uniform dağılım LSB'nin ±1/2'si aralığında
│   ├── SQNR (Signal-to-Quantization-Noise Ratio)
│   │   ├── Davranış: ≈ 6.02·N + 1.76 dB (N: bit sayısı, tam ölçek sinüs için) — sadece kuantizasyon gürültüsünü hesaba katar
│   │   └── Kullanım: bitwidth seçiminde hedef SNR'a göre gereken N'i belirlemek için
│   ├── SNR (Signal-to-Noise Ratio)
│   │   ├── Davranış: sinyal gücü / toplam gürültü gücü (termal, kuantizasyon, vs. dahil) — harmonikler hariç
│   │   └── Kullanım: gerçek ölçümlerde (simülasyon/donanım) genel gürültü tabanını değerlendirmek için
│   ├── SNDR / SINAD (Signal-to-Noise-and-Distortion Ratio)
│   │   ├── Davranış: sinyal gücü / (gürültü + harmonik distorsiyon) toplamı — SNR'dan daha kapsamlı
│   │   └── Kullanım: ADC/DAC ve FFT çıkış kalitesini tek sayıda özetlemek için, ENOB hesabının girdisi
│   ├── THD (Total Harmonic Distortion)
│   │   ├── Davranış: temel bileşenin gücü / harmonik bileşenlerin toplam gücü (genelde dB)
│   │   └── Kullanım: non-lineerlik kaynaklı bozulmayı izole etmek için (gürültüden ayrı)
│   ├── SFDR (Spurious-Free Dynamic Range)
│   │   ├── Davranış: temel bileşen ile en büyük spur (harmonik veya harmonik olmayan) arasındaki fark (dB)
│   │   └── Kullanım: radar/haberleşme gibi düşük seviyeli sinyalin büyük spur'lar arasında görünürlüğünün kritik olduğu sistemlerde
│   ├── ENOB (Effective Number of Bits)
│   │   ├── Davranış: ENOB = (SNDR − 1.76) / 6.02 — gerçek performansı "efektif bit sayısı"na çevirir
│   │   └── Kullanım: teorik N-bit tasarımın pratikte kaç bit performans verdiğini karşılaştırmak için (ADC/DAC datasheet standardı)
│   ├── Noise Shaping (Delta-Sigma tarzı)
│   │   ├── Davranış: kuantizasyon hatası feedback ile yüksek frekansa itilir
│   │   └── Kullanım: oversampling ile birlikte, bant-içi SNR'ı artırmak için (ADC/DAC tasarımlarında)
│   ├── Dithering
│   │   ├── Davranış: kuantizasyon öncesi küçük rastgele gürültü eklenir
│   │   └── Kullanım: deterministic pattern/idle-tone'ları kırmak için, ses/görüntü işlemede yaygın
│   ├── Processing Gain (FFT'ye özel)
│   │   ├── Davranış: FFT boyutu N için 10·log10(N/2) dB — coherent averaging ile gürültü tabanı düşer
│   │   └── Kullanım: spektral analizde efektif dinamik aralığı FFT boyutuna göre hesaplamak için
│   └── Bin/Leakage İlişkili Metrikler
│       ├── Noise Floor (dB/bin veya dBFS/Hz)
│       │   └── Kullanım: spektrumda gerçek sinyal ile gürültü tabanını ayırt etmek için referans seviye
│       └── Spectral Leakage
│           └── Kullanım: pencereleme (windowing) seçimiyle birlikte değerlendirilir, SFDR/THD ölçümünü etkiler
```
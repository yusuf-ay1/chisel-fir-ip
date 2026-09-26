---
durum: taslak
---

# Digital Design Sayı Gösterimleri

1. FP
2. Q
3. CSD
3. posit

## 1. Floating Point (Kayan Nokta)

{sign, exponent, mantissa}

- IEEE 754 formatları
  - **Half precision** (FP16) — 16 bit: 1 sign + 5 exponent + 10 mantissa. 
  - **Single precision** (FP32) — 32 bit: 1 + 8 + 23. Klasik "float".
  - **Double precision** (FP64) — 64 bit: 1 + 11 + 52. Klasik "double".
  - **bfloat16** — 16 bit ama 1 + 8 + 7
  - **Quad precision** (FP128) — 1 + 15 + 112

## 2. Fixed Point (Sabit Nokta)
- Qm.n gösterimi (m integer bit, n fractional bit); LSB = 2⁻ⁿ, sabit adım büyüklüğü
- FPGA DSP slice'larıyla (DSP48, vs.) native uyumlu → düşük alan/güç, deterministik gecikme
- Tasarım yükü: overflow/saturation yönetimi, quantization noise, word-growth analizi (özellikle FIR/IIR/FFT gibi çok kademeli yapılarda kritik)

## 3. Canonical Signed Digit (CSD)
- Signed-digit gösterimi: rakamlar {-1, 0, 1}, ardışık iki non-zero digit olamaz → minimum Hamming weight (bir sayının tüm signed-digit gösterimleri arasında en az non-zero terimli olanı)
- Amaç: sabit katsayılı çarpımı (constant multiplication) shift-add ağacına indirgeyip multiplier'sız (multiplier-less) implementasyon elde etmek
- Sabit çarpanlarda alan/güç tasarrufu sağlar
- MCM (multiple constant multiplication) + subexpression sharing ile CSD üzerine ek optimizasyon yapılabilir

## 4. Posit Numbers
- Floating point'e alternatif
- Değişken uzunlukta exponent alanı → tapered precision: 1'e yakın değerlerde yüksek hassasiyet, uç değerlerde (çok büyük/küçük) daha düşük hassasiyet
- Aynı bit genişliğinde IEEE 754'e göre genelde daha geniş dinamik aralık + daha iyi ortalama hassasiyet
- NaN yerine tek bir NaR (Not a Real) durumu; round-to-nearest-even default
- Donanım tarafı: değişken regime nedeniyle decode/encode IEEE FP'ye göre daha karmaşık

## Karşılaştırma Tablosu
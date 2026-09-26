---
durum: taslak
---

# Filter Design Methods

## Problem

FIR filter specification'ı elimizde olduğunda ($F_s$, passband, stopband, ripple, attenuation), asıl soru şu hale gelir:

```markdown
Customer
   │
   ▼
Filter Specification
   │
   ├── Fs
   ├── Passband
   ├── Stopband
   ├── Ripple
   └── Attenuation
   │
   ▼
  ???
   │
   ▼
h[0], h[1], ..., h[N-1]
```

"???" işte filter design algorithm'dır.

Burada iki ayrı problem birbirine karıştırılmamalı:

1. **FIR implementation:** — Verilen $h[n]$ coefficient'lerini hardware'de nasıl uygularım?
2. **FIR design:** — İstenen frequency response'u sağlayacak $h[n]$ coefficient'lerini nasıl bulurum?

Bu dosya FIR Design konusunu ele alıyor.

### İdeal Filtre Hayali ve Gerçeklik Duvarı

* **Brick-wall (Tuğla Duvar) Filtre:** İdeal bir low-pass filter aşağıdaki gibi tanımlanır:

$$ H(f)= \begin{cases} 1, & |f|\leq f_c\\ 0, & |f|>f_c \end{cases} $$

* **Sonsuzluk Problemi:** Bu filtrenin impulse response'u sonsuz uzunluktadır. İdeal low-pass'in impulse response'u sinc biçimindedir:

$$ h[n] = \frac{\sin(\omega_c(n-M))}{\pi(n-M)} $$

* Ama bizim FIR'ımız $N<∞$ olmak zorunda.

### Kesme Fikri (Truncation)

* **Kaba Kuvvet Çözümü:** Sonsuz uzanan sinc dalgasını uygun bir yerden keselim, elimizde sonlu bir FIR filtre kalır.

### Kesmenin Bedeli: Gibbs Fenomeni

- Pürüzsüz akan sonsuz bir dalgayı aniden kesip attığınızda, frekans domeninde keskin köşeler yüzünden istenmeyen **dalgalanmalar (ripple)** oluşur. 
- Bu bozulmaya **Gibbs fenomeni** denir. Filtre artık ideal olmaktan çıkar, sinyali bozar.

## Mathematical Foundation

* Dalgayı aniden kesmek yerine, kenarlara doğru **yumuşak bir şekilde sıfıra yaklaştıran** özel matematiksel eğriler (pencereler) kullanılır. Bu, dalgalanmaları en aza indirmek için kullanılan yaklaşımdır.

```text
FIR Filtre Tasarım Yöntemleri (Filter Design Methods)
│
├── 1. Pencereleme Metodu (Windowing)
│   ├── Davranış: İdeal (sonsuz) sinc yanıtını N uzunluğa kesip bir pencere fonksiyonuyla çarpar — optimizasyon yok, kapalı-form formül
│   ├── Kullanım: Basit, hızlı, öngörülebilir; equiripple'a göre aynı spec için daha fazla tap gerektirir
│   └── Pencere Tipleri
│       ├── Rectangular
│       │   ├── Davranış: w[n] = 1 — hiç şekillendirme yok, sert kesim
│       │   └── Kullanım: En dar geçiş bandı ama en yüksek ripple (~21 dB attenuation) — pratikte nadiren tercih edilir
│       ├── Hamming
│       │   ├── Davranış: w[n] = 0.54 − 0.46·cos(2πn/(N−1))
│       │   └── Kullanım: İyi denge — orta geçiş bandı, ~53 dB attenuation, en yaygın kullanılan pencere
│       ├── Hann (Hanning)
│       │   ├── Davranış: w[n] = 0.5 − 0.5·cos(2πn/(N−1))
│       │   └── Kullanım: Hamming'e yakın, biraz daha geniş geçiş bandı ama sidelobe'lar daha hızlı düşer
│       ├── Blackman
│       │   ├── Davranış: w[n] = 0.42 − 0.5·cos(2πn/(N−1)) + 0.08·cos(4πn/(N−1))
│       │   └── Kullanım: Çok düşük ripple (~74 dB attenuation) ama en geniş geçiş bandı — yüksek attenuation öncelikliyse
│       └── Kaiser
│           ├── Davranış: β parametresiyle ayarlanabilir (Bessel fonksiyonu tabanlı) — mainlobe genişliği/sidelobe attenuation trade-off'u sürekli ayarlanabilir
│           └── Kullanım: Diğer pencerelerin "yarı-optimize edilebilir" hali — tek formülle spec'e göre β seçilerek istenen attenuation hedeflenebilir
│
├── 2. Frekans Örnekleme Metodu (Frequency Sampling)
│   ├── Davranış: İstenen frekans yanıtı belirli noktalarda örneklenir, ters Fourier dönüşümüyle (IDFT) katsayılar elde edilir
│   └── Kullanım: Kavramsal olarak basit ama örnekler arası frekans yanıtı kontrolsüz salınabilir (zayıf stopband attenuation) — pratikte en az tercih edilen yöntem
│
├── 3. En Küçük Kareler Metodu (Least Squares)
│   ├── Davranış: İstenen ve elde edilen frekans yanıtı arasındaki hata karesini minimize eden bir lineer denklem sistemi (matris) çözülerek katsayılar bulunur — kapalı-form çözüm, iteratif değil
│   └── Kullanım: Windowing'den daha esnek ağırlıklandırma imkânı sunar (pasif/stop bandlara farklı önem verilebilir), equiripple kadar keskin değildir
│
└── 4. Parks-McClellan / Remez Exchange Algoritması (Equiripple)
    ├── Davranış: Hata, passband ve stopband boyunca eşit dalgalanacak (equiripple) şekilde iteratif olarak minimize edilir (minimax kriteri)
    └── Kullanım: Belirli bir spec için en az tap sayısıyla en keskin geçiş bandını sağlayan endüstri standardı yöntem
```


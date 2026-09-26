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

## Mathematical Foundation

### Kesme Fikri (Truncation)

* **Kaba Kuvvet Çözümü:** Sonsuz uzanan sinc dalgasını uygun bir yerden keselim, elimizde sonlu bir FIR filtre kalır.

### Kesmenin Bedeli: Gibbs Fenomeni

- Pürüzsüz akan sonsuz bir dalgayı aniden kesip attığınızda, frekans domeninde keskin köşeler yüzünden istenmeyen **dalgalanmalar (ripple)** oluşur. 
- Bu bozulmaya **Gibbs fenomeni** denir. Filtre artık ideal olmaktan çıkar, sinyali bozar.

### Çözüm: Window Method (Pencereleme)

* Dalgayı aniden kesmek yerine, kenarlara doğru **yumuşak bir şekilde sıfıra yaklaştıran** özel matematiksel eğriler (pencereler) kullanılır. Bu, dalgalanmaları en aza indirmek için kullanılan yaklaşımdır.

Temel fikir:

$$ h_{FIR}[n] = h_{ideal}[n]w[n] $$

Burada:

* $h_{ideal}[n]$ → ideal impulse response
* $w[n]$ → finite-length window



### Windowing types

detaylar windowing.md

1. Hamming
2. Hann
3. Blackman
4. Kaiser

Window, impulse response'u kenarlarda daha kontrollü şekilde sıfıra indiriyor.

#### Hamming Window

Hamming window, FIR tasarımında çok yaygın kullanılan klasik window'lardan biridir.

$$ w[n] = 0.54 - 0.46 \cos \left( \frac{2\pi n}{N-1} \right) $$ $$ 0\leq n\leq N-1 $$

#### Kaiser Window

Kaiser window, window method içerisinde daha fazla kontrol sağlar. Bir parametre ile $\beta$ window'un karakterini ayarlayabiliriz.

#### Parks–McClellan



#### Least-Squares FIR Design

Frequency response error'ının karesini minimize etmeye çalışırız.

$$ \min \int |H_{desired}(f)-H_{actual}(f)|^2 df $$

## Engineering Takeaway

| Method                | Temel fikir                     | Avantaj                              | Dezavantaj                             |
| --------------------- | ------------------------------- | ------------------------------------ | -------------------------------------- |
| Rectangular Window    | Direkt truncate                 | Çok basit                            | side lobes ripple yüksek               |
| Hamming/Hann/Blackman | Window ile kontrollü truncate   | Basit ve anlaşılır                   | Tasarım kontrolü sınırlı               |
| Kaiser                | Ayarlanabilir window            | Specification'a daha kolay uyarlanır | Optimal olmak zorunda değil            |
| Parks–McClellan       | Minimax/equiripple optimization | Çok verimli tasarımlar               | Daha karmaşık                          |
| Least-Squares         | Squared error minimization      | Esnek optimization                   | Maximum error garanti yaklaşımı farklı |

- $Δf↓⇒N↑$	​

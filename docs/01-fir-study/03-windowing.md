---
durum: taslak
---

# Windowing

> **status**: draft


## Problem

Sonsuz uzunluktaki ideal sinc impulse response'unu sonlu bir FIR'a indirgemek için bir noktada kesmek gerekiyor. Kesme işlemi (truncation) frekans domeninde Gibbs fenomenine yol açıyor = istenmeyen ripple.

Çözüm olarak sinc'i sert bir şekilde kesmek yerine, kenarlara doğru yumuşak biçimde sıfıra indiren bir window fonksiyonu ile çarpıyoruz

## Mathematical Foundation

### 1. Rectangular

* En basit (aslında hiç window kullanmama) durumu — sert kesme:

$$ w[n] = 1, \quad 0\leq n\leq N-1 $$

* **Trade off:** En dar mainlobe'a ama en yüksek sidelobe seviyesine (en kötü ripple) sahiptir.

### 2. Hamming

$$ w[n] = 0.54 - 0.46 \cos \left( \frac{2\pi n}{N-1} \right), \quad 0\leq n\leq N-1 $$

### 3. Hann

$$ w[n] = 0.5 \left(1 - \cos \left( \frac{2\pi n}{N-1} \right)\right), \quad 0\leq n\leq N-1 $$

### 4. Blackman

$$ w[n] = 0.42 - 0.5\cos\left(\frac{2\pi n}{N-1}\right) + 0.08\cos\left(\frac{4\pi n}{N-1}\right), \quad 0\leq n\leq N-1 $$

### 5. Kaiser

$$ w[n] = \frac{I_0\left(\beta\sqrt{1-\left(\frac{2n}{N-1}-1\right)^2}\right)}{I_0(\beta)}, \quad 0\leq n\leq N-1 $$

$I_0$: sıfırıncı dereceden modified Bessel function. $\beta$ parametresi window'un rectangular ile Blackman-benzeri arasında sürekli ayarlanmasını sağlar.

### 6. Parks–McClellan

### 7. Least-Squares

## Kaiser Formülü
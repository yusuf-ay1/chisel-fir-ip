# Filter Order

## Problem

Bu filtreyi kaç coefficient ile gerçekleştirebiliriz?

## Mathematical Foundation

FIR filter:

$$ y[n]=\sum_{k=0}^{N-1}h[k]x[n-k] $$

- $N$ → coefficient/tap sayısı
- $N-1$ → filter order

### Transition Width

Transition Width = Passband edge - Stopband edge

 $$\Delta f=f_s-f_p$$

Discrete-time filter design'da normalized angular transition width:

$$ \Delta\omega = 2\pi\frac{\Delta f}{F_s} $$

- $F_s$: sampling frequency
- $\Delta f$: transition bandwidth

### Kaiser Order Estimate

Kaiser window method için kullanılan yaygın yaklaşık filter-order ilişkisi:

$$N\approx \frac{A-8}{2.285\Delta\omega}$$

- $N$: yaklaşık filter order
- $A$: desired stopband attenuation, dB
- $\Delta\omega$: normalized angular transition width

Bu formül bir tahmin üretir; specification'ın kesin olarak karşılandığını kanıtlamaz.

### Example

bir örnek üzerinden gidelim. Tasarım isterleri:

```
Fs  = 100 MHz

Passband:
0 – 10 MHz

Stopband:
20+ MHz

Passband Ripple:
≤ 0.1 dB

Stopband Attenuation:
≥ 80 dB
```

- **Transition bandwidth**:

$$\Delta f=20-10=10\text{ MHz}$$

**Normalized transition width:**

$$ \Delta\omega = 2\pi\frac{10}{100} $$

$$ \boxed{\Delta\omega=0.2\pi} $$

$$ \boxed{\Delta\omega\approx0.6283} $$

- **Kaiser order estimate:**

$$N\approx\frac{80-8}{2.285(0.2\pi)}$$

yaklaşık olarak $ N\approx50.2 $ elde edilir = 51 order (50 tap). 

(_Eğer ki Transition = 2 MHz olsaydı order 250 civarı olurdu._)

## Engineering Takeaway

"Stopband'e geçiş çok keskin olsun" dediğimizde daha fazla tap anlamına gelir = hardware kaynak tüketimi ++
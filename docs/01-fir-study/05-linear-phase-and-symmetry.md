# Linear Phase & Symmetry

## Problem

Bir FIR filter'ın yalnızca magnitude response'u değil, phase response'u da önemlidir.

Bir discrete-time filter'ın frequency response'u:

$$
H(e^{j\omega}) = |H(e^{j\omega})|e^{j\phi(\omega)}
$$

Burada:

* $|H(e^{j\omega})|$ → magnitude response
* $\phi(\omega)$ → phase response

Bir filter farklı frequency components'larını farklı miktarlarda geciktirirse, sinyalin waveform'u bozulabilir.

Bu nedenle bazı signal-processing sistemlerinde **Linear phase response** önemli bir filter requirement'ıdır.

FIR filters'ın önemli avantajlarından biri, uygun coefficient symmetry ile exact linear phase elde edilebilmesidir.

## Mathematical Foundation

### Frequency Response

Bir FIR filter:

$$H(z)=\sum_{n=0}^{N-1}h[n]z^{-n}$$

şeklinde ifade edilir. 

Frequency response:

$$\sum_{n=0}^{N-1}h[n]e^{-j\omega n}$$

olarak elde edilir.

Genel olarak:

$$|H(e^{j\omega})|e^{j\phi(\omega)}$$

### Linear Phase

Linear phase için phase response:

$$\phi(\omega)=-\omega M$$

şeklindedir.

Burada (M) sabit bir delay'i temsil eder.

Group delay:

$$-\frac{d\phi(\omega)}{d\omega}$$

olarak tanımlanır.

Linear phase durumunda:

$$\tau_g(\omega)=M$$

olur.

Dolayısıyla:

$$\boxed{\text{Linear Phase} \Rightarrow \text{Constant Group Delay}}$$

Bu, filter'ın farklı frequency components'larını aynı miktarda geciktirmesine karşılık gelir.

## Engineering Intuition

Bir input signal'in farklı frequency components'ları olduğunu düşünelim.

Linear-phase bir filter'da:

```
1 MHz → aynı delay
3 MHz → aynı delay
7 MHz → aynı delay
9 MHz → aynı delay
```

uygulanabilir.

Buna karşılık frequency-dependent delay durumunda:

```
1 MHz →  8 ns
3 MHz → 11 ns
7 MHz → 15 ns
9 MHz → 19 ns
```
gibi farklı gecikmeler ortaya çıkabilir.

Bu durum signal waveform'unda distortion oluşturabilir.

Linear phase'in temel avantajı:

Frequency components arasındaki relative timing relationship'in korunmasıdır.

## Hardware Interpretation

Linear-phase FIR design'ın önemli sonuçlarından biri coefficient symmetry'dir.

Symmetric FIR için:

$$\boxed{h[n]=h[N-1-n]}$$

ilişkisi vardır.

Örneğin 7-tap symmetric FIR:

```text
h[0] h[1] h[2] h[3] h[4] h[5] h[6]

 A    B    C    D    C    B    A
```

şeklinde olabilir.

Yani:

$$h[0]=h[6]$$

$$h[1]=h[5]$$

$$h[2]=h[4]$$

olur.

Frequency response'u symmetry üzerinden gruplayarak:

$$H(e^{j\omega})=e^{-j3\omega}[h_3+2h_2\cos(\omega)+2h_1\cos(2\omega)+2h_0\cos(3\omega)]$$

şeklinde yazabiliriz.

Parantez içindeki ifade real-valued olduğundan phase'in temel bileşeni:

$$e^{-j3\omega}$$

olur.

Dolayısıyla:

$$\phi(\omega)=-3\omega$$

elde edilir.

Genel odd-length symmetric FIR için:

$$M=\frac{N-1}{2}$$

olur.

## Architecture Implications

Coefficient symmetry yalnızca DSP açısından teorik bir özellik değildir.

Hardware architecture'da arithmetic sharing yapılmasına olanak sağlayabilir.

7-tap FIR:

$$y[n]=Ax_0+Bx_1+Cx_2+Dx_3+Cx_4+Bx_5+Ax_6$$

şeklindeyse:

$$y[n]=A(x_0+x_6)+B(x_1+x_5)+C(x_2+x_4)+Dx_3$$

şeklinde gruplanabilir.

Böylece:

```text
x0 ─┐
    ├─► Pre-adder ─► × A  ─┐
x6 ─┘                      │
                           │
x1 ─┐                      │
    ├─► Pre-adder ─► × B  ─┤
x5 ─┘                      │
                           ├─► Adder Tree
x2 ─┐                      │
    ├─► Pre-adder ─► × C  ─┤
x4 ─┘                      │
                           │
x3 ────────────────► × D ──┘
```

şeklinde bir architecture oluşturulabilir.

7-tap naive implementation'da 7 multiplication gerekirken symmetric implementation'da:

$$\frac{7+1}{2}=4$$

unique coefficient multiplication yeterli olabilir.

Genel odd-length symmetric FIR için:

$$N=2M+1$$

ise unique coefficient sayısı:

$$M+1$$

olur.

## Linear-Phase FIR Types

Linear-phase FIR filters klasik olarak dört tipe ayrılır.

| Type     | Length | Symmetry      | Önemli özellik                                                   |
| -------- | ------ | ------------- | ---------------------------------------------------------------- |
| Type I   | Odd    | Symmetric     | Genel amaçlı low-pass/high-pass/band-pass için uygundur          |
| Type II  | Even   | Symmetric     | Nyquist frequency'de response zero olur                          |
| Type III | Odd    | Antisymmetric | Hilbert transformer/differentiator gibi uygulamalarda kullanılır |
| Type IV  | Even   | Antisymmetric | Hilbert transformer/differentiator gibi uygulamalarda kullanılır |

### Symmetric

$$h[n]=h[N-1-n]$$

### Antisymmetric

$$h[n]=-h[N-1-n]$$

Bu classification, filter design aşamasında hangi response'ların gerçekleştirilebileceğini etkiler.

## Type I

Odd number of taps ve symmetric coefficients:

```text
A B C D C B A
```

özelliğine sahiptir.

Type I FIR:

* odd length,
* symmetric coefficients,
* exact linear phase

özelliklerine sahiptir.

Genel amaçlı low-pass filter tasarımımız için doğal adaylardan biridir.

## Type II

Even number of taps ve symmetric coefficients:

```text
A B C D D C B A
```

şeklindedir.

Type II filter'ların önemli bir özelliği Nyquist frequency'de response'un zero olmasıdır:

$$H(e^{j\pi})=0$$

Bu nedenle bütün low-pass specifications için uygun değildir.

## Type III

Odd number of taps ve antisymmetric coefficients:

```text
A B C 0 -C -B -A
```

şeklinde düşünülebilir.

$$h[n]=-h[N-1-n]$$

ilişkisi vardır.

DC'de zero response gibi özel özelliklere sahiptir.

Hilbert transformer ve differentiator gibi uygulamalarda kullanışlıdır.

## Type IV

Even number of taps ve antisymmetric coefficients:

```text
A B C D -D -C -B -A
```

şeklindedir.

Yine:

$$h[n]=-h[N-1-n]$$

ilişkisi geçerlidir.

Hilbert transformer ve differentiator gibi özel filter structures için kullanılabilir.
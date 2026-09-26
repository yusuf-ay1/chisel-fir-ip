---
version: v1
---

# FIR Hardware Architecture

## Problem

FIR imp için çeşitli mimariler ve yaklaşımlar söz konusudur. Bu dökümanda mimariler incelenecektir.

## Mimariler

> ileride yeni mimarilerde eklenir. Şimdilik bunlar yeterli: direct (non-symmetric + symmetric) transpose(non-symmetric), sistolik(non-symmetric)

Aşağıdaki mimariler değerlendirmeye alınmıştır.

```text
FIR Filtre Mimarileri
│
├── 1. Direct Form
│   └── Tap delay line → çarpanlar → adder tree
│
├── 2. Transpoze Direct Form
│   └── Giriş broadcast → her tap'te çarpan + toplama
│       (DSP48 cascade'e uygun, en yaygın FPGA tercihi)
│
├── 3. Simetriye göre
│   ├── Non-symmetric
│   │   └── General FIR (linear phase yok, h[n] ≠ ±h[N-1-n])
│   ├── Symmetric (h[n] = h[N-1-n])
│   │   ├── Type I  — N tek  (odd length)
│   │   │   ├── Davranış: w=0 ve w=π'de sıfır zorunluluğu yok
│   │   │   └── Kullanım: en esnek tip — LP/HP/BP/BS her türlü filtre tasarlanabilir
│   │   └── Type II — N çift (even length)
│   │       ├── Davranış: w=π'de zorunlu sıfır
│   │       └── Kullanım: LP filtre için ideal, HP filtre için uygun değil
│   └── Antisymmetric (h[n] = -h[N-1-n])
│       ├── Type III — N tek  (merkezde h=0)
│       │   ├── Davranış: w=0 ve w=π'de zorunlu sıfır; 90° faz kayması
│       │   └── Kullanım: BP filtre ve differentiator'larda kullanılır
│       └── Type IV  — N çift (even length)
│           ├── Davranış: w=0'da zorunlu sıfır; 90° faz kayması
│           └── Kullanım: Hilbert transformer ve differentiator'larda tercih edilir, LP filtre için uygun değil
│
├── 4. Folded
│   └── Time multiplexing 
│
├── 5. Systolic Array
│   └── Local interconnect, komşu PE'ler arası veri + kısmi toplam
│       (Yüksek Fmax, ASIC routing congestion'ına iyi)
│
├── 6. Multiplierless (sabit/compile-time katsayı gerektirir)
│   ├── CSD (Canonic Signed Digit)
│   │   └── Shift-add zinciri (LUT/adder, DSP kullanmaz)
│   └── Distributed Arithmetic (DA)
│       └── Bit-seri + önceden hesaplanmış LUT
│
├── 7. Polyphase
│   └── Multi-rate (decimation/interpolation) için faz alt-filtrelere bölme
│
├── 8. Coefficient Configuration
│   ├── Fixed (compile-time) coefficients
│   │   └── Sentez zamanında sabitlenir — ROM/sabit-çarpıcı optimizasyonu, CSD/DA burada mümkün
│   └── Runtime-programmable coefficients
│       └── Çalışma zamanında yüklenebilir — coefficient RAM + genel amaçlı çarpıcı gerekir, Multiplierless (6) ile uyumsuz
│
```

```text
                              FIR
                               │
          ┌────────────────────┼────────────────────┐
          │                    │                    │
          ↓                    ↓                    ↓
       Direct              Transposed             Systolic
          │                    │                    │
     ┌────┴────┐          ┌────┴────┐          ┌────┴────┐
     │         │          │         │          │         │
     ↓         ↓          ↓         ↓          ↓         ↓
  Normal  Symmetric    Normal  Symmetric    Normal  Symmetric
     │         │          │         │
     └────┬────┘          └────┬────┘
          │                    │
          ↓                    ↓
     Folded (opt.)       Folded (opt.)
```

### 1. Direct Form

```mermaid
---
title: Direct Form | Pipelined | Non-symmetric | 4-Tap FIR
---
graph LR
    subgraph Gecikme Hattı / AREG
        X[x_n] --> D1[AREG / z^-1]
        D1 --> D2[AREG / z^-1]
        D2 --> D3[AREG / z^-1]
    end

    subgraph Çarpıcı & MREG
        X --> Mul0[× h0] --> M0[MREG]
        D1 --> Mul1[× h1] --> M1[MREG]
        D2 --> Mul2[× h2] --> M2[MREG]
        D3 --> Mul3[× h3] --> M3[MREG]
    end

    subgraph Toplama Ağacı & PREG
        M0 --> Add1((+))
        M1 --> Add1
        Add1 --> P1[PREG]
        
        M2 --> Add2((+))
        M3 --> Add2
        Add2 --> P2[PREG]

        P1 --> Add3((+))
        P2 --> Add3
        Add3 --> P3[PREG] --> Y[y_n]
    end
```




```mermaid
---
---
title: Direct Form | Pipelined | Symmetric | 4-Tap FIR
---
graph LR
    subgraph Gecikme Hattı & Pre-Adders
        X[x n] --> D1[AREG / z^-1]
        D1 --> D2[AREG / z^-1]
        D2 --> D3[AREG / z^-1]

        %% Simetrik uçların toplanması (Pre-adder)
        X --> AddPre1((+))
        D3 --> AddPre1
        
        D1 --> AddPre2((+))
        D2 --> AddPre2
    end

    subgraph Ön Toplayıcı Çıkış Regi / ADREG
        AddPre1 --> AD0[ADREG]
        AddPre2 --> AD1[ADREG]
    end

    subgraph Çarpıcı & MREG
        AD0 --> Mul0[× h0] --> M0[MREG]
        AD1 --> Mul1[× h1] --> M1[MREG]
    end

    subgraph Toplama Ağacı & PREG
        M0 --> AddFinal((+))
        M1 --> AddFinal
        AddFinal --> P1[PREG] --> Y[y n]
    end
```

### 2. Transposed Form

```mermaid
---
title: Transposed Form | Pipelined | Non-symmetric | 4-Tap FIR
---
graph LR
    subgraph Çarpıcı & MREG Kademesi
        X[AREG/x_n] --> Mul0[× h0] --> M0[MREG]
        X --> Mul1[× h1] --> M1[MREG]
        X --> Mul2[× h2] --> M2[MREG]
        X --> Mul3[× h3] --> M3[MREG]
    end

    subgraph Toplama ve Gecikme Hattı / PREG & z^-1
        M3 --> Add3((+)) --> Reg3[PREG / z^-1]
        M2 --> Add2((+))
        Reg3 --> Add2
        Add2 --> Reg2[PREG / z^-1]
        
        M1 --> Add1((+))
        Reg2 --> Add1
        Add1 --> Reg1[PREG / z^-1]
        
        M0 --> Add0((+))
        Reg1 --> Add0
        Add0 --> Reg0[PREG / z^-1] --> Y[y_n]
    end
```

> `Transposed Form | Pipelined | Symmetric | 4-Tap FIR` farklı bir mevzu. İleride geri dönüp bakılır. İlk etapta pas. (V2 konusu)

## 3. Folded Architecture

- Folding Factor (N): Bir fonksiyonel birimin kaç işlemi sırayla yaptığı.
    - Folding factor = 1 → tamamen parallel (unfolded)
    - Folding factor = K → K işlem tek çarpan/toplayıcıda sırayla yapılır

> ileride detaylandırılacak

## 5.6 Systolic Mimari

```text
                  ┌─────────────────────┐
  x_in  ─────────►│                     │─────────► x_out
                  │   PE (Processing    │
  sum_in ────────►│      Element)       │─────────► sum_out
                  │                     │
                  └─────────────────────┘
                            ▲
                            │
                    h (katsayı - sabit)
```

```mermaid
---
title: Systolic Form | Pipelined | Non-symmetric | 4-Tap FIR (PE Chain)
---
graph LR
    %% === Girişler ===
    XIN[x_in] --> PE0
    SUMIN[sum_in = 0] --> PE0

    %% ===================== PE0 =====================
    subgraph PE0["PE0  (tap 0)"]
        direction TB
        X0in[x_in] --> X0out[x_out]
        X0in --> Mul0["× h0"]
        Mul0 --> Add0((+))
        Sum0in[sum_in] --> Add0
        Add0 --> Sum0out[sum_out]
    end

    %% ===================== PE1 =====================
    subgraph PE1["PE1  (tap 1)"]
        direction TB
        X1in[x_in] --> RegX1["Reg<br/>z⁻¹"] --> X1out[x_out]
        X1in --> Mul1["× h1"]
        Mul1 --> Add1((+))
        Sum1in[sum_in] --> Add1
        Add1 --> Sum1out[sum_out]
    end

    %% ===================== PE2 =====================
    subgraph PE2["PE2  (tap 2)"]
        direction TB
        X2in[x_in] --> RegX2["Reg<br/>z⁻¹"] --> X2out[x_out]
        X2in --> Mul2["× h2"]
        Mul2 --> Add2((+))
        Sum2in[sum_in] --> Add2
        Add2 --> Sum2out[sum_out]
    end

    %% ===================== PE3 =====================
    subgraph PE3["PE3  (tap 3)"]
        direction TB
        X3in[x_in] --> RegX3["Reg<br/>z⁻¹"] --> X3out[x_out]
        X3in --> Mul3["× h3"]
        Mul3 --> Add3((+))
        Sum3in[sum_in] --> Add3
        Add3 --> Sum3out[sum_out]
    end

    %% === Bağlantılar ===
    PE0 -->|x_out| PE1
    PE0 -->|sum_out| PE1

    PE1 -->|x_out| PE2
    PE1 -->|sum_out| PE2

    PE2 -->|x_out| PE3
    PE2 -->|sum_out| PE3

    PE3 -->|sum_out| YOUT[y_n]
```
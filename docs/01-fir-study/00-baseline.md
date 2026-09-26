---
durum: taslak
---

# Base - main Doc

FIR filtre tasarım ve imp akışı:

```markdown
"I want this frequency response."
              │
              ▼
       Filter Specification
              │
              ▼
        Design Algorithm
              │
              ▼
       h[0] ... h[N-1]
              │
              ▼
       Quantized Coefficients
              │
              ▼
          Hardware
```

```
                        FIR
                         │
        ┌────────────────┼────────────────┐
        ↓                ↓                ↓
     Direct          Transposed        Systolic
        │                │
   ┌────┴────┐      ┌────┴────┐
   ↓         ↓      ↓         ↓
Normal  Symmetric Normal  Symmetric
           
               ↓
             Folded
```

## FIR filtre Taksonomi
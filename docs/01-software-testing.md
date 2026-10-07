# Software Testing

## Why Does This Matter?

Bir yazılımın compile olması ve normal koşullarda çalışması, requirement'ları doğru şekilde karşıladığı anlamına gelmez.

Industrial software development sürecinde sistemin yalnızca beklenen senaryolarda değil; boundary conditions, error conditions ve beklenmeyen durumlarda da nasıl davrandığının doğrulanması gerekir.

Software Testing'in temel amacı, yazılımın tanımlanan requirement'ları karşılayıp karşılamadığını sistematik ve tekrarlanabilir bir şekilde değerlendirmektir.

Bu nedenle testing yalnızca bug bulma süreci değildir. Aynı zamanda:

- requirement'ların doğruluğunu kontrol etmeye,
- beklenmeyen davranışları ortaya çıkarmaya,
- yapılan değişikliklerin mevcut sistemi bozup bozmadığını anlamaya,
- sistemin release için yeterli güven seviyesine ulaşıp ulaşmadığını değerlendirmeye

yardımcı olur.

---

## Core Concept

Software Testing'in temelinde iki şeyin karşılaştırılması vardır:

```text
Expected Result
       ↓
    Compare
       ↑
Actual Result
```
Eğer actual result, requirement tarafından tanımlanan expected result ile uyuşuyorsa test PASS olur.
Uyuşmuyorsa test FAIL olur.
Buradaki önemli nokta şudur:
Test sonucunu mühendisin kişisel yorumu değil, requirement belirlemelidir.

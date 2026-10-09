# Software Testing

## Contents

1. [Software Testing Nedir?](#software-testing-nedir)
2. [Temel Terminoloji](#temel-terminoloji)
3. [Gerçek Kod Örneği - Catch2](#gerçek-kod-örneği---catch2)
4. [Test Levels](#test-levels)
5. [Endüstride Test Süreci](#endüstride-test-süreci)
6. [Öğrendiklerim ve Kaynaklar](#öğrendiklerim-ve-kaynaklar)

---

## Software Testing Nedir?

Software Testing, bir yazılımın beklenen davranışları gösterip göstermediğini kontrol etmek ve olası hataları ortaya çıkarmak için kullanılan süreçtir.

Bir programın sorunsuz compile olması veya normal şartlarda çalışması, doğru çalıştığı anlamına gelmez. Önemli olan farklı input'lar ve çalışma koşulları altında nasıl davrandığıdır.

Örneğin bir motor kontrol uygulamasında hız komutunun doğru işlenmesi kadar, geçersiz hız değerleri veya communication failure durumlarında nasıl davranıldığı da önemlidir.

Testing ile Debugging arasındaki temel fark:

- **Testing:** Yazılımda bir problem olup olmadığını ortaya çıkarır.
- **Debugging:** Bulunan problemin neden oluştuğunu araştırır.

Testing, yazılımda hiç hata kalmadığını garanti etmez. Ancak hataların erkenden tespit edilmesine ve yazılım kalitesinin değerlendirilmesine yardımcı olur.

## Temel Terminoloji

| Terim | Açıklama |
|---|---|
| Requirement | Yazılımın karşılaması beklenen koşul veya davranış |
| Test Case | Belirli bir davranışı kontrol etmek için hazırlanan test senaryosu |
| Expected Result | Testten beklenen sonuç |
| Actual Result | Test sırasında elde edilen sonuç |
| PASS / FAIL | Testin başarılı veya başarısız olması |
| Bug / Defect | Yazılım içerisindeki hata |
| Regression Testing | Bir değişiklik sonrasında önceden çalışan özelliklerin bozulmadığının kontrol edilmesi |

Bu kavramlar basit görünse de test süreçlerinin temelini oluşturuyor.

## Gerçek Kod Örneği - Catch2

Testing yaklaşımını somut olarak anlamak için GitHub üzerinde bulunan **Catch2** projesindeki örneği seçtim.

Catch2, C++ projelerinde test yazmak için kullanılan bir testing framework'tür.

İncelediğim dosya: `examples/010-TestCase.cpp`

Örnekte `Factorial()` fonksiyonunun doğru çalışıp çalışmadığı test ediliyor.

Matematiksel olarak:

- `Factorial(0) = 1`
- `Factorial(1) = 1`
- `Factorial(2) = 2`
- `Factorial(3) = 6`

olması gerekiyor.

Ancak örnekteki fonksiyonun başlangıç koşulu hatalı tasarlanmış. `number <= 1` olduğunda doğrudan `number` döndürüyor. Bu nedenle `Factorial(0)` sonucu yanlışlıkla `0` oluyor.

Repository'deki ilgili test:

```cpp
TEST_CASE("Factorial of 0 is 1 (fail)", "[single-file]") {
    REQUIRE(Factorial(0) == 1);
}
```

Burada:

- `TEST_CASE` ile test tanımlanıyor.
- `Factorial(0)` test edilen fonksiyonu çağırıyor.
- `REQUIRE` ile sonucun `1` olması gerektiği kontrol ediliyor.

**Expected Result:** `1`

**Actual Result:** `0`

**Test Result:** `FAIL`

Bu, Catch2'nin kendi örneğinde bilerek oluşturulmuş bir başarısızlık senaryosu.

Hatanın nedeni anlaşıldığında fonksiyonun başlangıç koşulu düzeltilerek `Factorial(0)` için `1` döndürmesi sağlanabilir.

Ardından yalnızca sıfır değeri değil, daha önce doğru çalışan pozitif değerler de tekrar test edilmelidir.

Bu örnekten çıkardığım en önemli sonuç, normal input'larla başarılı çalışan bir fonksiyonun **boundary condition** nedeniyle yine de hatalı olabileceği.

> Test yazarken yalnızca yazılımın çalışmasını beklediğimiz durumları değil, hata yapmasının muhtemel olduğu sınırları da düşünmemiz gerekiyor.

## Test Levels

Software Testing farklı seviyelerde gerçekleştirilebilir.

| Test Level | Amaç |
|---|---|
| Unit Testing | Tek bir fonksiyonun veya component'ın kontrol edilmesi |
| Integration Testing | Birden fazla component'ın birlikte çalışmasının kontrol edilmesi |
| System Testing | Sistemin bütün olarak requirements'ı karşılayıp karşılamadığının değerlendirilmesi |
| Acceptance Testing | Ürünün kullanıcı veya müşteri beklentilerini karşılayıp karşılamadığının değerlendirilmesi |

Catch2 örneğinde yalnızca `Factorial()` fonksiyonunun davranışını kontrol ettiğimiz için **Unit Testing** gerçekleştiriliyor.

Endüstriyel bir robot uygulamasında ise bir sensor reading fonksiyonunun bağımsız olarak test edilmesi Unit Testing kapsamında olabilirken, Sensor Manager ile Controller arasındaki veri aktarımının kontrol edilmesi Integration Testing kapsamında değerlendirilebilir.

## Endüstride Test Süreci

Endüstride testing yalnızca developer'ın yazdığı kodu birkaç kez çalıştırmasından ibaret değildir. Farklı ekiplerin birlikte yürüttüğü, code changes'ın kontrol edildiği ve test sonuçlarının takip edildiği bir süreçtir.

Genel olarak süreç şu şekilde ilerleyebilir:

**Issue → Development → Merge Request → CI Pipeline & Code Review → Merge**

Burada özellikle iki kavram önemli:

- **Re-testing:** Bulunan bir bug düzeltildikten sonra aynı testin tekrar çalıştırılması.
- **Regression Testing:** Yapılan düzeltmenin yazılımın başka bölümlerini bozup bozmadığının kontrol edilmesi.

Testler manual veya automated olarak gerçekleştirilebilir. Özellikle sürekli tekrarlanması gereken Unit Testing ve Regression Testing için automation önemli avantaj sağlar.

Bu sürecin gerçek bir yazılım şirketinde nasıl uygulandığını anlamak için **GitLab'ın Engineering Handbook dokümantasyonunu ve açık Merge Request kayıtlarını inceledim.**

GitLab'da testing yalnızca ayrı bir QA ekibinin sorumluluğunda değil. Feature team'ler geliştirdikleri kodun testlerinden ve testlerin bakımından sorumlu tutulurken, ortak test altyapısı Developer Experience ekibi tarafından destekleniyor.

Bir developer değişiklik hazırladığında Merge Request üzerinden diğer mühendislerin incelemesine sunabiliyor. CI pipeline ile automated tests çalıştırılıyor ve sonuçlar merge öncesinde değerlendiriliyor.

Gerçek bir örnek olarak **GitLab Merge Request !218804** kaydını inceledim.

Bu Merge Request'te, dynamic input kullanan bir yapıda kullanıcı tarafından girilen değerin belirli koşullarda sıfırlanması problemi ele alınıyor.

Kayıtta:

- **5 Commits:** Kod değişikliklerinin geçmişi
- **10 Pipelines:** Değişikliklerle ilişkili CI pipeline kayıtları
- **6 Changed Files:** Değişiklik yapılan dosyalar

görülebiliyor.

Ayrıca Merge Request açıklamasında problemin nasıl doğrulanabileceğine ilişkin adımlar bulunuyor. Böylece yapılan değişikliğin hangi problemi çözmeye çalıştığı ve nasıl kontrol edileceği diğer mühendisler tarafından da anlaşılabiliyor.

Burada önemli olan commit veya pipeline sayısının yüksek olması değil; **kod değişikliklerinin, test süreçlerinin ve code review faaliyetlerinin kayıt altında tutulmasıdır.**

Bu örnek sayesinde profesyonel testing sürecinin yalnızca test yazmakla sınırlı olmadığını; version control, automated testing, code review ve ekip içi iş birliğiyle birlikte yürütüldüğünü görmek mümkün.

## Öğrendiklerim ve Kaynaklar

Software Testing araştırmasından çıkardığım temel sonuç, bir yazılımı test etmenin yalnızca programı çalıştırıp sonuçlarına bakmak olmadığıdır.

Özellikle dört nokta önemli:

1. **Requirement açık olmalı:** Neyi doğru kabul ettiğimizi bilmeden test tasarlamak zor.
2. **Boundary conditions önemli:** Normal input'larda çalışan bir yazılım sınır değerlerde başarısız olabilir.
3. **Testler tekrarlanabilir olmalı:** Bir bug düzeltildikten sonra aynı testin ve ilgili Regression Tests'in tekrar çalıştırılması gerekir.
4. **Testing ekip çalışmasının parçası:** Profesyonel projelerde testler code review, version control ve CI/CD süreçleriyle birlikte yönetilir.

Catch2 örneği üzerinden `TEST_CASE`, `REQUIRE`, Expected Result ve Actual Result kavramlarının gerçek bir C++ test dosyasında nasıl kullanıldığını inceleyebildim.

GitLab örneği ise daha büyük yazılım projelerinde testlerin ekip tarafından nasıl takip edildiğini ve kod değişiklikleriyle nasıl ilişkilendirildiğini anlamama yardımcı oldu.

### Kaynaklar

**Software Testing ve Catch2**

- [Catch2 - GitHub Repository](https://github.com/catchorg/Catch2)
- [Catch2 - İncelenen Kod Örneği](https://github.com/catchorg/Catch2/blob/devel/examples/010-TestCase.cpp)
- [Catch2 - Testing Tutorial](https://github.com/catchorg/Catch2/blob/devel/docs/tutorial.md)
- [ISTQB - Software Testing Terminology](https://glossary.istqb.org/)

**Industrial Testing Workflow**

- [GitLab Engineering - Testing](https://handbook.gitlab.com/handbook/engineering/testing/)
- [GitLab Testing Strategy](https://docs.gitlab.com/development/testing_guide/testing_strategy/)
- [GitLab Merge Request !218804 - Bug Fix Example](https://gitlab.com/gitlab-org/gitlab/-/merge_requests/218804)

---

[Back to Main Documentation](../../README.md)

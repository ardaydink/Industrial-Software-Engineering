## Requirements and Expected Behavior

Software Testing sürecinin başlangıç noktası, yazılımdan ne beklendiğinin açıkça tanımlanmasıdır. Bir sistemin doğru çalışıp çalışmadığını değerlendirebilmek için öncelikle **doğru davranışın ne olduğunu bilmemiz gerekir.**

Bu beklentiler çoğunlukla *requirements* üzerinden tanımlanır. Requirements, bir yazılımın gerçekleştirmesi gereken işlevleri, çalışma koşullarını ve uyması gereken kısıtlamaları belirtir.

Örneğin endüstriyel bir robot kontrol sistemi geliştirdiğimizi düşünelim. Sistem bir laser sensor'dan ölçüm almakta ve bu verileri motor kontrol algoritmasında kullanmaktadır.

İlk requirement şöyle olsun:

> Sensör bağlantısı kesildiğinde sistem hızlıca hata durumuna geçmelidir.

Bu ifade ilk bakışta anlaşılır görünse de testing açısından önemli bir eksikliği vardır: **"Hızlıca" ne kadar hızlı demektir?**

Bir mühendis için 10 ms hızlı olabilirken, başka bir uygulamada 500 ms kabul edilebilir olabilir.

Dolayısıyla bu requirement'ı daha ölçülebilir hale getirelim:

> **REQ-COM-001:** Sensör bağlantısı fiziksel olarak kesildikten sonra uygulama en geç 100 ms içerisinde `ERROR_STATE` durumuna geçmeli ve yeni motor hareket komutları üretmeyi durdurmalıdır.

Bu artık test edilebilir bir requirement'tır.

Ancak burada bile bazı detayların netleştirilmesi gerekebilir. Örneğin bağlantının kesildiği anın nasıl tespit edileceği, 100 ms sürenin hangi olaylar arasında ölçüleceği ve motorun fiziksel olarak nasıl güvenli duruma geçeceği ayrıca tanımlanmalıdır.

### Testable Requirements

Bir requirement'ın test edilebilir olması için beklenen davranışın gözlemlenebilir ve değerlendirilebilir olması gerekir.

Bu nedenle iyi tanımlanmış requirements genellikle üç temel soruya cevap verir:

- **Condition:** Hangi koşulda?
- **Expected Behavior:** Sistem ne yapmalı?
- **Acceptance Criteria:** Hangi şartlarda başarılı kabul edilmeli?

Örneğimizde:

| Alan | Tanım |
|---|---|
| Condition | Sensor connection lost |
| Expected Behavior | `ERROR_STATE` durumuna geçilmesi ve yeni hareket komutlarının engellenmesi |
| Acceptance Criteria | Bağlantının kesilmesinden itibaren en fazla 100 ms |
| Requirement ID | REQ-COM-001 |

Böylece test engineer kendi yorumuna göre değil, tanımlanmış acceptance criteria üzerinden karar verebilir.

### Expected Result and Actual Result

Testing sürecinde iki kavram özellikle önemlidir.

**Expected Result**, requirement veya başka bir test basis tarafından tanımlanan beklenen davranıştır.

**Actual Result**, test gerçekleştirildiğinde sistemde gözlemlenen gerçek davranıştır.

Örneğin:

| | Sonuç |
|---|---|
| Expected Result | 100 ms içerisinde `ERROR_STATE` |
| Actual Result | 145 ms içerisinde `ERROR_STATE` |
| Test Result | FAIL |

Bu örnekte sistem sonunda doğru duruma geçmiştir. Ancak bunu belirtilen deadline içerisinde gerçekleştiremediği için test başarısızdır.

Bu durum bize önemli bir mühendislik yaklaşımı gösterir:

**Yazılımın fonksiyonel olarak doğru çalışması, bütün requirements'ı karşıladığı anlamına gelmez.**

Özellikle industrial ve real-time systems içerisinde timing requirements da test sonucunun bir parçasıdır.

---

## Test Levels

Modern yazılım sistemleri genellikle tek bir fonksiyondan oluşmaz. Farklı modüller, services, communication interfaces ve hardware components birlikte çalışır.

Bu nedenle yalnızca bütün sistemi çalıştırarak test yapmak yeterli değildir. Bir problem ortaya çıktığında hatanın hangi modülden kaynaklandığını belirlemek zorlaşabilir.

Software Testing sürecinde farklı test levels kullanılmasının temel nedeni budur.

### Unit Testing

Unit Testing, yazılımın küçük ve bağımsız bileşenlerinin doğru çalışıp çalışmadığının değerlendirilmesidir.

Örneğin robot uygulamamızda motor hızını sınırlandıran bir fonksiyon olduğunu düşünelim.

Bu fonksiyonun görevi, negatif hız değerlerini sıfıra çekmek ve maksimum değerin üzerindeki komutları sınırlandırmak olsun.

Test sırasında fiziksel bir motor, CAN interface veya robot kontrol kartına ihtiyaç duymayabiliriz.

Yalnızca fonksiyonun farklı input'lara doğru output üretip üretmediğini kontrol ederiz.

| Input | Expected Output |
|---|---|
| 50 RPM | 50 RPM |
| 120 RPM | 100 RPM |
| -10 RPM | 0 RPM |

Burada temel amaç, küçük bir yazılım bileşenini diğer sistemlerden mümkün olduğunca bağımsız değerlendirerek hataları erken aşamada tespit etmektir.

Unit Testing genellikle hızlıdır ve automated testing için oldukça uygundur.

Ancak önemli bir sınırı vardır: **Bir fonksiyonun tek başına doğru çalışması, diğer modüllerle birlikte de doğru çalışacağını garanti etmez.**

### Integration Testing

Gerçek bir software application birçok modülün birlikte çalışmasıyla oluşur.

Örneğin:

Sensor Manager → Measurement Processor → Motor Controller

Bu üç modülün ayrı ayrı unit testlerinden geçtiğini düşünelim.

Sensor Manager doğru measurement değerlerini üretiyor olabilir. Measurement Processor kendi testlerinde doğru hesaplama yapıyor olabilir. Motor Controller da verilen komutları doğru yorumluyor olabilir.

Fakat bu modüller birbirlerine veri aktarırken bir problem oluşabilir.

Örneğin Sensor Manager sıcaklık bilgisini `0.1 °C` çözünürlükte integer olarak gönderirken, başka bir modül bu değeri doğrudan `°C` olarak yorumlayabilir.

Bu durumda ayrı ayrı testler başarılı olsa bile bütün sistem yanlış davranabilir.

**Integration Testing**, modüllerin aralarındaki interfaces ve interactions'ın doğru çalışıp çalışmadığını değerlendirir.

Burada test edilen yalnızca fonksiyonların kendi davranışları değil, birbirleriyle kurdukları ilişkidir.

### System Testing

System Testing aşamasında yazılımı artık bütün bir sistem olarak ele alırız.

Örneğin robot uygulamamızda aşağıdaki yapı bulunsun:

Sensor → Communication Layer → Processing → Controller → Motor Driver

Bu aşamada yalnızca `calculate_position()` fonksiyonunun doğru hesaplama yapmasıyla ilgilenmeyiz.

Sisteme verilen bir komutun beklenen sonuca ulaşıp ulaşmadığını kontrol ederiz.

Örnek bir requirement:

> Kullanıcı stop komutu verdiğinde sistem yeni hareket komutları üretmeyi durdurmalıdır.

System Testing kapsamında komutun kullanıcı arayüzünden başlayarak ilgili modüllere ulaşıp ulaşmadığı ve bütün sistemin beklenen davranışı gösterip göstermediği incelenebilir.

Ayrıca yalnızca functional requirements değil; performance, reliability ve resource usage gibi non-functional requirements da değerlendirilebilir.

### System Integration Testing

System Testing ile System Integration Testing arasında önemli bir fark vardır.

System Testing, incelenen sistemin bütün olarak davranışına odaklanır.

System Integration Testing ise bu sistemin diğer bağımsız sistemlerle veya external services ile kurduğu interactions'ı değerlendirir.

Örneğin robot kontrol uygulaması tek başına doğru çalışıyor olabilir. Ancak fabrika otomasyon sistemiyle haberleşirken communication protocol uyumsuzluğu ortaya çıkabilir.

Bu tür sorunlar System Integration Testing kapsamında ele alınabilir.

### Acceptance Testing

Bir sistemin teknik requirements'ı karşılaması, müşterinin veya son kullanıcının ihtiyacını mutlaka karşıladığı anlamına gelmez.

Acceptance Testing, sistemin kullanım amacı ve kabul kriterleri açısından değerlendirilmesine odaklanır.

Örneğin müşteri bir robotun belirli ağırlıktaki parçaları istenen hassasiyetle taşımasını bekliyorsa, acceptance criteria bu gerçek kullanım ihtiyacına göre oluşturulur.

Bu aşamada temel soru şudur:

**"Geliştirdiğimiz sistem, kullanıcının ihtiyaç duyduğu işi kabul edilebilir şekilde yapıyor mu?"**

Acceptance Testing, proje koşullarına göre müşteri, kullanıcı veya yetkili ekiplerin katılımıyla yürütülebilir.

### Test Levels Arasındaki İlişki

Test levels birbirlerinin alternatifi değildir. Her biri farklı türde problemleri tespit etmeye odaklanır.

| Test Level | Temel Soru |
|---|---|
| Unit Testing | Bileşen tek başına doğru çalışıyor mu? |
| Integration Testing | Modüller birbirleriyle doğru çalışıyor mu? |
| System Testing | Bütün sistem requirements'ı karşılıyor mu? |
| System Integration Testing | Sistem dış sistemlerle doğru haberleşiyor mu? |
| Acceptance Testing | Ürün kullanıcı ihtiyacını karşılıyor mu? |

Bu seviyeler her projede katı bir sırayla yürütülmek zorunda değildir. Modern development süreçlerinde farklı test levels paralel olarak uygulanabilir ve automation pipeline'larına dahil edilebilir.

---

## Test Case Design

Software Testing'in profesyonel olarak uygulanabilmesi için yalnızca neyi test edeceğimizi bilmek yeterli değildir.

Testin hangi koşullarda gerçekleştirileceğini, hangi input'ların kullanılacağını ve başarının nasıl değerlendirileceğini de tanımlamamız gerekir.

Bu amaçla **Test Case** oluşturulur.

Bir test case, belirli bir davranışı değerlendirmek için gerekli preconditions, inputs, test steps ve expected results gibi bilgileri içerir.

### Industrial Example: Sensor Connection Failure

Önceki requirement'ımıza geri dönelim.

> **REQ-COM-001:** Sensör bağlantısı kesildikten sonra sistem en geç 100 ms içerisinde `ERROR_STATE` durumuna geçmeli ve yeni motor hareket komutları üretmeyi durdurmalıdır.

Bu requirement için örnek bir test case hazırlayalım.

| Field | Description |
|---|---|
| Test Case ID | TC-COM-001 |
| Requirement ID | REQ-COM-001 |
| Test Objective | Sensor disconnect durumunda sistemin davranışını doğrulamak |
| Preconditions | Uygulama çalışıyor, sensör bağlantısı aktif ve sistem normal çalışma durumunda |
| Test Action | Sensör bağlantısını kontrollü olarak kes |
| Expected Result | En geç 100 ms içerisinde `ERROR_STATE` ve yeni hareket komutlarının engellenmesi |
| Actual Result | Test sırasında kaydedilecek |
| Test Status | NOT RUN |

Test sırasında sistemin state değişimi ve komut üretiminin durduğu an ölçülür. Her iki koşulun da sağlanıp sağlanmadığı değerlendirilir.

Bu örnekte özellikle dikkat edilmesi gereken şey, 100 ms'nin yalnızca uygulamanın kendi logları üzerinden tahmin edilmemesidir. Test ortamının gereksinime uygun zaman ölçümü yapabildiği de doğrulanmalıdır.

Ayrıca `ERROR_STATE` durumuna geçilmesi ve yeni komutların engellenmesi, motorun fiziksel olarak güvenli şekilde durduğunun tek başına kanıtı değildir. Böyle bir safety requirement varsa ayrıca test edilmelidir.

### Positive and Negative Testing

İyi bir test yaklaşımı yalnızca sistemin normal koşullarda doğru davranıp davranmadığına bakmaz.

**Positive Testing**, geçerli input'lar ve beklenen koşullar altında davranışı değerlendirir.

**Negative Testing** ise geçersiz input'lar, hatalı durumlar ve beklenmeyen koşullar altında sistemin davranışını inceler.

Örneğin sensor communication modülünde:

| Scenario | Expected Behavior |
|---|---|
| Valid sensor message | Measurement kabul edilir |
| Invalid message format | Mesaj reddedilir |
| Corrupted data | Veri kullanılmaz, uygun hata yönetimi uygulanır |
| Connection timeout | Error handling başlatılır |
| Sensor reconnect | Tanımlanmış recovery procedure uygulanır |

Burada bütün expected behaviors'ın önceden requirements veya interface specifications ile tanımlanması önemlidir.

### Boundary Value Analysis

Birçok software defect, normal çalışma aralığının merkezinde değil, sınırlarında ortaya çıkar.

Bu nedenle **Boundary Value Analysis**, giriş değerlerinin belirlenen sınırlarını ve bu sınırların hemen çevresini test etmeye odaklanır.

Örneğin bir motor komutunun geçerli aralığı 0–100 RPM olsun.

Yalnızca 50 RPM test etmek yeterli değildir.

Şu değerleri de değerlendirmek anlamlıdır:

| Input | Neden test edilir? |
|---|---|
| -1 | Alt sınırın dışı |
| 0 | Alt sınır |
| 1 | Alt sınırın hemen içi |
| 99 | Üst sınırın hemen içi |
| 100 | Üst sınır |
| 101 | Üst sınırın dışı |

Böylece özellikle `>` ile `>=` gibi koşulların yanlış uygulanması sonucu oluşabilecek defect'ler tespit edilebilir.

### Test Case ile Test Execution Farkı

Test case hazırlanması, testin gerçekleştirildiği anlamına gelmez.

Önce beklenen davranış ve prosedür tanımlanır. Ardından test çalıştırılır ve actual result kaydedilir.

Örneğin:

Test Case: `TC-COM-001`

Expected Result: En fazla 100 ms

Actual Result: 145 ms

Result: FAIL

Bu durumda test execution başarısızdır ve gözlemlenen failure kayıt altına alınmalıdır.

Bu kayıt, sonraki aşamada debugging ve Root Cause Analysis sürecinin başlangıç noktası olabilir.

### Engineering Perspective

Test case hazırlamanın temel amacı yalnızca bir kontrol listesi oluşturmak değildir.

Asıl amaç, yazılımın davranışını **ölçülebilir, tekrarlanabilir ve gerektiğinde başka bir mühendis tarafından da doğrulanabilir** hale getirmektir.

Bu sayede test süreci kişisel yorumlardan uzaklaşır ve daha sistematik bir mühendislik faaliyetine dönüşür.

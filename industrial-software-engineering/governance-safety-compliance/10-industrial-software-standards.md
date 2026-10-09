# Industrial Software Standards

## Contents

1. [Industrial Software Standards Nedir?](#industrial-software-standards-nedir)
2. [Temel Terminoloji](#temel-terminoloji)
3. [Gerçek Teknik Örnek - STM32 Safety Manual](#gerçek-teknik-örnek---stm32-safety-manual)
4. [Endüstride Software Standards - STMicroelectronics](#endüstride-software-standards---stmicroelectronics)
5. [Öğrendiklerim ve Kaynaklar](#öğrendiklerim-ve-kaynaklar)

---

## Industrial Software Standards Nedir?

Endüstriyel yazılım geliştirme sürecinde bir programın doğru çalışması tek başına yeterli olmayabilir. Özellikle robotik, otomotiv ve industrial automation gibi alanlarda yazılımın belirli kalite, güvenlik ve geliştirme gereksinimlerini karşılaması beklenir.

**Industrial Software Standards**, ürünlerin geliştirilmesi, test edilmesi, güvenliğinin değerlendirilmesi ve dokümante edilmesi için kullanılan standartları kapsar.

Burada önemli olan nokta, standartların yalnızca kodun nasıl yazılacağını belirlememesi. Development process, risk analysis, testing, verification ve documentation gibi faaliyetler de standartların kapsamına girebilir.

Araştırdığım temel standartlar:

| Standard | Kullanım Alanı | Temel Amaç |
|---|---|---|
| ISO 9001 | Farklı sektörlerdeki kuruluşlar | Quality Management System ve süreçlerin iyileştirilmesi |
| IEC 61508 | Industrial safety-related systems | Functional Safety |
| ISO 26262 | Automotive | Road vehicles için Functional Safety |
| IEC 62443 | Industrial automation & control systems | Cybersecurity |

Bu standartların hepsi aynı amaca hizmet etmiyor.

Örneğin **ISO 9001**, şirketin kalite yönetim süreçlerine odaklanırken **IEC 61508**, safety-related sistemlerin tehlikeli arızalara karşı nasıl geliştirileceğiyle ilgilenir.

**ISO 26262**, functional safety yaklaşımını automotive sektörüne uygular.

**IEC 62443** ise industrial systems içerisinde cybersecurity gereksinimlerine odaklanır.

ISO 9001'in güncel sürümü Eylül 2026'da yayımlanan ISO 9001:2026'dır.

### Functional Safety vs Cybersecurity

Bu iki kavramın arasındaki farkı bilmek önemli.

**Functional Safety**, sistemdeki arızaların tehlikeli sonuçlara yol açma riskinin kontrol edilmesiyle ilgilidir.

Örneğin motor kontrol sisteminde bir hardware failure tespit edildiğinde sistemin tanımlanmış safe state'e geçmesi gerekebilir.

**Cybersecurity** ise sistemlerin yetkisiz erişimlere, kötü niyetli müdahalelere ve dijital güvenlik tehditlerine karşı korunmasıyla ilgilidir.

Örneğin bir industrial controller'a network üzerinden yetkisiz komut gönderilmesinin önlenmesi cybersecurity kapsamında değerlendirilebilir.

Gerçek projelerde bu iki alan birbirini etkileyebilir. Ancak aynı kavramlar değildir.

---

## Temel Terminoloji

| Terim | Açıklama |
|---|---|
| Standard | Belirli bir alan için tanımlanmış ortak gereksinimler ve kurallar |
| Compliance | İlgili gereksinimlere uygunluk |
| Certification | Yetkili bir kuruluş tarafından belirli bir kapsamdaki uygunluğun değerlendirilip belgelendirilmesi |
| Functional Safety | Arızaların tehlikeli sonuçlara yol açma riskinin kontrol edilmesi |
| Safety Requirement | Sistemin safety açısından karşılaması gereken gereksinim |
| Hazard | Zarar oluşturabilecek potansiyel tehlike kaynağı |
| Risk Assessment | Tehlikelerin ve bunlarla ilişkili risklerin değerlendirilmesi |
| SIL | IEC 61508 kapsamında kullanılan Safety Integrity Level |
| ASIL | ISO 26262 kapsamında kullanılan Automotive Safety Integrity Level |
| Traceability | Requirements, design ve tests arasındaki ilişkilerin takip edilebilmesi |
| Verification & Validation | Gereksinimlere ve kullanım amacına uygunluğun değerlendirilmesi |
| Safety Manual | Bir bileşenin safety-related sistemde kullanılması için gerekli koşulları açıklayan teknik doküman |

### SIL ve ASIL

IEC 61508 içerisinde **SIL 1–4** seviyeleri bulunur. SIL 4 en yüksek safety integrity seviyesidir.

ISO 26262 içerisinde ise **ASIL A–D** sınıflandırması kullanılır. ASIL D en yüksek sınıftır.

Ancak SIL ile ASIL birbirinin doğrudan karşılığı değildir. Farklı standartlara ve risk değerlendirme yöntemlerine aittirler.

Önemli olan, bir projede hangi seviyenin gerekli olduğunun yalnızca developer tarafından seçilmemesi. Bu kararın sistemin tehlikeleri ve riskleri değerlendirilerek belirlenmesidir.

---

## Gerçek Teknik Örnek - STM32 Safety Manual

Standartların gerçek bir embedded system projesine nasıl yansıdığını anlamak için STMicroelectronics'in yayımladığı **STM32H5 Series Safety Manual (UM3150)** dokümanını inceledim.

Bu doküman, STM32H5 microcontroller'larının IEC 61508 kapsamında safety-related sistemlerde kullanılmasına ilişkin bilgileri içeriyor.

Dokümanda ilgimi çeken bölümlerden biri **Hardware and Software Diagnostics** oldu.

STMicroelectronics, belirli hardware failure durumlarını tespit etmek için uygulanabilecek safety mechanisms tanımlıyor.

Örneğin:

| Safety Mechanism | Görevi |
|---|---|
| CPU_SM_0 | CPU üzerinde periodic self-test gerçekleştirilmesi |
| RAM_SM_0 | SRAM üzerinde periodic memory test gerçekleştirilmesi |
| FLASH_SM_0 | Flash memory'nin periodic integrity test ile kontrol edilmesi |

Bu mekanizmalar ST'nin gerçek safety manual'ında tanımlanmış durumda.

### Bu Testler Neden Gerekiyor?

Normal bir embedded application düşünelim.

Microcontroller üzerinde çalışan yazılım motor kontrolü için hesaplamalar yapıyor olabilir.

Ancak CPU, RAM veya Flash memory içerisinde bir hardware fault meydana gelirse uygulamanın hesapladığı değerler veya programın execution behavior'ı etkilenebilir.

Bu nedenle safety-related sistemlerde belirli hardware failures'ın tespit edilmesi ve bunlara karşı uygun safety mechanisms'ın uygulanması gerekir.

Örneğin sistemde bir memory fault tespit edildiğini düşünelim.

Uygulamanın tasarımına göre motor komutlarının engellenmesi veya sistemin güvenli bir duruma geçirilmesi gerekebilir.

Bu mantığı basitleştirilmiş bir C örneğiyle gösterebiliriz:

```c id="6gphf4"
void SafetyMonitor(void)
{
    if (!CPU_Test_OK() ||
        !RAM_Test_OK() ||
        !FLASH_Test_OK())
    {
        DisableMotorOutput();
        system_state = FAULT_STATE;
    }
}
```

**Not:** Bu kod STMicroelectronics'in gerçek library source code'u değildir. Safety mechanism sonuçlarının application tarafından nasıl değerlendirilebileceğini anlatmak için hazırlanmış basitleştirilmiş bir örnektir.

Gerçek bir safety-related sistemde diagnostic functions, fault detection time, safe state ve motorun fiziksel olarak güvenli duruma geçirilmesi gibi ayrıntıların ayrıca tasarlanması ve doğrulanması gerekir.

### Safety Requirements ve Traceability

Burada standartların software development sürecine nasıl yansıdığını daha iyi anlamak mümkün.

Örneğin aşağıdaki gibi bir safety requirement tanımlandığını düşünelim:

**SR-001:** RAM diagnostic test başarısız olduğunda sistem yeni motor hareket komutlarının üretilmesini engellemelidir.

Bu requirement için bir test case hazırlanabilir:

| Requirement | Test Case | Expected Result |
|---|---|---|
| SR-001 | TC-001 | RAM diagnostic failure durumunda yeni hareket komutları engellenir |

Bu tablo eğitim amacıyla hazırlanmış örnek bir traceability ilişkisidir.

Böylece requirement'ın yalnızca yazılmış olmadığını, hangi test ile değerlendirildiğini de takip edebiliriz.

Bu yaklaşım, ilk konumuz olan **Software Testing** ile Industrial Software Standards arasındaki bağlantıyı gösteriyor.

**Bu örnekten çıkardığım sonuç:** Standards, yalnızca teorik kurallardan oluşmuyor. Gerçek bir projede diagnostic mechanisms, safety requirements, test cases ve documentation gibi somut mühendislik çalışmalarına dönüşüyor.

---

## Endüstride Software Standards - STMicroelectronics

Profesyonel bir şirketin safety requirements'a yönelik nasıl ürün geliştirdiğini anlamak için STMicroelectronics'in **X-CUBE-STL** ve **STSW-SILPLC** paketlerini inceledim.

### X-CUBE-STL

X-CUBE-STL, STM32 microcontroller'ları için geliştirilmiş bir **Functional Safety Package**.

IEC 61508 kapsamında safety functions geliştiren mühendislerin kullanabileceği diagnostic software ve teknik dokümantasyon sağlıyor.

Paket içerisinde:

- MCU Safety Manual
- FMEA / FMEDA bilgileri
- CPU, RAM ve Flash self-test library
- Kullanım koşulları ve teknik açıklamalar

bulunabiliyor.

**FMEA (Failure Mode and Effects Analysis)**, olası failure modes ve bunların etkilerini değerlendirmek için kullanılan bir analiz yöntemidir.

**FMEDA (Failure Modes, Effects and Diagnostic Analysis)** ise bu analizi diagnostic coverage ve failure rate gibi nicel bilgilerle destekler.

X-CUBE-STL'nin ilgili sürümleri TÜV Rheinland tarafından IEC 61508 gereksinimleri kapsamında değerlendirilmiş ve sertifikalandırılmıştır.

Burada önemli bir ayrıntı bulunuyor:

**Sertifikalı bir software library kullanmak, geliştirilen bütün ürünün otomatik olarak sertifikalı olduğu anlamına gelmez.**

Örneğin bir robot controller içerisinde X-CUBE-STL kullanılması, motorların, sensor interfaces'ın veya application software'in bütün safety requirements'ı karşıladığını tek başına göstermez.

Sistemin tamamı için gerekli safety analysis, integration ve verification faaliyetleri yine gerçekleştirilmelidir.

### STSW-SILPLC - Gerçek Uygulama

STMicroelectronics'in yayımladığı **STSW-SILPLC**, STEVAL-SILPLC01 platformu için hazırlanmış bir firmware paketidir.

Bu proje, industrial safety uygulamalarına yönelik daha somut bir örnek oluşturuyor.

Ürün dokümantasyonunda:

- Undervoltage / overvoltage detection
- Fault durumunda digital output disablement
- Safety function management
- Diagnostic mechanisms
- EtherCAT ve RS485 communication

gibi özellikler açıklanıyor.

Böylece daha önce ayrı konular olarak araştırdığımız **Communication, Memory Safety, Testing ve Real-Time Systems** kavramlarının tek bir industrial application içerisinde nasıl bir araya gelebildiğini görmek mümkün.

STMicroelectronics bu firmware paketinin tamamının TÜV değerlendirmesine dahil olmadığını, resmi sertifikasyona sahip firmware component'ın kullanılan X-CUBE-STL library olduğunu ayrıca belirtiyor.

Bu ayrım, endüstride certification kapsamının neden dikkatli incelenmesi gerektiğini gösteriyor.

### Şirketlerde Standards Süreci

Bu örnekler üzerinden standards odaklı development sürecini genel olarak şöyle özetleyebiliriz:

**Risk Assessment → Safety Requirements → Design & Development → Verification & Validation → Documentation → Assessment**

Burada documentation önemli bir yer tutuyor.

Çünkü mühendislerin yalnızca sistemin doğru çalıştığını söylemesi yeterli olmayabilir. Belirli requirements'ın karşılandığını test results, analysis reports ve diğer engineering evidence ile gösterebilmeleri gerekir.

Bu nedenle standards, software engineers'ın yalnızca kod yazma şeklini değil, gerçekleştirdikleri çalışmaları nasıl takip ettiklerini ve belgelediklerini de etkiler.

---

## Öğrendiklerim ve Kaynaklar

Industrial Software Standards araştırmasından çıkardığım temel sonuç, standartların yazılım geliştirme sürecini daha sistematik, izlenebilir ve değerlendirilebilir hale getirmek için kullanıldığıdır.

Özellikle dört nokta önemli:

1. **Her standardın amacı farklı:** ISO 9001 kalite yönetimine, IEC 61508 ve ISO 26262 functional safety'ye, IEC 62443 ise industrial cybersecurity'ye odaklanır.
2. **Standards yalnızca coding rules değildir:** Risk analysis, requirements, testing ve documentation da sürecin parçalarıdır.
3. **Traceability önemli:** Bir safety requirement'ın nasıl uygulandığı ve hangi testlerle doğrulandığı takip edilebilmelidir.
4. **Component certification, system certification anlamına gelmez:** X-CUBE-STL örneğinde görüldüğü gibi sertifikalı bir bileşen kullanılması, bütün sistemin otomatik olarak uygun olduğu anlamına gelmez.

STM32 Safety Manual üzerinden gerçek diagnostic mechanisms'ı, X-CUBE-STL ve STSW-SILPLC üzerinden ise bu mekanizmaların endüstriyel ürün geliştirme sürecinde nasıl kullanılabildiğini incelemek mümkün.

### Kaynaklar

**Industrial Software Standards**

- [ISO - ISO 9001 Quality Management](https://www.iso.org/9001-2026)
- [IEC - IEC 61508 Functional Safety](https://webstore.iec.ch/en/publication/5515)
- [ISO - ISO 26262 Road Vehicles Functional Safety](https://www.iso.org/publication/PUB200262.html)
- [IEC - IEC 62443-4-1 Secure Product Development](https://webstore.iec.ch/en/publication/33615)

**Gerçek Teknik Örnek - STM32**

- [STMicroelectronics - STM32H5 Series Safety Manual (UM3150)](https://www.st.com/resource/en/user_manual/um3150-stm32h5-series-safety-manual-stmicroelectronics.pdf)
- [STMicroelectronics - X-CUBE-STL Functional Safety Package](https://www.st.com/en/embedded-software/x-cube-stl.html)
- [STMicroelectronics - STSW-SILPLC Firmware Package](https://www.st.com/en/embedded-software/stsw-silplc.html)
- [STMicroelectronics - Functional Safety Engineering Blog](https://blog.st.com/x-cube-stl/)

---

[Back to Main Documentation](../../README.md)

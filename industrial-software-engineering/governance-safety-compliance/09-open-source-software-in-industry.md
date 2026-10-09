# Open Source Software in Industry

## Contents

1. [Open Source Software Nedir?](#open-source-software-nedir)
2. [Temel Terminoloji](#temel-terminoloji)
3. [Gerçek GitHub Örneği - Google Benchmark](#gerçek-github-örneği---google-benchmark)
4. [Endüstride Open Source - Microsoft VS Code](#endüstride-open-source---microsoft-vs-code)
5. [Öğrendiklerim ve Kaynaklar](#öğrendiklerim-ve-kaynaklar)

---

## Open Source Software Nedir?

Open Source Software (OSS), source code'u belirli lisans koşulları altında kullanılabilen, değiştirilebilen ve dağıtılabilen yazılımlardır.

Endüstriyel yazılım projelerinde her component'ın sıfırdan geliştirilmesi yerine mevcut open-source libraries ve frameworks kullanılabilir.

Örneğin:

- Robotik uygulamalarında ROS 2,
- Image processing uygulamalarında OpenCV,
- C++ testing için Catch2,
- Performance measurement için Google Benchmark,
- İşletim sistemi olarak Linux

kullanılabilir.

Bu yaklaşım development süresini azaltabilir ve önceden geliştirilmiş yazılım bileşenlerinden yararlanmayı sağlar.

Ancak araştırırken karşılaştığım en önemli ayrım şu oldu:

**Bir repository'nin public olması, kodun otomatik olarak open-source lisansına sahip olduğu anlamına gelmez.**

GitHub üzerinde bir repository public olarak paylaşılmış olabilir. Fakat açık bir kullanım izni veya software license bulunmuyorsa varsayılan copyright kuralları geçerlidir.

Dolayısıyla bir şirkette GitHub üzerinden alınan kodu kullanmadan önce lisansının incelenmesi gerekir.

### Open Source Licenses

Yaygın kullanılan bazı software licenses:

| License | Commercial Use | Temel Koşullar |
|---|---|---|
| MIT | İzin verilir | Copyright ve license notice korunmalıdır |
| Apache 2.0 | İzin verilir | License ve ilgili notices korunmalı, yapılan değişiklikler belirtilmelidir |
| GPL | İzin verilir | Dağıtım sırasında ilgili copyleft ve source code yükümlülüklerine uyulmalıdır |
| LGPL | İzin verilir | Library ve integration şekline bağlı ek koşullar bulunur |

**MIT ve Apache 2.0**, permissive licenses olarak adlandırılır. Bu lisanslar, koşulları karşılandığında kodun proprietary applications içerisinde kullanılmasına izin verebilir.

**GPL** ise copyleft license olarak değerlendirilir. GPL kapsamındaki kodun başka bir yazılımla nasıl birleştirildiği ve dağıtıldığı, ortaya çıkan yazılımın lisans yükümlülüklerini etkileyebilir.

Burada dikkatimi çeken nokta, GPL'nin ticari kullanımı yasaklamaması.

Ayrıca bir şirketin GPL lisanslı yazılımı yalnızca kendi bünyesinde kullanması, tek başına bütün şirket kodunu yayınlamasını gerektirmez.

Asıl önemli sorulardan biri yazılımın **yalnızca şirket içinde mi kullanıldığı, yoksa müşterilere dağıtılıp dağıtılmadığıdır.**

Bu nedenle lisans değerlendirmesi yapılırken yalnızca license name'e bakmak yeterli değildir. Kullanım ve dağıtım şekli de dikkate alınmalıdır.

---

## Temel Terminoloji

| Terim | Açıklama |
|---|---|
| Open Source | Kaynak kodun belirli özgürlükler ve lisans koşullarıyla sunulması |
| Proprietary Software | Kullanımı, değiştirilmesi veya dağıtılması hak sahibi tarafından sınırlandırılan yazılım |
| Software License | Kodun hangi koşullarda kullanılabileceğini belirleyen hukuki izin |
| Commercial Use | Yazılımın ticari amaçlarla kullanılması |
| Permissive License | Genellikle daha az dağıtım koşulu bulunan open-source lisans türü |
| Copyleft | Belirli türev çalışmaların dağıtımında lisans özgürlüklerinin korunmasını gerektiren yaklaşım |
| Dependency | Bir uygulamanın çalışmak veya geliştirilmek için ihtiyaç duyduğu başka yazılım |
| Third-Party Software | Başka kişi veya kuruluşlar tarafından geliştirilen yazılım bileşeni |
| License Compliance | Kullanılan yazılımların lisans koşullarına uygun hareket edilmesi |
| SBOM | Software Bill of Materials; üründe kullanılan yazılım bileşenlerinin envanteri |
| Security Vulnerability | Yazılımda güvenlik riski oluşturabilecek zayıflık |

Open-source kullanımı yalnızca licensing konusu değildir.

Bir library'nin güncel olup olmadığı, bilinen security vulnerabilities içerip içermediği ve projenin bakımının devam edip etmediği de değerlendirilmelidir.

Bu nedenle profesyonel projelerde third-party dependencies hem licensing hem de security açısından incelenebilir.

---

## Gerçek GitHub Örneği - Google Benchmark

Open-source lisanslarının gerçek bir repository üzerinde nasıl kullanıldığını anlamak için beşinci konumuzda araştırdığım **Google Benchmark** projesini tekrar inceledim.

Google Benchmark, C++ uygulamalarının performance measurement işlemlerinde kullanılan açık kaynak bir kütüphanedir.

Repository içerisinde `LICENSE` isimli bir dosya bulunuyor.

Bu dosyayı açtığımızda projenin **Apache License 2.0** kullandığını görüyoruz.

Lisans dosyasının başlangıcı:

```text
Apache License
Version 2.0, January 2004
```

Apache 2.0, commercial use'a izin veren bir open-source license.

Yani bir şirket, Google Benchmark kütüphanesini kendi ticari yazılım geliştirme süreçlerinde kullanabilir.

Ancak bu, hiçbir koşul bulunmadığı anlamına gelmez.

Örneğin lisanslı kodun dağıtılması durumunda:

- License metninin korunması,
- İlgili copyright ve attribution notices'ın korunması,
- Değiştirilen dosyalarda yapılan değişikliklerin belirtilmesi,
- Varsa ilgili `NOTICE` yükümlülüklerinin karşılanması

gerekebilir.

### Örnek Senaryo

Bir robotik yazılım şirketinde çalıştığımızı ve C++ ile geliştirilmiş motor control algorithm'ın performansını ölçmek istediğimizi düşünelim.

Google Benchmark kullanarak geliştirdiğimiz fonksiyonların execution time değerlerini ölçebiliriz.

Bu durumda önce ilgili kütüphanenin license dosyasını inceleriz.

Ardından:

1. Kullanacağımız library version'ını belirleriz.
2. License conditions'ı kontrol ederiz.
3. Kütüphanenin projede nasıl kullanılacağını ve dağıtılıp dağıtılmayacağını değerlendiririz.
4. Gerekli notices ve dependency kayıtlarını tutarız.
5. Library'nin security ve maintenance durumunu gözden geçiririz.

Burada Google Benchmark'ı yalnızca internal development ve testing için kullanmakla, kütüphaneyi müşteriye dağıtılan ürünün içine dahil etmek arasında fark bulunabilir.

**Bu örnekten çıkardığım sonuç:** Bir GitHub repository'sini kullanmadan önce yalnızca source code'u incelemek değil, `LICENSE` dosyasını ve kullanım koşullarını da değerlendirmek gerekir.

---

## Endüstride Open Source - Microsoft VS Code

Şirketlerin open-source yazılımları nasıl değerlendirdiğini anlamak için Microsoft'un **Visual Studio Code** GitHub repository'sini inceledim.

Bu repository özellikle uygun bir örnek çünkü hem Microsoft'un geliştirdiği source code'u hem de projede kullanılan third-party software bilgilerini açık şekilde görebiliyoruz.

### LICENSE.txt

Repository içerisinde `LICENSE.txt` isimli bir dosya bulunuyor.

Bu dosyada VS Code'un açık kaynak kod tabanının **MIT License** altında yayımlandığı belirtiliyor.

```text
MIT License
Copyright (c) 2015 - present Microsoft Corporation
```

MIT License, commercial use ve modification gibi faaliyetlere izin verir.

Ancak copyright ve license notice'ın korunması gerekir.

Burada dikkat edilmesi gereken bir başka ayrıntı var:

GitHub'daki `Code - OSS` source code'u ile Microsoft'un dağıttığı resmi Visual Studio Code ürünü aynı lisans kapsamında değildir.

Microsoft'un resmi VS Code dağıtımı, ek özelleştirmeler ve ürün bileşenleri nedeniyle farklı product license koşullarına sahiptir.

Bu durum, open-source source code ile o koddan oluşturulmuş ticari ürünün her zaman aynı lisans koşullarına sahip olmayabileceğini gösteriyor.

### ThirdPartyNotices.txt

Repository'de dikkatimi çeken ikinci dosya `ThirdPartyNotices.txt` oldu.

Bu dosyada Microsoft'un projede kullandığı çeşitli third-party software components ve bunların lisans bilgileri yer alıyor.

Örneğin listede MIT ve Apache 2.0 gibi farklı lisanslarla yayımlanmış bileşenler bulunuyor.

Dosyada yalnızca kütüphane adları değil, ilgili copyright ve license texts de yer alabiliyor.

Bu yaklaşım sayesinde projede kullanılan farklı yazılım bileşenlerinin hangi koşullara tabi olduğu takip edilebiliyor.

### Şirketlerde Open Source Kullanım Süreci

Bu örnekten hareketle bir şirketin third-party library kullanırken izleyebileceği genel süreci şu şekilde özetleyebiliriz:

**Library Selection → License Review → Security Check → Approval → Integration → Documentation & Maintenance**

Örneğin developer yeni bir library kullanmak istediğinde:

- Kütüphanenin lisansı incelenebilir.
- Mevcut project license ile uyumu değerlendirilebilir.
- Bilinen security vulnerabilities kontrol edilebilir.
- Şirketin engineering ve legal süreçlerine göre approval alınabilir.
- Kullanılan library ve version bilgileri kayıt altına alınabilir.

Bu, Microsoft'un iç onay sürecinin birebir açıklaması değildir; açık repository kayıtları ve yaygın software engineering uygulamaları üzerinden oluşturulmuş örnek bir workflow'dur.

Microsoft'un gerçek repository'sinden doğrulayabildiğimiz nokta ise **third-party license notices'ın açık şekilde belgelendiğidir**.

Ayrıca GitHub'ın **Dependabot** gibi araçları dependencies içerisinde bilinen security vulnerabilities tespit edildiğinde developer'ları uyarabilir.

Bu nedenle open-source library seçimi bir kere yapılıp unutulan bir işlem olmamalıdır. Yazılımın kullanıldığı süre boyunca maintenance ve security takibi de yapılmalıdır.

### Industrial Applications Açısından Değerlendirme

Bir industrial robot application içerisinde ROS 2, OpenCV veya farklı C/C++ libraries kullanıldığını düşünelim.

Bu libraries development açısından önemli avantajlar sağlayabilir.

Ancak ürün müşteriye teslim edilmeden önce kullanılan bütün third-party components'ın lisans koşulları ve security durumları değerlendirilmelidir.

Özellikle defense, automotive ve industrial automation gibi sektörlerde şirketin kendi intellectual property haklarını koruması da önemlidir.

Bu nedenle open-source kullanımı yalnızca developer'ın teknik kararı değil; engineering, security ve gerektiğinde legal ekiplerinin birlikte değerlendirebileceği bir konudur.

**Bu araştırmadan çıkardığım sonuç:** Open-source software kullanmak, internette bulduğumuz kodu doğrudan projeye kopyalamaktan daha kapsamlı bir süreçtir. License compliance, dependency management ve security birlikte düşünülmelidir.

---

## Öğrendiklerim ve Kaynaklar

Open Source Software in Industry araştırmasından çıkardığım temel sonuç, GitHub üzerinde bulunan yazılımların ticari projelerde kullanılabildiği ancak bunun belirli koşullara bağlı olduğudur.

Özellikle dört nokta önemli:

1. **Public repository, otomatik kullanım izni değildir:** License bulunmuyorsa copyright kuralları geçerlidir.
2. **Commercial use ile license compliance farklı konular:** MIT, Apache ve GPL gibi lisanslar ticari kullanıma izin verebilir ancak farklı yükümlülükler getirir.
3. **Kullanım ve dağıtım şekli önemli:** Bir library'nin internal development sırasında kullanılması ile müşteriye teslim edilen ürüne dahil edilmesi farklı koşullar oluşturabilir.
4. **Third-party software takip edilmeli:** Microsoft VS Code örneğinde olduğu gibi dependencies ve license notices'ın kayıt altında tutulması önemlidir.

Google Benchmark üzerinden gerçek bir Apache 2.0 license dosyasını, Microsoft VS Code üzerinden ise MIT License ve ThirdPartyNotices dosyalarının endüstriyel yazılım projelerinde nasıl kullanılabildiğini incelemek mümkün.

Bu araştırma genel bir teknik değerlendirmedir. Gerçek bir ticari ürün için lisans uygunluğu, kullanılan sürüm ve dağıtım modeli üzerinden ayrıca kontrol edilmelidir.

### Kaynaklar

**Open Source Licensing**

- [GitHub Docs - Licensing a Repository](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/licensing-a-repository)
- [Choose a License - License Comparison](https://choosealicense.com/licenses/)
- [GitHub Open Source Guides - The Legal Side of Open Source](https://opensource.guide/legal/)
- [GNU - Frequently Asked Questions About GPL](https://www.gnu.org/licenses/gpl-faq.html)

**Gerçek GitHub Örneği - Google Benchmark**

- [Google Benchmark - GitHub Repository](https://github.com/google/benchmark)
- [Google Benchmark - LICENSE](https://github.com/google/benchmark/blob/main/LICENSE)

**Industrial Open Source - Microsoft**

- [Microsoft VS Code - GitHub Repository](https://github.com/microsoft/vscode)
- [Microsoft VS Code - LICENSE.txt](https://github.com/microsoft/vscode/blob/main/LICENSE.txt)
- [Microsoft VS Code - ThirdPartyNotices.txt](https://github.com/microsoft/vscode/blob/main/ThirdPartyNotices.txt)
- [Visual Studio Code - Official FAQ](https://code.visualstudio.com/docs/supporting/faq)
- [GitHub Docs - Dependabot Alerts](https://docs.github.com/en/code-security/concepts/supply-chain-security/dependabot-alerts)

---

[Back to Main Documentation](../../README.md)

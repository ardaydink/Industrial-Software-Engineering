# Hardware & Software Communication

## Contents

1. [Hardware & Software Communication Nedir?](#hardware--software-communication-nedir)
2. [Temel Terminoloji](#temel-terminoloji)
3. [Gerçek Kod Örneği - pySerial](#gerçek-kod-örneği---pyserial)
4. [Endüstride Haberleşme - Universal Robots](#endüstride-haberleşme---universal-robots)
5. [Öğrendiklerim ve Kaynaklar](#öğrendiklerim-ve-kaynaklar)

---

## Hardware & Software Communication Nedir?

Endüstriyel yazılım sistemlerinde farklı cihazların ve yazılım bileşenlerinin birbirleriyle veri alışverişi yapması gerekir.

Örneğin bir industrial robot application içerisinde:

- Sensor, ölçüm verilerini controller'a gönderir.
- Controller, motor driver'a komut iletir.
- Industrial PC, sistemin durumunu kullanıcı arayüzüne aktarır.

Bu işlemler farklı communication interfaces ve protocols üzerinden gerçekleştirilebilir.

Haberleşmeyi anlamak için iki farklı durumu değerlendirdim.

**Hardware Communication:** Yazılımın fiziksel bir sensor, microcontroller veya controller gibi cihazlarla haberleşmesi.

**Software-to-Software Communication:** İki yazılım bileşeninin birbirine veri göndermesi. Örneğin aynı bilgisayardaki iki process veya ROS 2 node'unun haberleşmesi.

Bu ayrımın kesin iki protokol sınıfı olmadığını bilmek önemli. Yazılım bileşenleri farklı bilgisayarlarda çalışıyorsa aralarında yine fiziksel bir network bağlantısı bulunabilir.

Ayrıca geliştirilen yazılımın gerçek hardware olmadan test edilmesi için **Simulation**, **Mock** veya **Loopback** gibi yöntemler kullanılabilir.

Örneğin gerçek bir sensör olmadan önceden kaydedilmiş sensor measurements kullanılarak image processing algorithm test edilebilir.

Bu sayede hardware henüz hazır olmasa bile software development sürecine devam edilebilir.

---

## Temel Terminoloji

| Terim | Açıklama |
|---|---|
| UART | İki cihaz arasında asynchronous serial communication sağlayan arayüz |
| SPI | Clock kullanılarak gerçekleştirilen synchronous serial communication |
| I²C | Birden fazla cihazın ortak bus üzerinden haberleşmesini sağlayan serial protocol |
| CAN | Özellikle automotive ve industrial systems içerisinde kullanılan, mesaj tabanlı haberleşme |
| Ethernet | Yerel ağlarda veri iletimi için kullanılan bağlantı teknolojisi |
| TCP / UDP | IP networks üzerinde kullanılan transport protocols |
| Socket | Uygulamaların network üzerinden veri alışverişi için kullandığı software interface |
| Protocol | Verilerin formatını ve nasıl iletileceğini belirleyen kurallar |
| SDK | Bir cihaz veya yazılım sistemiyle çalışmak için sağlanan development tools ve libraries |
| Timeout | Belirlenen sürede beklenen cevap alınamadığında işlemin sonlandırılması veya hata olarak değerlendirilmesi |
| Loopback | Gönderilen verinin tekrar aynı sisteme döndürülerek test edilmesi |
| Mock / Simulator | Gerçek bir bileşenin davranışının yazılım ile temsil edilmesi |

Burada özellikle **Ethernet ile TCP/IP arasındaki farkı** anlamak önemli.

Ethernet, network bağlantısının daha alt katmanlarıyla ilgiliyken TCP ve UDP, network üzerinden uygulamalar arasında veri taşımak için kullanılır.

Dolayısıyla bir Ethernet kablosunun bağlı olması, uygulamanın kullandığı communication protocol'ün doğru çalıştığı anlamına gelmez.

Benzer şekilde ROS 2 içerisinde kullanılan Publisher/Subscriber yapısı, yazılım bileşenleri arasındaki iletişimi düzenler. Bu iletişim, kullanılan middleware ve sistem mimarisine göre aynı bilgisayarda veya farklı bilgisayarlar arasında gerçekleşebilir.

---

## Gerçek Kod Örneği - pySerial

Fiziksel cihazlarla haberleşme ve hardware olmadan communication testing arasındaki farkı anlamak için GitHub üzerinde bulunan **pySerial** projesinin resmi örneklerini inceledim.

pySerial, Python üzerinden serial ports kullanarak veri gönderme ve alma işlemlerini gerçekleştirmek için kullanılan bir kütüphanedir.

### Fiziksel Cihaz ile Haberleşme

pySerial dokümantasyonundaki port açma ve veri okuma örneklerinden uyarlanmış basit bir kullanım:

```python
import serial

with serial.Serial('/dev/ttyUSB0', 115200, timeout=1) as ser:
    ser.write(b'HELLO\n')
    response = ser.readline()
    print(response)
```

Burada:

- `/dev/ttyUSB0`, Linux üzerinde kullanılan örnek serial port adresidir.
- `115200`, communication baud rate değeridir.
- `ser.write()`, karşı tarafa veri gönderir.
- `ser.readline()`, newline ile sonlandırılmış cevabı okumaya çalışır.
- `timeout=1`, okuma işleminin sınırsız süre beklemesini engeller.

Bu kodun gerçekten veri alabilmesi için uygun serial device bağlantısı ve karşı tarafın mesaj formatına cevap veren bir firmware'e sahip olması gerekir.

`HELLO` mesajı burada haberleşme mantığını göstermek için kullanılmıştır; belirli bir endüstriyel cihazın gerçek komutu değildir.

### Hardware Olmadan Haberleşme Testi

pySerial'ın resmi dokümantasyonunda `loop://` isimli bir virtual serial interface de bulunuyor.

Bu özellik sayesinde fiziksel bir cihaz bağlamadan gönderilen verinin tekrar okunması sağlanabiliyor.

```python
import serial

with serial.serial_for_url('loop://', timeout=1) as ser:
    ser.write(b'HELLO\n')
    response = ser.readline()
    print(response)
```

Bu örnekte beklenen output:

```text
b'HELLO\n'
```

İlk örnekte veri fiziksel bir serial interface üzerinden gönderilirken, ikinci örnekte aynı veri software loopback mekanizmasıyla geri dönüyor.

Bu sayede uygulamanın veri gönderme ve okuma işlemleri basit şekilde kontrol edilebiliyor.

Ancak burada önemli bir sınır bulunuyor:

**Loopback, gerçek bir sensor simulator değildir.**

Yalnızca gönderilen veriyi geri döndürür. Sensörün gerçek ölçüm üretmesini, fiziksel bağlantı hatalarını veya cihazın kendi protocol behavior'ını taklit etmez.

Daha gerçekçi bir test için cihazın communication protocol'ünü uygulayan bir simulator veya mock component geliştirilmesi gerekir.

Bu örnekten çıkardığım sonuç, haberleşme kodunun bir kısmının fiziksel hardware bulunmadan da test edilebileceği. Ancak gerçek cihazla integration testing yine gerekli olabilir.

---

## Endüstride Haberleşme - Universal Robots

Hardware ve software communication'ın gerçek bir robotik sistemde nasıl kullanıldığını anlamak için **Universal Robots** tarafından geliştirilen RTDE interface'ini ve açık kaynak Python Client Library'yi inceledim.

**RTDE (Real-Time Data Exchange)**, robot controller ile external applications arasında veri alışverişi yapılmasını sağlayan bir communication interface'tir.

Bağlantı standart **TCP/IP** üzerinden gerçekleştiriliyor.

Örneğin bir Industrial PC üzerinde çalışan Python application, robot controller'dan aşağıdaki verileri alabilir:

- Joint positions
- Joint velocities
- Tool position
- Robot status
- Digital I/O states

Robot controller ise RTDE üzerinden desteklenen input fields aracılığıyla external application'dan belirli verileri kabul edebilir.

Bu iletişimde uygulamanın hangi verileri göndermek veya almak istediği, **Recipe** olarak adlandırılan field listeleriyle tanımlanıyor.

### Gerçek GitHub Örneği

Universal Robots'un GitHub repository'sinde `record.py` isimli bir örnek uygulama bulunuyor.

Bu uygulama robot controller'dan RTDE verilerini alarak CSV dosyasına kaydedebiliyor.

Repository'nin README dosyasındaki örnek komut:

```bash
python record.py --host 192.168.0.1 --frequency 10
```

Bu komutta:

- `--host`, bağlanılacak robot controller'ın IP adresini belirtir.
- `--frequency 10`, verilerin 10 Hz örnekleme frekansıyla istenmesini sağlar.
- `record.py`, alınan verileri CSV dosyasına kaydeder.

Buradaki IP adresi dokümantasyonda verilen örnektir. Gerçek kullanımda controller'ın IP adresi ve RTDE configuration bilgileri doğru belirlenmelidir.

RTDE interface'i için varsayılan TCP portu **30004**'tür.

### Aynı Uygulamanın Simulator ile Kullanılması

Bu repository'nin dikkat çekici tarafı, yalnızca gerçek robotlarla değil, Universal Robots'un **URSim** simulator'ıyla çalışmaya yönelik açıklamalar da içermesi.

URSim, robot controller ortamını fiziksel robot olmadan çalıştırmaya yardımcı olur.

Repository'de Docker üzerinde simulator başlatmak ve RTDE portunu dışarıya açmak için örnekler bulunuyor.

Böylece external application, gerçek robot controller yerine simulator'a bağlanarak communication işlemlerinin bir bölümünü test edebilir.

İki farklı çalışma ortamını şöyle karşılaştırabiliriz:

| Gerçek Robot | URSim Simulator |
|---|---|
| Fiziksel robot controller ile TCP/IP bağlantısı | Simulated controller ile TCP/IP bağlantısı |
| Gerçek robot status ve measurements | Simulated controller verileri |
| Hardware ve network davranışları değerlendirilebilir | Fiziksel robot olmadan software integration testleri yapılabilir |
| Fiziksel hareket ve güvenlik koşulları dikkate alınmalıdır | Gerçek robotun bütün fiziksel davranışlarını doğrulamaz |

Bu örnek, endüstride aynı communication interface'in hem gerçek sistemde hem de development/testing ortamında kullanılabilmesinin avantajını gösteriyor.

Özellikle hardware erişiminin sınırlı olduğu projelerde simulator kullanmak, developer'ların uygulama geliştirmeye devam etmesine yardımcı olabilir.

### Industrial Applications Açısından Değerlendirme

Benzer bir yaklaşım Ethernet üzerinden measurement gönderen laser sensors için de düşünülebilir.

Örneğin sensörden gelen verileri işleyen uygulama üç temel parçaya ayrılabilir:

**Sensor Communication → Data Processing → User Interface**

Gerçek sensor henüz bağlı değilken Data Processing bölümüne önceden kaydedilmiş measurement verileri gönderilebilir.

Bu şekilde yazılımın processing ve visualization bölümleri geliştirilmeye devam edebilir.

Gerçek sensor bağlandığında ise network connection, communication protocol, data format ve timeout behavior gibi özellikler ayrıca değerlendirilmelidir.

**Bu araştırmadan çıkardığım sonuç:** Endüstride hardware communication yalnızca bir cihazdan veri okumak anlamına gelmiyor. Communication interface'in doğru tasarlanması, error handling ve hardware olmadan testing yapılabilmesi de geliştirme sürecinin önemli parçaları.

---

## Öğrendiklerim ve Kaynaklar

Hardware & Software Communication araştırmasından çıkardığım temel sonuç, fiziksel bağlantı ile uygulamalar arasındaki veri alışverişinin farklı seviyelerde değerlendirilmesi gerektiğidir.

Özellikle dört nokta önemli:

1. **Interface ve Protocol farklı kavramlar:** Ethernet bağlantısının kurulması, application protocol'ünün doğru çalıştığını garanti etmez.
2. **Software-to-Software Communication da önemli:** Farklı processes ve ROS 2 nodes arasında veri aktarımı, fiziksel sensör olmadan gerçekleştirilebilir.
3. **Hardware olmadan testing mümkün:** Loopback, mock ve simulation yöntemleri development sürecini hızlandırabilir.
4. **Gerçek hardware testing gereklidir:** Simulator kullanmak fiziksel bağlantı, gerçek sensör davranışı ve hardware kaynaklı problemlerin tamamını doğrulamaz.

pySerial üzerinden gerçek ve virtual serial communication arasındaki farkı, Universal Robots'un RTDE örneği üzerinden ise aynı communication interface'in fiziksel robot ve simulator ortamlarında nasıl kullanılabildiğini incelemek mümkün.

### Kaynaklar

**Hardware & Software Communication**

- [pySerial - GitHub Repository](https://github.com/pyserial/pyserial)
- [pySerial - Short Introduction](https://pyserial.readthedocs.io/en/latest/shortintro.html)
- [pySerial - URL Handlers and Loopback](https://pyserial.readthedocs.io/en/stable/url_handlers.html)
- [ROS 2 - Publisher and Subscriber Tutorial](https://docs.ros.org/en/rolling/Tutorials/Beginner-Client-Libraries/Writing-A-Simple-Py-Publisher-And-Subscriber.html)

**Industrial Communication - Universal Robots**

- [Universal Robots - RTDE Guide](https://docs.universal-robots.com/tutorials/communication-protocol-tutorials/rtde-guide.html)
- [Universal Robots - RTDE Python Client Library](https://github.com/UniversalRobots/RTDE_Python_Client_Library)
- [Universal Robots - RTDE Client Python Module](https://docs.universal-robots.com/tutorials/communication-protocol-tutorials/rtde-python-client-guide.html)

---

[Back to Main Documentation](../../README.md)

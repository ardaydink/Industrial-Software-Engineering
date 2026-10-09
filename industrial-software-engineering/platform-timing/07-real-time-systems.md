# Real-Time Systems

## Contents

1. [Real-Time Systems Nedir?](#real-time-systems-nedir)
2. [Temel Terminoloji](#temel-terminoloji)
3. [Gerçek Kod Örneği - FreeRTOS](#gerçek-kod-örneği---freertos)
4. [Endüstride Real-Time Systems - Universal Robots](#endüstride-real-time-systems---universal-robots)
5. [Öğrendiklerim ve Kaynaklar](#öğrendiklerim-ve-kaynaklar)

---

## Real-Time Systems Nedir?

Real-Time Systems, bir işlemin yalnızca doğru sonuç üretmesinin değil, bu sonucu belirli bir zaman sınırı içerisinde üretmesinin de önemli olduğu sistemlerdir.

Örneğin bir industrial robot düşünelim. Robotun motor kontrol algoritması doğru hesaplamalar yapıyor olabilir. Ancak motorlara gönderilmesi gereken komutlar zamanında üretilmezse robotun hareketinde problemler oluşabilir.

Bu nedenle Real-Time Systems konusunda en önemli nokta şudur:

**Real-time, bir sistemin mutlaka çok hızlı olması değil, belirlenmiş timing requirements'ı karşılayabilmesidir.**

Bir işlemin tamamlanması için izin verilen zaman sınırına **Deadline** denir.

Örneğin motor controller'ın her 10 ms içerisinde yeni bir kontrol hesabı tamamlaması gerekiyorsa bu sistemde yalnızca hesaplamanın doğruluğu değil, süresi de önemlidir.

Real-Time Systems genellikle iki ana grupta değerlendirilir:

- **Hard Real-Time:** Deadline'ın kaçırılması kabul edilemez sonuçlar oluşturabilir. Örneğin safety-critical control functions.
- **Soft Real-Time:** Deadline'ın zaman zaman kaçırılması sistem kalitesini düşürür ancak her gecikme sistemin tamamen başarısız olduğu anlamına gelmez. Örneğin video streaming.

Real-time davranış elde etmek için birçok embedded system içerisinde **Real-Time Operating System (RTOS)** kullanılır.

FreeRTOS, Zephyr ve VxWorks bu alanda kullanılan işletim sistemlerine örnektir.

RTOS, task scheduling ve synchronization gibi mekanizmalar sağlayarak zaman açısından kritik uygulamaların geliştirilmesini kolaylaştırır.

Ancak bir sistemde RTOS kullanılması, bütün deadline'ların otomatik olarak karşılanacağını garanti etmez. Yazılımın execution time, scheduling ve resource usage davranışları da değerlendirilmelidir.

---

## Temel Terminoloji

| Terim | Açıklama |
|---|---|
| Deadline | Bir işlemin tamamlanması gereken zaman sınırı |
| Latency | Bir olay ile ona verilen cevap arasındaki gecikme |
| Jitter | Tekrarlanan işlemlerin zamanlamasındaki değişkenlik |
| Task | İşletim sistemi tarafından çalıştırılan iş birimi |
| Scheduler | Hangi task'ın ne zaman çalışacağını belirleyen mekanizma |
| Task Priority | Task'ların çalışma önceliği |
| Preemption | Çalışan bir task'ın daha yüksek öncelikli task tarafından kesilebilmesi |
| Periodic Task | Belirli aralıklarla tekrar çalışması planlanan task |
| WCET | Worst-Case Execution Time, bir işlemin en kötü koşullardaki execution time sınırı |
| Deadline Miss | Bir işlemin belirlenen deadline içerisinde tamamlanamaması |

Özellikle **Latency** ve **Jitter** arasındaki fark önemli.

Örneğin bir sensor task'ın her 10 ms'de bir çalışmasını istediğimizi düşünelim.

Gerçekte task bazen 10 ms, bazen 11 ms, bazen de 9 ms aralıklarla çalışıyorsa zamanlama değişkenliği oluşur. Buna Jitter denir.

Real-Time Systems'da yalnızca average execution time değerine bakmak yeterli değildir.

Bir task çoğu zaman 2 ms içerisinde tamamlanıyor olabilir. Ancak nadiren 15 ms sürdüğünde 10 ms'lik deadline'ı kaçırıyorsa sistemin timing requirements'ı karşılanmıyor demektir.

---

## Gerçek Kod Örneği - FreeRTOS

Real-time task scheduling mantığını anlamak için FreeRTOS'un resmi GitHub dokümantasyonundaki **`xTaskDelayUntil()`** örneğini inceledim.

FreeRTOS, özellikle microcontroller tabanlı embedded systems içerisinde kullanılan bir RTOS'tur.

İncelediğim örnekte bir task'ın belirli aralıklarla çalıştırılması amaçlanıyor.

Resmi örneğin mantığını koruyan, 10 ms periyot için uyarlanmış kod:

```c
void vPeriodicTask(void *pvParameters)
{
    TickType_t lastWakeTime = xTaskGetTickCount();
    const TickType_t period = pdMS_TO_TICKS(10);

    for (;;)
    {
        xTaskDelayUntil(&lastWakeTime, period);

        // Periodic task operations
    }
}
```

Bu kodda:

- `xTaskGetTickCount()` mevcut RTOS tick değerini alıyor.
- `pdMS_TO_TICKS(10)` 10 ms değerini RTOS tick birimine dönüştürüyor.
- `xTaskDelayUntil()` task'ın planlanan bir sonraki periyotta tekrar çalışmaya hazır hale gelmesini sağlıyor.
- `for (;;)` task'ın sürekli çalışmasını sağlıyor.

Burada dikkatimi çeken asıl nokta `vTaskDelay()` ile `xTaskDelayUntil()` arasındaki fark.

### vTaskDelay vs xTaskDelayUntil

**vTaskDelay()**, çağrıldığı andan itibaren belirli bir süre bekler.

Örneğin bir task'ın işlemi 3 ms sürüyor ve ardından 10 ms bekliyorsa toplam çalışma aralığı yaklaşık 13 ms olabilir.

**xTaskDelayUntil()** ise bekleme süresini önceki planlanan uyanma zamanına göre hesaplar.

Böylece task için daha düzenli bir execution period oluşturulabilir.

Ancak bu fonksiyonun kullanılması task'ın tam olarak o anda CPU üzerinde çalışmaya başlayacağını veya işlemini deadline içerisinde tamamlayacağını garanti etmez.

Başka yüksek öncelikli task'lar, interrupts ve işlem süreleri scheduling davranışını etkileyebilir.

Ayrıca zamanlama hassasiyeti RTOS tick configuration'a bağlıdır.

### Deadline Miss Nasıl Anlaşılır?

FreeRTOS'un `xTaskDelayUntil()` fonksiyonu bir return value üretir.

Bir sonraki planlanan uyanma zamanı geçmişse fonksiyon beklemeden dönebilir ve `pdFALSE` sonucu verebilir.

Bu bilgi timing problemlerini araştırmak için kullanılabilir. Ancak işlemin gerçek completion deadline'ını doğrulamak için ayrıca execution time ölçümü ve uygun deadline kontrolü gerekir.

**Bu örnekten çıkardığım sonuç:** Real-Time Systems geliştirilirken task'ların yalnızca çalışması değil, ne zaman çalıştığının da kontrol edilmesi gerekiyor.

---

## Endüstride Real-Time Systems - Universal Robots

Real-Time Systems kavramının gerçek robotlarda nasıl kullanıldığını anlamak için Universal Robots'un yayımladığı **Real-Time Data Exchange (RTDE)** dokümantasyonunu inceledim.

Universal Robots, endüstriyel robot kolları ve collaborative robots (cobots) geliştiren bir şirkettir.

Robot controller, hareketleri gerçekleştirebilmek için joint position, velocity ve diğer kontrol bilgilerini düzenli olarak işler.

Bu tür sistemlerde control loop'un zamanlaması önemlidir.

### Universal Robots - RTDE

Universal Robots'un RTDE interface'i, robot controller ile external applications arasında veri alışverişi yapılmasını sağlar.

Örneğin Ubuntu üzerinde çalışan bir uygulama, robot controller'dan şu bilgileri alabilir:

- Joint positions
- Joint velocities
- Tool position
- Robot status
- Digital I/O states

Universal Robots'un resmi RTDE dokümantasyonuna göre e-Series ve ur-Series robotlarda real-time control loop frekansı **500 Hz** seviyesindedir.

Bu, bir kontrol döngüsü periyodunun:

**1 / 500 Hz = 2 ms**

olduğu anlamına gelir.

RTDE interface'i de uygun koşullarda bu frekansta output messages üretebilir.

Ancak burada önemli bir ayrıntı bulunuyor.

Universal Robots'un dokümantasyonunda, controller yeterli computational resources'a sahip değilse bazı RTDE output packages'ın gönderilmeyebileceği belirtiliyor.

Bu durumda eski paketleri sonradan göndermek yerine en güncel veri gönderiliyor.

Bunun nedeni robotun kendi real-time control işlemlerinin, dışarıya veri aktarımından daha yüksek önceliğe sahip olmasıdır.

### Bu Yaklaşım Neden Önemli?

Robotun hareket kontrolüyle ilgili kritik işlemler zamanında gerçekleşmelidir.

Ancak external monitoring application'a gönderilen her veri paketinin aynı derecede kritik olması gerekmeyebilir.

Bu nedenle farklı işlemler arasında priority ve timing requirements açısından ayrım yapılır.

Bu olayda RTDE üzerinden veri aktarılması, external PC'nin de 2 ms deadline garantisiyle çalıştığı anlamına gelmez.

Network latency, CPU load ve application scheduling gibi faktörler external application'ın davranışını etkileyebilir.

### Gerçek GitHub Örneği

Universal Robots, RTDE için hazırladığı Python Client Library'yi GitHub üzerinde açık şekilde yayımlıyor.

Repository içerisinde:

- `record.py`: Robot verilerinin kaydedilmesi.
- `example_control_loop.py`: Örnek robot control communication.
- `example_plotting.py`: Kaydedilen verilerin görselleştirilmesi.

gibi örnekler bulunuyor.

Bu örnekler, gerçek robotlarla yazılım uygulamaları arasında iletişimin nasıl kurulabildiğini incelemek için kullanılabilir.

**Bu araştırmadan çıkardığım sonuç:** Endüstriyel robotlarda bütün işlemlerin aynı öncelikte olması gerekmiyor. Control loop gibi timing açısından kritik işlemler ile monitoring ve data logging gibi işlemler farklı gereksinimlere sahip olabiliyor.

Universal Robots örneği, real-time scheduling ve task priorities kavramlarının gerçek bir ürünün tasarımında nasıl önem kazandığını gösteriyor.

---

## Öğrendiklerim ve Kaynaklar

Real-Time Systems araştırmasından çıkardığım temel sonuç, real-time kavramının yüksek işlemci hızıyla değil, işlemlerin belirlenen zaman sınırlarında gerçekleştirilmesiyle ilişkili olduğudur.

Özellikle dört nokta önemli:

1. **Deadline önemli:** Bir işlemin doğru sonuç üretmesi kadar, sonucu ne zaman ürettiği de önemli olabilir.
2. **Jitter kontrol edilmeli:** Periodic task'ların çalışma zamanlarındaki değişkenlik sistem davranışını etkileyebilir.
3. **RTOS kullanmak tek başına yeterli değil:** Scheduling, task priorities ve execution time birlikte değerlendirilmelidir.
4. **Her işlem aynı önceliğe sahip değil:** Universal Robots örneğinde control loop ile external monitoring işlemlerinin farklı timing gereksinimleri olduğu görülüyor.

FreeRTOS örneği üzerinden periodic task scheduling mantığını, Universal Robots dokümantasyonu üzerinden ise real-time gereksinimlerinin endüstriyel robotların çalışma yapısını nasıl etkilediğini incelemek mümkün.

### Kaynaklar

**Real-Time Systems ve FreeRTOS**

- [FreeRTOS - Official Website](https://www.freertos.org/)
- [FreeRTOS - xTaskDelayUntil Documentation](https://freertos.org/Documentation/02-Kernel/04-API-references/02-Task-control/03-xTaskDelayUntil)
- [FreeRTOS - GitHub Source Example](https://github.com/FreeRTOS/FreeRTOS-Website-Content/blob/main/content/en-us/Documentation/02-Kernel/04-API-references/02-Task-control/03-xTaskDelayUntil.md)
- [FreeRTOS Kernel Book - Task Management](https://github.com/FreeRTOS/FreeRTOS-Kernel-Book/blob/main/ch04.md)

**Industrial Real-Time Systems**

- [Universal Robots - RTDE Guide](https://docs.universal-robots.com/tutorials/communication-protocol-tutorials/rtde-guide.html)
- [Universal Robots - RTDE Python Client GitHub Repository](https://github.com/UniversalRobots/RTDE_Python_Client_Library)

---

[Back to Main Documentation](../../README.md)

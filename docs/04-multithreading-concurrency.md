# Multithreading & Concurrency

## Contents

1. [Multithreading ve Concurrency Nedir?](#multithreading-ve-concurrency-nedir)
2. [Temel Terminoloji](#temel-terminoloji)
3. [Gerçek Kod Örneği - ThreadSanitizer](#gerçek-kod-örneği---threadsanitizer)
4. [Endüstride Concurrency - Mars Pathfinder](#endüstride-concurrency---mars-pathfinder)
5. [Öğrendiklerim ve Kaynaklar](#öğrendiklerim-ve-kaynaklar)

---

## Multithreading ve Concurrency Nedir?

Bir yazılımın aynı anda birden fazla işle ilgilenmesi gerektiğinde **Concurrency** kavramı karşımıza çıkar.

Örneğin bir industrial robot application düşünelim. Sistem bir taraftan sensor measurements alırken diğer taraftan motor control algorithm'ı çalıştırıyor ve aynı zamanda kullanıcı arayüzünü güncelliyor olabilir.

Bu işlemlerin hepsini tek bir execution flow içerisinde sırasıyla gerçekleştirmek her zaman verimli olmayabilir.

**Multithreading**, bir process içerisinde birden fazla thread oluşturarak farklı işlerin yürütülmesini sağlayan yaklaşımlardan biridir.

Bir process içerisindeki thread'ler aynı address space'i paylaşabilir ancak kendi execution state'lerine ve stack'lerine sahiptir.

Burada iki kavramı ayırmak önemli:

- **Concurrency:** Birden fazla işin zaman bakımından iç içe ilerlemesi. İşlemlerin fiziksel olarak aynı anda çalışması gerekmez.
- **Parallelism:** Birden fazla işlemin gerçekten aynı anda, örneğin farklı CPU core'larında çalışması.

Multithreading ile concurrency sağlanabilir; uygun hardware ve scheduling koşullarında parallel execution da mümkün olabilir.

Ancak thread kullanmak her zaman performansı artırmaz.

Thread'ler aynı memory alanını paylaştıkları için aralarında **Synchronization** sağlanması gerekebilir. Aksi halde beklenmeyen sonuçlar, veri kayıpları veya programın kilitlenmesi gibi problemler oluşabilir.

---

## Temel Terminoloji

| Terim | Açıklama |
|---|---|
| Process | Çalışan bir programın işletim sistemi tarafından yönetilen instance'ı |
| Thread | Bir process içerisindeki execution unit |
| Shared Resource | Birden fazla thread'in erişebildiği ortak kaynak |
| Race Condition | Sonucun işlemlerin gerçekleşme sırasına veya zamanlamasına bağlı olması |
| Data Race | Aynı memory alanına synchronization olmadan yapılan çakışan erişimler |
| Mutex | Shared resource'a erişimi kontrol etmek için kullanılan synchronization mekanizması |
| Deadlock | Thread'lerin birbirlerinin tuttuğu kaynakları bekleyerek ilerleyememesi |
| Atomic Operation | Diğer thread'lere bölünmez bir işlem olarak görünen operasyon |
| Priority Inversion | Yüksek öncelikli bir task'ın düşük öncelikli bir task yüzünden beklemek zorunda kalması |

Özellikle **Race Condition** ile **Data Race** arasındaki farkı bilmek önemli.

Race Condition daha geniş bir kavramdır. Programın doğru çalışması, işlemlerin hangi sırada gerçekleştiğine bağlı hale gelebilir.

Data Race ise belirli bir memory access problemidir. İki thread aynı memory location'a synchronization olmadan erişiyorsa ve en az birisi yazma işlemi yapıyorsa data race oluşabilir.

C++ içerisinde data race, **undefined behavior** oluşturur.

Bir diğer önemli kavram **Deadlock**.

Örneğin:

- Thread A, Mutex 1'i tutuyor ve Mutex 2'yi bekliyor.
- Thread B, Mutex 2'yi tutuyor ve Mutex 1'i bekliyor.

Bu durumda iki thread de birbirini beklediği için ilerleyemeyebilir.

---

## Gerçek Kod Örneği - ThreadSanitizer

Multithreading kaynaklı hataların nasıl oluştuğunu anlamak için Google'ın GitHub üzerinde yayımladığı **ThreadSanitizer (TSan)** dokümantasyonundaki C++ örneğini inceledim.

ThreadSanitizer, C/C++ uygulamalarında data race problemlerini tespit etmek için kullanılan bir araçtır.

İncelediğim `simple_race.cc` örneği:

```cpp
#include <pthread.h>
#include <stdio.h>

int Global;

void *Thread1(void *x) {
    Global++;
    return NULL;
}

void *Thread2(void *x) {
    Global--;
    return NULL;
}

int main() {
    pthread_t t[2];
    pthread_create(&t[0], NULL, Thread1, NULL);
    pthread_create(&t[1], NULL, Thread2, NULL);
    pthread_join(t[0], NULL);
    pthread_join(t[1], NULL);
}
```

Bu programda iki farklı thread oluşturuluyor:

- **Thread1:** `Global` değişkenini bir artırıyor.
- **Thread2:** `Global` değişkenini bir azaltıyor.

İlk bakışta `Global` değişkeninin başlangıçtaki `0` değerine geri dönmesini bekleyebiliriz.

Ancak iki thread de aynı değişkene herhangi bir synchronization mekanizması olmadan erişiyor.

`Global++` ve `Global--` işlemleri normal bir `int` üzerinde thread-safe değildir.

Bu yüzden programda **Data Race** oluşuyor. C++ açısından sonuç undefined behavior olduğu için her çalışmada doğru değeri elde edeceğimizi garanti edemeyiz.

### ThreadSanitizer ile Problemin Tespiti

Google'ın dokümantasyonunda örnek şu şekilde compile ediliyor:

```bash
clang++ simple_race.cc -fsanitize=thread -fPIE -pie -g
```

Buradaki `-fsanitize=thread` parametresi ThreadSanitizer'ı etkinleştiriyor.

Resmi dokümantasyonda program çalıştırıldığında şu uyarı gösteriliyor:

```text
WARNING: ThreadSanitizer: data race
```

Rapor içerisinde iki thread'in aynı memory alanına yaptığı çakışan erişimler ve ilgili source code satırları görülebiliyor.

Böylece programın çıktısı normal görünse bile synchronization problemi tespit edilebiliyor.

### Problem Nasıl Çözülür?

Bu örnekte `Global` değişkeni için **Atomic Operation** kullanılabilir.

C++ içerisinde:

```cpp
#include <atomic>

std::atomic<int> Global{0};
```

şeklinde tanımlanmış bir değişken üzerinde `Global++` ve `Global--` işlemleri atomik olarak gerçekleştirilir.

Bu, örnekteki shared integer üzerindeki data race problemini giderebilir.

Daha karmaşık durumlarda birden fazla işlemin veya shared resource'un birlikte korunması gerekebilir. Bu durumda `std::mutex` gibi synchronization mekanizmaları kullanılabilir.

Ancak mutex kullanılması tek başına bütün concurrency problemlerini çözmez. Yanlış lock ordering gibi durumlar deadlock oluşturabilir.

**Bu örnekten çıkardığım sonuç:** Multithreading kullanılan bir programın beklenen sonucu üretmesi, thread'ler arasında doğru synchronization yapıldığı anlamına gelmez. Bu tür hataları tespit etmek için ThreadSanitizer gibi özel analysis tools kullanılabilir.

---

## Endüstride Concurrency - Mars Pathfinder

Concurrency problemlerinin gerçek sistemlerde nasıl sonuçlar oluşturabileceğini anlamak için NASA'nın **Mars Pathfinder** görevinde yaşanan bir yazılım problemini araştırdım.

1997 yılında Mars'a gönderilen Pathfinder uzay aracının onboard computer'ı, **VxWorks Real-Time Operating System** kullanıyordu.

Sistem üzerinde farklı önceliklere sahip birden fazla task çalışıyordu.

Görev sırasında uzay aracının bilgisayarı belirli koşullarda beklenmedik şekilde reset atmaya başladı.

NASA/JPL yazılım ekibinin yayımladığı teknik açıklamaya göre problemin nedeni **Priority Inversion** idi.

### Problem Nasıl Oluştu?

Sistemde farklı önceliklere sahip task'lar ortak kaynaklara erişiyordu.

Basitleştirilmiş şekilde problem şu üç task ile açıklanabilir:

| Task | Priority | Görevi |
|---|---|---|
| High Priority | High | Kritik bus communication işlemleri |
| Medium Priority | Medium | Diğer sistem işlemleri |
| Low Priority | Low | Meteorological data işlemleri |

Düşük öncelikli task, ortak bir kaynağı koruyan mutex semaphore'u kullanırken daha yüksek öncelikli işlemler tarafından kesintiye uğrayabiliyordu.

Bu sırada yüksek öncelikli task aynı kaynağa erişmek istediğinde beklemek zorunda kalıyordu.

Orta öncelikli task'ların çalışması ise düşük öncelikli task'ın kaynağı serbest bırakmasını geciktiriyordu.

Sonuç olarak yüksek öncelikli task, kendisinden daha düşük öncelikli işlemler nedeniyle zamanında tamamlanamıyordu.

Sistem kritik bir deadline'ın kaçırıldığını tespit ettiğinde bilgisayarı resetliyordu.

Bu problem **Priority Inversion** olarak adlandırılıyor. Deadlock'tan farklı olarak task'lar kalıcı biçimde birbirini beklemek zorunda değil; fakat yüksek öncelikli task kabul edilemeyecek kadar gecikebiliyor.

### Engineering Team Problemi Nasıl Çözdü?

Mars Pathfinder yazılım ekibinin lideri Glenn E. Reeves tarafından yayımlanan açıklamada debugging süreci ayrıntılı olarak anlatılmış.

Ekip, problemin nedenini anlamak için uzay aracının laboratuvardaki benzer sisteminde aynı çalışma koşullarını yeniden oluşturmaya çalışmış.

Sistemde önceden geliştirilmiş **Tracing ve Logging** mekanizmaları bulunduğu için task execution ve synchronization olayları incelenebilmiş.

Problem yeniden oluşturulduktan sonra root cause'un Priority Inversion olduğu belirlenmiş.

Çözüm olarak ilgili semaphore için **Priority Inheritance** özelliği etkinleştirilmiş.

Priority Inheritance, yüksek öncelikli task düşük öncelikli bir task'ın tuttuğu resource'u beklerken, resource'u tutan task'ın önceliğinin geçici olarak yükseltilmesini sağlayan bir mekanizmadır.

Böylece düşük öncelikli task, araya giren orta öncelikli task'lar tarafından gereksiz yere geciktirilmeden işini tamamlayıp resource'u serbest bırakabilir.

Ekip değişikliğin diğer sistem bileşenlerine etkisini de test ettikten sonra spacecraft software üzerinde uzaktan güncelleme gerçekleştirmiş.

### Industrial Applications Açısından Değerlendirme

Bu olay özellikle Embedded Systems ve Robotics alanları için önemli bir örnek.

Bir industrial robot içerisinde de motor control, sensor acquisition ve logging gibi farklı önceliklere sahip task'lar çalışabilir.

Bu task'ların ortak kaynakları yanlış şekilde kullanması yalnızca performance düşüşüne değil, kritik timing requirements'ın karşılanamamasına da neden olabilir.

Mars Pathfinder örneğinde dikkatimi çeken nokta, problemin tek başına yanlış hesaplama yapan bir fonksiyondan kaynaklanmaması.

Sorun, farklı task'ların birlikte çalışırken ortaya çıkardığı zamanlama ve synchronization davranışından kaynaklanmış.

Bu nedenle **multithreaded systems test edilirken yalnızca fonksiyonların doğruluğunun değil, task interactions ve timing behavior'ın da değerlendirilmesi gerekiyor.**

---

## Öğrendiklerim ve Kaynaklar

Multithreading & Concurrency araştırmasından çıkardığım temel sonuç, birden fazla thread kullanmanın yalnızca işleri hızlandırmak anlamına gelmediğidir.

Özellikle dört nokta önemli:

1. **Concurrency ve Parallelism aynı şey değil:** Birden fazla işin birlikte ilerlemesi ile fiziksel olarak aynı anda çalışması farklı kavramlar.
2. **Shared resources dikkatli yönetilmeli:** Synchronization eksikliği Data Race ve Race Condition gibi problemler oluşturabilir.
3. **Testing tek başına yeterli olmayabilir:** ThreadSanitizer gibi araçlar, normal testlerde fark edilmeyen concurrency hatalarını ortaya çıkarabilir.
4. **Task priorities ve synchronization birlikte düşünülmeli:** Mars Pathfinder örneğinde görüldüğü gibi yanlış resource management kritik timing problemlerine neden olabilir.

ThreadSanitizer örneği üzerinden iki thread'in aynı değişkene erişirken nasıl data race oluşturabileceğini, Mars Pathfinder olayı üzerinden ise Priority Inversion probleminin gerçek bir embedded system üzerindeki etkilerini incelemek mümkün.

### Kaynaklar

**Multithreading ve Concurrency**

- [Google Sanitizers - ThreadSanitizer C++ Manual](https://github.com/google/sanitizers/wiki/ThreadSanitizerCppManual)
- [Google Sanitizers - Popular Data Races](https://github.com/google/sanitizers/wiki/ThreadSanitizerPopularDataRaces)
- [LLVM/Clang - ThreadSanitizer Documentation](https://clang.llvm.org/docs/ThreadSanitizer.html)

**Industrial Concurrency - Mars Pathfinder**

- [Glenn E. Reeves (NASA/JPL) - What Really Happened on Mars?](https://www.cs.cornell.edu/courses/cs614/1999sp/papers/pathfinder2.html)
- [Mike Jones - Mars Pathfinder Priority Inversion](https://space-robotics.org/snapshots/jones1997what.html)
- [Rapita Systems - Mars Pathfinder Software Analysis](https://www.rapitasystems.com/blog/what-really-happened-software-mars-pathfinder-spacecraft)

---

[Back to Main Documentation](../README.md)

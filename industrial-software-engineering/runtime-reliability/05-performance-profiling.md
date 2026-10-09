# Performance & Profiling

## Contents

1. [Performance ve Profiling Nedir?](#performance-ve-profiling-nedir)
2. [Temel Terminoloji](#temel-terminoloji)
3. [Gerçek Kod Örneği - Google Benchmark](#gerçek-kod-örneği---google-benchmark)
4. [Endüstride Performance & Profiling](#endüstride-performance--profiling)
5. [Öğrendiklerim ve Kaynaklar](#öğrendiklerim-ve-kaynaklar)

---

## Performance ve Profiling Nedir?

Bir yazılımın doğru sonuç üretmesi, her zaman yeterli performansa sahip olduğu anlamına gelmez.

Örneğin bir industrial robot application düşünelim. Sistem sensor measurements değerlerini doğru hesaplıyor olabilir. Ancak bu hesaplama beklenenden uzun sürüyorsa motor control işlemleri gecikebilir.

**Software Performance**, bir yazılımın işlemlerini ne kadar hızlı ve ne kadar sistem kaynağı kullanarak gerçekleştirdiğiyle ilgilidir.

**Profiling** ise program çalışırken CPU, memory ve diğer kaynakların hangi fonksiyonlar veya işlemler tarafından tüketildiğini analiz etme sürecidir.

Burada iki kavramı ayırmak önemli:

- **Benchmarking:** Bir işlemin performansını belirli koşullarda ölçmek ve karşılaştırmak.
- **Profiling:** Performans maliyetinin programın hangi bölümlerinden kaynaklandığını araştırmak.

Örneğin bir image processing algorithm'ın bir frame'i 50 ms içerisinde işlediğini ölçmek benchmarking kapsamında değerlendirilebilir.

Bu sürenin 40 ms'lik kısmının belirli bir image filtering fonksiyonundan kaynaklandığını tespit etmek ise profiling ile ilişkilidir.

Bu nedenle performance optimization sürecinde önce ölçüm yapmak, daha sonra bottleneck olarak belirlenen bölümü iyileştirmek önemlidir.

Genel yaklaşım:

**Measure → Profile → Identify Bottleneck → Optimize → Measure Again**

---

## Temel Terminoloji

| Terim | Açıklama |
|---|---|
| Latency | Bir işlemin veya isteğin tamamlanması için geçen süre |
| Throughput | Belirli bir zaman aralığında tamamlanan işlem miktarı |
| CPU Utilization | İşlemcinin kullanım oranı |
| Bottleneck | Sistemin performansını sınırlandıran bölüm |
| Profiling | Programın kaynak kullanımının analiz edilmesi |
| Benchmark | Performansı ölçmek veya karşılaştırmak amacıyla yapılan test |
| Hotspot | Profiling sonuçlarında yüksek kaynak tüketimiyle öne çıkan kod bölümü |
| Flame Graph | Fonksiyon çağrılarının kaynak tüketimini görselleştiren grafik |
| Performance Regression | Kod değişikliği sonrasında performansın kötüleşmesi |

Bu kavramlar özellikle Linux üzerinde çalışan endüstriyel uygulamalarda önemlidir.

Örneğin bir sistemin CPU Utilization değeri yüksek olabilir. Ancak bu bilgi tek başına hangi fonksiyonun performans problemi oluşturduğunu göstermez.

Benzer şekilde bir uygulamanın düşük CPU kullanması, mutlaka hızlı çalıştığı anlamına gelmez. Program I/O işlemleri veya synchronization nedeniyle bekliyor olabilir.

Bu nedenle performance analysis sırasında yalnızca tek bir metriğe bakmak yeterli olmayabilir.

---

## Gerçek Kod Örneği - Google Benchmark

Performance ölçümünün nasıl yapıldığını anlamak için Google'ın GitHub üzerinde geliştirdiği açık kaynak **Google Benchmark** kütüphanesini inceledim.

Bu kütüphane, C++ fonksiyonlarının çalışma sürelerini ölçmek ve farklı implementation'ların performansını karşılaştırmak için kullanılabiliyor.

Repository'nin README dosyasında bulunan örneklerden biri:

```cpp
#include <benchmark/benchmark.h>

static void BM_StringCopy(benchmark::State& state) {
    std::string x = "hello";

    for (auto _ : state)
        std::string copy(x);
}

BENCHMARK(BM_StringCopy);
BENCHMARK_MAIN();
```

Bu örnekte `std::string` kopyalama işleminin çalışma süresi ölçülmek isteniyor.

Kodun mantığı:

- `BM_StringCopy` ölçülecek benchmark fonksiyonunu tanımlıyor.
- `benchmark::State`, testin nasıl çalıştırılacağını ve ölçüleceğini yönetiyor.
- `for (auto _ : state)` içerisindeki işlem benchmark kapsamında tekrarlanıyor.
- `BENCHMARK()` ilgili fonksiyonu kaydediyor.
- `BENCHMARK_MAIN()` benchmark uygulamasının giriş noktasını oluşturuyor.

Benchmark sonuçlarında genellikle işlem başına geçen süre ve tekrar sayısı gibi bilgiler bulunuyor.

Burada dikkat edilmesi gereken nokta, compiler optimization işlemlerinin ölçüm sonuçlarını etkileyebilmesi. Özellikle sonucu kullanılmayan basit işlemler compiler tarafından kaldırılabilir.

Bu nedenle gerçek bir benchmark tasarlanırken `benchmark::DoNotOptimize()` gibi mekanizmalardan faydalanılabilir ve test ortamı mümkün olduğunca tutarlı tutulmalıdır.

### Linux perf ile Profiling

Benchmarking işlemin ne kadar sürdüğünü gösterirken, **Linux perf** gibi profiling tools bu sürenin hangi fonksiyonlarda harcandığını analiz etmeye yardımcı olur.

Linux kernel repository'sinde `perf` aracının kullanımına ilişkin gerçek örnekler bulunuyor.

Temel komutlar:

```bash
perf stat ./program
perf record -g ./program
perf report
```

- **perf stat:** Çalıştırılan programla ilgili performance counters ve istatistikleri gösterir.
- **perf record:** Program çalışırken profiling samples toplar.
- **perf report:** Toplanan verilerin fonksiyonlar ve call stacks üzerinden incelenmesini sağlar.

Özellikle `perf report` çıktısında hangi fonksiyonların daha fazla CPU samples içerdiğini görmek mümkün.

Ancak yüksek CPU sampling oranı, bir fonksiyonun otomatik olarak hatalı olduğu anlamına gelmez. Yalnızca optimization için incelenebilecek önemli bir aday olduğunu gösterir.

Bu örnekler sayesinde performance analysis'in rastgele kod değiştirmekten ziyade ölçümlere dayanması gerektiğini anlamak mümkün.

---

## Endüstride Performance & Profiling

Performance optimization çalışmalarının profesyonel yazılım şirketlerinde nasıl yapıldığını anlamak için Meta'nın yayımladığı **Strobelight: A Profiling Service Built on Open Source Technology** başlıklı mühendislik yazısını inceledim.

Meta, çok sayıda production server üzerinde çalışan uygulamalarının performansını analiz etmek için Strobelight isimli bir profiling altyapısı kullanıyor.

Strobelight farklı profiling tools'ları bir araya getirerek CPU usage, memory allocations ve function call stacks gibi verilerin incelenmesini sağlıyor.

Böylece mühendisler hangi kod bölümlerinin fazla sistem kaynağı tükettiğini belirleyebiliyor.

### Meta - The Biggest Ampersand

Meta'nın paylaştığı en dikkat çekici örneklerden biri, C++ içerisinde yapılan gereksiz bir `std::vector` kopyalama işlemi.

Bir performance engineer, profiling sonuçlarını incelerken şirketin büyük advertising services'lerinden birinde yüksek maliyet oluşturan bir vector copy işlemini fark etmiş.

Problemin nedeni, C++ içerisinde `auto` kullanımı nedeniyle büyük bir vector'ün gereksiz şekilde kopyalanmasıymış.

Bu farkı basitleştirilmiş bir örnekle gösterebiliriz:

**Gereksiz kopyalama:**

```cpp
auto values = existingVector;
```

**Reference kullanımı:**

```cpp
auto& values = existingVector;
```

İlk satırda mevcut vector'den yeni bir kopya oluşturuluyor.

İkinci satırda ise mevcut vector'e reference oluşturuluyor ve gereksiz kopyalama engelleniyor.

Bu iki satır Meta'nın yayımlanmamış proje kodundan alınmadı; mühendislik yazısında açıklanan değişikliğin mantığını göstermek için hazırlanmış basitleştirilmiş örneklerdir.

Gerçek olayda mühendis, `auto` ifadesine yalnızca bir `&` karakteri ekleyerek problemi düzeltmiş.

Meta'nın yayımladığı rapora göre değişiklik production ortamına aktarıldıktan sonra **yılda yaklaşık 15.000 sunuculuk kapasite tasarrufuna karşılık gelen** bir iyileştirme elde edilmiş.

Bu sayı doğrudan 15.000 fiziksel sunucunun kapatıldığı anlamına gelmiyor; şirketin raporladığı tahmini kapasite kazanımını ifade ediyor.

### Bu Örnek Neden Önemli?

Bu olayda fonksiyonun yanlış sonuç üretmesi gibi bir problem bulunmuyor.

Kod doğru çalışıyor ancak gereksiz memory copying nedeniyle fazla CPU ve memory resources tüketiyor.

Üstelik problem küçük bir kod değişikliğiyle çözülebiliyor.

Asıl önemli nokta, mühendislerin problemi tahmin ederek değil, **profiling verilerini inceleyerek tespit etmeleri**.

Meta'nın yaklaşımında profiling yalnızca uygulama yavaşladıktan sonra yapılmıyor. Production sistemlerinden düzenli veri toplanarak performance regressions'ın erken tespit edilmesi de hedefleniyor.

Benzer bir durum industrial image processing uygulamalarında da oluşabilir.

Örneğin her sensor frame'inin gereksiz yere kopyalanması CPU ve memory kullanımını artırabilir. Özellikle yüksek frekansta çalışan sistemlerde bu maliyetler birikerek latency problemlerine yol açabilir.

Ancak böyle bir optimization'ın gerçekten fayda sağlayıp sağlamadığı, değişiklik öncesi ve sonrasında yapılan ölçümlerle doğrulanmalıdır.

---

## Öğrendiklerim ve Kaynaklar

Performance & Profiling araştırmasından çıkardığım temel sonuç, yazılım optimizasyonunun yalnızca daha hızlı kod yazmaya çalışmak olmadığıdır.

Özellikle dört nokta önemli:

1. **Önce ölçüm yapılmalı:** Bir uygulamanın yavaş olduğunu bilmek, hangi fonksiyonun problemi oluşturduğunu bilmek anlamına gelmez.
2. **Benchmarking ve Profiling farklı amaçlara sahip:** Benchmark çalışma süresini ölçerken profiling performans maliyetinin nerede oluştuğunu araştırır.
3. **Küçük kod değişiklikleri büyük fark oluşturabilir:** Meta'nın C++ vector copy örneği, gereksiz işlemlerin büyük ölçekli sistemlerde önemli maliyetler oluşturabileceğini gösteriyor.
4. **Optimization sonuçları tekrar ölçülmeli:** Yapılan değişikliğin gerçekten fayda sağladığı ve başka bir performance regression oluşturmadığı kontrol edilmelidir.

Google Benchmark örneği üzerinden fonksiyonların performansının nasıl ölçülebildiğini, Linux perf üzerinden CPU profiling mantığını ve Meta'nın Strobelight çalışması üzerinden gerçek şirketlerde performance analysis'in nasıl kullanıldığını incelemek mümkün.

### Kaynaklar

**Benchmarking ve Profiling Tools**

- [Google Benchmark - GitHub Repository](https://github.com/google/benchmark)
- [Google Benchmark - User Guide](https://github.com/google/benchmark/blob/main/docs/user_guide.md)
- [Linux perf - Official GitHub Examples](https://github.com/torvalds/linux/blob/master/tools/perf/Documentation/examples.txt)
- [Linux perf - Profiling Tutorial](https://perfwiki.github.io/main/tutorial/)

**Industrial Performance & Profiling**

- [Meta Engineering - Strobelight: A Profiling Service Built on Open Source Technology](https://engineering.fb.com/2025/01/21/production-engineering/strobelight-a-profiling-service-built-on-open-source-technology/)
- [Meta Engineering - Accelerate Large-Scale Applications with BOLT](https://engineering.fb.com/2018/06/19/data-infrastructure/accelerate-large-scale-applications-with-bolt/)

---

[Back to Main Documentation](../../README.md)

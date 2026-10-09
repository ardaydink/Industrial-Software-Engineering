# Memory Management & Memory Safety

## Contents

1. [Memory Management Nedir?](#memory-management-nedir)
2. [Temel Terminoloji](#temel-terminoloji)
3. [Gerçek Kod Örneği - AddressSanitizer](#gerçek-kod-örneği---addresssanitizer)
4. [Endüstride Memory Management](#endüstride-memory-management)
5. [Öğrendiklerim ve Kaynaklar](#öğrendiklerim-ve-kaynaklar)

---

## Memory Management Nedir?

Bir yazılım çalışırken değişkenlerini, oluşturduğu object'leri ve işlediği verileri memory üzerinde tutar. **Memory Management**, bu kaynakların nasıl ayrıldığını, kullanıldığını ve artık ihtiyaç duyulmadığında nasıl serbest bırakıldığını yönetme sürecidir.

Özellikle C/C++ gibi dillerde developer'ın memory üzerinde daha fazla kontrol sahibi olması performans ve kaynak yönetimi açısından avantaj sağlarken, yanlış kullanım ciddi problemlere neden olabilir.

Örneğin uzun süre çalışan bir industrial application düşünelim. Program her yeni sensor measurement için memory ayırıyor fakat işi biten alanları serbest bırakmıyorsa zaman içerisinde memory tüketimi artabilir. Sonunda sistem yavaşlayabilir veya out-of-memory durumuyla karşılaşabilir.

Memory Management konusunda iki önemli kavram bulunur:

- **Stack:** Genellikle fonksiyon çağrıları ve local variables için kullanılan, yaşam süresi otomatik yönetilen memory alanıdır.
- **Heap:** Programın çalışma sırasında dinamik olarak memory ayırabildiği alandır.

Özellikle heap üzerinde oluşturulan object'lerin yaşam sürelerinin doğru yönetilmesi önemlidir.

Memory Safety ise programın yalnızca geçerli memory alanlarına, bu alanların yaşam süresi boyunca ve belirlenmiş sınırlar içerisinde erişebilmesiyle ilgilidir.

---

## Temel Terminoloji

| Terim | Açıklama |
|---|---|
| Memory Allocation | Programın kullanmak üzere memory ayırması |
| Memory Deallocation | Ayrılmış memory'nin serbest bırakılması |
| Memory Leak | Artık ihtiyaç duyulmayan memory'nin serbest bırakılmaması |
| Dangling Pointer | Artık geçerli olmayan bir object veya memory alanını gösteren pointer |
| Use-after-free | Serbest bırakılmış memory alanına tekrar erişilmesi |
| Buffer Overflow | Ayrılmış buffer sınırlarının dışına veri yazılması |
| Double Free | Aynı memory alanının birden fazla kez serbest bırakılması |
| Memory Corruption | Memory içerisindeki verilerin hatalı erişim veya yazma sonucunda bozulması |
| Out-of-Memory (OOM) | Sistemin ihtiyaç duyduğu memory allocation'ı karşılayamaması |

Bu hataların hepsi aynı sonucu oluşturmaz.

Örneğin **Memory Leak** genellikle zamanla artan resource consumption ile ilişkilidir. **Use-after-free** veya **Buffer Overflow** ise memory corruption, crash ve security vulnerability gibi sonuçlara neden olabilir.

Özellikle dikkat edilmesi gereken nokta, memory kaynaklı hataların her zaman ortaya çıktıkları anda görünür hale gelmemesidir.

Bir memory error programın bir bölümünde oluşup çok daha sonra başka bir fonksiyon çalışırken failure'a neden olabilir. Bu durum debugging sürecini zorlaştırır.

---

## Gerçek Kod Örneği - AddressSanitizer

Memory hatalarının nasıl tespit edildiğini anlamak için LLVM/Clang'ın resmi dokümantasyonunda yer alan **AddressSanitizer (ASan)** örneğini inceledim.

AddressSanitizer, C/C++ uygulamalarında invalid memory access gibi hataları tespit etmek amacıyla kullanılan bir araçtır.

İncelenen örnekte bilerek oluşturulmuş bir **Use-after-free** hatası bulunuyor.

```cpp
int main(int argc, char **argv) {
    int *array = new int[100];
    delete [] array;
    return array[argc];  // BOOM
}
```

Kodun çalışma mantığı oldukça basit:

1. `new int[100]` ile heap üzerinde 100 integer için memory ayrılıyor.
2. `delete [] array` ile ayrılan memory serbest bırakılıyor.
3. `return array[argc]` satırında ise artık geçerli olmayan memory alanından veri okunmaya çalışılıyor.

Buradaki problem, array'in daha önce oluşturulmuş olması değil, **serbest bırakıldıktan sonra kullanılmaya devam edilmesi**.

Bu işlem undefined behavior oluşturur. Programın her zaman aynı şekilde crash olacağı garanti değildir.

### AddressSanitizer ile Hatanın Tespiti

Resmi örnekte program AddressSanitizer etkinleştirilerek compile ediliyor:

```bash
clang++ -O1 -g -fsanitize=address -fno-omit-frame-pointer example_UseAfterFree.cc
```

Buradaki önemli parametre `-fsanitize=address` seçeneğidir. Compiler'ın memory erişimlerini kontrol edebilmek için gerekli instrumentation'ı eklemesini sağlar.

Program çalıştırıldığında resmi dokümantasyonda şu hata raporlanıyor:

```text
ERROR: AddressSanitizer: heap-use-after-free
READ of size 4
```

Rapor ayrıca hatalı memory erişiminin yapıldığı yeri, memory'nin daha önce nerede ayrıldığını ve nerede serbest bırakıldığını gösteriyor.

Bu sayede yalnızca programın crash ettiği noktayı değil, **memory'nin yaşam döngüsünü** de incelemek mümkün oluyor.

Bu örnekten çıkardığım en önemli sonuç, memory hatalarının yalnızca kodun çıktısına bakılarak her zaman tespit edilemeyeceği. AddressSanitizer gibi araçlar, normal testing sırasında fark edilmeyen invalid memory access problemlerini görünür hale getirebilir.

### Valgrind

Linux üzerinde memory analysis için kullanılan bir diğer önemli araç **Valgrind**'dir.

Valgrind içerisindeki **Memcheck**, özellikle memory leaks, invalid reads/writes ve hatalı deallocation işlemlerini tespit etmek amacıyla kullanılabilir.

Örneğin bir programın memory leak kontrolü şu şekilde gerçekleştirilebilir:

```bash
valgrind --leak-check=yes ./program
```

AddressSanitizer ve Valgrind farklı tekniklerle çalışır. Her ikisi de memory hatalarını tespit etmek için kullanılabilir, fakat performans maliyetleri ve tespit özellikleri farklıdır.

Burada önemli olan araçların tüm komutlarını öğrenmek değil, **hangi memory problemini araştırdığımıza göre uygun analysis tool'u seçmek**.

---

## Endüstride Memory Management

Memory Management'ın gerçek projelerde neden önemli olduğunu anlamak için Meta'nın Apache Spark üzerinde gerçekleştirdiği bir çalışmayı inceledim.

Meta mühendisleri, büyük veri işleme altyapısında Apache Spark kullanırken memory ve performance kaynaklı problemlerle karşılaşmış.

2016 yılında yayımlanan **Apache Spark @Scale: A 60 TB+ Production Use Case** çalışmasında, büyük veri kümelerinin işlenmesi sırasında yaşanan problemler ve bunlara uygulanan çözümler açıklanıyor.

### Meta - Apache Spark Memory Leak

Çalışmada tespit edilen problemlerden biri Spark'ın sorter component'ı içerisindeki memory leak.

Normalde task tamamlandığında kullanılan memory resources'ın serbest bırakılması beklenir.

Ancak mühendislerin tespit ettiği durumda memory pages serbest bırakılırken bunlarla ilişkili pointer array düzgün şekilde serbest bırakılmıyordu.

Bunun sonucunda:

- Kullanılmayan memory kaynakları tutulmaya devam ediyordu.
- Memory baskısı nedeniyle daha sık disk spilling gerçekleştiriliyordu.
- Bazı Spark executors out-of-memory problemleri yaşayabiliyordu.
- Büyük veri işlemlerinin performansı olumsuz etkileniyordu.

Problem **SPARK-14363** kaydı kapsamında ele alınmış ve ilgili memory'nin doğru şekilde serbest bırakılması sağlanmış.

Meta'nın yayımladığı rapora göre bu düzeltmenin ardından yaklaşık **%30 CPU performans iyileşmesi** gözlemlenmiş.

Bu örnek, memory problemlerinin yalnızca uygulamanın crash olmasına neden olmadığını gösteriyor. Yanlış memory management aynı zamanda performance kaybı ve sistem kaynaklarının verimsiz kullanılmasına da yol açabiliyor.

### Industrial Applications Açısından Değerlendirme

Benzer bir durum uzun süre çalışan sensor monitoring, image processing veya robot control applications içerisinde de ortaya çıkabilir.

Örneğin sürekli measurement alan bir uygulamada her frame için buffer oluşturuluyorsa bu buffer'ların yaşam döngüsünün doğru yönetilmesi gerekir.

Özellikle 7/24 çalışan sistemlerde küçük bir memory leak bile zamanla önemli bir probleme dönüşebilir.

Bu nedenle endüstride yalnızca uygulamanın doğru output üretmesi değil, uzun süreli çalışmalarda memory usage'ın takip edilmesi de önemlidir.

Memory hatalarını azaltmak için code review, static analysis, dynamic analysis ve long-duration testing gibi yöntemler birlikte kullanılabilir.

---

## Öğrendiklerim ve Kaynaklar

Memory Management araştırmasından çıkardığım temel sonuç, memory'nin yalnızca programın kullandığı bir kaynak olmadığı; aynı zamanda yazılımın reliability, performance ve security özelliklerini doğrudan etkilediğidir.

Özellikle dört nokta önemli:

1. **Memory yaşam döngüsü doğru yönetilmeli:** Allocation ve deallocation işlemlerinin hatalı uygulanması memory leak veya use-after-free gibi problemlere neden olabilir.
2. **Memory hataları her zaman hemen görünmez:** Program normal çalışıyormuş gibi görünürken arka planda memory problemleri oluşabilir.
3. **Analysis tools önemli:** AddressSanitizer ve Valgrind gibi araçlar invalid memory access ve memory leak problemlerini tespit etmeye yardımcı olur.
4. **Memory Management performansı da etkiler:** Meta'nın Apache Spark örneğinde görüldüğü gibi memory kaynaklarının yanlış yönetilmesi ciddi performance problemleri oluşturabilir.

AddressSanitizer örneği üzerinden `new`, `delete`, pointer ve use-after-free ilişkisini; Meta'nın Apache Spark çalışması üzerinden ise memory leak problemlerinin gerçek sistemlerdeki etkilerini incelemek mümkün.

### Kaynaklar

**Memory Management ve Analysis Tools**

- [LLVM/Clang - AddressSanitizer Documentation](https://clang.llvm.org/docs/AddressSanitizer.html)
- [Google Sanitizers - GitHub Repository](https://github.com/google/sanitizers)
- [Valgrind - Quick Start Guide](https://valgrind.org/docs/manual/quick-start.html)
- [Valgrind - Memcheck Documentation](https://valgrind.org/docs/manual/mc-manual.html)

**Industrial Memory Management**

- [Meta Engineering - Apache Spark @Scale: A 60 TB+ Production Use Case](https://engineering.fb.com/2016/08/31/core-infra/apache-spark-scale-a-60-tb-production-use-case/)
- [Apache Spark - SPARK-14363](https://issues.apache.org/jira/browse/SPARK-14363)

---

[Back to Main Documentation](../../README.md)

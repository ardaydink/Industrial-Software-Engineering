# Debugging & Root Cause Analysis

## Contents

1. [Debugging Nedir?](#debugging-nedir)
2. [Temel Terminoloji](#temel-terminoloji)
3. [Gerçek Kod Örneği - GDB](#gerçek-kod-örneği---gdb)
4. [Endüstride Debugging ve Root Cause Analysis](#endüstride-debugging-ve-root-cause-analysis)
5. [Öğrendiklerim ve Kaynaklar](#öğrendiklerim-ve-kaynaklar)

---

## Debugging Nedir?

Debugging, bir yazılımda tespit edilen hatanın neden oluştuğunu belirlemek ve problemi düzeltmek amacıyla yürütülen süreçtir.

Software Testing sırasında bir fonksiyonun beklenen sonucu üretmediğini tespit edebiliriz. Ancak bu sonuç, problemin hangi kod satırından veya hangi sistem bileşeninden kaynaklandığını doğrudan göstermez.

Debugging burada devreye girer.

Örneğin bir sensor application düşünelim. Sensör bağlantısı kesildiğinde uygulama kapanıyor olsun.

Bu durumda doğrudan kodu değiştirmek yerine şu soruları araştırmamız gerekir:

- Hata hangi koşullarda ortaya çıkıyor?
- Problem tekrar oluşturulabiliyor mu?
- Program kapanmadan önce hangi fonksiyonları çalıştırıyor?
- Değişkenlerin değerleri beklediğimiz gibi mi?
- Problemin gerçek nedeni sensor communication mı, yoksa yazılımdaki error handling eksikliği mi?

Genel olarak debugging sürecini şu şekilde düşünebiliriz:

**Reproduce → Logs & Debugging → Root Cause → Fix → Re-testing**

Burada önemli olan yalnızca programın yeniden çalışmasını sağlamak değil, hatanın neden oluştuğunu anlayarak aynı problemin tekrarlanmasını önlemektir.

## Temel Terminoloji

| Terim | Açıklama |
|---|---|
| Bug / Defect | Yazılımda bulunan hata |
| Debugger | Programın çalışma durumunu incelemeyi sağlayan araç |
| Breakpoint | Programın belirli bir noktada durdurulması |
| Step / Next | Kodun adım adım çalıştırılması |
| Stack Trace / Backtrace | Fonksiyonların çağrı zincirini gösteren bilgi |
| Logging | Programın çalışma sırasında olayları kayıt altına alması |
| Core Dump | Programın crash anındaki durumunun analiz için kaydedilmesi |
| Root Cause Analysis | Hatanın altında yatan gerçek nedenin araştırılması |

Bu araçlar aynı probleme farklı açılardan bakmamızı sağlar.

Örneğin logging, programın hata oluşmadan önce neler yaptığını anlamaya yardımcı olurken debugger, programın o andaki değişkenlerini ve execution flow'unu incelememizi sağlar.

## Gerçek Kod Örneği - GDB

Debugging'in nasıl çalıştığını anlamak için GitHub üzerinde yayımlanmış bir C++ GDB tutorial örneğini inceledim.

**GDB (GNU Debugger)**, Linux üzerinde özellikle C ve C++ programlarının debugging işlemlerinde kullanılan araçlardan biridir.

Örnekte aşağıdaki fonksiyon yer alıyor:

```cpp
int faulty_function(int x) {
    int* ptr = nullptr;
    *ptr = 42;
    return x + 1;
}
```

Burada `ptr` isimli pointer `nullptr` olarak tanımlanmış.

Ancak hemen sonraki satırda:

```cpp
*ptr = 42;
```

ile geçerli bir memory adresi göstermeyen pointer üzerinden veri yazılmaya çalışılıyor.

Bu işlem **undefined behavior** oluşturur ve Linux üzerinde tipik olarak `Segmentation Fault` ile sonuçlanır.

### GDB ile Hatanın İncelenmesi

Tutorial'da program önce debug symbols içerecek şekilde compile ediliyor.

```bash
g++ -g -O0 main.cpp -o program
```

Buradaki `-g`, debugger'ın source code ve değişkenler hakkında bilgi alabilmesini sağlar.

Program GDB üzerinden başlatılabilir:

```bash
gdb ./program
```

Ardından kullanılan temel komutlar:

| GDB Command | Görevi |
|---|---|
| `run` | Programı çalıştırır |
| `break main` | `main()` fonksiyonuna breakpoint koyar |
| `next` | Sonraki source line'a geçer |
| `step` | Çağrılan fonksiyonun içine girer |
| `print ptr` | Pointer değerini gösterir |
| `backtrace` | Call stack bilgisini gösterir |

Program geçersiz memory erişimi sırasında durduğunda `backtrace` kullanılarak hangi fonksiyon içerisinde hata oluştuğu ve bu fonksiyonun nereden çağrıldığı incelenebilir.

`print ptr` komutu ile pointer'ın null olduğu doğrulanabilir.

Bu örnekte hata oldukça basit olsa da aynı yöntem büyük projelerde çok daha karmaşık problemleri araştırmak için kullanılabilir.

Ancak debugger'ın gösterdiği satır her zaman problemin gerçek başlangıç noktası olmayabilir.

Örneğin bir sensor object'ın initialization işlemi başarısız olmuş, fakat bu durum kontrol edilmeden object kullanılmaya çalışılmış olabilir.

Böyle bir durumda segmentation fault hatanın görünen sonucu, eksik initialization veya error handling ise asıl root cause olabilir.

**Bu örnekten çıkardığım sonuç:** Debugging yalnızca crash olan satırı bulmak değil, programın o noktaya neden geldiğini anlamaktır.

---

## Endüstride Debugging ve Root Cause Analysis

Profesyonel yazılım projelerinde debugging yalnızca developer'ın bilgisayarında gerçekleştirilen bir işlem değildir.

Özellikle production ortamında oluşan problemlerin incelenmesi için logs, monitoring, error reports, code changes ve deployment geçmişi birlikte değerlendirilebilir.

Bu yaklaşımı anlamak için GitLab'ın yayımladığı gerçek bir **Incident Review** kaydını inceledim.

### GitLab Production Incident - June 2025

26 Haziran 2025 tarihinde GitLab'ın Security Dashboard özelliğinde bazı kullanıcılar `HTTP 500` hataları almaya başladı.

Bu, kullanıcıların ilgili security dashboard'larına erişmesini engelliyordu.

Engineering ekibi problemi araştırırken yakın zamanda yapılan bir Merge Request'in SQL function type mismatch oluşturduğunu belirledi.

Yani uygulamanın kullandığı SQL function ile database tarafındaki veri tipleri arasında uyumsuzluk oluşmuştu.

Ayrıca development ve test ortamlarındaki column type tanımları ile staging/production ortamlarındaki gerçek column type'ların birbirinden farklı olduğu anlaşıldı.

Bu nedenle problem development sürecinde tespit edilememişti.

Ekip önce problemi oluşturan değişikliği geri almak için **revert** işlemi gerçekleştirdi. Ardından düzeltme production ortamına aşamalı şekilde aktarıldı ve error logs takip edilerek problemin ortadan kalktığı doğrulandı.

Süreç şu şekilde özetlenebilir:

**HTTP 500 Errors → Investigation → SQL Type Mismatch → Revert → Deployment → Monitoring**

Bu örnek bize debugging'in yalnızca GDB gibi bir araç kullanmaktan ibaret olmadığını gösteriyor.

Gerçek bir production incident sırasında:

- Logs ve monitoring verileri inceleniyor.
- Son yapılan code changes araştırılıyor.
- Problemin root cause'u belirleniyor.
- Gerekli fix veya revert uygulanıyor.
- Production ortamındaki davranış tekrar gözlemleniyor.

GitLab'ın Incident Review yaklaşımında yalnızca anlık problemin çözülmesi değil, aynı tür hatanın tekrar oluşmasını engelleyecek corrective actions'ın belirlenmesi de önemli.

Bu örnekte özellikle dikkatimi çeken nokta, problemin kaynağının yalnızca yanlış bir SQL işlemi olmaması. Development ve production environments arasındaki uyumsuzluk da hatanın test aşamasından kaçmasına katkıda bulunmuş.

Dolayısıyla **Root Cause Analysis**, bazen tek bir kod satırından çok daha geniş bir development sürecinin incelenmesini gerektirebilir.

---

## Öğrendiklerim ve Kaynaklar

Debugging araştırmasından çıkardığım temel sonuç, yazılım hatalarının yalnızca görünen sonuçlarına odaklanmanın yeterli olmadığıdır.

Özellikle dört nokta önemli:

1. **Reproduce önemli:** Bir hatanın hangi koşullarda gerçekleştiğini bilmek, debugging sürecini kolaylaştırır.
2. **GDB gibi araçlar kullanışlı:** Breakpoint, variable inspection ve backtrace sayesinde programın çalışma davranışı incelenebilir.
3. **Symptom ile Root Cause farklı olabilir:** Crash olan satır, hatanın ilk oluştuğu yer olmak zorunda değildir.
4. **Production debugging daha kapsamlıdır:** Logs, monitoring, code history ve deployment bilgileri birlikte değerlendirilir.

GDB örneği sayesinde bir programın nasıl adım adım incelenebileceğini, GitLab Incident Review örneği sayesinde ise profesyonel bir engineering team'in production ortamındaki problemi nasıl araştırdığını görmek mümkün.

### Kaynaklar

**Debugging ve GDB**

- [GDB Cheat Sheet & C++ Debugging Tutorial](https://gist.github.com/shamim-hussain/f4570d236c855f2407ad4235b3e39a1a)
- [GNU Debugger - Official Documentation](https://sourceware.org/gdb/current/onlinedocs/gdb.html)

**Industrial Debugging & Root Cause Analysis**

- [GitLab - Incident Review Process](https://handbook.gitlab.com/handbook/engineering/infrastructure-platforms/incident-review/)
- [GitLab - Incident Review: 500 Error on Security Dashboard](https://gitlab.com/gitlab-com/gl-infra/production/-/issues/20082)

---

[Back to Main Documentation](../../README.md)

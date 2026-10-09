# Linux-Based Systems

## Contents

1. [Linux-Based Systems Nedir?](#linux-based-systems-nedir)
2. [Temel Terminoloji](#temel-terminoloji)
3. [Gerçek GitHub Örneği - systemd Service](#gerçek-github-örneği---systemd-service)
4. [Endüstride Linux - Netflix](#endüstride-linux---netflix)
5. [Öğrendiklerim ve Kaynaklar](#öğrendiklerim-ve-kaynaklar)

---

## Linux-Based Systems Nedir?

Linux, özellikle servers, embedded devices, robotics ve industrial automation gibi alanlarda kullanılan açık kaynak bir işletim sistemi altyapısıdır.

Teknik olarak **Linux**, işletim sisteminin Kernel kısmını ifade eder. Ubuntu, Debian ve Red Hat Enterprise Linux gibi dağıtımlar ise Linux Kernel ile birlikte çeşitli system tools, libraries ve applications içerir.

Linux'un endüstriyel projelerde tercih edilmesinin önemli nedenlerinden bazıları:

- Open-source olması ve özelleştirilebilmesi,
- C/C++, Python ve ROS gibi teknolojileri desteklemesi,
- Network ve hardware interfaces için geniş destek sunması,
- Processes, services ve system resources üzerinde kontrol sağlaması,
- Uzaktan erişim, monitoring ve automation araçlarıyla birlikte kullanılabilmesi.

Örneğin bir industrial robot içerisinde Ubuntu kullanan bir Industrial PC bulunduğunu düşünelim.

Bu bilgisayarda aynı anda sensor communication, image processing, motor control ve logging uygulamaları çalışabilir.

Linux bu uygulamalara memory management, process scheduling, file system ve hardware access gibi temel hizmetleri sağlar.

Basitleştirilmiş sistem yapısı:

**Hardware → Linux Kernel → System Libraries / Services → User Applications**

Burada özellikle **User Space** ve **Kernel Space** ayrımını bilmek önemli.

User Space, normal uygulamaların çalıştığı alandır.

Kernel Space ise işletim sisteminin hardware, memory ve processes gibi kaynakları yönettiği alandır.

Bir uygulama Ethernet üzerinden sensörle haberleşirken doğrudan network kartını kontrol etmek yerine genellikle Linux'un networking interfaces ve drivers altyapısını kullanır.

Bu sayede developer, hardware üzerindeki bütün düşük seviyeli işlemleri sıfırdan geliştirmek zorunda kalmaz.

---

## Temel Terminoloji

| Terim | Açıklama |
|---|---|
| Kernel | Hardware ile applications arasındaki temel kaynak yönetimini sağlayan işletim sistemi çekirdeği |
| Linux Distribution | Linux Kernel ile birlikte system tools ve packages içeren işletim sistemi dağıtımı |
| Process | Çalışmakta olan programın işletim sistemi tarafından yönetilen instance'ı |
| Daemon / Service | Genellikle arka planda çalışan ve belirli hizmet sağlayan uygulama |
| systemd | Birçok Linux dağıtımında services ve system startup süreçlerini yöneten sistem |
| Shell | Terminal üzerinden komutların çalıştırılmasını sağlayan arayüz |
| File Permissions | Dosya ve kaynaklara erişim yetkilerini belirleyen mekanizma |
| Device Driver | İşletim sistemi ile hardware arasındaki iletişimi sağlayan yazılım |
| System Logs | İşletim sistemi ve uygulamalarda gerçekleşen olayların kayıtları |
| SSH | Başka bir bilgisayara network üzerinden güvenli uzaktan erişim sağlayan protokol |

Linux üzerinde kullanılan bütün uygulamalar aynı şekilde yönetilmez.

Bazı uygulamalar doğrudan terminal üzerinden çalıştırılırken, bazıları background service olarak çalışır.

Örneğin bir sensor monitoring application'ın bilgisayar her açıldığında otomatik başlaması istenebilir.

Bu tür ihtiyaçlar için `systemd` kullanılabilir.

---

## Gerçek GitHub Örneği - systemd Service

Linux üzerinde applications'ın nasıl yönetildiğini anlamak için **Prometheus Node Exporter** projesinin GitHub repository'sini inceledim.

Prometheus, sistemlerin ve uygulamaların çeşitli metrics değerlerini toplayarak monitoring yapılmasını sağlayan açık kaynak bir projedir.

**Node Exporter** ise Linux gibi sistemlerde CPU, memory, disk ve network gibi kaynaklara ilişkin metrics sağlar.

Repository içerisinde uygulamanın `systemd` üzerinden çalıştırılması için hazırlanmış gerçek bir servis dosyası bulunuyor.

İncelenen dosya:

`examples/systemd/node_exporter.service`

```ini
[Unit]
Description=Node Exporter
Requires=node_exporter.socket

[Service]
User=node_exporter
Environment=OPTIONS=
EnvironmentFile=-/etc/sysconfig/node_exporter
ExecStart=/usr/sbin/node_exporter --web.systemd-socket $OPTIONS

[Install]
WantedBy=multi-user.target
```

Bu dosya bir C++ source code değil, **systemd service configuration** dosyasıdır.

Kodun önemli bölümleri:

- **[Unit]:** Servisin tanımını ve diğer unit'lerle ilişkilerini belirtir.
- **Requires:** Servisin ihtiyaç duyduğu başka bir unit'i tanımlar.
- **[Service]:** Uygulamanın nasıl çalıştırılacağını belirler.
- **User:** Uygulamanın hangi Linux kullanıcısının yetkileriyle çalışacağını belirtir.
- **ExecStart:** Çalıştırılacak programı ve parametrelerini tanımlar.
- **[Install]:** Servisin `systemctl enable` ile etkinleştirilmesine ilişkin bilgileri içerir.

Burada `ExecStart` satırı özellikle önemli.

Çünkü Linux'a hangi uygulamanın çalıştırılacağını söylüyor:

```ini
ExecStart=/usr/sbin/node_exporter --web.systemd-socket $OPTIONS
```

Bu örnek aynı zamanda `node_exporter.socket` isimli başka bir unit'e ihtiyaç duyuyor.

Repository'deki ilgili socket configuration:

```ini
[Socket]
ListenStream=9100
```

Bu tanım, Node Exporter için TCP 9100 portunun systemd tarafından yönetilmesini sağlıyor.

Yani Linux yalnızca uygulamayı başlatmakla kalmıyor; service dependencies, execution environment ve network socket gibi konuların yönetiminde de rol oynuyor.

### systemctl ve journalctl

Linux üzerinde bir systemd service'in durumunu incelemek için bazı temel komutlar kullanılabilir:

```bash
systemctl status node_exporter
systemctl start node_exporter
systemctl stop node_exporter
journalctl -u node_exporter
```

| Command | Görevi |
|---|---|
| `systemctl status` | Servisin durumunu görüntüler |
| `systemctl start` | Servisi başlatır |
| `systemctl stop` | Servisi durdurur |
| `systemctl enable` | Servisin uygun boot target kapsamında otomatik başlamasını etkinleştirir |
| `journalctl -u` | Belirli bir servise ait log kayıtlarını gösterir |

Bu komutlar ilgili servis sisteme kurulmuşsa kullanılabilir. Ayrıca örnekteki executable path, kullanıcı hesabı ve socket configuration'ın sistemde mevcut olması gerekir.

**Bu örnekten çıkardığım sonuç:** Endüstride geliştirilen bir uygulamanın yalnızca çalışması yeterli değil. Nasıl başlatıldığı, hangi yetkilerle çalıştığı, nasıl takip edildiği ve hata durumunda nasıl incelendiği de önemlidir.

---

## Endüstride Linux - Netflix

Linux'un profesyonel ekiplerde nasıl kullanıldığını anlamak için Netflix'in Performance Engineering ekibinin yayımladığı **Linux Performance Analysis in 60,000 Milliseconds** başlıklı çalışmayı inceledim.

Netflix, büyük ölçekli cloud infrastructure üzerinde Linux servers kullanıyor.

Bu sistemlerde bir performans veya kaynak problemi oluştuğunda mühendisler yalnızca uygulama kodunu incelemiyor.

Öncelikle Linux üzerinde çalışan sistemin genel durumunu değerlendiriyorlar.

Netflix mühendislerinin yayımladığı yöntemde, bir Linux server'a bağlandıktan sonraki ilk 60 saniyede temel system metrics kontrol ediliyor.

Bu kontrollerin bazıları:

| Linux Command | İncelenen Bilgi |
|---|---|
| `uptime` | Sistemin çalışma süresi ve load averages |
| `dmesg \| tail` | Son kernel messages ve olası system errors |
| `vmstat 1` | CPU, memory ve I/O ile ilgili genel istatistikler |
| `iostat -xz 1` | Disk I/O davranışı |
| `free -m` | Memory kullanım durumu |
| `top` | Çalışan processes ve resource usage |

Bu komutların amacı problemi ilk dakikada tamamen çözmek değil, **problemin hangi sistem kaynağıyla ilişkili olabileceğini belirlemek**.

Örneğin uygulama yavaş çalışıyorsa:

- CPU yoğun şekilde kullanılıyor olabilir.
- Memory baskısı oluşmuş olabilir.
- Disk I/O işlemleri darboğaz oluşturuyor olabilir.
- Kernel logs içerisinde önemli bir hata bulunabilir.

Bu kontrollerden sonra mühendisler daha ayrıntılı debugging veya profiling çalışmalarına geçebilir.

Netflix'in yayımladığı çalışma tek bir incident report değil; ekibin Linux üzerinde uyguladığı genel araştırma yönteminin açıklamasıdır.

### Industrial Applications Açısından Değerlendirme

Benzer bir yaklaşım industrial robot veya sensor monitoring applications için de kullanılabilir.

Örneğin Ubuntu üzerinde Ethernet bağlantısıyla çalışan bir laser sensor application düşünelim.

Sensörden veri alınamadığında problemin mutlaka application code içerisinden kaynaklandığını varsaymak doğru olmaz.

Öncelikle Linux tarafında network interface ve communication durumu kontrol edilebilir:

```bash
ip addr show
ip route
ping <sensor_ip>
```

Bu komutlarla network interface'in IP configuration'ı, routing bilgisi ve sensöre IP seviyesinde erişilip erişilemediği incelenebilir.

Ancak `ping` komutunun başarılı olması, sensor application's protocol communication işlemlerinin de doğru çalıştığını garanti etmez.

Bu nedenle problem hem operating system hem de application seviyesinde değerlendirilmelidir.

**Bu araştırmadan çıkardığım sonuç:** Linux bilmek yalnızca terminal komutlarını kullanabilmek değil, application ile operating system arasındaki ilişkiyi anlayarak problemleri sistematik şekilde araştırabilmektir.

---

## Öğrendiklerim ve Kaynaklar

Linux-Based Systems araştırmasından çıkardığım temel sonuç, Linux'un endüstride yalnızca uygulamaların çalıştığı bir platform olmadığıdır.

Aynı zamanda hardware access, process management, service management, networking ve monitoring gibi birçok mühendislik faaliyetinin temelini oluşturur.

Özellikle dört nokta önemli:

1. **Kernel ve User Space ayrımı:** Uygulamalar hardware kaynaklarına genellikle kernel tarafından sağlanan interfaces üzerinden erişir.
2. **Service Management önemli:** Gerçek projelerde uygulamaların başlatılması, durdurulması ve takip edilmesi systemd gibi araçlarla gerçekleştirilebilir.
3. **System Logs debugging için önemli:** Bir problem yalnızca application code içerisinde değil, operating system veya hardware interaction seviyesinde de oluşabilir.
4. **Linux commands problem analizinin parçası:** Netflix örneğinde görüldüğü gibi sistem kaynaklarının incelenmesi profesyonel debugging süreçlerinde önemli bir adımdır.

Prometheus Node Exporter örneği üzerinden bir Linux service configuration dosyasının nasıl hazırlandığını, Netflix'in mühendislik çalışması üzerinden ise Linux sunucularında sistem problemlerinin nasıl araştırıldığını incelemek mümkün.

### Kaynaklar

**Linux ve Service Management**

- [Linux Kernel - Official Documentation](https://docs.kernel.org/)
- [systemd - GitHub Repository](https://github.com/systemd/systemd)
- [systemctl - Linux Manual](https://man7.org/linux/man-pages/man1/systemctl.1.html)
- [journalctl - Linux Manual](https://man7.org/linux/man-pages/man1/journalctl.1.html)

**Gerçek GitHub Örneği - Prometheus**

- [Prometheus Node Exporter - GitHub Repository](https://github.com/prometheus/node_exporter)
- [Node Exporter - systemd Service File](https://github.com/prometheus/node_exporter/blob/master/examples/systemd/node_exporter.service)
- [Node Exporter - systemd Socket File](https://github.com/prometheus/node_exporter/blob/master/examples/systemd/node_exporter.socket)
- [Prometheus - Monitoring Linux Host Metrics](https://prometheus.io/docs/guides/node-exporter/)

**Industrial Linux - Netflix**

- [Netflix Performance Engineering - Linux Performance Analysis in 60,000 Milliseconds](https://www.linux.com/news/linux-performance-analysis-60000-milliseconds/)
- [Brendan Gregg - Linux Performance Analysis in 60s](https://www.brendangregg.com/blog/2015-12-03/linux-perf-60s-video.html)

---

[Back to Main Documentation](../../README.md)

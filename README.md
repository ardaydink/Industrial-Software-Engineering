# Industrial Software Engineering

Bir yazılımın çalışıyor olması, onun endüstride kullanılmaya hazır olduğu anlamına gelmez.

Industrial software yalnızca doğru sonuç üretmekle kalmamalı; aynı zamanda test edilebilir, debug edilebilir, sistem kaynaklarını kontrollü kullanabilen, timing requirements'ı karşılayan, donanım ve diğer yazılım bileşenleriyle güvenilir şekilde haberleşebilen ve failure durumlarında öngörülebilir davranabilen bir yapıya sahip olmalıdır.

Bu dokümantasyon, çalışan bir yazılımın **industrial software** seviyesine ulaşırken karşılaştığı temel engineering problemlerini ve bu problemlere endüstride nasıl yaklaşıldığını adım adım incelemek amacıyla hazırlanmıştır.

---

## About This Documentation

Bu repository, industrial software development sürecinde karşılaşılan temel konuları tek bir kaynak altında toplamak amacıyla oluşturulmuştur.

Amaç yalnızca kavramların tanımlarını vermek değil; her konunun:

- neden önemli olduğunu,
- hangi problemi çözmeye çalıştığını,
- endüstride nasıl karşımıza çıktığını,
- hangi yöntem ve araçlarla ele alındığını,
- diğer software engineering konularıyla nasıl ilişkili olduğunu

açıklamaktır.

---

## Learning Roadmap

Araştırdığım konuları, endüstriyel yazılım geliştirmenin farklı alanlarına göre beş ana grupta topladım. Her başlığa tıklayarak ilgili araştırma notlarına, kod örneklerine ve endüstriyel uygulamalara ulaşabilirsiniz.

### Software Quality & Verification

- [01. Software Testing](industrial-software-engineering/software-quality-verification/01-software-testing.md)
- [02. Debugging & Root Cause Analysis](industrial-software-engineering/software-quality-verification/02-debugging-root-cause-analysis.md)

### Runtime Reliability

- [03. Memory Management & Memory Safety](industrial-software-engineering/runtime-reliability/03-memory-management.md)
- [04. Multithreading & Concurrency](industrial-software-engineering/runtime-reliability/04-multithreading-concurrency.md)
- [05. Performance & Profiling](industrial-software-engineering/runtime-reliability/05-performance-profiling.md)

### Platform & Timing

- [06. Linux-Based Systems](industrial-software-engineering/platform-timing/06-linux-based-systems.md)
- [07. Real-Time Systems](industrial-software-engineering/platform-timing/07-real-time-systems.md)

### System Integration

- [08. Hardware & Software Communication](industrial-software-engineering/system-integration/08-hardware-software-communication.md)

### Governance, Safety & Compliance

- [09. Open Source Software in Industry](industrial-software-engineering/governance-safety-compliance/09-open-source-software-in-industry.md)
- [10. Industrial Software Standards](industrial-software-engineering/governance-safety-compliance/10-industrial-software-standards.md)

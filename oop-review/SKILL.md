---
name: oop-review
description: Pragmatic OOP/architecture review with battle-tested filters against false positives. Covers SOLID, DRY, KISS, design patterns. Prevents idealistic over-reporting.
---

# OOP Architecture Review

Pragmatik OOP mimari review skill'i. Teorik pattern tanımlama değil, gerçek değer üreten bulgular üretmeyi hedefler.

## ALTIN KURAL

Pattern tanımlamak ≠ "bunu düzelt" demek. Her bulguya pragmatik filtre uygula.

## PRAGMATIK FILTRE KURALLARI

### 1. ISP (Interface Segregation) Filtresi

**Bulgu geçerli SADECE şu koşullarda:**
- Interface'in birden fazla bağımsız consumer'ı var
- Consumer'lar method'ların <%50'sini kullanıyor
- Bölme sonrası consumer'lar gerçekten farklı sub-interface'leri kullanacak

**Bulgu GEÇERSİZ şu durumlarda:**
- Tek consumer var ve %70+ method kullanıyor
- Interface documentation/contract amaçlı (polimorfik kullanım yok)
- Bölme sonrası intersection type ile birleştirme gerekecek (aynı şey, ekstra ceremony)

**Why:** Tek consumer'lı interface'i bölmek sıfır kazancı olan ceremony ekler.

### 2. DRY (Don't Repeat Yourself) Filtresi

**Bulgu geçerli SADECE şu koşullarda:**
- Tekrar eden kod semantik olarak aynı işi yapıyor
- Extraction sonrası net satır azalması var (shared utility < toplam tekrar)
- Ortak abstraction doğal ve zorlanmış değil

**Bulgu GEÇERSİZ şu durumlarda:**
- Yapısal benzerlik var ama semantik farklı (farklı SQL pattern'leri, farklı domain kuralları)
- Extraction için interface/config + runtime dispatch gerekiyor ve net satır ARTIYOR
- Fonksiyonlar zaten tek satırlık delegation (binary op wrapper'lar gibi)
- Ortak olan sadece constructor çağrısı (return new X(node))

**Why:** Yüzeysel yapısal benzerliği DRY ihlali sanmak, semantik farklılığı görmezden gelmektir.

### 3. SRP (Single Responsibility) Filtresi

**Bulgu geçerli SADECE şu koşullarda:**
- Class gerçekten birden fazla bağımsız sorumluluk taşıyor
- Sorumluluklar farklı değişim nedenleriyle değişiyor
- Class büyüklüğü delegation'dan değil, iç karmaşıklıktan kaynaklanıyor

**Bulgu GEÇERSİZ şu durumlarda:**
- Class bir delegation facade (thin pass-through'lar + shared helpers)
- Strategy selection küçük (<20 satır) — factory extraction anlamsız
- Büyüklük static helper + delegation + shared utility'den kaynaklanıyor

**Why:** Delegation facade'ları çok sorumluluk taşıyor GIBI görünür ama asıl işi strategy'lere delege ederler.

### 4. Type Splitting Filtresi

**Bulgu geçerli SADECE şu koşullarda:**
- Field overlap <%50
- Variant'lar tamamen farklı davranış sergiliyor
- Mevcut type guard'lar karmaşık ve hata-prone

**Bulgu GEÇERSİZ şu durumlarda:**
- Field overlap >%80 — bölme DRY'a zarar verir
- Mevcut if guard'ları basit ve çalışıyor
- Parser her iki variant'ı bağımsız set edebiliyor
- Bölme sonrası BaseType + 2 extension = 3 tip (1 yerine) + migration maliyeti

**Why:** Yüksek overlap'li tipleri bölmek, tekrarı artırır ve migration maliyeti yaratır.

### 5. Factory Extraction Filtresi

**Bulgu geçerli SADECE şu koşullarda:**
- Object creation logic karmaşık (>30 satır)
- Birden fazla yerde aynı creation logic tekrarlanıyor
- Creation logic'in test edilmesi gerekiyor

**Bulgu GEÇERSİZ şu durumlarda:**
- Selection logic minimal (<20 satır if/else veya switch)
- Tek yerde kullanılıyor (getter veya constructor içinde)
- Extraction: dosya taşıma + context passing overhead > kazancı

**Why:** 14 satırlık bir getter'ı factory class'a çıkarmak, keşfedilebilirliği düşürür ve net kayba yol açar.

## SEVERITY TANIMLARI

Pragmatik filtreyi geçen bulgular şu kategorilere ayrılır:

### Gerçek Teknik Borç
Aktif olarak zarar veriyor. Düzeltilmeli.
- Tip güvenliği kapalı (@ts-nocheck, @ts-ignore istismarı)
- Güvenlik açıkları
- Runtime hatalarına yol açan design sorunları
- Circular dependency'ler (break edilmemiş)

### İyileştirme Fırsatı
Aktif zarar yok ama geliştirilebilir. Backlog'a alınabilir.
- Pragmatik filtreyi geçen DRY ihlalleri (semantik tekrar, net satır azalması)
- Eksik error handling (system boundary'lerde)
- Weak encapsulation (underscore prefix vs proper visibility)

### Nice-to-Have
Teorik olarak doğru ama pragmatik kazancı minimal. Yapma.
- Pattern doğru ama tek consumer / düşük satır / yüksek overlap
- Refactoring riski > kazancı
- Mevcut kod çalışıyor ve test ediliyor

## REVIEW PROSEDÜRÜ — SWARM + FILTER

### Adım 1: Paralel Review Agent'ları Başlat

5 bağımsız review agent'ı **aynı anda** başlat. Her biri aynı görevi alır:

```
"[paket/dizin yolu]'nu OOP/mimari açısından incele. SOLID, DRY, KISS, design pattern kullanımını değerlendir. Coupling / cohesion analizi, Layer violations, Transaction boundary leaks, Error propagation design, Async flow complexity, state management patterns konusunda gözlemler yap., Odakları kategorilendirerek odak başına alt agentler kullanabilirsin., Ham bulgularını severity ile birlikte raporla."
```

**Neden 5?** Tek agent kendi bias'ına kapılır. 5 bağımsız agent aynı kodu farklı açılardan inceler — gerçek sorunlar çoğunluğun bulduğu, sahte sorunlar tek agent'ın bulduğu şeylerdir.

**Agent'lar kendi iç yapılarını özgürce belirler** — paket başına sub-agent kullanabilir, dosya bazlı inceleme yapabilir, vb. Biz sadece nihai bulgu listesini alırız.

### Adım 2: Bulguları Topla ve Birleştir

5 agent'tan gelen tüm bulguları tek tabloda birleştir:

| Bulgu | Agent 1 | Agent 2 | Agent 3 | Agent 4 | Agent 5 | Oy |
|-------|---------|---------|---------|---------|---------|-----|
| X sorunu | ✓ | ✓ | ✓ | — | ✓ | 4/5 |
| Y sorunu | ✓ | — | — | — | — | 1/5 |

### Adım 3: Konsensüs Filtresi
- **Olumlu bulgu** İyi uygulanmış patternleri tespit ettiğinde gereksiz refactoru önlemek için konsesüs filtresi uygula ve raporda güçlü yönler olarak vurgula.
- **3+ agent aynı bulguyu raporladı** → Adım 4'e geç (pragmatik filtre)
- **1-2 agent raporladı** → Otomatik "Nice-to-Have" — muhtemelen idealistik bulgu. Sadece kullanıcı sorarsa değerlendir.
- **Çelişkili bulgular** (bir agent "sorun" diyor, diğeri "sorun değil") → Main agent kodu okuyup karar verir. fakat yine de kullanıya raporlanır. Çelişkili bulgular genellikle karmaşık, borderline durumları yansıtır ve dikkatle değerlendirilmelidir.
- **Güvenlikle Alakalı Bulguler** — Güvenlik açıkları için düşük konsensüs bile önemli olabilir. Güvenlik kritik bulgular, özellikle potansiyel olarak istismar edilebilir olanlar, genellikle daha düşük konsensüsle bile raporlanmalıdır. Bu tür bulgular, güvenlik uzmanları tarafından öncelikli olarak ele alınmalıdır.

### Adım 4: Pragmatik Filtre (Main Agent)

Konsensüs filtresini geçen her bulguya bu skill'deki pragmatik filtre kurallarını uygula (ISP, DRY, SRP, Type Splitting, Factory Extraction filtreleri).

### Adım 5: Raporla

1. **Gerçek Teknik Borç** — filtreyi geçen, 3+ agent konsensüsü olan bulgular
2. **İyileştirme Fırsatı** — filtreyi geçen, pragmatik kazancı olan bulgular
3. **Reddedilen Bulgular** — konsensüs veya pragmatik filtre tarafından elenen, neden elendiği açıklamasıyla

## ANTI-PATTERN: TEMİZ KOD İDEALİZMİ

Şunlardan kaçın:
- Tek consumer'lı interface'i bölme önerisi
- Yüzeysel yapısal benzerliği DRY ihlali olarak raporlama
- Delegation facade'ı SRP ihlali olarak raporlama
- %80+ field overlap'li tipi bölme önerisi
- <20 satır selection logic için factory extraction önerisi
- Zaten tek satıra delegate eden wrapper'ları "tekrar" olarak raporlama

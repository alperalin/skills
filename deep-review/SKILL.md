---
name: deep-review
description: >
  Deep review on given scope. Multi-phase, multi-agent pipeline for comprehensive analysis. Strict isolation of findings. No batching.
user-invocable: true
tools:vscode, execute, read, agent, browser, edit, search, web, todo
---

> **EXECUTION MODE**: Bu SKILL bir pipeline tanımıdır. Her PHASE sırasıyla çalıştırılacak. Her phase içindeki TASK'lar paralel sub-agent (Task tool) ile çalıştırılacak. Hiçbir phase atlanmayacak, birleştirilmeyecek, basitleştirilmeyecek. Her phase'in çıktısı belirtilen dosyaya persist edilecek. Sonraki phase ancak önceki phase'in çıktı dosyası diske yazıldıktan sonra başlayacak.

---

## GLOBAL RULES

```
0. EĞER SCOPE VERİLMEDİYSE SOR MUTLAKA SOR. SCOPE NET OLMALI. (örneğin: "test dosyaları", "src/ dizini", "tüm kod ve test dosyaları", "sadece docs/ dizini")
1. Her TASK bir sub-agent (Task tool) olarak spawn edilecek.
2. Bir phase içindeki TASK'lar paralel çalışacak.
3. Phase'ler sıralı çalışacak (PHASE-1 → PHASE-2 → PHASE-3 → PHASE-4 → PHASE-5).
4. Her phase sonunda OUTPUT_FILE'a yazılacak. Yazılmadan sonraki phase başlamayacak.
5. Sub-agent'ler birbirinin context'ini görmeyecek — her biri kendi scope'unda çalışacak.
6. Skorlama: 1-10 arası. 7+ = geçerli bulgu, 4-6 = tartışmalı, 1-3 = geçersiz.
7. Tüm çıktılar Türkçe olacak.
8. Artifact dizini: ./deep-review-artifacts-{timestamp}/ (yoksa oluştur)
```

---

## ⛔ STRICT ISOLATION CONSTRAINT — BATCHING YASAĞI

```
AŞAĞIDAKİ KURAL PHASE-2, PHASE-3 VE PHASE-4 İÇİN GEÇERLİDİR.
BU KURAL İHLAL EDİLEMEZ, OPTİMİZE EDİLEMEZ, YORUMLANAMAZ.

┌─────────────────────────────────────────────────────────────┐
│  1 TASK = 1 SUB-AGENT = 1 TEK KONU                         │
│                                                             │
│  • PHASE-2: 1 sub-agent = 1 feature. Sadece 1.             │
│  • PHASE-3: 1 sub-agent = 1 finding. Sadece 1.             │
│  • PHASE-4: 1 sub-agent = 1 finding. Sadece 1.             │
│                                                             │
│  Bir sub-agent'e ASLA 2 veya daha fazla feature/finding     │
│  VERİLMEYECEK. Bu bir performans optimizasyonu değildir,    │
│  bu bir kalite gereksinimidir.                              │
└─────────────────────────────────────────────────────────────┘

YASAKLANAN PATTERN'LER (bunları yapma):
  ✗ "F-001, F-002 ve F-003'ü birlikte incele" → YASAK
  ✗ "Şu 5 feature'ı batch olarak analiz et" → YASAK
  ✗ "Benzer feature'ları grupla ve tek agent'e ver" → YASAK
  ✗ "Verimlilik için birden fazla finding'i tek agent'e ata" → YASAK
  ✗ Tek bir Task tool çağrısının prompt'una birden fazla feature listesi koymak → YASAK

ZORUNLU PATTERN (bunu yap):
  ✓ Her feature için AYRI bir Task tool çağrısı yap
  ✓ Her Task tool çağrısının prompt'unda TAM OLARAK 1 feature olacak
  ✓ Feature sayısı = Task tool çağrısı sayısı. Eşit olmalı.
  ✓ 30 feature varsa → 30 ayrı Task tool çağrısı

NEDEN: Batch yapıldığında agent dikkatini böler, ilk feature'a odaklanıp
sonrakileri yüzeysel inceler. Bu, gerçek bugların gözden kaçmasına neden
olur. Her feature kendi bağımsız, derin analizini hak ediyor.
```

---

## PHASE-1: FEATURE CATALOG EXTRACTION (Paralel Swarm)

**AMAÇ**: Scope'un tüm feature'larının kapsamlı listesini çıkarmak. Üç farklı kaynaktan bağımsız olarak.

**OUTPUT_FILE**: `./deep-review-artifacts-{timestamp}/phase1-feature-catalog.md`

### TASK-1A: Test-Based Feature Extraction
```
SCOPE: Yalnızca test dosyaları (test/, tests/, **/*.test.*, **/*.spec.*)
GÖREV: Her test dosyasını oku. Test edilen feature'ları çıkar.
FORMAT:
  - feature_id: T-001, T-002, ...
  - feature_name: kısa isim
  - description: 1 cümle
  - source_files: hangi test dosyalarından çıkarıldı
  - confidence: high/medium/low
ÇIKTI: Listeyi döndür.
```

### TASK-1B: Codebase-Based Feature Extraction
```
SCOPE: Yalnızca kaynak kod (src/, lib/, core/ — test dosyaları HARİÇ)
GÖREV: Public API'leri, export edilen fonksiyonları, modülleri analiz et.
        Kodun sunduğu yetenekleri feature olarak listele.
FORMAT:
  - feature_id: C-001, C-002, ...
  - feature_name: kısa isim
  - description: 1 cümle
  - source_files: hangi kaynak dosyalarından çıkarıldı
  - confidence: high/medium/low
ÇIKTI: Listeyi döndür.
```

### TASK-1C: Existing Documentation Extraction
```
SCOPE: README.md, docs/, *.md dosyaları, inline doc comments, mevcut catalog dosyaları
GÖREV: Dokümantasyonda bahsedilen feature'ları çıkar.
FORMAT:
  - feature_id: D-001, D-002, ...
  - feature_name: kısa isim
  - description: 1 cümle
  - source_files: hangi döküman dosyalarından çıkarıldı
  - confidence: high/medium/low
ÇIKTI: Listeyi döndür.
```

### PHASE-1 MERGE (Orchestrator görevi — sub-agent değil)
```
Üç TASK'ın çıktılarını al. Deduplicate et. Birleştir.
Her feature için:
  - final_id: F-001, F-002, ...
  - feature_name: kısa isim
  - description: 1-2 cümle
  - sources: hangi TASK'lardan geldi (T-xxx, C-xxx, D-xxx)
  - coverage: [test, code, docs] — hangilerinde mevcut

OUTPUT_FILE'a yaz. Sonraki phase'e geçmeden önce dosyanın yazıldığını doğrula.
```

---

## PHASE-2: FEATURE REVIEW SWARM (Paralel Swarm)

**AMAÇ**: Her feature için derinlemesine review. Bug, zayıf test, yanlış assertion, false assumption taraması.

**INPUT_FILE**: `./deep-review-artifacts-{timestamp}/phase1-feature-catalog.md`

**OUTPUT_FILE**: `./deep-review-artifacts-{timestamp}/phase2-raw-findings.md`

### TASK GENERATION RULE — KRİTİK
```
⛔ BATCHING YASAĞI BURADA GEÇERLİDİR. Yukarıdaki STRICT ISOLATION CONSTRAINT'i tekrar oku.

ADIM 1: phase1-feature-catalog.md dosyasını oku.
ADIM 2: Tüm feature ID'lerini listele ve say. Bu sayı = N.
ADIM 3: Aşağıdaki loop'u uygula:

    for each feature_id in [F-001, F-002, ..., F-N]:
        spawn ONE Task tool call with ONLY this single feature_id

ADIM 4: İşlem bittikten sonra doğrula:
    - Spawn edilen agent sayısı = N olmalı.
    - Eğer eşit değilse, eksik feature'lar için ek agent'lar spawn et.

SAYISAL DOĞRULAMA ÖRNEĞİ:
  Catalog'da 25 feature var → 25 ayrı Task tool çağrısı yapılacak.
  Eğer 20 çağrı yaptıysan → 5 feature eksik → 5 ek çağrı daha yap.

KARŞI-ÖRNEK (YAPMA):
  ✗ 25 feature'ı 4 gruba bölüp 4 Task tool çağrısı yapmak
  ✗ "Benzer feature'ları grupla" diyerek sayıyı azaltmak
  ✗ Tek bir Task tool prompt'una "F-001, F-002, F-003" yazmak
  ✗ "İlgili feature'ları birlikte incele" demek
```

### HER SUB-AGENT'İN GÖREVİ
```
INPUT: Tek bir feature (F-xxx) ve onun source dosyaları
SCOPE: O feature'a ait tüm test ve kaynak kod dosyaları

⚠️ Bu agent YALNIZCA kendisine verilen TEK feature'ı inceleyecek.
   Başka hiçbir feature'a değinmeyecek, karşılaştırma yapmayacak.

ANALIZ KATEGORİLERİ (her biri için en az 1 bulgu üretmeye çalış):

  1. POTENTIAL_BUG
     - Mantık hataları, edge case eksiklikleri, race condition
     - Hatalı hata yönetimi, kaynak sızıntısı

  2. WEAK_TEST
     - Yetersiz coverage, eksik edge case testi
     - Çok basit assertion'lar, mock'ların gerçekliği yansıtmaması

  3. WRONG_ASSERTION
     - Yanlış beklenen değer, ters mantık
     - Tipi yanlış kontrol, boundary hataları

  4. FALSE_ASSUMPTION
     - Kodun varsaydığı ama garanti olmayan şeyler
     - Ortam bağımlılıkları, sıralama varsayımları

HER BULGU İÇİN FORMAT:
  - finding_id: F-xxx-B-001 (feature_id + bulgu numarası)
  - category: POTENTIAL_BUG | WEAK_TEST | WRONG_ASSERTION | FALSE_ASSUMPTION
  - severity: critical / high / medium / low
  - file: ilgili dosya yolu
  - line_range: başlangıç-bitiş satır numaraları
  - description: ne bulundu (3-5 cümle)
  - evidence: ilgili kod snippet'i
  - suggested_fix: önerilen düzeltme (1-2 cümle)

Hiçbir bulgu yoksa: "NO_FINDINGS" döndür ve nedenini 1 cümleyle açıkla.
Finding dosyasını persist et daha sonra manuel olarak kullanıcı okumak isteyebilir.
```

### PHASE-2 COLLECT (Orchestrator görevi)
```
Tüm sub-agent çıktılarını topla.
Her bulguyu finding_id ile indexle.
OUTPUT_FILE'a yaz.
Toplam bulgu sayısını, kategori dağılımını ve severity dağılımını başa özet olarak ekle.

DOĞRULAMA: Çıktıda temsil edilen benzersiz feature_id sayısını say.
Phase-1'deki feature sayısı (N) ile karşılaştır.
Eksik feature varsa → eksik feature'lar için ek agent spawn et, sonra tekrar topla.
Not: eğer dosya çok büyük olursa, bulguları parça parça farklı dosyalara yaz her dosyada 30 adet bulgu olacak şekilde, sonra bu dosyaları birleştirerek phase3'e input olarak verebilirsin.
```

---

## PHASE-3: FINDING CRITIQUE SWARM (Paralel Swarm)

**AMAÇ**: Phase-2'deki her bulguyu bağımsız bir agent ile kritize etmek. Bulgu gerçekten geçerli mi?

**INPUT_FILE**: `./deep-review-artifacts-{timestamp}/phase2-raw-findings.md`

**OUTPUT_FILE**: `./deep-review-artifacts-{timestamp}/phase3-critiqued-findings.md`

### TASK GENERATION RULE — KRİTİK
```
⛔ BATCHING YASAĞI BURADA GEÇERLİDİR. Yukarıdaki STRICT ISOLATION CONSTRAINT'i tekrar oku.

ADIM 1: phase2-raw-findings.md dosyasını oku. (daha fazla ise her parçayı oku)
ADIM 2: Tüm finding ID'lerini listele ve say. Bu sayı = M.
ADIM 3: Aşağıdaki loop'u uygula:

    for each finding_id in [F-001-B-001, F-001-B-002, ..., tüm finding'ler]:
        spawn ONE Task tool call with ONLY this single finding_id

ADIM 4: İşlem bittikten sonra doğrula: spawn edilen agent sayısı = M.
         Eksik varsa ek agent spawn et.

Bir sub-agent'e 2 veya daha fazla finding vermek YASAKTIR.
```

### HER SUB-AGENT'İN GÖREVİ
```
INPUT:
  - Orijinal bulgu (Phase-2'den finding_id, description, evidence, suggested_fix)
  - İlgili kaynak kod dosyası (doğrudan dosyayı oku)

⚠️ Bu agent YALNIZCA kendisine verilen TEK finding'i kritize edecek.

GÖREV:
  1. Orijinal bulguyu oku ve anla.
  2. İlgili kodu TEKRAR bağımsız olarak incele.
  3. Şu soruları yanıtla:
     a. Bulgu gerçekten var mı? (doğrulama)
     b. Severity doğru mu? (yeniden değerlendir)
     c. Suggested fix mantıklı mı?
     d. Kaçırılan bir şey var mı? (ek bağlam)

ÇIKTI FORMAT:
  - finding_id: (aynı ID)
  - original_summary: orijinal bulgunun 1 cümlelik özeti
  - critique_verdict: CONFIRMED | DISPUTED | REJECTED
  - critique_reasoning: neden bu karara varıldı (3-5 cümle)
  - adjusted_severity: critical / high / medium / low / none
  - additional_notes: varsa ek gözlemler

ARBİTRAJ SONUCU EĞER BARİZ BİR ŞEKİLDE GEREKSİZ BİR BULGU İSE GÖRMEZDEN GEL HİÇBİR AKSİYON ALMADAN "REJECTED" VER VE EKLEME NOTU OLARAK "BARİZ BİR ŞEKİLDE GEREKSİZ BULGU" YAZ. FINAL REPORT'ta bu bulguların gereksiz yere yer kaplamaması için INVALID FINDING olarak işaretlenip temizlenmesi sağlanacak.
Finding Critique dosyasını persist et daha sonra manuel olarak kullanıcı okumak isteyebilir.
```

### PHASE-3 COLLECT (Orchestrator görevi)
```
Her finding için iki veri noktasını yan yana koy:
  - ORIGINAL: Phase-2 agent'inin notları
  - CRITIQUE: Phase-3 agent'inin notları

Bu ara-rapor-1 formatında OUTPUT_FILE'a yaz.

DOĞRULAMA: Çıktıdaki finding sayısı = Phase-2'deki finding sayısı (M) olmalı.
Eksik varsa → eksik finding'ler için ek agent spawn et.
```

---

## PHASE-4: ARBITRATION SWARM (Paralel Swarm)

**AMAÇ**: Orijinal bulgu ve critique'i birlikte okuyarak nihai karar vermek. Bağımsız 3. göz.

**INPUT_FILE**: `./deep-review-artifacts-{timestamp}/phase3-critiqued-findings.md`

**OUTPUT_FILE**: `./deep-review-artifacts-{timestamp}/phase4-arbitrated-findings.md`

### TASK GENERATION RULE — KRİTİK
```
⛔ BATCHING YASAĞI BURADA GEÇERLİDİR. Yukarıdaki STRICT ISOLATION CONSTRAINT'i tekrar oku.

ADIM 1: phase3-critiqued-findings.md dosyasını oku.
ADIM 2: Tüm finding ID'lerini listele ve say. Bu sayı = M.
ADIM 3: Aşağıdaki loop'u uygula:

    for each finding_id in [tüm finding'ler]:
        spawn ONE Task tool call with ONLY this single finding_id

ADIM 4: İşlem bittikten sonra doğrula: spawn edilen agent sayısı = M.
         Eksik varsa ek agent spawn et.

Bir sub-agent'e 2 veya daha fazla finding vermek YASAKTIR.
```

### HER SUB-AGENT'İN GÖREVİ
```
INPUT:
  - Orijinal bulgu (Phase-2 agent'inden)
  - Critique (Phase-3 agent'inden)
  - İlgili kaynak kod dosyası (doğrudan dosyayı tekrar oku)

⚠️ Bu agent YALNIZCA kendisine verilen TEK finding'i değerlendirecek.

GÖREV:
  1. Hem orijinal bulguyu hem critique'i oku.
  2. İlgili kodu TEKRAR bağımsız olarak incele.
  3. Kendi bağımsız araştırmanı yap.
  4. Hangisinin haklı olduğuna karar ver.
  5. Geçersizlik çok bariz (agentlerden birisi bariz bir şekilde yanılmış) durumlarında hiçbirşey vermeden INVALID FINDING çıktısı üret (FINAL REPORT belgesi gereksiz yere büyümemeli.)
  6. Securtiy ile ilgili konularda Madde 5 Geçersiz. Güvenlik bulguları her durumda raporlanmalı, geçersiz ilan edilemez.

ÇIKTI FORMAT: (aşağıdaki format veya sadece INVALID FINDING çıktısı)
  - finding_id: (aynı ID)
  - original_verdict: orijinal agent ne dedi (1 cümle)
  - critique_verdict: critique agent ne dedi (1 cümle)
  - arbitration_decision: VALID | PARTIALLY_VALID | INVALID
  - confidence_score: 1-10 (7+ = geçerli, 4-6 = tartışmalı, 1-3 = geçersiz)
  - final_severity: critical / high / medium / low / none
  - reasoning: neden bu karara varıldı (3-5 cümle)
  - actionable: true/false — bu bulgu üzerine aksiyon alınmalı mı?

Dosyayı persist et daha sonra manuel olarak kullanıcı okumak isteyebilir.
```

### PHASE-4 COLLECT (Orchestrator görevi)
```
Tüm arbitration sonuçlarını topla.
OUTPUT_FILE'a yaz.
Başa özet ekle:
  - Toplam bulgu sayısı
  - VALID / PARTIALLY_VALID / INVALID dağılımı
  - Ortalama confidence score
  - Actionable bulgu sayısı

DOĞRULAMA: Çıktıdaki finding sayısı = Phase-3'teki finding sayısı (M) olmalı.
Eksik varsa → eksik finding'ler için ek agent spawn et.
SAYISAL DOĞRULAMAYI TAMAMLADIKTAN SONRA INVALID FINDINGLERI TEMİZLE (filter out) — FINAL REPORT'ta gereksiz yer kaplamasın.
```

---

## PHASE-5: FINAL REPORT GENERATION

**AMAÇ**: Tüm phase'lerin çıktılarını birleştiren kapsamlı final rapor.

**INPUT_FILES**:
  - `./deep-review-artifacts-{timestamp}/phase1-feature-catalog.md`
  - `./deep-review-artifacts-{timestamp}/phase2-raw-findings.md`
  - `./deep-review-artifacts-{timestamp}/phase3-critiqued-findings.md`
  - `./deep-review-artifacts-{timestamp}/phase4-arbitrated-findings.md`

**OUTPUT_FILE**: `./deep-review-artifacts-{timestamp}/FINAL-REPORT.md`

### REPORT STRUCTURE
```markdown
# Deep Review — Final Rapor
**Tarih**: [otomatik]
**Toplam Feature Sayısı**: [Phase-1'den]
**Toplam Bulgu Sayısı**: [Phase-2'den]
**Geçerli Bulgu Sayısı**: [Phase-4'ten, score >= 7]

---

## 1. Executive Summary
- En kritik 5 bulgu (confidence_score'a göre sıralı)
- Genel sağlık değerlendirmesi (1 paragraf)

## 2. Feature Catalog Özeti
- Feature listesi (tablo: ID | İsim | Coverage [test/code/docs])

## 3. Kritik Bulgular (score >= 7)
Her bulgu için:
  ### [finding_id] — [category] — Severity: [final_severity]
  **Confidence Score**: [X]/10
  **Dosya**: [file:line_range]

  **Orijinal Bulgu (Phase-2 Agent)**:
  [description + evidence]

  **Critique (Phase-3 Agent)**:
  [critique_verdict + reasoning]

  **Arbitration (Phase-4 Agent)**:
  [decision + reasoning]

  **Önerilen Aksiyon**:
  [suggested_fix]

## 4. Tartışmalı Bulgular (score 4-6)
[Aynı format, kısaltılmış]

## 5. Reddedilen Bulgular (score 1-3)
[Sadece finding_id, category, red nedeni — 1 cümle]

## 6. İstatistikler
- Kategori dağılımı tablosu
- Severity dağılımı tablosu
- Feature başına bulgu yoğunluğu
- Agent uyum oranı (orijinal vs critique vs arbitration ne sıklıkla aynı fikirde)

## 7. Öneriler
- Hemen aksiyon alınması gereken konular (actionable=true, score>=7)
- Test iyileştirme önerileri
- Kod kalitesi önerileri
```

---

## EXECUTION CHECKLIST (Claude Code bunu takip edecek)

```
[ ] mkdir -p ./deep-review-artifacts-{timetamp}/
[ ] PHASE-1 başlat → TASK-1A, 1B, 1C paralel spawn
[ ] PHASE-1 çıktıları merge → phase1-feature-catalog.md yazıldı mı? ✓
[ ] Feature sayısını say → N = ?
[ ] PHASE-2 başlat → TAM OLARAK N adet Task tool çağrısı yap (her biri 1 feature)
[ ] DOĞRULA: spawn edilen agent sayısı = N mi? Eğer değilse eksikleri tamamla.
[ ] PHASE-2 çıktıları collect → phase2-raw-findings.md yazıldı mı? ✓
[ ] Finding sayısını say → M = ?
[ ] PHASE-3 başlat → TAM OLARAK M adet Task tool çağrısı yap (her biri 1 finding)
[ ] DOĞRULA: spawn edilen agent sayısı = M mi? Eğer değilse eksikleri tamamla.
[ ] PHASE-3 çıktıları collect → phase3-critiqued-findings.md yazıldı mı? ✓
[ ] PHASE-4 başlat → TAM OLARAK M adet Task tool çağrısı yap (her biri 1 finding)
[ ] DOĞRULA: spawn edilen agent sayısı = M mi? Eğer değilse eksikleri tamamla.
[ ] PHASE-4 çıktıları collect → phase4-arbitrated-findings.md yazıldı mı? ✓
[ ] PHASE-5 → FINAL-REPORT.md oluştur
[ ] FINAL DOĞRULAMA: Tüm dosyaları listele, her phase'in çıktısının var olduğunu doğrula.
```
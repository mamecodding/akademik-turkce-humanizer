# Teknik Devir Notu

## 1. Projenin amacı

Amaç, dışarıdan üretilmiş Türkçe akademik metni:

- yeni bilgi eklemeden,
- mevcut anlamı koruyarak,
- sayı, tarih, teknik terim ve atıfları değiştirmeden,
- daha doğal ve insan eliyle yazılmış akademik Türkçeye dönüştürmektir.

Sistem bilgi üreticisi değil, metin dönüştürücü/editör olarak tasarlanmıştır.

## 2. Sağlam ve kullanılabilir proje varlıkları

### Nihai insan corpus'u
- 1000 Türkçe akademik makale
- 5 alan × 200 metin
- body-only temizlenmiş corpus
- Klasör: `12_Nihai_1000_BodyOnly_V2`

### Human target havuzu
- 5000 orijinal human target
- kalite filtresi sonrası:
  - 3638 CLEAN
  - 1001 SUSPICIOUS
  - 361 EXCLUDE
- Klasör: `31_Human_Target_Quality_Filter`

### Paired dataset
- 2967 accepted pair
- 2673 train
- 294 validation
- Klasör: `33_Final_Paired_Production`

### İlk final QLoRA adapter
- Base: `Qwen/Qwen2.5-14B-Instruct`
- 1 epoch
- final eval loss: yaklaşık `0.6764`
- Klasör: `34_Final_Qwen2_5_14B_Humanizer_SFT/final_adapter`

## 3. İlk SFT sonucu

İlk QLoRA SFT teknik olarak tamamlandı. Ancak gerçek kullanım testinde model:

- sayıları korudu,
- atıfları korudu,
- metin uzunluğunu büyük ölçüde korudu,

fakat metni yeterince dönüştürmedi.

Gerçek blok testte surface similarity yaklaşık `0.996` seviyesinde kaldı.

Bu, modelin metni neredeyse kopyaladığını gösterdi.

## 4. Kopyalama sorunu için yapılan teşhisler

### Dataset bozuk mu?
Hayır.

Train pair surface ortalaması:
`0.4279`

Validation:
`0.4483`

Aşırı benzer pair:
`>0.98 = 0 adet`

Bu nedenle “dataset zaten kopya olduğu için model kopyalıyor” ihtimali elendi.

### Label / SFT hizalama hatası var mı?
Hayır.

Train:
`PREFIX EXACT = 2673 / 2673`

Validation:
`294 / 294`

Label retention:
- Train yaklaşık `0.9988`
- Validation `1.0`

Training label masking / alignment tarafında belirgin hata bulunmadı.

### Adapter gerçekten çalışıyor mu?
Evet.

Base model ve adapterlı model karşılaştırıldı. Adapter çıktıları base modelden farklılaştı.

Dolayısıyla “adapter yüklenmiyor veya inference sırasında aktif değil” ihtimali elendi.

## 5. Greedy vs Sampling testi

Greedy decoding:
`Input similarity mean = 0.8864`

Sampling:
`Input similarity mean = 0.5221`

Target benzerliği:
- Greedy: `0.2730`
- Sampling: `0.2125`

Sonuç: Sampling kopyalamayı azaltıyor fakat modeli doğru human target yönüne götürmüyor. Sorun yalnızca decoding ayarlarından kaynaklanmıyor.

## 6. Preference pilot veri üretimi

3638 CLEAN human target içinden yaklaşık dengeli 100 örnek seçildi.

Qwen2.5-14B kullanılarak human targetlardan AI-benzeri akademik input üretildi.

Dosya:
`37_Preference_Pilot_100/preference_pilot_100.jsonl`

## 7. Preference pilot QA

100 örnek üzerinde:
- PASS: `50`
- REVIEW: `50`
- Number preservation: `66 / 100`
- Citation preservation: `85 / 100`
- Surface similarity mean: `0.2938`
- Length ratio mean: `0.9567`

Temel sorunlar:
- 34 number mismatch
- 15 citation mismatch

Yalnızca 50 PASS örnek preference pilotunda kullanıldı.

## 8. Preference training pilot

50 PASS örnek:
- 40 train
- 10 validation

Reference-free pairwise preference training uygulandı.

Eğitimden önce:
- Accuracy: `0.0`
- Mean margin: `-0.8674`

Eğitimden sonra:
- Accuracy: `1.0`
- Mean margin: `+1.0206`

Sonuç: Preference eğitimi model davranışını güçlü biçimde değiştirdi.

Adapter:
`39_Preference_Pilot_Adapter_V1/final_adapter`

Not: Bu deney formal DPO değildir; reference-free pairwise preference pilotudur.

## 9. Preference adapter gerçek test sonucu

Gerçek metin 3 blok halinde test edildi.

Input similarity:
- Eski adapter: `0.7502`
- Yeni preference adapter: `0.0234`

Number preservation:
- Eski: `1 / 3`
- Yeni: `0 / 3`

Citation preservation:
- Eski: `1 / 3`
- Yeni: `0 / 3`

Sonuç: Preference yaklaşımı kopyalamayı kırdı fakat metni aşırı serbest yeniden yazmaya başladı.

## 10. Anchor V2 + Preference Adapter testi

Preference adapter üzerinde sayı ve atıfları korumak için Anchor V2 placeholder yöntemi test edildi.

Toplam `15 anchor` kullanıldı.

Sonuç:
`Anchor preservation = 0 / 15`

Number preservation:
`0 / 3`

Citation preservation:
`0 / 3`

Mevcut preference adapter, eğitim sırasında Anchor V2 formatını görmediği için inference sırasında placeholderları korumadı.

## 11. Şu ana kadar teknik olarak kanıtlananlar

1. 1000 makalelik insan corpus'u kullanılabilir.
2. 3638 CLEAN human target kullanılabilir.
3. 2967 paired dataset teknik olarak sağlamdır.
4. SFT label hizalaması doğrudur.
5. İlk QLoRA adapter gerçekten aktif çalışmaktadır.
6. İlk SFT modeli humanization yerine kopyalamaya eğilimlidir.
7. Greedy decoding tek başına sorun değildir.
8. Sampling kopyalamayı azaltır ancak doğru dönüşüm üretmez.
9. Preference training model davranışını güçlü şekilde değiştirebilir.
10. Mevcut preference pilotu kopyalamayı kırmıştır.
11. Ancak mevcut preference pilotu bilgi/sayı/atıf korumasını kaybetmiştir.
12. Anchor V2, preference adapter üzerinde inference aşamasında çalışmamıştır.

## 12. Aynı haliyle tekrar edilmemesi gereken yollar

- yalnız continued pretraining,
- sentetik V3 SFT,
- Style Bank / rule-based rewrite MVP,
- mevcut paired dataset üzerinde aynı 1 epoch klasik SFT,
- yalnız greedy → sampling değişikliği,
- mevcut preference adapter + sonradan Anchor V2 eklenmesi.

## 13. Mevcut proje durumu

Proje:
- veri tarafında güçlü,
- eğitim altyapısı çalışan,
- ancak final ürün davranışı henüz çözülmemiş

durumdadır.

İki uç davranış vardır:

**İlk SFT adapter**
→ içeriği koruyor ama metni yeterince değiştirmiyor.

**Preference adapter**
→ metni güçlü biçimde değiştiriyor ama içerik korumasını kaybediyor.

Final ürün için gerekli birleşim:
**Dönüşüm + Koruma**

## 14. Kritik artefaktlar

- `12_Nihai_1000_BodyOnly_V2`
- `31_Human_Target_Quality_Filter`
- `33_Final_Paired_Production`
- `34_Final_Qwen2_5_14B_Humanizer_SFT`
- `37_Preference_Pilot_100`
- `38_Preference_Pilot_QA`
- `39_Preference_Pilot_Adapter_V1`
- `40_Old_vs_Preference_Real_Test`
- `41_Preference_AnchorV2_Real_Test`

## Son durum cümlesi

Projenin veri mühendisliği ve temel model eğitim altyapısı tamamlanmış; klasik SFT'nin kopyalama problemi ve preference training'in koruma problemi deneysel olarak gösterilmiş; ancak doğal humanization ile bilgi/sayı/atıf korumasını aynı model davranışında birleştiren final yöntem henüz elde edilmemiştir.

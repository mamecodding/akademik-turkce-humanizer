# Deney Sonuçları Özeti

## Corpus ve veri
- Final corpus: 1000 body-only Türkçe akademik makale
- Alanlar: Sosyal Bilimler, Fen Bilimleri, Mühendislik/Teknoloji, Sağlık Bilimleri, Eğitim/Beşeri
- CLEAN human target: 3638
- Accepted paired dataset: 2967
- Train/validation: 2673 / 294

## Paired dataset üretimi
- Kabul oranı: %81.56
- Semantic mean: 0.9862
- Semantic min: 0.9307
- Length ratio mean: 0.949
- Surface mean: 0.4312

Başlıca reject nedenleri:
- number_ok: 372
- placeholder_ok: 330
- length: 307
- citation: 134
- sentence: 58
- semantic: 37
- useful_change: 4

## İlk QLoRA SFT
Model:
`Qwen/Qwen2.5-14B-Instruct`

Yapı:
- 4-bit NF4
- LoRA r=16
- alpha=32
- dropout=0.05
- target modules: q/k/v/o/gate/up/down
- effective batch: 8
- learning rate: 1e-4
- epoch: 1
- max_length: 1536

Final:
- optimizer steps: 335
- süre: ~53.78 dakika A100
- final train loss: ~0.7129
- final eval loss: ~0.67644

## Gerçek test
Doğru 1453 kelimelik testte:
- çıktı: 1452 kelime
- length ratio: 0.9993
- number preservation: PASS
- citation preservation: PASS
- surface similarity: 0.996

Yorum: model muhafazakâr kopyalayıcı.

## Adapter etkisi testi
Adapter aktif olduğu doğrulandı; sorun yükleme hatası değil.

## Decoding testi
6 validation örneği:

Greedy:
- input similarity mean: 0.8864
- target similarity mean: 0.2730

Sampling:
- input similarity mean: 0.5221
- target similarity mean: 0.2125

Her iki modda da sayı/atıf koruması 6/6.

Yorum: Sampling kopyalamayı kırıyor fakat hedef human tarza doğru taşımıyor.

## Preference pilot
100 örnek üretildi.

QA:
- PASS: 50
- REVIEW: 50
- number preservation: 66/100
- citation preservation: 85/100
- surface mean: 0.2938
- length mean: 0.9567

## Reference-free pairwise preference pilot
50 PASS:
- 40 train
- 10 validation
- epoch: 2
- LR: 5e-5
- beta: 4.0
- SFT weight: 0.20

Validation before:
- accuracy: 0.0
- mean margin: -0.8674
- median margin: -0.8141

Validation after:
- accuracy: 1.0
- mean margin: 1.0206
- median margin: 1.0218

Yorum: scoring objective üzerinde davranış güçlü biçimde değişti; bu tek başına free-generation kalitesinin kanıtı değildir.

## Old vs Preference real test
3 blok, 1408 kelime:

Old:
- output: 972 kelime
- input similarity mean: 0.7502
- number preservation: 1/3
- citation preservation: 1/3

Preference:
- output: 1318 kelime
- input similarity mean: 0.0234
- number preservation: 0/3
- citation preservation: 0/3

Yorum: kopyalama kırıldı fakat içerik koruma çöktü.

## Preference + Anchor V2
3 blok, 15 anchor:
- output total: 1400 kelime
- similarity mean: 0.017
- length ratio mean: 1.0041
- number preservation: 0/3
- citation preservation: 0/3
- anchor preservation: 0/15

Yorum: placeholderlar tamamen silindi. Mevcut adapter anchor-aware değil.

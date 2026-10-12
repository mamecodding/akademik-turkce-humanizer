# Akademik Türkçe Humanizer

Türkçe akademik metinleri, bilgi içeriğini değiştirmeden daha doğal ve insan eliyle yazılmış akademik Türkçeye dönüştürmeyi amaçlayan Ar-Ge projesi.

> **Durum:** Veri mühendisliği ve eğitim altyapısı tamamlandı; final humanizer davranışı henüz çözülmedi. İlk SFT modeli metni fazla kopyalıyor, preference pilotu ise kopyalamayı kırarken sayı/atıf korumasını kaybediyor.

## Ürün hedefi

Sistem bir bilgi üreticisi değil, **metin dönüştürücü/editör** olarak tasarlanmıştır.

Beklenen davranış:
- yeni bilgi eklememek,
- mevcut anlamı korumak,
- sayı, tarih, kişi/kurum adı, teknik terim ve atıfları korumak,
- metni özetlememek,
- akademik Türkçeyi daha doğal ve daha az mekanik hale getirmek.

## Mevcut varlıklar

Google Drive ana kökü:

```text
/content/gdrive_real/MyDrive/19_Corpus_Genisletme
```

Kritik klasörler:

```text
12_Nihai_1000_BodyOnly_V2
31_Human_Target_Quality_Filter
33_Final_Paired_Production
34_Final_Qwen2_5_14B_Humanizer_SFT
37_Preference_Pilot_100
38_Preference_Pilot_QA
39_Preference_Pilot_Adapter_V1
40_Old_vs_Preference_Real_Test
41_Preference_AnchorV2_Real_Test
```

## Kritik metrikler

| Bileşen | Sonuç |
|---|---:|
| Temiz insan corpus'u | 1000 makale |
| Human target | 5000 |
| CLEAN target | 3638 |
| Accepted paired dataset | 2967 |
| Train / validation | 2673 / 294 |
| İlk SFT eval loss | 0.676443 |
| İlk gerçek test surface similarity | 0.996 |
| Preference pilot validation accuracy | 0.0 → 1.0 |
| Preference pilot mean margin | -0.8674 → +1.0206 |
| Preference real test input similarity | 0.0234 |
| Preference real test sayı korunumu | 0 / 3 |
| Preference real test atıf korunumu | 0 / 3 |
| Anchor V2 + preference anchor preservation | 0 / 15 |

## Şu ana kadar kanıtlananlar

- Corpus ve human target havuzu kullanılabilir.
- Paired dataset gerçekten farklı input-target çiftleri içeriyor.
- SFT label/prefix hizalaması doğru.
- LoRA adapter inference sırasında aktif.
- Greedy decoding tek başına kopyalama probleminin nedeni değil.
- Sampling kopyalamayı azaltıyor fakat human target yönüne götürmüyor.
- Preference learning model davranışını güçlü biçimde değiştirebiliyor.
- Mevcut preference pilotu içerik/sayı/atıf korumasını kaybediyor.
- Preference adapter, eğitimde görmediği Anchor V2 placeholderlarını inference sırasında siliyor.

## Tekrar edilmemesi gereken deneyler

Aynı haliyle tekrar uygulanmaması gereken yollar:
- yalnız continued pretraining,
- sentetik V3 SFT,
- Style Bank / rule-based rewrite MVP,
- mevcut paired dataset ile aynı 1-epoch klasik SFT,
- yalnız greedy → sampling değişikliği,
- mevcut preference adapter üzerine sonradan Anchor V2 eklemek.

## Devir dokümanları

- [Teknik devir notu](docs/TECHNICAL_HANDOFF.md)
- [Deney özeti](docs/EXPERIMENT_RESULTS.md)
- [Codex çalışma talimatı](AGENTS.md)

## Model ve büyük dosyalar

GitHub deposuna büyük model ağırlıkları, checkpointler ve ham corpus eklenmemelidir. Bunlar Google Drive / ayrı model deposunda tutulmalıdır.

Özellikle:
- Qwen2.5-14B base model ağırlıkları,
- adapter safetensors/checkpoint dosyaları,
- tüm 1000 makalelik corpus,
- büyük JSONL veri dosyaları

repo dışında tutulmalıdır.

## Proje devri

Bir sonraki teknik kişi önce `docs/TECHNICAL_HANDOFF.md` ve `AGENTS.md` dosyalarını okumalıdır. Yeni GPU eğitimi başlatmadan önce mevcut deney sonuçları incelenmeli ve daha önce elenmiş yollar tekrar edilmemelidir.

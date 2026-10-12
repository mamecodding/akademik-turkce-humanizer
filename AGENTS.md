# AGENTS.md

## Görev bağlamı

Bu repo, Türkçe Akademik Humanizer Ar-Ge projesinin teknik devridir.

Ana hedef:
Dışarıdan üretilmiş Türkçe akademik metni, yeni bilgi eklemeden ve mevcut bilgi içeriğini değiştirmeden daha doğal akademik Türkçeye dönüştürmek.

## Önce okunacak dosyalar

1. `docs/TECHNICAL_HANDOFF.md`
2. `docs/EXPERIMENT_RESULTS.md`
3. `README.md`

## Kritik çalışma ilkeleri

- Büyük GPU eğitimi başlatmadan önce mevcut deneyleri incele.
- Daha önce elenmiş yaklaşımı aynı haliyle tekrar etme.
- Deneme-yanılma maliyetini düşük tut.
- Önce ucuz, küçük, ölçülebilir pilotlarla hipotez doğrula.
- Kullanıcı onayı olmadan uzun A100 eğitimi başlatma.
- Sayı, tarih, kişi/kurum adı, teknik terim ve atıf koruması ürünün zorunlu koşuludur.
- Yeni bilgi, kaynak, sayı veya örnek eklemek yasaktır.
- AI detector skorunu ana optimizasyon hedefi yapma.
- Final kalite ölçütü yalnızca surface similarity değildir; dönüşüm ve içerik koruma birlikte değerlendirilmelidir.

## Bilinen başarısızlık kipleri

### 1. Muhafazakâr kopyalama
İlk SFT adapter gerçek metni neredeyse aynen döndürür.

### 2. Kontrolsüz yeniden yazım
Preference pilotu kopyalamayı kırar fakat sayı/atıf ve içerik korumasını kaybeder.

### 3. Decoding çözümü değil
Sampling edit miktarını artırır fakat human target yönüne güvenilir biçimde taşımaz.

### 4. Sonradan anchor eklemek çözmedi
Mevcut preference adapter QQQ placeholderlarını 0/15 oranında korudu.

## Veri ve model konumları

Ana Google Drive kökü:

```text
/content/gdrive_real/MyDrive/19_Corpus_Genisletme
```

Kritik klasörler README ve teknik devir notunda listelenmiştir.

Büyük model/checkpoint dosyalarını bu repoya commit etme.

## Çalışma biçimi

Yeni bir yaklaşım önerirken önce şu formatta raporla:

1. Hipotez
2. Bu hipotezin hangi mevcut bulguyla ilişkili olduğu
3. En ucuz doğrulama deneyi
4. Başarı ölçütü
5. Başarısızlık durumunda ne eleneceği
6. Tahmini GPU maliyeti

Kullanıcı onayı olmadan tam ölçek eğitim başlatma.

## Teknik doğruluk notu

`39_Preference_Pilot_Adapter_V1` deneyi formal DPO değildir. Reference-free pairwise preference loss + chosen-response SFT bileşeni kullanılmıştır. Teknik dokümantasyonda DPO olarak etiketlenmemelidir.

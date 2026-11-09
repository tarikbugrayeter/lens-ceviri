# Lens Çeviri — telefon prototipi

AR gözlükte yapılacak otomatik yazı çevirisinin telefonda çalışan provası.
Kamerayı bir yazıya tutup telefonu sabit tuttuğunda, bakış alanındaki yazı
okunur, çevrilir ve gözlük ekranındaki gibi yeşil alt yazı olarak gösterilir.
**Ses** düğmesi açıkken duyulan İngilizce konuşma da yazıya dökülüp Türkçe altyazı olarak gösterilir.

Tek dosya: `index.html`. Derleme, paket kurulumu yok.

Canlı adres: https://tarikbugrayeter.github.io/lens-ceviri/

## Sesli altyazı

```
mikrofon → konuşma tanıma (Web Speech API) → bölücü → bağlamlı çeviri (akış) → altyazı
```

- Kamera açılınca dinleme de kendiliğinden başlar (Ayarlar'dan kapatılabilir). Altyazı yalnızca konuşma duyulunca belirir, 8 sn sessizlikte kaybolur. **Ses** düğmesi elle açıp kapatır; girişteki **Sadece sesli altyazı** kamerasız çalışır.
- **Cümle sonu beklenmez.** Tarayıcının tanıyıcısı "cümle bitti" demeyi çok geç yapar (özellikle iPhone'da). Uygulama kendi böler: ~0,6 sn duraklamada o ana kadarki parça, konuşma uzarsa ~9 kelimede (mümkünse virgül ya da and/but/so gibi bir bağlaçtan) ilk kısım hemen çevrilir. Henüz bölünmemiş kısmın canlı çevirisi de ekranda akar. Kod: `createSegmenter()`.
- **Çeviri motoru:** Konuşma için **Claude** önerilir. Önceki 3 cümleyi bağlam olarak okur; noktalamasız, yarım ya da yanlış duyulmuş parçaları altyazı dilinde toparlar. Yanıt akış halinde gelir, ilk kelimeler hemen görünür. Anahtar: console.anthropic.com. **Google** hızlıdır ama her parçayı bağlamsız çevirir. **MyMemory** anahtarsızdır ama konuşmada kalitesi düşüktür.
- Dil Ayarlar → Sesli altyazı'dan değişir (EN→TR veya TR→EN).
- Android Chrome'da konuşma tanıma Google sunucularında yapılır, internet gerekir.
- **Netflix gibi videolar:** Video TV'de ya da bilgisayarda oynarken telefonu yakına koy. Video aynı telefonda oynuyorsa telefonun yankı engelleme özelliği hoparlör sesini bastırabilir.
- Gözlükte bu kısım gözlüğün mikrofon dizisi + konuşma tanıma servisiyle değişir; bölücü, çeviri ve altyazı mantığı aynı kalır.

## Hız

Telefon durduktan sonra yaklaşık 0,35 sn içinde okuma başlar. Yazı okunur okunmaz orijinal metin soluk görünür, her paragrafın çevirisi geldikçe yerine geçer. Konuşmada cümle bitmesi beklenmez; konuşma sürerken ara çeviri gösterilir, cümle bitince kesin çeviri yerleşir.

Gecikmenin büyük kısmı cihazdaki OCR'dan gelir. Hızlandırmak için:
- Çeviri yönünü sabitle (EN→TR gibi): OCR tek dil modeliyle çalışır, otomatik moddan yaklaşık iki kat hızlıdır.
- Google Vision + Google çeviri: telefonda genelde en hızlı ve en isabetli seçenek.
- Ayarlar → **Gecikme ölçümlerini göster** ile her adımın süresini görebilirsin.

## Bilinen sınırlar

- **MyMemory kotası:** Anonim kullanımda günlük karakter kotası düşük. Ayarlara e-posta yazınca kota yükselir. Ciddi testler için Google motoru daha sağlam.
- **Google anahtarı tarayıcıda:** Prototip için kabul edilebilir; gerçek üründe çeviri isteğini kendi sunucun üzerinden yap.
- **OCR kalitesi:** Tesseract kamera görüntüsünde orta düzeydedir, özellikle ekran ve yansımada. Ayarlardan **Google Vision** seçilince isabet belirgin artar (Cloud Vision API açık bir Google Cloud anahtarı gerekir). Android'de ML Kit hem hızlı hem isabetlidir (aşağıda).
- **Yazının üstünde modu:** Kutular OCR anındaki konumda çizilir; telefon hareket edince silinir, durunca geri gelir. Gerçek takip (tracking) bir sonraki adım.

## Gözlüğe (Rokid) taşırken

Ortadaki mantık (sabitlik, önbellekler, dil tespiti, yönlendirme) aynen kalır. Değişen kısımlar:

| Prototipte | Gözlükte |
|---|---|
| `getUserMedia` ile telefon kamerası | Rokid SDK üzerinden gözlük kamerası kareleri |
| Kare farkıyla sabitlik | Gözlüğün IMU'su (baş hareketi) + kare farkı |
| Tesseract.js | Google ML Kit Text Recognition (Latin alfabesi, Türkçe harfler dahil) |
| MyMemory / Google | ML Kit On-device Translation (TR↔EN, internetsiz) ya da bulut |
| HTML HUD paneli | Gözlük ekranına tek renk metin; ekran küçük olduğu için 2–3 satır sınırı |

Ağır işler (OCR, çeviri) büyük ihtimalle eşleşen Android telefonda çalışır, gözlük kare gönderip metni gösterir.

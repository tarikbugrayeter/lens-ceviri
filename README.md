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
- **İki altyazı modu** (Ayarlar → Altyazı ne zaman çevrilsin):
  - **Cümle bitince** (varsayılan): Konuşmacı ~0,85 sn durunca ya da tanıyıcı cümle sonu noktalaması koyunca bütün cümle bir kerede çevrilir. Konuşma sürerken yalnızca duyulan İngilizce soluk görünür. Çok uzun konuşmada ~22 kelimede bölünür.
  - **Anlık**: ~0,6 sn duraklamada ya da ~9 kelimede bölünür, bekleyen kısmın canlı çevirisi akar. Daha hızlı ama daha dağınık.
  - Kod: `createSegmenter()`.
- **Çeviri motoru:** Konuşma için **Claude** önerilir. Önceki 3 cümleyi bağlam olarak okur; noktalamasız, yarım ya da yanlış duyulmuş parçaları altyazı dilinde toparlar. Yanıt akış halinde gelir, ilk kelimeler hemen görünür. Anahtar: console.anthropic.com. **Google** hızlıdır ama her parçayı bağlamsız çevirir. **MyMemory** anahtarsızdır ama konuşmada kalitesi düşüktür.
- Dil Ayarlar → Sesli altyazı'dan değişir (EN→TR veya TR→EN).
- Android Chrome'da konuşma tanıma Google sunucularında yapılır, internet gerekir.
- **Netflix gibi videolar:** Video TV'de ya da bilgisayarda oynarken telefonu yakına koy. Video aynı telefonda oynuyorsa telefonun yankı engelleme özelliği hoparlör sesini bastırabilir.
- Gözlükte bu kısım gözlüğün mikrofon dizisi + konuşma tanıma servisiyle değişir; bölücü, çeviri ve altyazı mantığı aynı kalır.

## Hız ve hareket algılama

- **Jiroskop önce:** Telefonun dönüş hızı (°/sn) ölçülür. El titremesi (~5–15°/sn) "sabit", bakış çevirme (>28°/sn) "hareket" sayılır. Ekran titremesi ve otomatik pozlama bunu etkilemez. iPhone ilk açılışta hareket sensörü izni sorar. Gözlükte aynı rolü gözlüğün IMU'su görür.
- **Jiroskop yoksa görüntü farkı:** Parlaklık ortalaması çıkarılır (pozlama telafisi), eşik kameranın kendi gürültüsüne göre uyarlanır.
- Telefon **0,25 sn** sabit kalınca okuma başlar; bakış çerçevesinin altındaki çubuk dolarak bunu gösterir.
- İlk okuma bulanık kareye denk gelip yazı bulamazsa aynı sahnede **1,8 sn'de bir** yeniden denenir (önceden telefon oynatılana kadar beklenirdi).
- Okunan metin hemen soluk görünür, çeviriler geldikçe yerine geçer.
- Çeviri yönü sabitken OCR tek dil modeliyle çalışır (otomatikten ~2 kat hızlı). Google Vision + Google/Claude çeviri telefonda genelde en hızlı seçenektir.
- Ayarlar → **Gecikme ölçümlerini göster**: OCR, çeviri, toplam süre ve jiroskop/gürültü değeri.

## AR gözlük modu (varsayılan)

Ekranda gerçek bir şeffaf AR gözlüğü taklit eder:
- **Çeviri yazının üstüne yapışır.** Her karede görüntünün ne kadar kaydığı ölçülür (satır/sütun profilleriyle kaba tahmin + 96×72'de ince blok eşleme, kare başına ~1,5 ms). Telefonu çevirdikçe etiketler yazıyla birlikte kayar. Kod: `frameShift()`, `trackStep()`, `drawAR()`.
- **Takip koparsa** (başka yere bakıldı), etiketler gizlenir; telefon durunca yazı yeniden okunur.
- **Yalnızca gözlüğün görüş alanında çizer:** ortadaki çerçeve, gözlük ekranının kapladığı alanı temsil eder; dışına taşan etiket kırpılır.
- **Bakış imleci:** ortadaki halka sabit tutunca dolar, okurken döner, çeviri gelince yeşil olur.
- **Arayüz kaybolur:** kontroller 4 sn sonra gizlenir, ekrana dokununca geri gelir. Altyazı ekranın altında süzülür.
- Ekranda okunabilsin diye etiketlerin altına yumuşak koyu zemin konur; gerçek şeffaf gözlük karartamaz, yalnızca ışık ekler.

Gözlükte takip işini gözlüğün kendi 6DoF/SLAM takibi çok daha iyi yapar; bu prototip yalnızca 2B kaymayı izler (yaklaşma/uzaklaşma ve dönme yok).

**Panel** görünümü (alt çubukta gözlük simgesi ile geçiş): çeviriler çerçevenin altındaki sabit panelde listelenir.

## Arayüz

- Üstteki **EN → TR · Motor** etiketi ayarları açar. Ayarlar dört gruptadır: kamerayla okuma, sesli altyazı, çeviri motoru, ekran.
- Alt çubuk: Ses · Şimdi oku · Görünüm (AR / Panel).
- iPhone'da Safari → Paylaş → **Ana Ekrana Ekle** ile tam ekran uygulama gibi açılır (`manifest.json`, ikonlar).

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

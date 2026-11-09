# Lens Çeviri — telefon prototipi

AR gözlükte yapılacak otomatik yazı çevirisinin telefonda çalışan provası.
Kamerayı bir yazıya tutup telefonu sabit tuttuğunda, bakış alanındaki yazı
okunur, çevrilir ve gözlük ekranındaki gibi yeşil alt yazı olarak gösterilir.
**Ses** düğmesi açıkken duyulan İngilizce konuşma da yazıya dökülüp Türkçe altyazı olarak gösterilir.

Tek dosya: `index.html`. Derleme, paket kurulumu yok.

Canlı adres: https://tarikbugrayeter.github.io/lens-ceviri/

## Sesli altyazı

```
mikrofon → konuşma tanıma (Web Speech API) → cümle bitince çeviri → altyazı
```

- Kamerayla birlikte **Ses** düğmesinden ya da girişteki **Sadece sesli altyazı** ile kamerasız çalışır.
- Konuşma devam ederken o ana kadar duyulan İngilizce soluk yazıyla görünür; cümle bitince Türkçesi yeşil altyazı olur. Son 3 cümle ekranda kalır.
- Dil Ayarlar → Sesli altyazı'dan değişir (EN→TR veya TR→EN).
- Android Chrome'da konuşma tanıma Google sunucularında yapılır, internet gerekir. Tarayıcı sessizlikte dinlemeyi kapatır; uygulama kendiliğinden yeniden başlatır.
- **Netflix gibi videolar:** Video TV'de ya da bilgisayarda oynarken telefonu yakına koy. Video aynı telefonda oynuyorsa, telefonun yankı engelleme özelliği hoparlör sesini bastırabilir ve tanıma zayıflar. Videonun kendi İngilizce altyazısı varsa, kamerayla onu okumak da bir seçenek.
- Gözlükte bu kısım gözlüğün mikrofon dizisi + konuşma tanıma servisiyle (cihazda ya da bulutta) değişir; çeviri ve altyazı mantığı aynı kalır.

## Çalıştırma

Tarayıcılar kameraya yalnızca **HTTPS** üzerinden (ya da bilgisayarda `localhost`'ta) izin verir.

**Telefonda (en kolay yol):**
1. https://app.netlify.com/drop adresine git.
2. `ar-ceviri` klasörünü sürükleyip bırak. Birkaç saniyede `https://…netlify.app` adresi verir.
3. Bu adresi telefonda Chrome'da (Android) ya da Safari'de (iPhone) aç, **Kamerayı aç**'a bas.

GitHub Pages de olur: dosyayı bir repoya koy, Settings → Pages'i aç.

**Bilgisayarda (webcam ile):**
```bash
cd ar-ceviri
python3 -m http.server 8000
# tarayıcıda: http://localhost:8000
```
Not: Telefonu aynı ağdan `http://192.168.x.x:8000` ile bağlarsan kamera açılmaz (HTTPS değil).

İlk açılışta OCR modeli (İngilizce + Türkçe, birkaç MB) indirilir; sonra tarayıcı önbelleğinden gelir.

## Nasıl çalışıyor

```
kamera → sabitlik → sahne önbelleği → OCR → dil tespiti → çeviri (+önbellek) → çizim
```

| Adım | Ne yapıyor | Dosyadaki yeri |
|---|---|---|
| Sabitlik | Her 150 ms'de bakış alanının 48×36 gri küçük resmi alınır, önceki kareyle karşılaştırılır. ~0,65 s fark küçük kalırsa "okuyor" sayılır. | `tick()` |
| Sahne önbelleği | Görüntü son çevrilen sahneye benziyorsa OCR tekrar çalışmaz, eski sonuç geri gelir. Başını çevirip geri dönünce çeviri anında görünür. | `lastSceneThumb` |
| OCR | Tesseract.js, cihaz üzerinde. Sadece ortadaki bakış alanı okunur (hız + "baktığın yer" hissi). | `captureGaze()`, `extractBlocks()` |
| Dil tespiti | Türkçe harfler, sık kelimeler ve ekler üzerinden kural tabanlı. | `detectLang()` |
| Çeviri | MyMemory (anahtarsız) ya da Google Cloud (anahtarlı). Aynı metin ikinci kez çevrilmez. | `translate()` |
| Çizim | **HUD**: Rokid gibi şeffaf gözlüklerdeki alt yazı. **Yazının üstünde**: Quest gibi kameralı gözlüklerdeki yerine yazma. | `render()` |

Ayarlanabilir eşikler dosyanın başındaki `CFG` nesnesinde.

## Bilinen sınırlar

- **MyMemory kotası:** Anonim kullanımda günlük karakter kotası düşük. Ayarlara e-posta yazınca kota yükselir. Ciddi testler için Google motoru daha sağlam.
- **Google anahtarı tarayıcıda:** Prototip için kabul edilebilir; gerçek üründe çeviri isteğini kendi sunucun üzerinden yap.
- **OCR hızı:** Tesseract telefonda 1–3 s sürebilir. Android'de ML Kit çok daha hızlıdır (aşağıda).
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

# Dokuzatr (Volcanic Auto Peaks & Valleys v6) - Filtre Çizme Mantığı Raporu

Bu rapor, "Volcanic Auto Peaks & Valleys v6.6" indikatörünün çalışma prensiplerini ve manuel kullanım detaylarını özetlemektedir.

---

## 1. Genel Bakış
v6.6 sürümü ile indikatör, kullanıcıya **tam manuel kontrol** imkanı sunarken, istenildiğinde aktif edilebilen bir **otomatik optimizasyon motoru** seçeneğini de barındırır. "Sistem ATR" gibi karmaşık yan ayarlar kaldırılmış, sistem tamamen Tepe ve Dip bazlı asimetrik bir yapıya oturtulmuştur.

---

## 2. Temel Matematiksel Bileşenler

### 2.1. KAMA Tabanlı ATR (Adaptive ATR)
İndikatör, klasik ATR yerine Kaufman'ın Uyarlanabilir Hareketli Ortalaması (KAMA) mantığını kullanır. Bu sayede piyasa gürültüsü filtrelenir ve çizgiler daha net "basamak" (step) yapıları oluşturur.

### 2.2. Manuel ve Otomatik ATR Yönetimi (v6.6)
*   **Manuel Öncelik:** Kullanıcı, Tepe ve Dip için bağımsız ATR periyotlarını menüden kendisi seçebilir.
*   **Opsiyonel Motor:** "Otomatik Optimizasyon Motoru" açıldığında, sistem manuel değerleri devre dışı bırakır ve 0.09 - 900.0 aralığında fiyatı en iyi takip eden değeri otomatik bulur.
*   **Asimetri:** Tepe ve Dip filtreleri birbirinden tamamen bağımsızdır; biri manuel diğeri otomatik çalışabilir (kodun yapısı buna müsaittir) veya her ikisi de farklı periyotlarla piyasayı izleyebilir.

---

## 3. Filtre Güncelleme Mantığı (upd_filt)
Filtrenin çizilme algoritması `upd_filt` metoduyla fiyatın ekstrem noktalarından itibaren belirlenen volatilite payını takip eder:
- `Filtre Boyutu = KAMA(TR, Periyot) * 9.0`
- Gamma katsayısı her zaman **9.0**'da sabittir. Fiyat bu kalkanı her ittiğinde filtre bir basamak ötelenir.

---

## 4. Dinamik Çapalama (Anchor Settling)
Sistem, filtrelerin nereden başlayacağını (çapa noktası) otomatik belirler. v6.6 ile bu süreç daha da hassaslaştırılmıştır:
- **Settle Threshold:** Fiyat filtreden uzaklaşırsa çapa güncellenir. Bu hesaplamada artık "Sistem ATR" değil, doğrudan filtrenin kendi aktif ATR birimi kullanılır.
- **Proximity Threshold:** Fiyat filtreye %0.5 yaklaştığında sistem yeni bir ekstrem nokta arayışına girer.

---

## Özet
Dokuzatr v6.6, karmaşıklıktan arındırılmış ancak esnekliği artırılmış bir sürümdür. Sabit 9.0 Gamma koruması altında, kullanıcıya manuel veya otomatik takip arasında seçim yapma özgürlüğü sunar ve her iki filtre hattını da kendi özgün volatilite dinamiklerine göre yönetir.

# STM32 Bağımsız Bekçi Köpeği (IWDG) ve Non-Blocking Debounce Uygulaması

Bu proje, STM32 mikrodenetleyicilerinde endüstriyel gömülü sistemlerin en kritik iki güvenlik ve stabilite mekanizmasını pratik bir senaryo ile göstermektedir: **Bağımsız Bekçi Köpeği (IWDG - Independent Watchdog)** ile sistem çökme koruması ve **Durum Makinesi (State Machine)** kullanılarak donanımsal zamanlayıcılarla (Non-Blocking) uygulanan **Yazılımsal Debounce (Sıçrama Önleme)**.

Sistemde, işlemciyi kilitleyen `HAL_Delay()` fonksiyonu (başlangıç animasyonu hariç) kullanılmamış; tüm zaman yönetimi `HAL_GetTick()` üzerinden asenkron olarak kurgulanmıştır.

## 🌟 Öne Çıkan Mühendislik Yaklaşımları

### 1. Durum Makinesi ile Yazılımsal Debounce (Sıçrama Önleme)
Mekanik butonların içindeki metal plakalar birbirine temas ederken mikrosaniyeler seviyesinde defalarca seker (bouncing). Bu elektriksel gürültüyü filtrelemek için sistemi kilitleyen `HAL_Delay()` kullanmak yerine, asenkron çalışan 3 aşamalı bir **Durum Makinesi (State Machine)** tasarlanmıştır:

* **1. Aşama (`butona_basidimi`):** İşlemci butondan gelen ilk elektriksel "1" (High) sinyalini gördüğü an hemen işlem yapmaz. O anki zaman damgasını (`bsaniye = anliksan`) kaydeder ve beklemeye geçer.
* **2. Aşama (`ne_kadar_basildi`):** Sistem tam 50 milisaniye (`bson = 50`) boyunca bekler. Bu süre, arkların (titremelerin) bitmesi için gereken filtredir. Süre dolduğunda butona tekrar bakılır:
  * Sinyal hala "1" ise: Gerçek bir insan basımı olarak kabul edilir ve işlem başlatılır (`dongu` fazına geçilir).
  * Sinyal "0" olmuşsa: Elektriksel bir parazit veya yanlışlıkla dokunma sayılarak reddedilir ve sistem başa döner.
* **Sonuç:** Bu mimari sayesinde buton sekmesinden kaynaklanan sahte tetiklemeler tamamen engellenmiş ve işlemci gücü boşa harcanmamıştır.

### 2. Bağımsız Bekçi Köpeği (IWDG) ile Çökme Koruması
Sistemlerin kilitlenmesi durumunda kendi kendini kurtarabilmesi için IWDG donanımı aktif edilmiştir. 
* Normal çalışma durumunda ana döngü sürekli olarak `HAL_IWDG_Refresh()` fonksiyonunu çağırarak Watchdog sayacını sıfırlar ("köpeği besler").
* İşlemci beklenmedik bir sonsuz döngüye (`while(1)`) girerse (bu projede geçerli bir buton basımı ile simüle edilmiştir), Watchdog beslenemez ve süre dolduğunda mikrodenetleyiciye donanımsal bir **Reset** atarak sistemi tekrar hayata döndürür.

### 3. Engellemesiz (Non-Blocking) Zaman Yönetimi
Ana çalışma döngüsünde hiçbir bekleme komutu yoktur. LED'in her 500ms'de bir yanıp sönmesi (Heartbeat) işlemi, işlemciyi durdurmayan `HAL_GetTick()` fonksiyonu ile aradaki fark (`anliksan - lson >= laveraj`) hesaplanarak gerçekleştirilir.

## ⚙️ Sistemin Çalışma Senaryosu (Crash & Recovery)

1. **Başlangıç Animasyonu:** Sistem ilk açıldığında veya resetlendiğinde, sistemin hayatta olduğunu bildirmek için bir LED 100ms aralıklarla çok hızlı bir şekilde 5 kez yanıp söner.
2. **Kalp Atışı (Heartbeat):** Animasyon bitince LED yarım saniyede bir düzenli olarak yanıp sönmeye başlar ve IWDG sürekli beslenir.
3. **Tuzağa Düşme (Crash Simulation):** Kullanıcı butona basıp 50ms'lik Debounce filtresini başarıyla geçtiğinde, kod bilerek yazılmış bir boş sonsuz döngüye (`case dongu: while(1){}`) hapsolur.
4. **Kurtarma (Recovery):** İşlemci sonsuz döngüye girdiği için Heartbeat durur ve IWDG beslenemez. Watchdog sayacı sıfıra ulaştığında donanıma reset atar. Kullanıcı, 1. adımdaki "5 hızlı blink" animasyonunu tekrar görerek sistemin başarıyla kurtarıldığını anlar.

## 📌 Pin Konfigürasyonu
* **Sistem/Heartbeat LED'i (Çıkış):** `PE8` (veya ilgili `led_Pin`)
* **Kullanıcı Butonu (Giriş):** `PA0` (veya ilgili `button_Pin`)
* **Watchdog Saati:** Dahili LSI Osilatör (40 kHz)

## 🚀 Kurulum ve Kullanım

1. Bu projeyi bilgisayarınıza klonlayın:
   ```bash
   git clone [https://github.com/KULLANICI_ADINIZ/STM32-IWDG-Debounce-StateMachine.git](https://github.com/KULLANICI_ADINIZ/STM32-IWDG-Debounce-StateMachine.git)

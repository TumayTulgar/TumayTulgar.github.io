---
layout: post
title: "Log 001: Kaosun Sesini Dinlemeden Önce Zamanı Akort Etmek"
date: 2026-01-03 11:30:00 +0300
categories: [Tramesonet, Logbook]
keywords: [esp32 zaman senkronizasyonu, ds3231 rtc kayması, arduino meteoroloji istasyonu yapımı, gps atomik saat, tramesonet, lora haberleşme]
description: "Tramesonet meteoroloji ağı için ESP32, GPS ve RTC modüllerinin zaman senkronizasyon testleri. Gerçek devre kurulumu ve atomik saat hassasiyet analizi."
image: /assets/TIME.png
---

Tramesonet istasyon ağı için son derece önemli bir konu olan senkronize çalışma prensibini test ediyorum.

Şiddetli bir konveksiyonel yağışın ya da bir cephe geçişinin Trakya üzerindeki hızlı hareketini kayıt altına alabilmem için tüm sensörlerin birbirleri ile eş zamanlı haberleşmesi kritik önem taşıyor.

Dolayısıyla "kaosun sesini" dinlemeden önce, kendi enstrümanımın akordunu yapmalıyım.

### Deney Kurulumu: Zamanın Peşinde

İstasyonların ne kadar senkron olabildiğini görebilmek için kontrollü bir deney ortamı kurdum. Yaklaşık 22°C'lerdeki oda sıcaklığında; ESP32, DS3231 RTC ve GPS (GY-NEO6MV2) modülleri ile hem kapalı ortam hem de dış ortam (daha soğuk) testlerini gerçekleştirdim.

### Neden Bu Testi Yapıyoruz?

Aslında mutlak zamanı takip etmek muazzam derecede zor bir iştir. Başlangıçta tüm istasyon saatlerimizi eşitleyebiliriz. Ama bir süre sonra bakarız ki hepsi birbirinden farklı kaymalar gerçekleştirmiş. Peki, bu kaymalar ne kadar sorun yaratacak ölçüde?

Elektronik komponentler zamanı, doğal rezonans titreşimi 1 Hz'e (yani 1 saniyeye) karşılık gelen bir kuvars kristali ile takip ederler. Ancak kuvars kristali sıcaklığa karşı hassastır. Bu yüzden yaklaşık 7 saatlik dilimde hem kapalı ortam (22°C civarı) hem de açık hava (17°C civarı) koşullarında, mutlak zaman olan atomik saat verisini referans alarak deneyimi tamamladım.

### Verilerin Toplanması

Deneyde akış şu şekilde ilerledi:
* **Referans:** GPS sensörüne saat bilgisi geliyor ve mikroişlemcide (ESP32) tutuluyor.
* **Karşılaştırma 1:** RTC modülü kendi saatini ESP32'ye bildiriyor.
* **Karşılaştırma 2:** ESP32 kendi elektronik milisaniye mertebesindeki zaman sayacını bir değişkende tutuyor.

### Analiz ve Sonuçlar

Aşağıdaki grafikte sıcaklık değişiminin (üst panel) osilatör kararlılığı üzerindeki etkisini (alt panel) net bir şekilde görüyoruz.

![Sıcaklığın Zaman Kayması Üzerine Etkisi](https://tumaytulgar.github.io/assets/Figure_1.png)

| Süre | ESP32 Kayması | RTC Kayması |
| :--- | :--- | :--- |
| **1 Saat** | -0.17 s (170 ms) | -0.012 s (12 ms) |
| **1 Gün** | -4.14 s | -0.30 s (300 ms) |
| **1 Hafta** | -29 s | -2.07 s |
| **1 Ay** | -2 dk | -9 s |
| **1 Yıl** | -25 dk | -1.8 dk |

### Çözüm: Uyan ve Düzelt

Bu zaman kayması standart projelerde sorun yaratmayabilir ancak benim bilmekten haz duyduğum bir şey var: "Şu an" A noktasındaki atmosferik veriler bu şekildeyken, daha uzaktaki B noktasında tam olarak nasıl?

Bu yüzden tüm istasyonlarda GPS sensörü bulundurup, günde bir kez uyandırarak RTC saatini kalibre etmesi için programlamamız gerekli.

---

## English Version

### Tramesonet: Tuning Time Before Listening to the Sound of Chaos

I am testing the principle of synchronized operation, which is a highly critical issue for the Tramesonet station network.

It is vital for all sensors to communicate simultaneously to record the rapid movement of severe convectional precipitation or a frontal passage over Thrace.

Therefore, before listening to the "sound of chaos," I must tune my own instrument.

### Experimental Setup: Chasing Time

I set up a controlled experimental environment to observe how synchronous the stations could remain. I conducted tests in both an indoor environment (at approximately 22°C room temperature) and a colder outdoor environment using ESP32, DS3231 RTC, and GPS (GY-NEO6MV2) modules.

### Why Conduct This Test?

Tracking absolute time is actually a tremendously difficult task. Initially, we can synchronize all our station clocks. However, after a while, we realize that they have all drifted differently from one another. So, to what extent do these drifts create a problem?

Electronic components track time using a quartz crystal corresponding to a natural resonance vibration of 1 Hz (1 second). However, quartz crystals are sensitive to temperature. For this reason, I completed the experiment over a period of approximately 7 hours in both closed (approx. 22°C) and open air (approx. 17°C) environments, using atomic time data as a reference.

**Note:** We access atomic time data thanks to the precise time information transmitted at the speed of light by orbiting satellites to our GPS sensor.

### Data Collection

The flow of the experiment was as follows:
1.  **Reference:** Time information arrives at the GPS sensor and is held in the microcontroller (ESP32).
2.  **Comparison 1:** The RTC module reports its own time to the ESP32.
3.  **Comparison 2:** The ESP32 keeps its own electronic millisecond-level time counter in a variable.

### Analysis and Results

In the graph below, we can clearly see the effect of temperature change (top panel) on oscillator stability (bottom panel). Note specifically how the deviation in the ESP32's internal time counter (purple line) changes when the temperature drops from 24°C levels to 16°C levels.

![Effect of Temperature on Time Drift](https://tumaytulgar.github.io/assets/Figure_1.png)

| Duration | ESP32 (Millis) Drift | RTC (DS3231) Drift |
| :--- | :--- | :--- |
| **1 Hour** | -0.17 seconds (170 ms) | -0.012 seconds (12 ms) |
| **1 Day** | -4.14 seconds | -0.30 seconds (300 ms) |
| **1 Week** | -29 seconds | -2.07 seconds |
| **1 Month** | -2 minutes | -9 seconds |
| **1 Year** | -25 minutes | -1.8 minutes |

### Solution: Wake Up and Correct

This time drift might not cause serious problems in standard projects. However, there is something I take pleasure in knowing: While atmospheric data at point A is like this "right now," how is it exactly at the more distant point B?

To be sure of this accuracy, I want to be very cautious about the time counter in the station installations.

Therefore, we need to equip all stations with a GPS sensor and program them to wake up once a day to calibrate the RTC.
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTc4NTk2MzYxOSwtOTA5MzMyMjA5LDE2ND
IxMDQwMTFdfQ==
-->
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
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTkwOTMzMjIwOSwxNjQyMTA0MDExXX0=
-->
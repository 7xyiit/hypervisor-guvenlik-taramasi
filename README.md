# Crimson Desert / DenuvOwO Hypervisor Crack Analizi

![Status](https://img.shields.io/badge/analysis-static-blue)
![Target](https://img.shields.io/badge/target-Crimson%20Desert-darkgreen)
![Verdict](https://img.shields.io/badge/verdict-no%20clear%20trojan-success)
![Risk](https://img.shields.io/badge/risk-stability%20issues%20possible-orange)
![Mode](https://img.shields.io/badge/type-hypervisor%20bypass-critical)

> Bu çalışma, paylaşılan paketin statik analizi sonucunda hazırlandı.  
> Paket, Denuvo benzeri korumaları aşmak için **driver, service, DLL injection, thread manipulation ve düşük seviye bellek işlemleri** kullanmaktadır.  

---

## İçindekiler

- [Hızlı Karar](#hızlı-karar)
- [Genel Yapı](#genel-yapı)
- [Konfigürasyon Özeti](#konfigürasyon-özeti)
- [Ring Seviyeleri](#ring-seviyeleri)
- [Ana Teknik Bulgular](#ana-teknik-bulgular)
- [Ne Bulunmadı?](#ne-bulunmadı)
- [Temizlik / Kaldırma Davranışı](#temizlik--kaldırma-davranışı)
- [Riskler](#riskler)
- [Nihai Değerlendirme](#nihai-değerlendirme)

---

## Hızlı Karar

| Başlık | Durum |
|---|---|
| Açık trojan bulgusu | Yok |
| Keylogger bulgusu | Yok |
| Stealer / veri çalma bulgusu | Yok |
| Açık backdoor / C2 bulgusu | Yok |
| Düşük seviye bypass davranışı | Var |
| Driver / service kullanımı | Var |
| Hypervisor zinciri | Var |
| BSOD / çökme riski | Var |
| Driver / service kalıntısı ihtimali | Var |

---

## Genel Yapı

Bu paket tek dosyalık basit bir crack değil. Birkaç katmandan oluşuyor ve genel mantığı, oyunu normal şekilde çalıştırmak yerine korumayı alttan müdahale ederek aşmak.

### 1) Loader katmanı
Bu katman:
- oyunu başlatıyor
- Steam tarafını yönlendiriyor
- gerekli DLL zincirini devreye sokuyor

### 2) Patch / koordinasyon katmanı
Bu katman:
- servis ve sürücü yönetiyor
- thread akışına müdahale ediyor
- bellek eşleme işlemleri yapıyor
- koruma zincirini koordine ediyor

### 3) Hypervisor / kernel katmanı
Bu bölüm:
- çekirdek seviyesine iniyor
- hypervisor tarafını yüklemeye çalışıyor
- koruma kontrollerini daha alt seviyeden etkilemeyi hedefliyor

### 4) Steam emülasyon / uyumluluk katmanı
Bu katman:
- oyunun Steam ortamında çalışıyormuş gibi davranmasını sağlıyor
- Steam API / overlay / client tarafını yönlendiriyor

---

## Konfigürasyon Özeti

### `DenuvOwO.ini`
Bu dosyada görülen yapı şunu gösteriyor:

- patchlenecek bölüm otomatik seçiliyor
- hypervisor otomatik yükleniyor / kaldırılıyor

Öne çıkan mantık:
- `Target=CrimsonDesert.exe`
- `Section=auto`
- `AutoLoadHV=true`
- `GoRevertMsg=true`

### `ColdClientLoader.ini`
Bu dosya da loader tarafının nasıl çalıştığını gösteriyor:

- hedef EXE belirlenmiş
- Steam client DLL yolları verilmiş
- ek DLL yükleme mantığı tanımlanmış
- çalışma modu ayarlanmış

---

## Ring Seviyeleri

Kısa ayrım:

- **Ring 3** = kullanıcı modu
- **Ring 0** = çekirdek modu
- **Ring -1** = hypervisor seviyesi

### Ring 3
Aşağıdaki bileşenlerin büyük kısmı kullanıcı modunda çalışıyor ya da bu seviyede yükleniyor:

- `DenuvOwO.dll`
- `steam_api64.dll`
- `steamclient.dll`
- `steamclient64.dll`
- `GameOverlayRenderer64.dll`
- `amd_ags_x64.dll`

Bu katman:
- süreç yönetimi
- DLL yükleme
- servis çağrılarını başlatma
- thread müdahalesi
- kullanıcı modu koordinasyonu

işlerini yürütüyor.

### Ring 0
- `hyperkd.sys`

`.sys` uzantılı sürücü olduğu için bu bileşen çekirdek modunda çalışacak bölümdür.

### Ring -1
Burada doğrudan bir DLL değil, oluşturulmaya çalışılan **hypervisor katmanı** var.

Yani mantık şu şekilde:
- kullanıcı modu bileşenleri zinciri başlatıyor
- sürücü tarafı çekirdek erişimini sağlıyor
- başarılı olursa asıl gizli müdahale mantığı hypervisor seviyesinde çalışıyor

Kısaca:
- DLL’lerin çoğu → **Ring 3**
- sürücü → **Ring 0**
- aktif hypervisor → **Ring -1**

---

## Ana Teknik Bulgular

## `DenuvOwO.dll` tarafı

### 1) Service / driver yönetimi
Görülen çağrılar:
- `OpenSCManagerW`
- `OpenServiceW`
- `CreateServiceW`
- `StartServiceW`
- `ControlService`
- `DeleteService`

Bu zincir, modülün servis/sürücü oluşturup başlatabildiğini, durdurabildiğini ve silebildiğini göstermektedir.

### 2) Dosya silme
Görülen çağrı:
- `DeleteFileW`

Bu da kurulan bileşenleri veya yardımcı dosyaları kaldırma mantığı olduğunu göstermektedir.

### 3) Thread manipulation
Görülen çağrılar:
- `OpenThread`
- `GetThreadContext`
- `SuspendThread`
- `SetThreadContext`
- `ResumeThread`

Bu davranış, çalışan thread’lerin akışını durdurup değiştirerek patch uygulamaya veya kontrolü ele almaya işaret etmektedir.

### 4) Section mapping
Görülen çağrılar:
- `NtCreateSection`
- `NtMapViewOfSection`
- `NtUnmapViewOfSection`

Bu çağrılar düşük seviyeli bellek eşleme tekniği kullanıldığını göstermektedir.

### 5) DSE / EFI / Code Integrity ilişkisi
Öne çıkan izler:
- `CI.dll`
- `CiPolicy`
- `CiInitialize`
- `NtSetSystemEnvironmentValueEx`
- `Couldn't set DSE mode. EfiGuard is not running correctly!`

Bu kombinasyon, yapının:
- **DSE** (Driver Signature Enforcement / sürücü imza zorlaması kapatma)
- **Code Integrity** (kod bütünlüğü/bellek bütünlüğü kapatma)
- **EFI** (önyükleme / firmware ortamı) - yeni versiyonlarda bu durum yok.

tarafına dokunduğunu gösteriyor.

Burada klasik bootkit kanıtı yok; fakat korumayı aşmak için EFI/DSE çevresine temas eden davranış açık biçimde var(yeni versiyonlarda bu duruma rastlanılmadı).

---

## Ne Bulunmadı?

Bu analizde şunlara dair net bulgu çıkmadı:

- klasik **keylogger** davranışı
- açık **stealer** davranışı
- **ransomware** mantığı
- açık **backdoor / C2** izi
- tarayıcı şifresi / çerez / hesap çalma davranışı

Özellikle şu tip çağrılar net biçimde öne çıkmadı:
- `SetWindowsHookEx`
- `GetAsyncKeyState`
- `socket`
- `connect`
- `send`
- `recv`

Bu yüzden yapı, veri çalan klasik malware’den çok, koruma bypass amacıyla agresif çalışan düşük seviyeli bir araç setidir.

---

## Temizlik / Kaldırma Davranışı

Paket yalnızca kurma değil, kaldırma / temizlik mantığı da içermektedir.

Bunu destekleyen davranış zinciri:
- `DeleteService`
- `ControlService`
- `DeleteFileW`
- `PathAppendW`
- `PathFileExistsW`
- `PathRemoveFileSpecW`

Bu mantık genelde şuna gider:

1. servis / sürücüyü durdur  
2. servis kaydını sil  
3. dosya yolunu oluştur  
4. dosyayı fiziksel olarak kaldırmaya çalış  

Yani paket kendi zincirini geri alma mantığına da sahip.  
Bu, her zaman kusursuz temizlik yaptığı anlamına gelmez.

---

## Riskler

En olası pratik riskler:

- **BSOD / mavi ekran**
- sistem donması
- oyun açılış hataları
- bazı sistemlerde hiç çalışmama
- driver / service kalıntısı bırakma
- güvenlik ayarlarının tam geri dönmemesi
- anti-cheat / antivirus / integrity mekanizmalarıyla çakışma

Buradaki ana risk veri çalma değil, daha çok **stabilite ve uyumluluk**.

---

## Nihai Değerlendirme

> **Açık trojan, keylogger veya veri çalan zararlı davranışı tespit edilmedi.**  
> **Görülen düşük seviyeli müdahaleler, büyük ölçüde Denuvo'yu atlatma amacıyla kullanılan tekniklerle uyumludur.**  

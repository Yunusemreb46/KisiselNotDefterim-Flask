# Kişisel Not ve Dosya Yöneticisi (C# & Avalonia UI)

**C# (.NET 10)** ve **Avalonia UI** kullanılarak geliştirilmiş; metin/kod notları oluşturmayı, düzenlemeyi, silmeyi ve bu notlara harici dosyalar (PDF, görsel, kaynak kod vb.) ekleyip yerel dosya sistemi üzerinde yönetmeyi sağlayan karanlık temalı çapraz platform masaüstü uygulamasıdır.

---

## 1. Sistem Mimarisi ve Çalışma Mantığı

Uygulama, harici bir veritabanı sunucusuna ihtiyaç duymadan **yerel dosya sistemi** ve **nesne yönelimli (OOP) olay güdümlü (event-driven)** masaüstü mimarisi üzerine kuruludur:

1. **Arayüz ve Kod Ayrımı (AXAML & Code-Behind):** Uygulama görünümü declarative **AXAML** (`App.axaml`, `MainWindow.axaml`) dosyalarıyla tanımlanırken; kullanıcı etkileşimleri, dosya kopyalama ve veri serileştirme işlemleri **C#** (`MainWindow.axaml.cs`, `Program.cs`) katmanında yürütülür.
2. **Nesne Yönelimli (OOP) Yapı:**
   * **Veri Modeli (`NotVerisi` Sınıfı):** Her notun başlığı (`Baslik`), metin/kod içeriği (`Icerik`) ve bağlı dosya listesi (`EkliDosyalar`) tek bir nesne modeli altında kapsüllenir.
   * **Kalıtım ve Kapsülleme:** `MainWindow : Window` ve `App : Application` sınıfları Avalonia çekirdek sınıflarından miras alır; dizin yolları ve aktif not durumu (`_anaKlasor`, `_aktifNot`) `private` alanlarla korunur.
3. **Olay Güdümlü (Event-Driven) Akış:** Uygulama, kullanıcı arayüzündeki buton tıklamaları (`Click="Kaydet_Click"`, `Click="NotSil_Click"`, `Click="DosyaEkle_Click"`, `Click="DosyaSil_Click"`) ve liste seçim değişiklikleri (`SelectionChanged="NotSecildi_Changed"`) gerçekleştiğinde ilgili C# metotlarını tetikleyerek çalışır.
4. **JSON Tabanlı Kalıcı Depolama (`System.Text.Json`):** Kaydedilen her not, `NotVerisi` modeli üzerinden serileştirilerek kullanıcının `Belgelerim/KisiselNotlarim` dizini altına kendi başlığıyla `.json` dosyası olarak yazılır.
5. **İzole Dosya Yedekleme (`Ekler` Dizini) ve Sistem Entegrasyonu:** Nota harici bir dosya eklendiğinde, Avalonia'nın platform bağımsız dosya seçicisi (`IStorageProvider`) ile seçilen dosya otomatik olarak `Belgelerim/KisiselNotlarim/Ekler` dizinine kopyalanır (`File.Copy`). Ekli dosyalar `System.Diagnostics.Process` (`UseShellExecute = true`) aracılığıyla işletim sisteminin varsayılan uygulamasıyla doğrudan açılır.

---

## 2. Gereksinimler ve Kütüphane Bağımlılıkları

Projeyi derlemek ve çalıştırmak için bilgisayarınızda **.NET 10.0 SDK** ve **Visual Studio Code** (C# Dev Kit eklentisi ile) kurulu olmalıdır.

* **NuGet Paket Bağımlılıkları (`KisiselNotDefterim.csproj` içinde tanımlıdır, `dotnet restore` ile otomatik kurulur):**
  * `Avalonia` (`11.*`): Çapraz platform masaüstü arayüz çekirdeği.
  * `Avalonia.Desktop` (`11.*`): Windows, Linux ve macOS masaüstü pencere yönetim altyapısı.
  * `Avalonia.Themes.Fluent` (`11.*`): Modern karanlık tema (`FluentTheme`) bileşenleri.
  * `Avalonia.Fonts.Inter` (`11.*`): Arayüz tipografisi için gömülü yazı tipi paketi.
  * `AvaloniaUI.DiagnosticsSupport` (`2.*`): Geliştirme aşaması arayüz hata ayıklama desteği.
* **.NET Standart Sınıf Kütüphaneleri (SDK ile hazır gelir):**
  * `System.Text.Json`: `NotVerisi` nesnelerinin okunabilir (`WriteIndented = true`) JSON formatında diske yazılması ve okunması.
  * `System.IO`: Dizin oluşturma (`Directory.CreateDirectory`), dosya listeleme, kopyalama ve silme işlemleri.
  * `System.Diagnostics`: Ekli dosyaların varsayılan sistem programıyla başlatılması.

---

## 3. Aşama Aşama Kurulum ve Çalıştırma

Depo boyutunu küçük ve temiz tutmak amacıyla proje arşivinde geçici derleme klasörleri (`bin/` ve `obj/`) bulunmamaktadır. Kurulum adımları tamamlandığında **.NET SDK** bu klasörleri ve gerekli derleme dosyalarını otomatik olarak oluşturur.

### Aşama 1: Kaynak Kodların İndirilmesi ve Çıkarılması
1. Depo sayfasının sağ üst köşesindeki **`<> Code`** menüsünden **`Download ZIP`** seçeneği ile `KisiselNotDefterim.zip` arşivini bilgisayarınıza indirin.
2. İndirdiğiniz `.zip` dosyasına sağ tıklayıp **Tümünü Ayıkla (Extract All)** seçeneği ile dosyaları yerel bir klasöre çıkartın.

### Aşama 2: Çalışma Ortamının Visual Studio Code ile Açılması
1. **Visual Studio Code** uygulamasını başlatın.
2. Üst menüden **File > Open Folder...**  yolunu izleyerek arşivden çıkardığınız **`KisiselNotDefterim`** proje klasörünü seçin.
3. Sol taraftaki **Explorer** (Dosya Gezgini) panelinde temel kaynak dosyaların (`KisiselNotDefterim.csproj`, `Program.cs`, `MainWindow.axaml`, `MainWindow.axaml.cs`) yüklendiğini doğrulayın.

### Aşama 3: NuGet Paketlerinin Geri Yüklenmesi (`dotnet restore`)
1. Visual Studio Code üst menüsünden **Terminal > New Terminal** (`Ctrl + "`) seçeneği ile dahili terminali açın.
2. Açılan terminal satırına aşağıdaki komutu yazıp `Enter` tuşuna basarak Avalonia UI bağımlılıklarını projeye indirin:
   ```bash
   dotnet restore
   ```

### Aşama 4: `bin` ve `obj` Klasörlerinin Oluşturulması ve Projenin Derlenmesi (`dotnet run`)
1. Paket yüklemesi tamamlandıktan sonra proje dizininde gerekli derleme klasörlerinin (`bin/` ve `obj/`) oluşması ve uygulamanın derlenmesi için aynı VS Code terminaline aşağıdaki komutu yazıp `Enter` tuşuna basın:
   ```bash
   dotnet run
   ```
2. Bu komut çalıştırıldığında .NET SDK eksik olan `bin/` ve `obj/` klasörlerini otomatik olarak oluşturur ve projeyi hatasız şekilde derler.

### Aşama 5: Son Aşama — Uygulamanın Çalışması ve Otomatik Klasörler
1. Derleme işleminin hemen ardından son aşamada **`Kişisel Not ve Dosya Yöneticim`** masaüstü uygulama penceresi otomatik olarak açılır ve çalışmaya başlar.
2. Uygulama açıldığı anda arka planda `Belgelerim/KisiselNotlarim` ve `Belgelerim/KisiselNotlarim/Ekler` veri klasörlerini otomatik olarak oluşturur.

---

## 4. Proje Dosya Yapısı

```text
KisiselNotDefterim/
├── App.axaml                  # Uygulama geneli Karanlık Tema (Dark) ve FluentTheme tanımları
├── App.axaml.cs               # Uygulama yaşam döngüsü ve MainWindow başlatıcısı
├── MainWindow.axaml           # Üç sütunlu (Grid) ana pencere arayüz tasarımı
├── MainWindow.axaml.cs        # Not ekleme/silme, dosya kopyalama/açma ve JSON kayıt mantığı
├── Program.cs                 # Uygulama giriş noktası (Main metodu ve Avalonia yapılandırması)
├── KisiselNotDefterim.csproj  # .NET 10 hedef çerçevesi ve NuGet paket referansları
├── app.manifest               # Windows uygulama uyumluluk manifestosu
├── .gitignore                 # bin/ ve obj/ derleme çıktılarını hariç tutma kuralları
└── README.md                  # Proje dokümantasyonu
```
*(Not: `bin/` ve `obj/` klasörleri `dotnet run` komutu çalıştırıldıktan sonra proje dizininde otomatik olarak oluşacaktır.)*

---

## 5. Arayüz Ekranı ve Kullanım Akışı

Uygulama arayüzü, uzun süreli kod ve metin çalışmaları için **Karanlık Tema (`#1E1E1E` / `#252526`)** paletiyle üç dikey sütun (`Grid ColumnDefinitions="220, *, 240"`) halinde yapılandırılmıştır:

* **Sol Panel — Notlarım (Kayıt Yönetimi):** `Belgelerim/KisiselNotlarim` dizinindeki tüm kayıtlı `.json` not başlıklarını `ListBox` içerisinde listeler. **`+ Yeni Not`** butonu giriş alanlarını sıfırlarken, **`Seçili Notu Sil`** butonu aktif notu hem listeden hem de diskten kalıcı olarak kaldırır.
* **Orta Panel — Başlık ve Not/Kod Editörü:** Üst kısımda dosya adını belirleyen başlık giriş kutusu (`BaslikKutusu`) ve **`Kaydet`** butonu bulunur. Alt kısımda ise kod bloklarının okunabilirliğini koruyan sabit genişlikli (`Consolas`) çok satırlı metin alanı (`IcerikKutusu`) yer alır.
* **Sağ Panel — Ekli Dosyalar (Dosya Yöneticisi):** Aktif nota bağlanan harici dosyaları listeler. **`+ Dosya Ekle`** ile seçilen dosyalar `Ekler` klasörüne kopyalanır; **`Seçili Dosyayı Aç`** dosyayı sistemin varsayılan programıyla açar; **`Seçili Dosyayı Sil`** ise dosyayı hem `Ekler` klasöründen hem de notun JSON kaydından siler.

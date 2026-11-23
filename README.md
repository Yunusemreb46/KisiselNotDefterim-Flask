# KisiselNotDefterim-Flask
C# (.NET 10) ve Avalonia UI kullanılarak geliştirilmiş; metin ve kod notları oluşturmayı, düzenlemeyi, silmeyi ve bu notlara harici dosyalar ekleyip yerel JSON formatında yönetmeyi sağlayan karanlık temalı çapraz platform masaüstü not ve dosya yöneticisi.


# Kişisel Not ve Dosya Yöneticisi (C# & Avalonia UI)

**C# (.NET 10)** ve **Avalonia UI** kullanılarak geliştirilmiş; metin/kod notları oluşturmayı, düzenlemeyi, silmeyi ve bu notlara harici dosyalar (PDF, görsel, kaynak kod vb.) ekleyip yerel dosya sistemi üzerinde yönetmeyi sağlayan karanlık temalı masaüstü uygulamasıdır.

---

## 1. Sistem Mimarisi ve Çalışma Mantığı

Uygulama, harici bir veritabanı sunucusuna ihtiyaç duymadan **yerel dosya sistemi** ve **nesne yönelimli (OOP) olay güdümlü (event-driven)** masaüstü mimarisi üzerine kuruludur:

1. **Arayüz ve Kod Ayrımı (XAML & Code-Behind):** Uygulama görünümü declarative **AXAML** (`App.axaml`, `MainWindow.axaml`) dosyalarıyla tanımlanırken; kullanıcı etkileşimleri, dosya kopyalama ve veri serileştirme işlemleri **C#** (`MainWindow.axaml.cs`, `Program.cs`) katmanında yürütülür.
2. **JSON Tabanlı Kalıcı Depolama (`System.Text.Json`):** Kaydedilen her not, `NotVerisi` modeli üzerinden serileştirilerek kullanıcının `Belgelerim/KisiselNotlarim` dizini altına kendi başlığıyla `.json` dosyası olarak yazılır. Dosya içerisinde notun başlığı, metin/kod içeriği ve nota bağlı ek dosyaların listesi tutulur.
3. **İzole Dosya Yedekleme (`Ekler` Dizini):** Nota harici bir dosya eklendiğinde, Avalonia'nın platform bağımsız dosya seçicisi (`IStorageProvider`) ile seçilen dosya otomatik olarak `Belgelerim/KisiselNotlarim/Ekler` dizinine kopyalanır (`File.Copy`). Bu sayede orijinal dosya taşınırsa veya silinirse veri kaybı yaşanmaz.
4. **Doğrudan Sistem Entegrasyonu:** Ekli dosyalar `System.Diagnostics.Process` (`UseShellExecute = true`) aracılığıyla işletim sisteminin varsayılan uygulamasıyla (PDF okuyucu, görsel görüntüleyici, kod editörü vb.) doğrudan açılır.

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

## 3. Aşama Aşama Kurulum ve Çalıştırma (Başlangıçtan Sonuca)

Projeyi ZIP arşivinden çıkarıp **Visual Studio Code** ortamında çalıştırmak için aşağıdaki adımları sırasıyla uygulayın:

### Aşama 1: Kaynak Kodların İndirilmesi ve Çıkarılması
1. Depo sayfasının sağ üst köşesindeki **`<> Code`** menüsünden **`Download ZIP`** seçeneği ile proje arşivini bilgisayarınıza indirin.
2. İndirdiğiniz `.zip` dosyasına sağ tıklayıp **Tümünü Ayıkla (Extract All)** seçeneği ile dosyaları yerel bir klasöre çıkartın.

### Aşama 2: Çalışma Ortamının Visual Studio Code ile Açılması
1. **Visual Studio Code** uygulamasını başlatın.
2. Üst menüden **File > Open Folder...** (`Ctrl + K Ctrl + O`) yolunu izleyerek arşivden çıkardığınız **`KisiselNotDefterim`** proje klasörünü seçin.
3. Sol taraftaki **Explorer** (Dosya Gezgini) panelinde `KisiselNotDefterim.csproj`, `Program.cs`, `MainWindow.axaml` ve `MainWindow.axaml.cs` dosyalarının yüklendiğini doğrulayın.

### Aşama 3: NuGet Paketlerinin Geri Yüklenmesi (`dotnet restore`)
1. Visual Studio Code üst menüsünden **Terminal > New Terminal** (`Ctrl + "`) seçeneği ile dahili terminali açın.
2. Açılan terminal satırına aşağıdaki komutu yazıp `Enter` tuşuna basarak Avalonia UI kütüphanelerini projeye indirin:
   ```bash
   dotnet restore

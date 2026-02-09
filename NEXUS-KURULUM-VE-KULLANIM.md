# Nexus Repository Manager 3 – Kurulum ve Kullanım Rehberi

Bu rehber, NexusApp projesinde NuGet paketlerini Nexus üzerinden kullanmak için gereken kurulum ve kullanım adımlarını özetler.

---

## 1. Gereksinimler

- **Docker** ve **Docker Compose** yüklü olmalı.
- Windows’ta: [Docker Desktop](https://www.docker.com/products/docker-desktop/) kurulu ve çalışır durumda olmalı.

Kontrol için:

```bash
docker --version
docker compose version
```

---

## 2. Nexus’u Başlatma

Proje kök dizininde:

```bash
docker compose up -d
```

- Nexus **http://localhost:8081** adresinde çalışır.
- İlk açılışta veritabanı hazırlanır; 1–2 dakika sürebilir.
- Durumu kontrol etmek için: tarayıcıda `http://localhost:8081` açın.

---

## 3. İlk Giriş ve Admin Şifresi

1. **Admin şifresini bulma** (container içinden):

   ```bash
   docker exec nexus cat /nexus-data/admin.password
   ```

   Çıkan satırı kopyalayın; bu geçici admin şifresidir.

2. Tarayıcıda **http://localhost:8081** açın.
3. Sağ üstten **Sign in** → **admin** kullanıcı adı, şifre olarak yukarıdaki değeri yapıştırın.
4. İlk girişte **yeni bir kalıcı şifre** belirlemeniz istenir; bunu kaydedin.

---

## 4. NuGet Proxy Repository Oluşturma

NuGet paketlerini nuget.org üzerinden proxy’lemek için:

1. **Settings** (dişli ikon) → **Repository** → **Repositories** → **Create repository**.
2. **Recipe** olarak **nuget (proxy)** seçin → **Next**.
3. Örnek ayarlar:
   - **Name:** `nuget.org-proxy`
   - **Remote storage:** `https://api.nuget.org/v3-index/index.json`
   - **Blob store:** `default`
   - **Cleanup policies:** (isteğe bağlı)
4. **Create repository** ile kaydedin.

Bu adımlardan sonra NuGet proxy adresiniz:

- **http://localhost:8081/repository/nuget.org-proxy/index.json**

olur. Projedeki `nuget.config` zaten bu adresi kullanıyor.

---

## 5. (İsteğe Bağlı) Hosted Repository (Kendi Paketleriniz)

Kendi NuGet paketlerinizi yüklemek için:

1. **Create repository** → **nuget (hosted)** seçin.
2. **Name:** örneğin `nuget-hosted`.
3. **Create repository** ile oluşturun.
4. Proje kökündeki `nuget.config` içinde `nexus-hosted` ile ilgili yorum satırlarını kaldırıp kendi hosted repo URL’inizi kullanabilirsiniz.

---

## 6. Projede Kullanım (nuget.config)

Bu projede NuGet, varsayılan olarak Nexus proxy’yi kullanacak şekilde ayarlıdır:

- **Kaynak:** `http://localhost:8081/repository/nuget.org-proxy/index.json`
- Dosya: proje kökündeki **`nuget.config`**

Paket eklerken:

```bash
dotnet add package PaketAdi
```

komutu paketleri Nexus üzerinden indirir (Nexus çalışıyorsa).

---

## 7. Nexus’u Durdurma

```bash
docker compose down
```

Veriler `nexus-app-data` volume’unda kalır; tekrar `docker compose up -d` ile aynı verilerle devam edersiniz.

---

## 8. Sık Karşılaşılan Sorunlar

| Sorun | Çözüm |
|--------|--------|
| **8081’e bağlanamıyorum** | Docker Desktop’ın çalıştığından ve `docker compose up -d` yaptığınızdan emin olun. Birkaç dakika bekleyin. |
| **Admin şifresi bulunamıyor** | `docker exec nexus cat /nexus-data/admin.password` ile tekrar deneyin. Container adı `nexus` olmalı. |
| **NuGet restore/paket indirme hatası** | Nexus’un ayakta olduğunu ve `nuget.org-proxy` repository’sinin oluşturulduğunu kontrol edin. `nuget.config` yolunun doğru olduğundan emin olun. |
| **HTTPS / güvenlik uyarısı** | `nuget.config` içinde `allowInsecureConnections="true"` sadece local (http) kullanım içindir; production’da HTTPS kullanın. |

---

## Özet Kontrol Listesi

1. Docker kurulu ve çalışıyor.
2. `docker compose up -d` ile Nexus ayağa kalktı.
3. `http://localhost:8081` açılıyor.
4. Admin ile giriş yapıldı, kalıcı şifre belirlendi.
5. **nuget.org-proxy** repository’si oluşturuldu.
6. Projede `dotnet restore` veya `dotnet add package` sorunsuz çalışıyor.

Bu adımlar tamamsa Nexus kurulumu ve kullanımı hazırdır.

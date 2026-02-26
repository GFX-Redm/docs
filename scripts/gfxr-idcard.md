# gfxr-idcard

Western temalı kimlik kartı sistemi. Oyuncu bilgilerini parşömen tarzı bir ID card üzerinde gösterir.

## Özellikler

- Western/1899 temalı tasarım
- Otomatik mugshot çekimi (karakter ilk yüklendiğinde)
- Tüm framework'lerle uyumlu (VORP, RSG, RedEM)
- Configurable theme ve locale desteği
- Sheriff/Police mugshot çekme yetkisi

## Gereksinimler

| Dependency | Açıklama |
|------------|----------|
| `gfx-bridge` | Framework abstraction |
| `screenshot-basic` | Mugshot çekimi için |

## Kurulum

1. **Resource'u ekle:**
   ```
   ensure gfx-bridge
   ensure screenshot-basic
   ensure gfxr-idcard
   ```

2. **SQL'i çalıştır:**
   ```sql
   -- sql/install.sql dosyasını import et
   CREATE TABLE IF NOT EXISTS `gfxr_mugshots` (
       `id` INT(11) NOT NULL AUTO_INCREMENT,
       `identifier` VARCHAR(255) NOT NULL,
       `mugshot` TEXT NOT NULL,
       `taken_by` VARCHAR(255) DEFAULT NULL,
       `created_at` TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
       PRIMARY KEY (`id`),
       UNIQUE KEY `identifier` (`identifier`)
   ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
   ```

3. **Config'i ayarla:**
   - `config/client_config.lua` - Tema, mugshot ayarları
   - `config/locale.lua` - Dil çevirileri

## Config

### client_config.lua

```lua
Config = {}

Config.Debug = false

-- Tema renkleri
Config.Theme = {
    Primary = "#c21414",        -- Ana renk (mühür, ID no)
    PrimaryContent = "#ffffff",
    Secondary = "#DAA520",      -- İkincil renk
    Background = "#F4E4C1",     -- Parşömen arka plan
    Ink = "#1a1613",            -- Yazı rengi
    Parchment = "#F4E4C1"
}

-- ID Card bilgileri
Config.IdCard = {
    Territory = "New Hanover",
    EstablishedYear = "1899",
    Authority = "United States Marshal Service",
    Quote = "Justice is a commodity few can afford in these parts.",
    Signature = "Sheriff Curtis Malloy"
}

-- Mugshot ayarları
Config.Mugshot = {
    UploadURL = "https://your-server.com/upload",  -- Resim upload endpoint
    FieldName = "file",                             -- Form field adı
    AutoCapture = true,                             -- Otomatik çekim
    Camera = {
        Fov = 30.0,
        Distance = 0.6,
        HeightOffset = 0.63
    }
}

-- Komutlar
Config.Command = "idcard"           -- ID kartı aç
Config.MugshotCommand = "takemugshot"  -- Mugshot çek
Config.Keybind = "F5"               -- Tuş ataması (false = devre dışı)

-- Mugshot çekme yetkisi olan job'lar (boş = herkes)
Config.MugshotJobs = {
    "sheriff",
    "police",
    "lawman"
}

Config.Locale = "en"  -- "en" veya "tr"
```

### locale.lua

```lua
Locale = {}

Locale.en = {
    territory = "Territory of",
    identification_document = "IDENTIFICATION DOCUMENT",
    official_registry = "Official Registry of the",
    legal_name = "Legal Full Name",
    date_of_birth = "Date of Birth",
    occupation = "Current Occupation",
    town_county = "Town / County",
    official_signature = "Official Signature",
    approved_by = "APPROVED BY",
    judicial_court = "JUDICIAL COURT",
    subject_ref = "SUBJECT REF",
    id_no = "ID NO."
}

Locale.tr = {
    territory = "Bolgesi",
    identification_document = "KIMLIK BELGESI",
    -- ...
}
```

## Komutlar

| Komut | Açıklama |
|-------|----------|
| `/idcard` | ID kartını aç/kapat |
| `/takemugshot` | En yakın oyuncunun mugshot'ını çek |
| `/takemugshot [id]` | Belirtilen ID'li oyuncunun mugshot'ını çek |
| `/selfmugshot` | Kendi mugshot'ını çek |

## Exports (Client)

```lua
-- ID kartını aç
exports['gfxr-idcard']:OpenIdCard()

-- ID kartını kapat
exports['gfxr-idcard']:CloseIdCard()

-- ID kartı açık mı?
local isOpen = exports['gfxr-idcard']:IsIdCardOpen()

-- Mugshot çek (target server ID)
exports['gfxr-idcard']:TakeMugshot(targetServerId)

-- En yakın oyuncunun mugshot'ını çek
exports['gfxr-idcard']:TakeMugshotNearest()

-- Kendi mugshot'ını çek
exports['gfxr-idcard']:TakeSelfMugshot()
```

## Exports (Server)

```lua
-- Oyuncunun mugshot URL'ini al
local mugshotUrl = exports['gfxr-idcard']:GetMugshot(source)
```

## Events

### Client

```lua
-- ID kartı verileri alındığında
RegisterNetEvent('gfxr-idcard:receivePlayerData', function(data)
    -- data.firstName, data.lastName, data.mugshot, etc.
end)

-- Otomatik mugshot çekimi tetiklendiğinde
RegisterNetEvent('gfxr-idcard:autoCaptureMugshot', function()
    -- Mugshot yok, otomatik çekilecek
end)
```

### Server

```lua
-- Mugshot kaydetme
RegisterNetEvent('gfxr-idcard:saveMugshot', function(targetServerId, mugshotUrl)
    -- Mugshot kaydedildi
end)
```

## Mugshot Upload Server

Upload endpoint'in döndürmesi gereken format:

```json
{
    "url": "https://your-server.com/uploads/abc123.jpg"
}
```

### Örnek PHP Endpoint

```php
<?php
header('Content-Type: application/json');

$uploadDir = 'uploads/';
$fileName = uniqid() . '.jpg';
$targetPath = $uploadDir . $fileName;

if (move_uploaded_file($_FILES['file']['tmp_name'], $targetPath)) {
    echo json_encode([
        'url' => 'https://your-server.com/' . $targetPath
    ]);
} else {
    http_response_code(500);
    echo json_encode(['error' => 'Upload failed']);
}
```

## Veri Kaynakları

| UI Alanı | Bridge Export | Framework Kaynağı |
|----------|---------------|-------------------|
| Ad Soyad | `GetPlayer()` | `firstname`, `lastname` |
| Doğum Tarihi | `GetPlayer()` | `dateofbirth` / `birthdate` |
| Meslek | `GetPlayerJob()` | `job.label` |
| Mugshot | `ExecuteSql()` | `gfxr_mugshots` tablosu |
| ID No | `GetIdentifier()` | Hash'lenerek üretilir |

## Notlar

- Mugshot otomatik çekimi karakter yüklendiğinde tetiklenir
- Mugshot yoksa sessizce çekilir, kullanıcıya bildirim gösterilmez
- ID numarası karakter identifier'ından hash'lenerek üretilir (her seferinde aynı kalır)
- Sheriff/Police job'undaki oyuncular başka oyuncuların mugshot'larını çekebilir

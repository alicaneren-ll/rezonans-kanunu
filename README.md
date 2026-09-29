# Rezonans Kanunu — İnteraktif Yaşam ve Frekans Rehberi

Pierre Franckh'ın *"Rezonans Kanunu" (Das Gesetz der Resonanz)* eseri, seminer notları ve öğretilerinden derlenen **tek dosyalık** interaktif web uygulaması. İçerik dili Türkçedir, arayüz İngilizce/Türkçe arasında geçiş yapabilir ve tüm veriler tarayıcıda `localStorage` ile saklanır — sunucu, hesap veya derleme adımı yoktur.

> **Not:** Bu proje kişisel gelişim ve meditasyon amaçlı bir çalışma aracıdır. İçerikler bir psikiyatrik veya tıbbi öneri değildir.

---

## Hızlı Başlangıç

Proje tek bir `index.html` dosyasından oluşur. Kurulum, bağımlılık ve derleme gerektirmez.

```bash
# Dosyayı doğrudan tarayıcıda açın
start index.html
```

Yerel bir HTTP sunucusuyla çalıştırmak isterseniz:

```bash
python -m http.server 8000
# ardından tarayıcıdan http://localhost:8000 adresini açın
```

İnternet bağlantısı yalnızca CDN'den Tailwind CSS ve Google Fonts yüklenirken gereklidir; uygulamanın geri kalanı tamamen çevrimdışı çalışır.

---

## ⚠️ Yönetim Girişi

Uygulama açılışta bir **yönetim kilidi** ister:

| | |
| --- | --- |
| Hesap sayısı | Tam iki hesap. **Kullanıcı adları da kaynak kodda yoktur** — ilk çalıştırmada siz belirlersiniz. |
| Şifreler | Yine ilk çalıştırmada siz belirlersiniz (kaynak kodda yok, varsayılan yok) |
| Şifre saklama | Yalnızca SHA-256 hash'i (`localStorage`, düz metin hiçbir yerde yok) |
| Oturum süresi | 8 saat, sonra yeniden giriş gerekir |
| Minimum şifre | 8 karakter (Türkçe karakter ve emoji desteklenir) |
| Kullanıcı adı kuralı | En az 3 karakter; harf, rakam, `.`, `_`, `-` (Türkçe karakter desteklenmez) |

Giriş **kullanıcı adı + şifre** ister. Kullanıcı adı büyük/küçük harf ve çevresi boşluk duyarsızdır: `AHMET.Y`, `ahmet.y` ve `  ahmET.y  ` aynı hesaptır. Kullanıcı adı veya şifreden hangisinin yanlış olduğu bilinmez, hata mesajı her zaman aynıdır.

> **Bu bir güvenlik sınırı değildir.** Koruma tamamen tarayıcı tarafında çalışır ve yalnızca kaba yükten (çocuklar, misafirler, refleks) saklar. Gerçek gizlilik veya yetkilendirme için bir sunucu katmanı gerekir.
>
> **İlk açılışta kurulum ekranı gelir** — iki hesap için de kullanıcı adı + şifre + şifre tekrarı girip kaydedin. Kaydettikten sonra ilk hesapla otomatik giriş yapılır. Paneldeki **Yeni Şifre Üret** düğmesi o hesabın şifresini rastgele üretir ve **bir kez** gösterir; eski şifre anında geçersiz olur.

Kullanıcı adları kurulumda bir kez belirlenir ve sonradan değiştirilemez; şifreler panelden bağımsız olarak değiştirilebilir. Kurulumu tamamlamadan önce sayfayı kapatırsanız bir sonraki açılışta kurulum ekranı tekrar gelir.

### Panelde neler var?

| Sekme | İşlev |
| --- | --- |
| **📝 İçerik** | 9 koleksiyonu (düşünür kartları, dil tuzakları, su frekansları, günün kartları, inanç örnekleri, fiziksel yer kategorileri, alfa senaryoları, dirençsiz işaretler, arayüz sözlüğü) form veya JSON editörüyle düzenleme. Kaydet → uygulama anında yeniden render edilir. Ekle / sil / çoğalt / sırala ve "Geri Al" desteği. |
| **📊 Analitik** | Toplam ziyaret, yönetim girişi sayısı, ilk/son kullanım damgası, bölüm bazlı görüntülenme ve etkileşim sayaçları, en çok açılan alanlar. |
| **💾 Yedekleme** | Tüm içerik override'larını tek `.json` dosyasına aktarma / geri yükleme, hesap şifrelerini değiştirme, yeni şifre üretme, başlangıç değerlerine sıfırlama. **Şifreler ve kullanıcı adları yedeklenmez.** |

**Tehlikeli işlemler:** "Tüm içerikleri sıfırla" yalnızca içerik override'larını siler; Dilek Sandığı, kanıt zinciri, 72 saat sayacı ve şifreler korunur. "Analitiği sıfırla" yalnızca sayaçları temizler.

### Yedek dosyası biçimi

```json
{
  "meta": {
    "app": "rezonans-kanunu",
    "format": 1,
    "exportedAt": "2026-01-01T00:00:00.000Z",
    "accountCount": 2
  },
  "content": { "cards": [ /* ... */ ], "trapDictionary": [ /* ... */ ] }
}
```

`meta.accountCount` yalnızca kaç hesabın tanımlı olduğunu söyler. **Kullanıcı adları ve şifreler (hash dahil) yedeğe hiç dâhil edilmez.** Yedek yüklendiğinde yalnızca içerik gelir; giriş bilgileri o cihazdakilerden belirlenir. Bu sayede aynı yedek farklı cihazlara yüklendiğinde her cihaz kendi kullanıcı adlarını ve şifrelerini kullanır.

Eski formatlı yedekler (`meta.users`, `meta.username`, `meta.hasCustomPassword`, `version`) sorunsuz yüklenir; içerik alınır, kimlik bilgileri değişmez.

---

## Teknoloji

| Katman | Seçim |
| --- | --- |
| Yapı | Saf HTML5 + CSS + JavaScript (ES5 tarzı, framework yok) |
| Stil | Tailwind CSS 3 (`cdn.tailwindcss.com`, `tailwind.config` ile özel palet) |
| Fontlar | Plus Jakarta Sans (sans), Playfair Display (serif) — Google Fonts |
| Tema | Özel `cosmic` / `aura` renk paleti, `darkMode: 'class'` |
| Durum | `localStorage` (sunucu yok) |
| Bağımlılık | `package.json` yok, `node_modules` yok |

---

## Dosya Yapısı

```
rezon/
├── index.html   # Tüm uygulama: HTML, CSS, JS ve veri dizileri tek dosyada
└── README.md    # Bu dosya
```

---

## Özellikler

| Modül | Açıklama |
| --- | --- |
| **4 Temel Ayak** | Elektromanyetik alan, kalbin 5.000 kat gücü, bilinçaltı inanç kodları, *Loslassen* (bırakabilme) sanatı |
| **Kök İnanç Dedektifi** | "Ama" ile başlayan direnç cümlesini bulma, kaynak sorgulama (aile / okul / toplum / eski yara) ve teşekkürlü sembolik yakım animasyonu |
| **Köprü Cümle Laboratuvarı** | Konu + süreç kalıbı + esneklik eki birleşimiyle dirençsiz olumlama cümlesi üretimi, kopyalama ve sandığa ekleme |
| **Dil Filtresi & Tuzak Dedektörü** | Cümledeki negatif kök kelimeleri (`hasta`, `borç`, `yalnız`, `istemiyorum`…) ve olumsuzluk eklerini tespit edip "saf rezonans" karşılığını önerir; hazır sözlük akordeonu |
| **Fiziksel Yer Açma** (*Raum Schaffen*) | 4 kategoride (aşk, bolluk, kariyer, ev) 20 eylemlik kontrol listesi, ilerleme çubuğu, kalıcı işaretleme |
| **Su Rezonansı Laboratuvarı** | Dr. Masaru Emoto esinli 12 frekans seçeneği + özel niyet alanı, canlı su bardağı animasyonu ve 15 saniyelik koherans yükleme ritüeli |
| **Alfa / Teta Uyku Eşiği** | Gece öncesi ve sabah ilk 5 dakika modları, 5 rehberli imgeleme senaryosu, adım adım ilerleyen metin oynatıcı |
| **Restoran Siparişi** | Niyeti "sipariş" olarak gönderme ve kontrol arzusunu bırakma simülasyonu |
| **72 Saat Kuralı** | 15 hazır dirençsiz işaret kütüphanesi, rastgele işaret üretici, kalıcı geri sayım ve kanıt zinciri listesi |
| **İlişkilerde Aynalama** | Aynalama ilkesi, öz sevgi ve "kişiyi değil ilişkinin kalbini talep et" kartları |
| **6 Adımlı Niyet Sihirbazı** | Boyut → formül → kalp hissi → şükran → bırakış → ilham adımları, sonunda Dilek Sandığı'na mühürleme |
| **Kalp Koheransı** | 5 sn nefes al / 5 sn nefes ver ritmiyle animasyonlu nefes dairesi ve sıfırlama |
| **Düşünür Kartları** | Planck, Einstein, Tesla, Braden, Buddha, Goethe, Emoto, Saint-Exupéry, Franckh — kategoriye göre filtrelenebilir, kopyalanabilir |
| **Günün Frekansı** | Rastgele rezonans kartı modalı |
| **Alıntı Spotlights** | Rastgele alıntı döngüsü |
| **Dilek Sandığı** | Tüm niyet, köprü cümle ve teslimiyet kayıtlarının kart ızgarası; silme / toplu temizleme |
| **Tema & Dil** | Karanlık / aydınlık mod ve TR / EN dil geçişi, tercihler kalıcı |
| **Yönetim Girişi** | Kullanıcı adı + SHA-256 şifre, 8 saatlik oturum, iki kurulumda belirlenen hesap. **Varsayılan kullanıcı adı/şifre yoktur** (kaba yük engeli, güvenlik sınırı değil) |
| **Yönetim Paneli** | 9 koleksiyonu tarayıcıdan düzenleme, JSON yedekleme/geri yükleme, ziyaret ve bölüm analitiği |

---

## Veri Modeli ve localStorage Anahtarları

Tüm kalıcı veriler tarayıcının `localStorage` alanında tutulur; hiçbir istek dışarı çıkmaz.

| Anahtar | İçerik |
| --- | --- |
| `rezonans_kanunu_wishes_v2` | Dilek Sandığı kayıtları (`title`, `text`, `type`, `released`, `date`, `id`) |
| `rezonans_kanunu_evidence_v2` | 72 saat kuralı kanıt zinciri listesi |
| `rezonans_kanunu_sync_timer_v2` | Aktif 72 saatlik niyet (`text`, `endTime`) |
| `rezonans_clearing_checks_v1` | Fiziksel yer açma kontrol listesi durumu (id → boolean) |
| `rezonans_theme` | `dark` \| `light` |
| `rezonans_app_lang` | `tr` \| `en` |
| `rezonans_admin_session` | Yönetim oturumu (`{ exp, user }`, 8 saat) |
| `rezonans_admin_users_v1` | Yönetim kadrosu: `[{ "name": "...", "hash": "<64 hex>" }, ...]` — iki kayıt. Şifreler yalnızca hash olarak tutulur. |
| `rezonans_admin_content_v1` | Panelden yapılan içerik override'ları (anahtar → dizi/nesne) |
| `rezonans_admin_analytics_v1` | Sayaçlar (`visits`, `adminLogins`, `sections`, `events`, `firstSeen`, `lastSeen`) |

### İç Veri Dizileri

Veriler `index.html` içindeki `<script>` bloğunda `DEFAULT_*` önekiyle tanımlı düz diziler olarak durur ve kolayca genişletilebilir:

- `translations` — TR/EN arayüz sözlüğü (`data-i18n` anahtarları)
- `thinkerQuotes` / `quotes` — düşünür kartları ve alıntı havuzu
- `cards` — günün frekansı kartları
- `trapDictionary` — dil tuzakları sözlüğü (`wrong` / `traps` / `clean` / `reason`)
- `clearingCategories` — fiziksel yer açma kategorileri ve eylemleri
- `waterFrequencies` — 12 su frekansı seçeneği
- `alphaScenarios` — `{ evening: [...], morning: [...] }` imgeleme senaryoları
- `resistanceFreeSigns` — 15 dirençsiz senkronisite işareti
- `beliefExamples` — örnek inanç dönüşümü kalıpları

### Kaynak dizi mi, panel içeriği mi okunuyor?

Her koleksiyon şu sırayla çözülür:

```js
ADMIN.getContent('cards', DEFAULT_CARDS)   // panelde kaydedilmiş varsa o, yoksa kaynak dizi
```

Panelden yapılan her düzenleme `rezonans_admin_content_v1` içine yazılır ve **kaynak `DEFAULT_*` dizisine dokunulmaz**. Bu yüzden:

- `index.html` içindeki diziyi düzenlemek → kalıcı değişiklik, ama paneldeki bir override varsa gölgelenir.
- Panelde "Geri Al" → o koleksiyon kaynak diziye geri döner ve override silinir.
- Yedeği içe aktarmak → yalnızca override katmanını değiştirir; `index.html` olduğu gibi kalır.

Yani kaynak dosya her zaman yedek planı olarak çalışır.

---

## Özelleştirme

### Yeni bir dil tuzağı eklemek

`trapDictionary` dizisine yeni bir nesne eklemek yeterlidir; akordeon otomatik yeniden render edilir:

```js
{
  wrong: "Yalnız kalmak istemiyorum.",
  traps: ["yalnız", "istemiyorum"],
  clean: "Kendimle dolu, huzurlu ve sevgi dolu bağlar kurmaya açığım.",
  reason: "'Yalnız' kelimesi yokluk frekansını besler."
}
```

`analyzeTrapWords()` içindeki `triggerWords` listesine yeni kök kelimeler ekleyerek tespit kapsamını genişletebilirsiniz.

### Yeni bir su frekansı eklemek

```js
{ label: "Yaratıcılık", symbol: "🎨", note: "Özgün fikirlerin doğuşu" }
```

Bu kaynak diziye (`DEFAULT_WATER_FREQUENCIES`) eklenir. Yalnızca tarayıcıda değiştirmek isterseniz yönetim panelindeki **Su Frekansları** koleksiyonunu düzenleyin.

### Yeni bir fiziksel eylem eklemek

`clearingCategories` içindeki `items` dizisine `{ id, text }` formatında kayıt ekleyin. `id` benzersiz olmalıdır.

### Tailwind paletini değiştirmek

`<head>` içindeki `tailwind.config` bloğunda `colors.cosmic` ve `colors.aura` grupları, `animation` ve `keyframes` tanımları bulunur.

### Aydınlık tema uyarlamaları

`<style>` bloğundaki `body.light-mode ...` kuralları, karanlık temada kullanılan her Tailwind renk sınıfı için aydınlık karşılık tanımlar. Yeni bir koyu renkli yüzey eklerseniz aynı deseni burada taklit edin.

---

## Kullanım Notları

- **Köprü cümleler** yalnızca üç seçim kutusundan anında üretilir; üçünü değiştirdikçe cümle yeniden kurulur.
- **72 saat sayacı** `endTime` damgasını sakladığı için sayfa kapatılıp yeniden açıldığında kalan süre korunur.
- **Nefes egzersizi** saf istemci tarafıdır; tarayıc sekmesi arka plana alınırsa `setInterval` throttling'e tabi olabilir.
- **Panoya kopyalama** `document.execCommand('copy')` ile çalışır; bu yöntem modern tarayıcılarda giderek daha fazla tarayıcı güvenliği kısıtlamasına tabidir.
- **Panelden girilen metin çalıştırılamaz.** Tüm içerik render'ları `escapeHtml()` kullanır ve hiçbir yerde `innerHTML` içine gömülü `onclick` yoktur; etkileşimler `data-*` öznitelikleriyle olay temsilciliğine bağlanır. Yine de **JSON yedeği yalnızca kendi ürettiğiniz dosyalardan yükleyin** — başka birinin dosyası güvenilmeyen girdidir.
- **Kilit ve giriş katmanı Tailwind'e bağlı değildir.** `#admin-login`, `body.admin-locked .app-shell` ve `#admin-panel` kuralları dosyanın kendi `<style>` bloğundadır; CDN yüklenemese bile kilit ekranı doğru çalışır.

---

## Tarayıcı Desteği

Uygulama `localStorage`, `backdrop-filter`, `IntersectionObserver` ve `crypto.subtle` kullandığı için çok eski tarayıcılarda görsel bozulma veya eksik özellik beklenir.

Şifre hash'i iki yolla hesaplanır: `crypto.subtle` (güvenli bağlam) ve kullanılabilir olduğunda saf JS fallback'i. İkisi de UTF-8 baytlar üzerinde çalışır ve aynı sonucu üretir; `file://` gibi `crypto.subtle`'ın kapalı olduğu bağlamlarda da Türkçe karakterli şifreler sorunsuz çalışır.

Kullanıcı adı karşılaştırması `trim().toLowerCase()` ile yapılır, yani büyük/küçük harf ve çevresi boşluk fark etmez. Kullanıcı adı hatalı olsa bile şifre hash'i yine hesaplanır; böylece hata mesajı ve yanıt süresi hangi alanın yanlış olduğunu ele vermez.

### Doğrulama durumu

| Alan | Kapsam |
| --- | --- |
| SHA-256 (her iki yol) | Node 24 ile 16 farklı girdi (Türkçe karakter, emoji, CJK, 55/56/57/64/1000 bayt) karşılaştırmalı doğrulandı — hepsi referans hash ile aynı |
| Kaynak sızıntısı | `index.html` ve `README.md` üzerinde otomatik tarama: eski kullanıcı adları, eski üretilmiş parolalar, 64 haneli sabit hash, tek kullanıcılık dönemine ait iki eski localStorage anahtarı ve sabit hesap listesi **0 eşleşme** |
| Uçtan uca tarayıcı | Chrome (headless, `http://127.0.0.1` üzerinden) ile **50 test, 0 hata**: kaynak taraması, ilk kurulumda 6 alanın doğru çıkması, uygulama kabuğunun gizlenmesi, kısa/geçersiz/tekrarlı kullanıcı adı reddi, şifre eşleşmezliği reddi, iki hesabın hash olarak kaydı ve localStorage'da düz şifre bulunmaması, ilk hesapla otomatik giriş, yenilemede oturum kalıcılığı, yedekte kullanıcı adı/hash sızmaması, çıkış ve yanlış şifrede kilitli kalma, bilinmeyen kullanıcı adının aynı mesajla reddi, büyük/küçük harf ve boşluk duyarsız giriş, panelde iki kullanıcı satırı, panelden şifre değiştirme, rastgele şifre üretme ve üretilen şifreyle giriş |
| Duyarlılık | Chrome headless ile **7 viewport, 0 hata** (1920×1080, 1366×640, 1280×720, 1024×600, 820×1180, 390×844, 360×640): kurulum ve giriş kartı ekrana sığıyor, üstten kırpılmıyor |

Edge, Firefox ve Safari üzerinde otomatik test yapılmadı; bu tarayıcılarda manuel olarak doğrulanması önerilir.

---

## Lisans ve Atıf

İçerik ve tasarım, Pierre Franckh'ın *Das Gesetz der Resonanz* eseri ile Dr. Masaru Emoto'nun su kristali deneyleri, Gregg Braden'ın HeartMath çalışmaları ve sayfada alıntılanan diğer düşünürlerin kamu malı metinlerine dayanır. Kod tarafı bu çalışma için özgündür.

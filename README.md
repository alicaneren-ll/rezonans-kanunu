# Rezonans Kanunu — İnteraktif Yaşam ve Frekans Rehberi

Pierre Franckh'ın *"Rezonans Kanunu" (Das Gesetz der Resonanz)* eseri, seminer notları ve öğretilerinden derlenen **tek dosyalık** interaktif web uygulaması. İçerik dili Türkçedir, arayüz İngilizce/Türkçe arasında geçiş yapabilir ve tüm veriler tarayıcıda `localStorage` ile saklanır — sunucu, cihazlar arası hesap veya derleme adımı yoktur. Uygulama **doğrudan açılır**: giriş ekranı, hesap sistemi ve oturum yoktur. Dilek Sandığı, 72 saat kayıtları ve sayaç cihaz düzeyindedir; aynı tarayıcıda her açılışta aynı verileri görürsünüz.

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

## 🚫 Giriş Yok

Uygulamada **hesap, giriş ekranı ve kullanıcı oturumu yoktur.** Dosya tarayıcıda
açıldığında içerik doğrudan görünür; kurulum ekranı, kullanıcı adı, şifre veya
başlıktaki Çıkış düğmesi bulunmaz.

| | |
| --- | --- |
| Veri kapsamı | **Cihaz düzeyi** — aynı tarayıcıda her açılışta aynı kayıtlar |
| Kullanıcı hesabı, oturum, kadro | Yok, kaynak kodda da yok |
| Kişisel veri izolasyonu | Yok — cihazı eline alan kişi kayıtları görür |

> Aynı cihazda birden fazla kişi kullanacaksa bu uygulama uygun değildir:
> kayıtlar arasında ayrım yoktur. Gerçek gizlilik veya yetkilendirme için bir
> sunucu katmanı gerekir.

### Kayıtların cihaz anahtarına taşınması

Kullanıcı sistemi eklenmeden önce kayıtlar cihaz düzeyindeki anahtarlarda
tutuluyordu; hesap sistemi döneminde ise kullanıcı adı eklenerek şu anahtarlara
taşındı:

```
rezonans_kanunu_wishes_v2__<kullanici>      # o kullanıcının Dilek Sandığı
rezonans_kanunu_evidence_v2__<kullanici>    # o kullanıcının 72 saat kanıtı
rezonans_kanunu_sync_timer_v2__<kullanici>  # o kullanıcının aktif sayacı
```

`adoptUserScopedData()` her açılışta şunu yapar:

1. Cihaz anahtarı (`rezonans_kanunu_wishes_v2` vb.) **yoksa** o anda
   bulunan kullanıcı ekli anahtarlardan alfabetik olarak **ilk** olanı cihaz
   anahtarına kopyalar.
2. Kullanıcı ekli anahtarları **silmez** — yedek olarak durmaya devam eder.
3. Artık kullanılmayan `rezonans_user_session`, `rezonans_users_v1` ve
   `rezonans_legacy_migrated_v1` anahtarlarını temizler.

Cihaz anahtarı zaten varsa hiçbir şey kopyalanmaz; mevcut kayıtlar korunur.
Birden fazla kullanıcının ayrı kaydı varsa yalnızca biri cihaz anahtarına gelir;
diğerleri `__<kullanıcı>` anahtarlarında elle kalır.

> Fiziksel yer açma kontrol listesi (`rezonans_clearing_checks_v1`) zaten cihaz
> düzeyindedir; taşınan bir şey yoktur.

---

## 🛡️ Yönetim Kapısı

İçerik panelini açan kapı **uygulamanın geri kalanından bağımsızdır**: açılışta
hiçbir şey sormaz, yalnızca panel açılmak istendiğinde devreye girer. Dışarıdan
gelenler bu şifreyi bilmeden panele giremez:

| | |
| --- | --- |
| Şifre | Kaynak kodunda **varsayılanı yoktur**. İlk kez panel açılmak istendiğinde kapı "şifre belirle" modunda açılır ve belirlersiniz. |
| Saklama | Yalnızca SHA-256 hash'i (`rezonans_admin_pw`), düz metin hiçbir yerde yok |
| Oturum süresi | 8 saat, sonra tekrar şifre gerekir |
| Değiştirme | Panelden "Şifreyi Değiştir" (mevcut şifre doğrulanır) veya "Yeni Şifre Üret" (rastgele, bir kez gösterilir) |
| Unutma kurtarma | Şifre belirlenirken **bir kez gösterilen bir kurtarma kodu** üretilir. Kapıdaki "Şifremi Unuttum" ile bu kod doğrulanır; eski şifre ve eski kod silinir, hemen yeni şifre belirlenir. |
| Etkisi | **Yalnızca içerik paneline erişir.** Kayıtlara veya uygulama kullanımına dokunmaz. |

"Yönetimden Çık" yalnızca panelin kilidini sıfırlar; **uygulama açık kalır** ve
kayıtlar etkilenmez.

### Kurtarma kodu

Şifre her belirlendiğinde veya değiştirildiğinde 16 karakterlik, tahmin edilemez bir
**kurtarma kodu** üretilir (`XXXX-XXXX-XXXX-XXXX`, karışmayan karakterlerle). Düz
metin kod **yalnızca bir kez** panelde gösterilir; tarayıcıda yalnızca SHA-256
hash'i (`rezonans_admin_recovery_v1`) saklanır. Panel kapanınca veya kapı
açılınca kodun düz metni DOM'dan da silinir.

- **Kaybettiyseniz:** kapıdaki "Şifremi Unuttum" → kodu girin → eski şifre ve eski
  kod kalıcı olarak silinir → yeni şifre belirlersiniz. İçerik ve kayıtlar
  **hiç etkilenmez**.
- **Yanlış kod** şifreyi silmez; yalnızca reddedilir.
- **Yenileme:** panel açıkken "Kurtarma Kodu Yenile" eski kodu anında geçersiz
  kılar ve yenisini bir kez daha gösterir.
- **Kod da kaybolursa** kurtarma yolu yoktur: yönetim şifresi geri alınamaz. Bu
  durumda yedekten içerikler geri yüklenebilir, ancak yönetim kapısı için tarayıcı
  verisi elle temizlenmelidir.
- Kod **yedeklenmez**; yedek dosyasına hiçbir kimlik bilgisi girmez.

Kurtarma kodu, bu cihazdaki yönetim şifresine erişmek içindir. Cihaz başkasının
eliyle tutuluyorsa o kişi zaten tarayıcı verisine erişebilir; bu yüzden kod
**gizli bir kimlik doğrulama yöntemi değil, yalnızca hatırlama kolaylığıdır.**

### Panelde neler var?

| Sekme | İşlev |
| --- | --- |
| **📝 İçerik** | 9 koleksiyonu (düşünür kartları, dil tuzakları, su frekansları, günün kartları, inanç örnekleri, fiziksel yer kategorileri, alfa senaryoları, dirençsiz işaretler, arayüz sözlüğü) form veya JSON editörüyle düzenleme. Kaydet → uygulama anında yeniden render edilir. Ekle / sil / çoğalt / sırala ve "Geri Al" desteği. |
| **📊 Analitik** | Toplam ziyaret, ilk/son kullanım damgası, bölüm bazlı görüntülenme ve etkileşim sayaçları, en çok açılan alanlar. Sayaçlar cihaz düzeyindedir; "Sandıktaki Niyet" ve "Kanıt Kaydı" da o cihazdaki kayıtları sayar. |
| **💾 Yedekleme** | Tüm içerik override'larını tek `.json` dosyasına aktarma / geri yükleme, yönetim şifresini değiştirme veya yeni şifre üretme, kurtarma kodu yenileme, başlangıç değerlerine sıfırlama. **Şifreler ve kurtarma kodları yedeklenmez.** |

**Tehlikeli işlemler:** "Tüm içerikleri sıfırla" yalnızca içerik override'larını siler; Dilek Sandığı, kanıt zinciri, 72 saat sayacı ve şifreler korunur. "Analitiği sıfırla" yalnızca sayaçları temizler.

### Yedek dosyası biçimi

```json
{
  "meta": {
    "app": "rezonans-kanunu",
    "format": 1,
    "exportedAt": "2026-01-01T00:00:00.000Z"
  },
  "content": { "cards": [ /* ... */ ], "trapDictionary": [ /* ... */ ] }
}
```

**Yalnızca içerik yedeklenir.** Yönetim şifresi, kurtarma kodu ve kayıtlar yedeğe
hiç dâhil edilmez; yedek yüklendiğinde gelen tek şey içerik katmanıdır.

Eski formatlı yedekler (`meta.users`, `meta.username`, `meta.accountCount`,
`meta.hasCustomPassword`, `version`) sorunsuz yüklenir; içerik alınır, ek
alanlar yok sayılır.

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
| **Girişsiz kullanım** | Giriş ekranı, hesap, oturum ve Çıkış düğmesi **yoktur**; dosya açılır açılmaz uygulama görünür |
| **Cihaz düzeyi kayıtlar** | Dilek Sandığı, 72 saat kanıtı ve sayaç cihaz düzeyindeki sabit anahtarlarda tutulur; kullanıcıya göre ayrıştırma yapılmaz |
| **Yönetim Kapısı** | Yalnızca içerik panelini açan gizli şifre. Kaynak kodunda varsayılanı yoktur, ilk kullanımda belirlenir, yalnızca hash olarak saklanır |
| **Kurtarma Kodu** | Şifre belirlenirken bir kez gösterilen 16 karakterlik kod; unutulan yönetim şifresini kod ile sıfırlamayı sağlar, yalnızca hash olarak saklanır |
| **Yönetim Paneli** | 9 koleksiyonu tarayıcıdan düzenleme, JSON yedekleme/geri yükleme, ziyaret ve bölüm analitiği, yönetim şifresi yönetimi |

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
| `rezonans_admin_pw` | Yönetim şifresinin SHA-256 hash'i |
| `rezonans_admin_recovery_v1` | Kurtarma kodunun SHA-256 hash'i (düz metin hiçbir yerde saklanmaz) |
| `rezonans_admin_session` | Yönetim paneli oturumu (`{ exp }`, 8 saat) |
| `rezonans_admin_content_v1` | Panelden yapılan içerik override'ları (anahtar → dizi/nesne) |
| `rezonans_admin_analytics_v1` | Sayaçlar (`visits`, `sections`, `events`, `firstSeen`, `lastSeen`) |

`rezonans_kanunu_*_v2__<kullanıcı>` biçimindeki eski anahtarlar kullanılmaz;
`adoptUserScopedData()` onları okur, gerekirse cihaz anahtarına kopyalar ve
sonra yalnızca kullanılmayan `rezonans_user_session`, `rezonans_users_v1` ve
`rezonans_legacy_migrated_v1` anahtarlarını siler. Kullanıcı ekli kayıtlar
kullanıcı adı ekiyle birlikte **silinmez**.

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
- **Yönetim kapısı Tailwind'e bağlı değildir.** `#admin-gate`, `body.admin-gate-open #admin-gate` ve `#admin-panel` kuralları dosyanın kendi `<style>` bloğundadır; CDN yüklenemese bile yönetim kapısı doğru çalışır. Uygulama kabuğu kilitlenmez, `body.app-locked` kuralı artık yoktur.
- **Kayıtlar cihaz düzeyindedir.** Aynı tarayıcıda her açılışta aynı Dilek Sandığı ve 72 saat kayıtları görülür; ayrı hesap veya oturum kavramı yoktur.
- **Üst çubuk dar ekranda sarılır.** 390px altındaki genişliklerde başlık bloğu ile düğme grubu alt alta geçer, "Günün Frekansı" etiketi gizlenir; yatay sayfa kaydırması oluşmaz.

---

## Tarayıcı Desteği

Uygulama `localStorage`, `backdrop-filter`, `IntersectionObserver` ve `crypto.subtle` kullandığı için çok eski tarayıcılarda görsel bozulma veya eksik özellik beklenir.

Şifre hash'i iki yolla hesaplanır: `crypto.subtle` (güvenli bağlam) ve kullanılabilir olduğunda saf JS fallback'i. İkisi de UTF-8 baytlar üzerinde çalışır ve aynı sonucu üretir; `file://` gibi `crypto.subtle`'ın kapalı olduğu bağlamlarda da Türkçe karakterli şifreler sorunsuz çalışır.

Yönetim şifresi doğrulamasında hash yine her denemede hesaplanır; böylece hata mesajı ve yanıt süresi şifrenin doğru olup olmadığını ele vermez.

### Doğrulama durumu

| Alan | Kapsam |
| --- | --- |
| SHA-256 (her iki yol) | Node 24 ile 16 farklı girdi (Türkçe karakter, emoji, CJK, 55/56/57/64/1000 bayt) karşılaştırmalı doğrulandı — hepsi referans hash ile aynı |
| Kaynak sızıntısı | `index.html` ve `README.md` üzerinde tarama: giriş/kayıt/oturum/kullanıcı kadrosu işlevi, `userKey`, `accountCount`, `lastLogin`, giriş sayacı ve kullanıcı listesi işareti **0 eşleşme** (yalnızca `registerVisit` kalır, ziyaret sayacıdır) |
| Uçtan uca tarayıcı | Chrome headless (CDP) ile **27 test, 0 hata**: giriş markup'ının ve kilit sınıfının kalkması, uygulama kabuğunun görünürlüğü, bölüm render'ı, kapının başlangıçta gizli olması, **eski kullanıcı ekli kaydın cihaz anahtarına alınması ve sandıkta görünmesi, oturum ve kadro anahtarlarının silinmesi, kullanıcı ekli kaydın korunması, mevcut cihaz anahtarının ezilmemesi**, yönetim kapısının ilk kullanımda şifre belirlemesi, kurtarma kodunun bir kez gösterilip yalnızca hash olarak saklanması, yedekte kod sızmaması, yönetimden çıkışın uygulamayı açık bırakması ve kapıyı doğrulama moduna döndürmesi, yanlış şifrenin reddi, analitik görünümü ve giriş sayacının olmaması, Escape ile panel kapanması, 360px'te yatay kaydırma olmaması |
| Özellik dönüşü | Chrome headless (CDP) ile **13 test, 0 hata**: tema ve dil geçişi + kalıcılık, günün frekansı modalı, niyet ekleme/silme/yeniden yükleme sonrası kalıcılık, yedek özetinin kimlik bilgisi içermemesi, içerik override'ı ve sıfırlama, yakalanmamış istisna yok |
| Duyarlılık | Chrome headless ile **7 viewport, 0 hata** (1920×1080, 1366×640, 1280×720, 1024×600, 820×1180, 390×844, 360×640): uygulama kabuğu görünür, yönetim kapısı kartı ekrana sığıyor, yatay sayfa kaydırması oluşmuyor |
| Statik denetim | 4 satır içi `<script>` bloğunun tamamı `node --check` ile ayrıştırılıyor; `getElementById` hedeflerinin tamamı markup'ta mevcut; kullanılmayan `data-*` kancası yok |
| Yayınlanan sürüm | Bu değişiklikten sonra GitHub Pages sürümü yeniden test edilmedi. Yayın öncesi canlı adreste aynı CDP kontrollerinin çalıştırılması önerilir. |


Edge, Firefox ve Safari üzerinde otomatik test yapılmadı; bu tarayıcılarda manuel olarak doğrulanması önerilir.

---

## Lisans ve Atıf

İçerik ve tasarım, Pierre Franckh'ın *Das Gesetz der Resonanz* eseri ile Dr. Masaru Emoto'nun su kristali deneyleri, Gregg Braden'ın HeartMath çalışmaları ve sayfada alıntılanan diğer düşünürlerin kamu malı metinlerine dayanır. Kod tarafı bu çalışma için özgündür.

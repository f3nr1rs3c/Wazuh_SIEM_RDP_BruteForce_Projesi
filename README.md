# Wazuh SIEM ile RDP Brute-Force Saldırı Tespiti

![Wazuh](https://img.shields.io/badge/SIEM-Wazuh%20v4.12.0-1a73e8)
![Platform](https://img.shields.io/badge/platform-Windows%20Server%202022%20%7C%20Kali%20Linux-informational)
![Category](https://img.shields.io/badge/category-SOC%20%7C%20Blue%20Team-red)
![License](https://img.shields.io/badge/license-MIT-lightgrey)

Bu depo, sıfırdan kurulan bir **Wazuh SIEM** ortamında, **Windows Server
2022** hedefine karşı **Kali Linux** üzerinden **Hydra** ile gerçekleştirilen
bir **RDP brute-force (kaba kuvvet) saldırısının** uçtan uca tespit edilip
analiz edilmesini belgeleyen bir laboratuvar çalışmasıdır.

📄 **[Raporun tamamı (PDF)](./Wazuh_SIEM_Brute_Force_Tespit_Raporu.pdf)**

---

## Bu Proje Nedir?

Bu proje, gerçekçi bir SOC (Security Operations Center) laboratuvarının
küçük ölçekte kurulup işletildiği uygulamalı bir siber güvenlik çalışmasıdır.
VMware üzerinde birbirine bağlı üç sanal makine kullanılmıştır:

- **Wazuh Server (Linux)** — log toplama, korelasyon ve alert üretiminden
  sorumlu SIEM platformu,
- **Windows Server 2022** — saldırıya maruz bırakılan hedef sistem,
- **Kali Linux** — saldırının gerçekleştirildiği makine.

Kali üzerinden Windows Server'ın RDP servisine (port 3389) karşı Hydra
aracıyla gerçek bir parola deneme (brute-force) saldırısı başlatılmış; bu
saldırının Wazuh SIEM tarafından nasıl tespit edildiği, hangi kuralın
tetiklendiği ve oluşan alert'lerin nasıl analiz edildiği adım adım ekran
görüntüleriyle belgelenmiştir.

## Neden Yapıldı?

Bu çalışmanın amacı, bir SOC analistinin gerçek iş akışını uygulamalı olarak
deneyimlemektir:

**Log toplama → korelasyon → alert üretimi → olay (incident) analizi**

Saldırı senaryosu olarak **RDP brute-force** özellikle seçilmiştir, çünkü
gerçek dünyada kurumsal ağlara sızmak için kullanılan en yaygın "ilk erişim"
(initial access) tekniklerinden biridir (MITRE ATT&CK [T1110 — Brute
Force](https://attack.mitre.org/techniques/T1110/)). Bu nedenle, bu tür bir
saldırıyı bir SIEM üzerinde uçtan uca tespit edebilmek, defansif siber
güvenlik (blue team) pratiği açısından temel ve kritik bir beceridir.

Proje; kurulum, saldırı ve tespit aşamalarının her birinin gerçek çıktılarla
(terminal çıktıları, Wazuh Dashboard ekran görüntüleri, Windows Event Viewer
kayıtları) desteklenerek belgelenmesini hedeflemiştir.

## Lab Topolojisi

```
┌───────────────────────────┐
│   Wazuh Server (Linux)    │   192.168.230.128
│ Manager + Indexer +       │   v4.12.0
│ Dashboard                 │
└─────────────▲─────────────┘
              │ Wazuh Agent (TCP/1514)
┌─────────────┴─────────────┐
│  Windows Server 2022      │   192.168.230.130
│  Hedef — RDP (TCP/3389)   │   (Standard Evaluation, 21H2)
└─────────────▲─────────────┘
              │ Hydra ile brute-force (RDP)
┌─────────────┴─────────────┐
│     Kali Linux            │   192.168.230.131
│  Hydra v9.6, Nmap         │
└────────────────────────────┘

        Tamamı VMware Workstation üzerinde, 192.168.230.0/24 ağında
```

## Kullanılan Araç ve Ortamlar

| Bileşen | Detay |
|---|---|
| Sanallaştırma | VMware Workstation |
| SIEM platformu | Wazuh v4.12.0 (Manager + Indexer + Dashboard) |
| Hedef sistem | Windows Server 2022 Standard Evaluation (Build 20348.587) |
| Saldırı makinesi | Kali Linux |
| Saldırı aracı | Hydra v9.6 |
| Hedef protokol | RDP — TCP/3389 |
| Tespit kaynağı | Windows Security Event Log — Event ID 4625 |
| Tetiklenen kural | Wazuh Rule ID 60122 (Level 5) |

## Çalışmanın Adımları

Rapor, aşağıdaki yedi aşamayı ekran görüntüleriyle birlikte belgelemektedir:

1. **Wazuh Server Kurulumu** — Manager, Indexer ve Dashboard servislerinin
   Linux üzerine kurulumu ve servis durumlarının doğrulanması.
2. **Windows Server 2022 Kurulumu** — Hedef sistemin sürüm ve ağ
   yapılandırmasının doğrulanması.
3. **Wazuh Agent Kurulumu** — Windows Server'a agent kurulumu ve
   Dashboard üzerinden "Active" statüsünün teyidi.
4. **Ağ Bağlantısı Doğrulama** — Kali Linux'tan ping ve Nmap ile
   erişilebilirlik ve açık RDP portunun tespiti.
5. **Brute-Force Alert Kuralı** — Event ID 4625'i izleyen Wazuh varsayılan
   kuralının (Rule 60122) incelenmesi.
6. **Brute-Force Saldırısı** — Hydra ile Administrator hesabına karşı
   5 farklı parola denemesi gerçekleştirilmesi.
7. **Alert Tespiti ve Analizi** — Windows Event Viewer ve Wazuh Threat
   Hunting modülünde oluşan 7 alert'in detaylı incelenmesi (saldırgan IP,
   hedef kullanıcı, logon type vb.).

## Elde Edilen Sonuçlar

| Metrik | Değer |
|---|---|
| Denenen parola sayısı | 5 |
| Oluşan Wazuh alert sayısı | 7 (Rule 60122, Level 5) |
| Başarısız / başarılı girişim | 7 / 0 |
| Tespit edilen saldırgan IP | ✅ 192.168.230.131 |
| Tespit edilen hedef hesap | ✅ Administrator |
| Tespit gecikmesi | Neredeyse gerçek zamanlı |

## MITRE ATT&CK Eşleşmesi

| Taktik | Teknik |
|---|---|
| Credential Access | [T1110 — Brute Force](https://attack.mitre.org/techniques/T1110/) |
| Credential Access | [T1110.001 — Password Guessing](https://attack.mitre.org/techniques/T1110/001/) |
| Lateral Movement | [T1021.001 — Remote Desktop Protocol](https://attack.mitre.org/techniques/T1021/001/) |

## Çıkarımlar ve Öneriler

- RDP hesapları için **account lockout policy** (hesap kilitleme politikası)
  uygulanmalı.
- RDP portu (3389) doğrudan internete açılmamalı; VPN veya bastion host
  arkasına alınmalı.
- Ayrıcalıklı hesaplar için **MFA (çok faktörlü kimlik doğrulama)** zorunlu
  kılınmalı.
- Wazuh üzerinde, tekrarlayan 60122 uyarılarını tek bir yüksek öncelikli
  alert'e dönüştüren bir **frequency correlation kuralı** eklenmeli.
- Saldırgan IP'leri otomatik olarak engellemek için Wazuh'un **Active
  Response** modülü yapılandırılmalı.

## Sorumluluk Reddi

Bu çalışma tamamen izole, çevrimdışı bir sanal laboratuvar ortamında
**eğitim amaçlı** (defansif siber güvenlik / SOC analisti eğitimi)
gerçekleştirilmiştir. Kullanılan tüm IP adresleri özel bir lab alt ağına
(`192.168.230.0/24`) aittir; hiçbir harici veya canlı/üretim sistemine
saldırı yapılmamıştır.

## Lisans

[MIT](LICENSE) lisansı ile yayınlanmıştır.

---

**Yazar:** Doğukan İspirli

# 🌐 Cisco Packet Tracer Lab: 4 Router ile Merkez–Şube WAN ve Statik Yönlendirme

Bu proje, **Merkez Ofis** ve **Şube Ofis** yerel ağlarının (LAN), ortadaki iki omurga/WAN yönlendiricisi (`Omurga_R3` ve `Omurga_R4`) üzerinden seri bağlantılar (Serial DCE/DTE) ve **manuel statik yönlendirme (`ip route`)** kullanılarak uçtan uca haberleşmesini simüle eder.

---

## 🎯 Proje Hedefleri ve Öğrenilecekler
- **Noktadan Noktaya (Point-to-Point) WAN Tasarımı:** `/30` (`255.255.255.252`) alt ağ maskesi ile IP verimliliği.
- **Seri Hat Senkronizasyonu:** DCE (Data Communications Equipment) ve DTE (Data Terminal Equipment) kablo ayrımı ile `clock rate` yapılandırması.
- **Çok Atlamalı (Multi-Hop) Statik Yönlendirme:** Çift yönlü paket iletimi için her yönlendiricide hedef ağ ve sonraki durak (`next-hop`) tanımlama.
- **Uçtan Uca Doğrulama:** `ping`, `tracert`, `show ip route` ve `show interfaces` komutları ile ağ analizi.

---

## 🛠️ Topoloji ve Donanım Listesi
- **4 adet Router:** Cisco 2911 (`Merkez_R1`, `Omurga_R3`, `Omurga_R4`, `Sube_R2`)
- **4 adet Seri Arayüz Modülü:** `HWIC-2T` (Her bir router'a 1 adet eklenir)
- **2 adet Switch:** Cisco 2960 (`Merkez_SW1`, `Sube_SW2`)
- **4 adet PC:** `PC1`, `PC2` (Merkez Ofis) ve `PC3`, `PC4` (Şube Ofis)
- **Kablolama:**
  - **3 adet Serial DCE Kablosu** (Router'lar arası WAN hatları için)
  - **6 adet Copper Straight-Through (Düz) Kablo** (LAN bağlantıları için)

---

## 📊 IP Adresleme ve Bağlantı Planı

### 1. Yönlendirici (Router) Arayüzleri
| Cihaz Adı | Arayüz | Rol / Tip | IP Adresi | Alt Ağ Maskesi | Bağlı Olduğu Karşı Port |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Merkez_R1** | `Gi0/0` | Merkez LAN Gateway | `192.168.10.1` | `255.255.255.0` | `Merkez_SW1` (`Fa0/24`) |
| **Merkez_R1** | `Se0/0/0` | WAN (DCE - Clock) | `10.1.12.1` | `255.255.255.252` | `Omurga_R3` (`Se0/0/0` - DTE) |
| **Omurga_R3** | `Se0/0/0` | WAN (DTE) | `10.1.12.2` | `255.255.255.252` | `Merkez_R1` (`Se0/0/0` - DCE) |
| **Omurga_R3** | `Se0/0/1` | WAN (DCE - Clock) | `10.1.34.1` | `255.255.255.252` | `Omurga_R4` (`Se0/0/0` - DTE) |
| **Omurga_R4** | `Se0/0/0` | WAN (DTE) | `10.1.34.2` | `255.255.255.252` | `Omurga_R3` (`Se0/0/1` - DCE) |
| **Omurga_R4** | `Se0/0/1` | WAN (DCE - Clock) | `10.1.24.1` | `255.255.255.252` | `Sube_R2` (`Se0/0/0` - DTE) |
| **Sube_R2** | `Se0/0/0` | WAN (DTE) | `10.1.24.2` | `255.255.255.252` | `Omurga_R4` (`Se0/0/1` - DCE) |
| **Sube_R2** | `Gi0/0` | Şube LAN Gateway | `192.168.20.1` | `255.255.255.0` | `Sube_SW2` (`Fa0/24`) |

### 2. Uç Cihazlar (PC)
| Cihaz Adı | Lokasyon | Switch Portu | IP Adresi | Alt Ağ Maskesi | Default Gateway |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **PC1** | Merkez Ofis | `Merkez_SW1 (Fa0/1)` | `192.168.10.11` | `255.255.255.0` | `192.168.10.1` |
| **PC2** | Merkez Ofis | `Merkez_SW1 (Fa0/2)` | `192.168.10.12` | `255.255.255.0` | `192.168.10.1` |
| **PC3** | Şube Ofis | `Sube_SW2 (Fa0/1)` | `192.168.20.11` | `255.255.255.0` | `192.168.20.1` |
| **PC4** | Şube Ofis | `Sube_SW2 (Fa0/2)` | `192.168.20.12` | `255.255.255.0` | `192.168.20.1` |

---

## ⚙️ Router CLI Konfigürasyonları

### 1. `Merkez_R1`
```ios
enable
configure terminal
hostname Merkez_R1

interface GigabitEthernet0/0
 description Merkez_LAN
 ip address 192.168.10.1 255.255.255.0
 no shutdown
exit

interface Serial0/0/0
 description WAN_to_Omurga_R3_DCE
 ip address 10.1.12.1 255.255.255.252
 clock rate 64000
 no shutdown
exit

ip route 192.168.20.0 255.255.255.0 10.1.12.2
ip route 10.1.34.0 255.255.255.252 10.1.12.2
ip route 10.1.24.0 255.255.255.252 10.1.12.2
end
write memory
```

### 2. `Omurga_R3`
```ios
enable
configure terminal
hostname Omurga_R3

interface Serial0/0/0
 description WAN_to_Merkez_R1_DTE
 ip address 10.1.12.2 255.255.255.252
 no shutdown
exit

interface Serial0/0/1
 description WAN_to_Omurga_R4_DCE
 ip address 10.1.34.1 255.255.255.252
 clock rate 64000
 no shutdown
exit

ip route 192.168.10.0 255.255.255.0 10.1.12.1
ip route 192.168.20.0 255.255.255.0 10.1.34.2
ip route 10.1.24.0 255.255.255.252 10.1.34.2
end
write memory
```

### 3. `Omurga_R4`
```ios
enable
configure terminal
hostname Omurga_R4

interface Serial0/0/0
 description WAN_to_Omurga_R3_DTE
 ip address 10.1.34.2 255.255.255.252
 no shutdown
exit

interface Serial0/0/1
 description WAN_to_Sube_R2_DCE
 ip address 10.1.24.1 255.255.255.252
 clock rate 64000
 no shutdown
exit

ip route 192.168.20.0 255.255.255.0 10.1.24.2
ip route 192.168.10.0 255.255.255.0 10.1.34.1
ip route 10.1.12.0 255.255.255.252 10.1.34.1
end
write memory
```

### 4. `Sube_R2`
```ios
enable
configure terminal
hostname Sube_R2

interface Serial0/0/0
 description WAN_to_Omurga_R4_DTE
 ip address 10.1.24.2 255.255.255.252
 no shutdown
exit

interface GigabitEthernet0/0
 description Sube_LAN
 ip address 192.168.20.1 255.255.255.0
 no shutdown
exit

ip route 192.168.10.0 255.255.255.0 10.1.24.1
ip route 10.1.34.0 255.255.255.252 10.1.24.1
ip route 10.1.12.0 255.255.255.252 10.1.24.1
end
write memory
```

---

## ✅ Test ve Doğrulama Adımları

1. **Arayüz ve Hat Kontrolü:**
   Her router'da aşağıdaki komutu çalıştırarak fiziksel ve veri bağı katmanlarının (`up/up`) aktif olduğunu doğrulayın:
   ```ios
   show ip interface brief
   ```
2. **Yönlendirme Tablosu Kontrolü:**
   Statik rotaların (`S`) yönlendirme tablosuna doğru işlendiğini kontrol edin:
   ```ios
   show ip route
   ```
3. **Uçtan Uca Bağlantı Testi (PC3 -> PC1):**
   `PC3` (`192.168.20.11`) komut istemcisini (Command Prompt) açın ve Merkez Ofis'teki `PC1` (`192.168.10.11`) cihazına ping atın:
   ```cmd
   ping 192.168.10.11
   tracert 192.168.10.11
   ```
   **Beklenen Rota İzleme (`tracert`) Çıktısı:**
   1. `192.168.20.1` (`Sube_R2`)
   2. `10.1.24.1` (`Omurga_R4`)
   3. `10.1.34.1` (`Omurga_R3`)
   4. `10.1.12.1` (`Merkez_R1`)
   5. `192.168.10.11` (`PC1`)
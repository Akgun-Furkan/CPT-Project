# 🏢 Cisco Packet Tracer Lab: Çok Katlı Kurumsal Ağ ve Inter-VLAN Routing (Router-on-a-Stick)

Bu proje; 3 departman katı (**IT**, **ACCT/Muhasebe**, **İK**) ve en alt katta yer alan **Omurga Switch (`SW_Omurga`) + Router (`Kurumsal_Router`)** hiyerarşik mimarisi üzerinden departmanların mantıksal olarak ayrılmasını (VLAN) ve **IEEE 802.1Q (dot1Q)** protokolü ile **Router-on-a-Stick** yöntemi kullanılarak tam erişimle haberleşmesini simüle eder.

> **Not:** Bu senaryoda departmanlar arası herhangi bir ACL (IP engelleme) uygulanmamıştır; tüm departmanlar yönlendirici üzerinden birbirine serbestçe erişebilmektedir.

---

## 🎯 Proje Hedefleri ve Öğrenilecekler

* **Hiyerarşik Kat ve Omurga Tasarımı:** Kat erişim anahtarlarının (Access Switches) tek bir omurga anahtarında (Core/Distribution Switch) toplanması.
* **VLAN Segmentasyonu:** Departman bazlı yayın etki alanlarının (Broadcast Domain) oluşturulması (`VLAN 10`, `VLAN 20`, `VLAN 30`).
* **IEEE 802.1Q Trunk Protokolü:** Switch'ler arası ve Switch–Router arası hatlarda çoklu VLAN trafiğinin etiketlenerek taşınması.
* **Router-on-a-Stick (Sub-Interface):** Tek bir fiziksel router arayüzü üzerinde alt arayüzler oluşturularak Inter-VLAN Routing (VLAN'lar arası yönlendirme) sağlanması.

---

## 🛠️ Topoloji ve Donanım Listesi

* **1 adet Router:** Cisco 2911 (`Kurumsal_Router`)
* **1 adet Omurga Switch:** Cisco 2960 veya 3560 (`SW_Omurga` — Zemin Kat)
* **3 adet Kat Switch'i:** Cisco 2960 (`SW_IT`, `SW_ACCT`, `SW_IK`)
* **6 adet PC:** Her departman katında 2'şer adet istemci bilgisayar (`PC1`–`PC6`)
* **Kablolama:**
  * **10 adet Copper Straight-Through (Düz) Kablo** (6 adet PC–Switch, 3 adet Kat Switch–Omurga Switch, 1 adet Omurga Switch–Router)

---

## 📊 VLAN, IP Adresleme ve Port Planlaması

### 1. VLAN ve Alt Ağ (Subnet) Planı

| Departman | VLAN ID (`dot1Q`) | Alt Ağ (Subnet) | Alt Ağ Maskesi | Default Gateway (Router Sub-if) |
| :--- | :--- | :--- | :--- | :--- |
| **IT** | `VLAN 10` | `10.10.10.0 / 24` | `255.255.255.0` | `10.10.10.1` (`Gi0/0.10`) |
| **ACCT (Muhasebe)** | `VLAN 20` | `10.10.20.0 / 24` | `255.255.255.0` | `10.10.20.1` (`Gi0/0.20`) |
| **İK** | `VLAN 30` | `10.10.30.0 / 24` | `255.255.255.0` | `10.10.30.1` (`Gi0/0.30`) |

### 2. Port Port Bağlantı ve Uç Cihaz Tablosu

| Cihaz Adı | Bağlı Olduğu Switch & Port | Port Modu / VLAN | IP Adresi | Default Gateway |
| :--- | :--- | :--- | :--- | :--- |
| **PC1 (IT)** | `SW_IT` (`Fa0/1`) | Access — `VLAN 10` | `10.10.10.11` | `10.10.10.1` |
| **PC2 (IT)** | `SW_IT` (`Fa0/2`) | Access — `VLAN 10` | `10.10.10.12` | `10.10.10.1` |
| **PC3 (ACCT)** | `SW_ACCT` (`Fa0/1`) | Access — `VLAN 20` | `10.10.20.11` | `10.10.20.1` |
| **PC4 (ACCT)** | `SW_ACCT` (`Fa0/2`) | Access — `VLAN 20` | `10.10.20.12` | `10.10.20.1` |
| **PC5 (İK)** | `SW_IK` (`Fa0/1`) | Access — `VLAN 30` | `10.10.30.11` | `10.10.30.1` |
| **PC6 (İK)** | `SW_IK` (`Fa0/2`) | Access — `VLAN 30` | `10.10.30.12` | `10.10.30.1` |

### 3. Omurga ve Uplink Trunk Bağlantıları

* `SW_IT` (`Fa0/24`) $\leftrightarrow$ `SW_Omurga` (`Fa0/1`) — **802.1Q Trunk**
* `SW_ACCT` (`Fa0/24`) $\leftrightarrow$ `SW_Omurga` (`Fa0/2`) — **802.1Q Trunk**
* `SW_IK` (`Fa0/24`) $\leftrightarrow$ `SW_Omurga` (`Fa0/3`) — **802.1Q Trunk**
* `SW_Omurga` (`Fa0/24` veya `Gi0/1`) $\leftrightarrow$ `Kurumsal_Router` (`Gi0/0`) — **802.1Q Trunk**

---

## ⚙️ Cihaz CLI Konfigürasyonları

### 1. Kat 1 Switch: `SW_IT` (VLAN 10)

```text
enable
configure terminal
hostname SW_IT

vlan 10
 name IT
vlan 20
 name ACCT
vlan 30
 name IK
exit

interface range FastEthernet0/1 - 2
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast
exit

interface FastEthernet0/24
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
exit
end
write memory
```

### 2. Kat 2 Switch: `SW_ACCT` (VLAN 20)

```text
enable
configure terminal
hostname SW_ACCT

vlan 10
 name IT
vlan 20
 name ACCT
vlan 30
 name IK
exit

interface range FastEthernet0/1 - 2
 switchport mode access
 switchport access vlan 20
 spanning-tree portfast
exit

interface FastEthernet0/24
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
exit
end
write memory
```

### 3. Kat 3 Switch: `SW_IK` (VLAN 30)

```text
enable
configure terminal
hostname SW_IK

vlan 10
 name IT
vlan 20
 name ACCT
vlan 30
 name IK
exit

interface range FastEthernet0/1 - 2
 switchport mode access
 switchport access vlan 30
 spanning-tree portfast
exit

interface FastEthernet0/24
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
exit
end
write memory
```

### 4. Zemin Kat Omurga Switch: `SW_Omurga`

> **Not:** Omurga katında `3560-24PS` Multilayer Switch kullanıyorsanız trunk komutundan önce `switchport trunk encapsulation dot1q` komutunu girin. `2960` kullanıyorsanız doğrudan `switchport mode trunk` yeterlidir.

```text
enable
configure terminal
hostname SW_Omurga

vlan 10
 name IT
vlan 20
 name ACCT
vlan 30
 name IK
exit

interface range FastEthernet0/1 - 3
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
exit

interface FastEthernet0/24
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
exit
end
write memory
```

### 5. Zemin Kat Router: `Kurumsal_Router` (Router-on-a-Stick)

```text
enable
configure terminal
hostname Kurumsal_Router

interface GigabitEthernet0/0
 description Trunk_Uplink_SW_Omurga
 no shutdown
exit

interface GigabitEthernet0/0.10
 description Gateway_IT_VLAN10
 encapsulation dot1Q 10
 ip address 10.10.10.1 255.255.255.0
exit

interface GigabitEthernet0/0.20
 description Gateway_ACCT_VLAN20
 encapsulation dot1Q 20
 ip address 10.10.20.1 255.255.255.0
exit

interface GigabitEthernet0/0.30
 description Gateway_IK_VLAN30
 encapsulation dot1Q 30
 ip address 10.10.30.1 255.255.255.0
exit
end
write memory
```

---

## ✅ Doğrulama ve Test Adımları

1. **VLAN ve Trunk Kontrolü:**
   * Tüm switch'lerde `show vlan brief` ve `show interfaces trunk` komutlarıyla VLAN `10, 20, 30` tanımlarının ve trunk hatlarının aktif olduğunu doğrulayın.
2. **Router Alt Arayüz Kontrolü:**
   * `Kurumsal_Router` üzerinde `show ip interface brief` komutunu çalıştırın; `Gi0/0.10`, `Gi0/0.20` ve `Gi0/0.30` arayüzlerinin `up / up` durumda olduğunu görün.
3. **Departmanlar Arası Tam Erişim Testi (Ping):**
   * `PC1` (IT — `10.10.10.11`) komut istemcisinden diğer departmanlara ping atın:
     ```text
     ping 10.10.20.11   # ACCT (Muhasebe) erişimi -> Başarılı
     ping 10.10.30.11   # İK erişimi -> Başarılı
     ```
   * ACL uygulanmadığı için tüm departmanlar kendi ağ geçitleri (`10.10.x.1`) üzerinden birbirine sorunsuz erişebilmelidir.
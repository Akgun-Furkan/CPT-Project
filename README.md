#CPT-SOHO
================================================================================
                    CISCO PACKET TRACER LAB: SOHO & VLAN ROUTING
================================================================================

Proje Adı: Ev/Küçük Ofis (SOHO) Ağı ve Router-on-a-Stick (Statik Yönlendirme)
Açıklama: Bu proje, tek bir fiziksel anahtar (Switch) ve yönlendirici (Router) 
          kullanarak farklı sanal yerel ağların (VLAN) izole edilmesini ve 
          802.1Q (dot1Q) protokolü ile aralarında iletişim kurulmasını sağlar.

--------------------------------------------------------------------------------
1. TOPOLOJİ VE CİHAZ LİSTESİ
--------------------------------------------------------------------------------
- Router0 (Cisco 2911): Ağlar arası yönlendirme ve alt arayüz (sub-interface) merkezi.
- Switch0 (Cisco 2960): VLAN'ların tanımlandığı ve port gruplandırmasının yapıldığı cihaz.
- PC0 (Yöneticiler): VLAN 10 üyesi uç cihaz.
- PC1 (Personel): VLAN 20 üyesi uç cihaz.

--------------------------------------------------------------------------------
2. IP PLANLAMASI
--------------------------------------------------------------------------------
- VLAN 10 (Yöneticiler):
  * Ağ Adresi: 192.168.10.0 / 24
  * Gateway (Router Sub-if): 192.168.10.1
  * PC0 IP Adresi: 192.168.10.10 (Subnet: 255.255.255.0)

- VLAN 20 (Personel):
  * Ağ Adresi: 192.168.10.0 / 24 -> (192.168.20.0 / 24)
  * Gateway (Router Sub-if): 192.168.20.1
  * PC1 IP Adresi: 192.168.20.20 (Subnet: 255.255.255.0)

--------------------------------------------------------------------------------
3. KONFİGÜRASYON ÖZETİ
--------------------------------------------------------------------------------

[SWITCH KODLARI (Switch0)]
vlan 10
 name Yoneticiler
vlan 20
 name Personel
exit

interface fastethernet 0/1
 switchport mode access
 switchport access vlan 10

interface fastethernet 0/2
 switchport mode access
 switchport access vlan 20

interface fastethernet 0/24
 switchport mode trunk
end

[ROUTER KODLARI (Router0)]
interface gigabitethernet 0/0
 no shutdown
exit

interface gigabitethernet 0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
exit

interface gigabitethernet 0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
end

--------------------------------------------------------------------------------
4. DOĞRULAMA VE TEST ADIMLARI
--------------------------------------------------------------------------------
1. Switch üzerinde VLAN ve Port durumlarını kontrol edin:
   -> `show vlan brief`
   -> `show interfaces trunk`

2. Router üzerinde yönlendirme tablosunu kontrol edin:
   -> `show ip route`

3. PC0 üzerinden (192.168.10.10) farklı VLAN'daki PC1'e (192.168.20.20) ping atın:
   -> `ping 192.168.20.20`
   (İlk paket ARP çözümlemesi nedeniyle zaman aşımına uğrayabilir, sonrakiler başarılı olacaktır.)
================================================================================

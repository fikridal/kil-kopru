# kil-kopru
Türk Mitolojisi temalı, yüksek performanslı XDP (eXpress Data Path) tabanlı DDoS koruma sistemi.

# Kılköprü Nedir?

**Kılköprü**, trafiği kernel içinde filtreleyen, **XDP (eXpress Data Path)** tabanlı bir **DDoS scrubber** sistemidir. Saldırı trafiğini müşteri tanımlarına göre erken aşamada elemek veya sınırlamak için tasarlanmıştır.

---

## Neden "Kılköprü"?

İsim, Türk mitolojisindeki **Kıl Köprü** motifiyle uyumludur: sadece hak edenin geçebildiği bir köprü gibi, sadece meşru ve kurallara uyan trafik “köprüden” geçer; saldırı ve istenmeyen paketler elenir.

---

## Ne Yapar?

- **Gelen trafiği** hedef IP’ye göre müşteri prefix’lerine ayırır.
- **Whitelist / blacklist**, **port listeleri**, **rate limit** (PPS, BPS, SYN), **GeoIP**, **ASN** ve benzeri kuralları uygular.
- **SYN flood**, **UDP amplification**, **carpet bomb** gibi saldırı türlerini tespit edip sınırlar veya engeller.
- **Honeypot** subnet’leriyle otomatik blacklist’e alma desteği sunar.
- Geçen trafiği tanımlı çıkış arayüzüne yönlendirir; drop edilenleri isteğe bağlı olarak **ClickHouse**’a loglar.

Yani: **tek bir noktada**, kernel’e yakın ve düşük gecikmeyle, müşteri bazlı DDoS filtreleme ve yönlendirme yapar.

---

## Mimari (Özet)

| Katman | Görev |
|--------|--------|
| **XDP (eBPF)** | Kernel’de paket kararı: müşteri eşlemesi, blacklist/whitelist, port, rate limit, GeoIP/ASN, drop/pass. |
| **Daemon (Go)** | Yapılandırma ve müşteri verilerini yükler, BPF map’leri günceller, API ve Web arayüzünü sunar, drop log’u ClickHouse’a yazar. |
| **Web UI & API** | Müşteri, prefix, port profili, rate limit, whitelist/blacklist yönetimi; istatistik ve drop log görüntüleme. |
| **ClickHouse** (isteğe bağlı) | Drop log’larının saklanması ve analiz için kullanılması. |

Trafik önce XDP’den geçer; geçen paketler çıkış arayüzüne iletilir, drop edilenler loglanır (ve istenirse InfluxDB ile metriklenir).

---

## Kimler İçin?

- **Hosting / veri merkezi** tarafında müşteri bazlı DDoS koruma vermek isteyenler
- **Edge / scrubbing** katmanında XDP ile yüksek PPS kapasitesi hedefleyenler
- Kuralları **API ve Web UI** ile yönetmek isteyen ekipler

---

## Özellikler (Özet)

- XDP/eBPF ile kernel seviyesinde filtreleme, düşük latency
- Müşteri ve prefix bazlı **port profilleri** (TCP/UDP listesi, ICMP, bypass profili)
- **Rate limiting**: kaynak IP başına PPS, BPS, SYN (ör. 1 sn pencere)
- **GeoIP / ASN** whitelist veya blacklist
- **SYN flood** (SYN cookie challenge), **UDP amplification**, **carpet bomb** tespiti
- **Honeypot** subnet’leri ve otomatik blacklist
- **Global ve müşteri whitelist/blacklist**, RTBH entegrasyonu
- REST API ve Web arayüzü, ClickHouse drop log, isteğe bağlı InfluxDB/uyarılar

---

## Teknik Gereksinimler

- Linux kernel 5.8+ (XDP/eBPF destekli)
- XDP destekli NIC (ör. Mellanox ConnectX serisi)
- Go 1.21+, Clang/LLVM (BPF derleme)
- İsteğe bağlı: ClickHouse, InfluxDB

---




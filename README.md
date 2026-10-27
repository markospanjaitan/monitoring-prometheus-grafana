# Pemantauan Terpusat dengan Prometheus dan Grafana

Memantau CPU, RAM, dan disk beberapa mesin Linux dari satu dashboard, lewat jaringan Tailscale.

## Susunan

    laptop (browser)
         |  Tailscale
         v
    VM_3 (Ubuntu): Grafana --> Prometheus --> node-exporter (VM_3 sendiri)
                                    |
                                    +-------> node-exporter di Kali (via Tailscale)

## Langkah

1. Pasang Prometheus dan agennya: `sudo apt install -y prometheus prometheus-node-exporter`
2. Kunci keduanya ke `127.0.0.1` lewat `ARGS` di `/etc/default/prometheus` dan `/etc/default/prometheus-node-exporter`.
3. Pasang Grafana dari repositori resmi, lalu batasi `http_addr` ke alamat Tailscale VM_3.
4. Di Grafana, tambah data source Prometheus (`http://localhost:9090`) dan impor dashboard ID `1860` (Node Exporter Full).
5. Di mesin lain, pasang `prometheus-node-exporter`, kunci ke alamat Tailscale mesin itu, lalu daftarkan di `prometheus/prometheus.yml`.

## Konfigurasi

`prometheus/prometheus.yml` (alamat Tailscale disamarkan menjadi `100.x.x.x`).

## Keamanan

- Prometheus dan agen hanya bisa diakses dari mesinnya sendiri.
- Grafana hanya mendengarkan di alamat Tailscale, tidak terbuka ke jaringan sekitar.
- Tidak ada port yang dibuka ke internet.

## Hasil uji

- Saat VM Kali mati, target `mesin-lain` berstatus `down` di Prometheus. Pemantauan berhasil menangkap mesin yang mati.
- Screenshot: menyusul.

## Masalah yang ditemui

- Dashboard kosong setelah mesin kedua ditambahkan, karena pilihan Job dan Nodename di bagian atas dashboard tertukar.
- Setelah VM dinyalakan ulang, Grafana butuh waktu sebelum mendengarkan di port 3000.

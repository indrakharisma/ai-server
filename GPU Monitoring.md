# AI Server – GPU Monitoring & Audit (Admin Guide)

## Overview
Dokumen ini ditujukan untuk administrator dalam memonitor, mengaudit, dan mengendalikan penggunaan GPU pada AI server (multi-user environment).

Tujuan utama:
- Mendeteksi penggunaan GPU secara real-time
- Mengidentifikasi user dan proses yang menggunakan GPU
- Mencegah penyalahgunaan resource (GPU abuse)
- Menyediakan logging historis untuk analisis

---

## 1. Real-Time GPU Monitoring

### Basic Monitoring
```bash
watch -n 1 nvidia-smi
```

Perhatikan:
- `GPU-Util` → tingkat penggunaan GPU (%)
- `Memory-Usage` → VRAM yang digunakan
- `Processes` → proses aktif di GPU

---

### Process-Level Monitoring
```bash
nvidia-smi pmon -i 0
```

Interpretasi:
- `sm` → compute usage
- `mem` → memory usage
- nilai >80 → indikasi workload berat

---

## 2. GPU Audit Script (Recommended)

Buat file:
```bash
nano /home/aiserver/gpu_audit.sh
```

Isi:

```bash
#!/bin/bash

LOG_DIR="/home/aiserver/logs"
mkdir -p $LOG_DIR

USAGE_LOG="$LOG_DIR/gpu_usage_detailed.log"
ALERT_LOG="$LOG_DIR/gpu_alert.log"

TIMESTAMP=$(date "+%Y-%m-%d %H:%M:%S")

GPU_UTIL=$(nvidia-smi --query-gpu=utilization.gpu --format=csv,noheader,nounits)
GPU_MEM=$(nvidia-smi --query-gpu=memory.used --format=csv,noheader,nounits)

echo "[$TIMESTAMP] GPU_UTIL=${GPU_UTIL}% MEM=${GPU_MEM}MiB" >> $USAGE_LOG

nvidia-smi --query-compute-apps=pid,process_name,used_memory --format=csv,noheader | while IFS=',' read pid pname mem
do
    pid=$(echo $pid | xargs)
    pname=$(echo $pname | xargs)
    mem=$(echo $mem | xargs)

    user=$(ps -o user= -p $pid 2>/dev/null)

    echo "[$TIMESTAMP] PID=$pid USER=$user PROC=$pname MEM=$mem" >> $USAGE_LOG

    if [ "$GPU_UTIL" -gt 80 ]; then
        echo "[$TIMESTAMP] WARNING: High GPU usage by $user ($pname) PID=$pid MEM=$mem GPU=${GPU_UTIL}%" >> $ALERT_LOG
    fi
done
```

Aktifkan:
```bash
chmod +x /home/aiserver/gpu_audit.sh
```

---

## 3. Automation (Cron Job)

Edit cron:
```bash
crontab -e
```

Tambahkan:
```bash
* * * * * /home/aiserver/gpu_audit.sh
```

Penjelasan:
- Script dijalankan setiap 1 menit
- Semua aktivitas GPU akan dicatat

---

## 4. Log Monitoring

### Lihat aktivitas GPU
```bash
tail -f /home/aiserver/logs/gpu_usage_detailed.log
```

### Lihat alert penggunaan tinggi
```bash
cat /home/aiserver/logs/gpu_alert.log
```

---

## 5. Interpretasi Log

Contoh:
```
[2026-04-03 11:30:01] GPU_UTIL=95% MEM=12000MiB
[2026-04-03 11:30:01] PID=12345 USER=aiueo PROC=python MEM=8000 MiB
```

Makna:
- GPU digunakan berat (95%)
- oleh user `aiueo`
- proses `python`
- menggunakan 8GB VRAM

---

## 6. Indikator Penyalahgunaan (Abuse Detection)

### 🔴 High Risk
- GPU >90% dalam waktu lama (>30 menit)
- user tidak dikenal
- proses mencurigakan:
  - `xmrig`
  - binary random (`./a.out`)
- GPU usage tinggi tanpa proses terdeteksi

---

### 🟡 Moderate Risk
- penggunaan GPU di luar jam kerja
- penggunaan VRAM sangat besar tanpa konteks jelas
- multiple user tanpa koordinasi

---

### 🟢 Normal
- training model (PyTorch, TensorFlow)
- inference LLM (Ollama, vLLM)
- penggunaan intermiten

---

## 7. Investigasi Lanjutan

### Identifikasi proses
```bash
ps -fp <PID>
```

### Kill proses
```bash
kill -9 <PID>
```

### Monitor GPU live
```bash
watch -n 1 nvidia-smi
```

---

## 8. Best Practices (Admin)

- Gunakan isolasi user (Docker / container)
- Batasi akses GPU (CUDA_VISIBLE_DEVICES)
- Hindari direktori `777`
- Audit log secara berkala
- Bersihkan environment dan model yang tidak terpakai
- Monitor disk usage (model LLM bisa >10GB per file)

---

## 9. Security Notes

- Jangan expose service GPU ke publik tanpa kontrol
- Pastikan hanya user authorized yang bisa akses server
- Monitor `/var/log/auth.log` untuk login mencurigakan
- Gunakan firewall jika server terhubung ke jaringan publik

---

## 10. Summary

Dengan setup ini, administrator dapat:
- Melihat siapa yang menggunakan GPU
- Mendeteksi penggunaan berlebih
- Melacak histori penggunaan GPU
- Mengambil tindakan cepat jika terjadi misuse

Status ideal:
> GPU usage terkontrol, transparan, dan dapat diaudit setiap saat
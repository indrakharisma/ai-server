# AI Server — Research Infrastructure

![GPU](https://img.shields.io/badge/GPU-RTX%203090-76b900?style=flat-square&logo=nvidia)
![VRAM](https://img.shields.io/badge/VRAM-24GB-76b900?style=flat-square)
![CUDA](https://img.shields.io/badge/CUDA-13.0-76b900?style=flat-square)
![Ollama](https://img.shields.io/badge/Ollama-Online-brightgreen?style=flat-square)
![Python](https://img.shields.io/badge/Python-3.12-blue?style=flat-square&logo=python)

---

## Daftar Isi

1. [Tentang Server Ini](#1-tentang-server-ini)
2. [Layanan yang Tersedia](#2-layanan-yang-tersedia)
3. [Model LLM Tersedia](#3-model-llm-tersedia)
4. [Cara Request Akses](#4-cara-request-akses)
5. [Mulai Menggunakan](#5-mulai-menggunakan)
6. [ML Training Environment](#6-ml-training-environment)
7. [Storage Layout](#7-storage-layout)
8. [Jadwal & Koordinasi GPU](#8-jadwal--koordinasi-gpu)
9. [Panduan Dosen — Manajemen Akses Mahasiswa](#9-panduan-dosen--manajemen-akses-mahasiswa)
10. [Kontak](#10-kontak)

---

## 1. Tentang Server Ini

Server ini adalah infrastruktur riset milik **Dr. Indra Kharisma Raharjana, S.Kom., M.T.** yang dikelola untuk mendukung mahasiswa bimbingan dan Research Group **Center for Artificial Intelligence and Information Systems (CAIS)**, Universitas Airlangga — dalam melakukan penelitian dan publikasi ilmiah di bidang kecerdasan buatan.

GPU **NVIDIA RTX 3090 24GB VRAM** tersedia untuk training ML/AI dan inferensi LLM secara lokal — gratis, privat, tanpa biaya API cloud. Cocok untuk skripsi, tesis, maupun proyek riset yang butuh compute serius.

**Spesifikasi Server:**

| Komponen | Detail |
|----------|--------|
| Host | Proxmox VE (prox0) |
| CPU | AMD Ryzen 5 9600X (6C/12T) |
| RAM | 64GB DDR5 |
| GPU | NVIDIA RTX 3090 24GB VRAM |
| Storage OS/VM | 512GB NVMe |
| Storage Data | 1TB NVMe |
| OS (VM) | Ubuntu 24.04 LTS |

**Profil riset dosen:**
[ORCID](https://orcid.org/0000-0002-0622-3374) · [Scopus](https://www.scopus.com/authid/detail.uri?authorId=57202163318) · [SINTA](https://sinta.kemdiktisaintek.go.id/authors/profile/5980740) · [Google Scholar](https://scholar.google.com/citations?user=PaJeSJEAAAAJ)

---

## 2. Layanan yang Tersedia

| Layanan | URL | Fungsi |
|---------|-----|--------|
| **Ollama API** | [ollama.ebruar.my.id](https://ollama.ebruar.my.id) | LLM inference endpoint |
| **Open WebUI** | [chat.ebruar.my.id](https://chat.ebruar.my.id) | Chat interface (minta akses ke dosen) |
| **Carina** | [carina.ebruar.my.id](https://carina.ebruar.my.id) | JISEBI manuscript desk-review AI |
| **Coolify** | [coolify.ebruar.my.id](https://coolify.ebruar.my.id) | Web hosting platform (admin only) |

> Semua service dilindungi Cloudflare Access. Pastikan Anda memiliki **Cloudflare Service Token** yang diberikan dosen untuk mengakses Ollama API secara programatik.

---

## 3. Model LLM Tersedia

| Model | Ukuran | Kegunaan | Estimasi Speed |
|-------|--------|----------|----------------|
| `qwen3:14b` | ~9GB | General purpose, cepat — cocok untuk prototyping, eksperimen awal, dan iterasi cepat | ~50 tok/s |
| `qwen3:32b` | ~19GB | Kualitas lebih baik — cocok untuk hasil akhir penelitian, evaluasi, atau task kompleks | ~22 tok/s |

> Model akan terus diupdate sesuai kebutuhan. Cek ketersediaan sebelum digunakan atau tanyakan ke dosen.

**Cek model yang tersedia:**

```bash
curl https://ollama.ebruar.my.id/api/tags \
  -H "CF-Access-Client-Id: MINTA_KE_DOSEN" \
  -H "CF-Access-Client-Secret: MINTA_KE_DOSEN"
```

---

## 4. Cara Request Akses

Akses server tidak bersifat publik. Kirim email ke **indra.kharisma@fst.unair.ac.id** dengan subject:

```
[AI Server] Request Akses - NAMA - NIM
```

Sertakan di isi email:
- Nama lengkap & NIM
- Judul / topik penelitian
- Kebutuhan resource (LLM API / GPU training / keduanya)
- Estimasi durasi penggunaan

Akses akan diberikan dalam **1–2 hari kerja**. Dosen akan mengirimkan kredensial SSH dan Cloudflare Service Token.

---

## 5. Mulai Menggunakan

### Akses SSH via Cloudflare

**Install `cloudflared` di laptop:**

```bash
# macOS
brew install cloudflare/cloudflare/cloudflared

# Windows
winget install Cloudflare.cloudflared

# Linux — lihat: https://developers.cloudflare.com/cloudflared/get-started/
```

**Tambahkan ke SSH config:**

- macOS / Linux: `~/.ssh/config`
- Windows: `C:\Users\NAMA_ANDA\.ssh\config` (buat folder `.ssh` jika belum ada)

```
Host ai-server
    HostName ssh-ai.ebruar.my.id
    User USERNAME_KAMU
    ProxyCommand cloudflared access ssh --hostname %h
```

**Connect:**

```bash
ssh ai-server
```

Jika pertama kali login, `cloudflared` akan membuka browser untuk autentikasi Cloudflare Access.

---

### Akses Ollama API dari kode Python

```python
from openai import OpenAI

client = OpenAI(
    base_url="https://ollama.ebruar.my.id/v1",
    api_key="ollama",
    default_headers={
        "CF-Access-Client-Id": "MINTA_KE_DOSEN",
        "CF-Access-Client-Secret": "MINTA_KE_DOSEN"
    }
)

response = client.chat.completions.create(
    model="qwen3:14b",
    messages=[{"role": "user", "content": "Halo!"}]
)
print(response.choices[0].message.content)
```

> **Tips keamanan:** Jangan hardcode token di kode. Gunakan environment variable atau file `.env` yang tidak di-commit ke GitHub.

```python
import os
from dotenv import load_dotenv

load_dotenv()

default_headers = {
    "CF-Access-Client-Id": os.getenv("CF_CLIENT_ID"),
    "CF-Access-Client-Secret": os.getenv("CF_CLIENT_SECRET")
}
```

---

### Tunnel Ollama ke lokal (untuk development)

Berguna jika ingin menggunakan Ollama seolah-olah berjalan di laptop sendiri.

```bash
# Terminal 1 — biarkan tetap berjalan
ssh -L 11434:localhost:11434 ai-server

# Terminal 2 — gunakan di kode
LOCAL_LLM_BASE_URL=http://localhost:11434/v1
```

---

## 6. ML Training Environment

### Aktivasi environment

```bash
# Gunakan ml-env yang sudah tersedia (PyTorch + semua library siap pakai)
source ~/ml-env/bin/activate

# Atau aktifkan conda environment milik sendiri
conda activate /mnt/model-storage/research/NAMA_KAMU/env-penelitian
```

### Buat conda environment sendiri

Mahasiswa dapat membuat environment sendiri tanpa perlu izin admin — cukup simpan di folder research masing-masing.

```bash
conda create -p /mnt/model-storage/research/NAMA_KAMU/env python=3.11 -y
conda activate /mnt/model-storage/research/NAMA_KAMU/env

# Install PyTorch via pip agar dapat versi CUDA yang benar
pip install torch --index-url https://download.pytorch.org/whl/cu128
pip install transformers datasets scikit-learn pandas matplotlib

# Lihat semua environment
conda env list
```

> Gunakan `conda` untuk manage Python version, `pip` untuk packages ML.

### JupyterLab

```bash
# Di server (setelah SSH)
source ~/ml-env/bin/activate
jupyter lab --no-browser --port=8888 --ip=127.0.0.1

# Di terminal laptop — buka tunnel
ssh -L 8888:localhost:8888 ai-server

# Buka di browser
# http://localhost:8888
```

Pilih kernel: **Python (ml-env)**

### Opsi Interface untuk Training

#### Opsi 1 — Terminal SSH (paling ringan, direkomendasikan)
Cocok untuk: menjalankan training script, monitoring, quick test.

```bash
ssh ai-server
source ~/ml-env/bin/activate
cd /mnt/model-storage/research/NAMA_KAMU

python3 train.py

# Untuk training lama, gunakan screen agar tidak putus saat disconnect
screen -S training
python3 train.py
# Ctrl+A lalu D untuk detach
# screen -r training untuk reattach
```

#### Opsi 2 — JupyterLab via browser
Cocok untuk: eksplorasi data, prototyping, visualisasi hasil. Lihat sub-section JupyterLab di atas.

#### Opsi 3 — VS Code Remote SSH
Bisa digunakan tapi **koneksi terasa lambat** karena semua traffic lewat Cloudflare tunnel. Lebih cocok digunakan saat berada di jaringan lokal yang sama dengan server (di kampus).

Setup:
1. Install extension **Remote - SSH** (Microsoft) di VS Code
2. Pastikan `~/.ssh/config` sudah dikonfigurasi (lihat section SSH Access)
3. `F1` → `Remote-SSH: Connect to Host` → `ai-server`
4. Install extension **Jupyter** di remote
5. Register kernel: `python -m ipykernel install --user --name=ml-env --display-name="Python (ml-env)"`

**Catatan:** Jika VS Code Server belum terinstall otomatis, install manual:
```bash
wget -O vscode-server.tar.gz \
  "https://update.code.visualstudio.com/commit:COMMIT_ID/server-linux-x64/stable"
mkdir -p ~/.vscode-server/bin/COMMIT_ID
tar -xzf vscode-server.tar.gz -C ~/.vscode-server/bin/COMMIT_ID --strip-components=1
rm vscode-server.tar.gz
```
Ganti `COMMIT_ID` dengan versi VS Code yang digunakan (lihat di VS Code: `Help → About`).

### Library yang tersedia di ml-env

| Library | Versi | Kegunaan |
|---------|-------|----------|
| PyTorch | 2.11.0+cu128 | Deep learning framework |
| Transformers | 5.3.0 | Pre-trained models (HuggingFace) |
| PEFT | 0.18.1 | Fine-tuning efisien (LoRA, QLoRA) |
| TRL | 0.24.0 | Reinforcement learning dari feedback |
| Accelerate | latest | Multi-GPU training |
| Unsloth | latest | Fine-tuning LLM 2x lebih cepat |
| scikit-learn | latest | Classical ML |
| OpenCV | 4.13.0 | Computer vision |
| Datasets | latest | HuggingFace datasets |
| WandB | latest | Experiment tracking |
| TensorBoard | latest | Training visualization |
| JupyterLab | latest | Notebook interface |

### Contoh Training Sederhana (Fine-tuning dengan LoRA)

```python
from transformers import AutoModelForCausalLM, AutoTokenizer
from peft import LoraConfig, get_peft_model
import torch

model_name = "Qwen/Qwen2.5-1.5B"
model = AutoModelForCausalLM.from_pretrained(
    model_name,
    torch_dtype=torch.float16,
    device_map="cuda"
)

lora_config = LoraConfig(
    r=16,
    lora_alpha=32,
    target_modules=["q_proj", "v_proj"],
    lora_dropout=0.05,
    task_type="CAUSAL_LM"
)

model = get_peft_model(model, lora_config)
model.print_trainable_parameters()
```

---

## 7. Storage Layout

```
/mnt/model-storage/          (1 TB)
├── ollama/                  ← model LLM — JANGAN diubah manual
├── research/
│   └── nama_kamu/           ← folder kerjamu (maks 50 GB)
│       ├── env/             ← conda environment
│       └── notebooks/       ← jupyter notebooks
└── datasets/                ← dataset publik (baca saja, minta dosen untuk tambah)
```

**Aturan storage:**

- Model LLM **hanya** boleh ada di `/mnt/model-storage/ollama` — jangan download model ke folder lain
- Setiap mahasiswa **hanya** boleh menggunakan folder miliknya sendiri
- Dataset yang ingin dishare ke mahasiswa lain, minta dosen untuk taruh di `/mnt/model-storage/datasets/`
- Maksimal storage per mahasiswa: **50GB** — hubungi dosen jika butuh lebih
- Hapus checkpoint model yang sudah tidak dibutuhkan untuk menjaga ketersediaan storage

---

## 8. Jadwal & Koordinasi GPU

GPU RTX 3090 digunakan bersama — harap koordinasi agar tidak bentrok.

| Mode | Kondisi | VRAM |
|------|---------|------|
| Ollama aktif | Model LLM di-load | Hingga 19GB terpakai |
| Training aktif | Ollama di-unload manual | GPU bebas untuk training |

**Aturan penggunaan:**

- Cek dulu apakah ada yang sedang training: `nvidia-smi`
- Kalau VRAM penuh, tanya dulu di grup WhatsApp sebelum mulai
- Kalau mau training lama (>1 jam), kasih info di grup dulu
- Jangan tinggalkan proses idle yang memakan VRAM

> Minta link grup WhatsApp ke dosen saat request akses. Wajib gabung sebelum mulai menggunakan GPU untuk training.

**Cek status GPU:**

```bash
nvidia-smi            # lihat VRAM usage dan proses aktif
nvidia-smi pmon -s u  # lihat proses per user
```

**Unload Ollama sebelum training besar:**

```bash
curl -X POST http://localhost:11434/api/generate \
  -d '{"model": "qwen3:32b", "keep_alive": 0}'
curl -X POST http://localhost:11434/api/generate \
  -d '{"model": "qwen3:14b", "keep_alive": 0}'

# Verifikasi VRAM sudah bebas
nvidia-smi
```

Setelah selesai training, Ollama akan otomatis me-load model kembali saat ada request berikutnya.

**Troubleshooting umum:**

| Masalah | Kemungkinan Penyebab | Solusi |
|---------|----------------------|--------|
| SSH timeout / tidak bisa konek | `cloudflared` belum login atau sesi expired | Jalankan `cloudflared access login ssh-ai.ebruar.my.id` lalu coba SSH lagi |
| `CUDA out of memory` saat training | Model terlalu besar atau VRAM belum bebas | Unload Ollama dulu, kurangi batch size, atau koordinasi jadwal |
| Ollama tidak merespons / lambat | Ada training yang sedang berjalan | Tunggu beberapa menit atau tanya di grup |
| `401 Unauthorized` di API | Cloudflare Service Token salah atau expired | Minta token baru ke dosen |
| JupyterLab tidak bisa dibuka | Port forwarding tidak aktif | Pastikan `ssh -L 8888:localhost:8888 ai-server` sudah berjalan |
| Storage penuh | Melebihi kuota 50GB | Hapus file yang tidak dibutuhkan, atau hubungi dosen |

---

## 9. Panduan Dosen — Manajemen Akses Mahasiswa

### Tambah mahasiswa baru

```bash
# Format: sudo add-student.sh <username> <nama_lengkap> <email>
sudo add-student.sh mhs "Mahasiswa Pejuang Skripsi" mhs@fst.unair.ac.id
```

Script otomatis akan:
- Membuat user account
- Generate password temporary
- Membuat folder `/mnt/model-storage/research/<username>`
- Setup conda di bashrc
- Menampilkan info yang perlu dikirim ke mahasiswa

### Kirim info ke mahasiswa (output script)

```
Host     : ssh-ai.ebruar.my.id
Username : <username>
Password : <password dari output script>
Docs     : https://github.com/indrakharisma/ai-server

Password harus diganti saat login pertama.
Baca dokumentasi sebelum mulai.
```

### Nonaktifkan akses mahasiswa (lulus/selesai penelitian)

```bash
# Lock account (data tetap aman)
sudo usermod -L <username>

# Atau hapus permanent (data ikut terhapus)
sudo userdel -r <username>
sudo rm -rf /mnt/model-storage/research/<username>
```

### Monitoring

```bash
who                                         # siapa yang sedang login
nvidia-smi pmon -s u                        # siapa yang pakai GPU
du -sh /mnt/model-storage/research/*        # storage per mahasiswa
cat /etc/passwd | grep /home | cut -d: -f1  # daftar semua user
```

### Reset password mahasiswa (lupa password)

```bash
sudo passwd <username>
sudo chage -d 0 <username>  # force ganti password saat login berikutnya
```

---

## 10. Kontak

**Dr. Indra Kharisma Raharjana, S.Kom., M.T.**
Dosen Program Studi Sistem Informasi, Fakultas Sains dan Teknologi, Universitas Airlangga
Ketua Center for Artificial Intelligence and Information Systems (CAIS), Universitas Airlangga
Editor-in-Chief — Journal of Information Systems Engineering and Business Intelligence (JISEBI)

Email: [indra.kharisma@fst.unair.ac.id](mailto:indra.kharisma@fst.unair.ac.id)

> Sertakan nama, NIM, dan keperluan Anda di email agar permintaan dapat diproses lebih cepat.

---

<p align="center">
  Dikelola oleh Research Group CAIS — Universitas Airlangga<br>
  <sub>Infrastruktur ini disediakan untuk keperluan penelitian akademik.</sub>
</p>

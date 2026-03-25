# AI Server — Research Infrastructure

![GPU](https://img.shields.io/badge/GPU-RTX%203090-76b900?style=flat-square&logo=nvidia)
![VRAM](https://img.shields.io/badge/VRAM-24GB-76b900?style=flat-square)
![Ollama](https://img.shields.io/badge/Ollama-Online-brightgreen?style=flat-square)
![Models](https://img.shields.io/badge/Models-2%20tersedia-blue?style=flat-square)

Server AI milik dosen untuk mendukung penelitian mahasiswa di bidang kecerdasan buatan dan pemrosesan bahasa alami.

---

## Daftar Isi

1. [Overview](#1-overview)
2. [Available Services](#2-available-services)
3. [Storage Layout](#3-storage-layout)
4. [Akses Ollama API](#4-akses-ollama-api)
5. [Available Models](#5-available-models)
6. [ML Training Environment](#6-ml-training-environment)
7. [Storage & Etika Penggunaan](#7-storage--etika-penggunaan)
8. [GPU Scheduler](#8-gpu-scheduler-ollama--training)
9. [Cara Request Akses](#9-cara-request-akses)
10. [Troubleshooting](#10-troubleshooting)
11. [Kontak](#11-kontak)

---

## 1. Overview

Repository ini mendokumentasikan infrastruktur AI Server yang disediakan untuk mendukung **penelitian mahasiswa** di bawah bimbingan dosen, khususnya di lingkungan Research Group **Center for Artificial Intelligence and Information Systems (CAIS)**, Universitas Airlangga.

Server ini menyediakan:
- **LLM Inference** via Ollama — untuk eksperimen NLP, chatbot, RAG, dan sejenisnya
- **GPU Training** — untuk fine-tuning model atau training dari scratch
- **Web Hosting** via Coolify — untuk deploy prototype atau aplikasi penelitian

**Siapa yang boleh menggunakan?**
Mahasiswa yang sedang mengerjakan penelitian (skripsi, tesis, atau proyek riset) di bawah bimbingan atau persetujuan Dr. Indra Kharisma Raharjana. Akses diberikan atas permintaan dan tidak bersifat publik.

**Spesifikasi Server:**

| Komponen | Detail |
|----------|--------|
| Host | Proxmox VE (prox0) |
| CPU | AMD Ryzen 5 9600X (6C/12T) |
| RAM | 64GB DDR5 |
| GPU | NVIDIA RTX 3090 24GB VRAM |
| Storage OS/VM | 512GB NVMe |
| Storage Model | 2TB NVMe (1.37TB terpakai) |
| OS (VM) | Ubuntu 24.04 LTS |

---

## 2. Available Services

| Service | URL | Fungsi | Akses |
|---------|-----|--------|-------|
| **Ollama** | [ollama.ebruar.my.id](https://ollama.ebruar.my.id) | LLM inference API (OpenAI-compatible) | Mahasiswa dengan Cloudflare token |
| **Open WebUI** | [chat.ebruar.my.id](https://chat.ebruar.my.id) | Chat interface untuk berinteraksi dengan LLM | Mahasiswa dengan akun |
| **Coolify** | [coolify.ebruar.my.id](https://coolify.ebruar.my.id) | Platform hosting aplikasi/prototype | Atas permintaan khusus |
| **Carina** | [carina.ebruar.my.id](https://carina.ebruar.my.id) | JISEBI manuscript checker | Terbuka untuk pengguna JISEBI |

> Semua service dilindungi Cloudflare Access. Pastikan Anda memiliki **Cloudflare Service Token** yang diberikan dosen untuk mengakses Ollama API secara programatik.

---

## 3. Storage Layout

Seluruh penyimpanan data penelitian dan model berada di disk `/mnt/model-storage` (2TB NVMe):

```
/mnt/model-storage/          (1.37 TB total)
├── ollama/                  ← semua model LLM (dikelola otomatis oleh Ollama)
│   └── models/
│       ├── qwen3:14b        (~9 GB)
│       └── qwen3:32b        (~19 GB)
├── research/                ← folder kerja mahasiswa
│   ├── nama_mahasiswa_a/
│   ├── nama_mahasiswa_b/
│   └── ...
└── datasets/                ← dataset publik yang bisa dishare
```

**Aturan storage:**

- Model LLM **hanya** boleh ada di `/mnt/model-storage/ollama` — jangan download model ke folder lain
- Setiap mahasiswa **hanya** boleh menggunakan folder miliknya sendiri di `/mnt/model-storage/research/`
- Dataset yang ingin dishare ke mahasiswa lain taruh di `/mnt/model-storage/datasets/`
- Maksimal storage per mahasiswa: **50GB** (hubungi dosen jika butuh lebih)

---

## 4. Akses Ollama API

Ollama API kompatibel dengan format OpenAI. Anda bisa menggunakannya langsung dari kode Python, curl, atau LangChain dalam penelitian.

> **Catatan:** Header `CF-Access-Client-Id` dan `CF-Access-Client-Secret` wajib disertakan di setiap request. Minta token ini ke dosen.

### Via Python (OpenAI-compatible)

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
    messages=[{"role": "user", "content": "Hello"}]
)
print(response.choices[0].message.content)
```

### Via curl

```bash
curl https://ollama.ebruar.my.id/api/chat \
  -H "CF-Access-Client-Id: MINTA_KE_DOSEN" \
  -H "CF-Access-Client-Secret: MINTA_KE_DOSEN" \
  -d '{"model": "qwen3:14b", "messages": [{"role": "user", "content": "Hello"}]}'
```

### Via LangChain

```python
from langchain_ollama import OllamaLLM

llm = OllamaLLM(
    model="qwen3:14b",
    base_url="https://ollama.ebruar.my.id",
    headers={
        "CF-Access-Client-Id": "MINTA_KE_DOSEN",
        "CF-Access-Client-Secret": "MINTA_KE_DOSEN"
    }
)

response = llm.invoke("Jelaskan apa itu RAG dalam NLP.")
print(response)
```

> **Tips keamanan:** Jangan hardcode token Anda langsung di kode. Gunakan environment variable atau file `.env` yang tidak di-commit ke GitHub.

```python
import os
from dotenv import load_dotenv

load_dotenv()

default_headers={
    "CF-Access-Client-Id": os.getenv("CF_CLIENT_ID"),
    "CF-Access-Client-Secret": os.getenv("CF_CLIENT_SECRET")
}
```

---

## 5. Available Models

| Model | Ukuran | Kegunaan | Estimasi Speed |
|-------|--------|----------|----------------|
| `qwen3:14b` | ~9GB | General purpose, cepat — cocok untuk prototyping, eksperimen awal, dan iterasi cepat | ~50 tok/s |
| `qwen3:32b` | ~19GB | Kualitas lebih baik — cocok untuk hasil akhir penelitian, evaluasi, atau task kompleks | ~22 tok/s |

> `qwen3:32b` sedang dalam proses download. Cek ketersediaan sebelum digunakan atau tanyakan ke dosen.

**Cek model yang tersedia:**

```bash
curl https://ollama.ebruar.my.id/api/tags \
  -H "CF-Access-Client-Id: MINTA_KE_DOSEN" \
  -H "CF-Access-Client-Secret: MINTA_KE_DOSEN"
```

---

## 6. ML Training Environment

Untuk training model (fine-tuning, transfer learning, dsb.), mahasiswa dapat mengakses GPU server secara langsung via SSH melalui tunnel Cloudflare.

### Prasyarat

Install `cloudflared` di laptop Anda:

```bash
# macOS
brew install cloudflare/cloudflare/cloudflared

# Linux (Debian/Ubuntu)
wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb
sudo dpkg -i cloudflared-linux-amd64.deb

# Windows — download installer dari:
# https://github.com/cloudflare/cloudflared/releases/latest
```

### Konfigurasi SSH

Tambahkan ke file `~/.ssh/config` di laptop Anda:

```
Host ai-server
    HostName ssh-ai.ebruar.my.id
    User aiserver
    ProxyCommand cloudflared access ssh --hostname %h
```

### Koneksi SSH

```bash
ssh ai-server
```

Jika pertama kali login, `cloudflared` akan membuka browser untuk autentikasi Cloudflare Access.

### Aktivasi Environment

```bash
source ~/ml-env/bin/activate
```

### Library yang Tersedia

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

### Folder Kerja

```bash
# Ganti NAMA_ANDA dengan nama folder yang diberikan dosen
cd /mnt/model-storage/research/NAMA_ANDA
```

### Jalankan JupyterLab

```bash
~/scripts/start-jupyter.sh
```

### Akses JupyterLab dari Laptop (SSH Tunnel)

Buka terminal **baru** di laptop Anda:

```bash
ssh -L 8888:localhost:8888 ai-server
```

Kemudian buka di browser: `http://localhost:8888`

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

## 7. Storage & Etika Penggunaan

Setiap mahasiswa mendapatkan folder pribadi di `/mnt/model-storage/research/NAMA_MAHASISWA/`. Folder ini adalah ruang kerja Anda — simpan dataset, checkpoint model, dan hasil eksperimen di sini.

### Aturan Penggunaan

- **Jangan hapus atau modifikasi file milik mahasiswa lain** di luar folder Anda.
- **Kuota storage per mahasiswa: 50GB.** Jika membutuhkan lebih, hubungi dosen terlebih dahulu.
- **GPU bersifat shared.** Jika ada mahasiswa lain yang sedang training, koordinasikan jadwal agar tidak bentrok.
- **Jangan menjalankan proses yang memakan seluruh VRAM (24GB)** tanpa koordinasi terlebih dahulu.
- Hapus checkpoint model yang sudah tidak dibutuhkan untuk menjaga ketersediaan storage.

---

## 8. GPU Scheduler (Ollama & Training)

GPU RTX 3090 digunakan secara bergantian antara **Ollama inference** dan **ML training**. Keduanya tidak bisa berjalan bersamaan secara optimal karena berbagi VRAM.

| Mode | Kondisi | VRAM |
|------|---------|------|
| Ollama aktif | Model LLM di-load | Hingga 19GB terpakai |
| Training aktif | Ollama di-unload manual | GPU bebas untuk training |

### Cek Status GPU Sebelum Training

Jalankan perintah berikut setelah SSH ke server untuk memastikan GPU siap digunakan:

```bash
# Cek VRAM usage saat ini
nvidia-smi

# Cek apakah ada model Ollama yang sedang di-load
curl http://localhost:11434/api/ps

# Unload semua model Ollama sebelum training
curl -X POST http://localhost:11434/api/generate \
  -d '{"model": "qwen3:32b", "keep_alive": 0}'
curl -X POST http://localhost:11434/api/generate \
  -d '{"model": "qwen3:14b", "keep_alive": 0}'

# Verifikasi VRAM sudah bebas
nvidia-smi
```

> Perintah di atas dijalankan **di dalam server** (setelah SSH), bukan dari laptop.

Setelah selesai training, Ollama akan otomatis me-load model kembali saat ada request berikutnya.

---

## 9. Cara Request Akses

Akses server tidak bersifat publik. Mahasiswa yang membutuhkan akses harus mengajukan permintaan ke dosen.

**Langkah-langkah:**

1. Kirim email ke dosen (lihat bagian [Kontak](#11-kontak))
2. Sertakan informasi berikut di email:
   - Nama lengkap & NIM
   - Judul/topik penelitian
   - Jenis akses yang dibutuhkan (API only / SSH training / keduanya)
   - Estimasi kebutuhan resource (storage, durasi penggunaan GPU)
3. Dosen akan memberikan:
   - Akun SSH (jika diperlukan)
   - **Cloudflare Service Token** (`CF-Access-Client-Id` & `CF-Access-Client-Secret`)
   - Folder penelitian di server

---

## 10. Troubleshooting

| Masalah | Kemungkinan Penyebab | Solusi |
|---------|----------------------|--------|
| SSH timeout / tidak bisa konek | `cloudflared` belum login atau sesi expired | Jalankan `cloudflared access login ssh-ai.ebruar.my.id` lalu coba SSH lagi |
| `CUDA out of memory` saat training | Model terlalu besar untuk sisa VRAM | Gunakan batch size lebih kecil, atau unload Ollama dulu, atau koordinasi waktu dengan mahasiswa lain |
| Ollama tidak merespons / lambat | Model sedang di-unload atau ada training berjalan | Tunggu beberapa menit atau hubungi dosen untuk konfirmasi status server |
| `401 Unauthorized` di API | Cloudflare Service Token salah atau expired | Minta token baru ke dosen |
| JupyterLab tidak bisa dibuka | Port forwarding tidak aktif | Pastikan perintah `ssh -L 8888:localhost:8888 ai-server` sudah dijalankan di terminal terpisah |
| Storage penuh | Melebihi kuota 50GB | Hapus file yang tidak dibutuhkan, atau hubungi dosen untuk penambahan kuota |

---

## 11. Kontak

Untuk request akses, pertanyaan teknis, atau kendala penggunaan server:

**Dr. Indra Kharisma Raharjana**
Ketua Research Group — Center for Artificial Intelligence and Information Systems (CAIS)
Dosen Program Studi S1 Sistem Informasi, Universitas Airlangga

📧 Email: [indra.kharisma@fst.unair.ac.id](mailto:indra.kharisma@fst.unair.ac.id)

> Sertakan nama, NIM, dan keperluan Anda di email agar permintaan dapat diproses lebih cepat.

---

<p align="center">
  Dikelola oleh Research Group CAIS — Universitas Airlangga<br>
  <sub>Infrastruktur ini disediakan untuk keperluan penelitian akademik.</sub>
</p>

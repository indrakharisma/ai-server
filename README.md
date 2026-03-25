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
3. [Akses Ollama API](#3-akses-ollama-api)
4. [Available Models](#4-available-models)
5. [ML Training Environment](#5-ml-training-environment)
6. [Storage & Etika Penggunaan](#6-storage--etika-penggunaan)
7. [GPU Scheduler](#7-gpu-scheduler-ollama--training)
8. [Cara Request Akses](#8-cara-request-akses)
9. [Troubleshooting](#9-troubleshooting)
10. [Kontak](#10-kontak)

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

## 3. Akses Ollama API

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

## 4. Available Models

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

## 5. ML Training Environment

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

### Setup Environment Training

```bash
# Aktifkan virtual environment Python
source ~/ml-env/bin/activate

# Masuk ke folder penelitian Anda
cd /mnt/model-storage/research/NAMA_MAHASISWA

# Jalankan JupyterLab (tanpa membuka browser di server)
jupyter lab --no-browser --port=8888
```

### Forward JupyterLab ke Laptop

Buka terminal **baru** di laptop Anda:

```bash
ssh -L 8888:localhost:8888 ai-server
```

Kemudian buka di browser:

```
http://localhost:8888
```

---

## 6. Storage & Etika Penggunaan

### Struktur Folder

Setiap mahasiswa mendapatkan folder pribadi di:

```
/mnt/model-storage/research/NAMA_MAHASISWA/
```

Folder ini adalah ruang kerja Anda. Simpan dataset, checkpoint model, dan hasil eksperimen di sini.

### Aturan Penggunaan

- **Jangan hapus atau modifikasi file milik mahasiswa lain** di luar folder Anda.
- **Kuota storage per mahasiswa: 50GB.** Jika membutuhkan lebih, hubungi dosen terlebih dahulu.
- **GPU bersifat shared.** Jika ada mahasiswa lain yang sedang training, koordinasikan jadwal agar tidak bentrok.
- **Jangan menjalankan proses yang memakan seluruh VRAM (24GB)** tanpa koordinasi terlebih dahulu.
- Hapus checkpoint model yang sudah tidak dibutuhkan untuk menjaga ketersediaan storage.

---

## 7. GPU Scheduler (Ollama & Training)

GPU RTX 3090 digunakan secara bergantian antara **Ollama inference** dan **ML training**. Keduanya tidak bisa berjalan bersamaan secara optimal karena berbagi VRAM.

| Mode | Kondisi | VRAM |
|------|---------|------|
| Ollama aktif | Model LLM di-load | Hingga 19GB terpakai |
| Training aktif | Ollama di-unload manual | GPU bebas untuk training |

### Cara Unload Model Ollama Sebelum Training

Jika Anda akan memulai sesi training, unload model Ollama terlebih dahulu agar VRAM bebas:

```bash
# Unload qwen3:14b
curl -X POST http://localhost:11434/api/generate \
  -d '{"model": "qwen3:14b", "keep_alive": 0}'

# Unload qwen3:32b
curl -X POST http://localhost:11434/api/generate \
  -d '{"model": "qwen3:32b", "keep_alive": 0}'
```

> Perintah di atas dijalankan **di dalam server** (setelah SSH), bukan dari laptop.

Setelah selesai training, Ollama akan otomatis me-load model kembali saat ada request berikutnya.

---

## 8. Cara Request Akses

Akses server tidak bersifat publik. Mahasiswa yang membutuhkan akses harus mengajukan permintaan ke dosen.

**Langkah-langkah:**

1. Kirim email ke dosen (lihat bagian [Kontak](#10-kontak))
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

## 9. Troubleshooting

| Masalah | Kemungkinan Penyebab | Solusi |
|---------|----------------------|--------|
| SSH timeout / tidak bisa konek | `cloudflared` belum login atau sesi expired | Jalankan `cloudflared access login ssh-ai.ebruar.my.id` lalu coba SSH lagi |
| `CUDA out of memory` saat training | Model terlalu besar untuk sisa VRAM | Gunakan batch size lebih kecil, atau unload Ollama dulu, atau koordinasi waktu dengan mahasiswa lain |
| Ollama tidak merespons / lambat | Model sedang di-unload atau ada training berjalan | Tunggu beberapa menit atau hubungi dosen untuk konfirmasi status server |
| `401 Unauthorized` di API | Cloudflare Service Token salah atau expired | Minta token baru ke dosen |
| JupyterLab tidak bisa dibuka | Port forwarding tidak aktif | Pastikan perintah `ssh -L 8888:localhost:8888 ai-server` sudah dijalankan di terminal terpisah |
| Storage penuh | Melebihi kuota 50GB | Hapus file yang tidak dibutuhkan, atau hubungi dosen untuk penambahan kuota |

---

## 10. Kontak

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

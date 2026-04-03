## Plan: Add Ollama Model Management & Coolify Sections

### To-Do
- [x] Update Daftar Isi — tambahkan section 10 (Mengelola Model Ollama), 11 (Coolify), geser Kontak ke 12
- [x] Tambahkan section 10 "Mengelola Model Ollama" — tanpa sub-section hapus model untuk mahasiswa (diganti catatan), panduan dosen dengan storage monitoring dan kapasitas kritis (<200GB)
- [x] Tambahkan section 11 "Coolify — Web Hosting Platform"
- [x] Update section 9 "Panduan Dosen" — tambahkan catatan `sudo usermod -aG docker $USERNAME`
- [x] Commit: `docs: add Ollama model management and Coolify deployment guide`

### Review
- Section 10 (Mengelola Model Ollama): pull model dengan koordinasi, tabel estimasi ukuran, catatan storage kritis (<200GB dari 1.37TB), prosedur cleanup untuk dosen
- Section 11 (Coolify): request deploy untuk mahasiswa via email, panduan deploy + env vars + backup secrets untuk dosen
- Section 9: ditambahkan catatan docker group agar mahasiswa bisa `docker exec` tanpa sudo
- Committed sebagai `02d70b4`

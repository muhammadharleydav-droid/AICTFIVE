# AICTFIVE
AICTFIVE-dataset/
├── raw_images/
│   ├── open_hand/               ← folder gesture “open”
│   │    ├── open1.jpg
│   │    ├── open2.jpg
│   │    └── ...
│   ├── closed_hand/             ← folder gesture “grab / close”
│   │    ├── closed1.jpg
│   │    ├── closed2.jpg
│   │    └── ...
│   ├── move_left/               ← gesture “move” ke kiri
│   ├── move_right/
│   └── no_hand/                 ← gambar background / tanpa tangan (optional)
│
├── processed_images/
│   ├── open_hand/
│   ├── closed_hand/
│   ├── move_left/
│   ├── move_right/
│   └── no_hand/
│
├── landmarks/
│   ├── open_hand/
│   │    ├── open1.json          ← file landmark untuk tiap gambar
│   │    ├── open2.json
│   │    └── ...
│   ├── closed_hand/
│   └── move_left/
│   └── ... 
│
├── annotations.csv             ← CSV dengan kolom: image_path, gesture_label, landmarks (json string atau format standar)
├── train_test_split.txt        ← daftar gambar + label untuk train / test / val
├── LICENSE.md
├── README.md
└── .gitignore
# AICTFIVE Dataset

Dataset ini dikumpulkan dan disiapkan oleh tim **AICTFIVE** untuk proyek *Fruit Basket* (game terapi tangan / gesture untuk anak CP). Berisi gambar tangan dengan gesture open, grab (closed), gerakan kiri / kanan, dan kondisi tanpa tangan, plus anotasi landmark tangan dengan MediaPipe.

---

## 📂 Struktur Direktori

- `raw_images/` — gambar asli yang belum diproses; setiap folder adalah satu gesture (misalnya `open_hand`, `closed_hand`, `move_left`, `move_right`, `no_hand`).  
- `processed_images/` — gambar yang sudah diresize & augmentasi (flip, brightness, rotasi kecil).  
- `landmarks/` — file `.json` yang berisi koordinat landmark tangan (MediaPipe) untuk tiap gambar.  
- `annotations.csv` — ringkasan semua gambar: kolom `image_path`, `gesture_label`, `landmarks`.  
- `train_test_split.txt` — daftar file gambar yang dibagi ke `train`, `val`, `test`.

---

## ⚙️ Format Landmark (.json)

Setiap file landmark `.json` berisi struktur:

```json
{
  "image": "processed_images/open_hand/open1.jpg",
  "gesture": "open_hand",
  "landmarks": [
    { "x_norm": 0.45, "y_norm": 0.30, "z_norm": 0.00, "x_pix": 115, "y_pix": 76 },
    ... (21 titik tangan) ...
  ]
}
train processed_images/open_hand/open1.jpg
train processed_images/closed_hand/closed3.jpg
val processed_images/move_left/move5.jpg
test processed_images/no_hand/no2.jpg
...

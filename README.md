# Technical Test - Frontend Developer

Aplikasi Task Management dengan Vue.js yang memiliki fitur Table View dan Kanban View.

## Fitur

### Table View
- ✅ **New Task**: Menambahkan task baru di posisi paling atas
- ✅ **Search**: Mencari berdasarkan nama task
- ✅ **Filter by Developer**: Filter berdasarkan developer
- ✅ **Sort**: Sorting berdasarkan kolom (multi-column sort)
- ✅ **Inline Editing**: Edit langsung di tabel dengan klik
- ✅ **Status Options**: Ready to start, In Progress, Waiting for review, Pending Deploy, Done, Stuck
- ✅ **Priority Options**: Critical, High, Medium, Low, Best Effort
- ✅ **Type Options**: Feature Enhancements, Other, Bug
- ✅ **Date Format**: dd MMM, yyyy dengan datepicker
- ✅ **SP Columns**: Estimated SP dan Actual SP (integer)
- ✅ **Percentage Statistics**: Persentase dengan warna unik untuk Status, Priority, dan Type
- ✅ **No Refresh**: Semua aksi tanpa refresh halaman

### Kanban View (Bonus)
- ✅ **New Task Modal**: Form lengkap untuk menambahkan task baru
- ✅ **Search & Filter**: Sama dengan Table View
- ✅ **Kanban Columns**: 6 kolom berdasarkan Status
- ✅ **Task Cards**: Menampilkan Task, Priority, Type, Estimated SP, dan Developer
- ✅ **Drag & Drop**: Pindahkan task antar status dengan drag and drop
- ✅ **Sync with Table**: Data kanban tersinkronisasi dengan tabel

## Teknologi
- Vue 3 (Composition API)
- Tailwind CSS
- Vite

## Instalasi

```bash
# Install dependencies
npm install

# Run development server
npm run dev

# Build for production
npm run build
```

## Penggunaan

1. Buka aplikasi di browser (default: http://localhost:5173)
2. Toggle antara Table View dan Kanban View
3. Gunakan search dan filter untuk mencari task
4. Klik kolom header untuk sorting (dapat multiple columns)
5. Di Table View, klik cell untuk edit inline
6. Di Kanban View, drag and drop card untuk mengubah status

## API Integration

Update URL API di file `src/App.vue` baris 15:
```javascript
const response = await fetch('YOUR_API_URL_HERE')
```

## Struktur Folder

```
src/
├── App.vue                 # Main component
├── components/
│   ├── TableView.vue       # Table view dengan inline editing
│   └── KanbanView.vue      # Kanban board dengan drag & drop
├── style.css               # Global styles
└── main.js                 # Entry point
```

## Catatan

- Aplikasi menggunakan sample data jika API tidak tersedia
- Semua warna untuk Status, Priority, dan Type sudah di-set dengan warna yang berbeda
- Inline editing di table: klik cell untuk edit, Enter untuk save, Esc untuk cancel
- Drag and drop di kanban akan otomatis update status task

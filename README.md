# 🌿 Bonsai Garden Tracker — PWA

Aplikasi monitoring pohon bonsai lengkap yang bisa diinstall sebagai app di HP dan desktop.

## 📦 Isi Package

```
bonsai-garden/
├── index.html      ← Aplikasi utama (self-contained)
├── manifest.json   ← PWA manifest (nama, ikon, warna)
├── sw.js           ← Service Worker (offline support)
├── icon-192.png    ← App icon 192x192
├── icon-512.png    ← App icon 512x512
└── README.md
```

## 🚀 Cara Deploy

### Opsi 1: GitHub Pages (GRATIS, Paling Mudah)
1. Buat repository baru di GitHub
2. Upload semua file ke repo
3. Settings → Pages → Source: main branch
4. Akses via `https://username.github.io/nama-repo`
5. Buka di HP → "Add to Home Screen" → Terinstall!

### Opsi 2: Netlify (GRATIS)
1. Drag & drop folder ke netlify.com/drop
2. Dapat URL langsung (mis. `bonsai-garden.netlify.app`)
3. Buka URL di HP → Install

### Opsi 3: Vercel (GRATIS)
```bash
npm i -g vercel
vercel --prod
```

### Opsi 4: Server Sendiri
Upload semua file ke public_html / htdocs server.
Pastikan HTTPS aktif (diperlukan untuk PWA install).

## 📱 Cara Install di HP

### Android (Chrome)
1. Buka URL di Chrome
2. Tap menu ⋮ → "Add to Home screen"
3. Atau tunggu banner otomatis muncul
4. App tersimpan di home screen seperti app native!

### iPhone/iPad (Safari)
1. Buka URL di Safari
2. Tap Share button → "Add to Home Screen"
3. App terinstall dengan ikon 🌿

### Desktop (Chrome/Edge)
1. Buka URL
2. Klik ikon install (⊞) di address bar
3. Atau Menu → "Install Bonsai Garden"

## 🔗 Setup Google Sheets Sync

### Langkah 1: Siapkan Google Sheets
1. Buat Google Spreadsheet baru
2. Buat sheet bernama `Bonsai`
3. Row 1 isi header: `id | name | species | date | location | waterFreq | lastWater | style | notes | photo | history`

### Langkah 2: Buat Apps Script
1. Extensions → Apps Script
2. Hapus kode default, paste kode di bawah
3. Simpan → Deploy → New Deployment
4. Type: Web App, Execute as: Me, Access: Anyone
5. Copy URL Deployment

### Kode Apps Script:
```javascript
function doGet(e) {
  const sheet = SpreadsheetApp.getActiveSpreadsheet()
    .getSheetByName('Bonsai') || 
    SpreadsheetApp.getActiveSpreadsheet().insertSheet('Bonsai');
  
  if (e.parameter.action === 'getAll') {
    const data = sheet.getDataRange().getValues();
    if (data.length < 2) return json({ status: 'ok', data: [] });
    const headers = data[0];
    const rows = data.slice(1).map(row => {
      const obj = {};
      headers.forEach((h, i) => obj[h] = row[i]);
      try { obj.history = JSON.parse(obj.history || '[]'); } catch(e) { obj.history = []; }
      return obj;
    }).filter(r => r.id);
    return json({ status: 'ok', data: rows });
  }
  return json({ status: 'ok' });
}

function doPost(e) {
  const body = JSON.parse(e.postData.contents);
  const sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName('Bonsai');
  const { action, data, id } = body;
  
  if (action === 'upsert' && data) {
    const allData = sheet.getDataRange().getValues();
    const headers = allData[0];
    if (!headers || headers[0] !== 'id') {
      // Init headers
      sheet.getRange(1, 1, 1, 11).setValues([['id','name','species','date','location','waterFreq','lastWater','style','notes','photo','history']]);
      allData[0] = ['id','name','species','date','location','waterFreq','lastWater','style','notes','photo','history'];
    }
    const rows = allData.slice(1);
    const idx = rows.findIndex(r => r[0] === data.id);
    const row = allData[0].map(h => h === 'history' ? JSON.stringify(data[h] || []) : (data[h] !== undefined ? data[h] : ''));
    if (idx >= 0) sheet.getRange(idx + 2, 1, 1, row.length).setValues([row]);
    else sheet.appendRow(row);
  }
  
  if (action === 'delete' && id) {
    const allData = sheet.getDataRange().getValues();
    const idx = allData.slice(1).findIndex(r => r[0] === id);
    if (idx >= 0) sheet.deleteRow(idx + 2);
  }
  
  return json({ status: 'ok' });
}

function json(data) {
  return ContentService.createTextOutput(JSON.stringify(data))
    .setMimeType(ContentService.MimeType.JSON);
}
```

## 🏷️ Cara Print Label Barcode
1. Buka detail pohon
2. Klik "⬇️ Download Label (PNG)"
3. File PNG otomatis terdownload
4. Print dengan printer label (ukuran A7/kartu nama)
5. Tempel pada pot bonsai

## 📷 Cara Scan Barcode
1. Buka tab "📱 Scan Barcode"
2. Klik "📷 Mulai Kamera" → izinkan akses kamera
3. Arahkan ke barcode pada label pot
4. Info pohon otomatis muncul
5. Atau ketik ID secara manual

---
Made with 🌿 for Bonsai enthusiasts

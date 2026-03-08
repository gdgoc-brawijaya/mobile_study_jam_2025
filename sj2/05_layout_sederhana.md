# Layout Sederhana: Header, Content, Footer

## Struktur yang Akan Dibuat:

```
┌─────────────────────────────────┐
│         HEADER                  │  ← Fixed di atas
├─────────────────────────────────┤
│                                 │
│                                 │
│         CONTENT                 │  ← Mengisi ruang (scrollable)
│                                 │
│                                 │
├─────────────────────────────────┤
│         TOMBOL                  │  ← Fixed di bawah
└─────────────────────────────────┘
```

---

## Kode Lengkap

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: const LayoutSederhana(),
    );
  }
}

class LayoutSederhana extends StatelessWidget {
  const LayoutSederhana({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Layout Sederhana'),
        backgroundColor: Colors.blue,
        foregroundColor: Colors.white,
      ),
      body: Column(
        children: [
          // ========== HEADER ==========
          Container(
            width: double.infinity,
            padding: const EdgeInsets.all(20),
            color: Colors.blue[100],
            child: const Column(
              children: [
                CircleAvatar(
                  radius: 40,
                  backgroundColor: Colors.blue,
                  child: Icon(Icons.person, size: 40, color: Colors.white),
                ),
                SizedBox(height: 10),
                Text(
                  'John Doe',
                  style: TextStyle(
                    fontSize: 24,
                    fontWeight: FontWeight.bold,
                  ),
                ),
                Text(
                  'Flutter Developer',
                  style: TextStyle(
                    fontSize: 16,
                    color: Colors.grey,
                  ),
                ),
              ],
            ),
          ),

          // ========== CONTENT (Expanded agar mengisi ruang) ==========
          Expanded(
            child: SingleChildScrollView(
              padding: const EdgeInsets.all(16),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  const Text(
                    'Tentang Saya',
                    style: TextStyle(
                      fontSize: 20,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                  const SizedBox(height: 10),
                  const Text(
                    'Lorem ipsum dolor sit amet, consectetur adipiscing elit. '
                    'Sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. '
                    'Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris.',
                  ),
                  const SizedBox(height: 20),
                  
                  const Text(
                    'Skills',
                    style: TextStyle(
                      fontSize: 20,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                  const SizedBox(height: 10),
                  
                  Wrap(
                    spacing: 8,
                    runSpacing: 8,
                    children: [
                      _skillChip('Flutter'),
                      _skillChip('Dart'),
                      _skillChip('Firebase'),
                      _skillChip('REST API'),
                      _skillChip('Git'),
                    ],
                  ),
                  const SizedBox(height: 20),
                  
                  const Text(
                    'Pengalaman',
                    style: TextStyle(
                      fontSize: 20,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                  const SizedBox(height: 10),
                  
                  _experienceCard(
                    'PT. Tech Indonesia',
                    'Mobile Developer',
                    '2023 - Sekarang',
                  ),
                  _experienceCard(
                    'Startup ABC',
                    'Junior Developer',
                    '2021 - 2023',
                  ),
                ],
              ),
            ),
          ),

          // ========== TOMBOL DI BAWAH ==========
          Container(
            width: double.infinity,
            padding: const EdgeInsets.all(16),
            decoration: BoxDecoration(
              color: Colors.white,
              boxShadow: [
                BoxShadow(
                  color: Colors.black.withOpacity(0.1),
                  blurRadius: 10,
                  offset: const Offset(0, -5),
                ),
              ],
            ),
            child: ElevatedButton(
              onPressed: () {
                // Aksi ketika tombol ditekan
              },
              style: ElevatedButton.styleFrom(
                backgroundColor: Colors.blue,
                foregroundColor: Colors.white,
                padding: const EdgeInsets.symmetric(vertical: 16),
                shape: RoundedRectangleBorder(
                  borderRadius: BorderRadius.circular(10),
                ),
              ),
              child: const Text(
                'Hubungi Saya',
                style: TextStyle(fontSize: 16),
              ),
            ),
          ),
        ],
      ),
    );
  }

  static Widget _skillChip(String skill) {
    return Chip(
      label: Text(skill),
      backgroundColor: Colors.blue[50],
    );
  }

  static Widget _experienceCard(String company, String role, String period) {
    return Card(
      margin: const EdgeInsets.only(bottom: 10),
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text(
              company,
              style: const TextStyle(
                fontWeight: FontWeight.bold,
                fontSize: 16,
              ),
            ),
            Text(role),
            Text(
              period,
              style: const TextStyle(color: Colors.grey),
            ),
          ],
        ),
      ),
    );
  }
}
```

---

## Penjelasan Struktur:

### 1. Header (Container dengan ukuran tetap)
```dart
Container(
  width: double.infinity,  // Full width
  padding: EdgeInsets.all(20),
  color: Colors.blue[100],
  child: Column(...),  // Profile info
)
```

### 2. Content (Expanded + SingleChildScrollView)
```dart
Expanded(  // Mengisi sisa ruang
  child: SingleChildScrollView(  // Bisa di-scroll
    child: Column(...),  // Konten
  ),
)
```

### 3. Footer/Tombol (Container fixed di bawah)
```dart
Container(
  width: double.infinity,
  padding: EdgeInsets.all(16),
  child: ElevatedButton(...),
)
```

---

## Hasil Visual:

```
┌─────────────────────────────────┐
│        Layout Sederhana         │ ← AppBar
├─────────────────────────────────┤
│           👤                    │
│         John Doe                │ ← Header
│     Flutter Developer           │
├─────────────────────────────────┤
│                                 │
│ Tentang Saya                    │
│ Lorem ipsum dolor sit amet...   │
│                                 │
│ Skills                          │ ← Content
│ [Flutter] [Dart] [Firebase]     │   (scrollable)
│                                 │
│ Pengalaman                      │
│ ┌─────────────────────────────┐ │
│ │ PT. Tech Indonesia          │ │
│ │ Mobile Developer            │ │
│ └─────────────────────────────┘ │
│                                 │
├─────────────────────────────────┤
│ [========Hubungi Saya========]  │ ← Tombol
└─────────────────────────────────┘
```

---

## Variasi: Tanpa AppBar (Full Screen)

```dart
class LayoutFullScreen extends StatelessWidget {
  const LayoutFullScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: SafeArea(  // Hindari notch dan status bar
        child: Column(
          children: [
            // Header
            Container(...),
            
            // Content
            Expanded(
              child: SingleChildScrollView(...),
            ),
            
            // Footer
            Container(...),
          ],
        ),
      ),
    );
  }
}
```

---

## Tips Layout:

1. **Selalu gunakan `Expanded`** untuk bagian yang harus mengisi ruang
2. **Gunakan `SingleChildScrollView`** jika konten bisa lebih panjang dari layar
3. **`SafeArea`** penting untuk menghindari notch di HP modern
4. **`width: double.infinity`** untuk membuat widget full width

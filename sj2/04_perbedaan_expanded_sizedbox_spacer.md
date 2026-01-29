# Perbedaan Expanded, SizedBox, dan Spacer

## Ringkasan Singkat

| Widget | Fungsi | Ukuran |
|--------|--------|--------|
| **SizedBox** | Memberi jarak **tetap** | Fixed (misal: 20px) |
| **Spacer** | Memberi jarak **fleksibel** | Mengisi sisa ruang |
| **Expanded** | Membuat child **mengisi ruang** | Mengisi sisa ruang |

---

## 1. SizedBox - Jarak Tetap

SizedBox memberikan jarak dengan ukuran **pasti/tetap**.

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
      home: Scaffold(
        appBar: AppBar(title: const Text('SizedBox Demo')),
        body: Column(
          children: [
            Container(
              width: 100,
              height: 100,
              color: Colors.red,
              child: const Center(child: Text('1')),
            ),
            
            // SizedBox memberikan jarak 50 pixel
            const SizedBox(height: 50),
            
            Container(
              width: 100,
              height: 100,
              color: Colors.blue,
              child: const Center(child: Text('2')),
            ),
            
            // SizedBox memberikan jarak 20 pixel
            const SizedBox(height: 20),
            
            Container(
              width: 100,
              height: 100,
              color: Colors.green,
              child: const Center(child: Text('3')),
            ),
          ],
        ),
      ),
    );
  }
}
```

### Hasil:
```
┌─────────────────────────────────┐
│  [Merah]                        │
│                                 │
│  ← 50px jarak (SizedBox) →      │
│                                 │
│  [Biru]                         │
│  ← 20px jarak →                 │
│  [Hijau]                        │
│                                 │
│  (sisa ruang kosong)            │
│                                 │
└─────────────────────────────────┘
```

---

## 2. Spacer - Jarak Fleksibel

Spacer mengisi **semua sisa ruang kosong** secara otomatis.

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
      home: Scaffold(
        appBar: AppBar(title: const Text('Spacer Demo')),
        body: Column(
          children: [
            Container(
              width: 100,
              height: 100,
              color: Colors.red,
              child: const Center(child: Text('1')),
            ),
            
            // Spacer mengisi SEMUA ruang kosong di tengah
            const Spacer(),
            
            Container(
              width: 100,
              height: 100,
              color: Colors.blue,
              child: const Center(child: Text('2')),
            ),
          ],
        ),
      ),
    );
  }
}
```

### Hasil:
```
┌─────────────────────────────────┐
│  [Merah] ← di atas              │
│                                 │
│                                 │
│  ← Spacer mengisi ruang ini →   │
│                                 │
│                                 │
│  [Biru] ← di bawah              │
└─────────────────────────────────┘
```

---

## 3. Expanded - Widget Mengisi Ruang

Expanded membuat **child widget** mengisi sisa ruang yang tersedia.

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
      home: Scaffold(
        appBar: AppBar(title: const Text('Expanded Demo')),
        body: Column(
          children: [
            Container(
              height: 100,
              color: Colors.red,
              child: const Center(child: Text('Fixed 100px')),
            ),
            
            // Container ini akan mengisi SEMUA sisa ruang
            Expanded(
              child: Container(
                color: Colors.green,
                child: const Center(
                  child: Text(
                    'Expanded!\nSaya mengisi sisa ruang',
                    textAlign: TextAlign.center,
                  ),
                ),
              ),
            ),
            
            Container(
              height: 100,
              color: Colors.blue,
              child: const Center(child: Text('Fixed 100px')),
            ),
          ],
        ),
      ),
    );
  }
}
```

### Hasil:
```
┌─────────────────────────────────┐
│  [Merah - 100px]                │
├─────────────────────────────────┤
│                                 │
│                                 │
│  [Hijau - EXPANDED]             │
│  Mengisi semua sisa ruang       │
│                                 │
│                                 │
├─────────────────────────────────┤
│  [Biru - 100px]                 │
└─────────────────────────────────┘
```

---

## 4. Perbandingan Langsung (Row)

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
      home: Scaffold(
        appBar: AppBar(title: const Text('Perbandingan Lengkap')),
        body: Padding(
          padding: const EdgeInsets.all(16),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              // Contoh 1: Tanpa apapun
              const Text('1. Tanpa SizedBox/Spacer/Expanded:',
                  style: TextStyle(fontWeight: FontWeight.bold)),
              Container(
                height: 60,
                color: Colors.grey[200],
                child: Row(
                  children: [
                    _box(Colors.red),
                    _box(Colors.green),
                    _box(Colors.blue),
                  ],
                ),
              ),
              const SizedBox(height: 20),

              // Contoh 2: Dengan SizedBox
              const Text('2. Dengan SizedBox (jarak tetap 30px):',
                  style: TextStyle(fontWeight: FontWeight.bold)),
              Container(
                height: 60,
                color: Colors.grey[200],
                child: Row(
                  children: [
                    _box(Colors.red),
                    const SizedBox(width: 30),
                    _box(Colors.green),
                    const SizedBox(width: 30),
                    _box(Colors.blue),
                  ],
                ),
              ),
              const SizedBox(height: 20),

              // Contoh 3: Dengan Spacer
              const Text('3. Dengan Spacer (jarak fleksibel):',
                  style: TextStyle(fontWeight: FontWeight.bold)),
              Container(
                height: 60,
                color: Colors.grey[200],
                child: Row(
                  children: [
                    _box(Colors.red),
                    const Spacer(),
                    _box(Colors.green),
                    const Spacer(),
                    _box(Colors.blue),
                  ],
                ),
              ),
              const SizedBox(height: 20),

              // Contoh 4: Dengan Expanded
              const Text('4. Dengan Expanded (widget mengisi ruang):',
                  style: TextStyle(fontWeight: FontWeight.bold)),
              Container(
                height: 60,
                color: Colors.grey[200],
                child: Row(
                  children: [
                    _box(Colors.red),
                    Expanded(
                      child: Container(
                        margin: const EdgeInsets.symmetric(horizontal: 8),
                        color: Colors.orange,
                        child: const Center(child: Text('Expanded')),
                      ),
                    ),
                    _box(Colors.blue),
                  ],
                ),
              ),
              const SizedBox(height: 20),

              // Contoh 5: Expanded dengan flex
              const Text('5. Expanded dengan Flex (1:2:1):',
                  style: TextStyle(fontWeight: FontWeight.bold)),
              Container(
                height: 60,
                color: Colors.grey[200],
                child: Row(
                  children: [
                    Expanded(
                      flex: 1,
                      child: Container(
                        color: Colors.red,
                        child: const Center(child: Text('1')),
                      ),
                    ),
                    Expanded(
                      flex: 2,
                      child: Container(
                        color: Colors.green,
                        child: const Center(child: Text('2')),
                      ),
                    ),
                    Expanded(
                      flex: 1,
                      child: Container(
                        color: Colors.blue,
                        child: const Center(child: Text('1')),
                      ),
                    ),
                  ],
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }

  static Widget _box(Color color) {
    return Container(
      width: 50,
      height: 50,
      color: color,
    );
  }
}
```

### Hasil Visual:

```
1. Tanpa apapun:
   [R][G][B]                    ← Widget nempel

2. Dengan SizedBox (30px):
   [R]  30px  [G]  30px  [B]    ← Jarak tetap

3. Dengan Spacer:
   [R]      ←→      [G]      ←→      [B]    ← Jarak fleksibel, mengisi ruang

4. Dengan Expanded:
   [R][========ORANGE========][B]    ← Orange mengisi ruang

5. Expanded dengan Flex (1:2:1):
   [==R==][=====G=====][==B==]    ← Rasio 1:2:1
```

---

## 5. Kapan Pakai Apa?

### Gunakan **SizedBox** ketika:
- Butuh jarak dengan ukuran **pasti** (misal: 16px, 24px)
- Jarak antar widget yang konsisten

```dart
Column(
  children: [
    Text('Judul'),
    SizedBox(height: 8),   // Jarak 8px
    Text('Deskripsi'),
    SizedBox(height: 16),  // Jarak 16px
    ElevatedButton(...),
  ],
)
```

### Gunakan **Spacer** ketika:
- Butuh **mendorong** widget ke ujung
- Membuat jarak yang mengisi ruang kosong

```dart
Row(
  children: [
    Text('Logo'),
    Spacer(),           // Dorong ke ujung
    Icon(Icons.menu),
  ],
)
```

### Gunakan **Expanded** ketika:
- Widget perlu **mengisi ruang** yang tersedia
- Membuat layout **responsif**
- Membagi ruang dengan **rasio** tertentu

```dart
Row(
  children: [
    Expanded(
      flex: 2,
      child: TextField(),  // 2/3 layar
    ),
    SizedBox(width: 8),
    Expanded(
      flex: 1,
      child: ElevatedButton(...),  // 1/3 layar
    ),
  ],
)
```

---

## 6. Contoh Kasus: Layout Tombol

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
      home: Scaffold(
        appBar: AppBar(title: const Text('Layout Tombol')),
        body: Padding(
          padding: const EdgeInsets.all(16),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.stretch,
            children: [
              const Text('Tombol di pojok kanan:'),
              Row(
                children: [
                  const Spacer(),
                  ElevatedButton(
                    onPressed: () {},
                    child: const Text('Simpan'),
                  ),
                ],
              ),
              const SizedBox(height: 30),
              
              const Text('Dua tombol dengan jarak:'),
              Row(
                children: [
                  Expanded(
                    child: OutlinedButton(
                      onPressed: () {},
                      child: const Text('Batal'),
                    ),
                  ),
                  const SizedBox(width: 16),
                  Expanded(
                    child: ElevatedButton(
                      onPressed: () {},
                      child: const Text('Simpan'),
                    ),
                  ),
                ],
              ),
              const SizedBox(height: 30),
              
              const Text('Tombol full width:'),
              SizedBox(
                width: double.infinity,
                child: ElevatedButton(
                  onPressed: () {},
                  child: const Text('Kirim'),
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

### Hasil:
```
Tombol di pojok kanan:
                              [Simpan]

Dua tombol dengan jarak:
[=====Batal=====] 16px [=====Simpan=====]

Tombol full width:
[================Kirim================]
```

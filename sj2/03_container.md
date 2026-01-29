# Container di Flutter

## Apa itu Container?

Container adalah widget **serba bisa** yang sering dipakai untuk:
- Memberi **padding** (jarak dalam)
- Memberi **margin** (jarak luar)
- Memberi **background color**
- Mengatur **ukuran** (width/height)
- Memberi **border** dan **border radius**
- Memberi **shadow** (decoration)

---

## Contoh Lengkap Container

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
        appBar: AppBar(title: const Text('Container Demo')),
        backgroundColor: Colors.grey[100],
        body: SingleChildScrollView(
          padding: const EdgeInsets.all(16),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              // 1. Container dengan color
              const Text('1. Container dengan Color:'),
              Container(
                width: 200,
                height: 100,
                color: Colors.blue,
                child: const Center(
                  child: Text('Halo!', style: TextStyle(color: Colors.white)),
                ),
              ),
              const SizedBox(height: 20),

              // 2. Container dengan padding
              const Text('2. Container dengan Padding:'),
              Container(
                color: Colors.green,
                padding: const EdgeInsets.all(20),
                child: const Text(
                  'Ada padding 20 di semua sisi',
                  style: TextStyle(color: Colors.white),
                ),
              ),
              const SizedBox(height: 20),

              // 3. Container dengan margin
              const Text('3. Container dengan Margin:'),
              Container(
                color: Colors.orange,
                margin: const EdgeInsets.only(left: 50),
                padding: const EdgeInsets.all(10),
                child: const Text('Margin kiri 50'),
              ),
              const SizedBox(height: 20),

              // 4. Container dengan decoration (rounded corner)
              const Text('4. Container dengan Border Radius:'),
              Container(
                width: 200,
                height: 100,
                decoration: BoxDecoration(
                  color: Colors.purple,
                  borderRadius: BorderRadius.circular(20),
                ),
                child: const Center(
                  child: Text('Rounded!', style: TextStyle(color: Colors.white)),
                ),
              ),
              const SizedBox(height: 20),

              // 5. Container dengan border
              const Text('5. Container dengan Border:'),
              Container(
                width: 200,
                height: 100,
                decoration: BoxDecoration(
                  color: Colors.white,
                  border: Border.all(color: Colors.red, width: 3),
                  borderRadius: BorderRadius.circular(10),
                ),
                child: const Center(child: Text('Ada Border!')),
              ),
              const SizedBox(height: 20),

              // 6. Container dengan shadow
              const Text('6. Container dengan Shadow:'),
              Container(
                width: 200,
                height: 100,
                decoration: BoxDecoration(
                  color: Colors.white,
                  borderRadius: BorderRadius.circular(15),
                  boxShadow: [
                    BoxShadow(
                      color: Colors.black.withOpacity(0.3),
                      blurRadius: 10,
                      offset: const Offset(0, 5),
                    ),
                  ],
                ),
                child: const Center(child: Text('Ada Shadow!')),
              ),
              const SizedBox(height: 20),

              // 7. Container dengan gradient
              const Text('7. Container dengan Gradient:'),
              Container(
                width: 200,
                height: 100,
                decoration: BoxDecoration(
                  borderRadius: BorderRadius.circular(15),
                  gradient: const LinearGradient(
                    colors: [Colors.blue, Colors.purple],
                    begin: Alignment.topLeft,
                    end: Alignment.bottomRight,
                  ),
                ),
                child: const Center(
                  child: Text('Gradient!', style: TextStyle(color: Colors.white)),
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

---

## Hasil Visual:

```
1. Container dengan Color:
   ┌──────────────────────┐
   │     Halo! (biru)     │
   └──────────────────────┘

2. Container dengan Padding:
   ┌──────────────────────────────────┐
   │ ┌──────────────────────────────┐ │
   │ │ Ada padding 20 (hijau)       │ │
   │ └──────────────────────────────┘ │
   └──────────────────────────────────┘

3. Container dengan Margin:
        ← 50px →┌─────────────────┐
                │ Margin kiri 50  │
                └─────────────────┘

4. Container dengan Border Radius:
   ╭──────────────────────╮
   │     Rounded!         │
   ╰──────────────────────╯

5. Container dengan Border:
   ┏━━━━━━━━━━━━━━━━━━━━━━┓
   ┃     Ada Border!      ┃  (border merah)
   ┗━━━━━━━━━━━━━━━━━━━━━━┛

6. Container dengan Shadow:
   ╭──────────────────────╮
   │     Ada Shadow!      │
   ╰──────────────────────╯
      ▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒ (bayangan)

7. Container dengan Gradient:
   ╭──────────────────────╮
   │ 🔵 → 🟣 Gradient!   │
   ╰──────────────────────╯
```

---

## Tips Penting!

⚠️ **Jangan pakai `color` dan `decoration` bersamaan!**

```dart
// ❌ SALAH - akan error
Container(
  color: Colors.blue,
  decoration: BoxDecoration(...),
)

// ✅ BENAR - color di dalam decoration
Container(
  decoration: BoxDecoration(
    color: Colors.blue,
    borderRadius: BorderRadius.circular(10),
  ),
)
```

---

## Padding vs Margin

```dart
Container(
  margin: EdgeInsets.all(20),   // Jarak dari LUAR container
  padding: EdgeInsets.all(20),  // Jarak dari DALAM container ke child
  color: Colors.blue,
  child: Text('Hello'),
)
```

```
        ← margin →
┌─────────────────────────────┐
│       ← padding →           │
│   ┌─────────────────────┐   │
│   │      Hello          │   │
│   └─────────────────────┘   │
│                             │
└─────────────────────────────┘
```

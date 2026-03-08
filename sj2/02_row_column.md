# Row dan Column di Flutter

## Konsep Dasar
- **Row**: Menyusun widget secara **horizontal** (kiri ke kanan)
- **Column**: Menyusun widget secara **vertikal** (atas ke bawah)

---

## Contoh Row (Horizontal)

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
        appBar: AppBar(title: const Text('Row Demo')),
        body: Row(
          children: [
            Container(
              width: 100,
              height: 100,
              color: Colors.red,
              child: const Center(child: Text('1')),
            ),
            Container(
              width: 100,
              height: 100,
              color: Colors.green,
              child: const Center(child: Text('2')),
            ),
            Container(
              width: 100,
              height: 100,
              color: Colors.blue,
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
┌─────────────────────────────────────┐
│ [Merah 1] [Hijau 2] [Biru 3]        │
│                                     │
│                                     │
└─────────────────────────────────────┘
```
3 kotak warna tersusun **horizontal** dari kiri ke kanan.

---

## Contoh Column (Vertikal)

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
        appBar: AppBar(title: const Text('Column Demo')),
        body: Column(
          children: [
            Container(
              width: 100,
              height: 100,
              color: Colors.red,
              child: const Center(child: Text('1')),
            ),
            Container(
              width: 100,
              height: 100,
              color: Colors.green,
              child: const Center(child: Text('2')),
            ),
            Container(
              width: 100,
              height: 100,
              color: Colors.blue,
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
┌─────────────────────────────────────┐
│ [Merah 1]                           │
│ [Hijau 2]                           │
│ [Biru 3]                            │
│                                     │
└─────────────────────────────────────┘
```
3 kotak warna tersusun **vertikal** dari atas ke bawah.

---

## MainAxisAlignment & CrossAxisAlignment

### Ilustrasi Axis:

**Column:**
```
      ← crossAxis →
         ┌───┐
    ↑    │   │
    │    │   │
  main   │   │
  Axis   │   │
    │    │   │
    ↓    └───┘
```

**Row:**
```
         ↑ crossAxis
         │
    ┌────┴────┐
    │         │
    ← mainAxis →
    │         │
    └────┬────┘
         ↓
```

---

## Contoh MainAxisAlignment

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
        appBar: AppBar(title: const Text('MainAxisAlignment Demo')),
        body: Column(
          children: [
            // SpaceBetween
            Container(
              height: 100,
              color: Colors.grey[200],
              child: Row(
                mainAxisAlignment: MainAxisAlignment.spaceBetween,
                children: [
                  _box(Colors.red, 'spaceBetween'),
                  _box(Colors.green, ''),
                  _box(Colors.blue, ''),
                ],
              ),
            ),
            const SizedBox(height: 10),
            
            // Center
            Container(
              height: 100,
              color: Colors.grey[300],
              child: Row(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  _box(Colors.red, 'center'),
                  _box(Colors.green, ''),
                  _box(Colors.blue, ''),
                ],
              ),
            ),
            const SizedBox(height: 10),
            
            // SpaceEvenly
            Container(
              height: 100,
              color: Colors.grey[200],
              child: Row(
                mainAxisAlignment: MainAxisAlignment.spaceEvenly,
                children: [
                  _box(Colors.red, 'spaceEvenly'),
                  _box(Colors.green, ''),
                  _box(Colors.blue, ''),
                ],
              ),
            ),
            const SizedBox(height: 10),
            
            // SpaceAround
            Container(
              height: 100,
              color: Colors.grey[300],
              child: Row(
                mainAxisAlignment: MainAxisAlignment.spaceAround,
                children: [
                  _box(Colors.red, 'spaceAround'),
                  _box(Colors.green, ''),
                  _box(Colors.blue, ''),
                ],
              ),
            ),
          ],
        ),
      ),
    );
  }

  static Widget _box(Color color, String text) {
    return Container(
      width: 80,
      height: 80,
      color: color,
      child: Center(
        child: Text(text, style: const TextStyle(color: Colors.white, fontSize: 10)),
      ),
    );
  }
}
```

### Hasil Visual:

```
spaceBetween:  [R]                    [G]                    [B]
center:                    [R][G][B]
spaceEvenly:        [R]         [G]         [B]
spaceAround:     [R]      [G]      [B]
```

---

## Contoh CrossAxisAlignment

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
        appBar: AppBar(title: const Text('CrossAxisAlignment Demo')),
        body: Row(
          children: [
            // Start
            Expanded(
              child: Container(
                height: 200,
                color: Colors.grey[200],
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    _box(Colors.red),
                    const Text('start'),
                  ],
                ),
              ),
            ),
            
            // Center
            Expanded(
              child: Container(
                height: 200,
                color: Colors.grey[300],
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.center,
                  children: [
                    _box(Colors.green),
                    const Text('center'),
                  ],
                ),
              ),
            ),
            
            // End
            Expanded(
              child: Container(
                height: 200,
                color: Colors.grey[200],
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.end,
                  children: [
                    _box(Colors.blue),
                    const Text('end'),
                  ],
                ),
              ),
            ),
          ],
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

### Hasil:
- **start**: Widget menempel di kiri
- **center**: Widget di tengah
- **end**: Widget menempel di kanan

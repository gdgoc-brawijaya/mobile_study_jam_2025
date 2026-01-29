# Widget Tree di Flutter

## Konsep: Everything is a Widget

Di Flutter, semua elemen UI adalah widget. Text, Button, bahkan halaman penuh sekalipun adalah widget.

## Contoh Widget Tree Sederhana

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
        appBar: AppBar(
          title: const Text('Widget Tree Demo'),
        ),
        body: Column(
          children: [
            const Text('Ini adalah Text Widget'),
            ElevatedButton(
              onPressed: () {},
              child: const Text('Ini adalah Button'),
            ),
            Container(
              padding: const EdgeInsets.all(16),
              color: Colors.blue,
              child: const Text(
                'Ini Text di dalam Container',
                style: TextStyle(color: Colors.white),
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

## Struktur Widget Tree dari Kode di Atas:

```
MaterialApp
└── Scaffold
    ├── AppBar
    │   └── Text ('Widget Tree Demo')
    └── Column (body)
        ├── Text ('Ini adalah Text Widget')
        ├── ElevatedButton
        │   └── Text ('Ini adalah Button')
        └── Container
            └── Text ('Ini Text di dalam Container')
```

## Hasil ketika di-run:
- Akan muncul AppBar dengan judul "Widget Tree Demo"
- Di body ada 3 widget tersusun vertikal:
  1. Text biasa
  2. Tombol biru dengan text
  3. Container berwarna biru dengan text putih di dalamnya

---

## StatelessWidget vs StatefulWidget

### StatelessWidget (Statis - tidak berubah)
```dart
class ProfileCard extends StatelessWidget {
  const ProfileCard({super.key});

  @override
  Widget build(BuildContext context) {
    return const Card(
      child: Text('Nama: John Doe'),
    );
  }
}
```

### StatefulWidget (Dinamis - bisa berubah)
```dart
class CounterWidget extends StatefulWidget {
  const CounterWidget({super.key});

  @override
  State<CounterWidget> createState() => _CounterWidgetState();
}

class _CounterWidgetState extends State<CounterWidget> {
  int counter = 0;

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text('Counter: $counter'),
        ElevatedButton(
          onPressed: () {
            setState(() {
              counter++;
            });
          },
          child: const Text('Tambah'),
        ),
      ],
    );
  }
}
```

**Perbedaan:**
- `StatelessWidget`: Tampilan tidak berubah setelah di-render
- `StatefulWidget`: Tampilan bisa berubah menggunakan `setState()`

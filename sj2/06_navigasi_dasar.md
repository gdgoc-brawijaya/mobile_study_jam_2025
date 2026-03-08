# Navigasi Dasar di Flutter

## Konsep Stack

Navigasi di Flutter menggunakan konsep **Stack** (tumpukan):

```
         ┌─────────────┐
         │   Detail    │  ← push (tambah di atas)
         ├─────────────┤
         │    Home     │  ← halaman awal
         └─────────────┘
         
Ketika back/pop:
         ┌─────────────┐
         │    Home     │  ← Detail di-pop (dihapus)
         └─────────────┘
```

---

## Navigasi Dasar: Navigator.push & Navigator.pop

### Kode Lengkap (3 Halaman)

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
      title: 'Navigasi Demo',
      theme: ThemeData(
        primarySwatch: Colors.blue,
        useMaterial3: true,
      ),
      home: const HomePage(),
    );
  }
}

// ==================== HOME PAGE ====================
class HomePage extends StatelessWidget {
  const HomePage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Home'),
        backgroundColor: Colors.blue,
        foregroundColor: Colors.white,
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Icon(
              Icons.home,
              size: 80,
              color: Colors.blue,
            ),
            const SizedBox(height: 20),
            const Text(
              'Ini adalah Home Page',
              style: TextStyle(fontSize: 20),
            ),
            const SizedBox(height: 40),
            
            // Tombol ke Detail Page
            ElevatedButton.icon(
              onPressed: () {
                // PUSH ke halaman Detail
                Navigator.push(
                  context,
                  MaterialPageRoute(
                    builder: (context) => const DetailPage(),
                  ),
                );
              },
              icon: const Icon(Icons.arrow_forward),
              label: const Text('Ke Detail Page'),
              style: ElevatedButton.styleFrom(
                padding: const EdgeInsets.symmetric(
                  horizontal: 30,
                  vertical: 15,
                ),
              ),
            ),
            const SizedBox(height: 16),
            
            // Tombol ke Profile Page
            ElevatedButton.icon(
              onPressed: () {
                // PUSH ke halaman Profile
                Navigator.push(
                  context,
                  MaterialPageRoute(
                    builder: (context) => const ProfilePage(),
                  ),
                );
              },
              icon: const Icon(Icons.person),
              label: const Text('Ke Profile Page'),
              style: ElevatedButton.styleFrom(
                padding: const EdgeInsets.symmetric(
                  horizontal: 30,
                  vertical: 15,
                ),
              ),
            ),
          ],
        ),
      ),
    );
  }
}

// ==================== DETAIL PAGE ====================
class DetailPage extends StatelessWidget {
  const DetailPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Detail'),
        backgroundColor: Colors.green,
        foregroundColor: Colors.white,
        // Tombol back otomatis muncul!
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Icon(
              Icons.article,
              size: 80,
              color: Colors.green,
            ),
            const SizedBox(height: 20),
            const Text(
              'Ini adalah Detail Page',
              style: TextStyle(fontSize: 20),
            ),
            const SizedBox(height: 40),
            
            // Tombol kembali (manual)
            ElevatedButton.icon(
              onPressed: () {
                // POP - kembali ke halaman sebelumnya
                Navigator.pop(context);
              },
              icon: const Icon(Icons.arrow_back),
              label: const Text('Kembali'),
              style: ElevatedButton.styleFrom(
                backgroundColor: Colors.green,
                foregroundColor: Colors.white,
              ),
            ),
          ],
        ),
      ),
    );
  }
}

// ==================== PROFILE PAGE ====================
class ProfilePage extends StatelessWidget {
  const ProfilePage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Profile'),
        backgroundColor: Colors.purple,
        foregroundColor: Colors.white,
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const CircleAvatar(
              radius: 50,
              backgroundColor: Colors.purple,
              child: Icon(Icons.person, size: 60, color: Colors.white),
            ),
            const SizedBox(height: 20),
            const Text(
              'Ini adalah Profile Page',
              style: TextStyle(fontSize: 20),
            ),
            const SizedBox(height: 10),
            const Text(
              'John Doe',
              style: TextStyle(
                fontSize: 24,
                fontWeight: FontWeight.bold,
              ),
            ),
            const SizedBox(height: 40),
            
            // Tombol ke Detail dari Profile
            ElevatedButton.icon(
              onPressed: () {
                Navigator.push(
                  context,
                  MaterialPageRoute(
                    builder: (context) => const DetailPage(),
                  ),
                );
              },
              icon: const Icon(Icons.arrow_forward),
              label: const Text('Ke Detail Page'),
            ),
            const SizedBox(height: 16),
            
            ElevatedButton.icon(
              onPressed: () {
                Navigator.pop(context);
              },
              icon: const Icon(Icons.arrow_back),
              label: const Text('Kembali ke Home'),
              style: ElevatedButton.styleFrom(
                backgroundColor: Colors.purple,
                foregroundColor: Colors.white,
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

---

## Penjelasan Kode:

### 1. Navigator.push - Pindah ke halaman baru
```dart
Navigator.push(
  context,
  MaterialPageRoute(
    builder: (context) => const DetailPage(),
  ),
);
```
- `context` = konteks widget saat ini
- `MaterialPageRoute` = transisi halaman default Material Design
- `builder` = fungsi yang mengembalikan widget halaman tujuan

### 2. Navigator.pop - Kembali ke halaman sebelumnya
```dart
Navigator.pop(context);
```
- Menghapus halaman paling atas dari stack
- Kembali ke halaman sebelumnya

---

## Ilustrasi Alur:

```
Awal:
┌─────────────┐
│    Home     │
└─────────────┘

Setelah push DetailPage:
┌─────────────┐
│   Detail    │ ← Halaman aktif
├─────────────┤
│    Home     │
└─────────────┘

Setelah pop:
┌─────────────┐
│    Home     │ ← Kembali ke Home
└─────────────┘

Push ProfilePage, lalu push DetailPage:
┌─────────────┐
│   Detail    │ ← Halaman aktif
├─────────────┤
│   Profile   │
├─────────────┤
│    Home     │
└─────────────┘
```

---

## Hasil ketika di-run:

1. **Home Page** muncul pertama dengan 2 tombol
2. Klik "Ke Detail Page" → pindah ke halaman Detail (hijau)
3. Klik "Kembali" atau tombol back → kembali ke Home
4. Klik "Ke Profile Page" → pindah ke halaman Profile (ungu)
5. Dari Profile bisa ke Detail atau kembali ke Home

---

## Navigasi dengan Kirim Data

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
      home: const ProductListPage(),
    );
  }
}

// Halaman List Produk
class ProductListPage extends StatelessWidget {
  const ProductListPage({super.key});

  @override
  Widget build(BuildContext context) {
    final products = [
      {'name': 'iPhone 15', 'price': 15000000},
      {'name': 'Samsung Galaxy S24', 'price': 12000000},
      {'name': 'Pixel 8', 'price': 10000000},
    ];

    return Scaffold(
      appBar: AppBar(title: const Text('Daftar Produk')),
      body: ListView.builder(
        itemCount: products.length,
        itemBuilder: (context, index) {
          final product = products[index];
          return ListTile(
            leading: const Icon(Icons.phone_android),
            title: Text(product['name'] as String),
            subtitle: Text('Rp ${product['price']}'),
            trailing: const Icon(Icons.arrow_forward_ios),
            onTap: () {
              // Kirim data ke halaman detail
              Navigator.push(
                context,
                MaterialPageRoute(
                  builder: (context) => ProductDetailPage(
                    name: product['name'] as String,
                    price: product['price'] as int,
                  ),
                ),
              );
            },
          );
        },
      ),
    );
  }
}

// Halaman Detail Produk (menerima data)
class ProductDetailPage extends StatelessWidget {
  final String name;
  final int price;

  const ProductDetailPage({
    super.key,
    required this.name,
    required this.price,
  });

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text(name)),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Icon(Icons.phone_android, size: 100, color: Colors.blue),
            const SizedBox(height: 20),
            Text(
              name,
              style: const TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 10),
            Text(
              'Rp $price',
              style: const TextStyle(fontSize: 20, color: Colors.green),
            ),
            const SizedBox(height: 30),
            ElevatedButton(
              onPressed: () {},
              child: const Text('Beli Sekarang'),
            ),
          ],
        ),
      ),
    );
  }
}
```

### Cara Kirim Data:
```dart
Navigator.push(
  context,
  MaterialPageRoute(
    builder: (context) => ProductDetailPage(
      name: 'iPhone 15',      // Data yang dikirim
      price: 15000000,        // Data yang dikirim
    ),
  ),
);
```

### Cara Terima Data:
```dart
class ProductDetailPage extends StatelessWidget {
  final String name;   // Variabel untuk terima data
  final int price;     // Variabel untuk terima data

  const ProductDetailPage({
    required this.name,   // Required parameter
    required this.price,  // Required parameter
  });
  
  // Gunakan di build()
  Text(name),
  Text('Rp $price'),
}
```

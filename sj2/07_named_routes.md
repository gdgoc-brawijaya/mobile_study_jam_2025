# Named Routes (Bonus)

## Apa itu Named Routes?

Named Routes adalah cara navigasi dengan menggunakan **nama string** untuk setiap halaman, bukan langsung memanggil widget-nya.

### Perbedaan:

```dart
// Tanpa Named Routes (langsung)
Navigator.push(
  context,
  MaterialPageRoute(builder: (context) => DetailPage()),
);

// Dengan Named Routes (pakai nama)
Navigator.pushNamed(context, '/detail');
```

---

## Kapan Pakai Named Routes?

✅ **Gunakan Named Routes ketika:**
- Aplikasi punya banyak halaman
- Navigasi dari berbagai tempat ke halaman yang sama
- Ingin navigasi lebih terpusat dan rapi

❌ **Tidak perlu Named Routes ketika:**
- Aplikasi sederhana (< 5 halaman)
- Masih belajar Flutter dasar

---

## Contoh Lengkap Named Routes

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
      title: 'Named Routes Demo',
      
      // Halaman awal
      initialRoute: '/',
      
      // Daftar semua routes
      routes: {
        '/': (context) => const HomePage(),
        '/detail': (context) => const DetailPage(),
        '/profile': (context) => const ProfilePage(),
        '/settings': (context) => const SettingsPage(),
      },
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
            const Icon(Icons.home, size: 80, color: Colors.blue),
            const SizedBox(height: 20),
            const Text('Home Page', style: TextStyle(fontSize: 24)),
            const SizedBox(height: 40),
            
            _navButton(context, 'Ke Detail', '/detail', Icons.article),
            const SizedBox(height: 10),
            _navButton(context, 'Ke Profile', '/profile', Icons.person),
            const SizedBox(height: 10),
            _navButton(context, 'Ke Settings', '/settings', Icons.settings),
          ],
        ),
      ),
    );
  }

  Widget _navButton(
    BuildContext context,
    String text,
    String route,
    IconData icon,
  ) {
    return ElevatedButton.icon(
      onPressed: () {
        // Navigasi dengan Named Route
        Navigator.pushNamed(context, route);
      },
      icon: Icon(icon),
      label: Text(text),
      style: ElevatedButton.styleFrom(
        padding: const EdgeInsets.symmetric(horizontal: 30, vertical: 15),
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
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Icon(Icons.article, size: 80, color: Colors.green),
            const SizedBox(height: 20),
            const Text('Detail Page', style: TextStyle(fontSize: 24)),
            const SizedBox(height: 40),
            ElevatedButton(
              onPressed: () => Navigator.pop(context),
              child: const Text('Kembali'),
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
            const Text('Profile Page', style: TextStyle(fontSize: 24)),
            const SizedBox(height: 40),
            
            // Navigasi ke halaman lain dari sini
            ElevatedButton.icon(
              onPressed: () => Navigator.pushNamed(context, '/settings'),
              icon: const Icon(Icons.settings),
              label: const Text('Ke Settings'),
            ),
            const SizedBox(height: 10),
            ElevatedButton(
              onPressed: () => Navigator.pop(context),
              child: const Text('Kembali'),
            ),
          ],
        ),
      ),
    );
  }
}

// ==================== SETTINGS PAGE ====================
class SettingsPage extends StatelessWidget {
  const SettingsPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Settings'),
        backgroundColor: Colors.orange,
        foregroundColor: Colors.white,
      ),
      body: ListView(
        children: [
          ListTile(
            leading: const Icon(Icons.dark_mode),
            title: const Text('Dark Mode'),
            trailing: Switch(value: false, onChanged: (v) {}),
          ),
          ListTile(
            leading: const Icon(Icons.notifications),
            title: const Text('Notifications'),
            trailing: Switch(value: true, onChanged: (v) {}),
          ),
          ListTile(
            leading: const Icon(Icons.language),
            title: const Text('Language'),
            subtitle: const Text('Indonesia'),
            trailing: const Icon(Icons.arrow_forward_ios),
            onTap: () {},
          ),
          const Divider(),
          ListTile(
            leading: const Icon(Icons.logout, color: Colors.red),
            title: const Text('Logout', style: TextStyle(color: Colors.red)),
            onTap: () {
              // Kembali ke Home dan hapus semua halaman di stack
              Navigator.pushNamedAndRemoveUntil(
                context,
                '/',
                (route) => false,
              );
            },
          ),
        ],
      ),
    );
  }
}
```

---

## Navigasi Methods

### 1. pushNamed - Tambah halaman baru
```dart
Navigator.pushNamed(context, '/detail');
```

### 2. pop - Kembali
```dart
Navigator.pop(context);
```

### 3. pushReplacementNamed - Ganti halaman (tidak bisa back)
```dart
Navigator.pushReplacementNamed(context, '/home');
```
Contoh: Setelah login, ganti halaman Login dengan Home

### 4. pushNamedAndRemoveUntil - Hapus semua halaman sebelumnya
```dart
Navigator.pushNamedAndRemoveUntil(
  context,
  '/',
  (route) => false,  // Hapus semua route
);
```
Contoh: Setelah logout, kembali ke Home dan hapus history

---

## Named Routes dengan Arguments

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
      initialRoute: '/',
      routes: {
        '/': (context) => const ProductListPage(),
      },
      // Untuk halaman yang butuh arguments, pakai onGenerateRoute
      onGenerateRoute: (settings) {
        if (settings.name == '/product-detail') {
          final args = settings.arguments as Map<String, dynamic>;
          return MaterialPageRoute(
            builder: (context) => ProductDetailPage(
              name: args['name'],
              price: args['price'],
            ),
          );
        }
        return null;
      },
    );
  }
}

class ProductListPage extends StatelessWidget {
  const ProductListPage({super.key});

  @override
  Widget build(BuildContext context) {
    final products = [
      {'name': 'iPhone 15', 'price': 15000000},
      {'name': 'Samsung S24', 'price': 12000000},
    ];

    return Scaffold(
      appBar: AppBar(title: const Text('Produk')),
      body: ListView.builder(
        itemCount: products.length,
        itemBuilder: (context, index) {
          final product = products[index];
          return ListTile(
            title: Text(product['name'] as String),
            subtitle: Text('Rp ${product['price']}'),
            onTap: () {
              // Kirim arguments via Named Route
              Navigator.pushNamed(
                context,
                '/product-detail',
                arguments: {
                  'name': product['name'],
                  'price': product['price'],
                },
              );
            },
          );
        },
      ),
    );
  }
}

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
            Text(name, style: const TextStyle(fontSize: 24)),
            Text('Rp $price', style: const TextStyle(fontSize: 20)),
          ],
        ),
      ),
    );
  }
}
```

---

## Struktur Routes yang Rapi

Untuk project besar, pisahkan routes ke file sendiri:

```dart
// routes.dart
class AppRoutes {
  static const String home = '/';
  static const String detail = '/detail';
  static const String profile = '/profile';
  static const String settings = '/settings';
  static const String login = '/login';
  
  static Map<String, WidgetBuilder> get routes => {
    home: (context) => const HomePage(),
    detail: (context) => const DetailPage(),
    profile: (context) => const ProfilePage(),
    settings: (context) => const SettingsPage(),
    login: (context) => const LoginPage(),
  };
}

// main.dart
MaterialApp(
  initialRoute: AppRoutes.home,
  routes: AppRoutes.routes,
)

// Penggunaan
Navigator.pushNamed(context, AppRoutes.detail);
```

---

## Kesimpulan

| Metode | Kegunaan |
|--------|----------|
| `pushNamed` | Pindah ke halaman baru |
| `pop` | Kembali ke halaman sebelumnya |
| `pushReplacementNamed` | Ganti halaman (tidak bisa back) |
| `pushNamedAndRemoveUntil` | Ke halaman & hapus history |

**Untuk pemula:** Fokus dulu ke `Navigator.push` dan `Navigator.pop`. Named Routes bisa dipelajari nanti ketika aplikasi sudah makin kompleks! 🚀

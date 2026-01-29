# Best Practice Layout Flutter

## 1. Jangan Nesting Terlalu Dalam

### ❌ Kode Buruk (Terlalu Dalam):

```dart
Scaffold(
  body: Container(
    child: Column(
      children: [
        Container(
          child: Row(
            children: [
              Container(
                child: Column(
                  children: [
                    Container(
                      child: Row(
                        children: [
                          Container(
                            child: Text('Terlalu dalam!'),
                          ),
                        ],
                      ),
                    ),
                  ],
                ),
              ),
            ],
          ),
        ),
      ],
    ),
  ),
)
```

**Masalah:**
- Sulit dibaca
- Sulit di-maintain
- Sulit di-debug

---

### ✅ Kode Baik (Dipecah jadi Widget):

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
      home: const ProfilePage(),
    );
  }
}

// Halaman utama
class ProfilePage extends StatelessWidget {
  const ProfilePage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Profile')),
      body: const SingleChildScrollView(
        padding: EdgeInsets.all(16),
        child: Column(
          children: [
            ProfileHeader(),      // Widget terpisah
            SizedBox(height: 20),
            StatsSection(),       // Widget terpisah
            SizedBox(height: 20),
            ActionButtons(),      // Widget terpisah
          ],
        ),
      ),
    );
  }
}

// Widget 1: Profile Header
class ProfileHeader extends StatelessWidget {
  const ProfileHeader({super.key});

  @override
  Widget build(BuildContext context) {
    return const Column(
      children: [
        CircleAvatar(
          radius: 50,
          backgroundColor: Colors.blue,
          child: Icon(Icons.person, size: 50, color: Colors.white),
        ),
        SizedBox(height: 10),
        Text(
          'John Doe',
          style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
        ),
        Text(
          'Flutter Developer',
          style: TextStyle(color: Colors.grey),
        ),
      ],
    );
  }
}

// Widget 2: Stats Section
class StatsSection extends StatelessWidget {
  const StatsSection({super.key});

  @override
  Widget build(BuildContext context) {
    return const Row(
      mainAxisAlignment: MainAxisAlignment.spaceEvenly,
      children: [
        StatItem(label: 'Posts', value: '120'),
        StatItem(label: 'Followers', value: '5.2K'),
        StatItem(label: 'Following', value: '300'),
      ],
    );
  }
}

// Widget 3: Stat Item (reusable)
class StatItem extends StatelessWidget {
  final String label;
  final String value;

  const StatItem({
    super.key,
    required this.label,
    required this.value,
  });

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text(
          value,
          style: const TextStyle(
            fontSize: 20,
            fontWeight: FontWeight.bold,
          ),
        ),
        Text(
          label,
          style: const TextStyle(color: Colors.grey),
        ),
      ],
    );
  }
}

// Widget 4: Action Buttons
class ActionButtons extends StatelessWidget {
  const ActionButtons({super.key});

  @override
  Widget build(BuildContext context) {
    return Row(
      children: [
        Expanded(
          child: ElevatedButton(
            onPressed: () {},
            child: const Text('Edit Profile'),
          ),
        ),
        const SizedBox(width: 10),
        Expanded(
          child: OutlinedButton(
            onPressed: () {},
            child: const Text('Share'),
          ),
        ),
      ],
    );
  }
}
```

**Keuntungan:**
- Mudah dibaca
- Mudah di-maintain
- Widget bisa di-reuse
- Mudah di-test

---

## 2. Buat Widget Reusable

### Contoh: ProductCard yang Bisa Dipakai Ulang

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

class ProductListPage extends StatelessWidget {
  const ProductListPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Produk')),
      body: ListView(
        padding: const EdgeInsets.all(16),
        children: const [
          // Pakai ProductCard berulang kali
          ProductCard(
            name: 'iPhone 15 Pro',
            price: 18999000,
            imageUrl: 'https://via.placeholder.com/150',
            rating: 4.8,
          ),
          SizedBox(height: 16),
          ProductCard(
            name: 'Samsung Galaxy S24',
            price: 14999000,
            imageUrl: 'https://via.placeholder.com/150',
            rating: 4.6,
          ),
          SizedBox(height: 16),
          ProductCard(
            name: 'Google Pixel 8',
            price: 11999000,
            imageUrl: 'https://via.placeholder.com/150',
            rating: 4.5,
          ),
        ],
      ),
    );
  }
}

// Widget Reusable: ProductCard
class ProductCard extends StatelessWidget {
  final String name;
  final int price;
  final String imageUrl;
  final double rating;

  const ProductCard({
    super.key,
    required this.name,
    required this.price,
    required this.imageUrl,
    required this.rating,
  });

  @override
  Widget build(BuildContext context) {
    return Card(
      elevation: 2,
      shape: RoundedRectangleBorder(
        borderRadius: BorderRadius.circular(12),
      ),
      child: Padding(
        padding: const EdgeInsets.all(12),
        child: Row(
          children: [
            // Image placeholder
            Container(
              width: 80,
              height: 80,
              decoration: BoxDecoration(
                color: Colors.grey[200],
                borderRadius: BorderRadius.circular(8),
              ),
              child: const Icon(Icons.phone_android, size: 40),
            ),
            const SizedBox(width: 16),
            
            // Info
            Expanded(
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Text(
                    name,
                    style: const TextStyle(
                      fontSize: 16,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                  const SizedBox(height: 4),
                  Text(
                    'Rp ${_formatPrice(price)}',
                    style: const TextStyle(
                      fontSize: 14,
                      color: Colors.green,
                      fontWeight: FontWeight.w600,
                    ),
                  ),
                  const SizedBox(height: 4),
                  Row(
                    children: [
                      const Icon(Icons.star, size: 16, color: Colors.amber),
                      const SizedBox(width: 4),
                      Text('$rating'),
                    ],
                  ),
                ],
              ),
            ),
            
            // Action button
            IconButton(
              onPressed: () {},
              icon: const Icon(Icons.shopping_cart_outlined),
            ),
          ],
        ),
      ),
    );
  }

  String _formatPrice(int price) {
    return price.toString().replaceAllMapped(
      RegExp(r'(\d{1,3})(?=(\d{3})+(?!\d))'),
      (Match m) => '${m[1]}.',
    );
  }
}
```

---

## 3. Gunakan const Whenever Possible

```dart
// ❌ Tanpa const
Column(
  children: [
    Text('Hello'),
    SizedBox(height: 10),
    Icon(Icons.star),
  ],
)

// ✅ Dengan const (lebih efisien)
const Column(
  children: [
    Text('Hello'),
    SizedBox(height: 10),
    Icon(Icons.star),
  ],
)
```

**Keuntungan `const`:**
- Widget tidak perlu di-rebuild
- Hemat memori
- Performa lebih baik

---

## 4. Ekstrak Style ke Variabel/Theme

### ❌ Buruk: Style Hardcoded di Mana-mana

```dart
Text(
  'Judul',
  style: TextStyle(
    fontSize: 24,
    fontWeight: FontWeight.bold,
    color: Colors.blue,
  ),
)

// Di tempat lain...
Text(
  'Judul Lain',
  style: TextStyle(
    fontSize: 24,
    fontWeight: FontWeight.bold,
    color: Colors.blue,  // Copy paste 🙁
  ),
)
```

### ✅ Baik: Pakai Theme atau Konstanta

```dart
// Di main.dart
MaterialApp(
  theme: ThemeData(
    textTheme: const TextTheme(
      headlineLarge: TextStyle(
        fontSize: 24,
        fontWeight: FontWeight.bold,
        color: Colors.blue,
      ),
      bodyMedium: TextStyle(
        fontSize: 14,
        color: Colors.black87,
      ),
    ),
  ),
  home: const HomePage(),
)

// Penggunaan
Text(
  'Judul',
  style: Theme.of(context).textTheme.headlineLarge,
)
```

### Atau buat file constants:

```dart
// lib/constants/app_styles.dart
class AppStyles {
  static const TextStyle heading = TextStyle(
    fontSize: 24,
    fontWeight: FontWeight.bold,
    color: Colors.blue,
  );
  
  static const TextStyle body = TextStyle(
    fontSize: 14,
    color: Colors.black87,
  );
  
  static const EdgeInsets screenPadding = EdgeInsets.all(16);
}

// Penggunaan
Text('Judul', style: AppStyles.heading)
Padding(padding: AppStyles.screenPadding, ...)
```

---

## 5. Struktur Folder yang Rapi

```
lib/
├── main.dart
├── screens/           # Halaman-halaman
│   ├── home_screen.dart
│   ├── detail_screen.dart
│   └── profile_screen.dart
├── widgets/           # Widget reusable
│   ├── product_card.dart
│   ├── custom_button.dart
│   └── stat_item.dart
├── constants/         # Konstanta
│   ├── app_colors.dart
│   ├── app_styles.dart
│   └── app_strings.dart
└── routes/            # Navigasi
    └── app_routes.dart
```

---

## 6. Hindari Magic Numbers

```dart
// ❌ Magic numbers
Padding(padding: EdgeInsets.all(16), ...)
SizedBox(height: 24)
BorderRadius.circular(12)

// ✅ Pakai konstanta
class AppSpacing {
  static const double small = 8;
  static const double medium = 16;
  static const double large = 24;
}

class AppRadius {
  static const double small = 8;
  static const double medium = 12;
  static const double large = 16;
}

// Penggunaan
Padding(padding: EdgeInsets.all(AppSpacing.medium), ...)
SizedBox(height: AppSpacing.large)
BorderRadius.circular(AppRadius.medium)
```

---

## Checklist Best Practice

- [ ] Widget tidak lebih dari 3-4 level nesting
- [ ] Widget besar dipecah jadi widget kecil
- [ ] Widget yang dipakai berulang dijadikan reusable
- [ ] Gunakan `const` untuk widget statis
- [ ] Style dikelola via Theme atau constants
- [ ] Tidak ada magic numbers
- [ ] Folder terstruktur dengan baik
- [ ] Nama widget/class jelas dan deskriptif

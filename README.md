# WooCommerce Middleware

ميدلوير مستقل يخفي WordPress/WooCommerce ويوفر REST API نظيف.

## للعميل الجديد

### 1. نسخ المشروع
```bash
git clone https://github.com/your-company/wc-middleware.git client-name-middleware
cd client-name-middleware
npm install
```

### 2. إعداد متغيرات البيئة
انسخ `.env.example` إلى `.env` وعدّل القيم:

```env
WP_API_URL=https://client-wordpress.com
WC_CONSUMER_KEY=ck_xxxxxxxxxxxxx
WC_CONSUMER_SECRET=cs_xxxxxxxxxxxxx
JWT_SECRET=random-32-character-string
ALLOWED_ORIGINS=https://client-store.vercel.app
```

### 3. النشر على Vercel
```bash
vercel --prod
```

أو من لوحة Vercel:
1. Import Git Repository
2. أضف Environment Variables
3. Deploy

---

## نقاط النهاية (API Endpoints)

### المنتجات
| Method | Endpoint | الوصف |
|--------|----------|-------|
| GET | `/api/products` | قائمة المنتجات |
| GET | `/api/products/:slug` | منتج بالـ slug |
| GET | `/api/products?search=كلمة` | بحث |
| GET | `/api/products?category=id` | تصفية بفئة |

### الفئات
| Method | Endpoint | الوصف |
|--------|----------|-------|
| GET | `/api/categories` | قائمة الفئات |
| GET | `/api/categories/:slug` | فئة مع منتجاتها |

### المصادقة
| Method | Endpoint | Body | الوصف |
|--------|----------|------|-------|
| POST | `/api/auth/login` | `{username, password}` | تسجيل دخول |
| POST | `/api/auth/register` | `{username, email, password, companyName}` | إنشاء حساب |
| POST | `/api/auth/refresh` | `{refreshToken}` | تجديد token |

### الملف الشخصي (تتطلب token)
| Method | Endpoint | الوصف |
|--------|----------|-------|
| GET | `/api/user/profile` | عرض الملف |
| PUT | `/api/user/profile` | تحديث الملف |

### الطلبات (تتطلب token)
| Method | Endpoint | الوصف |
|--------|----------|-------|
| GET | `/api/orders` | قائمة طلبات المستخدم |
| GET | `/api/orders/:id` | تفاصيل طلب |
| POST | `/api/orders` | إنشاء طلب جديد |

---

## ربط الواجهة بالميدلوير

في الواجهة، أضف متغير البيئة:
```env
VITE_API_URL=https://client-middleware.vercel.app
```

استخدام في الكود:
```javascript
const API = import.meta.env.VITE_API_URL;

// جلب المنتجات
const res = await fetch(`${API}/api/products`);
const { products } = await res.json();

// تسجيل الدخول
const login = await fetch(`${API}/api/auth/login`, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ username, password })
});
const { accessToken, user } = await login.json();

// طلب محمي
const orders = await fetch(`${API}/api/orders`, {
  headers: { 'Authorization': `Bearer ${accessToken}` }
});
```

---

## السلة (Cart)

السلة تُدار في الواجهة (localStorage) لتجنب مشاكل Serverless. مثال:

```javascript
// حفظ السلة
localStorage.setItem('cart', JSON.stringify(cartItems));

// استرجاع السلة
const cart = JSON.parse(localStorage.getItem('cart') || '[]');

// عند الطلب، أرسل items للـ API
await fetch(`${API}/api/orders`, {
  method: 'POST',
  headers: { 
    'Content-Type': 'application/json',
    'Authorization': `Bearer ${token}`
  },
  body: JSON.stringify({
    items: cart.map(item => ({
      productId: item.id,
      quantity: item.quantity
    })),
    billing: { ... },
    paymentMethod: 'cod'
  })
});
```

---

## البنية

```
wc-middleware/
├── api/
│   └── index.ts          # نقطة دخول Vercel
├── src/
│   ├── index.ts          # Express app + routes
│   ├── config.ts         # إعدادات البيئة
│   ├── woocommerce.ts    # عميل WooCommerce
│   ├── id-mapper.ts      # تشفير IDs
│   ├── auth.ts           # JWT authentication
│   └── storage.ts        # WooCommerce Customers API
├── .env.example
├── package.json
├── tsconfig.json
└── vercel.json
```

---

## متطلبات WordPress

يجب تثبيت **JWT Authentication for WP REST API** plugin في WordPress:
1. من لوحة WordPress: Plugins → Add New
2. ابحث عن "JWT Authentication for WP REST API"
3. Install و Activate
4. أضف للـ wp-config.php:
```php
define('JWT_AUTH_SECRET_KEY', 'your-secret-key');
define('JWT_AUTH_CORS_ENABLE', true);
```

---

## الأمان

- ✅ جميع WordPress IDs مشفرة (XOR encoding)
- ✅ لا تظهر أي URLs أو metadata من WordPress
- ✅ JWT مع Access/Refresh tokens منفصلة
- ✅ المستخدمون يُحفظون في WooCommerce (دائم)
- ✅ التحقق من كلمة المرور عبر WordPress JWT
- ✅ CORS قابل للتكوين

---

## تطوير محلي

```bash
npm run dev
```

يعمل على http://localhost:3000

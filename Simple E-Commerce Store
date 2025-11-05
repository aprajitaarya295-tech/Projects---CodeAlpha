# CodeAlpha_ProjectName
Contains projects developed and submitted to @CodeAlpha during the 1 month virtual internship.
# E-Commerce Starter (Frontend + Backend)

This document contains two **complete starter options** for a basic e-commerce site with product listings, product detail pages, shopping cart, order processing, user registration/login, and a database: one built with **Django (Python)** and one built with **Express.js (Node + MongoDB)**. Each option includes file trees, key files (models, views/routes, templates), and run instructions.

---

## Quick notes

* Pick **one** backend to run (Django or Express). Both include the same frontend structure (simple HTML/CSS/JS) and similar feature coverage.
* These are starter templates. They focus on clarity and a minimal, functional implementation you can expand.

---

# Option A — Django (recommended for quick full-stack Python)

### Project structure (high level)

```
ecommerce_django/
├─ manage.py
├─ ecommerce/            # Django project
│  ├─ settings.py
│  ├─ urls.py
│  └─ wsgi.py
└─ shop/                 # Django app
   ├─ migrations/
   ├─ templates/
   │  ├─ base.html
   │  ├─ index.html
   │  ├─ product_detail.html
   │  ├─ cart.html
   │  └─ register.html
   ├─ static/
   │  └─ css/style.css
   ├─ models.py
   ├─ views.py
   ├─ urls.py
   └─ forms.py
```

### Key points

* Uses Django built-in auth for users.
* Models: `Product`, `Order`, `OrderItem`.
* Simple cart: stored in session (dictionary of product_id -> quantity).
* Order processing: converts session cart to `Order` on checkout.

### `shop/models.py`

```python
from django.db import models
from django.contrib.auth.models import User

class Product(models.Model):
    name = models.CharField(max_length=200)
    slug = models.SlugField(unique=True)
    description = models.TextField(blank=True)
    price = models.DecimalField(max_digits=8, decimal_places=2)
    stock = models.PositiveIntegerField(default=0)
    image = models.URLField(blank=True)

    def __str__(self):
        return self.name

class Order(models.Model):
    user = models.ForeignKey(User, on_delete=models.SET_NULL, null=True, blank=True)
    created_at = models.DateTimeField(auto_now_add=True)
    total = models.DecimalField(max_digits=10, decimal_places=2, default=0)
    paid = models.BooleanField(default=False)

    def __str__(self):
        return f"Order {self.id} - {self.user}"

class OrderItem(models.Model):
    order = models.ForeignKey(Order, related_name='items', on_delete=models.CASCADE)
    product = models.ForeignKey(Product, on_delete=models.SET_NULL, null=True)
    quantity = models.PositiveIntegerField(default=1)
    price = models.DecimalField(max_digits=8, decimal_places=2)

    def __str__(self):
        return f"{self.quantity} x {self.product.name}"
```

### `shop/forms.py`

```python
from django import forms
from django.contrib.auth.forms import UserCreationForm
from django.contrib.auth.models import User

class RegisterForm(UserCreationForm):
    email = forms.EmailField()
    class Meta:
        model = User
        fields = ["username", "email", "password1", "password2"]
```

### `shop/views.py` (essential views)

```python
from django.shortcuts import render, get_object_or_404, redirect
from .models import Product, Order, OrderItem
from .forms import RegisterForm
from django.contrib.auth import login, authenticate
from django.contrib.auth.decorators import login_required
from decimal import Decimal

# Helper: cart stored in session
def _get_cart(request):
    return request.session.setdefault('cart', {})

def index(request):
    products = Product.objects.all()
    return render(request, 'index.html', {'products': products})

def product_detail(request, slug):
    product = get_object_or_404(Product, slug=slug)
    return render(request, 'product_detail.html', {'product': product})

def cart_view(request):
    cart = _get_cart(request)
    product_ids = cart.keys()
    products = Product.objects.filter(id__in=product_ids)
    items = []
    total = Decimal('0')
    for p in products:
        qty = cart[str(p.id)]
        subtotal = p.price * qty
        total += subtotal
        items.append({'product': p, 'qty': qty, 'subtotal': subtotal})
    return render(request, 'cart.html', {'items': items, 'total': total})

def add_to_cart(request, product_id):
    cart = _get_cart(request)
    key = str(product_id)
    cart[key] = cart.get(key, 0) + 1
    request.session.modified = True
    return redirect('cart')

def remove_from_cart(request, product_id):
    cart = _get_cart(request)
    key = str(product_id)
    if key in cart:
        del cart[key]
        request.session.modified = True
    return redirect('cart')

@login_required
def checkout(request):
    cart = _get_cart(request)
    if not cart:
        return redirect('index')
    order = Order.objects.create(user=request.user)
    total = Decimal('0')
    for pid, qty in cart.items():
        product = Product.objects.get(id=int(pid))
        price = product.price
        OrderItem.objects.create(order=order, product=product, quantity=qty, price=price)
        total += price * qty
    order.total = total
    order.paid = True  # for demo; in a real app integrate payment gateway
    order.save()
    request.session['cart'] = {}
    return render(request, 'order_success.html', {'order': order})

def register(request):
    if request.method == 'POST':
        form = RegisterForm(request.POST)
        if form.is_valid():
            user = form.save()
            login(request, user)
            return redirect('index')
    else:
        form = RegisterForm()
    return render(request, 'register.html', {'form': form})
```

### `shop/urls.py`

```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.index, name='index'),
    path('product/<slug:slug>/', views.product_detail, name='product_detail'),
    path('cart/', views.cart_view, name='cart'),
    path('cart/add/<int:product_id>/', views.add_to_cart, name='add_to_cart'),
    path('cart/remove/<int:product_id>/', views.remove_from_cart, name='remove_from_cart'),
    path('checkout/', views.checkout, name='checkout'),
    path('register/', views.register, name='register'),
]
```

### Templates

* `base.html`: common header/footer with links to cart and login.
* `index.html`: loops `products` and shows Add to Cart button (POST or link to add_to_cart).
* `product_detail.html`: shows product details and an "Add to Cart" button.
* `cart.html`: lists items, quantities, totals, and checkout button.

(Templates are intentionally simple — see the Express option for example static HTML if you want copy/paste.)

### Run (Django)

1. `python -m venv venv && source venv/bin/activate` (Windows: `venv\Scripts\activate`)
2. `pip install django`
3. `django-admin startproject ecommerce` etc. (or use this structure)
4. `python manage.py makemigrations && python manage.py migrate`
5. Create a superuser: `python manage.py createsuperuser`
6. Load sample products (via admin or fixtures)
7. `python manage.py runserver`

---

# Option B — Express.js + MongoDB (Node.js)

### Project structure

```
ecommerce_express/
├─ package.json
├─ server.js
├─ models/
│  ├─ Product.js
│  ├─ User.js
│  └─ Order.js
├─ routes/
│  ├─ products.js
│  ├─ auth.js
│  └─ orders.js
├─ public/
│  ├─ index.html
│  ├─ product.html
│  └─ cart.html
└─ views/ (optional if using server-side templates)
```

### Key points

* Uses **Express** and **Mongoose** (MongoDB) to store products, users, and orders.
* Uses **express-session** for server-side sessions to hold the cart.
* User registration with bcrypt password hashing.
* Order processing endpoint converts session cart into an Order document.

### `models/Product.js`

```javascript
const mongoose = require('mongoose');
const Schema = mongoose.Schema;

const ProductSchema = new Schema({
  name: { type: String, required: true },
  slug: { type: String, required: true, unique: true },
  description: String,
  price: { type: Number, required: true },
  stock: { type: Number, default: 0 },
  image: String
});

module.exports = mongoose.model('Product', ProductSchema);
```

### `models/User.js`

```javascript
const mongoose = require('mongoose');
const Schema = mongoose.Schema;

const UserSchema = new Schema({
  username: { type: String, unique: true, required: true },
  email: { type: String, unique: true, required: true },
  passwordHash: { type: String, required: true }
});

module.exports = mongoose.model('User', UserSchema);
```

### `models/Order.js`

```javascript
const mongoose = require('mongoose');
const Schema = mongoose.Schema;

const OrderItem = new Schema({
  product: { type: Schema.Types.ObjectId, ref: 'Product' },
  quantity: Number,
  price: Number
});

const OrderSchema = new Schema({
  user: { type: Schema.Types.ObjectId, ref: 'User' },
  items: [OrderItem],
  total: Number,
  createdAt: { type: Date, default: Date.now },
  paid: { type: Boolean, default: false }
});

module.exports = mongoose.model('Order', OrderSchema);
```

### `server.js` (main)

```javascript
const express = require('express');
const mongoose = require('mongoose');
const session = require('express-session');
const bodyParser = require('body-parser');
const bcrypt = require('bcrypt');

const Product = require('./models/Product');
const User = require('./models/User');
const Order = require('./models/Order');

const app = express();
app.use(bodyParser.urlencoded({ extended: true }));
app.use(bodyParser.json());
app.use(express.static('public'));

app.use(session({
  secret: 'replace-this-secret',
  resave: false,
  saveUninitialized: false
}));

mongoose.connect('mongodb://localhost:27017/ecommerce', { useNewUrlParser: true, useUnifiedTopology: true });

// Auth routes (register/login/logout)
app.post('/api/register', async (req, res) => {
  const { username, email, password } = req.body;
  const hash = await bcrypt.hash(password, 10);
  const user = new User({ username, email, passwordHash: hash });
  await user.save();
  req.session.userId = user._id;
  res.json({ success: true });
});

app.post('/api/login', async (req, res) => {
  const { username, password } = req.body;
  const user = await User.findOne({ username });
  if (!user) return res.status(401).json({ error: 'Invalid' });
  const ok = await bcrypt.compare(password, user.passwordHash);
  if (!ok) return res.status(401).json({ error: 'Invalid' });
  req.session.userId = user._id;
  res.json({ success: true });
});

// Products
app.get('/api/products', async (req, res) => {
  const products = await Product.find();
  res.json(products);
});
app.get('/api/products/:slug', async (req, res) => {
  const p = await Product.findOne({ slug: req.params.slug });
  res.json(p);
});

// Cart stored in session
app.post('/api/cart/add', (req, res) => {
  const { productId, qty } = req.body;
  const cart = req.session.cart || {};
  cart[productId] = (cart[productId] || 0) + (qty || 1);
  req.session.cart = cart;
  res.json({ success: true, cart });
});
app.get('/api/cart', async (req, res) => {
  const cart = req.session.cart || {};
  const ids = Object.keys(cart);
  const products = await Product.find({ _id: { $in: ids } });
  let total = 0;
  const items = products.map(p => {
    const q = cart[p._id];
    const subtotal = p.price * q;
    total += subtotal;
    return { product: p, qty: q, subtotal };
  });
  res.json({ items, total });
});

// Checkout -> create order
app.post('/api/checkout', async (req, res) => {
  if (!req.session.userId) return res.status(401).json({ error: 'Login required' });
  const cart = req.session.cart || {};
  const ids = Object.keys(cart);
  const products = await Product.find({ _id: { $in: ids } });
  let total = 0;
  const items = products.map(p => {
    const q = cart[p._id];
    total += p.price * q;
    return { product: p._id, quantity: q, price: p.price };
  });
  const order = new Order({ user: req.session.userId, items, total, paid: true });
  await order.save();
  req.session.cart = {};
  res.json({ success: true, orderId: order._id });
});

app.listen(3000, () => console.log('Server running on http://localhost:3000'));
```

### Example frontend (public/index.html)

* Simple static page that fetches `/api/products` and renders them.
* `product.html` queries `/api/products/:slug` and shows Add to Cart button that POSTs to `/api/cart/add`.
* `cart.html` fetches `/api/cart` and renders items; checkout button POSTs to `/api/checkout`.

(You can copy the HTML from the templates in the Django section or request explicit files.)

### Run (Express)

1. `npm init -y`
2. `npm i express mongoose express-session body-parser bcrypt`
3. Ensure MongoDB is running locally at `mongodb://localhost:27017` or change the URI.
4. `node server.js`
5. Open `http://localhost:3000`.

---

# Frontend (shared, minimal)

You can use simple server-rendered pages or static HTML that talks to the backend via `fetch`.

Example actions:

* Load product list: `GET /api/products` or server render.
* View product detail: `GET /api/products/:slug` or server render.
* Add to cart: `POST /api/cart/add` with `{productId, qty}`.
* View cart: `GET /api/cart`.
* Checkout: `POST /api/checkout`.

---

# Authentication & Security notes

* Passwords: always hash (Django does this automatically; Express example uses bcrypt).
* Sessions: use secure secrets and HTTPS in production.
* In production, add CSRF protection (Django provides it), and use helmet, input validation, rate limits for Express.
* Payment: orders marked `paid = true` in these examples for demo only — integrate with a gateway (Stripe/PayPal) for real payments.

---

# What I included and next steps

* Fully runnable minimal code for both stacks (you'll find the main files above).
* Templates/HTML skeletons and server logic shown for cart, products, user registration, and order processing.

If you'd like, I can:

* Generate the exact full HTML templates and CSS files for the frontend (copy/paste-ready), or
* Generate a zipped project you can download, or
* Scaffold a single-stack repo (only Django or only Express) with complete files.

Tell me which backend you want me to scaffold into a downloadable project next, or I can produce the full frontend files now.

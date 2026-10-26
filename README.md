import React, { useState, useEffect } from 'react';
import { 
  ShoppingBag, Search, Sun, Moon, User, Tag, PlusCircle, 
  Trash2, Edit, CheckCircle, Clock, ShieldCheck, RefreshCw, 
  Menu, X, Filter, LogOut, ChevronRight, QrCode, CreditCard
} from 'lucide-react';

const INITIAL_PRODUCTS = [
  {
    id: 1,
    title: 'Nike Air Jordan 1 Retro High OG',
    category: 'Sneakers',
    brand: 'Nike',
    price: 6500,
    condition: 'new',
    size: 'US 10',
    description: 'สินค้าของแท้ 100% อุปกรณ์ครบกล่อง สภาพใหม่มือหนึ่ง พร้อมจัดส่งทันที',
    imageUrl: 'https://images.unsplash.com/photo-1552346154-21d32810aba3?auto=format&fit=crop&w=600&q=80',
    sellerId: 'vendor_1',
    sellerName: 'KICKR Official Store',
    rating: 4.9
  },
  {
    id: 2,
    title: 'Adidas Yeezy Boost 350 V2 Zebra',
    category: 'Sneakers',
    brand: 'Adidas',
    price: 4900,
    condition: 'used',
    size: 'US 9.5',
    description: 'สภาพ 95% ใส่น้อยมาก พื้นยังเต็ม มีรอยใช้งานทั่วไป อุปกรณ์ครบ',
    imageUrl: 'https://images.unsplash.com/photo-1584735935682-2f2b69dff9d2?auto=format&fit=crop&w=600&q=80',
    sellerId: 'vendor_2',
    sellerName: 'SneakerHead TH',
    rating: 4.7
  },
  {
    id: 3,
    title: 'Supreme Box Logo Heavyweight Hoodie',
    category: 'Streetwear',
    brand: 'Supreme',
    price: 12500,
    condition: 'new',
    size: 'L',
    description: 'เสื้อฮู้ดดี้ Supreme ของแท้จากช็อปสหรัฐอเมริกา ผ้าหนา นุ่ม อุ่น ทรงสวย',
    imageUrl: 'https://images.unsplash.com/photo-1556905055-8f358a7a47b2?auto=format&fit=crop&w=600&q=80',
    sellerId: 'vendor_1',
    sellerName: 'KICKR Official Store',
    rating: 4.9
  }
];

export default function App() {
  // DB States (LocalStorage Backup)
  const [products, setProducts] = useState(() => {
    const saved = localStorage.getItem('kickr_products');
    return saved ? JSON.parse(saved) : INITIAL_PRODUCTS;
  });

  const [orders, setOrders] = useState(() => {
    const saved = localStorage.getItem('kickr_orders');
    return saved ? JSON.parse(saved) : [];
  });

  const [user, setUser] = useState(() => {
    const saved = localStorage.getItem('kickr_user');
    return saved ? JSON.parse(saved) : null;
  });

  // UI States
  const [darkMode, setDarkMode] = useState(true);
  const [viewMode, setViewMode] = useState('buyer'); // 'buyer' | 'seller'
  const [cart, setCart] = useState([]);
  const [isCartOpen, setIsCartOpen] = useState(false);
  const [isAuthOpen, setIsAuthOpen] = useState(false);
  const [authTab, setAuthTab] = useState('login'); // 'login' | 'register'
  const [isAddProductOpen, setIsAddProductOpen] = useState(false);
  const [isCheckoutOpen, setIsCheckoutOpen] = useState(false);
  const [paymentMethod, setPaymentMethod] = useState('promptpay');
  const [mobileMenuOpen, setMobileMenuOpen] = useState(false);
  const [searchQuery, setSearchQuery] = useState('');
  const [selectedCategory, setSelectedCategory] = useState('All');
  const [selectedCondition, setSelectedCondition] = useState('all');

  // Form States
  const [loginForm, setLoginForm] = useState({ email: '', password: '' });
  const [regForm, setRegForm] = useState({ name: '', email: '', password: '', isVendor: true });
  const [newProduct, setNewProduct] = useState({
    title: '', category: 'Sneakers', brand: '', price: '', condition: 'new', size: '', description: '', imageUrl: ''
  });

  // Sync LocalStorage
  useEffect(() => {
    localStorage.setItem('kickr_products', JSON.stringify(products));
  }, [products]);

  useEffect(() => {
    localStorage.setItem('kickr_orders', JSON.stringify(orders));
  }, [orders]);

  useEffect(() => {
    if (user) localStorage.setItem('kickr_user', JSON.stringify(user));
    else localStorage.removeItem('kickr_user');
  }, [user]);

  // Auth Handlers
  const handleLogin = (e) => {
    e.preventDefault();
    const mockUser = {
      id: 'user_' + Date.now(),
      name: loginForm.email.split('@')[0] || 'Member User',
      email: loginForm.email,
      isVendor: true
    };
    setUser(mockUser);
    setIsAuthOpen(false);
  };

  const handleRegister = (e) => {
    e.preventDefault();
    const newUser = {
      id: 'user_' + Date.now(),
      name: regForm.name || 'New Vendor',
      email: regForm.email,
      isVendor: regForm.isVendor
    };
    setUser(newUser);
    setIsAuthOpen(false);
  };

  const handleLogout = () => {
    setUser(null);
    setViewMode('buyer');
  };

  // Add Product Handler
  const handleAddProduct = (e) => {
    e.preventDefault();
    if (!user) {
      setIsAuthOpen(true);
      return;
    }
    const productToAdd = {
      ...newProduct,
      id: Date.now(),
      price: Number(newProduct.price),
      sellerId: user.id,
      sellerName: user.name,
      rating: 5.0,
      imageUrl: newProduct.imageUrl || 'https://images.unsplash.com/photo-1542291026-7eec264c27ff?auto=format&fit=crop&w=600&q=80'
    };
    setProducts([productToAdd, ...products]);
    setIsAddProductOpen(false);
    setNewProduct({ title: '', category: 'Sneakers', brand: '', price: '', condition: 'new', size: '', description: '', imageUrl: '' });
  };

  // Cart & Checkout
  const addToCart = (product) => {
    setCart(prev => {
      const existing = prev.find(item => item.id === product.id);
      if (existing) {
        return prev.map(item => item.id === product.id ? { ...item, qty: item.qty + 1 } : item);
      }
      return [...prev, { ...product, qty: 1 }];
    });
    setIsCartOpen(true);
  };

  const handleCheckout = () => {
    if (!user) {
      setIsCartOpen(false);
      setIsAuthOpen(true);
      return;
    }
    const newOrder = {
      id: 'ORD-' + Date.now(),
      items: cart,
      total: cart.reduce((sum, item) => sum + item.price * item.qty, 0),
      buyerName: user.name,
      buyerId: user.id,
      paymentMethod,
      status: 'Paid',
      createdAt: new Date().toLocaleDateString('th-TH')
    };
    setOrders([newOrder, ...orders]);
    setCart([]);
    setIsCheckoutOpen(false);
    setIsCartOpen(false);
    alert('ชำระเงินสำเร็จ! คำสั่งซื้อของคุณได้รับการยืนยันแล้ว');
  };

  // Filter products
  const filteredProducts = products.filter(p => {
    const matchesSearch = p.title.toLowerCase().includes(searchQuery.toLowerCase()) || p.brand.toLowerCase().includes(searchQuery.toLowerCase());
    const matchesCategory = selectedCategory === 'All' || p.category === selectedCategory;
    const matchesCondition = selectedCondition === 'all' || p.condition === selectedCondition;
    return matchesSearch && matchesCategory && matchesCondition;
  });

  return (
    <div className={`min-h-screen font-sans ${darkMode ? 'bg-slate-950 text-slate-100' : 'bg-slate-50 text-slate-900'}`}>
      
      {/* HEADER */}
      <header className={`sticky top-0 z-40 backdrop-blur-md border-b ${darkMode ? 'bg-slate-900/80 border-slate-800' : 'bg-white/80 border-slate-200'}`}>
        <div className="max-w-7xl mx-auto px-4 h-16 flex items-center justify-between gap-4">
          
          {/* Logo */}
          <div className="flex items-center gap-2 cursor-pointer" onClick={() => setViewMode('buyer')}>
            <div className="w-10 h-10 bg-gradient-to-tr from-orange-500 to-amber-400 rounded-xl flex items-center justify-center font-black text-white text-xl shadow-lg shadow-orange-500/30">
              K
            </div>
            <span className="text-2xl font-black tracking-wider bg-gradient-to-r from-orange-500 via-amber-400 to-yellow-500 bg-clip-text text-transparent">
              KICKR
            </span>
          </div>

          {/* Search Bar */}
          <div className="hidden md:flex flex-1 max-w-md relative">
            <Search className="absolute left-3 top-1/2 -translate-y-1/2 text-slate-400 w-4 h-4" />
            <input 
              type="text" 
              placeholder="ค้นหารองเท้า เสื้อผ้า แบรนด์..." 
              value={searchQuery}
              onChange={(e) => setSearchQuery(e.target.value)}
              className={`w-full pl-9 pr-4 py-2 rounded-full text-sm outline-none border transition ${
                darkMode ? 'bg-slate-800 border-slate-700 focus:border-orange-500' : 'bg-slate-100 border-slate-300 focus:border-orange-500'
              }`}
            />
          </div>

          {/* Right Menu */}
          <div className="flex items-center gap-3">
            <button onClick={() => setDarkMode(!darkMode)} className="p-2 rounded-full hover:bg-slate-800/50 transition">
              {darkMode ? <Sun className="w-5 h-5 text-amber-400" /> : <Moon className="w-5 h-5 text-slate-600" />}
            </button>

            {user ? (
              <div className="flex items-center gap-2">
                <button 
                  onClick={() => setViewMode(viewMode === 'buyer' ? 'seller' : 'buyer')}
                  className="hidden sm:flex items-center gap-1.5 px-3 py-1.5 rounded-full text-xs font-semibold bg-orange-500/10 text-orange-500 border border-orange-500/20 hover:bg-orange-500/20 transition"
                >
                  {viewMode === 'buyer' ? 'แดชบอร์ดผู้ขาย' : 'โหมดช้อปปิ้ง'}
                </button>
                <div className="flex items-center gap-2 pl-2 border-l border-slate-700">
                  <div className="w-8 h-8 rounded-full bg-slate-800 flex items-center justify-center font-bold text-orange-400 border border-orange-500/30">
                    {user.name[0]?.toUpperCase()}
                  </div>
                  <button onClick={handleLogout} className="p-1 text-slate-400 hover:text-red-400">
                    <LogOut className="w-4 h-4" />
                  </button>
                </div>
              </div>
            ) : (
              <button 
                onClick={() => setIsAuthOpen(true)}
                className="px-4 py-2 rounded-full bg-gradient-to-r from-orange-500 to-amber-500 text-white font-medium text-sm shadow-md shadow-orange-500/20 hover:opacity-90 transition"
              >
                เข้าสู่ระบบ
              </button>
            )}

            {/* Cart Icon */}
            <button onClick={() => setIsCartOpen(true)} className="relative p-2 rounded-full hover:bg-slate-800/50 transition">
              <ShoppingBag className="w-5 h-5" />
              {cart.length > 0 && (
                <span className="absolute -top-1 -right-1 w-5 h-5 bg-orange-500 text-white text-xs font-bold rounded-full flex items-center justify-center">
                  {cart.reduce((a, b) => a + b.qty, 0)}
                </span>
              )}
            </button>
          </div>

        </div>
      </header>

      {/* MAIN CONTENT */}
      {viewMode === 'buyer' ? (
        <main className="max-w-7xl mx-auto px-4 py-6">
          
          {/* Hero Banner */}
          <div className="relative rounded-3xl overflow-hidden bg-gradient-to-r from-orange-600 via-amber-600 to-yellow-600 p-8 sm:p-12 mb-8 shadow-2xl">
            <div className="max-w-xl z-10 relative">
              <span className="px-3 py-1 rounded-full text-xs font-bold uppercase tracking-wider bg-black/30 text-white mb-4 inline-block">
                Streetwear & Sneakers Marketplace
              </span>
              <h1 className="text-3xl sm:text-5xl font-black text-white leading-tight mb-4">
                ตลาดออนไลน์ สำหรับสายสตรีทตัวจริง
              </h1>
              <p className="text-white/90 text-sm sm:text-base mb-6">
                ซื้อง่าย ขายสะดวก มั่นใจได้ด้วยระบบการชำระเงินที่ปลอดภัยและตรวจสอบได้ 100%
              </p>
              {user?.isVendor && (
                <button 
                  onClick={() => setIsAddProductOpen(true)}
                  className="px-6 py-3 rounded-full bg-black text-white font-bold hover:bg-slate-900 transition flex items-center gap-2 shadow-lg"
                >
                  <PlusCircle className="w-5 h-5 text-orange-400" />
                  ลงขายสินค้าของคุณ
                </button>
              )}
            </div>
          </div>

          {/* Filters & Grid */}
          <div className="flex flex-col md:flex-row gap-6">
            
            {/* Sidebar Filter */}
            <aside className="w-full md:w-64 space-y-6">
              <div className={`p-5 rounded-2xl border ${darkMode ? 'bg-slate-900 border-slate-800' : 'bg-white border-slate-200'}`}>
                <h3 className="font-bold text-lg mb-4 flex items-center gap-2">
                  <Filter className="w-4 h-4 text-orange-500" /> ตัวกรองสินค้า
                </h3>
                
                {/* Category Filter */}
                <div className="mb-4">
                  <label className="text-xs font-semibold uppercase text-slate-400 block mb-2">หมวดหมู่</label>
                  <div className="space-y-1">
                    {['All', 'Sneakers', 'Streetwear', 'Accessories'].map(cat => (
                      <button 
                        key={cat}
                        onClick={() => setSelectedCategory(cat)}
                        className={`w-full text-left px-3 py-1.5 rounded-lg text-sm transition ${
                          selectedCategory === cat ? 'bg-orange-500 text-white font-semibold' : 'text-slate-400 hover:bg-slate-800/40'
                        }`}
                      >
                        {cat}
                      </button>
                    ))}
                  </div>
                </div>

                {/* Condition Filter */}
                <div>
                  <label className="text-xs font-semibold uppercase text-slate-400 block mb-2">สภาพสินค้า</label>
                  <select 
                    value={selectedCondition} 
                    onChange={e => setSelectedCondition(e.target.value)}
                    className={`w-full p-2 rounded-lg text-sm outline-none border ${
                      darkMode ? 'bg-slate-800 border-slate-700' : 'bg-slate-100 border-slate-300'
                    }`}
                  >
                    <option value="all">ทั้งหมด</option>
                    <option value="new">มือหนึ่ง (New)</option>
                    <option value="used">มือสอง (Pre-owned)</option>
                  </select>
                </div>
              </div>
            </aside>

            {/* Product Grid */}
            <section className="flex-1">
              <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-6">
                {filteredProducts.map(product => (
                  <div 
                    key={product.id}
                    className={`group rounded-2xl border overflow-hidden transition hover:shadow-xl hover:-translate-y-1 ${
                      darkMode ? 'bg-slate-900 border-slate-800' : 'bg-white border-slate-200'
                    }`}
                  >
                    <div className="relative aspect-square overflow-hidden bg-slate-800">
                      <img 
                        src={product.imageUrl} 
                        alt={product.title} 
                        className="w-full h-full object-cover group-hover:scale-105 transition duration-300"
                      />
                      <span className={`absolute top-3 left-3 px-2.5 py-1 rounded-full text-xs font-bold text-white shadow-md ${
                        product.condition === 'new' ? 'bg-emerald-500' : 'bg-amber-500'
                      }`}>
                        {product.condition === 'new' ? 'มือหนึ่ง' : 'มือสอง'}
                      </span>
                    </div>

                    <div className="p-4 space-y-2">
                      <div className="text-xs font-semibold text-orange-500 uppercase">{product.brand}</div>
                      <h4 className="font-bold text-base line-clamp-1">{product.title}</h4>
                      <p className="text-xs text-slate-400">ไซส์: {product.size} | คนขาย: {product.sellerName}</p>
                      
                      <div className="pt-2 flex items-center justify-between border-t border-slate-800">
                        <span className="text-lg font-black text-amber-400">฿{product.price.toLocaleString()}</span>
                        <button 
                          onClick={() => addToCart(product)}
                          className="px-3 py-1.5 rounded-xl bg-orange-500 hover:bg-orange-600 text-white font-medium text-xs shadow-md shadow-orange-500/20 transition"
                        >
                          ใส่ตะกร้า
                        </button>
                      </div>
                    </div>
                  </div>
                ))}
              </div>
            </section>

          </div>

        </main>
      ) : (
        /* SELLER DASHBOARD */
        <main className="max-w-7xl mx-auto px-4 py-6">
          <div className="flex items-center justify-between mb-8">
            <div>
              <h2 className="text-2xl font-black">แดชบอร์ดผู้ขาย</h2>
              <p className="text-sm text-slate-400">จัดการสินค้าและรายการสั่งซื้อของคุณ</p>
            </div>
            <button 
              onClick={() => setIsAddProductOpen(true)}
              className="px-4 py-2 rounded-xl bg-orange-500 text-white font-bold text-sm flex items-center gap-2 shadow-lg"
            >
              <PlusCircle className="w-4 h-4" /> ลงขายสินค้าใหม่
            </button>
          </div>

          {/* Stats Cards */}
          <div className="grid grid-cols-1 sm:grid-cols-3 gap-4 mb-8">
            <div className={`p-6 rounded-2xl border ${darkMode ? 'bg-slate-900 border-slate-800' : 'bg-white border-slate-200'}`}>
              <div className="text-slate-400 text-xs font-semibold mb-1">สินค้าที่กำลังวางขาย</div>
              <div className="text-3xl font-black text-orange-500">
                {products.filter(p => p.sellerId === user?.id).length} ชิ้น
              </div>
            </div>
            <div className={`p-6 rounded-2xl border ${darkMode ? 'bg-slate-900 border-slate-800' : 'bg-white border-slate-200'}`}>
              <div className="text-slate-400 text-xs font-semibold mb-1">ยอดขายรวม</div>
              <div className="text-3xl font-black text-emerald-400">
                ฿{orders.reduce((acc, curr) => acc + curr.total, 0).toLocaleString()}
              </div>
            </div>
            <div className={`p-6 rounded-2xl border ${darkMode ? 'bg-slate-900 border-slate-800' : 'bg-white border-slate-200'}`}>
              <div className="text-slate-400 text-xs font-semibold mb-1">คำสั่งซื้อที่ต้องจัดส่ง</div>
              <div className="text-3xl font-black text-amber-400">
                {orders.length} รายการ
              </div>
            </div>
          </div>
        </main>
      )}

      {/* MODALS */}
      {/* AUTH MODAL */}
      {isAuthOpen && (
        <div className="fixed inset-0 z-50 bg-black/60 backdrop-blur-sm flex items-center justify-center p-4">
          <div className={`w-full max-w-md p-6 rounded-3xl border shadow-2xl ${darkMode ? 'bg-slate-900 border-slate-800' : 'bg-white border-slate-200'}`}>
            <div className="flex justify-between items-center mb-6">
              <h3 className="text-xl font-bold">{authTab === 'login' ? 'เข้าสู่ระบบ' : 'สมัครสมาชิก'}</h3>
              <button onClick={() => setIsAuthOpen(false)}><X className="w-5 h-5" /></button>
            </div>
            
            <form onSubmit={authTab === 'login' ? handleLogin : handleRegister} className="space-y-4">
              {authTab === 'register' && (
                <div>
                  <label className="text-xs font-semibold mb-1 block">ชื่อผู้ใช้งาน / ชื่อร้านค้า</label>
                  <input 
                    type="text" 
                    required 
                    value={regForm.name}
                    onChange={e => setRegForm({...regForm, name: e.target.value})}
                    className={`w-full p-2.5 rounded-xl text-sm border outline-none ${darkMode ? 'bg-slate-800 border-slate-700' : 'bg-slate-100 border-slate-300'}`}
                  />
                </div>
              )}
              <div>
                <label className="text-xs font-semibold mb-1 block">อีเมล</label>
                <input 
                  type="email" 
                  required 
                  value={loginForm.email}
                  onChange={e => setLoginForm({...loginForm, email: e.target.value})}
                  className={`w-full p-2.5 rounded-xl text-sm border outline-none ${darkMode ? 'bg-slate-800 border-slate-700' : 'bg-slate-100 border-slate-300'}`}
                />
              </div>
              <div>
                <label className="text-xs font-semibold mb-1 block">รหัสผ่าน</label>
                <input 
                  type="password" 
                  required 
                  className={`w-full p-2.5 rounded-xl text-sm border outline-none ${darkMode ? 'bg-slate-800 border-slate-700' : 'bg-slate-100 border-slate-300'}`}
                />
              </div>

              <button className="w-full py-3 rounded-xl bg-orange-500 font-bold text-white shadow-lg hover:bg-orange-600 transition">
                {authTab === 'login' ? 'เข้าสู่ระบบ' : 'สมัครสมาชิก'}
              </button>
            </form>

            <div className="mt-4 text-center text-xs text-slate-400">
              {authTab === 'login' ? 'ยังไม่มีบัญชี?' : 'มีบัญชีอยู่แล้ว?'} {' '}
              <button 
                onClick={() => setAuthTab(authTab === 'login' ? 'register' : 'login')}
                className="text-orange-500 font-bold underline"
              >
                {authTab === 'login' ? 'สมัครสมาชิก' : 'เข้าสู่ระบบ'}
              </button>
            </div>
          </div>
        </div>
      )}

      {/* ADD PRODUCT MODAL */}
      {isAddProductOpen && (
        <div className="fixed inset-0 z-50 bg-black/60 backdrop-blur-sm flex items-center justify-center p-4">
          <div className={`w-full max-w-lg p-6 rounded-3xl border shadow-2xl max-h-[90vh] overflow-y-auto ${darkMode ? 'bg-slate-900 border-slate-800' : 'bg-white border-slate-200'}`}>
            <div className="flex justify-between items-center mb-6">
              <h3 className="text-xl font-bold">ลงขายสินค้าใหม่</h3>
              <button onClick={() => setIsAddProductOpen(false)}><X className="w-5 h-5" /></button>
            </div>

            <form onSubmit={handleAddProduct} className="space-y-4">
              <div>
                <label className="text-xs font-semibold mb-1 block">ชื่อสินค้า</label>
                <input 
                  type="text" required 
                  value={newProduct.title}
                  onChange={e => setNewProduct({...newProduct, title: e.target.value})}
                  className={`w-full p-2.5 rounded-xl text-sm border outline-none ${darkMode ? 'bg-slate-800 border-slate-700' : 'bg-slate-100 border-slate-300'}`}
                />
              </div>
              <div className="grid grid-cols-2 gap-4">
                <div>
                  <label className="text-xs font-semibold mb-1 block">หมวดหมู่</label>
                  <select 
                    value={newProduct.category}
                    onChange={e => setNewProduct({...newProduct, category: e.target.value})}
                    className={`w-full p-2.5 rounded-xl text-sm border outline-none ${darkMode ? 'bg-slate-800 border-slate-700' : 'bg-slate-100 border-slate-300'}`}
                  >
                    <option value="Sneakers">Sneakers</option>
                    <option value="Streetwear">Streetwear</option>
                    <option value="Accessories">Accessories</option>
                  </select>
                </div>
                <div>
                  <label className="text-xs font-semibold mb-1 block">ราคา (บาท)</label>
                  <input 
                    type="number" required 
                    value={newProduct.price}
                    onChange={e => setNewProduct({...newProduct, price: e.target.value})}
                    className={`w-full p-2.5 rounded-xl text-sm border outline-none ${darkMode ? 'bg-slate-800 border-slate-700' : 'bg-slate-100 border-slate-300'}`}
                  />
                </div>
              </div>
              <div>
                <label className="text-xs font-semibold mb-1 block">URL รูปภาพสินค้า</label>
                <input 
                  type="url" required 
                  placeholder="https://images.unsplash.com/..."
                  value={newProduct.imageUrl}
                  onChange={e => setNewProduct({...newProduct, imageUrl: e.target.value})}
                  className={`w-full p-2.5 rounded-xl text-sm border outline-none ${darkMode ? 'bg-slate-800 border-slate-700' : 'bg-slate-100 border-slate-300'}`}
                />
              </div>
              <button className="w-full py-3 rounded-xl bg-orange-500 font-bold text-white shadow-lg hover:bg-orange-600 transition">
                ลงขายสินค้า
              </button>
            </form>
          </div>
        </div>
      )}

      {/* CART DRAWER */}
      {isCartOpen && (
        <div className="fixed inset-0 z-50 bg-black/60 backdrop-blur-sm flex justify-end">
          <div className={`w-full max-w-md h-full p-6 border-l flex flex-col ${darkMode ? 'bg-slate-900 border-slate-800' : 'bg-white border-slate-200'}`}>
            <div className="flex justify-between items-center mb-6">
              <h3 className="text-xl font-bold flex items-center gap-2">
                <ShoppingBag className="w-5 h-5 text-orange-500" /> ตะกร้าสินค้า
              </h3>
              <button onClick={() => setIsCartOpen(false)}><X className="w-5 h-5" /></button>
            </div>

            <div className="flex-1 overflow-y-auto space-y-4">
              {cart.map(item => (
                <div key={item.id} className="flex gap-4 p-3 rounded-xl bg-slate-800/40 border border-slate-800">
                  <img src={item.imageUrl} className="w-16 h-16 rounded-lg object-cover" />
                  <div className="flex-1">
                    <h5 className="font-bold text-sm line-clamp-1">{item.title}</h5>
                    <div className="text-xs text-orange-400 font-bold">฿{item.price.toLocaleString()}</div>
                    <div className="text-xs text-slate-400">จำนวน: {item.qty}</div>
                  </div>
                </div>
              ))}
            </div>

            <div className="pt-4 border-t border-slate-800 space-y-4">
              <div className="flex justify-between font-bold text-lg">
                <span>ราคารวมทั้งหมด</span>
                <span className="text-amber-400">
                  ฿{cart.reduce((sum, item) => sum + item.price * item.qty, 0).toLocaleString()}
                </span>
              </div>
              <button 
                onClick={handleCheckout}
                disabled={cart.length === 0}
                className="w-full py-3 rounded-xl bg-gradient-to-r from-orange-500 to-amber-500 font-bold text-white shadow-lg disabled:opacity-50"
              >
                ดำเนินการชำระเงิน
              </button>
            </div>
          </div>
        </div>
      )}

    </div>
  );
}

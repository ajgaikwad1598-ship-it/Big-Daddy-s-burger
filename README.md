<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no" />
  <meta name="apple-mobile-web-app-capable" content="yes" />
  <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent" />
  <meta name="apple-mobile-web-app-title" content="Big Daddy's POS" />
  <title>BIG DADDY'S Burger — POS & Billing</title>
  <style>
    :root {
      --bg: #f4f1ea;
      --surface: #ffffff;
      --primary: #b91c1c;
      --primary-dark: #7f1d1d;
      --accent: #15803d;
      --text: #1c1917;
      --muted: #57534e;
      --border: #e7e5e4;
    }
    * { box-sizing: border-box; margin: 0; padding: 0; font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif; }
    body { background: var(--bg); color: var(--text); height: 100vh; display: flex; flex-direction: column; overflow: hidden; }

    /* Top Header */
    header {
      background: var(--primary-dark); color: #fff; padding: 12px 20px;
      display: flex; justify-content: space-between; align-items: center;
      box-shadow: 0 2px 6px rgba(0,0,0,0.15);
    }
    header h1 { font-size: 1.25rem; letter-spacing: 0.5px; }
    header span { font-size: 0.8rem; opacity: 0.85; }
    .nav-tabs button {
      background: rgba(255,255,255,0.15); color: #fff; border: none;
      padding: 8px 14px; border-radius: 6px; margin-left: 8px; cursor: pointer; font-weight: 600;
    }
    .nav-tabs button.active { background: #fff; color: var(--primary-dark); }

    /* Main Split Layout */
    .workspace { display: flex; flex: 1; overflow: hidden; }
    .menu-pane { flex: 1.6; padding: 16px; overflow-y: auto; }
    .cart-pane {
      flex: 1; max-width: 420px; background: var(--surface); border-left: 1px solid var(--border);
      display: flex; flex-direction: column; padding: 16px;
    }

    /* Search & Categories */
    .controls { display: flex; gap: 10px; margin-bottom: 12px; }
    .controls input {
      flex: 1; padding: 10px 14px; border: 1px solid var(--border); border-radius: 8px; font-size: 0.95rem;
    }
    .categories { display: flex; gap: 8px; overflow-x: auto; padding-bottom: 10px; margin-bottom: 12px; }
    .cat-btn {
      padding: 8px 14px; border-radius: 20px; border: 1px solid var(--border);
      background: var(--surface); cursor: pointer; white-space: nowrap; font-weight: 600; font-size: 0.85rem;
    }
    .cat-btn.active { background: var(--primary); color: #fff; border-color: var(--primary); }

    /* Menu Grid */
    .menu-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(175px, 1fr)); gap: 12px; }
    .item-card {
      background: var(--surface); border: 1px solid var(--border); border-radius: 10px;
      padding: 12px; cursor: pointer; transition: transform 0.1s, box-shadow 0.1s;
      display: flex; flex-direction: column; justify-content: space-between;
    }
    .item-card:active { transform: scale(0.97); }
    .badge {
      display: inline-block; width: 12px; height: 12px; border-radius: 2px; margin-right: 6px;
      border: 1.5px solid currentColor; position: relative;
    }
    .badge::after {
      content: ""; width: 6px; height: 6px; border-radius: 50%; background: currentColor;
      position: absolute; top: 1.5px; left: 1.5px;
    }
    .veg { color: var(--accent); }
    .nonveg { color: var(--primary); }
    .item-name { font-weight: 600; font-size: 0.92rem; margin: 6px 0; }
    .item-sub { font-size: 0.75rem; color: var(--muted); margin-bottom: 8px; }
    .item-price { font-weight: 700; color: var(--primary-dark); font-size: 1rem; }

    /* Cart Section */
    .order-meta { display: flex; gap: 8px; margin-bottom: 12px; }
    .order-meta select, .order-meta input {
      flex: 1; padding: 8px; border: 1px solid var(--border); border-radius: 6px; font-size: 0.85rem;
    }
    .cart-items { flex: 1; overflow-y: auto; border-top: 1px solid var(--border); border-bottom: 1px solid var(--border); padding: 8px 0; }
    .cart-row { display: flex; justify-content: space-between; align-items: center; padding: 8px 0; border-bottom: 1px dashed var(--border); }
    .qty-controls button {
      width: 26px; height: 26px; border: 1px solid var(--border); background: var(--bg);
      border-radius: 4px; font-weight: bold; cursor: pointer;
    }
    .qty-controls span { margin: 0 8px; font-weight: 600; }

    /* Totals & Actions */
    .totals { padding: 12px 0; font-size: 0.9rem; }
    .total-line { display: flex; justify-content: space-between; margin-bottom: 6px; }
    .grand-total { font-size: 1.2rem; font-weight: 800; color: var(--primary-dark); border-top: 2px solid var(--border); padding-top: 8px; }
    .action-btns { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; }
    .btn { padding: 12px; border: none; border-radius: 8px; font-weight: 700; cursor: pointer; font-size: 0.95rem; }
    .btn-clear { background: #e7e5e4; color: var(--text); }
    .btn-pay { background: var(--accent); color: #fff; }

    /* Orders History View */
    .history-view { display: none; padding: 20px; overflow-y: auto; flex: 1; }
    table { width: 100%; border-collapse: collapse; background: #fff; border-radius: 8px; overflow: hidden; }
    th, td { padding: 12px; text-align: left; border-bottom: 1px solid var(--border); font-size: 0.9rem; }
    th { background: #f5f5f4; }

    /* Thermal Print Styling (80mm Receipt) */
    #receipt-area { display: none; }
    @media print {
      body * { visibility: hidden; }
      #receipt-area, #receipt-area * { visibility: visible; }
      #receipt-area {
        display: block; position: absolute; left: 0; top: 0; width: 76mm;
        font-family: "Courier New", monospace; font-size: 12px; color: #000;
      }
    }
  </style>
</head>
<body>

<header>
  <div>
    <h1>BIG DADDY'S BURGER</h1>
    <span>Shop No.177, Sector 11, Juhuagaon, Vashi, Navi Mumbai | Call: 7775022085</span>
  </div>
  <div class="nav-tabs">
    <button id="tab-pos" class="active" onclick="switchTab('pos')">POS Billing</button>
    <button id="tab-orders" onclick="switchTab('orders')">Sales History</button>
  </div>
</header>

<!-- POS VIEW -->
<div id="pos-view" class="workspace">
  <div class="menu-pane">
    <div class="controls">
      <input type="text" id="searchBox" placeholder="Search burgers, pizzas, combos, mojitos..." oninput="renderMenu()" />
    </div>
    <div class="categories" id="categoryBar"></div>
    <div class="menu-grid" id="menuGrid"></div>
  </div>

  <div class="cart-pane">
    <div class="order-meta">
      <select id="orderType">
        <option value="Dine-In">Dine-In</option>
        <option value="Takeaway">Takeaway</option>
        <option value="Delivery">Delivery</option>
      </select>
      <input type="text" id="tableOrCustomer" placeholder="Table # or Customer Name" />
    </div>

    <div class="cart-items" id="cartItems">
      <p style="text-align:center; color:#78716c; margin-top:40px;">Tap items on the left to start an order</p>
    </div>

    <div class="totals">
      <div class="total-line"><span>Subtotal</span><span id="subtotal">₹0</span></div>
      <div class="total-line">
        <span>Discount (₹)</span>
        <input type="number" id="discountInput" value="0" min="0" style="width:70px; text-align:right;" oninput="updateTotals()" />
      </div>
      <div class="total-line">
        <label><input type="checkbox" id="gstToggle" onchange="updateTotals()" /> Include 5% GST</label>
        <span id="gstAmount">₹0</span>
      </div>
      <div class="total-line grand-total"><span>Total Payable</span><span id="grandTotal">₹0</span></div>
    </div>

    <div class="action-btns">
      <button class="btn btn-clear" onclick="clearCart()">Clear</button>
      <button class="btn btn-pay" onclick="checkoutAndPrint()">Save & Print Bill</button>
    </div>
  </div>
</div>

<!-- SALES HISTORY VIEW -->
<div id="orders-view" class="history-view">
  <div style="display:flex; justify-content:space-between; align-items:center; margin-bottom:16px;">
    <h2>Completed Orders (<span id="totalRevenue">₹0</span> Total Sales)</h2>
    <button class="btn btn-clear" onclick="exportCSV()">Export CSV</button>
  </div>
  <table>
    <thead>
      <tr><th>Bill #</th><th>Time</th><th>Type / Info</th><th>Items</th><th>Payment</th><th>Total</th></tr>
    </thead>
    <tbody id="ordersTableBody"></tbody>
  </table>
</div>

<!-- HIDDEN 80MM RECEIPT TEMPLATE -->
<div id="receipt-area"></div>

<script>
  // Complete Menu Data extracted from Big Daddy's Burger Menu Cards
  const MENU_ITEMS = [
    // Chicken Burgers
    { id: 1, name: "Classic Chicken Burger", cat: "Chicken Burgers", price: 90, veg: false },
    { id: 2, name: "Kala Masala Chicken Burger", cat: "Chicken Burgers", price: 120, veg: false },
    { id: 3, name: "Nashville Fried Chicken Burger", cat: "Chicken Burgers", price: 130, veg: false },
    { id: 4, name: "Thecha Chicken Burger", cat: "Chicken Burgers", price: 130, veg: false },
    { id: 5, name: "BBQ Fried Chicken Burger", cat: "Chicken Burgers", price: 150, veg: false },
    { id: 6, name: "Devil's Breath Fried Chicken Burger", cat: "Chicken Burgers", price: 140, veg: false },
    { id: 7, name: "Inferno Fried Chicken Burger", cat: "Chicken Burgers", price: 120, veg: false },

    // Veg Burgers
    { id: 8, name: "Aloo Tikki Burger", cat: "Veg Burgers", price: 60, veg: true },
    { id: 9, name: "Veg Crispy Burger", cat: "Veg Burgers", price: 80, veg: true },
    { id: 10, name: "Paneer Burger", cat: "Veg Burgers", price: 120, veg: true },
    { id: 11, name: "Cheese Burger", cat: "Veg Burgers", price: 100, veg: true },
    { id: 12, name: "BBQ Veg Burger", cat: "Veg Burgers", price: 100, veg: true },

    // Combo Meals & Specials
    { id: 13, name: "#1 Classic Combo", sub: "Classic Chicken + Mint Mojito", cat: "Combos", price: 129, veg: false },
    { id: 14, name: "#2 Veg Saver Combo", sub: "Aloo Tikki + Lemon Mojito", cat: "Combos", price: 99, veg: true },
    { id: 15, name: "#3 Daddy's Special", sub: "Kala Masala + Mojito + Jamoon Shots", cat: "Combos", price: 179, veg: false },
    { id: 16, name: "#4 Crispy Treat", sub: "Veg Crispy + Strawberry/Blueberry Mojito", cat: "Combos", price: 159, veg: true },
    { id: 17, name: "#5 King Combo", sub: "BBQ Chicken + Cold Coffee + Jamoon Shots", cat: "Combos", price: 229, veg: false },
    { id: 18, name: "#6 Mastani Combo", sub: "Paneer Burger + Mango Mastani", cat: "Combos", price: 199, veg: true },
    { id: 19, name: "#7 Boba Blast", sub: "Cheese/Veg Crispy + Any Boba Drink", cat: "Combos", price: 199, veg: true },
    { id: 20, name: "#8 Spicy Fire Combo", sub: "Devil's Breath + Lemon Mojito + Jamoon", cat: "Combos", price: 219, veg: false },
    { id: 21, name: "#9 Mini Mafia Burgers", sub: "3 Mini Burgers Platter", cat: "Combos", price: 250, veg: false },
    { id: 22, name: "#10 Kurkure Burger", cat: "Combos", price: 160, veg: true },
    { id: 23, name: "#11 Crunchy Mojito Chicken Cup", cat: "Combos", price: 159, veg: false },
    { id: 24, name: "#12 Sizzler Burger", cat: "Combos", price: 199, veg: false },

    // Add-Ons
    { id: 25, name: "Add-On: Upgrade to Boba", cat: "Add-Ons", price: 40, veg: true },
    { id: 26, name: "Add-On: Jamoon Shots", cat: "Add-Ons", price: 20, veg: true },

    // Beverages
    { id: 27, name: "Green Apple Mojito", cat: "Beverages", price: 50, veg: true },
    { id: 28, name: "Guava Mojito", cat: "Beverages", price: 50, veg: true },
    { id: 29, name: "Pineapple Mojito", cat: "Beverages", price: 50, veg: true },
    { id: 30, name: "Hajma Mojito", cat: "Beverages", price: 50, veg: true },
    { id: 31, name: "Strawberry Mojito", cat: "Beverages", price: 50, veg: true },
    { id: 32, name: "Blueberry Mojito", cat: "Beverages", price: 50, veg: true },
    { id: 33, name: "Mint Mojito", cat: "Beverages", price: 50, veg: true },
    { id: 34, name: "Lemon Mojito (Sweet/Salty)", cat: "Beverages", price: 50, veg: true },
    { id: 35, name: "Jeera Mojito", cat: "Beverages", price: 50, veg: true },
    { id: 36, name: "Litchi Mojito", cat: "Beverages", price: 50, veg: true },
    { id: 37, name: "Jamoon Shots", cat: "Beverages", price: 30, veg: true },
    { id: 38, name: "Cold Coffee (Regular)", cat: "Beverages", price: 50, veg: true },
    { id: 39, name: "Cold Coffee (Large)", cat: "Beverages", price: 100, veg: true },
    { id: 40, name: "Cold Coffee with Boba", cat: "Beverages", price: 160, veg: true },
    { id: 41, name: "Cold Coffee with Ice Cream", cat: "Beverages", price: 100, veg: true },
    { id: 42, name: "Boba Drink", cat: "Beverages", price: 100, veg: true },
    { id: 43, name: "Mango Mastani (Regular)", cat: "Beverages", price: 80, veg: true },
    { id: 44, name: "Mango Mastani (Large)", cat: "Beverages", price: 120, veg: true },
    { id: 45, name: "Chocolate Mastani (Regular)", cat: "Beverages", price: 80, veg: true },
    { id: 46, name: "Chocolate Mastani (Large)", cat: "Beverages", price: 120, veg: true },

    // Pizzas
    { id: 47, name: "Volcano Pizza", cat: "Pizzas", price: 249, veg: true },
    { id: 48, name: "Cheese Burst Pizza", cat: "Pizzas", price: 279, veg: true },
    { id: 49, name: "Classic Cheese Pizza", cat: "Pizzas", price: 69, veg: true },
    { id: 50, name: "Corn Pizza", cat: "Pizzas", price: 89, veg: true },
    { id: 51, name: "Cheesy Cheese Pizza", cat: "Pizzas", price: 109, veg: true },
    { id: 52, name: "Paneer & Onion Capsicum Pizza", cat: "Pizzas", price: 119, veg: true },
    { id: 53, name: "Veg. Loaded Pizza", cat: "Pizzas", price: 139, veg: true },
    { id: 54, name: "Margherita Pizza (Reg)", cat: "Pizzas", price: 99, veg: true },
    { id: 55, name: "Margherita Pizza (Med)", cat: "Pizzas", price: 139, veg: true },
    { id: 56, name: "Single Toppings Pizza (Reg)", cat: "Pizzas", price: 109, veg: true },
    { id: 57, name: "Single Toppings Pizza (Med)", cat: "Pizzas", price: 149, veg: true },
    { id: 58, name: "Fresh Veggie Pizza (Reg)", cat: "Pizzas", price: 119, veg: true },
    { id: 59, name: "Fresh Veggie Pizza (Med)", cat: "Pizzas", price: 149, veg: true },
    { id: 60, name: "Country Pizza (Reg)", cat: "Pizzas", price: 129, veg: true },
    { id: 61, name: "Country Pizza (Med)", cat: "Pizzas", price: 159, veg: true },
    { id: 62, name: "Double Cheese Margherita (Reg)", cat: "Pizzas", price: 129, veg: true },
    { id: 63, name: "Double Cheese Margherita (Med)", cat: "Pizzas", price: 159, veg: true },
    { id: 64, name: "Paneer Pizza (Reg)", cat: "Pizzas", price: 139, veg: true },
    { id: 65, name: "Paneer Pizza (Med)", cat: "Pizzas", price: 179, veg: true },
    { id: 66, name: "Paneer Makhani Pizza (Reg)", cat: "Pizzas", price: 149, veg: true },
    { id: 67, name: "Paneer Makhani Pizza (Med)", cat: "Pizzas", price: 189, veg: true },
    { id: 68, name: "Special Paneer Pizza (Reg)", cat: "Pizzas", price: 149, veg: true },
    { id: 69, name: "Special Paneer Pizza (Med)", cat: "Pizzas", price: 189, veg: true },
    { id: 70, name: "Farmhouse Pizza (Reg)", cat: "Pizzas", price: 149, veg: true },
    { id: 71, name: "Farmhouse Pizza (Med)", cat: "Pizzas", price: 189, veg: true },
    { id: 72, name: "Peppy Paneer Pizza (Reg)", cat: "Pizzas", price: 159, veg: true },
    { id: 73, name: "Peppy Paneer Pizza (Med)", cat: "Pizzas", price: 199, veg: true },
    { id: 74, name: "Paradise Pizza (Reg)", cat: "Pizzas", price: 159, veg: true },
    { id: 75, name: "Paradise Pizza (Med)", cat: "Pizzas", price: 199, veg: true },
    { id: 76, name: "Mexican Green Pizza (Reg)", cat: "Pizzas", price: 159, veg: true },
    { id: 77, name: "Mexican Green Pizza (Med)", cat: "Pizzas", price: 199, veg: true },
    { id: 78, name: "Indi Tandoori Pizza (Reg)", cat: "Pizzas", price: 159, veg: true },
    { id: 79, name: "Indi Tandoori Pizza (Med)", cat: "Pizzas", price: 199, veg: true },
    { id: 80, name: "Veggie Blast Pizza (Reg)", cat: "Pizzas", price: 159, veg: true },
    { id: 81, name: "Veggie Blast Pizza (Med)", cat: "Pizzas", price: 199, veg: true },
    { id: 82, name: "Veg Extravaganza Pizza (Reg)", cat: "Pizzas", price: 159, veg: true },
    { id: 83, name: "Veg Extravaganza Pizza (Med)", cat: "Pizzas", price: 219, veg: true }
  ];

  let activeCategory = "All";
  let cart = [];
  let orders = JSON.parse(localStorage.getItem("bd_orders") || "[]");

  function init() {
    const categories = ["All", ...new Set(MENU_ITEMS.map(i => i.cat))];
    const bar = document.getElementById("categoryBar");
    bar.innerHTML = categories.map(c =>
      `<button class="cat-btn ${c === activeCategory ? 'active' : ''}" onclick="setCategory('${c}')">${c}</button>`
    ).join("");
    renderMenu();
    renderOrders();
  }

  function setCategory(cat) {
    activeCategory = cat;
    init();
  }

  function renderMenu() {
    const query = document.getElementById("searchBox").value.toLowerCase();
    const filtered = MENU_ITEMS.filter(item =>
      (activeCategory === "All" || item.cat === activeCategory) &&
      (item.name.toLowerCase().includes(query) || (item.sub && item.sub.toLowerCase().includes(query)))
    );

    document.getElementById("menuGrid").innerHTML = filtered.map(item => `
      <div class="item-card" onclick="addToCart(${item.id})">
        <div>
          <span class="badge ${item.veg ? 'veg' : 'nonveg'}"></span>
          <div class="item-name">${item.name}</div>
          ${item.sub ? `<div class="item-sub">${item.sub}</div>` : ""}
        </div>
        <div class="item-price">₹${item.price}</div>
      </div>
    `).join("");
  }

  function addToCart(id) {
    const item = MENU_ITEMS.find(i => i.id === id);
    const existing = cart.find(c => c.id === id);
    if (existing) existing.qty++;
    else cart.push({ ...item, qty: 1 });
    renderCart();
  }

  function changeQty(id, delta) {
    const idx = cart.findIndex(c => c.id === id);
    if (idx === -1) return;
    cart[idx].qty += delta;
    if (cart[idx].qty <= 0) cart.splice(idx, 1);
    renderCart();
  }

  function renderCart() {
    const container = document.getElementById("cartItems");
    if (cart.length === 0) {
      container.innerHTML = `<p style="text-align:center; color:#78716c; margin-top:40px;">Tap items on the left to start an order</p>`;
    } else {
      container.innerHTML = cart.map(item => `
        <div class="cart-row">
          <div style="flex:1; padding-right:8px;">
            <div style="font-weight:600; font-size:0.88rem;">${item.name}</div>
            <div style="font-size:0.78rem; color:#57534e;">₹${item.price} × ${item.qty} = ₹${item.price * item.qty}</div>
          </div>
          <div class="qty-controls">
            <button onclick="changeQty(${item.id}, -1)">-</button>
            <span>${item.qty}</span>
            <button onclick="changeQty(${item.id}, 1)">+</button>
          </div>
        </div>
      `).join("");
    }
    updateTotals();
  }

  function updateTotals() {
    const sub = cart.reduce((sum, i) => sum + i.price * i.qty, 0);
    const disc = Math.min(sub, Number(document.getElementById("discountInput").value) || 0);
    const taxable = sub - disc;
    const gst = document.getElementById("gstToggle").checked ? Math.round(taxable * 0.05) : 0;
    const grand = taxable + gst;

    document.getElementById("subtotal").innerText = `₹${sub}`;
    document.getElementById("gstAmount").innerText = `₹${gst}`;
    document.getElementById("grandTotal").innerText = `₹${grand}`;
    return { sub, disc, gst, grand };
  }

  function clearCart() {
    cart = [];
    document.getElementById("discountInput").value = 0;
    document.getElementById("tableOrCustomer").value = "";
    renderCart();
  }

  function checkoutAndPrint() {
    if (cart.length === 0) return alert("Please add items to the order first.");
    const { sub, disc, gst, grand } = updateTotals();
    const billNo = "BD-" + (1001 + orders.length);
    const time = new Date().toLocaleString("en-IN");
    const orderType = document.getElementById("orderType").value;
    const info = document.getElementById("tableOrCustomer").value || "Walk-in";

    const order = { billNo, time, orderType, info, items: [...cart], sub, disc, gst, grand };
    orders.unshift(order);
    localStorage.setItem("bd_orders", JSON.stringify(orders));

    // Populate 80mm Receipt
    document.getElementById("receipt-area").innerHTML = `
      <div style="text-align:center;">
        <strong>BIG DADDY'S BURGER</strong><br/>
        Shop No.177, Sector 11, Juhuagaon,<br/>
        Vashi, Navi Mumbai<br/>
        Call: 7775022085<br/>
        --------------------------------<br/>
        Bill: ${billNo} | ${orderType} (${info})<br/>
        ${time}<br/>
        --------------------------------
      </div>
      ${cart.map(i => `<div>${i.name}<br/>  ${i.qty} x ₹${i.price} = ₹${i.qty * i.price}</div>`).join("")}
      <div>
        --------------------------------<br/>
        Subtotal: ₹${sub}<br/>
        ${disc > 0 ? `Discount: -₹${disc}<br/>` : ""}
        ${gst > 0 ? `GST (5%): ₹${gst}<br/>` : ""}
        <strong>GRAND TOTAL: ₹${grand}</strong><br/>
        --------------------------------<br/>
        <div style="text-align:center;">Thank You! Visit Again</div>
      </div>
    `;

    renderOrders();
    window.print();
    clearCart();
  }

  function switchTab(tab) {
    document.getElementById("pos-view").style.display = tab === "pos" ? "flex" : "none";
    document.getElementById("orders-view").style.display = tab === "orders" ? "block" : "none";
    document.getElementById("tab-pos").classList.toggle("active", tab === "pos");
    document.getElementById("tab-orders").classList.toggle("active", tab === "orders");
  }

  function renderOrders() {
    const totalRev = orders.reduce((s, o) => s + o.grand, 0);
    document.getElementById("totalRevenue").innerText = `₹${totalRev}`;
    document.getElementById("ordersTableBody").innerHTML = orders.map(o => `
      <tr>
        <td><strong>${o.billNo}</strong></td>
        <td>${o.time}</td>
        <td>${o.orderType} (${o.info})</td>
        <td>${o.items.map(i => `${i.name} (x${i.qty})`).join(", ")}</td>
        <td>Paid</td>
        <td><strong>₹${o.grand}</strong></td>
      </tr>
    `).join("");
  }

  function exportCSV() {
    const rows = [["Bill No", "Time", "Type", "Customer/Table", "Total"]];
    orders.forEach(o => rows.push([o.billNo, `"${o.time}"`, o.orderType, `"${o.info}"`, o.grand]));
    const csv = rows.map(r => r.join(",")).join("\n");
    const blob = new Blob([csv], { type: "text/csv" });
    const a = document.createElement("a");
    a.href = URL.createObjectURL(blob);
    a.download = "Big_Daddys_Sales.csv";
    a.click();
  }

  init();
</script>
</body>
</html> 

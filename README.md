<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>古早亭線上點餐</title>
    <script src="https://static.line-scdn.net/liff/edge/2/sdk.js"></script>
    <style>
        body { font-family: "Microsoft JhengHei", -apple-system, sans-serif; background: #f8f9fa; margin: 0; padding: 15px; padding-bottom: 140px; }
        h2 { text-align: center; color: #2c3e50; margin-bottom: 5px; }
        p.subtitle { text-align: center; color: #7f8c8d; font-size: 0.9em; margin-bottom: 20px; }
        .category { display: block; background: #27ae60; color: white; padding: 6px 15px; border-radius: 20px; margin: 25px 0 10px 0; font-size: 0.95em; font-weight: bold; width: fit-content; }
        .item { background: white; padding: 15px; border-radius: 12px; margin-bottom: 12px; box-shadow: 0 2px 8px rgba(0,0,0,0.08); }
        .item-name { font-size: 1.1em; font-weight: bold; color: #34495e; margin-bottom: 12px; }
        .btn-group { display: flex; gap: 10px; flex-wrap: wrap; }
        button { border: none; border-radius: 8px; cursor: pointer; font-size: 0.95em; transition: 0.2s; padding: 12px; font-weight: bold; }
        .btn-l { background: #e8f6ef; color: #27ae60; border: 1.5px solid #27ae60; flex: 1; }
        .btn-xl { background: #27ae60; color: white; border: 1.5px solid #27ae60; flex: 1; }
        .btn-l:active, .btn-xl:active { opacity: 0.7; transform: scale(0.95); }
        
        /* 底部購物車 */
        .cart-bar { position: fixed; bottom: 0; left: 0; right: 0; background: white; padding: 15px 20px; box-shadow: 0 -5px 20px rgba(0,0,0,0.15); display: flex; justify-content: space-between; align-items: center; z-index: 100; border-radius: 20px 20px 0 0; }
        .total-info { display: flex; flex-direction: column; }
        .total-price { font-size: 1.4em; font-weight: bold; color: #e67e22; }
        .btn-submit { background: #e67e22; color: white; padding: 12px 25px; font-weight: bold; font-size: 1.1em; border-radius: 30px; border: none; }
        .btn-clear { background: #eee; color: #777; font-size: 0.8em; border-radius: 5px; padding: 5px 10px; margin-top: 5px; border: none; }
        
        /* 載入中遮罩 */
        .loading-mask { display:none; position:fixed; top:0; left:0; width:100%; height:100%; background:rgba(0,0,0,0.7); z-index:999; color:white; justify-content:center; align-items:center; flex-direction: column; }
    </style>
</head>
<body>

    <div id="loading" class="loading-mask">
        <div>傳送訂單中...</div>
    </div>

    <h2>古早亭</h2>
    <p class="subtitle">仙草與古早味茶飲專賣</p>

    <div id="menu-container"></div>

    <div class="cart-bar">
        <div class="total-info">
            <span style="font-size: 0.85em; color: #7f8c8d;">已選 <span id="count" style="color:#27ae60; font-weight:bold;">0</span> 杯</span>
            <span class="total-price">總計: $<span id="total">0</span></span>
            <button class="btn-clear" onclick="clearCart()">🗑️ 清空重選</button>
        </div>
        <button class="btn-submit" onclick="sendOrder()">確認送出</button>
    </div>

<script>
    const menuData = [
        { cat: "仙草嫩凍系列", items: [
            {n:"招牌仙草嫩凍", p:[40, 45]}, {n:"冬瓜仙草嫩凍", p:[35, 40]}, {n:"仙草拿鐵嫩凍", p:[55, 60]}, 
            {n:"冬瓜拿鐵嫩凍", p:[55, 60]}, {n:"古早亭紅茶拿鐵嫩凍", p:[55, 60]}, {n:"阿薩姆紅茶拿鐵嫩凍", p:[55, 60]},
            {n:"阿薩姆奶茶嫩凍", p:[55, 60]}, {n:"茉香綠拿鐵嫩凍", p:[55, 60]}, {n:"茉香綠奶茶嫩凍", p:[55, 60]}, {n:"嫩仙草凍奶", p:[0, 70]}
        ]},
        { cat: "古早亭一飲甘甜", items: [
            {n:"招牌仙草甘茶", p:[30, 35]}, {n:"招牌冬瓜茶", p:[25, 30]}, {n:"古早亭紅茶", p:[25, 30]}, 
            {n:"阿薩姆紅茶", p:[30, 35]}, {n:"茉香綠茶", p:[30, 35]}, {n:"四季青茶", p:[30, 35]}, 
            {n:"黃金烏龍茶", p:[30, 35]}, {n:"黃金麥茶", p:[30, 35]}
        ]},
        { cat: "奶茶 / 鮮奶茶系列", items: [
            {n:"仙草奶茶", p:[45, 50]}, {n:"阿薩姆奶茶", p:[45, 50]}, {n:"茉香綠奶茶", p:[45, 50]}, 
            {n:"珍珠奶茶/奶綠", p:[50, 55]}, {n:"焦糖奶茶", p:[0, 60]}, {n:"椰果奶茶", p:[55, 60]},
            {n:"紅茶拿鐵", p:[45, 50]}, {n:"冬瓜拿鐵", p:[45, 50]}, {n:"珍珠紅茶拿鐵", p:[50, 55]}
        ]},
        { cat: "手調風味 / 翡翠冰飲", items: [
            {n:"仙草冰鎮檸檬", p:[45, 50]}, {n:"冬瓜檸檬", p:[45, 50]}, {n:"凍檸紅茶", p:[45, 50]}, 
            {n:"檸檬青/綠茶", p:[45, 50]}, {n:"蜂蜜檸檬", p:[50, 55]}, {n:"蜂蜜檸檬愛玉", p:[60, 65]},
            {n:"冬瓜檸檬愛玉", p:[55, 60]}, {n:"甘蔗檸檬青茶", p:[0, 70]}
        ]}
    ];

    let cart = [];
    let totalPrice = 0;

    // 渲染選單
    const container = document.getElementById('menu-container');
    menuData.forEach(section => {
        let html = `<span class="category">${section.cat}</span>`;
        section.items.forEach(item => {
            html += `<div class="item"><div class="item-name">${item.n}</div><div class="btn-group">`;
            if(item.p[0] > 0) html += `<button class="btn-l" onclick="add('${item.n}(L)', ${item.p[0]})">L $${item.p[0]}</button>`;
            if(item.p[1] > 0) html += `<button class="btn-xl" onclick="add('${item.n}(XL)', ${item.p[1]})">XL $${item.p[1]}</button>`;
            html += `</div></div>`;
        });
        container.innerHTML += html;
    });

    // 初始化 LIFF
    liff.init({ liffId: "2009451030-e48JrFOA" }).then(() => {
        if (!liff.isLoggedIn()) liff.login();
    }).catch(err => console.error(err));

    function add(name, price) {
        const sweet = prompt("甜度？(例:微糖/無糖)", "微糖") || "正常";
        const ice = prompt("冰量？(例:少冰/去冰)", "微冰") || "正常";
        cart.push({name, price, sweet, ice});
        totalPrice += price;
        updateDisplay();
    }

    function clearCart() {
        if(confirm("確定要清空所有已選飲料嗎？")) {
            cart = [];
            totalPrice = 0;
            updateDisplay();
        }
    }

    function updateDisplay() {
        document.getElementById('total').innerText = totalPrice;
        document.getElementById('count').innerText = cart.length;
    }

    function sendOrder() {
        if (cart.length === 0) return alert("購物車空空的喔！");
        
        document.getElementById('loading').style.display = 'flex';

        let msg = "📝 古早亭新訂單\n------------------\n";
        cart.forEach((it, i) => {
            msg += `${i+1}. ${it.name} (${it.sweet}/${it.ice}) $${it.price}\n`;
        });
        msg += `------------------\n💰 總計金額：$${totalPrice}`;

        if (liff.isInClient()) {
            liff.sendMessages([{ type: 'text', text: msg }]).then(() => {
                alert("訂單已傳送成功！");
                liff.closeWindow();
            }).catch(err => {
                document.getElementById('loading').style.display = 'none';
                alert("傳送失敗，請截圖購物車。");
            });
        } else {
            document.getElementById('loading').style.display = 'none';
            alert("請在 LINE App 內開啟方可下單。");
            console.log(msg);
        }
    }
</script>
</body>
</html>

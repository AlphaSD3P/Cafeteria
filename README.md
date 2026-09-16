<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>스위치 기법 자동 퉁치기 시스템</title>
    
    <!-- Firebase SDK 불러오기 -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/10.8.0/firebase-app.js";
        import { getFirestore, doc, getDoc, setDoc } from "https://www.gstatic.com/firebasejs/10.8.0/firebase-firestore.js";
        import { getAuth, signInWithPopup, GoogleAuthProvider, signOut, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/10.8.0/firebase-auth.js";

        // ==========================================
        // [필수] 본인의 파이어베이스 설정값 입력
        // ==========================================
        const firebaseConfig = {
            apiKey: "AIzaSyDeULQ3tftmP7itfEmXYEdqSWorIOdlwrs",
            authDomain: "cafeteria-investing.firebaseapp.com",
            projectId: "cafeteria-investing",
            storageBucket: "cafeteria-investing.firebasestorage.app",
            messagingSenderId: "405486491315",
            appId: "1:405486491315:web:2bbabc9ac8cbcbaa3cc2c6"
        };

        const OWNER_EMAIL = "alphasd3p@gmail.com";

        const app = initializeApp(firebaseConfig);
        const db = getFirestore(app);
        const auth = getAuth(app);
        const provider = new GoogleAuthProvider();

        window.loginWithGoogle = async function() {
            try { await signInWithPopup(auth, provider); } catch (e) { alert("로그인 실패: " + e.message); }
        };
        window.logout = async function() { await signOut(auth); };

        onAuthStateChanged(auth, async (user) => {
            const loginScreen = document.getElementById('login-screen');
            const appScreen = document.getElementById('app-screen');
            if (user) {
                if (user.email !== OWNER_EMAIL) {
                    alert(`접근 권한이 없습니다!\n계정: ${user.email}`);
                    await signOut(auth);
                    return;
                }
                loginScreen.style.display = 'none';
                appScreen.style.display = 'block';
                document.getElementById('user-email').textContent = `접속 계정: ${user.email}`;
                await loadCloudData();
            } else {
                loginScreen.style.display = 'flex';
                appScreen.style.display = 'none';
            }
        });

        async function loadCloudData() {
            try {
                const docSnap = await getDoc(doc(db, "appData", "switchAutoState"));
                if (docSnap.exists()) {
                    let data = docSnap.data();
                    if(data.closePrice) document.getElementById('close-price').value = data.closePrice;
                    if(data.totalRounds) document.getElementById('total-rounds').value = data.totalRounds;
                    if(data.targetDropPct) document.getElementById('target-drop-pct').value = data.targetDropPct;
                    if(data.currentRound) document.getElementById('current-round').value = data.currentRound;
                    if(data.qtyPerRound) document.getElementById('qty-per-round').value = data.qtyPerRound;
                }
            } catch (e) { console.error("데이터 로드 실패:", e); }
            calculateSwitchOrders();
        }

        window.saveAndCalculate = async function() {
            calculateSwitchOrders();
            try {
                await setDoc(doc(db, "appData", "switchAutoState"), {
                    closePrice: document.getElementById('close-price').value,
                    totalRounds: document.getElementById('total-rounds').value,
                    targetDropPct: document.getElementById('target-drop-pct').value,
                    currentRound: document.getElementById('current-round').value,
                    qtyPerRound: document.getElementById('qty-per-round').value,
                    updatedAt: new Date().toISOString()
                });
            } catch (e) { console.error("클라우드 저장 실패:", e); }
        }
    </script>

    <style>
        * { box-sizing: border-box; }
        body { font-family: 'Malgun Gothic', sans-serif; background-color: #f4f5f7; display: flex; justify-content: center; align-items: center; min-height: 100vh; margin: 0; padding: 20px; color: #333; }
        #login-screen { display: flex; flex-direction: column; align-items: center; justify-content: center; background: #fff; padding: 40px; border-radius: 12px; box-shadow: 0 4px 15px rgba(0,0,0,0.08); width: 100%; max-width: 400px; text-align: center; }
        .google-btn { width: 100%; padding: 12px; background: #fff; border: 1px solid #cbd5e0; border-radius: 8px; font-weight: bold; cursor: pointer; margin-top: 15px; }
        #app-screen { display: none; width: 100%; max-width: 900px; background: #fff; border-radius: 12px; box-shadow: 0 4px 15px rgba(0,0,0,0.08); padding: 30px; }
        .control-box { background: #f7fafc; border: 1px solid #e2e8f0; border-radius: 8px; padding: 15px; margin-bottom: 20px; display: grid; grid-template-columns: repeat(auto-fit, minmax(160px, 1fr)); gap: 12px; }
        .input-group { display: flex; flex-direction: column; gap: 5px; }
        .input-group label { font-size: 11px; font-weight: bold; color: #4a5568; }
        .input-group input { padding: 8px; border: 1px solid #cbd5e0; border-radius: 6px; text-align: center; font-size: 14px; }
        .execute-btn { width: 100%; padding: 14px; background: #2b6cb0; color: white; border: none; border-radius: 8px; font-size: 15px; font-weight: bold; cursor: pointer; margin-bottom: 20px; box-shadow: 0 2px 5px rgba(0,0,0,0.1); }
        .execute-btn:hover { background: #2c5282; }
        
        .result-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 15px; margin-bottom: 20px; }
        .result-card { background: #fff; border: 1px solid #e2e8f0; border-radius: 8px; padding: 15px; min-height: 220px; }
        .card-title { font-size: 13px; font-weight: bold; text-align: center; margin-bottom: 12px; padding-bottom: 6px; border-bottom: 2px solid #edf2f7; }
        .title-buy { color: #e53e3e; }
        .title-sell { color: #3182ce; }
        .order-item { display: flex; justify-content: space-between; padding: 6px 10px; margin-bottom: 5px; border-radius: 5px; font-size: 13px; border: 1px solid #edf2f7; background: #fafbfc; font-weight: bold; }
        .summary-box { background: #ebf8ff; border: 1px solid #90cdf4; border-radius: 8px; padding: 15px; text-align: center; font-size: 14px; color: #2b6cb0; line-height: 1.5; }
    </style>
</head>
<body>

<div id="login-screen">
    <h2>🔒 스위치 자동화 보안 로그인</h2>
    <p>지정된 관리자 계정으로만 접속 가능합니다.</p>
    <button class="google-btn" onclick="loginWithGoogle()">구글 계정으로 로그인</button>
</div>

<div id="app-screen">
    <div style="display:flex; justify-content:space-between; align-items:center; font-size:12px; background:#f7fafc; padding:8px 12px; border-radius:6px; margin-bottom:15px;">
        <span id="user-email">확인 중...</span>
        <button onclick="logout()" style="background:none; border:1px solid #cbd5e0; padding:3px 8px; border-radius:4px; cursor:pointer; color:#e53e3e; font-weight:bold;">로그아웃</button>
    </div>

    <h2 style="text-align:center; color:#1a365d; margin-bottom:20px;">⚡ 스위치 기법 자동 퉁치기 시스템</h2>

    <div class="control-box">
        <div class="input-group">
            <label>📅 기준 날짜</label>
            <input type="date" id="target-date">
        </div>
        <div class="input-group">
            <label>💵 당일 종가 ($)</label>
            <input type="number" step="0.01" id="close-price" value="106.44" oninput="saveAndCalculate()">
        </div>
        <div class="input-group">
            <label>🔄 총 회차</label>
            <input type="number" id="total-rounds" value="20" oninput="saveAndCalculate()">
        </div>
        <div class="input-group">
            <label>📉 매수 목표 하락률 (%)</label>
            <input type="number" step="0.1" id="target-drop-pct" value="20" oninput="saveAndCalculate()">
        </div>
        <div class="input-group">
            <label>🔢 현재 진행 회차</label>
            <input type="number" id="current-round" value="4" oninput="saveAndCalculate()">
        </div>
        <div class="input-group">
            <label>📦 회당 수량 (주)</label>
            <input type="number" id="qty-per-round" value="10" oninput="saveAndCalculate()">
        </div>
    </div>

    <button class="execute-btn" onclick="saveAndCalculate()">스위치 계산 및 클라우드 동기화 실행</button>

    <div class="result-grid">
        <div class="result-card">
            <div class="card-title title-buy">🔴 매수 LOC 주문 목록 (방식 ① & ②)</div>
            <div id="buy-order-list"></div>
        </div>
        <div class="result-card">
            <div class="card-title title-sell">🔵 매도 / 퉁치기 주문 목록</div>
            <div id="sell-order-list"></div>
        </div>
    </div>

    <div class="summary-box" id="summary-display"></div>
</div>

<script>
    document.getElementById('target-date').value = new Date().toISOString().split('T')[0];

    function calculateSwitchOrders() {
        let closePrice = parseFloat(document.getElementById('close-price').value) || 0;
        let totalRounds = parseInt(document.getElementById('total-rounds').value) || 20;
        let targetDropPct = parseFloat(document.getElementById('target-drop-pct').value) || 20;
        let currentRound = parseInt(document.getElementById('current-round').value) || 1;
        let qtyPerRound = parseInt(document.getElementById('qty-per-round').value) || 10;

        if (closePrice <= 0) return;

        // 1. 첫 번째 매수 방식 계산: 전일 종가 * (1 - (목표하락률 / 전체회차) * 진행회차)
        let dropRatio = (targetDropPct / 100) / totalRounds * currentRound;
        let method1BuyPrice = closePrice * (1 - dropRatio);

        // 2. 두 번째 매수 방식 계산: 전일 종가 - 0.01
        let method2BuyPrice = closePrice - 0.01;

        // 매수 주문 리스트 구성
        let buyOrders = [
            { type: '매수·LOC (방식① 비율목표)', price: method1BuyPrice, qty: qtyPerRound },
            { type: '매수·LOC (방식② 초밀착)', price: method2BuyPrice, qty: qtyPerRound }
        ];

        // 3. 매도 / 추종 로직 시뮬레이션
        // 방식 ① 매수 물량에 대한 기준 매도가 (종가보다 높거나 특정 기준)
        let method1SellPrice = closePrice * (1 + (targetDropPct / 100) / totalRounds); 
        // 방식 ② 매수 물량 추종 매도가 (종가보다 소폭 위 혹은 가격 추종)
        let method2SellPrice = closePrice + 0.01;

        let sellOrders = [
            { type: '매도·LOC (방식① 기준분할)', price: method1SellPrice, qty: qtyPerRound },
            { type: '매도·LOC (방식② 가격추종)', price: method2SellPrice, qty: qtyPerRound }
        ];

        // 화면 렌더링
        let buyHtml = '';
        buyOrders.forEach(o => {
            buyHtml += `<div class="order-item"><span style="color:#e53e3e;">${o.type} ($${o.price.toFixed(2)})</span><span style="color:#e53e3e;">${o.qty}주</span></div>`;
        });
        document.getElementById('buy-order-list').innerHTML = buyHtml;

        let sellHtml = '';
        sellOrders.forEach(o => {
            sellHtml += `<div class="order-item"><span style="color:#3182ce;">${o.type} ($${o.price.toFixed(2)})</span><span style="color:#3182ce;">${o.qty}주</span></div>`;
        });
        document.getElementById('sell-order-list').innerHTML = sellHtml;

        let totalNeedMoney = (method1BuyPrice + method2BuyPrice) * qtyPerRound;
        document.getElementById('summary-display').innerHTML = `
            💡 <b>스위치 기법 자동 퉁치기 요약</b><br>
            ➔ 현재 진행 회차 <b>${currentRound}회차</b> 기준, 신규 필요 예수금 약 <b>$${totalNeedMoney.toLocaleString('en-US', {minimumFractionDigits: 2, maximumFractionDigits: 2})}</b> 산출완료.<br>
            <span style="font-size:12px; color:#4a5568;">설정값과 진행 상태는 변경 즉시 클라우드에 안전하게 자동 저장됩니다.</span>
        `;
    }
</script>
</body>
</html>

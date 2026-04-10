<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>The Travel Curation</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/gh/orioncactus/pretendard@v1.3.9/dist/web/static/pretendard.css" />
    <style>
        :root { --primary: #1a1a1a; --blue: #007aff; --red: #ff3b30; --gray: #8e8e93; --light-bg: #f5f5f7; }
        body { font-family: 'Pretendard', sans-serif; margin: 0; background: var(--light-bg); color: var(--primary); -webkit-font-smoothing: antialiased; }
        .container { max-width: 600px; margin: 0 auto; padding: 20px; }
        header { text-align: center; padding: 40px 0; }
        h1 { font-size: 1.8rem; letter-spacing: -1.5px; font-weight: 900; margin: 0; color: var(--primary); }
        header p { color: var(--gray); margin-top: 8px; font-size: 0.95rem; font-weight: 500; }
        #coupon-list { display: grid; gap: 16px; }
        .coupon-card { background: #fff; border: 1px solid #e1e1e3; border-radius: 20px; padding: 24px; box-shadow: 0 4px 6px rgba(0,0,0,0.02); transition: transform 0.2s ease; }
        .coupon-card:active { transform: scale(0.98); }
        .benefit-tag { color: var(--red); font-weight: 800; font-size: 0.85rem; margin-bottom: 8px; display: inline-block; background: rgba(255,59,48,0.1); padding: 2px 8px; border-radius: 4px; }
        .coupon-title { font-size: 1.15rem; font-weight: 700; margin-bottom: 20px; line-height: 1.45; color: #1c1c1e; word-break: keep-all; }
        .code-box { background: #f2f2f7; border-radius: 12px; padding: 16px; display: flex; justify-content: space-between; align-items: center; border: 1px solid #e5e5ea; }
        .code-text { font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, monospace; font-size: 1.1rem; font-weight: 700; color: var(--blue); letter-spacing: 0.5px; }
        .copy-btn { background: var(--blue); color: #fff; border: none; padding: 10px 18px; border-radius: 8px; cursor: pointer; font-weight: 700; font-size: 0.9rem; transition: opacity 0.2s; }
        .copy-btn:hover { opacity: 0.9; }
        .expiry-info { font-size: 0.8rem; color: var(--gray); margin-top: 16px; text-align: right; font-weight: 500; }
        .loading { text-align: center; padding: 60px; color: var(--gray); font-weight: 600; }
        .empty-msg { text-align: center; padding: 40px; color: var(--gray); }
    </style>
</head>
<body>

<div class="container">
    <header>
        <h1>THE TRAVEL CURATION</h1>
        <p>매일 업데이트되는 여행 할인 혜택</p>
    </header>
    <div id="loading" class="loading">최신 혜택을 불러오는 중...</div>
    <div id="coupon-list"></div>
</div>

<script>
    const SHEET_CSV_URL = 'https://docs.google.com/spreadsheets/d/e/2PACX-1vTvONfyw4ziH3o1cjLzNDIKTx80-4hVfmgHhqmNXtKIebZYvHJyl0fb07Ms_TOB1Fydgv8l2Xvixikd/pub?gid=0&single=true&output=csv';

    async function fetchCoupons() {
        try {
            const response = await fetch(SHEET_CSV_URL);
            const csvText = await response.text();
            const rows = csvText.split(/\r?\n/).slice(1); 
            const listContainer = document.getElementById('coupon-list');
            const loading = document.getElementById('loading');
            const now = new Date();

            loading.style.display = 'none';

            rows.forEach(row => {
                const cols = row.split(/,(?=(?:(?:[^"]*"){2})*[^"]*$)/);
                if (cols.length < 7) return;

                const code = cols[0].replace(/"/g, "").trim();         
                const discount = cols[1].replace(/"/g, "").trim();     
                const title = cols[2].replace(/"/g, "").trim();        
                const expiryStr = cols[6].replace(/"/g, "").trim();    

                if (!code || code === "" || code === "Promo code") return;

                const expiryDate = new Date(expiryStr);
                if (expiryDate < now) return;

                const cardHtml = `
                    <div class="coupon-card">
                        <span class="benefit-tag">${discount}</span>
                        <div class="coupon-title">${title}</div>
                        <div class="code-box">
                            <span class="code-text">${code}</span>
                            <button class="copy-btn" onclick="copyCode('${code}')">복사</button>
                        </div>
                        <div class="expiry-info">유효기간: ~${expiryStr.split(' ')[0]}</div>
                    </div>
                `;
                listContainer.innerHTML += cardHtml;
            });

            if (listContainer.innerHTML === "") {
                listContainer.innerHTML = '<div class="empty-msg">현재 사용 가능한 쿠폰이 없습니다.</div>';
            }
        } catch (error) {
            document.getElementById('loading').innerText = "데이터 로드 실패";
            console.error(error);
        }
    }

    function copyCode(text) {
        navigator.clipboard.writeText(text).then(() => {
            alert('쿠폰 코드가 복사되었습니다: ' + text);
        });
    }

    fetchCoupons();
</script>

</body>
</html>

<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0">
    <title>The Travel Curation</title>
    <style>
        * { box-sizing: border-box; margin: 0; padding: 0; }
        body { font-family: 'Pretendard', -apple-system, sans-serif; background-color: #f8f9fa; padding: 0 0 20px 0; color: #333; }
        header {
            background-color: #fff;
            padding: 30px 20px;
            border-bottom: 1px solid #eee;
            margin-bottom: 35px;
            display: flex;
            align-items: baseline;
            gap: 15px;
            max-width: 1200px;
            margin-left: auto;
            margin-right: auto;
            width: 100%;
        }
        .header-logo {
            font-size: 1.8rem;
            font-weight: 900;
            color: #000;
            letter-spacing: -0.04em;
            text-transform: lowercase;
            line-height: 1;
        }
        .header-slogan {
            font-size: 0.85rem;
            color: #999;
            font-weight: 300;
            letter-spacing: -0.01em;
            word-break: keep-all;
            line-height: 1;
        }
        .top3-section { width: 100%; max-width: 1200px; margin: 0 auto 40px auto; padding: 0 15px; }
        .section-title { font-size: 1.3rem; font-weight: 800; margin-bottom: 20px; color: #111; }
        .top3-grid { display: grid; grid-template-columns: repeat(3, 1fr); gap: 15px; }
        .top3-card { background: #fff; border-radius: 16px; overflow: hidden; border: 1px solid #eee; box-shadow: 0 4px 15px rgba(0,0,0,0.05); text-decoration: none; color: inherit; display: flex; flex-direction: column; transition: transform 0.2s; height: 100%; }
        .top3-card:hover { transform: translateY(-5px); }
        .slider-container { width: 100%; height: 200px; position: relative; overflow: hidden; background: #eee; }
        .slider-img { width: 100%; height: 100%; object-fit: cover; position: absolute; top: 0; left: 0; opacity: 0; transition: opacity 0.8s ease-in-out; }
        .slider-img.active { opacity: 1; }
        .top3-info { padding: 18px; flex-grow: 1; display: flex; flex-direction: column; }
        .top3-title { font-size: 1.1rem; font-weight: 800; line-height: 1.4; margin-bottom: 12px; color: #111; }
        .top3-features { list-style: none; padding: 0; margin: 0; }
        .top3-features li { font-size: 0.85rem; color: #555; line-height: 1.6; position: relative; padding-left: 14px; margin-bottom: 6px; word-break: keep-all; }
        .top3-features li::before { content: "•"; position: absolute; left: 0; color: #ff5b00; font-weight: bold; }
        .top3-price { font-size: 1.2rem; font-weight: 900; color: #ff5b00; text-align: right; border-top: 1px solid #f1f1f1; padding-top: 12px; margin-top: auto; }
        .horizontal-row { width: 100%; max-width: 1200px; margin: 0 auto 25px auto; padding: 0 15px; display: flex; gap: 12px; align-items: stretch; justify-content: flex-start; } 
        .coupon-card { flex: 0 0 380px; background: #fff; border-radius: 12px; display: flex; overflow: hidden; box-shadow: 0 4px 12px rgba(0,0,0,0.05); position: relative; border: 1px solid #eee; }
        .coupon-left { background: linear-gradient(135deg, #ff5b00 0%, #ff8540 100%); color: white; padding: 10px; display: flex; flex-direction: column; justify-content: center; align-items: center; width: 120px; min-width: 120px; text-align: center; border-right: 2px dashed rgba(255,255,255,0.4); }
        .coupon-info { padding: 18px; flex-grow: 1; display: flex; flex-direction: column; justify-content: center; background: #fff; }
        .coupon-info h4 { font-size: 1.05rem; color: #111; line-height: 1.3; font-weight: 800; margin-bottom: 8px; }
        .copy-btn { margin-top: 12px; background: #111; color: #fff; border: none; padding: 8px 16px; border-radius: 6px; font-size: 0.8rem; font-weight: 700; cursor: pointer; transition: background 0.2s; width: fit-content; }
        .coupon-card::before, .coupon-card::after { content: ''; position: absolute; left: 110px; width: 20px; height: 20px; background-color: #f8f9fa; border-radius: 50%; z-index: 2; border: 1px solid #eee; }
        .coupon-card::before { top: -11px; }
        .coupon-card::after { bottom: -11px; }
        .banner-container { flex: 1; display: flex; gap: 12px; align-items: stretch; justify-content: flex-start; }
        .dynamic-banner { flex: 0 0 260px; background: #fff; border-radius: 12px; overflow: hidden; box-shadow: 0 2px 8px rgba(0,0,0,0.05); display: flex; border: 1px solid #eee; }
        .dynamic-banner img { width: 100%; height: 100%; object-fit: cover; display: block; }
        @media (max-width: 1024px) {
            header { padding: 20px 15px; flex-direction: column; gap: 5px; align-items: flex-start; }
            .header-logo { font-size: 1.6rem; }
            .header-slogan { font-size: 0.75rem; }
            .top3-grid { grid-template-columns: 1fr; }
            .horizontal-row { flex-direction: column; align-items: flex-start; }
            .coupon-card, .banner-container { width: 100%; max-width: 100%; flex: none; }
        }
    </style>
</head>
<body>
<header>
    <h1 class="header-logo">thetravelcuration</h1>
    <p class="header-slogan">여행자의, 여행자에 의한, 여행자를 위한 감도 높은 여행큐레이션</p>
</header>
<section class="top3-section">
    <h3 class="section-title">✨ 이것만은 꼭, 큐레이터 픽 TOP3</h3>
    <div id="top3Container" class="top3-grid"></div>
</section>
<div id="contentWrapper"></div>
<script>
    const SHEET_ID = '1AACJ3r6VIcK2AH65wlJdLPP-7yJKTSW4Z-39msUgDu0';
    function formatTop3Price(val) {
        if (!val) return "";
        let str = String(val).trim();
        let numStr = str.replace(/[^0-9.-]+/g, "");
        let num = parseFloat(numStr);
        if (isNaN(num)) return str;
        return new Intl.NumberFormat('ko-KR').format(num) + "원";
    }
    function cleanText(text) { return !text ? '' : String(text).replace(/<[^>]*>/g, '').replace(/&nbsp;/g, ' ').trim(); }
    function fetchData() {
        const pushScript = document.createElement('script');
        pushScript.src = `https://docs.google.com/spreadsheets/d/${SHEET_ID}/gviz/tq?tqx=responseHandler:handlePushResponse&sheet=${encodeURIComponent('Home-Push Top3')}`;
        document.body.appendChild(pushScript);
        const couponScript = document.createElement('script');
        couponScript.src = `https://docs.google.com/spreadsheets/d/${SHEET_ID}/gviz/tq?tqx=responseHandler:handleCouponResponse&sheet=${encodeURIComponent('Home-Coupon List')}`;
        document.body.appendChild(couponScript);
    }
    window.handlePushResponse = function(response) {
        if (!response || !response.table) return;
        const rows = response.table.rows.slice(0, 3);
        const container = document.getElementById('top3Container');
        container.innerHTML = '';
        rows.forEach((row, idx) => {
            const title = row.c[1]?.v || ''; 
            const rawDesc = row.c[2]?.v || ''; 
            const price = formatTop3Price(row.c[3]?.v || '');
            const url = row.c[4]?.v || '#';
            const descLines = rawDesc.split('\n').map(l => l.trim()).filter(l => l !== '');
            const featureHtml = descLines.map(l => `<li>${l}</li>`).join('');
            const imgs = [];
            for(let i=5; i<=9; i++) { if(row.c[i]?.v) imgs.push(row.c[i].v); }
            const card = document.createElement('a');
            card.href = url; card.className = 'top3-card'; card.target = '_blank';
            let imgHtml = imgs.map((img, i) => `<img src="${img}" class="slider-img img-set-${idx} ${i === 0 ? 'active' : ''}">`).join('');
            card.innerHTML = `<div class="slider-container">${imgHtml}</div><div class="top3-info"><div class="top3-title">${title}</div><div class="top3-desc-container"><ul class="top3-features">${featureHtml}</ul></div><div class="top3-price">${price}~</div></div>`;
            container.appendChild(card);
            if (imgs.length > 1) {
                let currentImgIdx = 0;
                setInterval(() => {
                    const allImgs = card.querySelectorAll(`.img-set-${idx}`);
                    if(allImgs.length > 0) {
                        allImgs[currentImgIdx].classList.remove('active');
                        currentImgIdx = (currentImgIdx + 1) % imgs.length;
                        allImgs[currentImgIdx].classList.add('active');
                    }
                }, 3000);
            }
        });
    };
    window.handleCouponResponse = function(response) {
        if (!response || !response.table) return;
        const rows = response.table.rows;
        const wrapper = document.getElementById('contentWrapper');
        rows.forEach(row => {
            const coupon = {
                code: row.c[0] ? String(row.c[0].v) : '',
                discount: row.c[1] ? row.c[1].v : '',
                title: cleanText(row.c[18] ? row.c[18].v : (row.c[2] ? row.c[2].v : "특별 할인")),
                landingUrl: row.c[4] ? row.c[4].v : '#'
            };
            const rowDiv = document.createElement('div');
            rowDiv.className = 'horizontal-row';
            rowDiv.innerHTML = `<div class="coupon-card"><div class="coupon-left"><span style="font-size:1.8rem; font-weight:900;">${coupon.discount}</span><span style="font-size:0.7rem;">할인</span></div><div class="coupon-info"><h4>${coupon.title}</h4><button class="copy-btn" onclick="event.preventDefault(); navigator.clipboard.writeText('${coupon.code}').then(()=>alert('쿠폰이 복사되었습니다'))">쿠폰복사</button></div></div>`;
            wrapper.appendChild(rowDiv);
        });
    };
    window.onload = fetchData;
</script>
</body>
</html>

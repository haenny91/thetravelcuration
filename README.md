<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0">
    <title>The Travel Curation</title>
    <style>
        * { box-sizing: border-box; margin: 0; padding: 0; }
        body { font-family: 'Pretendard', sans-serif; background-color: #f8f9fa; padding: 20px; }
        .horizontal-row { width: 100%; max-width: 1200px; margin: 0 auto 30px auto; display: flex; gap: 12px; align-items: stretch; }
        .coupon-card { flex: 0 0 380px; background: #fff; border-radius: 12px; display: flex; overflow: hidden; box-shadow: 0 4px 12px rgba(0, 0, 0, 0.05); position: relative; border: 1px solid #eee; }
        .coupon-left { background: linear-gradient(135deg, #ff5b00 0%, #ff8540 100%); color: white; padding: 10px; display: flex; flex-direction: column; justify-content: center; align-items: center; width: 120px; min-width: 120px; text-align: center; border-right: 2px dashed rgba(255,255,255,0.4); }
        .coupon-left .currency { font-size: 0.9rem; font-weight: 700; opacity: 0.9; margin-bottom: 2px; }
        .coupon-left .amount { font-size: 1.8rem; font-weight: 900; letter-spacing: -0.05em; line-height: 1; }
        .coupon-left .unit { font-size: 0.75rem; margin-top: 8px; font-weight: 600; opacity: 0.9; }
        .coupon-info { padding: 18px; flex-grow: 1; display: flex; flex-direction: column; justify-content: center; background: #fff; }
        .coupon-tag { font-size: 0.65rem; font-weight: 800; color: #ff5b00; margin-bottom: 6px; background: #fff0e9; padding: 2px 8px; border-radius: 4px; width: fit-content; }
        .coupon-info h4 { font-size: 1.05rem; color: #111; line-height: 1.3; font-weight: 800; margin-bottom: 8px; letter-spacing: -0.03em; }
        .coupon-subtext { font-size: 0.75rem; color: #666; line-height: 1.5; font-weight: 500; white-space: pre-line; }
        .coupon-valid { font-size: 0.7rem; color: #ff5b00; margin-top: 8px; font-weight: 700; }
        .copy-btn { margin-top: 12px; background: #111; color: #fff; border: none; padding: 8px 16px; border-radius: 6px; font-size: 0.8rem; font-weight: 700; cursor: pointer; transition: background 0.2s; width: fit-content; }
        .copy-btn:active { background: #ff5b00; }
        .coupon-card::before, .coupon-card::after { content: ''; position: absolute; left: 110px; width: 20px; height: 20px; background-color: #f8f9fa; border-radius: 50%; z-index: 2; border: 1px solid #eee; }
        .coupon-card::before { top: -11px; }
        .coupon-card::after { bottom: -11px; }
        .banner-container { flex: 1; display: flex; gap: 12px; align-items: stretch; justify-content: flex-start; }
        .dynamic-banner { flex: 0 0 260px; background: #fff; border-radius: 12px; overflow: hidden; box-shadow: 0 2px 8px rgba(0,0,0,0.05); display: flex; border: 1px solid #eee; min-height: 50px; }
        .dynamic-banner > div, .dynamic-banner a, .dynamic-banner img { width: 100%; height: 100%; object-fit: cover; display: block; }
        .pc-only-container { display: flex; gap: 12px; flex: 1; }
        .mobile-only-container { display: none; }
        @media (max-width: 1024px) {
            .horizontal-row { flex-direction: column; align-items: center; }
            .coupon-card { width: 100%; max-width: 450px; flex: none; margin-bottom: 12px; }
            .banner-container { width: 100%; max-width: 450px; flex-direction: column; gap: 12px; flex: none; }
            .pc-only-container { display: none; }
            .mobile-only-container { display: flex; flex-direction: column; gap: 10px; width: 100%; }
            .m-product-card { display: flex; background: #fff; border-radius: 10px; overflow: hidden; border: 1px solid #eee; text-decoration: none; color: inherit; height: 90px; align-items: center; }
            .m-product-img { flex: 0 0 90px; height: 90px; object-fit: cover; background: #eee; }
            .m-product-info { padding: 10px 15px; flex: 1; display: flex; flex-direction: column; justify-content: space-between; height: 100%; }
            .m-product-title { font-size: 0.85rem; font-weight: 700; color: #111; line-height: 1.3; overflow: hidden; text-overflow: ellipsis; display: -webkit-box; -webkit-line-clamp: 2; -webkit-box-orient: vertical; margin-bottom: 3px; }
            .m-product-meta { font-size: 0.75rem; color: #666; display: flex; align-items: center; gap: 5px; }
            .m-star { color: #ff5b00; }
            .m-price { font-size: 0.9rem; font-weight: 800; color: #111; margin-top: 2px; }
        }
    </style>
</head>
<body>
<div id="contentWrapper"></div>
<script>
    const SHEET_ID = '1AACJ3r6VIcK2AH65wlJdLPP-7yJKTSW4Z-39msUgDu0';
    const TAB_NAME = 'Home-Coupon List';
    function cleanText(text) {
        if (!text) return '';
        let t = String(text);
        if (t.includes("limit exceeded") || t.includes("Please upgrade")) return "";
        return t.replace(/<!DOCTYPE html>|<html>|<\/html>|<body>|<\/body>|<head>[\s\S]*?<\/head>/gi, '').trim();
    }
    function summarizeTermsNarrative(text) {
        if (!text) return "";
        const clean = text.replace(/\s+/g, " ").trim();
        const lines = [];
        const minMatch = clean.match(/최소\s*(?:결제|구매)?\s*금액은?\s*([\d,.]+\s*(?:원|달러|USD|엔|HKD|만원|천원|홍콩달러))/i);
        if (minMatch) lines.push(`최소 ${minMatch[1].trim()} 이상 결제 시 사용 가능합니다.`);
        if (clean.includes("선착순") || clean.includes("소진 시") || clean.includes("한정")) {
            lines.push("선착순 한정 수량 쿠폰으로 비용 소진 시 사용이 불가할 수 있습니다.");
        }
        if (clean.includes("앱에서 처음") || clean.includes("첫 예약")) {
            lines.push("클룩 앱에서 첫 예약 시에만 사용 가능한 쿠폰입니다.");
        }
        if (clean.includes("계정당 1회") || clean.includes("1인당 1회") || clean.includes("당 1회")) {
            lines.push("계정당 1회에 한해 사용하실 수 있습니다.");
        }
        if (clean.includes("2인 이상 구매 시 1인 무료") || clean.includes("1+1")) {
            lines.push("2인 이상 구매 시 1인 혜택이 무료로 적용됩니다(1+1).");
        }
        if (clean.includes("일부 상품 제외")) {
            lines.push("일부 상품은 쿠폰 적용 대상에서 제외될 수 있습니다.");
        }
        if (clean.includes("중복 적용 불가") || clean.includes("중복 사용 불가")) {
            lines.push("다른 할인 혜택이나 프로모션과 중복 적용되지 않습니다.");
        }
        if (lines.length === 0) return "";
        return lines.join("\n");
    }
    function formatKlookDate(text) {
        if (!text) return "";
        const dateMatch = text.match(/Date\((\d+),(\d+),(\d+)/);
        if (dateMatch) {
            const month = parseInt(dateMatch[2]) + 1;
            const day = dateMatch[3];
            return `사용기간: ${month}월 ${day}일까지`;
        }
        return text; 
    }
    function copyToClipboard(code) {
        navigator.clipboard.writeText(code).then(() => {
            alert('쿠폰 코드가 복사되었습니다: ' + code);
        });
    }
    function renderDiscountValue(discountStr) {
        if (!discountStr) return '';
        const match = discountStr.match(/([a-zA-Z$]+|[^0-9\s,.]+)?\s*([0-9,.]+.*)/);
        if (match) {
            return `<span class="currency">${match[1] || ''}</span><span class="amount">${match[2] || ''}</span>`;
        }
        return `<span class="amount">${discountStr}</span>`;
    }
    function fetchData() {
        const script = document.createElement('script');
        script.src = `https://docs.google.com/spreadsheets/d/${SHEET_ID}/gviz/tq?tqx=responseHandler:handleResponse&sheet=${encodeURIComponent(TAB_NAME)}`;
        document.body.appendChild(script);
    }
    window.handleResponse = function(response) {
        if (!response || !response.table) return;
        const rows = response.table.rows;
        const coupons = rows.map(row => {
            return {
                code: row.c[0] ? String(row.c[0].v) : '',
                discount: row.c[1] ? String(row.c[1].v) : '',
                title: cleanText(row.c[18] ? row.c[18].v : (row.c[2] ? row.c[2].v : "특별 할인")),
                terms: row.c[17] ? summarizeTermsNarrative(cleanText(row.c[17].v)) : '',
                valid: formatKlookDate(cleanText(row.c[7] ? row.c[7].v : "")),
                pcBanners: [row.c[13], row.c[14], row.c[15], row.c[16]].map(c => c ? c.v : null).filter(b => b),
                productImg: row.c[19] ? row.c[19].v : '', 
                productTitle: row.c[2] ? row.c[2].v : '', 
                rating: row.c[12] ? row.c[12].v : '', 
                reviewCount: row.c[11] ? row.c[11].v : '', 
                price: row.c[10] ? row.c[10].v : '', 
                landingUrl: row.c[4] ? row.c[4].v : '#' 
            };
        });
        renderContent(coupons);
    };
    function renderContent(coupons) {
        const wrapper = document.getElementById('contentWrapper');
        wrapper.innerHTML = '';
        coupons.forEach(coupon => {
            const rowDiv = document.createElement('div');
            rowDiv.className = 'horizontal-row';
            const ratingHtml = coupon.rating ? `<span class="m-star">★</span> ${coupon.rating}` : '';
            const reviewHtml = coupon.reviewCount ? `(${coupon.reviewCount})` : '';
            rowDiv.innerHTML = `
                <div class="coupon-card">
                    <div class="coupon-left">
                        ${renderDiscountValue(coupon.discount)}
                        <span class="unit">할인 혜택</span>
                    </div>
                    <div class="coupon-info">
                        <span class="coupon-tag">특별 혜택</span>
                        <h4 translate="no">${coupon.title}</h4>
                        <p class="coupon-subtext" style="white-space: pre-line;">${coupon.terms}</p>
                        <p class="coupon-valid">${coupon.valid}</p>
                        <button class="copy-btn" onclick="copyToClipboard('${coupon.code}')">할인쿠폰받기</button>
                    </div>
                </div>
                <div class="banner-container">
                    <div class="pc-only-container">
                        ${coupon.pcBanners.map(b => `<div class="dynamic-banner">${cleanText(b)}</div>`).join('')}
                    </div>
                    <div class="mobile-only-container">
                        <a href="${coupon.landingUrl}" class="m-product-card" target="_blank">
                            <img src="${coupon.productImg}" class="m-product-img" alt="${coupon.productTitle}" onerror="this.src='https://via.placeholder.com/90x90?text=Image'">
                            <div class="m-product-info">
                                <div>
                                    <div class="m-product-title">${coupon.productTitle}</div>
                                    <div class="m-product-meta">${ratingHtml} ${reviewHtml}</div>
                                </div>
                                <div class="m-price">${coupon.price}</div>
                            </div>
                        </a>
                    </div>
                </div>
            `;
            wrapper.appendChild(rowDiv);
            const scripts = rowDiv.querySelectorAll('.pc-only-container script');
            scripts.forEach(s => {
                const ns = document.createElement('script');
                if (s.src) ns.src = s.src; else ns.textContent = s.textContent;
                document.body.appendChild(ns);
            });
        });
    }
    window.onload = fetchData;
</script>
</body>
</html>

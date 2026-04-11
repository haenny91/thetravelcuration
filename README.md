<html>
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0">
    <title>The Travel Curation</title>
    <style>
        * { box-sizing: border-box; margin: 0; padding: 0; }
        body { font-family: 'Pretendard', sans-serif; background-color: #f8f9fa; padding: 20px; }
        .horizontal-row { width: 100%; max-width: 1200px; margin: 0 auto 30px auto; display: flex; gap: 12px; align-items: stretch; }
        .coupon-card { flex: 0 0 40%; background: #fff; border-radius: 12px; display: flex; overflow: hidden; box-shadow: 0 4px 12px rgba(0, 0, 0, 0.05); position: relative; border: 1px solid #eee; }
        .coupon-left { background: linear-gradient(135deg, #ff5b00 0%, #ff8540 100%); color: white; padding: 10px; display: flex; flex-direction: column; justify-content: center; align-items: center; width: 130px; min-width: 130px; text-align: center; border-right: 2px dashed rgba(255,255,255,0.4); }
        .coupon-left .currency { font-size: 1rem; font-weight: 700; opacity: 0.9; margin-bottom: 2px; }
        .coupon-left .amount { font-size: 2rem; font-weight: 900; letter-spacing: -0.05em; line-height: 1; }
        .coupon-left .unit { font-size: 0.8rem; margin-top: 8px; font-weight: 600; opacity: 0.9; }
        .coupon-info { padding: 18px 25px; flex-grow: 1; display: flex; flex-direction: column; justify-content: center; background: #fff; }
        .coupon-tag { font-size: 0.7rem; font-weight: 800; color: #ff5b00; margin-bottom: 6px; background: #fff0e9; padding: 2px 8px; border-radius: 4px; width: fit-content; }
        .coupon-info h4 { font-size: 1.1rem; color: #111; line-height: 1.3; font-weight: 800; margin-bottom: 8px; letter-spacing: -0.03em; }
        .coupon-subtext { font-size: 0.8rem; color: #666; line-height: 1.6; font-weight: 500; white-space: pre-line; }
        .coupon-valid { font-size: 0.75rem; color: #ff5b00; margin-top: 10px; font-weight: 700; }
        .copy-btn { margin-top: 14px; background: #111; color: #fff; border: none; padding: 10px 18px; border-radius: 6px; font-size: 0.85rem; font-weight: 700; cursor: pointer; transition: background 0.2s; width: fit-content; }
        .copy-btn:active { background: #ff5b00; }
        .coupon-card::before, .coupon-card::after { content: ''; position: absolute; left: 120px; width: 20px; height: 20px; background-color: #f8f9fa; border-radius: 50%; z-index: 2; border: 1px solid #eee; }
        .coupon-card::before { top: -11px; }
        .coupon-card::after { bottom: -11px; }
        .banner-container { flex: 0 0 60%; display: flex; gap: 12px; }
        .dynamic-banner { flex: 1; background: #fff; border-radius: 12px; overflow: hidden; display: flex; align-items: center; justify-content: center; box-shadow: 0 2px 8px rgba(0,0,0,0.05); }
        @media (max-width: 1024px) { .horizontal-row { flex-direction: column; } .coupon-card, .banner-container { flex: 1 1 auto; } }
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

    function formatKlookDate(text) {
        if (!text) return "";
        const dateMatch = text.match(/Date\((\d+),(\d+),(\d+)/);
        if (dateMatch) {
            const month = parseInt(dateMatch[2]) + 1;
            const day = dateMatch[3];
            return `사용기간: ${month}월 ${day}일까지`;
        }
        const simpleDate = text.match(/(\d+)월\s*(\d+)일/);
        if (simpleDate) return `사용기간: ${simpleDate[1]}월 ${simpleDate[2]}일까지`;
        return text; 
    }

    function summarizeTermsNarrative(text) {
        if (!text) return "";
        const clean = text.replace(/\s+/g, " ");
        const patterns = {
            max: /최대\s*(?:할인\s*금액은?)?\s*([\d,.]+\s*(?:원|달러|USD|엔|HKD|홍콩달러|만원|천원|HK))/i,
            min: /최소\s*(?:결제|구매)?\s*금액은?\s*([\d,.]+\s*(?:원|달러|USD|엔|HKD|홍콩달러|만원|천원|HK))/i,
            limited: /수량[은\s]*한정|제한되어|랏\s*소진/,
            firstCome: /결제\s*순서|선착순/
        };

        const max = clean.match(patterns.max);
        const min = clean.match(patterns.min);
        const isLtd = patterns.limited.test(clean);
        const isFc = patterns.firstCome.test(clean);
        
        let lines = [];
        if (max) lines.push(`최대 할인액: ${max[1].trim()}`);
        if (min) lines.push(`최소 구매금액: ${min[1].trim()}`);
        if (isLtd || isFc) lines.push(`한정수량 쿠폰으로 할인혜택은 선착순으로 제공됩니다.`);
        
        return lines.join("\n");
    }

    function copyToClipboard(code) {
        navigator.clipboard.writeText(code).then(() => {
            alert('쿠폰 코드가 복사되었습니다: ' + code);
        }).catch(err => {
            console.error('복사 실패', err);
        });
    }

    function renderDiscountValue(discountStr) {
        if (!discountStr) return '';
        const match = discountStr.match(/([a-zA-Z]+|[^0-9\s,.]+)?\s*([0-9,.]+.*)/);
        if (match) {
            const currency = match[1] || '';
            const amount = match[2] || '';
            return `<span class="currency">${currency}</span><span class="amount">${amount}</span>`;
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
            let promoCode = row.c[0] ? String(row.c[0].v) : '';
            let rawTitle = cleanText(row.c[18] ? row.c[18].v : "");
            if (!rawTitle) rawTitle = cleanText(row.c[2] ? row.c[2].v : "특별 할인");
            let rawTerms = cleanText(row.c[17] ? row.c[17].v : "");
            let rawValid = cleanText(row.c[7] ? row.c[7].v : "");
            return {
                code: promoCode,
                discount: row.c[1] ? String(row.c[1].v) : '',
                title: rawTitle,
                terms: summarizeTermsNarrative(rawTerms),
                valid: formatKlookDate(rawValid),
                banners: [row.c[13], row.c[14], row.c[15], row.c[16]].map(c => c ? c.v : null).filter(b => b)
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
            rowDiv.innerHTML = `
                <div class="coupon-card">
                    <div class="coupon-left">
                        ${renderDiscountValue(coupon.discount)}
                        <span class="unit">할인 혜택</span>
                    </div>
                    <div class="coupon-info">
                        <span class="coupon-tag">특별 혜택</span>
                        <h4>${coupon.title}</h4>
                        <p class="coupon-subtext">${coupon.terms}</p>
                        <p class="coupon-valid">${coupon.valid}</p>
                        <button class="copy-btn" onclick="copyToClipboard('${coupon.code}')">할인쿠폰받기</button>
                    </div>
                </div>
                <div class="banner-container">
                    ${coupon.banners.map(b => `<div class="dynamic-banner">${cleanText(b)}</div>`).join('')}
                </div>
            `;
            wrapper.appendChild(rowDiv);
            const scripts = rowDiv.getElementsByTagName('script');
            for (let s of scripts) {
                const ns = document.createElement('script');
                if (s.src) ns.src = s.src; else ns.textContent = s.textContent;
                document.body.appendChild(ns);
            }
        });
    }
    window.onload = fetchData;
</script>
</body>
</html>

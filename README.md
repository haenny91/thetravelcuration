<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0">
    <title>The Travel Curation</title>
    <style>
        * { box-sizing: border-box; margin: 0; padding: 0; }
        body { font-family: 'Pretendard', sans-serif; background-color: #f8f9fa; padding: 20px; }
        
        .horizontal-row {
            width: 100%;
            max-width: 1200px;
            margin: 0 auto 30px auto;
            display: flex;
            gap: 12px;
            align-items: stretch;
        }

        .coupon-card { 
            flex: 0 0 40%;
            background: #fff; 
            border-radius: 12px; 
            display: flex; 
            overflow: hidden; 
            box-shadow: 0 4px 12px rgba(0, 0, 0, 0.05);
            position: relative;
            border: 1px solid #eee;
        }
        
        /* 쿠폰 왼쪽 영역 수정 */
        .coupon-left {
            background: linear-gradient(135deg, #ff5b00 0%, #ff8540 100%);
            color: white;
            padding: 10px;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            width: 130px;
            min-width: 130px;
            text-align: center;
            border-right: 2px dashed rgba(255,255,255,0.4);
        }
        
        /* 통화 기호 스타일 (상단) */
        .coupon-left .currency { 
            font-size: 1rem; 
            font-weight: 700; 
            opacity: 0.9;
            margin-bottom: 2px;
        }
        
        /* 금액 숫자 스타일 (하단) */
        .coupon-left .amount { 
            font-size: 2rem; 
            font-weight: 900; 
            letter-spacing: -0.05em; 
            line-height: 1; 
        }

        .coupon-left .unit { font-size: 0.8rem; margin-top: 8px; font-weight: 600; opacity: 0.9; }

        .coupon-info { 
            padding: 18px 25px; 
            flex-grow: 1; 
            display: flex; 
            flex-direction: column; 
            justify-content: center;
            background: #fff;
        }
        
        .coupon-tag { 
            font-size: 0.7rem; 
            font-weight: 800; 
            color: #ff5b00; 
            margin-bottom: 6px;
            background: #fff0e9;
            padding: 2px 8px;
            border-radius: 4px;
            width: fit-content;
        }
        
        .coupon-info h4 { 
            font-size: 1.25rem; 
            color: #111; 
            line-height: 1.2; 
            font-weight: 800;
            margin-bottom: 8px;
            letter-spacing: -0.03em;
        }
        
        .coupon-subtext { font-size: 0.8rem; color: #666; line-height: 1.4; font-weight: 500; }
        .coupon-valid { font-size: 0.7rem; color: #aaa; margin-top: 6px; }

        .coupon-card::before, .coupon-card::after {
            content: '';
            position: absolute;
            left: 120px;
            width: 20px;
            height: 20px;
            background-color: #f8f9fa;
            border-radius: 50%;
            z-index: 2;
            border: 1px solid #eee;
        }
        .coupon-card::before { top: -11px; }
        .coupon-card::after { bottom: -11px; }

        .banner-container {
            flex: 0 0 60%;
            display: flex;
            gap: 12px;
        }

        .dynamic-banner { 
            flex: 1;
            background: #fff; 
            border-radius: 12px; 
            overflow: hidden; 
            display: flex; 
            align-items: center; 
            justify-content: center; 
            box-shadow: 0 2px 8px rgba(0,0,0,0.05);
        }

        @media (max-width: 1024px) {
            .horizontal-row { flex-direction: column; }
            .coupon-card, .banner-container { flex: 1 1 auto; }
        }
    </style>
</head>
<body>

<div id="contentWrapper"></div>

<script>
    const SHEET_ID = '1AACJ3r6VIcK2AH65wlJdLPP-7yJKTSW4Z-39msUgDu0';
    const TAB_NAME = 'Home-Coupon List';

    /* 통화와 금액을 분리하여 렌더링하는 함수 */
    function renderDiscountValue(discountStr) {
        if (!discountStr) return '';
        
        // HKD20, JPY3,000 등 통화 기호(알파벳)와 숫자를 분리
        const match = discountStr.match(/([a-zA-Z]+|[^0-9\s,.]+)?\s*([0-9,.]+.*)/);
        
        if (match) {
            const currency = match[1] || '';
            const amount = match[2] || '';
            return `<span class="currency">${currency}</span><span class="amount">${amount}</span>`;
        }
        
        return `<span class="amount">${discountStr}</span>`;
    }

    function smartTranslate(text) {
        if (!text) return '';
        let t = text.trim();
        const replaceMap = [
            { en: /Promo code descriptions/gi, ko: "쿠폰 상세 혜택" },
            { en: /Promo code des/gi, ko: "쿠폰 상세 혜택" },
            { en: /Terms & Conditions/gi, ko: "이용 약관 및 유의사항" },
            { en: /Valid until/gi, ko: "유효 기간" },
            { en: /Discount description/gi, ko: "할인 혜택 상세" },
            { en: /Discount descrip/gi, ko: "할인 혜택 상세" },
            { en: /App only/gi, ko: "앱 전용" },
            { en: /New users only/gi, ko: "신규 회원 전용" },
            { en: /Minimum spend/gi, ko: "최소 결제 금액" },
            { en: /Maximum discount/gi, ko: "최대 할인액" }
        ];
        replaceMap.forEach(item => { t = t.replace(item.en, item.ko); });
        return t;
    }

    function formatSheetDate(dateStr) {
        if (!dateStr || typeof dateStr !== 'string' || !dateStr.startsWith('Date(')) return dateStr;
        try {
            const params = dateStr.match(/\(([^)]+)\)/)[1].split(',').map(Number);
            const date = new Date(params[0], params[1], params[2], params[3], params[4], params[5]);
            date.setHours(date.getHours() - 1);
            const now = new Date();
            const diffDays = (date - now) / (1000 * 60 * 60 * 24);
            if (diffDays >= 365) return "무제한";
            const y = date.getFullYear();
            const m = String(date.getMonth() + 1).padStart(2, '0');
            const d = String(date.getDate()).padStart(2, '0');
            return `${y}.${m}.${d}`;
        } catch (e) { return dateStr; }
    }

    function fetchData() {
        const script = document.createElement('script');
        script.src = `https://docs.google.com/spreadsheets/d/${SHEET_ID}/gviz/tq?tqx=responseHandler:handleResponse&sheet=${encodeURIComponent(TAB_NAME)}`;
        document.body.appendChild(script);
    }

    window.handleResponse = function(response) {
        if (!response || !response.table) return;
        const rows = response.table.rows;
        const coupons = rows.map(row => ({
            discount: row.c[1] ? String(row.c[1].v) : '',
            title: row.c[2] ? String(row.c[2].v) : '',
            valid: row.c[7] ? String(row.c[7].v) : '',
            terms: row.c[9] ? String(row.c[9].v) : '',
            banners: [
                row.c[13] ? row.c[13].v : null,
                row.c[14] ? row.c[14].v : null,
                row.c[15] ? row.c[15].v : null,
                row.c[16] ? row.c[16].v : null
            ].filter(b => b !== null)
        }));
        renderContent(coupons);
    };

    function renderContent(coupons) {
        const wrapper = document.getElementById('contentWrapper');
        wrapper.innerHTML = '';
        coupons.forEach(coupon => {
            const rowDiv = document.createElement('div');
            rowDiv.className = 'horizontal-row';
            const couponHtml = `
                <div class="coupon-card">
                    <div class="coupon-left">
                        ${renderDiscountValue(coupon.discount)}
                        <span class="unit">할인 혜택</span>
                    </div>
                    <div class="coupon-info">
                        <span class="coupon-tag">특별 혜택</span>
                        <h4>${smartTranslate(coupon.title)}</h4>
                        <p class="coupon-subtext">${smartTranslate(coupon.terms)}</p>
                        <p class="coupon-valid">유효기간: ${formatSheetDate(coupon.valid)}</p>
                    </div>
                </div>
            `;
            let bannerHtml = '<div class="banner-container">';
            coupon.banners.forEach(html => {
                bannerHtml += `<div class="dynamic-banner">${html}</div>`;
            });
            bannerHtml += '</div>';
            rowDiv.innerHTML = couponHtml + bannerHtml;
            wrapper.appendChild(rowDiv);
            const scripts = rowDiv.getElementsByTagName('script');
            for (let i = 0; i < scripts.length; i++) {
                const newScript = document.createElement('script');
                if (scripts[i].src) { newScript.src = scripts[i].src; }
                else { newScript.textContent = scripts[i].textContent; }
                document.body.appendChild(newScript);
            }
        });
    }
    window.onload = fetchData;
</script>

</body>
</html>

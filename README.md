<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>KT 통합 견적서 및 고객 신청 시스템</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
    <style>
        @page { size: A4; margin: 0; }
        * { 
            box-sizing: border-box; 
            font-family: -apple-system, BlinkMacSystemFont, "Apple SD Gothic Neo", "Pretendard Variable", Pretendard, "Malgun Gothic", "맑은 고딕", sans-serif;
            -webkit-user-select: none;
            -moz-user-select: none;
            -ms-user-select: none;
            user-select: none;
        }
        body {
            background-color: #f4f6f9;
            padding: 20px 0;
            margin: 0;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            -webkit-font-smoothing: antialiased;
            position: relative;
        }

        @media (prefers-color-scheme: dark) {
            body { background-color: #f4f6f9 !important; }
            .invoice-container { background-color: #ffffff !important; color: #000000 !important; }
            th { background-color: #f1f5f9 !important; color: #000000 !important; }
            td { background-color: #ffffff !important; color: #000000 !important; border-color: #a0a0a0 !important; }
            input[type="text"], select, textarea { background-color: #f8fafc !important; color: #000000 !important; }
        }

        /* KT CS 김영훈 시그니처 워터마크 오버레이 */
        .watermark-overlay {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%) rotate(-25deg);
            font-size: 21px;
            font-weight: 900;
            color: rgba(0, 75, 141, 0.08) !important;
            white-space: nowrap;
            pointer-events: none;
            z-index: 999;
            letter-spacing: -0.5px;
            text-align: center;
        }

        /* 최상단 신뢰 배지 스타일 */
        .signature-badge {
            width: 794px;
            margin-bottom: 12px;
            padding: 10px 14px;
            background: #f0f9ff;
            border: 1px solid #bae6fd;
            border-radius: 8px;
            text-align: center;
            box-shadow: 0 2px 6px rgba(2, 132, 199, 0.06);
        }

        .tab-menu-container {
            width: 794px;
            display: flex;
            gap: 10px;
            margin-bottom: 12px;
        }
        .tab-btn {
            flex: 1;
            padding: 12px 15px;
            font-size: 14px;
            font-weight: 800;
            color: #475569;
            background-color: #e2e8f0;
            border: none;
            border-radius: 8px 8px 0 0;
            cursor: pointer;
            transition: all 0.2s ease;
        }
        .tab-btn.active {
            color: #ffffff;
            background-color: #004b8d;
            box-shadow: 0 -2px 8px rgba(0, 75, 141, 0.2);
        }

        .toolbar-area {
            width: 100%;
            display: flex;
            justify-content: space-between;
            align-items: center;
            background-color: #f1f5f9;
            padding: 8px 12px;
            border-radius: 6px;
            margin-bottom: 12px;
            border: 1px solid #cbd5e1;
        }
        .toolbar-group { display: flex; gap: 6px; }
        .tool-btn {
            padding: 6px 12px;
            font-size: 11.5px;
            font-weight: 700;
            border-radius: 4px;
            cursor: pointer;
            border: 1px solid #cbd5e1;
            background-color: #ffffff;
            color: #334155;
            transition: all 0.2s;
        }
        .tool-btn:hover { background-color: #e2e8f0; }
        .tool-btn.btn-send { background-color: #0284c7; color: #ffffff; border-color: #0284c7; }
        .tool-btn.btn-send:hover { background-color: #0369a1; }
        .tool-btn.btn-save { background-color: #0d9488; color: #ffffff; border-color: #0d9488; }
        .tool-btn.btn-save:hover { background-color: #0f766e; }
        .tool-btn.btn-reset { background-color: #ef4444; color: #ffffff; border-color: #ef4444; }
        .tool-btn.btn-reset:hover { background-color: #dc2626; }

        .invoice-container {
            width: 794px;
            background-color: #ffffff;
            padding: 25px 35px;
            box-shadow: 0 4px 20px rgba(0,0,0,0.08);
            box-sizing: border-box;
            image-rendering: -webkit-optimize-contrast;
            margin: 0 auto;
            position: relative;
            overflow: hidden;
        }

        .invoice-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 15px;
            border-bottom: 3px solid #1c5280;
            padding-bottom: 6px;
        }
        .logo-area {
            font-size: 28px;
            font-weight: 900;
            color: #000000 !important;
            font-family: 'Arial Black', Impact, sans-serif;
            letter-spacing: -2px;
            line-height: 1;
        }
        .title-area {
            font-size: 24px;
            font-weight: 800;
            color: #004b8d !important;
            text-align: center;
            flex-grow: 1;
            letter-spacing: -0.5px;
            margin-right: 40px;
        }

        table {
            width: 100%;
            border-collapse: collapse;
            font-size: 11px;
            color: #000000 !important;
            margin-bottom: 8px;
            table-layout: fixed;
            box-sizing: border-box;
        }
        th, td {
            border: 1px solid #a0a0a0 !important;
            padding: 4px 5px;
            height: 25px;
            color: #000000 !important;
            vertical-align: middle !important;
            text-align: center !important;
            box-sizing: border-box;
            overflow: hidden;
        }
        th {
            background-color: #f1f5f9 !important;
            font-weight: 700 !important;
            color: #000000 !important;
        }

        .info-table th { width: 14%; }
        .info-table td { width: 36%; }

        .notice-container-table th { width: 11% !important; }
        .notice-container-table td { width: 89% !important; }

        .product-table th {
            background-color: #f1f5f9 !important;
            color: #000000 !important;
            font-weight: 700;
            font-size: 11px;
        }

        input[type="text"], input[type="date"], select, textarea {
            width: 100%;
            height: 100%;
            border: none !important;
            background-color: #f8fafc !important;
            padding: 0 4px;
            border-radius: 3px;
            font-size: 11px;
            font-family: inherit;
            color: #000000 !important;
            -webkit-text-fill-color: #000000 !important;
            opacity: 1 !important;
            font-weight: 600;
            box-sizing: border-box;
            outline: none;
            text-align: center !important;
            box-shadow: none !important;
        }

        textarea {
            text-align: left !important;
            padding: 6px !important;
            resize: none;
            height: 55px;
            line-height: 1.4;
        }

        .capture-text-node {
            font-size: 11px !important;
            font-weight: 600 !important;
            color: #000000 !important;
            text-align: center !important;
            width: 100%;
            display: inline-block;
            line-height: 1.4 !important;
            white-space: normal !important;
            word-break: break-all;
            vertical-align: middle;
        }

        input[type="date"] {
            color: transparent !important;
            -webkit-text-fill-color: transparent !important;
        }
        input[type="date"].has-value {
            color: #000000 !important;
            -webkit-text-fill-color: #000000 !important;
        }

        input:focus, select:focus, textarea:focus { background-color: #e0f2fe !important; }
        input[readonly], select[readonly], .lock-cell {
            color: #475569 !important;
            -webkit-text-fill-color: #475569 !important;
            background-color: #f1f5f9 !important;
            font-weight: 600;
            cursor: not-allowed;
        }
        .blue-readonly { color: #004b8d !important; }
        .benefit-highlight {
            color: #d91414 !important;
            font-size: 13px;
            font-weight: 800;
            text-align: center !important;
            background-color: #fef2f2 !important;
            letter-spacing: 0.5px;
        }

        .total-row { background-color: #e2e8f0 !important; font-weight: 700; }

        .notice-text {
            font-size: 10.5px;
            color: #222222 !important;
            line-height: 1.45;
            font-weight: 500;
            text-align: left !important;
            white-space: normal !important;
            overflow: visible !important;
        }
        .bg-alert { background-color: #fffbeb !important; color: #b45309 !important; }

        .section-title-hai {
            background: #1c5280; color: #fff; padding: 4px 8px; font-size: 12px;
            font-weight: 700; margin-top: 10px; margin-bottom: 2px; border-radius: 2px;
        }
        .offer-container { display: flex; align-items: center; justify-content: center; width: 100%; gap: 4px; }
        .offer-container input { width: 65px !important; text-align: right; font-weight: bold; }
        .offer-total-label { font-size: 10px; color: #2c3e50; font-weight: 600; white-space: nowrap; }

        table.items-hai { width: 100%; border-collapse: collapse; font-size: 10.5px; margin-bottom: 4px; table-layout: fixed; }
        table.items-hai th, table.items-hai td { border: 1px solid #a0a0a0 !important; padding: 3px 4px; text-align: center; height: 23px; }
        table.items-hai thead th { background: #f1f5f9 !important; font-weight: 700; }
        table.items-hai td.item-name { text-align: left; padding-left: 6px; }
        table.items-hai td.group-label { background: #f8fafc; font-weight: 700; text-align: left; padding-left: 6px; height: 20px; color: #004b8d; }
        table.items-hai tr.subtotal td { background: #fef2f2 !important; font-weight: 700; }
        table.items-hai tr.subtotal td.item-name { text-align: right; padding-right: 8px; }

        .qty-box { display: inline-flex; align-items: center; justify-content: center; width: 100%; gap: 2px; }
        .qty-box input { width: 32px !important; text-align: center; font-weight: bold; padding: 0 !important; }
        .qty-btn {
            width: 16px; height: 16px; font-size: 10px; font-weight: bold;
            cursor: pointer; border: 1px solid #ccc; background: #ffffff;
            border-radius: 2px; display: inline-flex; align-items: center; justify-content: center;
            padding: 0; user-select: none; color: #333;
        }
        .qty-btn:hover { background: #e2e8f0; }

        .grand-total-hai { margin-top: 4px; text-align: right; }
        .grand-total-hai table { border-collapse: collapse; font-size: 12px; margin-left: auto; width: 280px; }
        .grand-total-hai td { border: 1px solid #a0a0a0; padding: 4px 8px; }
        .grand-total-hai td.label { background: #1c5280; color: #fff; font-weight: 700; text-align: center; }
        .grand-total-hai td.value { font-weight: 800; font-size: 13px; color: #d91414; text-align: right; }

        .hide-for-photo { display: none !important; }

        .btn-area {
            margin-top: 15px;
            margin-bottom: 20px;
            width: 794px;
            display: flex;
            gap: 15px;
        }
        .download-btn {
            flex: 1;
            background-color: #004b8d;
            color: white;
            border: none;
            padding: 14px;
            font-size: 16px;
            font-weight: 700;
            cursor: pointer;
            border-radius: 6px;
            box-shadow: 0 4px 12px rgba(0, 61, 141, 0.25);
            transition: background 0.2s;
        }
        .download-btn:hover { background-color: #143757; }
        .pdf-btn { background-color: #10b981; box-shadow: 0 4px 12px rgba(16, 185, 129, 0.25); }
        .pdf-btn:hover { background-color: #047857; }

        .responsive-wrapper {
            width: 100%;
            overflow-x: auto;
            display: flex;
            flex-direction: column;
            align-items: center;
        }
    </style>
</head>
<body oncontextmenu="return false" onselectstart="return false" ondragstart="return false">
    <div class="responsive-wrapper">
        
        <!-- 최상단 공식 시그니처 배지 -->
        <div class="signature-badge">
            <div style="font-size:11px; color:#0284c7; font-weight:800; letter-spacing:1px; margin-bottom:2px;">KT CS TELECOM AUTOMATION ARCHITECT</div>
            <div style="font-size:13px; color:#1e293b; font-weight:700;">
                🔒 본 시스템은 <span style="color:#0284c7;">KT CS 김영훈</span>의 공식 지적 자산입니다. (무단 복제 및 사용 금지)
            </div>
        </div>

        <div class="tab-menu-container">
            <button class="tab-btn active" id="btn-renewal" onclick="switchEstimateTab('renewal')">법인회선 재약정 견적서</button>
            <button class="tab-btn" id="btn-total" onclick="switchEstimateTab('total')">유무선 통합 견적서</button>
            <button class="tab-btn" id="btn-haiorder" onclick="switchEstimateTab('haiorder')">하이오더 견적서</button>
        </div>

        <!-- ================= [서식 1: 법인회선 재약정 견적서] ================= -->
        <div class="invoice-container" id="capture-area-renewal">
            <div class="watermark-overlay">KT CS 대구센터 김영훈 무단 복사 및 사용을 금지합니다</div>
            
            <div class="toolbar-area no-print-target">
                <div class="toolbar-group">
                    <button class="tool-btn" onclick="recalcCurrentActiveTab()">🔄 합계 새로고침</button>
                    <button class="tool-btn btn-reset" onclick="resetActiveTabForm()">♻️ 입력 내용 초기화</button>
                </div>
                <div class="toolbar-group">
                    <button class="tool-btn btn-send" onclick="sendQuoteDataGas()">📩 DB 적재 & 알림 전송</button>
                    <button class="tool-btn btn-save" onclick="saveCurrentEstimateData()">💾 최근 작성 저장</button>
                    <button class="tool-btn" onclick="loadSavedEstimateData()">📂 불러오기</button>
                </div>
            </div>

            <div class="invoice-header">
                <div class="logo-area">kt</div>
                <div class="title-area">법인회선 재약정 견적서</div>
            </div>

            <table class="info-table">
                <tr>
                    <th>견적일자</th>
                    <td><input type="text" class="invoice-date blue-readonly" readonly /></td>
                    <th>사업자번호</th>
                    <td><input type="text" value="314-81-42683" readonly /></td>
                </tr>
                <tr>
                    <th>업체명</th>
                    <td><input type="text" value=" 귀하" class="client-name" id="renewal-client-name" onfocus="clearGuidance(this)" onblur="restoreGuidance(this)" /></td>
                    <th>회사명</th>
                    <td><input type="text" value="(주) KT CS" readonly /></td>
                </tr>
                <tr>
                    <th>사업자번호</th>
                    <td><input type="text" id="renewal-biz-num" placeholder="고객 사업자번호 입력" /></td>
                    <th>대표자명</th>
                    <td><input type="text" value="이창호" readonly /></td>
                </tr>
                <tr>
                    <th>총 제공되는 혜택</th>
                    <td class="benefit-highlight" id="total-benefits-renewal">₩0</td>
                    <th>주소</th>
                    <td><input type="text" value="대전 서구 갈마로 160(괴정동) KT 인재개발원" readonly /></td>
                </tr>
                <tr>
                    <th>수수료</th>
                    <td><input type="text" id="fee-renewal" value="0" oninput="runBenefitCalculationsRenewal(this)" /></td>
                    <th>업종</th>
                    <td><input type="text" value="정보통신업" readonly /></td>
                </tr>
                <tr>
                    <th>통합사은품</th>
                    <td><input type="text" id="gift-renewal" value="0" oninput="runBenefitCalculationsRenewal(this)" /></td>
                    <th>담당부서</th>
                    <td><input type="text" value="KT CS 대구센터" readonly /></td>
                </tr>
                <tr>
                    <th rowspan="2">재약정<br />구비서류</th>
                    <td rowspan="2" class="notice-text lock-cell" style="background-color: #fafafa; font-size: 10px; line-height: 1.45; user-select: none; text-align: center !important;">
                        <span class="capture-text-node" style="font-size: 10px !important;">법인 대표자 신분증 / 명함<br />사업자 등록증 (사본가능)</span>
                    </td>
                    <th>담당자</th>
                    <td>
                        <select class="manager-select blue-readonly" onchange="updateManagerInfo(this, 'renewal')">
                            <option value="김영훈 과장" data-phone="010-8290-9971" data-email="tsmobile1@naver.com" selected>김영훈 과장</option>
                            <option value="custom" data-phone="" data-email="">직접입력</option>
                        </select>
                    </td>
                </tr>
                <tr>
                    <th>연락처</th>
                    <td><input type="text" id="manager-phone-renewal" class="blue-readonly" value="010-8290-9971" readonly /></td>
                </tr>
                <tr>
                    <th style="user-select: none;">견적유효기간</th>
                    <td class="notice-text lock-cell" style="background-color: #ffffff; font-weight: bold; color: #004b8d !important; user-select: none; text-align: center !important;">
                        <span class="capture-text-node" style="color: #004b8d !important; font-weight: bold !important;">견적서 제출일로부터 30일 이내</span>
                    </td>
                    <th>이메일주소</th>
                    <td><input type="text" id="manager-email-renewal" class="blue-readonly" value="tsmobile1@naver.com" readonly /></td>
                </tr>
            </table>

            <table class="product-table" id="product-table-renewal">
                <thead>
                    <tr>
                        <th style="width:18%">설치장소</th>
                        <th style="width:15%">가입 상품</th>
                        <th style="width:16%">약정만료기간</th>
                        <th style="width:13%">상품 요금제</th>
                        <th style="width:11%">청구금액</th>
                        <th style="width:12%">갱신시 요금</th>
                        <th style="width:15%">기타 (요금변동)</th>
                    </tr>
                </thead>
                <tbody>
                    <script>
                        for(let i=0; i<15; i++) {
                            document.write(`
                            <tr>
                                <td><input type="text" /></td>
                                <td><input type="text" /></td>
                                <td><input type="date" onchange="checkDateValue(this)" /></td>
                                <td><input type="text" /></td>
                                <td><input type="text" class="calc-charge-renewal" oninput="runCalculationsRenewal(this)" /></td>
                                <td><input type="text" class="calc-renew-renewal" oninput="runCalculationsRenewal(this)" /></td>
                                <td><input type="text" class="calc-diff-renewal" readonly placeholder="-" /></td>
                            </tr>`);
                        }
                    </script>
                    <tr class="total-row">
                        <td colspan="4" class="text-center">최종합계</td>
                        <td id="total-charge-renewal">0</td>
                        <td id="total-renew-renewal">0</td>
                        <td id="total-diff-renewal">0</td>
                    </tr>
                </tbody>
            </table>

            <table class="notice-container-table" style="user-select: none;">
                <tr>
                    <th style="color: #d91414 !important; background-color: #fef2f2 !important; text-align: center !important;">필수 안내</th>
                    <td class="notice-text bg-alert" style="padding: 10px; font-weight: 700;">
                        • 수수료 지급 : 재약정 완료 ➔ 익월 말 ➔ 법인 대표님 휴대폰으로 모바일 상품권 발송<br />
                        • 사은품 지급 : 재약정 완료 ➔ 1주 이내 ➔ 법인 대표님 휴대폰으로 모바일 상품권 발송<br />
                        • <span style="color:#d91414;">[지급 제한 조건]</span> 수수료 및 통합 사은품은 법인 대표님의 명함에 기재된 명확한 전화번호로만 전송이 가능합니다.
                    </td>
                </tr>
            </table>

            <table class="notice-container-table" style="user-select: none;">
                <tr>
                    <th style="text-align: center !important;">유의 사항</th>
                    <td class="notice-text" style="padding: 10px; background-color: #fafafa; color: #555 !important;">
                        1. 상기 법인 회선 재약정 기준 KT 정식 약정 기간은 총 3년(36개월)이며, 기타 상세 세부사항은 KT 이용약관에 준합니다.<br />
                        2. 재약정 시점에 따라 일부 가입 상품(기업용 TV 등)의 (구)요금제가 (신)요금제로 변경될 수 있습니다.<br />
                        3. 현장 설비 이상 및 장애 발생 시 즉시 KT 고객센터(국번없이 100번)를 통해 접수하시거나 담당부서로 상담 바랍니다.
                    </td>
                </tr>
            </table>

            <table class="notice-container-table">
                <tr>
                    <th style="text-align: center !important; background-color: #f1f5f9;">메모 사항</th>
                    <td style="padding: 0; background-color: #f8fafc;">
                        <textarea placeholder="여기에 특이사항이나 추가 협의 내용을 입력하세요. (출력 시 줄바꿈이 그대로 유지됩니다)"></textarea>
                    </td>
                </tr>
            </table>
        </div>

        <!-- ================= [서식 2: 유무선 통합 견적서] ================= -->
        <div class="invoice-container" id="capture-area-total" style="display: none;">
            <div class="watermark-overlay">KT CS 대구센터 김영훈 무단 복사 및 사용을 금지합니다</div>

            <div class="toolbar-area no-print-target">
                <div class="toolbar-group">
                    <button class="tool-btn" onclick="recalcCurrentActiveTab()">🔄 합계 새로고침</button>
                    <button class="tool-btn btn-reset" onclick="resetActiveTabForm()">♻️ 입력 내용 초기화</button>
                </div>
                <div class="toolbar-group">
                    <button class="tool-btn btn-send" onclick="sendQuoteDataGas()">📩 DB 적재 & 알림 전송</button>
                    <button class="tool-btn btn-save" onclick="saveCurrentEstimateData()">💾 최근 작성 저장</button>
                    <button class="tool-btn" onclick="loadSavedEstimateData()">📂 불러오기</button>
                </div>
            </div>

            <div class="invoice-header">
                <div class="logo-area">kt</div>
                <div class="title-area">유무선 통합 견적서</div>
            </div>

            <table class="info-table">
                <tr>
                    <th>견적일자</th>
                    <td><input type="text" class="invoice-date blue-readonly" readonly /></td>
                    <th>사업자번호</th>
                    <td><input type="text" value="314-81-42683" readonly /></td>
                </tr>
                <tr>
                    <th>고객명/업체명</th>
                    <td><input type="text" value=" 귀하" class="client-name" id="total-client-name" onfocus="clearGuidance(this)" onblur="restoreGuidance(this)" /></td>
                    <th>회사명</th>
                    <td><input type="text" value="(주) KT CS" readonly /></td>
                </tr>
                <tr>
                    <th>생년월일/사업자</th>
                    <td><input type="text" id="total-biz-num" placeholder="생년월일 또는 사업자번호" /></td>
                    <th>대표자명</th>
                    <td><input type="text" value="이창호" readonly /></td>
                </tr>
                <tr>
                    <th>총 제공 혜택</th>
                    <td class="benefit-highlight" id="total-benefits-total">₩0</td>
                    <th>주소</th>
                    <td><input type="text" value="대전 서구 갈마로 160(괴정동) KT 인재개발원" readonly /></td>
                </tr>
                <tr>
                    <th>지원금/수수료</th>
                    <td><input type="text" id="fee-total" value="0" oninput="runBenefitCalculationsTotal(this)" /></td>
                    <th>업종</th>
                    <td><input type="text" value="정보통신업" readonly /></td>
                </tr>
                <tr>
                    <th>통합사은품</th>
                    <td><input type="text" id="gift-total" value="0" oninput="runBenefitCalculationsTotal(this)" /></td>
                    <th>담당부서</th>
                    <td><input type="text" value="KT CS 대구센터" readonly /></td>
                </tr>
                <tr>
                    <th rowspan="2">구비서류</th>
                    <td rowspan="2" class="notice-text lock-cell" style="background-color: #fafafa; font-size: 9.5px; line-height: 1.35; user-select: none; text-align: left !important; padding-left: 8px;">
                        <span class="capture-text-node" style="font-size: 9.5px !important; text-align: left !important;">
                            • 개인 : 신분증<br />
                            • 개인사업자 : 신분증 + 사업자등록증<br />
                            • 법인사업자 : 대표자신분증 + 법인사업자등록증
                        </span>
                    </td>
                    <th>담당자</th>
                    <td>
                        <select class="manager-select blue-readonly" onchange="updateManagerInfo(this, 'total')">
                            <option value="김영훈 과장" data-phone="010-8290-9971" data-email="tsmobile1@naver.com" selected>김영훈 과장</option>
                            <option value="custom" data-phone="" data-email="">직접입력</option>
                        </select>
                    </td>
                </tr>
                <tr>
                    <th>연락처</th>
                    <td><input type="text" id="manager-phone-total" class="blue-readonly" value="010-8290-9971" readonly /></td>
                </tr>
                <tr>
                    <th style="user-select: none;">견적유효기간</th>
                    <td class="notice-text lock-cell" style="background-color: #ffffff; font-weight: bold; color: #004b8d !important; user-select: none; text-align: center !important;">
                        <span class="capture-text-node" style="color: #004b8d !important; font-weight: bold !important;">견적서 제출일로부터 30일 이내</span>
                    </td>
                    <th>이메일주소</th>
                    <td><input type="text" id="manager-email-total" class="blue-readonly" value="tsmobile1@naver.com" readonly /></td>
                </tr>
            </table>

            <table class="product-table" id="product-table-total">
                <thead>
                    <tr>
                        <th style="width:13%">구분</th>
                        <th style="width:22%">상품 / 요금제명</th>
                        <th style="width:10%">회선수</th>
                        <th style="width:14%">기준 월정액</th>
                        <th style="width:14%">약정/결합할인</th>
                        <th style="width:14%">월 납부예정액</th>
                        <th style="width:13%">약정/비고</th>
                    </tr>
                </thead>
                <tbody>
                    <script>
                        for(let i=0; i<12; i++) {
                            document.write(`
                            <tr>
                                <td>
                                    <select style="font-size:10px;">
                                        <option value="">--선택--</option>
                                        <option value="모바일">모바일</option>
                                        <option value="인터넷">인터넷</option>
                                        <option value="TV">TV</option>
                                        <option value="전화">전화</option>
                                        <option value="CCTV">CCTV</option>
                                        <option value="테이블오더">테이블오더</option>
                                        <option value="기타">기타</option>
                                    </select>
                                </td>
                                <td><input type="text" /></td>
                                <td><input type="text" class="calc-qty-total" oninput="runCalculationsTotal(this)" placeholder="1" /></td>
                                <td><input type="text" class="calc-base-total" oninput="runCalculationsTotal(this)" /></td>
                                <td><input type="text" class="calc-discount-total" oninput="runCalculationsTotal(this)" /></td>
                                <td><input type="text" class="calc-final-total blue-readonly" readonly placeholder="0" /></td>
                                <td><input type="text" placeholder="3년약정 등" /></td>
                            </tr>`);
                        }
                    </script>
                    <tr class="total-row">
                        <td colspan="2" class="text-center">최종합계</td>
                        <td id="total-qty-total">0</td>
                        <td id="total-base-total">0</td>
                        <td id="total-discount-total">0</td>
                        <td id="total-final-total" style="color:#004b8d;">0</td>
                        <td>-</td>
                    </tr>
                </tbody>
            </table>

            <table class="notice-container-table" style="user-select: none;">
                <tr>
                    <th style="color: #d91414 !important; background-color: #fef2f2 !important; text-align: center !important;">필수 안내</th>
                    <td class="notice-text bg-alert" style="padding: 10px; font-weight: 700;">
                        • 지원금/수수료 지급 : 개통 완료 ➔ 익월 말 ➔ 명의자 휴대폰으로 모바일 상품권 발송<br />
                        • 사은품 지급 : 개통 완료 ➔ 1주 이내 ➔ 명의자 휴대폰으로 모바일 상품권 발송<br />
                        • <span style="color:#d91414;">[지급 제한 조건]</span> 지원금 및 사은품은 명의자 본인 명의의 휴대폰 번호로만 발송이 가능합니다.
                    </td>
                </tr>
            </table>

            <table class="notice-container-table" style="user-select: none;">
                <tr>
                    <th style="text-align: center !important;">유의 사항</th>
                    <td class="notice-text" style="padding: 10px; background-color: #fafafa; color: #555 !important;">
                        1. KT 정식 약정 기간은 상품별 3년(36개월) 기준이며, 약정 기간 내 해지 시 위약금(할인반환금)이 발생할 수 있습니다.<br />
                        2. 결합할인 금액은 결합 구성 회선 수 및 상품 요금제에 따라 실시간 변동될 수 있습니다.<br />
                        3. 현장 설비 이상 및 장애 발생 시 즉시 KT 고객센터(국번없이 100번)를 통해 접수하시거나 담당부서로 상담 바랍니다.
                    </td>
                </tr>
            </table>

            <table class="notice-container-table">
                <tr>
                    <th style="text-align: center !important; background-color: #f1f5f9;">메모 사항</th>
                    <td style="padding: 0; background-color: #f8fafc;">
                        <textarea placeholder="여기에 특이사항이나 추가 협의 내용을 입력하세요."></textarea>
                    </td>
                </tr>
            </table>
        </div>

        <!-- ================= [서식 3: 하이오더 상품 견적서] ================= -->
        <div class="invoice-container" id="capture-area-haiorder" style="display: none;">
            <div class="watermark-overlay">KT CS 대구센터 김영훈 무단 복사 및 사용을 금지합니다</div>

            <div class="toolbar-area no-print-target">
                <div class="toolbar-group">
                    <button class="tool-btn" onclick="recalcCurrentActiveTab()">🔄 합계 새로고침</button>
                    <button class="tool-btn btn-reset" onclick="resetActiveTabForm()">♻️ 수량 초기화(0)</button>
                </div>
                <div class="toolbar-group">
                    <button class="tool-btn btn-send" onclick="sendQuoteDataGas()">📩 DB 적재 & 알림 전송</button>
                    <button class="tool-btn btn-save" onclick="saveCurrentEstimateData()">💾 최근 작성 저장</button>
                    <button class="tool-btn" onclick="loadSavedEstimateData()">📂 불러오기</button>
                </div>
            </div>

            <div class="invoice-header">
                <div class="logo-area">kt</div>
                <div class="title-area">하이오더(테이블오더) 견적서</div>
            </div>

            <table class="info-table">
                <tr>
                    <th>견적일자</th>
                    <td><input type="text" class="invoice-date blue-readonly" readonly /></td>
                    <th>사업자번호</th>
                    <td><input type="text" value="314-81-42683" readonly /></td>
                </tr>
                <tr>
                    <th>매장명/업체명</th>
                    <td><input type="text" value=" 귀하" class="client-name" id="hai-store-name" onfocus="clearGuidance(this)" onblur="restoreGuidance(this)" /></td>
                    <th>회사명</th>
                    <td><input type="text" value="(주) KT CS" readonly /></td>
                </tr>
                <tr>
                    <th>서비스명</th>
                    <td><input type="text" value="하이오더 외" readonly /></td>
                    <th>대표자명</th>
                    <td><input type="text" value="이창호" readonly /></td>
                </tr>
                <tr class="amount-row">
                    <th>견적금액(VAT포함)</th>
                    <td class="benefit-highlight" id="topQuoteAmount">0원</td>
                    <th>주소</th>
                    <td><input type="text" value="대전 서구 갈마로 160(괴정동) KT 인재개발원" readonly /></td>
                </tr>
                <tr>
                    <th>견적유효기간</th>
                    <td>견적 제출일로부터 30일</td>
                    <th>업종</th>
                    <td><input type="text" value="정보통신업" readonly /></td>
                </tr>
                <tr>
                    <th>고객 오퍼</th>
                    <td>
                        <div class="offer-container">
                            <input type="text" id="offerPriceInput" class="price-input" value="0" />원
                            <span class="offer-total-label" id="offerTotalText">(총합계: 0원)</span>
                        </div>
                    </td>
                    <th>담당부서</th>
                    <td><input type="text" value="KT CS 대구센터" readonly /></td>
                </tr>
                <tr>
                    <th rowspan="2">구비서류</th>
                    <td rowspan="2" class="notice-text lock-cell" style="background-color: #fafafa; font-size: 9.5px; line-height: 1.35; user-select: none; text-align: left !important; padding-left: 8px;">
                        <span class="capture-text-node" style="font-size: 9.5px !important; text-align: left !important;">
                            • 개인 : 신분증<br />
                            • 개인사업자 : 신분증 + 사업자등록증<br />
                            • 법인사업자 : 대표자신분증 + 법인사업자등록증
                        </span>
                    </td>
                    <th>담당자</th>
                    <td>
                        <select class="manager-select blue-readonly" onchange="updateManagerInfo(this, 'haiorder')">
                            <option value="김영훈 과장" data-phone="010-8290-9971" data-email="tsmobile1@naver.com" selected>김영훈 과장</option>
                            <option value="custom" data-phone="" data-email="">직접입력</option>
                        </select>
                    </td>
                </tr>
                <tr>
                    <th>연락처</th>
                    <td><input type="text" id="manager-phone-haiorder" class="blue-readonly" value="010-8290-9971" readonly /></td>
                </tr>
                <tr>
                    <th style="user-select: none;">담당자 이메일</th>
                    <td colspan="3"><input type="text" id="manager-email-haiorder" class="blue-readonly" value="tsmobile1@naver.com" readonly /></td>
                </tr>
            </table>

            <div class="section-title-hai">하이오더 서비스 견적 - 메뉴판</div>
            <table class="items-hai" data-group="g1">
                <thead>
                    <tr>
                        <th style="width:28%">품목</th>
                        <th style="width:12%">약정</th>
                        <th style="width:14%">수량(EA)</th>
                        <th style="width:11%">단가</th>
                        <th style="width:14%">공급가액<br>(VAT별도)</th>
                        <th style="width:10%">세액</th>
                        <th style="width:11%">합계액<br>(VAT포함)</th>
                    </tr>
                </thead>
                <tbody>
                    <tr><td colspan="7" class="group-label">a) 태블릿</td></tr>
                    <tr class="item-row device-row">
                        <td class="item-name">하이오더 2 단말기</td>
                        <td><input type="text" value="36개월"></td>
                        <td>
                            <div class="qty-box">
                                <button type="button" class="qty-btn no-print-target" onclick="changeQty(this, -1)">-</button>
                                <input type="text" inputmode="numeric" class="qty-input device-qty" id="hai-table-qty" value="15">
                                <button type="button" class="qty-btn no-print-target" onclick="changeQty(this, 1)">+</button>
                            </div>
                        </td>
                        <td><input type="text" class="price-input price" value="5,000" readonly></td>
                        <td class="supply">0</td><td class="tax">0</td><td class="total">0</td>
                    </tr>
                    <tr><td colspan="7" class="group-label">b) 카드리더기</td></tr>
                    <tr class="item-row reader-row">
                        <td class="item-name">카드리더기 종류</td>
                        <td>
                            <select class="reader-type" onchange="recalcAllHaiorder()">
                                <option value="선불형(NICE)">선불형(NICE)</option>
                                <option value="후불형(그 외)">후불형(그 외)</option>
                            </select>
                        </td>
                        <td>
                            <div class="qty-box">
                                <button type="button" class="qty-btn no-print-target" onclick="changeQty(this, -1)">-</button>
                                <input type="text" inputmode="numeric" class="qty-input reader-qty" id="hai-reader-qty" value="0">
                                <button type="button" class="qty-btn no-print-target" onclick="changeQty(this, 1)">+</button>
                            </div>
                        </td>
                        <td><input type="text" class="price-input price" value="2,000" readonly></td>
                        <td class="supply">0</td><td class="tax">0</td><td class="total">0</td>
                    </tr>
                    <tr><td colspan="7" class="group-label">c) 서비스이용료</td></tr>
                    <tr class="item-row service-row">
                        <td class="item-name">하이오더 서비스</td>
                        <td><input type="text" value="36개월"></td>
                        <td><input type="text" inputmode="numeric" class="qty-input service-qty" value="15" readonly></td>
                        <td><input type="text" class="price-input price" value="15,000" readonly></td>
                        <td class="supply">0</td><td class="tax">0</td><td class="total">0</td>
                    </tr>
                    <tr class="subtotal">
                        <td class="item-name" colspan="4">① 합계 (a+b+c)</td>
                        <td class="g-supply">0</td><td class="g-tax">0</td><td class="g-total">0</td>
                    </tr>
                </tbody>
            </table>

            <div class="section-title-hai">알림판 (주방)</div>
            <table class="items-hai" data-group="g2">
                <thead>
                    <tr>
                        <th style="width:28%">품목</th>
                        <th style="width:12%">약정</th>
                        <th style="width:14%">수량</th>
                        <th style="width:11%">단가</th>
                        <th style="width:14%">공급가액<br>(VAT별도)</th>
                        <th style="width:10%">세액</th>
                        <th style="width:11%">합계액<br>(VAT포함)</th>
                    </tr>
                </thead>
                <tbody>
                    <tr><td colspan="7" class="group-label">a) 태블릿(단말기)</td></tr>
                    <tr class="item-row device-row">
                        <td class="item-name">알림판 10인치 (하이오더2)</td>
                        <td><input type="text" value="36개월"></td>
                        <td>
                            <div class="qty-box">
                                <button type="button" class="qty-btn no-print-target" onclick="changeQty(this, -1)">-</button>
                                <input type="text" inputmode="numeric" class="qty-input device-qty" id="hai-k10-qty" value="1">
                                <button type="button" class="qty-btn no-print-target" onclick="changeQty(this, 1)">+</button>
                            </div>
                        </td>
                        <td><input type="text" class="price-input price" value="5,000" readonly></td>
                        <td class="supply">0</td><td class="tax">0</td><td class="total">0</td>
                    </tr>
                    <tr class="item-row device-row">
                        <td class="item-name">알림판 15인치</td>
                        <td><input type="text" value="36개월"></td>
                        <td>
                            <div class="qty-box">
                                <button type="button" class="qty-btn no-print-target" onclick="changeQty(this, -1)">-</button>
                                <input type="text" inputmode="numeric" class="qty-input device-qty" id="hai-k15-qty" value="1">
                                <button type="button" class="qty-btn no-print-target" onclick="changeQty(this, 1)">+</button>
                            </div>
                        </td>
                        <td><input type="text" class="price-input price" value="7,000" readonly></td>
                        <td class="supply">0</td><td class="tax">0</td><td class="total">0</td>
                    </tr>
                    <tr><td colspan="7" class="group-label">b) 서비스이용료</td></tr>
                    <tr class="item-row service-row">
                        <td class="item-name">알림판(모니터) 서비스</td>
                        <td><input type="text" value="36개월"></td>
                        <td><input type="text" inputmode="numeric" class="qty-input service-qty" value="2" readonly></td>
                        <td><input type="text" class="price-input price" value="15,000" readonly></td>
                        <td class="supply">0</td><td class="tax">0</td><td class="total">0</td>
                    </tr>
                    <tr class="subtotal">
                        <td class="item-name" colspan="4">② 합계 (a+b)</td>
                        <td class="g-supply">0</td><td class="g-tax">0</td><td class="g-total">0</td>
                    </tr>
                </tbody>
            </table>

            <div class="section-title-hai">웨이팅 솔루션</div>
            <table class="items-hai" data-group="g3">
                <thead>
                    <tr>
                        <th style="width:28%">품목</th>
                        <th style="width:12%">약정</th>
                        <th style="width:14%">수량</th>
                        <th style="width:11%">단가</th>
                        <th style="width:14%">공급가액<br>(VAT별도)</th>
                        <th style="width:10%">세액</th>
                        <th style="width:11%">합계액<br>(VAT포함)</th>
                    </tr>
                </thead>
                <tbody>
                    <tr><td colspan="7" class="group-label">a) 태블릿(단말기)</td></tr>
                    <tr class="item-row device-row">
                        <td class="item-name">10인치 블랙 (하이오더2)</td>
                        <td><input type="text" value="36개월"></td>
                        <td>
                            <div class="qty-box">
                                <button type="button" class="qty-btn no-print-target" onclick="changeQty(this, -1)">-</button>
                                <input type="text" inputmode="numeric" class="qty-input device-qty" id="hai-waiting-qty" value="1">
                                <button type="button" class="qty-btn no-print-target" onclick="changeQty(this, 1)">+</button>
                            </div>
                        </td>
                        <td><input type="text" class="price-input price" value="5,000" readonly></td>
                        <td class="supply">0</td><td class="tax">0</td><td class="total">0</td>
                    </tr>
                    <tr><td colspan="7" class="group-label">b) 서비스이용료</td></tr>
                    <tr class="item-row service-row">
                        <td class="item-name">웨이팅 솔루션</td>
                        <td><input type="text" value="36개월"></td>
                        <td><input type="text" inputmode="numeric" class="qty-input service-qty" value="1" readonly></td>
                        <td><input type="text" class="price-input price" value="15,000" readonly></td>
                        <td class="supply">0</td><td class="tax">0</td><td class="total">0</td>
                    </tr>
                    <tr><td colspan="7" class="group-label">c) 웨이팅 전용거치대 (멀티탭, 배송료 포함)</td></tr>
                    <tr class="item-row etc-row">
                        <td class="item-name">전용거치대</td>
                        <td><input type="text" value="일시불"></td>
                        <td>
                            <div class="qty-box">
                                <button type="button" class="qty-btn no-print-target" onclick="changeQty(this, -1)">-</button>
                                <input type="text" inputmode="numeric" class="qty-input" value="1">
                                <button type="button" class="qty-btn no-print-target" onclick="changeQty(this, 1)">+</button>
                            </div>
                        </td>
                        <td><input type="text" class="price-input price" value="180,000" readonly></td>
                        <td class="supply">0</td><td class="tax">0</td><td class="total">0</td>
                    </tr>
                    <tr class="subtotal">
                        <td class="item-name" colspan="4">③ 합계 (a+b+c)</td>
                        <td class="g-supply">0</td><td class="g-tax">0</td><td class="g-total">0</td>
                    </tr>
                </tbody>
            </table>

            <div class="grand-total-hai">
                <table>
                    <tr>
                        <td class="label">하이오더 합계 (①+②+③)</td>
                        <td class="value" id="haiorderTotal">0원</td>
                    </tr>
                </table>
            </div>

            <div class="section-title-hai">설치비 및 부자재</div>
            <table class="items-hai" data-group="g4">
                <thead>
                    <tr>
                        <th style="width:24%">품목</th>
                        <th style="width:10%">할부</th>
                        <th style="width:8%">수량</th>
                        <th style="width:11%">단가</th>
                        <th style="width:13%">공급가액<br>(VAT별도)</th>
                        <th style="width:9%">세액</th>
                        <th style="width:13%">합계액<br>(VAT포함)</th>
                        <th style="width:12%">비고</th>
                    </tr>
                </thead>
                <tbody>
                    <tr class="item-row4 install-fee-row">
                        <td class="item-name">설치비용</td>
                        <td><input type="text" value="일시불"></td>
                        <td><input type="text" class="qty4 auto-device-qty" value="0" readonly></td>
                        <td><input type="text" class="price-input price4" id="installPriceInput" value="0" readonly></td>
                        <td class="supply4">0</td><td class="tax4">0</td><td class="total4">0</td>
                        <td rowspan="5" style="font-size:9.5px; text-align: center; vertical-align: middle; line-height: 1.4; padding: 2px;">
                            기본 25만원<br>+대당 1만원<br><span style="color:#d91414; font-weight:bold;">최초 1회 면제</span><br>중도해지시 청구
                        </td>
                    </tr>
                    <tr class="item-row4">
                        <td class="item-name">배터리</td>
                        <td><input type="text" value="일시불"></td>
                        <td><input type="text" class="qty4 auto-device-qty" value="0" readonly></td>
                        <td><input type="text" class="price-input price4" value="17,300" readonly></td>
                        <td class="supply4">0</td><td class="tax4">0</td><td class="total4">0</td>
                    </tr>
                    <tr class="item-row4">
                        <td class="item-name">부자재 (거치대 등)</td>
                        <td><input type="text" value="일시불"></td>
                        <td><input type="text" class="qty4 auto-device-qty" value="0" readonly></td>
                        <td><input type="text" class="price-input price4" value="16,000" readonly></td>
                        <td class="supply4">0</td><td class="tax4">0</td><td class="total4">0</td>
                    </tr>
                    <tr class="item-row4">
                        <td class="item-name">와이파이 AP</td>
                        <td><input type="text" value="일시불"></td>
                        <td><input type="text" class="qty4" value="1" readonly></td>
                        <td><input type="text" class="price-input price4" value="59,000" readonly></td>
                        <td class="supply4">0</td><td class="tax4">0</td><td class="total4">0</td>
                    </tr>
                    <tr class="item-row4">
                        <td class="item-name">증폭기</td>
                        <td><input type="text" value="일시불"></td>
                        <td><input type="text" class="qty4" value="1" readonly></td>
                        <td><input type="text" class="price-input price4" value="0" readonly></td>
                        <td class="supply4">0</td><td class="tax4">0</td><td class="total4">0</td>
                    </tr>
                    <tr class="subtotal">
                        <td class="item-name" colspan="4">합계</td>
                        <td class="g-supply4" id="hai-onetime-supply">0</td><td class="g-tax4">0</td><td class="g-total4" id="hai-onetime-total">0</td>
                        <td></td>
                    </tr>
                </tbody>
            </table>

            <table class="notice-container-table" style="user-select: none;">
                <tr>
                    <th style="text-align: center !important;">유의 사항</th>
                    <td class="notice-text" style="padding: 6px; background-color: #fafafa; color: #555 !important;">
                        1. 본 견적서의 유효기간은 견적서 제출일로부터 30일입니다.<br />
                        2. 상기 견적서 중 KT 단말의 A/S 보증기간은 3년이며, 그 외 세부사항은 이용약관에 표기된 기준을 따릅니다.<br />
                        3. 하이오더 운영 및 POS 연동을 위한 KT 유무선통신 요금은 별도입니다.<br />
                        4. 가맹점별 추가적인 시설에 대한 연동비용은 현장 실사 후 추가될 수 있습니다.
                    </td>
                </tr>
            </table>

            <table class="notice-container-table">
                <tr>
                    <th style="text-align: center !important; background-color: #f1f5f9;">메모 사항</th>
                    <td style="padding: 0; background-color: #f8fafc;">
                        <textarea placeholder="여기에 특이사항이나 추가 협의 내용을 입력하세요."></textarea>
                    </td>
                </tr>
            </table>
        </div>

        <div class="btn-area">
            <button class="download-btn" onclick="generateActiveInvoiceImage('jpg')">견적서 JPG 다운로드</button>
            <button class="download-btn pdf-btn" onclick="generateActiveInvoiceImage('pdf')">견적서 PDF 다운로드</button>
        </div>
    </div>

    <script>
        let activeTab = 'renewal';
        // 구글 앱스 스크립트 웹 앱 최신 배포 엔드포인트 URL
        const GAS_URL = "https://script.google.com/macros/s/AKfycbzriskJha8aL9cnErvdImPwMBxLi690oyLCUgrTBHJHcvHiWlNvGwU3ferdftgx-sml/exec";

        window.onload = function() {
            setTodayDateAll();
            calculateBenefitsRenewal();
            calculateTableTotalsRenewal();
            calculateBenefitsTotal();
            calculateTableTotalsTotal();
            attachCommaFormattingHai();
            recalcAllHaiorder();
            initKeyLock();
        }

        /* 단축키 및 무단 복사 차단 스크립트 */
        function initKeyLock() {
            document.addEventListener('contextmenu', e => e.preventDefault());
            document.addEventListener('dragstart', e => e.preventDefault());
            document.addEventListener('selectstart', e => e.preventDefault());

            document.addEventListener('keydown', function(e) {
                if (e.keyCode === 123 || 
                   (e.ctrlKey && e.shiftKey && (e.keyCode === 73 || e.keyCode === 74 || e.keyCode === 67)) || 
                   (e.ctrlKey && (e.keyCode === 85 || e.keyCode === 83 || e.keyCode === 67 || e.keyCode === 65))) {
                    e.preventDefault();
                    e.stopPropagation();
                    return false;
                }
            });
        }

        function setTodayDateAll() {
            const today = new Date();
            const yyyy = today.getFullYear();
            const mm = String(today.getMonth() + 1).padStart(2, '0');
            const dd = String(today.getDate()).padStart(2, '0');
            document.querySelectorAll('.invoice-date').forEach(el => {
                el.value = `${yyyy}-${mm}-${dd}`;
            });
        }

        function switchEstimateTab(tabName) {
            activeTab = tabName;
            document.getElementById('capture-area-renewal').style.display = (tabName === 'renewal') ? 'block' : 'none';
            document.getElementById('capture-area-total').style.display = (tabName === 'total') ? 'block' : 'none';
            document.getElementById('capture-area-haiorder').style.display = (tabName === 'haiorder') ? 'block' : 'none';
            
            document.getElementById('btn-renewal').classList.toggle('active', tabName === 'renewal');
            document.getElementById('btn-total').classList.toggle('active', tabName === 'total');
            document.getElementById('btn-haiorder').classList.toggle('active', tabName === 'haiorder');
        }

        function clearGuidance(el) { if(el.value === " 귀하") { el.value = ""; } }
        function restoreGuidance(el) {
            if(el.value.trim() === "") { el.value = " 귀하"; } 
            else if(!el.value.endsWith(" 귀하")) { el.value = el.value + " 귀하"; }
        }

        function updateManagerInfo(selectEl, scope) {
            const selectedOption = selectEl.options[selectEl.selectedIndex];
            const phoneInput = document.getElementById(`manager-phone-${scope}`);
            const emailInput = document.getElementById(`manager-email-${scope}`);

            if (selectedOption.value === 'custom') {
                phoneInput.value = '';
                emailInput.value = '';
                phoneInput.removeAttribute('readonly');
                emailInput.removeAttribute('readonly');
                phoneInput.placeholder = '연락처 직접 입력';
                emailInput.placeholder = '이메일 직접 입력';
                phoneInput.focus();
            } else {
                phoneInput.value = selectedOption.getAttribute('data-phone') || '';
                emailInput.value = selectedOption.getAttribute('data-email') || '';
                phoneInput.setAttribute('readonly', true);
                emailInput.setAttribute('readonly', true);
            }
        }

        function checkDateValue(el) {
            if (el.value) { el.classList.add('has-value'); } else { el.classList.remove('has-value'); }
        }

        /* ----- 백엔드(Google Sheets/Solapi) 연동 스크립트 ----- */
        function sendQuoteDataGas() {
            let userPhone = prompt("DB 적재 및 안내 문자를 수신할 고객/담당자 연락처를 입력하세요:", "01082909971");
            if (!userPhone) {
                alert("연락처가 입력되지 않아 전송이 취소되었습니다.");
                return;
            }

            let payload = {};

            if (activeTab === 'haiorder') {
                let storeName = document.getElementById('hai-store-name').value.replace(' 귀하', '').trim() || '미입력 매장';
                payload = {
                    quoteType: '하이오더',
                    storeName: storeName,
                    userPhone: userPhone,
                    consultDay: '견적서 즉시발행',
                    consultTime: '상관없음',
                    table: document.getElementById('hai-table-qty').value,
                    reader: document.getElementById('hai-reader-qty').value,
                    k10: document.getElementById('hai-k10-qty').value,
                    k15: document.getElementById('hai-k15-qty').value,
                    waiting: parseInt(document.getElementById('hai-waiting-qty').value) > 0 ? '포함' : '미포함',
                    monthlyPrice: document.getElementById('topQuoteAmount').innerText,
                    oneTimePrice: document.getElementById('hai-onetime-total').innerText + ' 원'
                };
            } else if (activeTab === 'renewal') {
                let storeName = document.getElementById('renewal-client-name').value.replace(' 귀하', '').trim() || '미입력 법인';
                let totalDiff = document.getElementById('total-diff-renewal').innerText;
                payload = {
                    quoteType: '법인회선',
                    storeName: storeName,
                    userPhone: userPhone,
                    consultDay: '견적서 즉시발행',
                    consultTime: '상관없음',
                    corpType: '법인사업자',
                    corpCurrentTelecom: 'KT',
                    corpCurrentFee: '월 ' + document.getElementById('total-charge-renewal').innerText + '원',
                    corpQueryType: '법인회선 재약정 문의 (요금변동: ' + totalDiff + '원)'
                };
            } else {
                let storeName = document.getElementById('total-client-name').value.replace(' 귀하', '').trim() || '미입력 고객';
                payload = {
                    quoteType: '인터넷/유무선',
                    storeName: storeName,
                    userPhone: userPhone,
                    consultDay: '견적서 즉시발행',
                    consultTime: '상관없음',
                    inetTelecom: 'KT',
                    inetProducts: '유무선 통합견적',
                    inetPeriod: '3년 약정',
                    mobileTelecom: 'KT',
                    isCombined: '결합적용',
                    inflowPath: '견적서시스템'
                };
            }

            fetch(GAS_URL, {
                method: 'POST',
                mode: 'no-cors',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(payload)
            }).then(() => {
                alert('구글 시트 DB 적재 및 Solapi 알림 문자 전송이 성공적으로 완료되었습니다.');
            }).catch(err => {
                alert('전송 중 오류가 발생했습니다: ' + err);
            });
        }

        /* ----- 액션 스크립트 ----- */
        function recalcCurrentActiveTab() {
            if(activeTab === 'renewal') {
                calculateBenefitsRenewal();
                calculateTableTotalsRenewal();
            } else if(activeTab === 'total') {
                calculateBenefitsTotal();
                calculateTableTotalsTotal();
            } else if(activeTab === 'haiorder') {
                recalcAllHaiorder();
            }
            alert('현재 견적서의 모든 합계 및 금액이 새로고침되었습니다.');
        }

        function resetActiveTabForm() {
            if(!confirm('현재 활성화된 견적서의 모든 입력 데이터를 초기화하시겠습니까?')) return;
            const targetContainer = document.getElementById(`capture-area-${activeTab}`);
            
            if(activeTab === 'haiorder') {
                targetContainer.querySelectorAll('.device-qty, .reader-qty').forEach(inp => { inp.value = '0'; });
                targetContainer.querySelectorAll('[data-group="g3"] .qty-input:not(.device-qty):not(.service-qty)').forEach(inp => { inp.value = '0'; });
                const offerPriceInput = document.getElementById('offerPriceInput');
                if(offerPriceInput) offerPriceInput.value = '0';
                recalcAllHaiorder();
            } else {
                targetContainer.querySelectorAll('input[type="text"]:not([readonly]), textarea').forEach(inp => {
                    if(!inp.classList.contains('invoice-date') && !inp.classList.contains('blue-readonly')) {
                        inp.value = '';
                    }
                });
                targetContainer.querySelectorAll('select').forEach(sel => { sel.selectedIndex = 0; });
                recalcCurrentActiveTab();
            }
        }

        function saveCurrentEstimateData() {
            const targetContainer = document.getElementById(`capture-area-${activeTab}`);
            const formData = {};

            targetContainer.querySelectorAll('input, select, textarea').forEach((el, index) => {
                if(el.id) {
                    formData[`id_${el.id}`] = el.value;
                } else {
                    formData[`idx_${index}`] = el.value;
                }
            });

            const storageKey = `saved_estimate_${activeTab}`;
            localStorage.setItem(storageKey, JSON.stringify(formData));
            alert('현재 견적서의 입력 내용이 브라우저에 정상적으로 저장되었습니다.');
        }

        function loadSavedEstimateData() {
            const storageKey = `saved_estimate_${activeTab}`;
            const savedData = localStorage.getItem(storageKey);

            if(!savedData) {
                alert('저장된 견적서 데이터가 존재하지 않습니다.');
                return;
            }

            if(!confirm('저장된 최근 견적서를 불러오시겠습니까? 기존 작성 내용은 덮어씌워집니다.')) return;

            const formData = JSON.parse(savedData);
            const targetContainer = document.getElementById(`capture-area-${activeTab}`);

            targetContainer.querySelectorAll('input, select, textarea').forEach((el, index) => {
                const keyById = `id_${el.id}`;
                const keyByIdx = `idx_${index}`;

                if(el.id && formData[keyById] !== undefined) {
                    el.value = formData[keyById];
                } else if(formData[keyByIdx] !== undefined) {
                    el.value = formData[keyByIdx];
                }
            });

            recalcCurrentActiveTab();
            alert('저장된 견적서 데이터를 성공적으로 불러왔습니다.');
        }

        /* ----- 재약정 견적서 계산 로직 ----- */
        function runCalculationsRenewal(el) {
            formatNumberInput(el);
            calculateRowDifferenceRenewal(el.closest('tr'));
            calculateTableTotalsRenewal();
        }

        function calculateRowDifferenceRenewal(row) {
            const chargeInput = row.querySelector('.calc-charge-renewal');
            const renewInput = row.querySelector('.calc-renew-renewal');
            const diffInput = row.querySelector('.calc-diff-renewal');
            if(!chargeInput || !renewInput || !diffInput) return;

            const charge = parseInt(chargeInput.value.replace(/,/g, '')) || 0;
            const renew = parseInt(renewInput.value.replace(/,/g, '')) || 0;

            if(chargeInput.value === "" && renewInput.value === "") {
                diffInput.value = "";
                return;
            }
            const diff = renew - charge;
            diffInput.value = (diff >= 0 ? "+" : "") + diff.toLocaleString();
        }

        function runBenefitCalculationsRenewal(el) {
            formatNumberInput(el, "0");
            calculateBenefitsRenewal();
        }

        function calculateBenefitsRenewal() {
            const fee = parseInt(document.getElementById('fee-renewal').value.replace(/,/g, '')) || 0;
            const gift = parseInt(document.getElementById('gift-renewal').value.replace(/,/g, '')) || 0;
            document.getElementById('total-benefits-renewal').innerText = '₩' + (fee + gift).toLocaleString();
        }

        function calculateTableTotalsRenewal() {
            let totalCharge = 0, totalRenew = 0;
            document.querySelectorAll('.calc-charge-renewal').forEach(i => { totalCharge += parseInt(i.value.replace(/,/g, '')) || 0; });
            document.querySelectorAll('.calc-renew-renewal').forEach(i => { totalRenew += parseInt(i.value.replace(/,/g, '')) || 0; });
            
            document.getElementById('total-charge-renewal').innerText = totalCharge.toLocaleString();
            document.getElementById('total-renew-renewal').innerText = totalRenew.toLocaleString();
            
            const totalDiff = totalRenew - totalCharge;
            const diffDisplay = document.getElementById('total-diff-renewal');
            if(totalCharge === 0 && totalRenew === 0) {
                diffDisplay.innerText = "0";
            } else {
                diffDisplay.innerText = (totalDiff >= 0 ? "+" : "") + totalDiff.toLocaleString();
            }
        }

        /* ----- 유무선 통합 견적서 계산 로직 ----- */
        function runCalculationsTotal(el) {
            formatNumberInput(el);
            calculateRowTotal(el.closest('tr'));
            calculateTableTotalsTotal();
        }

        function calculateRowTotal(row) {
            const qtyInput = row.querySelector('.calc-qty-total');
            const baseInput = row.querySelector('.calc-base-total');
            const discountInput = row.querySelector('.calc-discount-total');
            const finalInput = row.querySelector('.calc-final-total');
            if(!qtyInput || !baseInput || !finalInput) return;

            const qty = parseInt(qtyInput.value.replace(/,/g, '')) || 1;
            const base = parseInt(baseInput.value.replace(/,/g, '')) || 0;
            const discount = parseInt(discountInput.value.replace(/,/g, '')) || 0;

            if(baseInput.value === "" && discountInput.value === "") {
                finalInput.value = "";
                return;
            }

            const monthlyOne = Math.max(0, base - discount);
            const rowTotal = monthlyOne * qty;
            finalInput.value = rowTotal.toLocaleString();
        }

        function runBenefitCalculationsTotal(el) {
            formatNumberInput(el, "0");
            calculateBenefitsTotal();
        }

        function calculateBenefitsTotal() {
            const fee = parseInt(document.getElementById('fee-total').value.replace(/,/g, '')) || 0;
            const gift = parseInt(document.getElementById('gift-total').value.replace(/,/g, '')) || 0;
            document.getElementById('total-benefits-total').innerText = '₩' + (fee + gift).toLocaleString();
        }

        function calculateTableTotalsTotal() {
            let totalQty = 0, totalBase = 0, totalDiscount = 0, totalFinal = 0;
            
            document.querySelectorAll('#product-table-total tbody tr:not(.total-row)').forEach(row => {
                const qty = parseInt(row.querySelector('.calc-qty-total').value.replace(/,/g, '')) || 0;
                const base = parseInt(row.querySelector('.calc-base-total').value.replace(/,/g, '')) || 0;
                const discount = parseInt(row.querySelector('.calc-discount-total').value.replace(/,/g, '')) || 0;
                const finalVal = parseInt(row.querySelector('.calc-final-total').value.replace(/,/g, '')) || 0;

                if(base > 0 || qty > 0) {
                    totalQty += (qty || 1);
                    totalBase += (base * (qty || 1));
                    totalDiscount += (discount * (qty || 1));
                    totalFinal += finalVal;
                }
            });

            document.getElementById('total-qty-total').innerText = totalQty.toLocaleString();
            document.getElementById('total-base-total').innerText = totalBase.toLocaleString();
            document.getElementById('total-discount-total').innerText = totalDiscount.toLocaleString();
            document.getElementById('total-final-total').innerText = totalFinal.toLocaleString();
        }

        /* ----- 하이오더 견적서 계산 로직 ----- */
        function formatWithComma(n){ return Number(String(n).replace(/[^\d]/g,'')).toLocaleString('ko-KR'); }
        function won(n){ return Math.round(n).toLocaleString('ko-KR') + '원'; }
        function parseNum(str){ return parseFloat(String(str).replace(/,/g,'')) || 0; }

        function changeQty(btn, amount) {
            const input = btn.parentNode.querySelector('input');
            let currentQty = parseNum(input.value);
            currentQty += amount;
            if(currentQty < 0) currentQty = 0;
            input.value = formatWithComma(currentQty);
            recalcAllHaiorder();
        }

        function attachCommaFormattingHai(){
            document.querySelectorAll('#capture-area-haiorder .qty-input, #capture-area-haiorder .price-input, #capture-area-haiorder .qty4').forEach(inp=>{
                inp.addEventListener('input', (e)=>{
                    const cursorFromEnd = inp.value.length - inp.selectionStart;
                    let raw = inp.value.replace(/[^\d]/g,'');
                    if(raw === ''){ raw = '0'; }
                    inp.value = formatWithComma(raw);
                    const pos = inp.value.length - cursorFromEnd;
                    inp.setSelectionRange(pos, pos);
                    recalcAllHaiorder();
                });
            });
        }

        function recalcGroupHai(groupSelector){
            const table = document.querySelector(groupSelector);
            if(!table) return {total:0, deviceQty:0, firstRowQty:0};

            let deviceQty = 0;
            let firstRowQty = 0;
            const devInputs = table.querySelectorAll('.device-qty');
            devInputs.forEach((inp, idx)=>{
                const val = parseNum(inp.value);
                deviceQty += val;
                if(idx === 0) firstRowQty = val; 
            });

            table.querySelectorAll('.service-qty').forEach(inp=>{
                inp.value = formatWithComma(deviceQty);
            });

            let sumSupply=0, sumTax=0, sumTotal=0;
            table.querySelectorAll('tr.item-row').forEach(row=>{
                const qtyInput = row.querySelector('.qty-input');
                const qty = qtyInput ? parseNum(qtyInput.value) : 0;
                const price = parseNum(row.querySelector('.price').value);
                const supply = qty*price;
                const tax = Math.round(supply*0.1);
                const total = supply+tax;
                
                row.querySelector('.supply').textContent = supply.toLocaleString('ko-KR');
                row.querySelector('.tax').textContent = tax.toLocaleString('ko-KR');
                row.querySelector('.total').textContent = total.toLocaleString('ko-KR');

                sumSupply+=supply; sumTax+=tax; sumTotal+=total;
            });
            table.querySelector('.g-supply').textContent = sumSupply.toLocaleString('ko-KR');
            table.querySelector('.g-tax').textContent = sumTax.toLocaleString('ko-KR');
            table.querySelector('.g-total').textContent = sumTotal.toLocaleString('ko-KR');
            return {total: sumTotal, deviceQty, firstRowQty};
        }

        function recalcGroup4Hai(totalDeviceQty){
            const table = document.querySelector('[data-group="g4"]');
            if(!table) return;
            
            const installPriceInput = document.getElementById('installPriceInput');
            if(installPriceInput) {
                const calcInstallPrice = 250000 + (10000 * totalDeviceQty);
                installPriceInput.value = formatWithComma(calcInstallPrice);
            }

            document.querySelectorAll('.auto-device-qty').forEach(inp=>{
                inp.value = formatWithComma(totalDeviceQty);
            });
            
            let sumSupply=0, sumTax=0, sumTotal=0;
            table.querySelectorAll('tr.item-row4').forEach(row=>{
                const qty = parseNum(row.querySelector('.qty4').value);
                const price = parseNum(row.querySelector('.price4').value);
                
                let supply = qty * price;
                if(row.classList.contains('install-fee-row')) {
                    supply = price; 
                }
                
                const tax = Math.round(supply*0.1);
                const total = supply+tax;
                
                row.querySelector('.supply4').textContent = supply.toLocaleString('ko-KR');
                row.querySelector('.tax4').textContent = tax.toLocaleString('ko-KR');
                row.querySelector('.total4').textContent = total.toLocaleString('ko-KR');
                sumSupply+=supply; sumTax+=tax; sumTotal+=total;
            });
            table.querySelector('.g-supply4').textContent = sumSupply.toLocaleString('ko-KR');
            table.querySelector('.g-tax4').textContent = sumTax.toLocaleString('ko-KR');
            table.querySelector('.g-total4').textContent = sumTotal.toLocaleString('ko-KR');
        }

        function calcOfferTotalHai(menuTabletQty) {
            const offerPriceInput = document.getElementById('offerPriceInput');
            const offerTotalText = document.getElementById('offerTotalText');
            if(!offerPriceInput || !offerTotalText) return;

            const offerUnitPrice = parseNum(offerPriceInput.value);
            const totalOfferPrice = offerUnitPrice * menuTabletQty;
            offerTotalText.textContent = `(총합계: ${totalOfferPrice.toLocaleString('ko-KR')}원)`;
        }

        function recalcAllHaiorder(){
            const r1 = recalcGroupHai('[data-group="g1"]');
            const r2 = recalcGroupHai('[data-group="g2"]');
            const r3 = recalcGroupHai('[data-group="g3"]');
            
            calcOfferTotalHai(r1.firstRowQty);

            const totalDeviceQty = r1.deviceQty + r2.deviceQty + r3.deviceQty;
            recalcGroup4Hai(totalDeviceQty);
            const haiOrderTotal = r1.total + r2.total + r3.total;
            
            const hTotalEl = document.getElementById('haiorderTotal');
            const tQuoteEl = document.getElementById('topQuoteAmount');
            if(hTotalEl) hTotalEl.textContent = won(haiOrderTotal);
            if(tQuoteEl) tQuoteEl.textContent = won(haiOrderTotal);
        }

        function formatNumberInput(el, defaultVal = "") {
            let cursorPosition = el.selectionStart;
            let originalLength = el.value.length;
            let value = el.value.replace(/[^0-9]/g, '');
            if (value !== "") { el.value = parseInt(value).toLocaleString(); } else { el.value = defaultVal; }
            let newLength = el.value.length;
            el.setSelectionRange(cursorPosition + (newLength - originalLength), cursorPosition + (newLength - originalLength));
        }

        /* ----- 이미지 및 PDF 생성 엔진 ----- */
        function generateActiveInvoiceImage(format) {
            const originArea = document.getElementById(`capture-area-${activeTab}`);
            if (document.activeElement) { document.activeElement.blur(); }

            originArea.querySelectorAll('.no-print-target').forEach(btn => btn.classList.add('hide-for-photo'));

            const cloneArea = originArea.cloneNode(true);
            cloneArea.style.position = 'fixed';
            cloneArea.style.top = '-9999px';
            cloneArea.style.left = '-9999px';
            cloneArea.style.display = 'block';
            document.body.appendChild(cloneArea);

            const originInputs = originArea.querySelectorAll('input, select, textarea');
            const cloneInputs = cloneArea.querySelectorAll('input, select, textarea');

            originInputs.forEach((originInput, idx) => {
                const cloneInput = cloneInputs[idx];
                if (!cloneInput) return;
                let text = '';
                if (originInput.tagName === 'SELECT') {
                    text = originInput.options[originInput.selectedIndex] ? originInput.options[originInput.selectedIndex].text : '';
                    if (text.includes('--선택--')) text = ' ';
                } else if (originInput.type === 'date') {
                    text = originInput.value || ' ';
                } else if (originInput.tagName === 'TEXTAREA') {
                    text = originInput.value || ' ';
                } else {
                    text = originInput.value || originInput.placeholder || ' ';
                }

                const textNode = document.createElement('span');
                textNode.className = 'capture-text-node';
                if(originInput.classList.contains('blue-readonly')) {
                    textNode.style.color = '#004b8d';
                    textNode.style.fontWeight = 'bold';
                }

                if (originInput.tagName === 'TEXTAREA') {
                    textNode.style.whiteSpace = 'pre-wrap';
                    textNode.style.textAlign = 'left';
                    textNode.style.padding = '6px';
                    textNode.style.display = 'block';
                }

                textNode.innerText = text;
                if(cloneInput.parentNode) {
                    cloneInput.parentNode.replaceChild(textNode, cloneInput);
                }
            });

            html2canvas(cloneArea, {
                scale: 3,
                useCORS: true,
                backgroundColor: '#ffffff',
                logging: false,
                letterRendering: true
            }).then(canvas => {
                const imageData = canvas.toDataURL('image/jpeg', 1.0);
                let fileName = '견적서';
                if(activeTab === 'renewal') fileName = '법인회선_재약정_견적서';
                else if(activeTab === 'total') fileName = '유무선_통합_견적서';
                else if(activeTab === 'haiorder') {
                    let storeName = document.getElementById('hai-store-name').value.trim() || "매장";
                    storeName = storeName.replace(" 귀하", "");
                    fileName = `하이오더_견적서_${storeName}`;
                }

                if (format === 'jpg') {
                    const link = document.createElement('a');
                    link.href = imageData;
                    link.download = `${fileName}.jpg`;
                    link.click();
                } else if (format === 'pdf') {
                    const { jsPDF } = window.jspdf;
                    const pdf = new jsPDF('p', 'mm', 'a4');
                    const imgProps = pdf.getImageProperties(imageData);
                    const pdfWidth = pdf.internal.pageSize.getWidth();
                    const pdfHeight = (imgProps.height * pdfWidth) / imgProps.width;
                    pdf.addImage(imageData, 'JPEG', 0, 0, pdfWidth, pdfHeight);
                    pdf.save(`${fileName}.pdf`);
                }
                originArea.querySelectorAll('.no-print-target').forEach(btn => btn.classList.remove('hide-for-photo'));
                document.body.removeChild(cloneArea);
            }).catch(err => {
                originArea.querySelectorAll('.no-print-target').forEach(btn => btn.classList.remove('hide-for-photo'));
                if(document.body.contains(cloneArea)) document.body.removeChild(cloneArea);
                alert("파일 생성 중 오류가 발생했습니다: " + err);
            });
        }
    </script>
</body>
</html>

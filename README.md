<html lang="ko">
<head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>KT 통합 견적서 및 BIZ 시뮬레이션 시스템</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/html2pdf.js/0.10.1/html2pdf.bundle.min.js"></script>
    <style>
        @page { size: A4; margin: 0; }
        * { 
            box-sizing: border-box; 
            font-family: 'Pretendard', -apple-system, BlinkMacSystemFont, 'Apple SD Gothic Neo', 'Malgun Gothic', '맑은 고딕', sans-serif;
            -webkit-user-select: none; -moz-user-select: none; -ms-user-select: none; user-select: none;
        }
        body {
            background-color: #f4f6f9;
            padding: 20px 0;
            margin: 0;
            display: flex; flex-direction: column; align-items: center; justify-content: center;
            -webkit-font-smoothing: antialiased; position: relative;
        }

        @media (prefers-color-scheme: dark) {
            body { background-color: #f4f6f9 !important; }
            .invoice-container { background-color: #ffffff !important; color: #000000 !important; }
            th { background-color: #f1f5f9 !important; color: #000000 !important; }
            td { background-color: #ffffff !important; color: #000000 !important; border-color: #a0a0a0 !important; }
            input[type="text"], select, textarea { background-color: #f8fafc !important; color: #000000 !important; }
        }

        /* 🔒 전체 화면 보안 인증 오버레이 */
        .auth-overlay {
            position: fixed; top: 0; left: 0; width: 100vw; height: 100vh;
            background: rgba(15, 23, 42, 0.92); backdrop-filter: blur(8px);
            z-index: 999999; display: flex; justify-content: center; align-items: center;
        }
        .auth-card {
            background: #ffffff; width: 90%; max-width: 360px;
            border-radius: 16px; border-top: 6px solid #004b8d;
            box-shadow: 0 25px 50px -12px rgba(0, 75, 141, 0.35);
            padding: 28px 22px; text-align: center;
            animation: popIn 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275) forwards;
        }
        .auth-icon { font-size: 40px; margin-bottom: 8px; display: inline-block; }
        .auth-title { font-size: 19px; font-weight: 900; color: #004b8d; margin-bottom: 6px; letter-spacing: -0.5px; }
        .auth-sub { font-size: 12px; font-weight: 600; color: #64748b; margin-bottom: 18px; line-height: 1.4; }
        
        .auth-input {
            width: 100%; height: 44px; border: 2px solid #cbd5e1 !important;
            border-radius: 8px; font-size: 16px; font-weight: 700;
            text-align: center !important; letter-spacing: 4px; color: #1e293b !important;
            background: #f8fafc !important; margin-bottom: 12px; outline: none; transition: border-color 0.2s;
        }
        .auth-input:focus { border-color: #004b8d !important; background: #ffffff !important; }
        
        .auth-btn {
            width: 100%; height: 44px; background: #004b8d; color: #ffffff;
            font-size: 15px; font-weight: 800; border: none; border-radius: 8px;
            cursor: pointer; transition: background 0.2s, transform 0.1s;
        }
        .auth-btn:hover { background: #003666; }
        .auth-btn:active { transform: scale(0.98); }
        .auth-error {
            color: #dc2626; font-size: 12px; font-weight: 700; margin-top: 10px; display: none;
        }

        /* 📩 비밀번호 신청하기 링크 및 서브 폼 스타일 */
        .auth-req-link {
            margin-top: 14px; font-size: 12px; color: #64748b; font-weight: 600; cursor: pointer; text-decoration: underline; transition: color 0.2s;
        }
        .auth-req-link:hover { color: #0284c7; }
        
        .req-form-area {
            display: none; margin-top: 14px; padding-top: 14px; border-top: 1px dashed #cbd5e1;
        }
        .req-input {
            width: 100%; height: 38px; border: 1px solid #cbd5e1 !important;
            border-radius: 6px; font-size: 13px; font-weight: 600;
            text-align: center !important; color: #1e293b !important;
            background: #f8fafc !important; margin-bottom: 8px; outline: none;
        }
        .req-input:focus { border-color: #0284c7 !important; background: #ffffff !important; }
        .req-submit-btn {
            width: 100%; height: 38px; background: #0284c7; color: #ffffff;
            font-size: 13.5px; font-weight: 800; border: none; border-radius: 6px;
            cursor: pointer; transition: background 0.2s;
        }
        .req-submit-btn:hover { background: #0369a1; }

        .shake { animation: shake 0.35s ease-in-out; }
        @keyframes shake {
            0%, 100% { transform: translateX(0); }
            20%, 60% { transform: translateX(-8px); }
            40%, 80% { transform: translateX(8px); }
        }

        .watermark-overlay {
            position: absolute; top: 50%; left: 50%;
            transform: translate(-50%, -50%) rotate(-25deg);
            font-size: 21px; font-weight: 900;
            color: rgba(0, 75, 141, 0.08) !important;
            white-space: nowrap; pointer-events: none; z-index: 999;
            letter-spacing: -0.5px; text-align: center;
        }

        .signature-badge {
            width: 794px; margin-bottom: 12px; padding: 10px 14px;
            background: #f0f9ff; border: 1px solid #bae6fd; border-radius: 8px;
            text-align: center; box-shadow: 0 2px 6px rgba(2, 132, 199, 0.06);
        }

        .red-alert-overlay {
            position: fixed; top: 0; left: 0; width: 100vw; height: 100vh;
            background: rgba(15, 23, 42, 0.7); backdrop-filter: blur(4px);
            z-index: 99999; display: none; justify-content: center; align-items: center;
            animation: fadeIn 0.2s ease-out;
        }
        .red-alert-card {
            background: #ffffff; width: 90%; max-width: 380px;
            border-radius: 16px; border-top: 6px solid #dc2626;
            box-shadow: 0 20px 30px rgba(220, 38, 38, 0.25);
            padding: 24px 20px; text-align: center;
            animation: popIn 0.25s cubic-bezier(0.175, 0.885, 0.32, 1.275) forwards;
        }
        .red-alert-icon { font-size: 40px; margin-bottom: 8px; display: inline-block; animation: pulse 1.2s infinite; }
        .red-alert-title { font-size: 18px; font-weight: 900; color: #dc2626; margin-bottom: 10px; letter-spacing: -0.5px; }
        .red-alert-msg {
            font-size: 15px; font-weight: 800; color: #1e293b; background: #fef2f2;
            border: 1px solid #fecaca; padding: 12px; border-radius: 8px; margin-bottom: 18px; line-height: 1.4;
        }
        .red-alert-btn {
            width: 100%; height: 42px; background: #dc2626; color: #ffffff;
            font-size: 14px; font-weight: 800; border: none; border-radius: 8px; cursor: pointer; transition: background 0.2s;
        }
        .red-alert-btn:hover { background: #b91c1c; }

        @keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }
        @keyframes popIn { from { transform: scale(0.85); opacity: 0; } to { transform: scale(1); opacity: 1; } }
        @keyframes pulse { 0%, 100% { transform: scale(1); } 50% { transform: scale(1.15); } }

        /* 대분류 메인 탭 메뉴 */
        .tab-menu-container {
            width: 794px; display: flex; gap: 8px; margin-bottom: 12px;
        }
        .tab-btn {
            flex: 1; padding: 12px 10px; font-size: 13.5px; font-weight: 800;
            color: #475569; background-color: #e2e8f0; border: none; border-radius: 8px 8px 0 0;
            cursor: pointer; transition: all 0.2s ease; text-align: center;
        }
        .tab-btn.active {
            color: #ffffff; background-color: #004b8d; box-shadow: 0 -2px 8px rgba(0, 75, 141, 0.2);
        }

        .toolbar-area {
            width: 100%; display: flex; justify-content: space-between; align-items: center;
            background-color: #f1f5f9; padding: 8px 12px; border-radius: 6px; margin-bottom: 12px; border: 1px solid #cbd5e1;
        }
        .toolbar-group { display: flex; gap: 6px; }
        .tool-btn {
            padding: 6px 12px; font-size: 11.5px; font-weight: 700; border-radius: 4px;
            cursor: pointer; border: 1px solid #cbd5e1; background-color: #ffffff; color: #334155; transition: all 0.2s;
        }
        .tool-btn:hover { background-color: #e2e8f0; }
        .tool-btn.btn-send { background-color: #0284c7; color: #ffffff; border-color: #0284c7; }
        .tool-btn.btn-send:hover { background-color: #0369a1; }
        .tool-btn.btn-save { background-color: #0d9488; color: #ffffff; border-color: #0d9488; }
        .tool-btn.btn-save:hover { background-color: #0f766e; }
        .tool-btn.btn-reset { background-color: #ef4444; color: #ffffff; border-color: #ef4444; }
        .tool-btn.btn-reset:hover { background-color: #dc2626; }

        .invoice-container {
            width: 794px; background-color: #ffffff; padding: 25px 35px;
            box-shadow: 0 4px 20px rgba(0,0,0,0.08); box-sizing: border-box;
            image-rendering: -webkit-optimize-contrast; margin: 0 auto; position: relative; overflow: hidden;
        }

        .invoice-header {
            display: flex; justify-content: space-between; align-items: center;
            margin-bottom: 15px; border-bottom: 3px solid #1c5280; padding-bottom: 6px;
        }
        .logo-area {
            font-size: 28px; font-weight: 900; color: #000000 !important;
            font-family: 'Arial Black', Impact, sans-serif; letter-spacing: -2px; line-height: 1;
        }
        .title-area {
            font-size: 24px; font-weight: 800; color: #004b8d !important;
            text-align: center flex-grow: 1; letter-spacing: -0.5px; margin-right: 40px;
        }

        table {
            width: 100%; border-collapse: collapse; font-size: 11px; color: #000000 !important;
            margin-bottom: 8px; table-layout: fixed; box-sizing: border-box;
        }
        th, td {
            border: 1px solid #a0a0a0 !important; padding: 4px 5px; height: 26px;
            color: #000000 !important; vertical-align: middle !important; text-align: center !important;
            box-sizing: border-box; overflow: hidden;
        }
        th { background-color: #f1f5f9 !important; font-weight: 700 !important; color: #000000 !important; }

        .info-table th { width: 14%; }
        .info-table td { width: 36%; }

        .notice-container-table th { width: 11% !important; }
        .notice-container-table td { width: 89% !important; }

        .product-table th { background-color: #f1f5f9 !important; color: #000000 !important; font-weight: 700; font-size: 11px; }

        input[type="text"], input[type="date"], input[type="number"], select, textarea {
            width: 100%; height: 100%; border: none !important; background-color: #f8fafc !important;
            padding: 0 4px; border-radius: 3px; font-size: 11px; font-family: inherit;
            color: #000000 !important; -webkit-text-fill-color: #000000 !important; opacity: 1 !important;
            font-weight: 600; box-sizing: border-box; outline: none; text-align: center !important; box-shadow: none !important;
        }

        textarea { text-align: left !important; padding: 6px !important; resize: none; height: 55px; line-height: 1.4; }

        /* html2canvas 변환 시 텍스트 노드 붕괴 완벽 차단 규격 */
        .capture-text-node {
            font-size: 11px !important; font-weight: 600 !important; color: #000000 !important;
            text-align: center !important; width: 100% !important; display: block !important;
            line-height: 24px !important; height: 24px !important; white-space: nowrap !important;
            overflow: hidden !important; text-overflow: ellipsis !important; box-sizing: border-box !important;
        }

        input[type="date"] { color: transparent !important; -webkit-text-fill-color: transparent !important; }
        input[type="date"].has-value { color: #000000 !important; -webkit-text-fill-color: #000000 !important; }

        input:focus, select:focus, textarea:focus { background-color: #e0f2fe !important; }
        input[readonly], select[readonly], .lock-cell {
            color: #475569 !important; -webkit-text-fill-color: #475569 !important;
            background-color: #f1f5f9 !important; font-weight: 600; cursor: not-allowed;
        }
        .blue-readonly { color: #004b8d !important; }
        .benefit-highlight {
            color: #d91414 !important; font-size: 13px; font-weight: 800;
            text-align: center !important; background-color: #fef2f2 !important; letter-spacing: 0.5px;
        }

        .total-row { background-color: #e2e8f0 !important; font-weight: 700; }

        .notice-text {
            font-size: 10.5px; color: #222222 !important; line-height: 1.45; font-weight: 500;
            text-align: left !important; white-space: normal !important; overflow: visible !important;
        }
        .bg-alert { background-color: #fffbeb !important; color: #b45309 !important; }

        /* BIZ 대량 계산기 전용 소분류 탭 스타일 */
        .biz-sub-tab-container {
            display: flex; gap: 4px; margin-bottom: 12px; border-bottom: 2px solid #cbd5e1;
        }
        .biz-sub-tab-btn {
            padding: 6px 14px; font-size: 11px; font-weight: 700; border: 1px solid #cbd5e1;
            border-bottom: none; border-radius: 4px 4px 0 0; background-color: #f1f5f9; color: #64748b; cursor: pointer;
        }
        .biz-sub-tab-btn.active { background-color: #e60012; color: #ffffff; border-color: #e60012; }

        /* BIZ 입력 영역 Table 변환용 전용 스타일 (렌더링 무결성) */
        .biz-input-table {
            width: 100% !important; margin-bottom: 12px !important; border: 1px solid #cbd5e1 !important;
            background-color: #f8fafc !important; table-layout: fixed !important;
        }
        .biz-input-table th {
            text-align: left !important; padding-left: 8px !important; background-color: #f1f5f9 !important;
            font-size: 10.5px !important; font-weight: 700 !important; color: #334155 !important;
        }
        .biz-input-table td {
            background-color: #ffffff !important; padding: 2px 4px !important;
        }
        .card-title { 
            font-size: 11px; font-weight: bold; color: #1e293b; margin-bottom: 6px; 
            border-left: 4px solid #e60012; padding-left: 6px; text-align: left !important; 
        }

        .summary-banner {
            background: linear-gradient(135deg, #f8fafc 0%, #f1f5f9 100%); border: 1.5px solid #cbd5e1;
            border-radius: 6px; padding: 10px 14px; display: flex; justify-content: space-between; align-items: center; margin-bottom: 12px;
        }
        .summary-val { font-size: 15px; font-weight: 800; color: #1e293b; }
        .summary-sub { font-size: 10px; color: #475569; }

        .btn-area {
            margin-top: 15px; margin-bottom: 20px; width: 794px; display: flex; gap: 15px;
        }
        .download-btn {
            flex: 1; background-color: #004b8d; color: white; border: none; padding: 14px;
            font-size: 16px; font-weight: 700; cursor: pointer; border-radius: 6px;
            box-shadow: 0 4px 12px rgba(0, 61, 141, 0.25); transition: background 0.2s;
        }
        .download-btn:hover { background-color: #143757; }
        .pdf-btn { background-color: #10b981; box-shadow: 0 4px 12px rgba(16, 185, 129, 0.25); }
        .pdf-btn:hover { background-color: #047857; }

        .responsive-wrapper {
            width: 100%; overflow-x: auto; display: flex; flex-direction: column; align-items: center;
        }
    
        /* 수량 조절 박스 */
        .qty-box {
            display: inline-flex !important; flex-direction: row !important;
            align-items: center !important; justify-content: center !important;
            gap: 4px !important; width: auto !important; margin: 0 auto !important;
        }
        .qty-box input {
            width: 38px !important; height: 22px !important; font-size: 11px !important;
            font-weight: 700 !important; text-align: center !important; padding: 0 !important;
            margin: 0 !important; border: 1px solid #cbd5e1 !important; border-radius: 3px !important;
            vertical-align: middle !important; background-color: #ffffff !important;
        }
        .qty-btn {
            width: 20px !important; height: 22px !important; min-width: 20px !important;
            min-height: 22px !important; font-size: 12px !important; font-weight: bold !important;
            line-height: 1 !important; display: inline-flex !important; align-items: center !important;
            justify-content: center !important; border: 1px solid #a0a0a0 !important; background: #ffffff !important;
            border-radius: 3px !important; color: #333333 !important; cursor: pointer !important;
            padding: 0 !important; margin: 0 !important; vertical-align: middle !important;
            user-select: none !important; transition: background 0.15s !important;
        }
        .qty-btn:hover {
            background-color: #e2e8f0 !important; border-color: #004b8d !important; color: #004b8d !important;
        }

        .group-label {
            text-align: left !important; padding-left: 10px !important;
            font-weight: 700 !important; background-color: #f8fafc !important; color: #004b8d !important;
        }
    </style>
</head>
<body oncontextmenu="return false" onselectstart="return false" ondragstart="return false">

    <!-- 🔐 시스템 접속 보안 인증 모달 (첫 화면 암호 및 신청 기능) -->
    <div id="authOverlay" class="auth-overlay">
        <div class="auth-card" id="authCard">
            <div class="auth-icon">🔒</div>
            <div class="auth-title">시스템 보안 인증</div>
            <div class="auth-sub">인가된 사용자만 접속 가능합니다.<br/>비밀번호를 입력하세요.</div>
            
            <!-- 비밀번호 입력 영역 -->
            <input type="password" id="authPasswordInput" class="auth-input" placeholder="••••" maxlength="20" autofocus />
            <button class="auth-btn" onclick="validatePassword()">확인 (접속하기)</button>
            <div id="authErrorMsg" class="auth-error">⚠️ 비밀번호가 일치하지 않습니다.</div>
            
            <!-- 비밀번호 신청 토글 링크 -->
            <div class="auth-req-link" onclick="toggleRequestForm()">비밀번호가 없으신가요? (암호 신청하기)</div>

            <!-- 비밀번호 신청 폼 (기본 숨김) -->
            <div id="reqFormArea" class="req-form-area">
                <div style="font-size: 11.5px; font-weight: 700; color: #0284c7; margin-bottom: 8px;">🔑 비밀번호 발급 신청</div>
                <input type="text" id="reqUserName" class="req-input" placeholder="신청자 성함 입력" maxlength="20" />
                <input type="text" id="reqUserPhone" class="req-input" placeholder="연락처 (예: 010-1234-5678)" maxlength="15" />
                <button class="req-submit-btn" id="reqSubmitBtn" onclick="submitPasswordRequest()">신청 문자/알림 전송</button>
            </div>
        </div>
    </div>

    <!-- 🚨 강렬한 레드 경고 모달 레이어 -->
    <div id="redAlertOverlay" class="red-alert-overlay">
        <div class="red-alert-card">
            <div class="red-alert-icon">🚨</div>
            <div class="red-alert-title">경 고 (WARNING)</div>
            <div class="red-alert-msg">여긴 내가 만든 내 세상이야, 불펌금지</div>
            <button class="red-alert-btn" onclick="closeRedAlert()">확인 (닫기)</button>
        </div>
    </div>

    <div class="responsive-wrapper">
        
        <!-- 최상단 공식 시그니처 배지 -->
        <div class="signature-badge">
            <div style="font-size:11px; color:#0284c7; font-weight:800; letter-spacing:1px; margin-bottom:2px;">KT CS TELECOM AUTOMATION ARCHITECT</div>
            <div style="font-size:13px; color:#1e293b; font-weight:700;">
                🔒 본 시스템은 <span style="color:#0284c7;">KT CS 김영훈</span>의 지적 자산입니다. (무단 복제 및 사용 금지)
            </div>
        </div>

        <!-- 통합 대분류 탭 메뉴 (4대 견적서) -->
        <div class="tab-menu-container">
            <button class="tab-btn active" id="btn-renewal" onclick="switchEstimateTab('renewal')">법인회선 재약정</button>
            <button class="tab-btn" id="btn-total" onclick="switchEstimateTab('total')">유무선 통합</button>
            <button class="tab-btn" id="btn-haiorder" onclick="switchEstimateTab('haiorder')">하이오더</button>
            <button class="tab-btn" id="btn-bizcalc" onclick="switchEstimateTab('bizcalc')" style="background-color:#e60012; color:#fff;">🏢 BIZ 대량 계산기</button>
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
                    <td><input type="text" value="박은영" readonly /></td>
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
                    <th rowspan="2">구비서류</th>
                    <td rowspan="2" class="notice-text lock-cell" style="background-color: #fafafa; font-size: 10px; line-height: 1.45; user-select: none; text-align: center !important;">
                        <span class="capture-text-node" style="font-size: 10px !important;">대표자신분증, 사업자등록증, 통장사본</span>
                    </td>
                    <th>담당자</th>
                    <td>
                        <select class="manager-select blue-readonly" onchange="updateManagerInfo(this, 'renewal')">
                            <option value="반청용 부장" data-phone="010-3484-0709">반청용 부장</option>
                            <option value="김문신 파트장" data-phone="010-5555-6031">김문신 파트장</option>
                            <option value="이경태 파트장" data-phone="010-2622-6222">이경태 파트장</option>
                            <option value="곽준섭 과장" data-phone="010-5015-9908">곽준섭 과장</option>
                            <option value="권순일 과장" data-phone="010-5564-8610">권순일 과장</option>
                            <option value="김영현 과장" data-phone="010-3214-0555">김영현 과장</option>
                            <option value="김영훈 과장" data-phone="010-8290-9971" selected>김영훈 과장</option>
                            <option value="김준엽 과장" data-phone="010-3777-8560">김준엽 과장</option>
                            <option value="김진 과장" data-phone="010-9877-5479">김진 과장</option>
                            <option value="박혜임 과장" data-phone="010-3311-0340">박혜임 과장</option>
                            <option value="배민혁 과장" data-phone="010-6670-6870">배민혁 과장</option>
                            <option value="홍성희 과장" data-phone="010-6727-9993">홍성희 과장</option>
                            <option value="custom">직접입력</option>
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
                        • 수수료 지급 : 계약/개통 완료 ➔ 익월 말 ➔ 법인 대표자(또는 명의자) 휴대폰으로 모바일 상품권 발송<br />
                        • 사은품 지급 : 계약/개통 완료 ➔ 1주 이내 ➔ 법인 대표자(또는 명의자) 휴대폰으로 모바일 상품권 발송<br />
                        • <span style="color:#d91414;">[지급 제한 조건]</span> 수수료 및 통합 사은품은 명함/서류에 기재된 확인 가능한 본인 전화번호로만 발송됩니다.
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
                        <textarea placeholder="여기에 특이사항이나 추가 협의 내용을 입력하세요."></textarea>
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
                    <td><input type="text" value="박은영" readonly /></td>
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
                        대표자신분증, 사업자등록증, 통장사본
                    </td>
                    <th>담당자</th>
                    <td>
                        <select class="manager-select blue-readonly" onchange="updateManagerInfo(this, 'total')">
                            <option value="반청용 부장" data-phone="010-3484-0709">반청용 부장</option>
                            <option value="김문신 파트장" data-phone="010-5555-6031">김문신 파트장</option>
                            <option value="이경태 파트장" data-phone="010-2622-6222">이경태 파트장</option>
                            <option value="곽준섭 과장" data-phone="010-5015-9908">곽준섭 과장</option>
                            <option value="권순일 과장" data-phone="010-5564-8610">권순일 과장</option>
                            <option value="김영현 과장" data-phone="010-3214-0555">김영현 과장</option>
                            <option value="김영훈 과장" data-phone="010-8290-9971" selected>김영훈 과장</option>
                            <option value="김준엽 과장" data-phone="010-3777-8560">김준엽 과장</option>
                            <option value="김진 과장" data-phone="010-9877-5479">김진 과장</option>
                            <option value="박혜임 과장" data-phone="010-3311-0340">박혜임 과장</option>
                            <option value="배민혁 과장" data-phone="010-6670-6870">배민혁 과장</option>
                            <option value="홍성희 과장" data-phone="010-6727-9993">홍성희 과장</option>
                            <option value="custom">직접입력</option>
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
                        • 수수료 지급 : 계약/개통 완료 ➔ 익월 말 ➔ 법인 대표자(또는 명의자) 휴대폰으로 모바일 상품권 발송<br />
                        • 사은품 지급 : 계약/개통 완료 ➔ 1주 이내 ➔ 법인 대표자(또는 명의자) 휴대폰으로 모바일 상품권 발송<br />
                        • <span style="color:#d91414;">[지급 제한 조건]</span> 수수료 및 통합 사은품은 명함/서류에 기재된 확인 가능한 본인 전화번호로만 발송됩니다.
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

        <!-- ================= [서식 3: 하이오더 견적서] ================= -->
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
                    <th>업체명</th>
                    <td><input type="text" value=" 귀하" class="client-name" id="hai-store-name" onfocus="clearGuidance(this)" onblur="restoreGuidance(this)" /></td>
                    <th>회사명</th>
                    <td><input type="text" value="(주) KT CS" readonly /></td>
                </tr>
                <tr>
                    <th>서비스명</th>
                    <td><input type="text" value="하이오더 외" readonly /></td>
                    <th>대표자명</th>
                    <td><input type="text" value="박은영" readonly /></td>
                </tr>
                <tr class="amount-row">
                    <th>월요금(VAT포함)</th>
                    <td class="benefit-highlight" id="topQuoteAmount">0원</td>
                    <th>주소</th>
                    <td><input type="text" value="대전 서구 갈마로 160(괴정동) KT 인재개발원" readonly /></td>
                </tr>
                <tr>
                    <th>고객 오퍼</th>
                    <td>
                        <div class="offer-container">
                            <input type="text" id="offerPriceInput" class="price-input" value="0" />원
                            <span class="offer-total-label" id="offerTotalText">(총합계: 0원)</span>
                        </div>
                    </td>
                    <th>업종</th>
                    <td><input type="text" value="정보통신업" readonly /></td>
                </tr>
                <tr>
                    <th>비고</th>
                    <td><input type="text" value="테이블오더 솔루션" readonly /></td>
                    <th>담당부서</th>
                    <td><input type="text" value="KT CS 대구센터" readonly /></td>
                </tr>
                <tr>
                    <th rowspan="2">구비서류</th>
                    <td rowspan="2" class="notice-text lock-cell" style="background-color: #fafafa; font-size: 10px; line-height: 1.45; user-select: none; text-align: center !important;">
                        <span class="capture-text-node" style="font-size: 10px !important;">대표자신분증, 사업자등록증, 통장사본</span>
                    </td>
                    <th>담당자</th>
                    <td>
                        <select class="manager-select blue-readonly" onchange="updateManagerInfo(this, 'haiorder')">
                            <option value="반청용 부장" data-phone="010-3484-0709">반청용 부장</option>
                            <option value="김문신 파트장" data-phone="010-5555-6031">김문신 파트장</option>
                            <option value="이경태 파트장" data-phone="010-2622-6222">이경태 파트장</option>
                            <option value="곽준섭 과장" data-phone="010-5015-9908">곽준섭 과장</option>
                            <option value="권순일 과장" data-phone="010-5564-8610">권순일 과장</option>
                            <option value="김영현 과장" data-phone="010-3214-0555">김영현 과장</option>
                            <option value="김영훈 과장" data-phone="010-8290-9971" selected>김영훈 과장</option>
                            <option value="김준엽 과장" data-phone="010-3777-8560">김준엽 과장</option>
                            <option value="김진 과장" data-phone="010-9877-5479">김진 과장</option>
                            <option value="박혜임 과장" data-phone="010-3311-0340">박혜임 과장</option>
                            <option value="배민혁 과장" data-phone="010-6670-6870">배민혁 과장</option>
                            <option value="홍성희 과장" data-phone="010-6727-9993">홍성희 과장</option>
                            <option value="custom">직접입력</option>
                        </select>
                    </td>
                </tr>
                <tr>
                    <th>연락처</th>
                    <td><input type="text" id="manager-phone-haiorder" class="blue-readonly" value="010-8290-9971" readonly /></td>
                </tr>
                <tr>
                    <th style="user-select: none;">견적유효기간</th>
                    <td class="notice-text lock-cell" style="background-color: #ffffff; font-weight: bold; color: #004b8d !important; user-select: none; text-align: center !important;">
                        <span class="capture-text-node" style="color: #004b8d !important; font-weight: bold !important;">견적서 제출일로부터 30일 이내</span>
                    </td>
                    <th>이메일주소</th>
                    <td><input type="text" id="manager-email-haiorder" class="blue-readonly" value="tsmobile1@naver.com" readonly /></td>
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

            <div class="grand-total-hai">
                <table>
                    <tr>
                        <td class="label">하이오더 합계 (①+②)</td>
                        <td class="value" id="haiorderTotal">0원</td>
                    </tr>
                </table>
            </div>

            <table class="notice-container-table" style="user-select: none;">
                <tr>
                    <th style="color: #d91414 !important; background-color: #fef2f2 !important; text-align: center !important;">필수 안내</th>
                    <td class="notice-text bg-alert" style="padding: 10px; font-weight: 700;">
                        • 수수료 지급 : 계약/개통 완료 ➔ 익월 말 ➔ 법인 대표자(또는 명의자) 휴대폰으로 모바일 상품권 발송<br />
                        • 사은품 지급 : 계약/개통 완료 ➔ 1주 이내 ➔ 법인 대표자(또는 명의자) 휴대폰으로 모바일 상품권 발송<br />
                        • <span style="color:#d91414;">[지급 제한 조건]</span> 수수료 및 통합 사은품은 명함/서류에 기재된 확인 가능한 본인 전화번호로만 발송됩니다.
                    </td>
                </tr>
            </table>

            <table class="notice-container-table" style="user-select: none;">
                <tr>
                    <th style="text-align: center !important;">유의 사항</th>
                    <td class="notice-text" style="padding: 10px; background-color: #fafafa; color: #555 !important;">
                        1. 본 하이오더 견적서의 유효기간은 견적서 제출일로부터 30일입니다.<br />
                        2. 상기 견적 단말의 A/S 보증기간은 3년이며, 세부사항은 이용약관에 준합니다.<br />
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

        <!-- ================= [서식 4: BIZ 대량 계산기 통합 영역] ================= -->
        <div class="invoice-container" id="capture-area-bizcalc" style="display: none;">
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
                <div class="title-area">KT 인터넷 & TV BIZ 표준 견적서</div>
            </div>

            <!-- 약정 선택 소분류 탭 -->
            <div class="biz-sub-tab-container">
                <button class="biz-sub-tab-btn active" onclick="switchBizSubTab('1')">요금제 (3년약정)</button>
                <button class="biz-sub-tab-btn" onclick="switchBizSubTab('2')">BIZ벌크 요금제 (4년약정)</button>
                <button class="biz-sub-tab-btn" onclick="switchBizSubTab('3')">BIZ벌크 요금제 (5년약정)</button>
            </div>

            <!-- 상단 고객 & 공급자 헤더 표 -->
            <table class="info-table">
                <tr>
                    <th>견적일자</th>
                    <td><input type="text" class="invoice-date blue-readonly" readonly /></td>
                    <th>사업자번호</th>
                    <td><input type="text" value="314-81-42683" readonly /></td>
                </tr>
                <tr>
                    <th>업체명</th>
                    <td><input type="text" id="biz-client-company" value="귀하" placeholder="고객 업체명 입력" /></td>
                    <th>회사명</th>
                    <td><input type="text" value="(주) KT CS" readonly /></td>
                </tr>
                <tr>
                    <th>사업자번호</th>
                    <td><input type="text" id="biz-client-bizno" placeholder="고객 사업자번호 입력" /></td>
                    <th>대표자명</th>
                    <td><input type="text" value="박은영" readonly /></td>
                </tr>
                <tr>
                    <th>총 제공 혜택</th>
                    <td class="benefit-highlight" id="totalBenefitVal">₩0</td>
                    <th>주소</th>
                    <td><input type="text" value="대전 서구 갈마로 160(괴정동) KT 인재개발원" readonly /></td>
                </tr>
                <tr>
                    <th>수수료</th>
                    <td><input type="number" id="feeVal" value="0" min="0" oninput="updateBenefit()" style="text-align:center;"></td>
                    <th>업종</th>
                    <td><input type="text" value="정보통신업" readonly /></td>
                </tr>
                <tr>
                    <th>통합사은품</th>
                    <td><input type="number" id="giftVal" value="0" min="0" oninput="updateBenefit()" style="text-align:center;"></td>
                    <th>담당부서</th>
                    <td><input type="text" value="KT CS 대구센터" readonly /></td>
                </tr>
                <tr>
                    <th rowspan="2">구비서류</th>
                    <td rowspan="2" class="notice-text lock-cell" style="background-color: #fafafa; font-size: 9.5px; line-height: 1.35; user-select: none; text-align: center !important;">
                        <span class="capture-text-node">대표자신분증, 사업자등록증, 통장사본</span>
                    </td>
                    <th>담당자</th>
                    <td>
                        <select class="manager-select blue-readonly" onchange="updateManagerInfo(this, 'bizcalc')">
                            <option value="반청용 부장" data-phone="010-3484-0709">반청용 부장</option>
                            <option value="김문신 파트장" data-phone="010-5555-6031">김문신 파트장</option>
                            <option value="이경태 파트장" data-phone="010-2622-6222">이경태 파트장</option>
                            <option value="곽준섭 과장" data-phone="010-5015-9908">곽준섭 과장</option>
                            <option value="권순일 과장" data-phone="010-5564-8610">권순일 과장</option>
                            <option value="김영현 과장" data-phone="010-3214-0555">김영현 과장</option>
                            <option value="김영훈 과장" data-phone="010-8290-9971" selected>김영훈 과장</option>
                            <option value="김준엽 과장" data-phone="010-3777-8560">김준엽 과장</option>
                            <option value="김진 과장" data-phone="010-9877-5479">김진 과장</option>
                            <option value="박혜임 과장" data-phone="010-3311-0340">박혜임 과장</option>
                            <option value="배민혁 과장" data-phone="010-6670-6870">배민혁 과장</option>
                            <option value="홍성희 과장" data-phone="010-6727-9993">홍성희 과장</option>
                            <option value="custom">직접입력</option>
                        </select>
                    </td>
                </tr>
                <tr>
                    <th>연락처</th>
                    <td><input type="text" id="manager-phone-bizcalc" class="blue-readonly" value="010-8290-9971" readonly /></td>
                </tr>
                <tr>
                    <th style="user-select: none;">견적유효기간</th>
                    <td class="notice-text lock-cell" style="background-color: #ffffff; font-weight: bold; color: #004b8d !important; text-align: center !important;">
                        <span class="capture-text-node" style="color: #004b8d !important; font-weight: bold !important;">견적서 제출일로부터 30일 이내</span>
                    </td>
                    <th>이메일주소</th>
                    <td><input type="text" id="manager-email-bizcalc" class="blue-readonly" value="tsmobile1@naver.com" readonly /></td>
                </tr>
            </table>

            <!-- BIZ 대량 조건 입력 카드 (html2canvas 무결성 구조 Table 적용) -->
            <div class="card-title">■ BIZ 대량 수량 및 가입 조건 선택</div>
            <table class="biz-input-table">
                <tr>
                    <th style="width: 25%;">1. TV 제공수 (직접 입력):</th>
                    <td style="width: 25%;"><input type="number" id="tvCount" value="19" min="0" max="500" oninput="calculateBiz()"></td>
                    <th style="width: 25%;">2. 인터넷 상품 선택:</th>
                    <td style="width: 25%;">
                        <select id="internetProd" onchange="calculateBiz()">
                            <option value="인터넷 베이직 (500M)" selected>인터넷 베이직 (500M)</option>
                            <option value="인터넷 슬림 (100M)">인터넷 슬림 (100M)</option>
                            <option value="인터넷 에센스 (1G)">인터넷 에센스 (1G)</option>
                        </select>
                    </td>
                </tr>
                <tr>
                    <th>3. TV 상품 선택:</th>
                    <td><select id="tvProd" onchange="calculateBiz()"></select></td>
                    <th>4. TV STB 선택:</th>
                    <td>
                        <select id="stbProd" onchange="calculateBiz()">
                            <option value="올인원 STB" selected>올인원 STB</option>
                            <option value="STB A">STB A</option>
                        </select>
                    </td>
                </tr>
                <tr>
                    <th colspan="2">5. 인터넷 패밀리 가능 회선수 (명의당 최대 2회선):</th>
                    <td colspan="2"><input type="number" id="familyLines" value="0" min="0" max="2" oninput="calculateBiz()"></td>
                </tr>
            </table>

            <!-- 산출 요약 배너 -->
            <div class="summary-banner">
                <div>
                    <div style="font-size: 11px; font-weight: bold; color: #1e293b;" id="summaryTitle">■ 요금제 (3년약정) 총 월정액 (VAT포함)</div>
                    <div class="summary-sub" id="summaryNotice">KT CS 대구센터 B2B 표준 출고가 산출 기준</div>
                </div>
                <div style="text-align: right;">
                    <div class="summary-val" id="totalMonthly">301,950 원 / 월</div>
                    <div class="summary-sub" id="perRoomPrice">룸당 단가: 약 15,892원/월</div>
                </div>
            </div>

            <!-- 세부 가입 품목 내역표 -->
            <div class="card-title">■ 가입 품목별 상세 산출 내역표</div>
            <table class="product-table">
                <thead>
                    <tr>
                        <th style="width: 10%;">항목</th>
                        <th style="width: 24%;">상품명</th>
                        <th style="width: 8%;">회선수</th>
                        <th style="width: 12%;">이용료</th>
                        <th style="width: 10%;">STB</th>
                        <th style="width: 12%;">월 이용료</th>
                        <th style="width: 12%;">설치비 단가</th>
                        <th style="width: 12%;">설치비 합계</th>
                    </tr>
                </thead>
                <tbody id="excelItemizedBody"></tbody>
                <tfoot>
                    <tr class="total-row">
                        <td colspan="2">합 계</td>
                        <td id="totalLines">19 회선</td>
                        <td>-</td>
                        <td>-</td>
                        <td class="text-right text-red" id="sumMonthly">301,950 원</td>
                        <td>-</td>
                        <td class="text-right font-bold" id="sumInstall">309,400 원</td>
                    </tr>
                </tfoot>
            </table>

            <table class="notice-container-table" style="user-select: none;">
                <tr>
                    <th style="color: #d91414 !important; background-color: #fef2f2 !important; text-align: center !important;">필수 안내</th>
                    <td class="notice-text bg-alert" style="padding: 10px; font-weight: 700;">
                        • 수수료 지급 : 계약/개통 완료 ➔ 익월 말 ➔ 법인 대표자(또는 명의자) 휴대폰으로 모바일 상품권 발송<br />
                        • 사은품 지급 : 계약/개통 완료 ➔ 1주 이내 ➔ 법인 대표자(또는 명의자) 휴대폰으로 모바일 상품권 발송<br />
                        • <span style="color:#d91414;">[지급 제한 조건]</span> 수수료 및 통합 사은품은 명함/서류에 기재된 확인 가능한 본인 전화번호로만 발송됩니다.
                    </td>
                </tr>
            </table>

            <table class="notice-container-table" style="user-select: none;">
                <tr>
                    <th style="text-align: center !important;">유의 사항</th>
                    <td class="notice-text" style="padding: 10px; background-color: #fafafa; color: #555 !important;">
                        1. 상기 BIZ 대량 인터넷/TV 견적은 KT CS 대구센터 정식 B2B 표준 출고 단가가 반영된 산출 내역입니다.<br />
                        2. 인터넷 패밀리 회선 수는 동일 명의당 최대 2회선까지 결합 가능하며, 약정 세부조건은 이용약관에 준합니다.<br />
                        3. 현장 구내 설비 환경 및 장애 발생 시 KT 고객센터(국번없이 100번) 또는 담당부서로 상담 바랍니다.
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

        <!-- 하단 전용 다운로드 버튼 영역 -->
        <div class="btn-area">
            <button class="download-btn" onclick="generateActiveInvoiceImage('jpg')">견적서 JPG 다운로드</button>
            <button class="download-btn pdf-btn" onclick="generateActiveInvoiceImage('pdf')">견적서 PDF 다운로드</button>
        </div>
    </div>

    <script>
        // 🔑 [접속 비밀번호 설정]
        const SYSTEM_PASSWORD = "0729";

        let activeTab = 'renewal';
        let currentBizTab = '1';
        const GAS_URL = "https://script.google.com/macros/s/AKfycbzriskJha8aL9cnErvdImPwMBxLi690oyLCUgrTBHJHcvHiWlNvGwU3ferdftgx-sml/exec";

        // BIZ 대량 계산기 약정별 1:1 매칭 데이터
        const SHEET_STRICT_DATA = {
            "1": {
                name: "요금제 (3년약정)",
                tvOptions: ["OTV Biz 벌크 베이직", "OTV Biz 슬림", "OTV Biz 베이직", "OTV Biz 라이트", "OTV Biz 에센스", "OTV Biz 벌크 슬림", "OTV Biz 벌크 라이트", "OTV Biz 벌크 에센스"],
                defaultTv: "OTV Biz 벌크 베이직",
                internet: {
                    "인터넷 슬림 (100M)": { fee: 22000, family: 16500, install: 32000 },
                    "인터넷 베이직 (500M)": { fee: 27500, family: 22000, install: 32000 },
                    "인터넷 에센스 (1G)": { fee: 33000, family: 27500, install: 32000 }
                },
                tv: {
                    "OTV Biz 벌크 베이직": { base: 8250, addNormal: 8250, install: 11000 },
                    "OTV Biz 슬림": { base: 11000, addNormal: 6600, install: 11000 },
                    "OTV Biz 베이직": { base: 12100, addNormal: 7370, install: 11000 },
                    "OTV Biz 라이트": { base: 13200, addNormal: 7920, install: 11000 },
                    "OTV Biz 에센스": { base: 16500, addNormal: 10120, install: 11000 },
                    "OTV Biz 벌크 슬림": { base: 7700, addNormal: 7700, install: 11000 },
                    "OTV Biz 벌크 라이트": { base: 8800, addNormal: 8800, install: 11000 },
                    "OTV Biz 벌크 에센스": { base: 11000, addNormal: 11000, install: 11000 }
                },
                stb: { "올인원 STB": 3300, "STB A": 4400 },
                tvInstallBase: 4400
            },
            "2": {
                name: "BIZ벌크 요금제 (4년약정)",
                tvOptions: ["OTV Biz 벌크 슬림", "OTV Biz 벌크 베이직", "OTV Biz 벌크 라이트", "OTV Biz 벌크 에센스"],
                defaultTv: "OTV Biz 벌크 슬림",
                internet: {
                    "인터넷 슬림 (100M)": { fee: 22000, family: 16500, install: 23100 },
                    "인터넷 베이직 (500M)": { fee: 27500, family: 22000, install: 23100 },
                    "인터넷 에센스 (1G)": { fee: 33000, family: 27500, install: 23100 }
                },
                tv: {
                    "OTV Biz 벌크 슬림": { base: 6875, addNormal: 6875, install: 11000 },
                    "OTV Biz 벌크 베이직": { base: 7425, addNormal: 7425, install: 11000 },
                    "OTV Biz 벌크 라이트": { base: 7975, addNormal: 7975, install: 11000 },
                    "OTV Biz 벌크 에센스": { base: 10175, addNormal: 10175, install: 11000 }
                },
                stb: { "올인원 STB": 3300, "STB A": 4400 },
                tvInstallBase: 4400
            },
            "3": {
                name: "BIZ벌크 요금제 (5년약정)",
                tvOptions: ["OTV Biz 벌크 슬림", "OTV Biz 벌크 베이직", "OTV Biz 벌크 라이트", "OTV Biz 벌크 에센스"],
                defaultTv: "OTV Biz 벌크 라이트",
                internet: {
                    "인터넷 슬림 (100M)": { fee: 22000, family: 16500, install: 23100 },
                    "인터넷 베이직 (500M)": { fee: 27500, family: 22000, install: 23100 },
                    "인터넷 에센스 (1G)": { fee: 33000, family: 27500, install: 23100 }
                },
                tv: {
                    "OTV Biz 벌크 슬림": { base: 6050, addNormal: 6050, install: 11000 },
                    "OTV Biz 벌크 베이직": { base: 6600, addNormal: 6600, install: 11000 },
                    "OTV Biz 벌크 라이트": { base: 7150, addNormal: 7150, install: 11000 },
                    "OTV Biz 벌크 에센스": { base: 9350, addNormal: 9350, install: 11000 }
                },
                stb: { "올인원 STB": 3300, "STB A": 4400 },
                tvInstallBase: 4400
            }
        };

        window.onload = function() {
            setTodayDateAll();
            calculateBenefitsRenewal();
            calculateTableTotalsRenewal();
            calculateBenefitsTotal();
            calculateTableTotalsTotal();
            attachCommaFormattingHai();
            recalcAllHaiorder();
            initKeyLock();
            switchBizSubTab('1');

            // 비밀번호 입력창 엔터키 이벤트 바인딩
            const authInput = document.getElementById('authPasswordInput');
            if (authInput) {
                authInput.value = '';
                setTimeout(() => authInput.focus(), 100);
                authInput.addEventListener('keydown', function(e) {
                    e.stopPropagation();
                    if (e.key === 'Enter' || e.keyCode === 13) {
                        e.preventDefault();
                        validatePassword();
                    }
                });
            }

            // 신청자 연락처 입력창 엔터키 바인딩
            const phoneInput = document.getElementById('reqUserPhone');
            if (phoneInput) {
                phoneInput.addEventListener('keydown', function(e) {
                    e.stopPropagation();
                    if (e.key === 'Enter' || e.keyCode === 13) {
                        e.preventDefault();
                        submitPasswordRequest();
                    }
                });
            }
        }

        // 🔐 비밀번호 검증 함수
        function validatePassword() {
            const input = document.getElementById('authPasswordInput');
            const errMsg = document.getElementById('authErrorMsg');
            const card = document.getElementById('authCard');

            const enteredVal = String(input.value).trim();

            if (enteredVal === SYSTEM_PASSWORD) {
                document.getElementById('authOverlay').style.display = 'none';
                errMsg.style.display = 'none';
            } else {
                errMsg.style.display = 'block';
                card.classList.add('shake');
                input.value = '';
                input.focus();
                setTimeout(() => { card.classList.remove('shake'); }, 400);
            }
        }

        // 🔑 비밀번호 신청 영역 펼침/닫힘 토글
        function toggleRequestForm() {
            const area = document.getElementById('reqFormArea');
            if (area.style.display === 'block') {
                area.style.display = 'none';
            } else {
                area.style.display = 'block';
                document.getElementById('reqUserName').focus();
            }
        }

        // 📩 비밀번호 신청 데이터 GAS 전송
        function submitPasswordRequest() {
            const nameInput = document.getElementById('reqUserName');
            const phoneInput = document.getElementById('reqUserPhone');
            const btn = document.getElementById('reqSubmitBtn');

            const name = nameInput.value.trim();
            const phone = phoneInput.value.trim();

            if (!name) {
                alert('신청자 성함을 입력해 주세요.');
                nameInput.focus();
                return;
            }
            if (!phone) {
                alert('연락처를 입력해 주세요.');
                phoneInput.focus();
                return;
            }

            btn.disabled = true;
            btn.innerText = '전송 중...';

            const payload = {
                type: 'PASSWORD_REQUEST',
                userName: name,
                userPhone: phone,
                requestDate: new Date().toLocaleString('ko-KR', { timeZone: 'Asia/Seoul' }),
                consultDay: `[암호신청] ${name} (${phone})`
            };

            fetch(GAS_URL, {
                method: 'POST',
                mode: 'no-cors',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(payload)
            }).then(() => {
                alert(`[신청 완료]\n${name}님의 암호 발급 신청이 담당자에게 전달되었습니다.\n확인 후 안내 문자를 발송해 드립니다.`);
                nameInput.value = '';
                phoneInput.value = '';
                document.getElementById('reqFormArea').style.display = 'none';
                btn.disabled = false;
                btn.innerText = '신청 문자/알림 전송';
            }).catch(err => {
                alert('신청 전송 중 오류가 발생했습니다. 다시 시도해 주세요.');
                btn.disabled = false;
                btn.innerText = '신청 문자/알림 전송';
            });
        }

        function showRedAlert() { document.getElementById('redAlertOverlay').style.display = 'flex'; }
        function closeRedAlert() { document.getElementById('redAlertOverlay').style.display = 'none'; }

        function initKeyLock() {
            document.addEventListener('contextmenu', function(e) { e.preventDefault(); showRedAlert(); });
            document.addEventListener('dragstart', e => e.preventDefault());
            document.addEventListener('selectstart', function(e) {
                if (e.target && (e.target.id === 'authPasswordInput' || e.target.id === 'reqUserName' || e.target.id === 'reqUserPhone')) return true;
                e.preventDefault();
            });

            document.addEventListener('keydown', function(e) {
                if (e.target && (e.target.id === 'authPasswordInput' || e.target.id === 'reqUserName' || e.target.id === 'reqUserPhone')) return;

                if (e.keyCode === 123 || 
                   (e.ctrlKey && e.shiftKey && (e.keyCode === 73 || e.keyCode === 74 || e.keyCode === 67)) || 
                   (e.ctrlKey && (e.keyCode === 85 || e.keyCode === 83 || e.keyCode === 67 || e.keyCode === 65))) {
                    e.preventDefault(); e.stopPropagation(); showRedAlert(); return false;
                }
            });
        }

        function setTodayDateAll() {
            const today = new Date();
            const yyyy = today.getFullYear();
            const mm = String(today.getMonth() + 1).padStart(2, '0');
            const dd = String(today.getDate()).padStart(2, '0');
            document.querySelectorAll('.invoice-date').forEach(el => { el.value = `${yyyy}-${mm}-${dd}`; });
            const qDate = document.getElementById('quoteDate');
            if(qDate) qDate.value = `${yyyy}-${mm}-${dd}`;
        }

        function switchEstimateTab(tabName) {
            activeTab = tabName;
            document.getElementById('capture-area-renewal').style.display = (tabName === 'renewal') ? 'block' : 'none';
            document.getElementById('capture-area-total').style.display = (tabName === 'total') ? 'block' : 'none';
            document.getElementById('capture-area-haiorder').style.display = (tabName === 'haiorder') ? 'block' : 'none';
            document.getElementById('capture-area-bizcalc').style.display = (tabName === 'bizcalc') ? 'block' : 'none';
            
            document.getElementById('btn-renewal').classList.toggle('active', tabName === 'renewal');
            document.getElementById('btn-total').classList.toggle('active', tabName === 'total');
            document.getElementById('btn-haiorder').classList.toggle('active', tabName === 'haiorder');
            document.getElementById('btn-bizcalc').classList.toggle('active', tabName === 'bizcalc');

            if(tabName === 'bizcalc') calculateBiz();
        }

        function updateBenefit() {
            const fee = parseInt(document.getElementById('feeVal').value) || 0;
            const gift = parseInt(document.getElementById('giftVal').value) || 0;
            document.getElementById('totalBenefitVal').textContent = `₩${(fee + gift).toLocaleString()}`;
        }

        function switchBizSubTab(sheetKey) {
            currentBizTab = sheetKey;
            document.querySelectorAll('.biz-sub-tab-btn').forEach((btn, idx) => {
                btn.classList.toggle('active', (sheetKey === '1' && idx === 0) || (sheetKey === '2' && idx === 1) || (sheetKey === '3' && idx === 2));
            });
            
            const tvSelect = document.getElementById('tvProd');
            tvSelect.innerHTML = "";
            SHEET_STRICT_DATA[sheetKey].tvOptions.forEach(opt => {
                const option = document.createElement('option');
                option.value = opt; option.textContent = opt;
                tvSelect.appendChild(option);
            });
            tvSelect.value = SHEET_STRICT_DATA[sheetKey].defaultTv;
            calculateBiz();
        }

        function calculateBiz() {
            const data = SHEET_STRICT_DATA[currentBizTab];
            const tvCount = parseInt(document.getElementById('tvCount').value) || 0;
            const inetProdKey = document.getElementById('internetProd').value;
            
            let inetKeyMatched = "인터넷 베이직 (500M)";
            if (inetProdKey.includes("슬림")) inetKeyMatched = "인터넷 슬림 (100M)";
            else if (inetProdKey.includes("에센스")) inetKeyMatched = "인터넷 에센스 (1G)";

            const tvProd = document.getElementById('tvProd').value;
            const stbProd = document.getElementById('stbProd').value;
            let famInput = parseInt(document.getElementById('familyLines').value) || 0;

            const inetCountCalc = Math.ceil(tvCount / 8);
            let famLines = (inetCountCalc > famInput) ? (famInput > 2 ? 2 : famInput) : 0;

            const mainInetLines = (inetCountCalc - 1) > famLines ? (inetCountCalc - famLines) : (inetCountCalc - famLines);
            const mainInetLinesFinal = mainInetLines < 1 ? 1 : mainInetLines;

            const inetInfo = data.internet[inetKeyMatched];
            const tvInfo = data.tv[tvProd] || { base: 0, addNormal: 0, install: 11000 };
            const stbFee = data.stb[stbProd] || 3300;

            const tvAddPrice = tvInfo.addNormal;
            const baseTvLines = Math.ceil(tvCount / 8);
            const addTvLinesTotal = tvCount - baseTvLines;

            let addGroups = [];
            let remAdd = addTvLinesTotal;
            for (let i = 1; i <= 7; i++) {
                let linesInGroup = Math.floor(tvCount / 8) + ((tvCount % 8) > i ? 1 : 0);
                if (i === 7) linesInGroup = Math.floor(tvCount / 8);
                if (linesInGroup > 0 && remAdd > 0) {
                    let actualCount = Math.min(linesInGroup, remAdd);
                    addGroups.push({ groupIndex: i, count: actualCount });
                    remAdd -= actualCount;
                }
            }
            if (remAdd > 0) addGroups.push({ groupIndex: 8, count: remAdd });

            const mainInetMonthly = inetInfo.fee * mainInetLinesFinal;
            const mainInetInstallTotal = inetInfo.install + (mainInetLinesFinal - 1) * inetInfo.install;
            const famInetMonthly = inetInfo.family * famLines;

            const baseTvMonthly = (tvInfo.base + stbFee) * baseTvLines;
            const baseTvInstallTotal = tvInfo.install * baseTvLines;

            let addTvMonthlyTotal = 0;
            let addTvInstallTotalSum = 0;

            let tbodyHTML = "";

            // 1. 인터넷 메인
            tbodyHTML += `
                <tr>
                    <td class="font-bold">인터넷</td>
                    <td class="text-left">${inetKeyMatched}</td>
                    <td>${mainInetLinesFinal}</td>
                    <td class="text-right">${inetInfo.fee.toLocaleString()}원</td>
                    <td>-</td>
                    <td class="text-right font-bold">${mainInetMonthly.toLocaleString()}원</td>
                    <td class="text-right">${inetInfo.install.toLocaleString()}원</td>
                    <td class="text-right">${mainInetInstallTotal.toLocaleString()}원</td>
                </tr>`;

            // 2. 패밀리
            if (famLines > 0) {
                tbodyHTML += `
                    <tr>
                        <td class="font-bold">인터넷</td>
                        <td class="text-left">패밀리</td>
                        <td>${famLines}</td>
                        <td class="text-right">${inetInfo.family.toLocaleString()}원</td>
                        <td>-</td>
                        <td class="text-right font-bold">${famInetMonthly.toLocaleString()}원</td>
                        <td class="text-right">0원</td>
                        <td class="text-right">0원</td>
                    </tr>`;
            }

            // 3. TV 기본 설치비
            tbodyHTML += `
                <tr>
                    <td class="font-bold">TV 기본</td>
                    <td class="text-left">기본설치비 (동시오더/복수)</td>
                    <td>-</td>
                    <td class="text-right">-</td>
                    <td>-</td>
                    <td class="text-right">-</td>
                    <td class="text-right">${data.tvInstallBase.toLocaleString()}원</td>
                    <td class="text-right">${data.tvInstallBase.toLocaleString()}원</td>
                </tr>`;

            // 4. TV 주회선
            tbodyHTML += `
                <tr>
                    <td class="font-bold">TV 기본</td>
                    <td class="text-left">${tvProd} (주회선)</td>
                    <td>${baseTvLines}</td>
                    <td class="text-right">${tvInfo.base.toLocaleString()}원</td>
                    <td class="text-right">${stbFee.toLocaleString()}원</td>
                    <td class="text-right font-bold">${baseTvMonthly.toLocaleString()}원</td>
                    <td class="text-right">${tvInfo.install.toLocaleString()}원</td>
                    <td class="text-right">${baseTvInstallTotal.toLocaleString()}원</td>
                </tr>`;

            // 5. TV 추가회선
            if (addGroups.length > 0) {
                addGroups.forEach((g) => {
                    let mVal = (tvAddPrice + stbFee) * g.count;
                    let iVal = tvInfo.install * g.count;
                    addTvMonthlyTotal += mVal;
                    addTvInstallTotalSum += iVal;

                    tbodyHTML += `
                        <tr>
                            <td>TV 추가</td>
                            <td class="text-left">${tvProd} (추가 ${g.groupIndex})</td>
                            <td>${g.count}</td>
                            <td class="text-right">${Math.round(tvAddPrice).toLocaleString()}원</td>
                            <td class="text-right">${stbFee.toLocaleString()}원</td>
                            <td class="text-right">${Math.round(mVal).toLocaleString()}원</td>
                            <td class="text-right">${iVal.toLocaleString()}원</td>
                            <td class="text-right">${iVal.toLocaleString()}원</td>
                        </tr>`;
                });
            }

            const totalMonthly = mainInetMonthly + famInetMonthly + baseTvMonthly + addTvMonthlyTotal;
            const totalInstall = mainInetInstallTotal + data.tvInstallBase + baseTvInstallTotal + addTvInstallTotalSum;

            document.getElementById('excelItemizedBody').innerHTML = tbodyHTML;
            document.getElementById('summaryTitle').textContent = `■ ${data.name} 총 월정액 (VAT포함)`;
            document.getElementById('totalMonthly').textContent = `${Math.round(totalMonthly).toLocaleString()} 원 / 월`;
            document.getElementById('perRoomPrice').textContent = `룸당 평균가: 약 ${Math.round(totalMonthly / (tvCount || 1)).toLocaleString()}원/월`;
            document.getElementById('totalLines').textContent = `${tvCount} 회선`;
            document.getElementById('sumMonthly').textContent = `${Math.round(totalMonthly).toLocaleString()} 원`;
            document.getElementById('sumInstall').textContent = `${totalInstall.toLocaleString()} 원`;
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

            if(!phoneInput) return;

            if (selectedOption.value === 'custom') {
                phoneInput.value = '';
                if(emailInput) emailInput.value = '';
                phoneInput.removeAttribute('readonly');
                if(emailInput) emailInput.removeAttribute('readonly');
                phoneInput.placeholder = '연락처 직접 입력';
                if(emailInput) emailInput.placeholder = '이메일 직접 입력';
                phoneInput.focus();
            } else {
                phoneInput.value = selectedOption.getAttribute('data-phone') || '';
                if(emailInput) emailInput.value = selectedOption.getAttribute('data-email') || 'tsmobile1@naver.com';
                phoneInput.setAttribute('readonly', true);
                if(emailInput) emailInput.setAttribute('readonly', true);
            }
        }

        function checkDateValue(el) {
            if (el.value) { el.classList.add('has-value'); } else { el.classList.remove('has-value'); }
        }

        function sendQuoteDataGas() {
            const userPhone = "01082909971";

            let payload = {
                quoteType: activeTab,
                userPhone: userPhone,
                consultDay: '견적서 즉시발행'
            };

            fetch(GAS_URL, {
                method: 'POST', mode: 'no-cors',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(payload)
            }).then(() => alert('구글 시트 DB 적재 및 알림 전송 완료')).catch(err => alert('전송 오류: ' + err));
        }

        function recalcCurrentActiveTab() {
            if(activeTab === 'renewal') { calculateBenefitsRenewal(); calculateTableTotalsRenewal(); }
            else if(activeTab === 'total') { calculateBenefitsTotal(); calculateTableTotalsTotal(); }
            else if(activeTab === 'haiorder') { recalcAllHaiorder(); }
            else if(activeTab === 'bizcalc') { calculateBiz(); }
            alert('모든 금액 합계가 새로고침되었습니다.');
        }

        function resetActiveTabForm() {
            if(!confirm('현재 견적서 내용을 초기화하시겠습니까?')) return;
            const targetContainer = document.getElementById(`capture-area-${activeTab}`);
            
            if(activeTab === 'bizcalc') {
                document.getElementById('tvCount').value = 0;
                document.getElementById('familyLines').value = 0;
                document.getElementById('feeVal').value = 0;
                document.getElementById('giftVal').value = 0;
                document.getElementById('biz-client-company').value = "귀하";
                document.getElementById('biz-client-bizno').value = "";
                updateBenefit();
                calculateBiz();
            } else if(activeTab === 'haiorder') {
                targetContainer.querySelectorAll('.device-qty, .reader-qty').forEach(inp => { inp.value = '0'; });
                recalcAllHaiorder();
            } else {
                targetContainer.querySelectorAll('input[type="text"]:not([readonly]), textarea').forEach(inp => {
                    if(!inp.classList.contains('invoice-date') && !inp.classList.contains('blue-readonly')) { inp.value = ''; }
                });
                recalcCurrentActiveTab();
            }
        }

        function saveCurrentEstimateData() {
            const targetContainer = document.getElementById(`capture-area-${activeTab}`);
            const formData = {};
            targetContainer.querySelectorAll('input, select, textarea').forEach((el, index) => {
                formData[el.id ? `id_${el.id}` : `idx_${index}`] = el.value;
            });
            localStorage.setItem(`saved_estimate_${activeTab}`, JSON.stringify(formData));
            alert('입력 내용이 브라우저에 저장되었습니다.');
        }

        function loadSavedEstimateData() {
            const savedData = localStorage.getItem(`saved_estimate_${activeTab}`);
            if(!savedData) { alert('저장된 데이터가 없습니다.'); return; }
            if(!confirm('불러오시겠습니까?')) return;

            const formData = JSON.parse(savedData);
            const targetContainer = document.getElementById(`capture-area-${activeTab}`);
            targetContainer.querySelectorAll('input, select, textarea').forEach((el, index) => {
                const kId = `id_${el.id}`, kIdx = `idx_${index}`;
                if(el.id && formData[kId] !== undefined) el.value = formData[kId];
                else if(formData[kIdx] !== undefined) el.value = formData[kIdx];
            });
            recalcCurrentActiveTab();
        }

        function runCalculationsRenewal(el) { formatNumberInput(el); calculateRowDifferenceRenewal(el.closest('tr')); calculateTableTotalsRenewal(); }
        function calculateRowDifferenceRenewal(row) {
            const c = row.querySelector('.calc-charge-renewal'), r = row.querySelector('.calc-renew-renewal'), d = row.querySelector('.calc-diff-renewal');
            if(!c || !r || !d) return;
            const charge = parseInt(c.value.replace(/,/g, '')) || 0, renew = parseInt(r.value.replace(/,/g, '')) || 0;
            if(c.value === "" && r.value === "") { d.value = ""; return; }
            const diff = renew - charge;
            d.value = (diff >= 0 ? "+" : "") + diff.toLocaleString();
        }
        function runBenefitCalculationsRenewal(el) { formatNumberInput(el, "0"); calculateBenefitsRenewal(); }
        function calculateBenefitsRenewal() {
            const f = parseInt(document.getElementById('fee-renewal').value.replace(/,/g, '')) || 0;
            const g = parseInt(document.getElementById('gift-renewal').value.replace(/,/g, '')) || 0;
            document.getElementById('total-benefits-renewal').innerText = '₩' + (f + g).toLocaleString();
        }
        function calculateTableTotalsRenewal() {
            let tc = 0, tr = 0;
            document.querySelectorAll('.calc-charge-renewal').forEach(i => { tc += parseInt(i.value.replace(/,/g, '')) || 0; });
            document.querySelectorAll('.calc-renew-renewal').forEach(i => { tr += parseInt(i.value.replace(/,/g, '')) || 0; });
            document.getElementById('total-charge-renewal').innerText = tc.toLocaleString();
            document.getElementById('total-renew-renewal').innerText = tr.toLocaleString();
            const td = tr - tc;
            document.getElementById('total-diff-renewal').innerText = (tc === 0 && tr === 0) ? "0" : ((td >= 0 ? "+" : "") + td.toLocaleString());
        }

        function runCalculationsTotal(el) { formatNumberInput(el); calculateRowTotal(el.closest('tr')); calculateTableTotalsTotal(); }
        function calculateRowTotal(row) {
            const q = row.querySelector('.calc-qty-total'), b = row.querySelector('.calc-base-total'), d = row.querySelector('.calc-discount-total'), f = row.querySelector('.calc-final-total');
            if(!q || !b || !f) return;
            const qty = parseInt(q.value.replace(/,/g, '')) || 1, base = parseInt(b.value.replace(/,/g, '')) || 0, disc = parseInt(d.value.replace(/,/g, '')) || 0;
            if(b.value === "" && d.value === "") { f.value = ""; return; }
            f.value = (Math.max(0, base - disc) * qty).toLocaleString();
        }
        function runBenefitCalculationsTotal(el) { formatNumberInput(el, "0"); calculateBenefitsTotal(); }
        function calculateBenefitsTotal() {
            const f = parseInt(document.getElementById('fee-total').value.replace(/,/g, '')) || 0;
            const g = parseInt(document.getElementById('gift-total').value.replace(/,/g, '')) || 0;
            document.getElementById('total-benefits-total').innerText = '₩' + (f + g).toLocaleString();
        }
        function calculateTableTotalsTotal() {
            let tq = 0, tb = 0, td = 0, tf = 0;
            document.querySelectorAll('#product-table-total tbody tr:not(.total-row)').forEach(row => {
                const q = parseInt(row.querySelector('.calc-qty-total').value.replace(/,/g, '')) || 0;
                const b = parseInt(row.querySelector('.calc-base-total').value.replace(/,/g, '')) || 0;
                const d = parseInt(row.querySelector('.calc-discount-total').value.replace(/,/g, '')) || 0;
                const f = parseInt(row.querySelector('.calc-final-total').value.replace(/,/g, '')) || 0;
                if(b > 0 || q > 0) { tq += (q || 1); tb += (b * (q || 1)); td += (d * (q || 1)); tf += f; }
            });
            document.getElementById('total-qty-total').innerText = tq.toLocaleString();
            document.getElementById('total-base-total').innerText = tb.toLocaleString();
            document.getElementById('total-discount-total').innerText = td.toLocaleString();
            document.getElementById('total-final-total').innerText = tf.toLocaleString();
        }

        function formatWithComma(n){ return Number(String(n).replace(/[^\d]/g,'')).toLocaleString('ko-KR'); }
        function won(n){ return Math.round(n).toLocaleString('ko-KR') + '원'; }
        function parseNum(str){ return parseFloat(String(str).replace(/,/g,'')) || 0; }

        function changeQty(btn, amount) {
            const input = btn.parentNode.querySelector('input');
            let currentQty = parseNum(input.value) + amount;
            if(currentQty < 0) currentQty = 0;
            input.value = formatWithComma(currentQty);
            recalcAllHaiorder();
        }

        function attachCommaFormattingHai(){
            document.querySelectorAll('#capture-area-haiorder .qty-input, #capture-area-haiorder .price-input, #capture-area-haiorder .qty4').forEach(inp=>{
                inp.addEventListener('input', (e)=>{
                    const cursorFromEnd = inp.value.length - inp.selectionStart;
                    let raw = inp.value.replace(/[^\d]/g,'');
                    if(raw === '') raw = '0';
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
            let deviceQty = 0, firstRowQty = 0;
            table.querySelectorAll('.device-qty').forEach((inp, idx)=>{
                const val = parseNum(inp.value);
                deviceQty += val;
                if(idx === 0) firstRowQty = val; 
            });
            table.querySelectorAll('.service-qty').forEach(inp=> { inp.value = formatWithComma(deviceQty); });

            let sumSupply=0, sumTax=0, sumTotal=0;
            table.querySelectorAll('tr.item-row').forEach(row=>{
                const qtyInput = row.querySelector('.qty-input');
                const qty = qtyInput ? parseNum(qtyInput.value) : 0;
                const price = parseNum(row.querySelector('.price').value);
                const supply = qty*price, tax = Math.round(supply*0.1), total = supply+tax;
                
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
            if(installPriceInput) installPriceInput.value = formatWithComma(250000 + (10000 * totalDeviceQty));

            document.querySelectorAll('.auto-device-qty').forEach(inp=> { inp.value = formatWithComma(totalDeviceQty); });
            
            let sumSupply=0, sumTax=0, sumTotal=0;
            table.querySelectorAll('tr.item-row4').forEach(row=>{
                const qty = parseNum(row.querySelector('.qty4').value);
                const price = parseNum(row.querySelector('.price4').value);
                let supply = row.classList.contains('install-fee-row') ? price : qty * price;
                const tax = Math.round(supply*0.1), total = supply+tax;
                
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
            offerTotalText.textContent = `(총합계: ${(parseNum(offerPriceInput.value) * menuTabletQty).toLocaleString('ko-KR')}원)`;
        }

        function recalcAllHaiorder(){
            const r1 = recalcGroupHai('[data-group="g1"]'), r2 = recalcGroupHai('[data-group="g2"]'), r3 = recalcGroupHai('[data-group="g3"]');
            calcOfferTotalHai(r1.firstRowQty);
            recalcGroup4Hai(r1.deviceQty + r2.deviceQty + r3.deviceQty);
            const haiOrderTotal = r1.total + r2.total + r3.total;
            const hTotalEl = document.getElementById('haiorderTotal'), tQuoteEl = document.getElementById('topQuoteAmount');
            if(hTotalEl) hTotalEl.textContent = won(haiOrderTotal);
            if(tQuoteEl) tQuoteEl.textContent = won(haiOrderTotal);
        }

        function formatNumberInput(el, defaultVal = "") {
            let cursorPosition = el.selectionStart, originalLength = el.value.length;
            let value = el.value.replace(/[^0-9]/g, '');
            el.value = (value !== "") ? parseInt(value).toLocaleString() : defaultVal;
            let newLength = el.value.length;
            el.setSelectionRange(cursorPosition + (newLength - originalLength), cursorPosition + (newLength - originalLength));
        }

        /* 📸 [최종 보정 Engine] Table 구조 완전 보존 렌더링 */
        function generateActiveInvoiceImage(format) {
            const originArea = document.getElementById(`capture-area-${activeTab}`);
            if (document.activeElement) document.activeElement.blur();

            const cloneArea = originArea.cloneNode(true);
            cloneArea.querySelectorAll('.no-print-target').forEach(btn => btn.style.display = 'none');

            cloneArea.style.position = 'absolute';
            cloneArea.style.top = '-9999px';
            cloneArea.style.left = '-9999px';
            cloneArea.style.width = '794px';
            cloneArea.style.backgroundColor = '#ffffff';
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
                } else if (originInput.type === 'date' || originInput.tagName === 'TEXTAREA') {
                    text = originInput.value || ' ';
                } else {
                    text = originInput.value || originInput.placeholder || ' ';
                }

                const textNode = document.createElement('span');
                textNode.className = 'capture-text-node';
                
                if (originInput.classList.contains('blue-readonly')) {
                    textNode.style.color = '#004b8d'; 
                    textNode.style.fontWeight = 'bold';
                }
                if (originInput.id === 'tvCount' || originInput.id === 'familyLines' || originInput.id === 'feeVal' || originInput.id === 'giftVal') {
                    textNode.style.fontWeight = '800';
                    textNode.style.color = '#1e293b';
                }

                if (originInput.tagName === 'TEXTAREA') {
                    textNode.style.whiteSpace = 'pre-wrap'; 
                    textNode.style.textAlign = 'left'; 
                    textNode.style.padding = '6px'; 
                    textNode.style.display = 'block';
                    textNode.style.lineHeight = '1.4';
                    textNode.style.height = 'auto';
                }

                textNode.innerText = text;
                if (cloneInput.parentNode) cloneInput.parentNode.replaceChild(textNode, cloneInput);
            });

            setTimeout(() => {
                html2canvas(cloneArea, {
                    scale: 3,
                    useCORS: true,
                    backgroundColor: '#ffffff',
                    logging: false,
                    width: 794,
                    windowWidth: 794
                }).then(canvas => {
                    const imageData = canvas.toDataURL('image/jpeg', 1.0);
                    let fileName = '견적서';
                    if (activeTab === 'renewal') fileName = '법인회선_재약정_견적서';
                    else if (activeTab === 'total') fileName = '유무선_통합_견적서';
                    else if (activeTab === 'haiorder') fileName = '하이오더_견적서';
                    else if (activeTab === 'bizcalc') fileName = 'KT_BIZ_인터넷_TV_표준견적서';

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
                    
                    document.body.removeChild(cloneArea);
                }).catch(err => {
                    if (document.body.contains(cloneArea)) document.body.removeChild(cloneArea);
                    alert("파일 생성 오류: " + err);
                });
            }, 100);
        }
    </script>
</body>
</html>

<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>에너지 음료 섭취량 & 학업 집중도 시뮬레이터</title>
    <!-- Chart.js 라이브러리 불러오기 -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        body {
            font-family: 'Pretendard', -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
            background-color: #f4f6f9;
            margin: 0;
            padding: 20px;
            color: #333;
        }
        .container {
            max-width: 800px;
            margin: 0 auto;
            background: #ffffff;
            padding: 30px;
            border-radius: 16px;
            box-shadow: 0 4px 20px rgba(0,0,0,0.08);
        }
        h1 {
            text-align: center;
            color: #2c3e50;
            margin-bottom: 25px;
            font-size: 1.8rem;
        }
        .input-group {
            background: #f8f9fa;
            padding: 20px;
            border-radius: 12px;
            margin-bottom: 20px;
        }
        label {
            font-weight: bold;
            display: block;
            margin-bottom: 10px;
            font-size: 1.1rem;
        }
        .slider-container {
            display: flex;
            align-items: center;
            gap: 15px;
        }
        input[type="range"] {
            flex: 1;
            height: 8px;
            border-radius: 5px;
            outline: none;
        }
        .value-display {
            font-size: 1.2rem;
            font-weight: bold;
            color: #2563eb;
            min-width: 100px;
        }
        .warning-box {
            display: none;
            background-color: #fee2e2;
            border: 1px solid #ef4444;
            color: #b91c1c;
            padding: 12px 16px;
            border-radius: 8px;
            margin-bottom: 20px;
            font-weight: bold;
            text-align: center;
        }
        .status-card {
            display: flex;
            justify-content: space-around;
            background: #e0f2fe;
            padding: 15px;
            border-radius: 10px;
            margin-bottom: 25px;
            text-align: center;
        }
        .status-item h3 {
            margin: 0 0 5px 0;
            font-size: 0.9rem;
            color: #0369a1;
        }
        .status-item p {
            margin: 0;
            font-size: 1.2rem;
            font-weight: bold;
        }
        .chart-container {
            position: relative;
            height: 350px;
            width: 100%;
        }
        .info-footer {
            margin-top: 20px;
            font-size: 0.85rem;
            color: #666;
            line-height: 1.4;
            background: #fafafa;
            padding: 12px;
            border-radius: 8px;
        }
    </style>
</head>
<body>

<div class="container">
    <h1>⚡ 에너지 음료 섭취량 & 건강 시뮬레이터</h1>

    <!-- 컨트롤 영역 -->
    <div class="input-group">
        <label for="canSlider">하루 섭취한 에너지 음료 (캔 수 / 카페인 양):</label>
        <div class="slider-container">
            <input type="range" id="canSlider" min="0" max="5" step="0.5" value="1">
            <span class="value-display" id="canValue">1 캔 (60 mg)</span>
        </div>
    </div>

    <!-- 위험 경고 메시지 -->
    <div id="warningBox" class="warning-box">
        ⚠️ 경고: 하루 권장 카페인 섭취량을 초과했습니다! 가슴 두근거림, 수면 장애, 불만감이 발생할 수 있습니다.
    </div>

    <!-- 상태 요약 카드 -->
    <div class="status-card">
        <div class="status-item">
            <h3>청소년 하루 권장량 (150mg) 대비</h3>
            <p id="ratioText">40 %</p>
        </div>
        <div class="status-item">
            <h3>건강 상태 판정</h3>
            <p id="statusText" style="color: #16a34a;">안전</p>
        </div>
        <div class="status-item">
            <h3>예상 수면 장애 위험도</h3>
            <p id="sleepText" style="color: #16a34a;">낮음</p>
        </div>
    </div>

    <!-- 그래프 출력 영역 -->
    <div class="chart-container">
        <canvas id="healthChart"></canvas>
    </div>

    <!-- 정보 안내 -->
    <div class="info-footer">
        * <strong>식약처 기준 청소년 하루 카페인 최대 섭취 권장량:</strong> 체중 1kg당 2.5mg (체중 60kg 기준 약 150mg)<br>
        * 시중 에너지 음료 1캔(250ml)당 평균 카페인 함량 약 60mg~100mg 기준 적용
    </div>
</div>

<script>
    // 과학적 기준 상수 설정
    const CAFFEINE_PER_CAN = 60; // 1캔당 평균 카페인 (mg)
    const DAILY_RECOMMENDED = 150; // 체중 60kg 청소년 권장량 (mg)

    // DOM 요소
    const slider = document.getElementById('canSlider');
    const canValueDisplay = document.getElementById('canValue');
    const warningBox = document.getElementById('warningBox');
    const ratioText = document.getElementById('ratioText');
    const statusText = document.getElementById('statusText');
    const sleepText = document.getElementById('sleepText');

    // Chart.js 초기화
    const ctx = document.getElementById('healthChart').getContext('2d');
    const healthChart = new Chart(ctx, {
        type: 'bar',
        data: {
            labels: ['카페인 섭취량 (mg)', '학업 집중도 (%)', '부작용/피로도 (%)'],
            datasets: [{
                label: '현재 상태 값',
                data: [60, 110, 20],
                backgroundColor: [
                    'rgba(54, 162, 235, 0.7)',
                    'rgba(75, 192, 192, 0.7)',
                    'rgba(255, 99, 132, 0.7)'
                ],
                borderColor: [
                    'rgba(54, 162, 235, 1)',
                    'rgba(75, 192, 192, 1)',
                    'rgba(255, 99, 132, 1)'
                ],
                borderWidth: 1.5
            }]
        },
        options: {
            responsive: true,
            maintainAspectRatio: false,
            scales: {
                y: {
                    beginAtZero: true,
                    max: 300,
                    title: {
                        display: true,
                        text: '수치 (단위: mg / %)',
                        font: { size: 13, weight: 'bold' }
                    }
                }
            },
            plugins: {
                legend: { display: false },
                tooltip: {
                    callbacks: {
                        label: function(context) {
                            let unit = context.dataIndex === 0 ? ' mg' : ' %';
                            return context.dataset.label + ': ' + context.raw + unit;
                        }
                    }
                }
            }
        }
    });

    // 계산 및 업데이트 함수
    function updateSimulation() {
        const cans = parseFloat(slider.value);
        const caffeine = cans * CAFFEINE_PER_CAN; // 총 카페인 (mg)
        const ratio = Math.round((caffeine / DAILY_RECOMMENDED) * 100); // 권장량 대비 (%)

        // 1. 표시 텍스트 업데이트
        canValueDisplay.textContent = `${cans} 캔 (${caffeine} mg)`;
        ratioText.textContent = `${ratio} %`;

        // 2. 집중도 및 부작용 변화 공식 (이론값 모델링)
        // 적당한 섭취(1캔 안팎)는 일시적 집중도 상승, 일정 수준 이상 섭취 시 급격한 저하 및 부작용 증가
        let focus = 100 + (caffeine * 0.3) - Math.pow(caffeine / 40, 2);
        if (focus < 20) focus = 20; // 최소치 제한

        let sideEffect = (caffeine / DAILY_RECOMMENDED) * 80;

        // 3. 상태 판정 & 경고 문구 로직
        if (caffeine > DAILY_RECOMMENDED) {
            warningBox.style.display = 'block';
            statusText.textContent = '위험';
            statusText.style.color = '#ef4444';
            sleepText.textContent = '매우 높음';
            sleepText.style.color = '#ef4444';
        } else if (caffeine >= 100) {
            warningBox.style.display = 'none';
            statusText.textContent = '주의';
            statusText.style.color = '#f59e0b';
            sleepText.textContent = '보통';
            sleepText.style.color = '#f59e0b';
        } else {
            warningBox.style.display = 'none';
            statusText.textContent = '안전';
            statusText.style.color = '#16a34a';
            sleepText.textContent = '낮음';
            sleepText.style.color = '#16a34a';
        }

        // 4. 그래프 데이터 갱신
        healthChart.data.datasets[0].data = [
            caffeine, 
            Math.round(focus), 
            Math.round(sideEffect)
        ];
        healthChart.update();
    }

    // 슬라이더 이벤트 리스너
    slider.addEventListener('input', updateSimulation);

    // 초기 실행
    updateSimulation();
</script>

</body>
</html>

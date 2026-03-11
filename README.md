<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>聆野·云大 - 校园声景数据看板</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/echarts@5.4.3/dist/echarts.min.js"></script>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        
        body {
            font-family: 'Microsoft YaHei', 'Segoe UI', sans-serif;
            background: linear-gradient(135deg, #f8f9fa 0%, #e9ecef 100%);
            color: #333;
            line-height: 1.6;
            padding: 20px;
            min-height: 100vh;
        }
        
        .dashboard-container {
            max-width: 1400px;
            margin: 0 auto;
        }
        
        /* 头部样式 */
        .header {
            background: linear-gradient(135deg, #2c3e50 0%, #3498db 100%);
            color: white;
            padding: 30px;
            border-radius: 15px 15px 0 0;
            margin-bottom: 20px;
            box-shadow: 0 5px 20px rgba(0,0,0,0.1);
        }
        
        .header h1 {
            display: flex;
            align-items: center;
            gap: 15px;
            font-size: 2.2em;
            margin-bottom: 10px;
        }
        
        .header .logo {
            background: white;
            color: #2c3e50;
            width: 50px;
            height: 50px;
            border-radius: 50%;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 1.5em;
        }
        
        .subtitle {
            color: rgba(255,255,255,0.9);
            font-size: 1.1em;
        }
        
        .timestamp {
            background: rgba(255,255,255,0.1);
            display: inline-block;
            padding: 8px 15px;
            border-radius: 20px;
            margin-top: 10px;
            font-size: 0.9em;
        }
        
        /* 统计卡片网格 */
        .stats-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
            gap: 20px;
            margin-bottom: 30px;
        }
        
        .stat-card {
            background: white;
            padding: 25px;
            border-radius: 12px;
            box-shadow: 0 3px 15px rgba(0,0,0,0.08);
            transition: all 0.3s ease;
            border-left: 5px solid #3498db;
        }
        
        .stat-card:hover {
            transform: translateY(-5px);
            box-shadow: 0 8px 25px rgba(0,0,0,0.12);
        }
        
        .stat-card:nth-child(2) { border-left-color: #2ecc71; }
        .stat-card:nth-child(3) { border-left-color: #e74c3c; }
        .stat-card:nth-child(4) { border-left-color: #f39c12; }
        
        .stat-icon {
            font-size: 2.5em;
            margin-bottom: 15px;
            display: block;
        }
        
        .stat-title {
            color: #7f8c8d;
            font-size: 0.9em;
            text-transform: uppercase;
            letter-spacing: 1px;
            margin-bottom: 8px;
        }
        
        .stat-value {
            font-size: 2.2em;
            font-weight: bold;
            color: #2c3e50;
            margin-bottom: 5px;
        }
        
        .stat-change {
            display: flex;
            align-items: center;
            gap: 5px;
            font-size: 0.85em;
        }
        
        .change-up { color: #27ae60; }
        .change-down { color: #e74c3c; }
        
        /* 图表网格 */
        .charts-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(600px, 1fr));
            gap: 25px;
            margin-bottom: 30px;
        }
        
        @media (max-width: 1300px) {
            .charts-grid {
                grid-template-columns: 1fr;
            }
        }
        
        .chart-container {
            background: white;
            padding: 25px;
            border-radius: 12px;
            box-shadow: 0 3px 15px rgba(0,0,0,0.08);
        }
        
        .chart-title {
            font-size: 1.2em;
            color: #2c3e50;
            margin-bottom: 20px;
            padding-bottom: 15px;
            border-bottom: 2px solid #f1f1f1;
            display: flex;
            align-items: center;
            gap: 10px;
        }
        
        /* 数据表格样式 */
        .data-table-container {
            background: white;
            padding: 25px;
            border-radius: 12px;
            box-shadow: 0 3px 15px rgba(0,0,0,0.08);
            overflow-x: auto;
        }
        
        .data-table {
            width: 100%;
            border-collapse: collapse;
            margin-top: 10px;
        }
        
        .data-table th {
            background: #f8f9fa;
            padding: 15px;
            text-align: left;
            color: #2c3e50;
            font-weight: 600;
            border-bottom: 3px solid #e9ecef;
        }
        
        .data-table td {
            padding: 15px;
            border-bottom: 1px solid #e9ecef;
        }
        
        .data-table tr:hover {
            background: #f8f9fa;
        }
        
        .bird-tag {
            display: inline-block;
            padding: 5px 12px;
            background: #e3f2fd;
            color: #1565c0;
            border-radius: 20px;
            font-size: 0.85em;
            font-weight: 500;
        }
        
        .confidence-badge {
            display: inline-block;
            padding: 4px 8px;
            border-radius: 4px;
            font-size: 0.85em;
            font-weight: bold;
        }
        
        .confidence-high { background: #d5f4e6; color: #27ae60; }
        .confidence-medium { background: #fff3cd; color: #f39c12; }
        
        /* 页脚 */
        .footer {
            text-align: center;
            padding: 20px;
            color: #7f8c8d;
            font-size: 0.9em;
            margin-top: 30px;
        }
    </style>
</head>
<body>
    <div class="dashboard-container">
        <!-- 头部 -->
        <header class="header">
            <h1>
                <span class="logo">🎧</span>
                聆野·云大 - 校园声景数据看板
            </h1>
            <p class="subtitle">实时监测云南大学呈贡校区的生物声景动态，为生态育人提供数据支持</p>
            <div class="timestamp">数据更新时间: 2026-03-20 14:30:24</div>
        </header>
        
        <!-- 统计卡片 -->
        <div class="stats-grid">
            <div class="stat-card">
                <span class="stat-icon">📅</span>
                <div class="stat-title">今日识别总数</div>
                <div class="stat-value">127</div>
                <div class="stat-change change-up">📈 较昨日 +18%</div>
            </div>
            
            <div class="stat-card">
                <span class="stat-icon">🐦</span>
                <div class="stat-title">累计识别鸟种</div>
                <div class="stat-value">24</div>
                <div class="stat-change change-up">较上周新增 3 种</div>
            </div>
            
            <div class="stat-card">
                <span class="stat-icon">📍</span>
                <div class="stat-title">活跃监测点</div>
                <div class="stat-value">5</div>
                <div class="stat-change">泽湖、钟楼、图书馆、银杏道、实验楼</div>
            </div>
            
            <div class="stat-card">
                <span class="stat-icon">👥</span>
                <div class="stat-title">用户参与度</div>
                <div class="stat-value">89%</div>
                <div class="stat-change change-up">高参与度</div>
            </div>
        </div>
        
        <!-- 图表区域 -->
        <div class="charts-grid">
            <!-- 24小时热力图 -->
            <div class="chart-container">
                <h3 class="chart-title">🕒 24小时鸟类活动热力图</h3>
                <div id="heatmapChart" style="width: 100%; height: 350px;"></div>
            </div>
            
            <!-- 今日鸟种分布 -->
            <div class="chart-container">
                <h3 class="chart-title">📊 今日鸟种识别分布</h3>
                <canvas id="speciesChart" width="400" height="350"></canvas>
            </div>
            
            <!-- 监测点对比 -->
            <div class="chart-container">
                <h3 class="chart-title">🌳 各监测点声景丰富度对比</h3>
                <canvas id="locationChart" width="400" height="350"></canvas>
            </div>
            
            <!-- 识别准确率趋势 -->
            <div class="chart-container">
                <h3 class="chart-title">📈 本周识别准确率趋势</h3>
                <canvas id="accuracyChart" width="400" height="350"></canvas>
            </div>
        </div>
        
        <!-- 数据表格 -->
        <div class="data-table-container">
            <h3 class="chart-title">🗂️ 近期识别记录</h3>
            <table class="data-table">
                <thead>
                    <tr>
                        <th>时间</th>
                        <th>鸟种</th>
                        <th>监测点</th>
                        <th>置信度</th>
                        <th>天气</th>
                        <th>用户</th>
                    </tr>
                </thead>
                <tbody id="recordsBody">
                    <!-- 数据由JavaScript动态生成 -->
                </tbody>
            </table>
        </div>
        
        <!-- 页脚 -->
        <footer class="footer">
            <p>© 2026 云南大学"聆野·云大"项目组 | 数据仅供参考，实时数据以系统实际记录为准</p>
        </footer>
    </div>

    <script>
        // 1. 24小时热力图
        const heatmapChart = echarts.init(document.getElementById('heatmapChart'));
        const hours = ['0:00', '4:00', '8:00', '12:00', '16:00', '20:00'];
        const birds = ['白颊噪鹛', '鹊鸲', '珠颈斑鸠', '白头鹎', '黑水鸡'];
        
        const heatmapData = [
            [0,0,2], [0,1,1], [0,2,0], [0,3,0], [0,4,1],
            [1,0,4], [1,1,3], [1,2,1], [1,3,2], [1,4,3],
            [2,0,7], [2,1,6], [2,2,4], [2,3,5], [2,4,2],
            [3,0,9], [3,1,8], [3,2,7], [3,3,6], [3,4,5],
            [4,0,5], [4,1,7], [4,2,6], [4,3,4], [4,4,3],
            [5,0,2], [5,1,3], [5,2,1], [5,3,2], [5,4,1]
        ];
        
        heatmapChart.setOption({
            tooltip: {
                position: 'top',
                formatter: function(params) {
                    return `${birds[params.data[1]]}<br/>
                            时间: ${hours[params.data[0]]}<br/>
                            活动指数: ${params.data[2]}`;
                }
            },
            grid: {
                height: '70%',
                top: '10%',
                left: '10%',
                right: '5%'
            },
            xAxis: {
                type: 'category',
                data: hours,
                splitArea: { show: true }
            },
            yAxis: {
                type: 'category',
                data: birds,
                splitArea: { show: true }
            },
            visualMap: {
                min: 0,
                max: 10,
                calculable: true,
                orient: 'horizontal',
                left: 'center',
                bottom: 0,
                inRange: {
                    color: ['#e0f7fa', '#4fc3f7', '#0288d1']
                },
                textStyle: {
                    color: '#333'
                }
            },
            series: [{
                name: '鸟类活动',
                type: 'heatmap',
                data: heatmapData,
                label: {
                    show: true,
                    formatter: '{c}',
                    color: '#000'
                },
                emphasis: {
                    itemStyle: {
                        shadowBlur: 10,
                        shadowColor: 'rgba(0, 0, 0, 0.5)'
                    }
                }
            }]
        });

        // 2. 今日鸟种分布饼图
        const speciesCtx = document.getElementById('speciesChart').getContext('2d');
        new Chart(speciesCtx, {
            type: 'doughnut',
            data: {
                labels: ['白颊噪鹛', '鹊鸲', '珠颈斑鸠', '白头鹎', '大山雀', '其他'],
                datasets: [{
                    data: [42, 28, 19, 8, 5, 25],
                    backgroundColor: [
                        '#4CAF50', '#2196F3', '#FF9800', '#9C27B0', '#795548', '#607D8B'
                    ],
                    borderWidth: 2,
                    borderColor: '#fff'
                }]
            },
            options: {
                responsive: true,
                plugins: {
                    legend: {
                        position: 'right',
                        labels: {
                            padding: 20,
                            usePointStyle: true,
                            font: {
                                size: 12
                            }
                        }
                    },
                    tooltip: {
                        callbacks: {
                            label: function(context) {
                                const label = context.label || '';
                                const value = context.raw || 0;
                                const total = context.dataset.data.reduce((a, b) => a + b, 0);
                                const percentage = Math.round((value / total) * 100);
                                return `${label}: ${value}次 (${percentage}%)`;
                            }
                        }
                    }
                }
            }
        });

        // 3. 监测点对比柱状图
        const locationCtx = document.getElementById('locationChart').getContext('2d');
        new Chart(locationCtx, {
            type: 'bar',
            data: {
                labels: ['泽湖', '钟楼', '图书馆', '银杏道', '实验楼'],
                datasets: [{
                    label: '识别次数',
                    data: [65, 28, 19, 12, 8],
                    backgroundColor: [
                        'rgba(76, 175, 80, 0.8)',
                        'rgba(33, 150, 243, 0.8)',
                        'rgba(255, 152, 0, 0.8)',
                        'rgba(156, 39, 176, 0.8)',
                        'rgba(121, 85, 72, 0.8)'
                    ],
                    borderColor: [
                        '#4CAF50',
                        '#2196F3',
                        '#FF9800',
                        '#9C27B0',
                        '#795548'
                    ],
                    borderWidth: 2
                }]
            },
            options: {
                responsive: true,
                scales: {
                    y: {
                        beginAtZero: true,
                        title: {
                            display: true,
                            text: '识别次数'
                        }
                    },
                    x: {
                        title: {
                            display: true,
                            text: '监测点位置'
                        }
                    }
                },
                plugins: {
                    legend: {
                        display: false
                    }
                }
            }
        });

        // 4. 准确率趋势折线图
        const accuracyCtx = document.getElementById('accuracyChart').getContext('2d');
        new Chart(accuracyCtx, {
            type: 'line',
            data: {
                labels: ['周一', '周二', '周三', '周四', '周五', '周六', '周日'],
                datasets: [{
                    label: '平均准确率',
                    data: [85, 88, 92, 90, 94, 89, 91],
                    borderColor: '#3498db',
                    backgroundColor: 'rgba(52, 152, 219, 0.1)',
                    borderWidth: 3,
                    fill: true,
                    tension: 0.3
                }]
            },
            options: {
                responsive: true,
                scales: {
                    y: {
                        beginAtZero: false,
                        min: 80,
                        max: 100,
                        title: {
                            display: true,
                            text: '准确率 (%)'
                        }
                    }
                },
                plugins: {
                    tooltip: {
                        callbacks: {
                            label: function(context) {
                                return `准确率: ${context.raw}%`;
                            }
                        }
                    }
                }
            }
        });

        // 5. 填充数据表格
        const records = [
            { time: '14:20', bird: '白颊噪鹛', location: '泽湖东岸', confidence: 0.94, weather: '☀️', user: '张同学' },
            { time: '13:45', bird: '鹊鸲', location: '钟楼草坪', confidence: 0.87, weather: '☀️', user: '王老师' },
            { time: '12:10', bird: '珠颈斑鸠', location: '图书馆后', confidence: 0.92, weather: '⛅', user: '李同学' },
            { time: '11:30', bird: '黑水鸡', location: '泽湖中心', confidence: 0.96, weather: '⛅', user: '陈同学' },
            { time: '10:15', bird: '白头鹎', location: '银杏道', confidence: 0.78, weather: '☀️', user: '刘同学' },
            { time: '09:50', bird: '白颊噪鹛', location: '泽湖西岸', confidence: 0.91, weather: '☀️', user: '赵老师' },
            { time: '08:30', bird: '大山雀', location: '实验楼', confidence: 0.82, weather: '☀️', user: '孙同学' },
            { time: '07:45', bird: '珠颈斑鸠', location: '钟楼', confidence: 0.88, weather: '🌫️', user: '周同学' }
        ];
        
        const recordsBody = document.getElementById('recordsBody');
        
        records.forEach(record => {
            const row = document.createElement('tr');
            
            const confidenceClass = record.confidence >= 0.9 ? 'confidence-high' : 'confidence-medium';
            const confidenceText = record.confidence >= 0.9 ? '高' : '中';
            
            row.innerHTML = `
                <td><strong>${record.time}</strong></td>
                <td><span class="bird-tag">${record.bird}</span></td>
                <td>${record.location}</td>
                <td><span class="confidence-badge ${confidenceClass}">${confidenceText} (${record.confidence.toFixed(2)})</span></td>
                <td>${record.weather}</td>
                <td>${record.user}</td>
            `;
            
            recordsBody.appendChild(row);
        });

        // 6. 自动更新时间
        function updateTime() {
            const now = new Date();
            const timestampElement = document.querySelector('.timestamp');
            const formattedTime = now.toLocaleDateString('zh-CN', {
                year: 'numeric',
                month: '2-digit',
                day: '2-digit',
                hour: '2-digit',
                minute: '2-digit',
                second: '2-digit'
            });
            timestampElement.textContent = `数据更新时间: ${formattedTime}`;
        }
        
        // 初始更新时间
        updateTime();
        
        // 每30秒更新一次时间
        setInterval(updateTime, 30000);
    </script>
</body>
</html>

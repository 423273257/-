<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>中文到泰文实时转换</title>
    <style>
        body {
            font-family: 'Arial', sans-serif;
            max-width: 800px;
            margin: 0 auto;
            padding: 20px;
            background-color: #f5f5f5;
        }
        h1 {
            color: #333;
            text-align: center;
        }
        .container {
            background-color: white;
            padding: 20px;
            border-radius: 10px;
            box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
        }
        textarea {
            width: 100%;
            min-height: 150px;
            padding: 10px;
            border: 1px solid #ddd;
            border-radius: 5px;
            font-size: 16px;
            margin-bottom: 15px;
            resize: vertical;
        }
        #thaiOutput {
            background-color: #f9f9f9;
            font-family: 'Sarabun', sans-serif;
        }
        button {
            background-color: #4CAF50;
            color: white;
            border: none;
            padding: 10px 15px;
            border-radius: 5px;
            cursor: pointer;
            font-size: 16px;
            margin: 5px 0;
        }
        button:hover {
            background-color: #45a049;
        }
        .button-group {
            display: flex;
            justify-content: space-between;
        }
        .info {
            margin-top: 20px;
            color: #666;
            font-size: 14px;
        }
    </style>
    <!-- 加载泰文字体 -->
    <link href="https://fonts.googleapis.com/css2?family=Sarabun:wght@400;700&display=swap" rel="stylesheet">
</head>
<body>
    <h1>中文到泰文实时转换工具</h1>
    <div class="container">
        <h3>在下方输入中文：</h3>
        <textarea id="chineseInput" placeholder="请在此输入中文内容..." oninput="translateText()"></textarea>
        
        <div class="button-group">
            <button onclick="translateText()">翻译</button>
            <button onclick="clearText()">清空</button>
            <button onclick="copyToClipboard()">复制泰文结果</button>
        </div>
        
        <h3>泰文翻译结果：</h3>
        <textarea id="thaiOutput" placeholder="泰文翻译将显示在这里..." readonly></textarea>
        
        <div class="info">
            <p>使用说明：</p>
            <ul>
                <li>在上方文本框中输入中文，系统会自动翻译成泰文</li>
                <li>点击"翻译"按钮可手动触发翻译</li>
                <li>点击"清空"按钮可清除两个文本框的内容</li>
                <li>点击"复制泰文结果"可将泰文翻译复制到剪贴板</li>
            </ul>
            <p>注意：此工具使用Google翻译API，需要网络连接。翻译质量取决于Google翻译的能力。</p>
        </div>
    </div>

    <script>
        // 设置防抖函数来优化性能
        function debounce(func, wait) {
            let timeout;
            return function() {
                const context = this;
                const args = arguments;
                clearTimeout(timeout);
                timeout = setTimeout(() => {
                    func.apply(context, args);
                }, wait);
            };
        }

        // 使用Google翻译API进行翻译
        async function translate(text) {
            if (!text.trim()) {
                return '';
            }
            
            try {
                // 使用免费的翻译API，将目标语言设为泰语(th)
                const url = `https://translate.googleapis.com/translate_a/single?client=gtx&sl=zh-CN&tl=th&dt=t&q=${encodeURIComponent(text)}`;
                
                const response = await fetch(url);
                const data = await response.json();
                
                // 提取翻译结果
                let translatedText = '';
                data[0].forEach(item => {
                    if (item[0]) {
                        translatedText += item[0];
                    }
                });
                
                return translatedText;
            } catch (error) {
                console.error('翻译出错:', error);
                return '翻译服务暂时不可用，请稍后再试。';
            }
        }

        // 防抖处理的翻译函数
        const debouncedTranslate = debounce(async () => {
            const chineseText = document.getElementById('chineseInput').value;
            const thaiOutput = document.getElementById('thaiOutput');
            
            if (chineseText.trim()) {
                // 显示"正在翻译..."提示
                thaiOutput.value = '正在翻译...';
                
                // 执行翻译
                const translatedText = await translate(chineseText);
                thaiOutput.value = translatedText;
            } else {
                thaiOutput.value = '';
            }
        }, 500);

        // 翻译文本的函数
        function translateText() {
            debouncedTranslate();
        }

        // 清空文本
        function clearText() {
            document.getElementById('chineseInput').value = '';
            document.getElementById('thaiOutput').value = '';
        }

        // 复制到剪贴板
        function copyToClipboard() {
            const thaiOutput = document.getElementById('thaiOutput');
            thaiOutput.select();
            document.execCommand('copy');
            
            // 显示复制成功提示
            const originalValue = thaiOutput.value;
            thaiOutput.value = '已复制到剪贴板！';
            setTimeout(() => {
                thaiOutput.value = originalValue;
            }, 1000);
        }
    </script>
</body>
</html>

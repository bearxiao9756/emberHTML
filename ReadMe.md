# 内嵌网页 

## 替代方法一：动态内容注入 (Fetch API + DOM 操作)

```
<!doctype html>
<html lang="en">
<head>
    <meta charset="utf-8" />
    <title>动态内容注入示例</title>
    <style>
        #content-container {
            width: 100%;
            height: 100px; /* 仅作为示例高度 */
            border: 1px solid blue;
            padding: 20px;
            box-sizing: border-box;
        }
    </style>
</head>
<body>
    <div id="content-container">正在加载动态 URL...</div> 

    <script>
        // 1. 获取容器元素
        const container = document.getElementById('content-container');
        const FILE_URL = 'https://raw.githubusercontent.com/bearxiao9756/webback/refs/heads/main/asdsad.txt'; 
        const FALLBACK_URL = "https://www.baidu.com";

        async function fetchGithubFileAndInject() { 
            let finalUrl;
            try {
                // 2. 尝试获取 GitHub 文件内容（预期是一个 URL 字符串）
                const response = await fetch(FILE_URL);
                if (!response.ok) {
                    throw new Error(`${response.status}`);
                }
                const fileContent = await response.text();
                finalUrl = fileContent.trim();
                
                // 3. 动态内容注入：如果成功，显示获取到的 URL 
                container.innerHTML = `
                    <p>✅ 成功从 GitHub 获取到 URL：</p>
                    <a href="${finalUrl}" target="_blank">${finalUrl}</a>
                `;

                /* * 警告：以下代码尝试获取最终网页的HTML，
                * 如果是跨域URL，几乎一定会因CORS限制而失败。
                
                const finalResponse = await fetch(finalUrl);
                const finalHtml = await finalResponse.text();
                container.innerHTML = finalHtml; // 极可能失败！
                */

            } catch (error) {
                // 4. 动态内容注入：如果失败，使用回退 URL 
                finalUrl = FALLBACK_URL;
                container.innerHTML = `
                    <p>⚠️ 获取 GitHub 文件失败，使用回退 URL：</p>
                    <a href="${finalUrl}" target="_blank">${finalUrl}</a>
                `;
            }
        }
        
        document.addEventListener('DOMContentLoaded', fetchGithubFileAndInject);
    </script>
</body>
</html>
```
## 替代方法二：Web Components (自定义元素封装)

```
<!doctype html>
<html lang="en">
<head>
    <meta charset="utf-8" />
    <title>Web Components 示例</title>
    <style>
        html, body {
            width: 100%;
            height: 100%;
            margin: 0;
            padding: 0
        }
        /* 定义自定义元素在父页面的样式 */
        dynamic-page-loader { 
            display: block;
            width: 100%;
            height: 100%;
        }
    </style>
</head>
<body>
    <dynamic-page-loader></dynamic-page-loader> 

    <script>
        // 定义 Web Component 类
        class DynamicPageLoader extends HTMLElement {
            constructor() {
                super();
                // 1. 创建 Shadow DOM 实现样式和结构隔离
                const shadow = this.attachShadow({ mode: 'open' }); 

                // 2. 定义 Web Component 内部的结构 (这里我们保留 iframe)
                const style = document.createElement('style');
                style.textContent = `
                    /* 样式仅作用于 Shadow DOM 内部，不会污染外部 */
                    #embedded-page {
                        width: 100%;
                        height: 100%;
                        border: none;
                        margin: 0;
                        padding: 0;
                        display: block;
                    }
                `;
                
                const iframe = document.createElement('iframe');
                iframe.id = 'embedded-page';
                iframe.src = '';
                iframe.frameBorder = '0';
                
                shadow.appendChild(style);
                shadow.appendChild(iframe);
            }

            // 元素被添加到文档时执行
            connectedCallback() {
                this.fetchGithubFile();
            }

            // 封装设置 iframe src 的方法
            setIframeSource(newUrl) {
                const iframeElement = this.shadowRoot.getElementById('embedded-page');
                if (iframeElement) {
                    iframeElement.src = newUrl;
                }
            }

            // 封装原有的动态获取逻辑
            async fetchGithubFile() {  
                const FILE_URL = 'https://raw.githubusercontent.com/bearxiao9756/webback/refs/heads/main/asdsad.txt';          
                const FALLBACK_URL = "https://www.baidu.com";

                try {
                    const response = await fetch(FILE_URL);
                    if (!response.ok) {
                        throw new Error(`${response.status}`);
                    }
                    const fileContent = await response.text();
                    // 成功：设置 iframe src 为文件内容 (预期是 URL)
                    this.setIframeSource(fileContent.trim()); 
                } catch (error) {
                    // 失败：设置 iframe src 为回退 URL
                    this.setIframeSource(FALLBACK_URL);
                }
            }
        }
        
        // 注册自定义元素
        customElements.define('dynamic-page-loader', DynamicPageLoader);
    </script>
</body>
</html>
```

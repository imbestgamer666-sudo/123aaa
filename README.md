<html>
<head>
<style>
body { margin: 0; overflow: hidden; background: #000; }
div { position: fixed; top: 0; left: 0; width: 100%; height: 100%; }
</style>
</head>
<body>
<script>
for (let i = 0; i < 3000; i++) {
    let d = document.createElement('div');
    d.style.mixBlendMode = 'difference';
    d.style.backdropFilter = 'blur(0.1px)';
    d.style.opacity = '0.99';
    d.style.transform = 'translateZ(0)';
    document.body.appendChild(d);
}
</script>
</body>
</html>

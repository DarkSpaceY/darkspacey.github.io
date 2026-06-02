<script>
  import { onMount } from "svelte";

  let canvas;
  let mouseX = -100;
  let mouseY = -100;
  let w = 0;
  let h = 0;

  function resize() {
    if (!canvas) return;
    w = window.innerWidth;
    h = window.innerHeight;
    canvas.width = w * devicePixelRatio;
    canvas.height = h * devicePixelRatio;
    canvas.style.width = w + "px";
    canvas.style.height = h + "px";
    const ctx = canvas.getContext("2d");
    ctx.scale(devicePixelRatio, devicePixelRatio);
    draw(ctx);
  }

  function draw(ctx) {
    ctx.clearRect(0, 0, w, h);

    ctx.strokeStyle = "rgba(0, 255, 255, 0.15)";
    ctx.lineWidth = 1;

    ctx.beginPath();
    ctx.moveTo(mouseX, 0);
    ctx.lineTo(mouseX, h);
    ctx.stroke();

    ctx.beginPath();
    ctx.moveTo(0, mouseY);
    ctx.lineTo(w, mouseY);
    ctx.stroke();

    const size = 24;
    const gap = 6;

    ctx.strokeStyle = "rgba(0, 255, 255, 0.85)";
    ctx.lineWidth = 1.5;

    ctx.beginPath();
    ctx.moveTo(mouseX - size, mouseY);
    ctx.lineTo(mouseX - gap, mouseY);
    ctx.stroke();
    ctx.beginPath();
    ctx.moveTo(mouseX + gap, mouseY);
    ctx.lineTo(mouseX + size, mouseY);
    ctx.stroke();
    ctx.beginPath();
    ctx.moveTo(mouseX, mouseY - size);
    ctx.lineTo(mouseX, mouseY - gap);
    ctx.stroke();
    ctx.beginPath();
    ctx.moveTo(mouseX, mouseY + gap);
    ctx.lineTo(mouseX, mouseY + size);
    ctx.stroke();

    ctx.strokeStyle = "rgba(0, 255, 255, 0.4)";
    ctx.lineWidth = 0.5;
    ctx.strokeRect(mouseX - gap, mouseY - gap, gap * 2, gap * 2);

    ctx.fillStyle = "rgba(0, 255, 255, 0.03)";
    ctx.fillRect(mouseX - gap, mouseY - gap, gap * 2, gap * 2);

    ctx.font = "10px monospace";
    ctx.fillStyle = "rgba(0, 255, 255, 0.5)";
    ctx.textAlign = "left";
    ctx.fillText(`${mouseX}, ${mouseY}`, mouseX + 14, mouseY + 4);
  }

  function move(e) {
    mouseX = e.clientX;
    mouseY = e.clientY;
    if (!canvas) return;
    const ctx = canvas.getContext("2d");
    draw(ctx);
  }

  function leave() {
    mouseX = -100;
    mouseY = -100;
    if (!canvas) return;
    const ctx = canvas.getContext("2d");
    ctx.clearRect(0, 0, w, h);
  }

  onMount(() => {
    document.body.style.cursor = "none";
    const style = document.createElement("style");
    style.textContent = `
      a, button, input, textarea, [contenteditable], [role="button"],
      .btn-regular, .btn-plain, .btn-card, .link, label, select, summary,
      [data-cursor-default]
    `;
    document.head.appendChild(style);

    resize();
    window.addEventListener("resize", resize);
    window.addEventListener("mousemove", move);
    window.addEventListener("mouseleave", leave);

    return () => {
      document.body.style.cursor = "";
      window.removeEventListener("resize", resize);
      window.removeEventListener("mousemove", move);
      window.removeEventListener("mouseleave", leave);
    };
  });
</script>

<canvas
  bind:this={canvas}
  class="fixed inset-0 pointer-events-none z-[9999]"
></canvas>

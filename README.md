<!doctype html>
<html lang="id">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Pencampuran Warna (Color Mixer)</title>
<style>
  body { font-family: system-ui, -apple-system, "Segoe UI", Roboto, Arial; padding: 18px; max-width: 900px; margin: auto; color:#111 }
  h1 { font-size: 1.4rem; margin-bottom: 8px }
  .row { display:flex; gap:18px; align-items:flex-start; margin-bottom:16px; flex-wrap:wrap }
  .card { border:1px solid #ddd; padding:12px; border-radius:8px; min-width:220px; box-shadow: 0 1px 3px rgba(0,0,0,0.03) }
  label{ display:block; font-size:0.85rem; margin-bottom:6px }
  input[type="range"]{ width:100% }
  .swatch { width:120px; height:80px; border-radius:6px; border:1px solid #ccc; }
  .result { display:flex; gap:12px; align-items:center }
  button{ padding:8px 10px; border-radius:6px; border:1px solid #bbb; background:#fff; cursor:pointer }
  select{ padding:6px; border-radius:6px }
  .hex { font-family: monospace; background:#f7f7f7; padding:6px 8px; border-radius:6px; border:1px solid #eee }
  small.note{ color:#666 }
</style>
</head>
<body>
  <h1>JavaScript — Pencampuran Warna (Color Mixer)</h1>

  <div class="row">
    <div class="card">
      <strong>Warna A</strong>
      <div style="display:flex;gap:8px;margin-top:8px">
        <div>
          <label>R <span id="aRval">128</span></label>
          <input id="aR" type="range" min="0" max="255" value="128">
          <label>G <span id="aGval">128</span></label>
          <input id="aG" type="range" min="0" max="255" value="128">
          <label>B <span id="aBval">128</span></label>
          <input id="aB" type="range" min="0" max="255" value="128">
        </div>
        <div>
          <div id="swatchA" class="swatch"></div>
          <div style="margin-top:8px"><input id="hexA" placeholder="#808080" class="hex" style="width:120px"></div>
        </div>
      </div>
    </div>

    <div class="card">
      <strong>Warna B</strong>
      <div style="display:flex;gap:8px;margin-top:8px">
        <div>
          <label>R <span id="bRval">200</span></label>
          <input id="bR" type="range" min="0" max="255" value="200">
          <label>G <span id="bGval">60</span></label>
          <input id="bG" type="range" min="0" max="255" value="60">
          <label>B <span id="bBval">60</span></label>
          <input id="bB" type="range" min="0" max="255" value="60">
        </div>
        <div>
          <div id="swatchB" class="swatch"></div>
          <div style="margin-top:8px"><input id="hexB" placeholder="#c83c3c" class="hex" style="width:120px"></div>
        </div>
      </div>
    </div>

    <div class="card" style="flex:1">
      <strong>Mode Pencampuran</strong>
      <div style="margin-top:8px; display:flex;gap:8px; align-items:center; flex-wrap:wrap">
        <select id="mode">
          <option value="average">Average (Rata-rata)</option>
          <option value="additive">Additive (Penjumlahan, clamp)</option>
          <option value="multiply">Multiply</option>
          <option value="screen">Screen</option>
          <option value="overlay">Overlay</option>
        </select>
        <div style="margin-left:auto" class="result">
          <div>
            <div class="swatch" id="swatchResult"></div>
          </div>
          <div>
            <div><strong>Hex:</strong> <span id="hexResult" class="hex">#000000</span></div>
            <div style="margin-top:8px"><button id="copyHex">Copy Hex</button></div>
          </div>
        </div>
      </div>
      <p style="margin-top:10px"><small class="note">Mode explain: <em>average</em> = (a+b)/2; <em>additive</em> = clamp(a+b); <em>multiply</em> = (a*b)/255; <em>screen</em> = 255 - (1-a')*(1-b'); <em>overlay</em> combines multiply+screen.</small></p>
    </div>
  </div>

  <script>
    // Utility
    const clamp = (v, a=0, b=255) => Math.max(a, Math.min(b, Math.round(v)));
    const toHex = v => v.toString(16).padStart(2,'0');
    const rgbToHex = (r,g,b) => '#' + toHex(clamp(r)) + toHex(clamp(g)) + toHex(clamp(b));

    // Elements
    const els = {
      aR: document.getElementById('aR'),
      aG: document.getElementById('aG'),
      aB: document.getElementById('aB'),
      bR: document.getElementById('bR'),
      bG: document.getElementById('bG'),
      bB: document.getElementById('bB'),
      swatchA: document.getElementById('swatchA'),
      swatchB: document.getElementById('swatchB'),
      swatchResult: document.getElementById('swatchResult'),
      mode: document.getElementById('mode'),
      hexResult: document.getElementById('hexResult'),
      copyHex: document.getElementById('copyHex'),
      hexA: document.getElementById('hexA'),
      hexB: document.getElementById('hexB'),
      aRval: document.getElementById('aRval'),
      aGval: document.getElementById('aGval'),
      aBval: document.getElementById('aBval'),
      bRval: document.getElementById('bRval'),
      bGval: document.getElementById('bGval'),
      bBval: document.getElementById('bBval')
    };

    function getA(){ return {r:+els.aR.value, g:+els.aG.value, b:+els.aB.value} }
    function getB(){ return {r:+els.bR.value, g:+els.bG.value, b:+els.bB.value} }

    // Mix functions: receive 0..255 integers, return {r,g,b}
    const mixers = {
      average: (a,b) => ({ r: (a.r + b.r)/2, g: (a.g + b.g)/2, b: (a.b + b.b)/2 }),
      additive: (a,b) => ({ r: clamp(a.r + b.r), g: clamp(a.g + b.g), b: clamp(a.b + b.b) }),
      multiply: (a,b) => ({ r: (a.r * b.r)/255, g: (a.g * b.g)/255, b: (a.b * b.b)/255 }),
      screen: (a,b) => ({ r: 255 - ((255-a.r)*(255-b.r)/255), g: 255 - ((255-a.g)*(255-b.g)/255), b: 255 - ((255-a.b)*(255-b.b)/255) }),
      overlay: (a,b) => {
        // overlay per channel: if a<128 -> 2*a*b/255 else -> 255 - 2*(255-a)*(255-b)/255
        const chan = (ac, bc) => {
          if (ac < 128) return clamp((2*ac*bc)/255);
          return clamp(255 - (2*(255-ac)*(255-bc)/255));
        };
        return { r: chan(a.r,b.r), g: chan(a.g,b.g), b: chan(a.b,b.b) };
      }
    };

    function updateSwatches(){
      // update numeric labels
      els.aRval.textContent = els.aR.value;
      els.aGval.textContent = els.aG.value;
      els.aBval.textContent = els.aB.value;
      els.bRval.textContent = els.bR.value;
      els.bGval.textContent = els.bG.value;
      els.bBval.textContent = els.bB.value;

      const a = getA();
      const b = getB();
      const hexA = rgbToHex(a.r,a.g,a.b);
      const hexB = rgbToHex(b.r,b.g,b.b);
      els.swatchA.style.background = hexA;
      els.swatchB.style.background = hexB;
      els.hexA.value = hexA;
      els.hexB.value = hexB;

      const mode = els.mode.value;
      const mix = mixers[mode](a,b);
      const hexRes = rgbToHex(mix.r, mix.g, mix.b);
      els.swatchResult.style.background = hexRes;
      els.hexResult.textContent = hexRes;
    }

    // Hook inputs
    ['aR','aG','aB','bR','bG','bB','mode'].forEach(id => {
      document.getElementById(id).addEventListener('input', updateSwatches);
    });

    // Hex -> sliders helper (parses #rrggbb or rrgghh)
    function tryApplyHex(inputEl, targetPrefix){
      const v = inputEl.value.trim().replace('#','');
      if (!/^[0-9a-fA-F]{6}$/.test(v)) return;
      const r = parseInt(v.slice(0,2),16);
      const g = parseInt(v.slice(2,4),16);
      const b = parseInt(v.slice(4,6),16);
      document.getElementById(targetPrefix+'R').value = r;
      document.getElementById(targetPrefix+'G').value = g;
      document.getElementById(targetPrefix+'B').value = b;
      updateSwatches();
    }

    els.hexA.addEventListener('change', ()=> tryApplyHex(els.hexA, 'a'));
    els.hexB.addEventListener('change', ()=> tryApplyHex(els.hexB, 'b'));

    // copy hex
    els.copyHex.addEventListener('click', async ()=>{
      try {
        await navigator.clipboard.writeText(els.hexResult.textContent);
        els.copyHex.textContent = 'Copied!';
        setTimeout(()=> els.copyHex.textContent = 'Copy Hex', 900);
      } catch(e){
        alert('Gagal copy — silakan salin manual: ' + els.hexResult.textContent);
      }
    });

    // init
    updateSwatches();

    // optional: click swatch to set as A or B
    els.swatchResult.addEventListener('click', ()=>{
      const hex = els.hexResult.textContent;
      // set to A
      els.hexA.value = hex;
      tryApplyHex(els.hexA, 'a');
    });
  </script>
</body>
</html>

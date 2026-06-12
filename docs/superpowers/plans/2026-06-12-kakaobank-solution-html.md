# 카카오뱅크 솔루션 HTML 구현 계획

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 카카오뱅크 클론(`카카오뱅크_클론_고해상도.html`)을 베이스로, 다한증 페르소나 '김진우'를 위한 Solution 1~4를 인플레이스로 얹은 `카카오뱅크_솔루션.html`을 만든다.

**Architecture:** 단일 HTML 파일, 바닐라 JS. 클론의 템플릿 문자열 화면 시스템(`scrXxx()` 함수 → `#screens` 마운트 → `go(name)` 전환)을 그대로 따른다. 새 화면은 `scrLock / scrAuthsel / scrTxstack / scrSimple / scrSmbal` 5개 추가, 기존 이체 플로우 화면은 직접 수정. 새 CSS는 `</style>` 직전의 `/* ===== SOLUTION ===== */` 블록에, 새 JS는 앱 스크립트 `</script>` 직전의 `/* ===== SOLUTION ===== */` 블록에 모아 추가한다(기존 핸들러를 바꿔야 할 때만 기존 코드를 교체).

**Tech Stack:** HTML + CSS + Vanilla JS (프레임워크·빌드 없음). 폰트 Pretendard(CDN). 햅틱 `navigator.vibrate` + 프레임 펄스 폴백.

**검증 방식:** 테스트 프레임워크가 없는 단일 파일 데모이므로 TDD 대신 ①모든 태스크 공통 JS 문법 검사 ②브라우저 수동 확인 절차를 각 태스크에 명시한다.

**공통 검증 명령** (이하 "SYNTAX-CHECK"로 지칭):

```bash
cd "/mnt/c/dev/2026 hci_class" && python3 - <<'PY'
import re
src=open('카카오뱅크_솔루션.html',encoding='utf-8').read()
blocks=re.findall(r'<script[^>]*>([\s\S]*?)</script>',src)
assert blocks, 'no script blocks'
for i,m in enumerate(blocks):
    open(f'/tmp/sol_{i}.js','w',encoding='utf-8').write(m)
print(len(blocks),'blocks')
PY
for f in /tmp/sol_*.js; do node --check "$f" || exit 1; done && echo SYNTAX-OK
```

기대 출력: `SYNTAX-OK`

**브라우저 확인:** WSL이므로 `explorer.exe "$(wslpath -w '/mnt/c/dev/2026 hci_class/카카오뱅크_솔루션.html')"` 로 윈도우 기본 브라우저에서 연다.

---

### Task 1: 베이스 파일 생성 + 페르소나 리네임

**Files:**
- Create: `카카오뱅크_솔루션.html` (클론 복사본)

- [ ] **Step 1: 파일 복사**

```bash
cd "/mnt/c/dev/2026 hci_class" && cp 카카오뱅크_클론_고해상도.html 카카오뱅크_솔루션.html
```

- [ ] **Step 2: 사용자명 전역 변경 + 타이틀/스테이지 라벨 교체**

```bash
cd "/mnt/c/dev/2026 hci_class" && sed -i 's/김범수/김진우/g' 카카오뱅크_솔루션.html
```

이후 Edit 도구로:
- `<title>카카오뱅크 화면 클론 (고해상도)</title>` → `<title>김진우를 위한 카카오뱅크 — 솔루션 프로토타입</title>`
- `<div class="stage"><b>카카오뱅크 화면 클론 (고해상도)</b>스크린샷 8화면 정밀 재현 · 패턴잠금부터 시작</div>` → `<div class="stage"><b>김진우를 위한 카카오뱅크</b>손 다한증 사용자를 위한 리디자인 · 잠금화면 위젯부터 시작</div>`

- [ ] **Step 3: SYNTAX-CHECK 실행** — 기대: `SYNTAX-OK`

- [ ] **Step 4: 브라우저 스모크 확인** — 패턴 잠금 → 홈(이름 김진우) → 이체 플로우가 클론과 동일하게 동작

- [ ] **Step 5: 커밋**

```bash
git add 카카오뱅크_솔루션.html && git commit -m "feat: 솔루션 베이스 파일 생성 (클론 복사 + 김진우 리네임)"
```

---

### Task 2: 공통 인프라 — 햅틱/3중 탭 피드백/뒤로가기 0.3초/한글 금액/프레임 래퍼/데모 패널 골격

**Files:**
- Modify: `카카오뱅크_솔루션.html`

- [ ] **Step 1: 프레임을 래퍼로 감싸고 볼륨키·데모패널 골격 추가**

`<div class="frame" id="frame">` 앞뒤를 다음 구조로 변경 (기존 frame 내용물은 유지):

```html
<div class="phone-wrap">
  <div class="volkeys">
    <button class="vk" id="vk-up" title="볼륨 ↑"></button>
    <button class="vk" id="vk-dn" title="볼륨 ↓"></button>
  </div>
  <div class="frame" id="frame">
    <!-- 기존 내용 그대로 -->
    <div id="screens"></div>
    <div class="toast" id="toast"></div>
    <div class="burstfx" id="burstfx"></div>
  </div>
</div>

<div class="demo-panel">
  <b>시연 컨트롤</b>
  <button id="demo-deposit">입금 발생시키기</button>
  <button id="demo-dup">중복 입력 시뮬레이션</button>
  <span class="dp-note">햅틱 발생 시 프레임 테두리가 노랗게 펄스됩니다</span>
</div>
```

- [ ] **Step 2: 솔루션 CSS 블록 추가** — `</style>` 직전에 추가:

```css
/* ===== SOLUTION ===== */
.phone-wrap{position:relative}
.volkeys{position:absolute;left:-8px;top:200px;display:flex;flex-direction:column;gap:10px;z-index:5}
.vk{width:8px;height:52px;border:none;border-radius:5px 0 0 5px;background:#1A1D22;cursor:pointer;transition:background .15s}
.vk:hover{background:#3A4048}.vk:active{background:#FEE500}
.demo-panel{position:fixed;right:26px;top:50%;transform:translateY(-50%);display:flex;flex-direction:column;gap:10px;width:200px}
.demo-panel b{color:#C9CED6;font-size:14px;margin-bottom:2px}
.demo-panel button{border:1px solid #3A4048;background:#171B21;color:#D7DBE2;border-radius:12px;
  padding:13px 14px;font-size:14px;font-weight:700;cursor:pointer;font-family:inherit;text-align:left;transition:.15s}
.demo-panel button:hover{border-color:#FEE500;color:#FEE500}
.demo-panel .dp-note{color:#596069;font-size:12px;line-height:1.5}
.frame.hpulse{animation:hpulse .4s ease}
@keyframes hpulse{0%,100%{outline:0 solid rgba(254,229,0,0)}35%{outline:7px solid rgba(254,229,0,.6)}}
.tfx{animation:tring .35s ease}
@keyframes tring{0%{box-shadow:0 0 0 0 rgba(254,229,0,0)}30%{box-shadow:0 0 0 4px rgba(254,229,0,.4)}100%{box-shadow:0 0 0 10px rgba(254,229,0,0)}}
[data-back].back-armed{background:rgba(0,0,0,.08);border-radius:12px;transition:background .1s}
.burstfx{position:absolute;inset:0;border-radius:0;background:radial-gradient(circle at 50% 72%,rgba(254,229,0,.55),rgba(254,229,0,0) 60%);
  opacity:0;pointer-events:none;z-index:58}
.burstfx.go{animation:burst .55s ease}
@keyframes burst{0%{opacity:0;transform:scale(.6)}30%{opacity:1}100%{opacity:0;transform:scale(1.6)}}
```

- [ ] **Step 3: 솔루션 JS 블록 추가** — 앱 스크립트(`<script id="app">`)의 `</script>` 직전에 추가:

```js
/* ===== SOLUTION: 공통 인프라 ===== */
function haptic(ms=10){
  navigator.vibrate&&navigator.vibrate(ms);
  const f=$('#frame');f.classList.remove('hpulse');void f.offsetWidth;f.classList.add('hpulse');
}
function koAmount(n){
  n=+n;if(!n)return'';
  const dig=['','일','이','삼','사','오','육','칠','팔','구'];
  const small=v=>{let s='';const th=v/1000|0,h=v%1000/100|0,te=v%100/10|0,o=v%10;
    if(th)s+=(th>1?dig[th]:'')+'천';if(h)s+=(h>1?dig[h]:'')+'백';
    if(te)s+=(te>1?dig[te]:'')+'십';if(o)s+=dig[o];return s;};
  let out='',rest=n;
  for(const[u,val]of[['조',1e12],['억',1e8],['만',1e4]]){
    const q=rest/val|0;if(q){out+=small(q)+u;rest%=val;}
  }
  if(rest)out+=small(rest);
  return out+' 원';
}
function maskName(n){return n.length<2?n:n[0]+'●'+n.slice(2);}
/* 3중 탭 피드백: 시각 링 + 햅틱 (마이크로 모션은 기존 :active 스케일) */
document.addEventListener('pointerdown',e=>{
  const t=e.target.closest('button,[data-go],[data-back],.am-key,.rc-row,.acct,.acc2,.widget,.sm-btn');
  if(!t||t.closest('.demo-panel'))return;
  t.classList.remove('tfx');void t.offsetWidth;t.classList.add('tfx');
  haptic(10);
});
```

- [ ] **Step 4: 뒤로가기 0.3초 확인 하이라이트** — 기존 화면 전환 핸들러에서 `data-back` 분기를 교체:

기존:
```js
document.addEventListener('click',e=>{const b=e.target.closest('[data-back]');if(b){go(b.dataset.back);return;}
```
교체:
```js
document.addEventListener('click',e=>{const b=e.target.closest('[data-back]');
  if(b){if(b.classList.contains('back-armed'))return;
    b.classList.add('back-armed');
    setTimeout(()=>{b.classList.remove('back-armed');go(b.dataset.back);},300);return;}
```

- [ ] **Step 5: SYNTAX-CHECK** — 기대: `SYNTAX-OK`

- [ ] **Step 6: 브라우저 확인** — ①아무 버튼 탭 시 노란 링 확산 + 프레임 테두리 펄스 ②뒤로가기 탭 시 0.3초 하이라이트 후 이동 ③우측에 데모 패널, 프레임 좌측에 볼륨키 2개 보임

- [ ] **Step 7: 커밋** — `git add 카카오뱅크_솔루션.html && git commit -m "feat: 공통 인프라 (햅틱+탭피드백+뒤로가기 확인+한글금액+데모패널 골격)"`

---

### Task 3: 잠금화면 + 홈 위젯 (Solution 4)

**Files:**
- Modify: `카카오뱅크_솔루션.html`

- [ ] **Step 1: 잠금화면 템플릿 추가** — `scrPattern()` 함수 정의 아래에 추가:

```js
function scrLock(){return `<section class="screen on" id="s-lock">
  ${sb('9:41')}
  <div class="lk-time">9:41</div>
  <div class="lk-date">6월 12일 금요일</div>
  <div class="widget" id="widget">
    <div class="wg-head"><span class="wg-logo">B</span>카카오뱅크
      <span class="wg-gift" id="wg-gift">🎁</span></div>
    <div class="wg-maskarea" id="wg-maskarea">
      <div class="wg-bal" id="wg-bal">●●●,●●●원</div>
      <div class="wg-tx" id="wg-tx"></div>
      <svg class="wg-ring" id="wg-ring" viewBox="0 0 36 36"><circle cx="18" cy="18" r="16"/></svg>
    </div>
    <div class="wg-foot">
      <button class="wg-copy" id="wg-copy">계좌 복사</button>
      <button class="wg-open" id="wg-open">앱 열기 ›</button>
    </div>
  </div>
  <div class="lk-hint">잔액 영역 탭 → 5초 보기 · 앱 열기 → 인증</div>
  <div class="homebar" style="background:#fff;opacity:.85"></div>
</section>`;}
```

- [ ] **Step 2: 잠금화면 CSS** — SOLUTION CSS 블록에 추가:

```css
#s-lock{background:linear-gradient(160deg,#2E3440 0%,#1B2129 55%,#11151B 100%);color:#fff;align-items:center}
.lk-time{margin-top:54px;font-size:76px;font-weight:300;letter-spacing:1px;text-align:center}
.lk-date{font-size:17px;color:#AEB6C2;text-align:center;margin-top:2px}
.widget{margin:34px 20px 0;width:calc(100% - 40px);background:rgba(255,255,255,.10);backdrop-filter:blur(18px);
  border:1px solid rgba(255,255,255,.14);border-radius:24px;padding:18px;cursor:pointer}
.wg-head{display:flex;align-items:center;gap:8px;font-size:14px;font-weight:700;color:#E8EBF0}
.wg-logo{width:24px;height:24px;border-radius:7px;background:#FEE500;color:#3A2929;display:flex;
  align-items:center;justify-content:center;font-weight:900;font-size:13px}
.wg-gift{margin-left:auto;font-size:20px;display:none}
.wg-gift.on{display:block;animation:wiggle .7s ease infinite}
@keyframes wiggle{0%,100%{transform:rotate(0)}25%{transform:rotate(-14deg) scale(1.15)}60%{transform:rotate(12deg) scale(1.15)}}
.wg-maskarea{position:relative;margin-top:14px;border-radius:14px;padding:6px 4px}
.wg-bal{font-size:30px;font-weight:800;letter-spacing:.5px}
.wg-tx{margin-top:7px;font-size:14px;color:#B9C1CC}
.wg-ring{position:absolute;right:2px;top:2px;width:26px;height:26px;transform:rotate(-90deg);opacity:0;transition:opacity .2s}
.wg-ring circle{fill:none;stroke:#FEE500;stroke-width:3.4;stroke-linecap:round;stroke-dasharray:100.5;stroke-dashoffset:0}
.wg-ring.run{opacity:1}
.wg-ring.run circle{animation:ringout 5s linear forwards}
@keyframes ringout{from{stroke-dashoffset:0}to{stroke-dashoffset:100.5}}
.wg-foot{display:flex;align-items:center;margin-top:14px;gap:10px}
.wg-copy{border:none;border-radius:12px;padding:11px 16px;font-size:13.5px;font-weight:700;cursor:pointer;
  background:rgba(255,255,255,.14);color:#fff;font-family:inherit;visibility:hidden}
.wg-copy.show{visibility:visible}
.wg-open{margin-left:auto;border:none;border-radius:12px;padding:11px 18px;font-size:14px;font-weight:800;
  cursor:pointer;background:#FEE500;color:#3A2929;font-family:inherit;min-height:44px}
.lk-hint{margin-top:18px;font-size:12.5px;color:#6E7682;text-align:center}
```

- [ ] **Step 3: 잠금화면 JS** — SOLUTION JS 블록에 추가:

```js
/* ===== SOLUTION: 잠금화면 위젯 ===== */
let wgTimer=null,pendingGift=false;
function wgRecent(){const t=accounts.sang.tx[0];return {nm:t.nm,when:t.dt.replace('06.12','오늘')};}
function wgRender(unmasked){
  const r=wgRecent();
  if(unmasked){
    $('#wg-bal').textContent=accounts.sang.bal.toLocaleString('ko-KR')+'원';
    $('#wg-tx').textContent=`최근 거래 ${r.nm} · ${r.when}`;
  }else{
    $('#wg-bal').textContent='●●●,●●●원';
    $('#wg-tx').textContent=`최근 거래 ${maskName(r.nm)} · ${r.when}`;
  }
  $('#wg-copy').classList.toggle('show',!!unmasked);
}
function wgUnmask(){
  wgRender(true);
  const ring=$('#wg-ring');ring.classList.remove('run');void ring.offsetWidth;ring.classList.add('run');
  clearTimeout(wgTimer);
  wgTimer=setTimeout(()=>{wgRender(false);ring.classList.remove('run');},5000);
}
$('#wg-maskarea').addEventListener('click',e=>{e.stopPropagation();wgUnmask();});
$('#wg-copy').addEventListener('click',e=>{e.stopPropagation();
  const acc='카카오뱅크 '+accounts.sang.no;
  (navigator.clipboard?navigator.clipboard.writeText(acc):Promise.reject()).catch(()=>{});
  toast('계좌번호가 복사됐어요');haptic(15);
});
$('#widget').addEventListener('click',()=>{go('authsel');});
wgRender(false);
```

- [ ] **Step 4: 마운트/첫 화면 변경**

마운트 라인에 `scrLock()+` 를 맨 앞에 추가하고, `screens` 배열에 `'lock'` 추가, `go('pattern')` → `go('lock')`:

```js
$('#screens').innerHTML = scrLock()+scrPattern()+scrHome()+ /* …기존 그대로… */;
const screens=['lock','pattern','home', /* …기존 그대로… */];
go('lock');
```

주의: `scrLock`이 `class="screen on"`을 갖고 기존 `scrHome`도 `on`을 가지므로, `scrHome` 템플릿의 `class="screen on"` → `class="screen"` 으로 수정한다.

(authsel 화면은 Task 4에서 추가됨 — 이 시점에서 `위젯 탭`은 콘솔 에러가 나므로 Task 4까지는 임시로 `go('home')`이 아닌 그대로 두되, 브라우저 확인은 마스킹/복사 동작까지만 본다.)

- [ ] **Step 5: SYNTAX-CHECK** — 기대: `SYNTAX-OK`

- [ ] **Step 6: 브라우저 확인** — ①첫 화면이 잠금화면+위젯 ②잔액 영역 탭 → 실제 잔액 + 링이 5초간 줄며 재마스킹 ③언마스크 중 `계좌 복사` → "복사됐어요" 토스트

- [ ] **Step 7: 커밋** — `git commit -am "feat: 잠금화면 홈 위젯 (마스킹 5초 토글 + 계좌 복사)"`

---

### Task 4: 인증 선택 화면 (지문/Face ID/패턴, 실패 없음)

**Files:**
- Modify: `카카오뱅크_솔루션.html`

- [ ] **Step 1: 인증 선택 템플릿 추가** — `scrLock()` 아래에 추가:

```js
function scrAuthsel(){return `<section class="screen" id="s-authsel">
  ${sb('9:41')}
  <div class="as-top"><button class="iconbtn" data-back="lock">${I.back}</button></div>
  <h1 class="as-title">어떻게 인증할까요?</h1>
  <p class="as-sub">한 번 선택하면 기본 방법으로 저장돼요</p>
  <div class="as-cards" id="as-cards">
    <button class="as-card" data-auth="finger">${I.finger}<span>지문</span><em class="dft">기본</em></button>
    <button class="as-card" data-auth="face"><svg viewBox="0 0 24 24" width="44" height="44" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round"><path d="M4 8V6a2 2 0 0 1 2-2h2M16 4h2a2 2 0 0 1 2 2v2M20 16v2a2 2 0 0 1-2 2h-2M8 20H6a2 2 0 0 1-2-2v-2"/><circle cx="9" cy="10" r=".8" fill="currentColor"/><circle cx="15" cy="10" r=".8" fill="currentColor"/><path d="M9 15q3 2.4 6 0"/></svg><span>Face ID</span><em class="dft">기본</em></button>
    <button class="as-card" data-auth="pattern"><svg viewBox="0 0 24 24" width="44" height="44" fill="currentColor"><circle cx="5" cy="5" r="2.2"/><circle cx="12" cy="5" r="2.2"/><circle cx="19" cy="5" r="2.2"/><circle cx="5" cy="12" r="2.2"/><circle cx="12" cy="12" r="2.2"/><circle cx="19" cy="12" r="2.2"/><circle cx="5" cy="19" r="2.2"/><circle cx="12" cy="19" r="2.2"/><circle cx="19" cy="19" r="2.2"/></svg><span>패턴</span><em class="dft">기본</em></button>
  </div>
  <div class="as-ok" id="as-ok"><div class="as-okring">${I.check}</div><b>인증 완료</b><span>안전하게 들어갈게요</span></div>
</section>`;}
```

- [ ] **Step 2: CSS 추가**

```css
#s-authsel{background:#fff}
.as-top{padding:6px 10px 0}
.as-title{padding:18px 26px 0;font-size:25px;font-weight:800;letter-spacing:-.4px}
.as-sub{padding:10px 26px 0;font-size:15px;color:#8C9298}
.as-cards{display:flex;flex-direction:column;gap:16px;padding:28px 22px}
.as-card{position:relative;display:flex;align-items:center;gap:18px;border:1.5px solid #ECEDEF;background:#FAFBFC;
  border-radius:20px;padding:20px 22px;min-height:84px;font-size:18px;font-weight:800;color:#2B2E33;
  cursor:pointer;font-family:inherit;transition:.15s}
.as-card svg{color:#5A6068}
.as-card:active{transform:scale(.985)}
.as-card.default{border-color:#FEE500;background:#FFFBE0}
.as-card .dft{position:absolute;right:18px;font-style:normal;font-size:12px;font-weight:800;color:#B7A400;
  background:#FEE500;border-radius:8px;padding:3px 9px;display:none}
.as-card.default .dft{display:block}
.as-ok{position:absolute;inset:0;background:#fff;display:none;flex-direction:column;align-items:center;
  justify-content:center;gap:10px;z-index:40}
.as-ok.on{display:flex}
.as-okring{width:96px;height:96px;border-radius:50%;background:#16C09A;display:flex;align-items:center;
  justify-content:center;animation:okpop .45s cubic-bezier(.2,1.4,.4,1)}
.as-okring svg{width:46px;height:46px;color:#fff}
@keyframes okpop{0%{transform:scale(.3);opacity:0}100%{transform:scale(1);opacity:1}}
.as-ok b{font-size:21px;font-weight:800;margin-top:8px}
.as-ok span{font-size:14.5px;color:#8C9298}
```

- [ ] **Step 3: JS 추가** — SOLUTION JS 블록에:

```js
/* ===== SOLUTION: 인증 선택 (실패 시나리오 없음) ===== */
function markAuthDefault(){
  const d=localStorage.getItem('jw-auth');
  $$('#as-cards .as-card').forEach(c=>c.classList.toggle('default',c.dataset.auth===d));
}
function entrySuccess(){
  const ok=$('#as-ok');ok.classList.add('on');haptic(30);
  setTimeout(()=>{ok.classList.remove('on');
    if(pendingGift){pendingGift=false;$('#wg-gift').classList.remove('on');openDetail('sang');}
    else go('home');
  },1000);
}
$$('#as-cards .as-card').forEach(c=>c.addEventListener('click',()=>{
  localStorage.setItem('jw-auth',c.dataset.auth);markAuthDefault();
  if(c.dataset.auth==='pattern'){go('pattern');return;}
  entrySuccess();
}));
markAuthDefault();
```

- [ ] **Step 4: 패턴 화면 성공 경로 연결** — 기존 패턴 잠금 IIFE의 `if(seq.length>=4)setTimeout(()=>go('home'),180);` 를 다음으로 교체:

```js
if(seq.length>=4)setTimeout(()=>{go('authsel');entrySuccess();},180);
```

- [ ] **Step 5: 마운트/배열 등록** — 마운트 라인에 `scrAuthsel()` 추가(`scrLock()` 다음), `screens` 배열에 `'authsel'` 추가.

- [ ] **Step 6: SYNTAX-CHECK** — 기대: `SYNTAX-OK`

- [ ] **Step 7: 브라우저 확인** — ①위젯 탭 → 인증 선택 3카드 ②지문/Face ID 탭 → 1초 체크 애니메이션 → 홈 ③패턴 탭 → 패턴 드로잉(4점+) → 체크 → 홈 ④새로고침 후 재진입 시 마지막 수단에 "기본" 배지 ⑤실패 경로가 어디에도 없음

- [ ] **Step 8: 커밋** — `git commit -am "feat: 선택형 인증 화면 (지문/FaceID/패턴, 실패 없음, 기본값 저장)"`

---

### Task 5: 홈 강화 — 간단 모드 토글 + 잔액 마스킹 + 보내기 FAB

**Files:**
- Modify: `카카오뱅크_솔루션.html`

- [ ] **Step 1: scrHome 템플릿 수정**

`<div class="h2-top"><h1>김진우</h1>` 다음(벨 아이콘 앞)에 토글 추가:

```html
<label class="smode" id="smode-sw"><span>간단 모드</span><i class="sw"></i></label>
```

`${tab2('home')}` 바로 앞에 FAB 추가:

```html
<button class="fab-send" id="fab-send"><span class="wc">₩</span> 보내기</button>
```

- [ ] **Step 2: CSS 추가**

```css
.smode{margin-left:auto;display:flex;align-items:center;gap:7px;font-size:13px;font-weight:700;color:#6E747B;cursor:pointer}
.smode .sw{width:40px;height:24px;border-radius:13px;background:#D8DBDF;position:relative;transition:.2s}
.smode .sw::after{content:'';position:absolute;top:3px;left:3px;width:18px;height:18px;border-radius:50%;background:#fff;transition:.2s;box-shadow:0 1px 3px rgba(0,0,0,.2)}
.smode.on .sw{background:#16C09A}
.smode.on .sw::after{left:19px}
.fab-send{position:absolute;right:16px;bottom:84px;z-index:25;min-height:58px;border:none;border-radius:20px;
  background:#FEE500;color:#3A2929;font-size:17px;font-weight:800;font-family:inherit;cursor:pointer;
  padding:0 24px;display:flex;align-items:center;gap:8px;box-shadow:0 8px 22px rgba(0,0,0,.18);transition:transform .12s}
.fab-send:active{transform:scale(.96)}
.fab-send .wc{font-size:19px}
.acct .amt{cursor:pointer}
.acct .amt.masked{letter-spacing:1px}
```

`.smode`가 `margin-left:auto`를 갖도록 했으므로 기존 `.h2-top`의 벨 정렬이 깨지면 `.h2-top .bell{margin-left:12px}` 를 함께 추가한다.

- [ ] **Step 3: JS 추가** — SOLUTION JS 블록에:

```js
/* ===== SOLUTION: 홈 잔액 마스킹 + FAB ===== */
const maskTimers=new Map();
function setupHomeMask(){
  $$('#s-home .acct .amt').forEach(a=>{
    if(!a.dataset.full)a.dataset.full=a.textContent;
    a.textContent='●●●,●●●원';a.classList.add('masked');
    a.addEventListener('click',e=>{
      e.stopPropagation();
      a.textContent=a.dataset.full;a.classList.remove('masked');
      clearTimeout(maskTimers.get(a));
      maskTimers.set(a,setTimeout(()=>{a.textContent='●●●,●●●원';a.classList.add('masked');},5000));
    });
  });
}
setupHomeMask();
$('#fab-send').addEventListener('click',()=>{curAcct='sang';go('recipient');});
```

주의: `finishTransfer`가 홈 카드 잔액을 직접 갱신하지 않고 `.acc2`(내계좌 화면)만 갱신하므로 충돌 없음. 단 `dataset.full`이 stale해지지 않도록 `finishTransfer` 끝에 다음 한 줄 추가:

```js
const ha=document.querySelector(`#s-home .acct[data-acct="${curAcct}"] .amt`);
if(ha){ha.dataset.full=a.bal.toLocaleString('ko-KR')+'원';if(!ha.classList.contains('masked'))ha.textContent=ha.dataset.full;}
```

- [ ] **Step 4: SYNTAX-CHECK** — 기대: `SYNTAX-OK`

- [ ] **Step 5: 브라우저 확인** — ①홈 진입 시 모든 계좌 금액 마스킹 ②금액 탭 → 5초 보였다가 재마스킹(카드 상세로 안 넘어감) ③카드의 금액 외 영역 탭 → 상세 정상 진입 ④우측 하단 노란 `보내기` FAB → 받는사람 화면 ⑤우측 상단 간단 모드 토글 UI 렌더(동작은 Task 11)

- [ ] **Step 6: 커밋** — `git commit -am "feat: 홈 강화 (잔액 마스킹 토글 + 보내기 FAB + 간단모드 토글 UI)"`

---

### Task 6: 받는사람 — 최근 상대 큰 카드 그리드

**Files:**
- Modify: `카카오뱅크_솔루션.html`

- [ ] **Step 1: scrRecipient 템플릿 수정** — body 최상단(기존 검색/리스트 위)에 추가:

```html
<div class="rc-sec">자주 보내는 사람</div>
<div class="rc-grid">
  <button class="rc-big rc-row" data-name="박지훈" data-bank="토스뱅크"><span class="ini" style="background:#2E5BFF">박</span><b>박지훈</b><span class="bk">토스뱅크</span></button>
  <button class="rc-big rc-row" data-name="이서연" data-bank="카카오뱅크"><span class="ini" style="background:#F5A623">이</span><b>이서연</b><span class="bk">카카오뱅크</span></button>
  <button class="rc-big rc-row" data-name="최준호" data-bank="국민"><span class="ini" style="background:#5F6A45">최</span><b>최준호</b><span class="bk">국민은행</span></button>
  <button class="rc-big rc-row" data-name="정다은" data-bank="신한"><span class="ini" style="background:#2B65D9">정</span><b>정다은</b><span class="bk">신한은행</span></button>
</div>
<div class="rc-sec" style="margin-top:6px">전체</div>
```

주의: 기존 `.rc-row` 클릭 바인딩(`$$('.rc-row').forEach(...)`)이 `data-name`/`data-bank`를 읽으므로 클래스 `rc-row`를 함께 부여하면 추가 JS 없이 동작한다. 바인딩은 마운트 후 실행되므로 그대로 적용된다.

- [ ] **Step 2: CSS 추가**

```css
.rc-sec{padding:14px 22px 10px;font-size:13.5px;font-weight:700;color:#8C9298}
.rc-grid{display:grid;grid-template-columns:1fr 1fr;gap:16px;padding:0 18px 8px}
.rc-big{display:flex;flex-direction:column;align-items:center;gap:6px;border:1.5px solid #ECEDEF;background:#FAFBFC;
  border-radius:20px;padding:18px 10px 16px;min-height:112px;cursor:pointer;font-family:inherit;transition:.15s}
.rc-big:active{transform:scale(.97)}
.rc-big .ini{width:44px;height:44px;border-radius:50%;color:#fff;font-size:18px;font-weight:800;
  display:flex;align-items:center;justify-content:center}
.rc-big b{font-size:16.5px;font-weight:800;color:#2B2E33}
.rc-big .bk{font-size:12.5px;color:#8C9298}
```

- [ ] **Step 3: SYNTAX-CHECK + 브라우저 확인** — 그리드 카드 탭 → 해당 이름/은행으로 금액 화면 진입. 기존 리스트도 그대로 동작.

- [ ] **Step 4: 커밋** — `git commit -am "feat: 받는사람 큰 카드 그리드 (최근 상대 4명)"`

---

### Task 7: 금액 키패드 — 64px 키 + 중복 입력 방지 + 한글 보조 표기

**Files:**
- Modify: `카카오뱅크_솔루션.html`

- [ ] **Step 1: scrAmount 템플릿 수정** — `<div class="am-hint" id="am-hint"></div>` 바로 아래에 추가:

```html
<div class="am-ko" id="am-ko"></div>
<div class="am-dup" id="am-dup">한 번만 입력됐어요</div>
```

- [ ] **Step 2: CSS 추가**

```css
#am-keypad{gap:12px;padding:8px 16px 0}
#am-keypad .am-key{height:64px;border-radius:16px}
#am-keypad .am-key:active{background:#F2F3F5}
.am-key.pulse{animation:keypulse .25s ease}
@keyframes keypulse{0%{background:#FFF6BF}100%{background:transparent}}
.am-key.shake{animation:keyshake .3s ease}
@keyframes keyshake{0%,100%{transform:translateX(0)}25%{transform:translateX(-5px)}65%{transform:translateX(5px)}}
.am-amount{font-size:48px}
.am-ko{text-align:center;margin-top:10px;font-size:18px;font-weight:700;color:#16C09A;min-height:24px}
.am-dup{text-align:center;margin-top:6px;font-size:13.5px;font-weight:700;color:#E8A000;opacity:0;transition:opacity .2s;min-height:18px}
.am-dup.on{opacity:1}
```

- [ ] **Step 3: 키 핸들러 교체** — 기존:

```js
$$('.am-key').forEach(b=>b.addEventListener('click',()=>{const k=b.dataset.k;navigator.vibrate&&navigator.vibrate(7);
  if(k==='back')amt=amt.slice(0,-1);else if(amt.length<9)amt=(amt+k).replace(/^0+(?=\d)/,'');renderAmt();}));
```

교체 (백스페이스 포함 동일 디바운스, 무시 사실을 숨기지 않고 라벨로 안내):

```js
let lastKey={k:null,t:0};
$$('.am-key').forEach(b=>b.addEventListener('click',()=>{
  const k=b.dataset.k,now=performance.now();
  if(k===lastKey.k&&now-lastKey.t<250){
    b.classList.remove('shake');void b.offsetWidth;b.classList.add('shake');
    const d=$('#am-dup');d.classList.add('on');
    clearTimeout(d._t);d._t=setTimeout(()=>d.classList.remove('on'),1100);
    lastKey.t=now;
    return;
  }
  lastKey={k,t:now};
  haptic(10);
  b.classList.remove('pulse');void b.offsetWidth;b.classList.add('pulse');
  if(k==='back')amt=amt.slice(0,-1);else if(amt.length<9)amt=(amt+k).replace(/^0+(?=\d)/,'');
  renderAmt();
}));
```

- [ ] **Step 4: renderAmt에 한글 표기 연결** — `renderAmt()` 함수 끝(빈 금액 early-return 분기 포함 양쪽)에 한글 라벨 갱신 추가. 기존 함수를 다음으로 교체:

```js
function renderAmt(){
  const d=$('#am-amt'),h=$('#am-hint'),n=$('#am-next'),ko=$('#am-ko');
  if(!amt){d.classList.add('empty');d.classList.remove('lack');d.textContent='보낼금액';h.textContent='';h.classList.remove('lack');n.classList.remove('on');ko.textContent='';return;}
  d.classList.remove('empty');
  d.innerHTML=(+amt).toLocaleString('ko-KR')+'<span class="cur"></span><span class="wn">원</span>';
  ko.textContent=koAmount(+amt);
  const lack=(+amt)>accounts[curAcct].bal;
  d.classList.toggle('lack',lack);
  h.innerHTML=lack?'출금계좌 잔고 부족 <u>예약이체</u>':'';
  h.classList.toggle('lack',lack);
  n.classList.toggle('on',!lack);
}
```

- [ ] **Step 5: SYNTAX-CHECK** — 기대: `SYNTAX-OK`

- [ ] **Step 6: 브라우저 확인** — ①키가 64px로 커짐 ②`5` 입력 후 250ms 내 `5` 재탭 → 키 흔들림 + "한 번만 입력됐어요" + 금액은 5 한 번만 ③백스페이스 연타도 동일 ④500000 입력 시 금액 아래 "오십만 원" 민트색 표기 ⑤키마다 펄스+프레임 펄스

- [ ] **Step 7: 커밋** — `git commit -am "feat: 키패드 중복입력 방지(250ms)+한글 금액 보조표기+64px 키"`

---

### Task 8: 확인 단계 — 슬라이드 투 센드

**Files:**
- Modify: `카카오뱅크_솔루션.html`

- [ ] **Step 1: cf-sheet를 프레임 레벨로 이동 + 내용 교체**

`scrMemo()` 템플릿에서 아래 블록을 **삭제**:

```html
<div class="sheet-dim" id="cf-dim"></div>
<div class="sheet" id="cf-sheet"> … </div>
```

그리고 frame 정적 HTML(`<div class="toast" id="toast"></div>` 다음)에 새 버전 **추가**:

```html
<div class="sheet-dim" id="cf-dim"></div>
<div class="sheet" id="cf-sheet">
  <div class="q1"><b id="cf-who">김진우</b>님에게 <b id="cf-amt">2,500원</b><br>이체하시겠습니까?</div>
  <div class="cf-ko" id="cf-ko"></div>
  <div class="cf-bank" id="cf-bank"></div>
  <div class="slide" id="slide">
    <div class="sl-fill" id="sl-fill"></div>
    <span class="sl-txt">밀어서 보내기 ››</span>
    <div class="sl-knob" id="sl-knob">${I.chevRight}</div>
  </div>
  <button class="cf-cancel2" id="cf-cancel">취소</button>
</div>
```

주의: 이 HTML은 정적 영역이므로 `${I.chevRight}` 템플릿 문법을 쓸 수 없다. `I.chevRight`의 SVG 문자열을 그대로 풀어서 넣는다:

```html
<div class="sl-knob" id="sl-knob"><svg viewBox="0 0 24 24" width="22" height="22" fill="none" stroke="currentColor" stroke-width="2.6" stroke-linecap="round" stroke-linejoin="round"><polyline points="9 6 15 12 9 18"/></svg></div>
```

- [ ] **Step 2: CSS 추가**

```css
#cf-dim{z-index:54}
#cf-sheet{z-index:55}
.cf-ko{text-align:center;margin-top:8px;font-size:16px;font-weight:700;color:#16C09A}
.cf-bank{text-align:center;margin-top:6px;font-size:13.5px;color:#8C9298}
.slide{position:relative;margin:22px 18px 0;height:62px;border-radius:31px;background:#F2F3F5;overflow:hidden;
  display:flex;align-items:center;justify-content:center;user-select:none;touch-action:none}
.sl-fill{position:absolute;left:0;top:0;bottom:0;width:62px;border-radius:31px;background:#FFE24A;transition:background .2s}
.slide.hot .sl-fill{background:#16C09A}
.sl-txt{position:relative;z-index:1;font-size:15px;font-weight:700;color:#9298A0;pointer-events:none}
.sl-knob{position:absolute;left:4px;top:4px;width:54px;height:54px;border-radius:50%;background:#fff;z-index:2;
  display:flex;align-items:center;justify-content:center;color:#6E747B;cursor:grab;
  box-shadow:0 2px 8px rgba(0,0,0,.16);transition:transform .25s ease}
.slide.drag .sl-knob{transition:none;cursor:grabbing}
.slide.hot .sl-knob{color:#16C09A}
.cf-cancel2{display:block;margin:14px auto 6px;border:none;background:none;font-size:14.5px;font-weight:700;
  color:#8C9298;cursor:pointer;font-family:inherit;padding:10px 18px}
```

- [ ] **Step 3: JS — openConfirm + 슬라이더** — 기존 `$('#mm-next')` 핸들러를 교체하고 SOLUTION JS 블록에 슬라이더 추가:

기존:
```js
$('#mm-next').addEventListener('click',()=>{
  $('#cf-who').textContent=$('#mm-to').value||payee.name;
  $('#cf-amt').textContent=(+amt).toLocaleString('ko-KR')+'원';
  $('#cf-dim').classList.add('on');$('#cf-sheet').classList.add('on');
});
```
교체:
```js
function openConfirm(){
  $('#cf-who').textContent=($('#s-memo').classList.contains('on')&&$('#mm-to').value)||payee.name;
  $('#cf-amt').textContent=(+amt).toLocaleString('ko-KR')+'원';
  $('#cf-ko').textContent=koAmount(+amt);
  $('#cf-bank').textContent=payee.bank+' '+payee.no;
  $('#cf-dim').classList.add('on');$('#cf-sheet').classList.add('on');
}
$('#mm-next').addEventListener('click',openConfirm);
```

기존 `$('#cf-ok')` 핸들러 블록(이체하기 버튼)은 **삭제**하고 SOLUTION JS 블록에 슬라이더 로직 추가:

```js
/* ===== SOLUTION: 슬라이드 투 센드 ===== */
(function(){
  const sl=$('#slide'),knob=$('#sl-knob'),fill=$('#sl-fill');
  let drag=false,sx=0,px=0,max=1;
  function resetSlide(){knob.style.transform='';fill.style.width='';sl.classList.remove('hot','drag');px=0;}
  knob.addEventListener('pointerdown',e=>{
    drag=true;sx=e.clientX;max=sl.clientWidth-62;sl.classList.add('drag');
    knob.setPointerCapture(e.pointerId);
  });
  knob.addEventListener('pointermove',e=>{
    if(!drag)return;
    px=Math.max(0,Math.min(max,e.clientX-sx));
    knob.style.transform=`translateX(${px}px)`;
    fill.style.width=(px+62)+'px';
    sl.classList.toggle('hot',px/max>.55);
  });
  knob.addEventListener('pointerup',()=>{
    if(!drag)return;drag=false;
    if(px/max>=.92){
      haptic(50);
      const bf=$('#burstfx');bf.classList.remove('go');void bf.offsetWidth;bf.classList.add('go');
      $('#cf-dim').classList.remove('on');$('#cf-sheet').classList.remove('on');
      resetSlide();resetAuth2();go('auth2');
    }else{
      sl.classList.remove('drag');
      resetSlide();
    }
  });
})();
```

- [ ] **Step 4: SYNTAX-CHECK** — 기대: `SYNTAX-OK`

- [ ] **Step 5: 브라우저 확인** — ①표기 화면 `다음` → 시트에 [받는 사람/금액/한글 금액/은행] + 슬라이드 바 ②중간에 놓으면 부드럽게 복귀 ③끝까지 밀면 게이지 민트색 → 강한 펄스 + 화면 확산 → 지문 인증 화면 ④`취소`로 닫기 정상

- [ ] **Step 6: 커밋** — `git commit -am "feat: 슬라이드 투 센드 (게이지+복귀+완료 햅틱 50ms+화면 확산)"`

---

### Task 9: 완료 화면 — 확신의 절정

**Files:**
- Modify: `카카오뱅크_솔루션.html`

- [ ] **Step 1: scrDone2 템플릿 수정**

`.dn-msg` 블록을 다음으로 교체:

```html
<div class="dn-msg">김진우님,<br><span id="dn-who">김진우</span>님께 <em id="dn-amt">2,500원</em>을<br><b class="dn-exact">정확히</b> 보냈어요</div>
<div class="dn-card">
  <div class="row"><span>보낸 시각</span><b id="dn-time">—</b></div>
  <div class="row"><span>거래 상태</span><b class="ok">완료 ✓</b></div>
</div>
```

하단 버튼 영역 `.dn-btns` 를 다음으로 교체 (우측 하단 정렬, `영수증 보기`가 `홈으로` 위):

```html
<div class="dn-btns2">
  <button class="receipt" id="dn-receipt">영수증 보기</button>
  <button class="ok" id="dn-ok">홈으로</button>
</div>
```

- [ ] **Step 2: CSS 추가**

```css
.dn-exact{color:#16C09A}
.dn-card{margin:26px 24px 0;background:#F7F8FA;border-radius:18px;padding:18px 20px;display:flex;flex-direction:column;gap:12px}
.dn-card .row{display:flex;justify-content:space-between;font-size:15px}
.dn-card .row span{color:#8C9298}
.dn-card .row b{font-weight:800}
.dn-card .row b.ok{color:#16C09A}
.dn-btns2{margin-top:auto;display:flex;flex-direction:column;align-items:flex-end;gap:12px;padding:0 16px 26px}
.dn-btns2 .receipt{border:1.5px solid #ECEDEF;background:#fff;border-radius:16px;min-height:50px;padding:0 22px;
  font-size:15px;font-weight:700;color:#46494E;cursor:pointer;font-family:inherit}
.dn-btns2 .ok{border:none;background:#FEE500;color:#3A2929;border-radius:18px;min-height:58px;min-width:160px;
  padding:0 30px;font-size:17px;font-weight:800;cursor:pointer;font-family:inherit}
.dn-check{animation:okpop .5s cubic-bezier(.2,1.4,.4,1)}
```

기존 `.dn-btns` 관련 CSS는 그대로 두어도 무방(사용처 없음). `.dn-ad`(광고 배너)가 `.dn-btns2`와 겹치면 scrDone2 템플릿에서 `.dn-ad` 블록을 삭제한다.

- [ ] **Step 3: finishTransfer 수정** — 기존 함수를 교체:

```js
function finishTransfer(){
  const a=accounts[curAcct],v=+amt;
  a.bal-=v;
  const t=new Date(),p=x=>String(x).padStart(2,'0');
  const timeStr=`오늘 ${p(t.getHours())}:${p(t.getMinutes())}:${p(t.getSeconds())}`;
  a.tx.unshift({dt:'06.12',nm:($('#mm-to').value||payee.name),memo:'#이체',v:'-'+v.toLocaleString('ko-KR')+'원',bal:a.bal.toLocaleString('ko-KR')+'원'});
  const card=document.querySelector(`.acc2[data-acct="${curAcct}"] .amt`);
  if(card)card.textContent=a.bal.toLocaleString('ko-KR')+'원';
  const ha=document.querySelector(`#s-home .acct[data-acct="${curAcct}"] .amt`);
  if(ha){ha.dataset.full=a.bal.toLocaleString('ko-KR')+'원';if(!ha.classList.contains('masked'))ha.textContent=ha.dataset.full;}
  $('#dn-who').textContent=($('#mm-to').value||payee.name);
  $('#dn-amt').textContent=v.toLocaleString('ko-KR')+'원';
  $('#dn-time').textContent=timeStr;
  go('done2');haptic(40);
}
```

(주의: Task 5에서 추가한 홈 카드 잔액 갱신 두 줄이 이 교체본에 이미 포함되어 있다 — 중복 추가하지 말 것.)

- [ ] **Step 4: 영수증 버튼 동작** — SOLUTION JS 블록에:

```js
$('#dn-receipt').addEventListener('click',()=>toast('영수증이 저장됐어요 (데모)'));
```

- [ ] **Step 5: SYNTAX-CHECK + 브라우저 확인** — 송금 완주 시 ①체크 팝 애니메이션+진동 ②"김진우님, ○○님께 ○원을 **정확히** 보냈어요" ③보낸 시각이 초 단위 실시간 ④상태 "완료 ✓" ⑤우측 하단 `홈으로`, 그 위 `영수증 보기`

- [ ] **Step 6: 커밋** — `git commit -am "feat: 완료 화면 강화 (정확히 보냈어요+초단위 시각+우측하단 버튼)"`

---

### Task 10: 거래내역 카드 스택 + 볼륨 버튼 시뮬레이션

**Files:**
- Modify: `카카오뱅크_솔루션.html`

- [ ] **Step 1: 카드 스택 화면 추가** — `scrAuthsel()` 아래:

```js
function scrTxstack(){return `<section class="screen" id="s-txstack">
  ${sb('11:36')}
  <div class="ts-top"><button class="iconbtn" id="ts-back" data-back="detail">${I.back}</button><h1>한 건씩 확인</h1></div>
  <div class="ts-stage" id="ts-stage"></div>
  <div class="ts-pos" id="ts-pos"></div>
  <div class="ts-hint">프레임 좌측 볼륨 버튼 또는 키보드 ↑↓로 넘겨보세요 — 터치 없이 재확인</div>
  <div class="homebar"></div>
</section>`;}
```

- [ ] **Step 2: CSS 추가**

```css
#s-txstack{background:#F2F4F6}
.ts-top{display:flex;align-items:center;gap:4px;padding:6px 10px 0}
.ts-top h1{font-size:19px;font-weight:800}
.ts-stage{flex:1;display:flex;align-items:center;justify-content:center;padding:0 26px;position:relative}
.ts-card{width:100%;background:#fff;border-radius:26px;padding:30px 26px;box-shadow:0 12px 34px rgba(0,0,0,.10);
  display:flex;flex-direction:column;gap:14px;animation:tsin .28s ease}
@keyframes tsin{from{opacity:0;transform:translateY(14px) scale(.97)}to{opacity:1;transform:none}}
.ts-card .who{font-size:23px;font-weight:800}
.ts-card .v{font-size:34px;font-weight:800;letter-spacing:-.5px}
.ts-card .v.plus{color:#4C7DF0}
.ts-card .meta{display:flex;flex-direction:column;gap:8px;font-size:14.5px;color:#8C9298;margin-top:6px}
.ts-card .meta b{color:#16C09A;font-weight:800}
.ts-pos{text-align:center;font-size:14px;font-weight:700;color:#8C9298;padding:6px 0 4px}
.ts-hint{text-align:center;font-size:12.5px;color:#B0B5BB;padding:0 30px 18px;line-height:1.5}
```

- [ ] **Step 3: 상세 화면에 진입 버튼** — `scrDetail()` 템플릿의 `<div id="tx-list">` 바로 위에 추가:

```html
<button class="ts-open" id="ts-open">카드로 한 건씩 확인 ›</button>
```

CSS:

```css
.ts-open{display:block;margin:0 18px 12px;width:calc(100% - 36px);min-height:50px;border:none;border-radius:16px;
  background:rgba(255,255,255,.6);font-size:15px;font-weight:800;color:#2B2E33;cursor:pointer;font-family:inherit}
```

- [ ] **Step 4: JS** — SOLUTION JS 블록에:

```js
/* ===== SOLUTION: 거래 카드 스택 + 볼륨 버튼 ===== */
let tsIdx=0,tsList=[];
function tsRender(){
  const t=tsList[tsIdx];if(!t)return;
  $('#ts-stage').innerHTML=`<div class="ts-card">
    <div class="who">${t.nm}</div>
    <div class="v ${t.plus?'plus':''}">${t.v}</div>
    <div class="meta">
      <span>${t.dt} · 잔액 ${t.bal}</span>
      <span>상태 <b>완료 ✓</b></span>
    </div></div>`;
  $('#ts-pos').textContent=`${tsIdx+1} / ${tsList.length}`;
}
function tsOpen(list,backTo){
  tsList=list;tsIdx=0;
  $('#ts-back').dataset.back=backTo;
  tsRender();go('txstack');
}
function tsMove(d){
  if(!$('#s-txstack').classList.contains('on'))return;
  const n=tsIdx+d;
  if(n<0||n>=tsList.length){haptic(8);return;}
  tsIdx=n;haptic(12);tsRender();
}
document.addEventListener('click',e=>{if(e.target.closest('#ts-open'))tsOpen(curTx,'detail');});
$('#vk-up').addEventListener('click',()=>tsMove(-1));
$('#vk-dn').addEventListener('click',()=>tsMove(1));
document.addEventListener('keydown',e=>{
  if(e.key==='ArrowUp'){tsMove(-1);e.preventDefault();}
  if(e.key==='ArrowDown'){tsMove(1);e.preventDefault();}
});
```

주의: `#ts-open`은 화면 재마운트가 없으므로 위임 바인딩(document)으로 처리했다.

- [ ] **Step 5: 마운트/배열 등록** — 마운트 라인에 `scrTxstack()` 추가, `screens` 배열에 `'txstack'` 추가.

- [ ] **Step 6: SYNTAX-CHECK + 브라우저 확인** — ①계좌 상세 → `카드로 한 건씩 확인` → 카드 스택 ②볼륨키 클릭/↑↓ 키로 한 건씩 이동(애니메이션+햅틱) ③끝에서 더 누르면 약한 펄스만 ④뒤로가기 → 상세 복귀

- [ ] **Step 7: 커밋** — `git commit -am "feat: 거래 카드 스택 + 볼륨 버튼 시뮬레이션 (터치 없는 재확인)"`

---

### Task 11: 간단 모드

**Files:**
- Modify: `카카오뱅크_솔루션.html`

- [ ] **Step 1: 화면 2개 추가** — `scrTxstack()` 아래:

```js
function scrSimple(){return `<section class="screen" id="s-simple">
  ${sb('11:35')}
  <div class="sm-top"><h1>간단 모드</h1>
    <label class="smode on" id="smode-sw2"><span>간단 모드</span><i class="sw"></i></label></div>
  <div class="sm-btns">
    <button class="sm-btn" id="sm-send"><span class="big">₩</span>보내기</button>
    <button class="sm-btn" id="sm-bal"><span class="big">👁</span>잔액</button>
    <button class="sm-btn" id="sm-tx"><span class="big">✓</span>거래내역</button>
  </div>
  <div class="homebar"></div></section>`;}
function scrSmbal(){return `<section class="screen" id="s-smbal">
  ${sb('11:35')}
  <div class="ts-top"><button class="iconbtn" data-back="simple">${I.back}</button><h1>잔액</h1></div>
  <div class="smb-wrap">
    <div class="smb-card" id="smb-card">
      <div class="nm">김진우의 통장</div>
      <div class="amt" id="smb-amt">●●●,●●●원</div>
      <div class="hint">탭하면 5초간 보여요</div>
    </div>
  </div>
  <div class="homebar"></div></section>`;}
```

- [ ] **Step 2: CSS 추가**

```css
#s-simple{background:#fff}
.sm-top{display:flex;align-items:center;padding:14px 22px 0}
.sm-top h1{font-size:25px;font-weight:800}
.sm-top .smode{margin-left:auto}
.sm-btns{flex:1;display:flex;flex-direction:column;gap:18px;padding:26px 20px 90px}
.sm-btn{flex:1;border:1.5px solid #ECEDEF;border-radius:26px;background:#FAFBFC;font-family:inherit;cursor:pointer;
  font-size:24px;font-weight:800;color:#2B2E33;display:flex;flex-direction:column;align-items:center;
  justify-content:center;gap:10px;transition:.15s;animation:smin .3s ease backwards}
.sm-btn:nth-child(2){animation-delay:.06s}.sm-btn:nth-child(3){animation-delay:.12s}
@keyframes smin{from{opacity:0;transform:translateY(16px)}to{opacity:1;transform:none}}
.sm-btn:active{transform:scale(.97);background:#FFFBE0;border-color:#FEE500}
.sm-btn .big{font-size:34px}
#s-smbal{background:#F2F4F6}
.smb-wrap{flex:1;display:flex;align-items:center;padding:0 22px}
.smb-card{width:100%;background:#fff;border-radius:26px;padding:38px 28px;box-shadow:0 12px 34px rgba(0,0,0,.10);cursor:pointer;text-align:center}
.smb-card .nm{font-size:16px;font-weight:700;color:#8C9298}
.smb-card .amt{font-size:40px;font-weight:800;margin-top:14px;letter-spacing:-.5px}
.smb-card .hint{font-size:13px;color:#B0B5BB;margin-top:16px}
body.smode-on .rc-list-rest{display:none}
```

(`.rc-list-rest`: Step 4에서 받는사람 화면의 "전체" 섹션을 감싸는 클래스)

- [ ] **Step 3: JS** — SOLUTION JS 블록에:

```js
/* ===== SOLUTION: 간단 모드 ===== */
let smode=false;
function setSmode(v){
  smode=v;document.body.classList.toggle('smode-on',v);
  $('#smode-sw').classList.toggle('on',v);
  go(v?'simple':'home');
}
$('#smode-sw').addEventListener('click',()=>setSmode(true));
$('#smode-sw2').addEventListener('click',()=>setSmode(false));
$('#sm-send').addEventListener('click',()=>{curAcct='sang';go('recipient');});
$('#sm-bal').addEventListener('click',()=>{$('#smb-amt').textContent='●●●,●●●원';go('smbal');});
$('#sm-tx').addEventListener('click',()=>tsOpen(accounts.sang.tx,'simple'));
let smbT=null;
$('#smb-card').addEventListener('click',()=>{
  $('#smb-amt').textContent=accounts.sang.bal.toLocaleString('ko-KR')+'원';
  clearTimeout(smbT);smbT=setTimeout(()=>{$('#smb-amt').textContent='●●●,●●●원';},5000);
});
```

- [ ] **Step 4: 간단 모드용 플로우 압축**

scrRecipient에서 큰 그리드 아래 "전체" 섹션부터 기존 리스트 전체를 `<div class="rc-list-rest"> … </div>` 로 감싼다 (간단 모드에서 숨김 → 최근 4명 + 검색만).

`$('#am-next')` 핸들러 시작부에 간단 모드 분기 추가 — 기존:

```js
$('#am-next').addEventListener('click',()=>{
  if(!$('#am-next').classList.contains('on'))return;
```
교체:
```js
$('#am-next').addEventListener('click',()=>{
  if(!$('#am-next').classList.contains('on'))return;
  if(smode){openConfirm();return;}   // 간단 모드: 표기 화면 생략, 바로 슬라이드
```

받는사람 화면 뒤로가기/취소가 간단 모드에서 simple로 가도록, `go()` 직전 분기는 두지 않고 `data-back="home"`인 요소를 그대로 두되 `go` 함수를 교체:

```js
function go(n){
  if(smode&&n==='home')n='simple';
  screens.forEach(s=>$('#s-'+s)&&$('#s-'+s).classList.remove('on'));
  $('#s-'+n).classList.add('on');
}
```

done 화면 `홈으로`(#dn-ok)는 기존 핸들러가 `go('home')`을 호출하므로 위 `go` 교체로 자동 처리된다.

- [ ] **Step 5: 마운트/배열 등록** — `scrSimple()+scrSmbal()` 마운트, `screens`에 `'simple','smbal'` 추가.

- [ ] **Step 6: SYNTAX-CHECK + 브라우저 확인** — ①홈 토글 ON → 3개 초대형 버튼이 순차 등장 ②보내기 → 최근 4명 카드만(리스트 숨김) → 금액 → `다음` → 바로 슬라이드 시트(표기 화면 생략) → 인증 → 완료 → `홈으로` → 간단 모드 홈 ③잔액 → 카드 탭 5초 보기 ④거래내역 → 카드 스택(뒤로가기 → 간단 모드) ⑤토글 OFF → 일반 홈 ⑥자동 진입 경로 없음

- [ ] **Step 7: 커밋** — `git commit -am "feat: 간단 모드 (3버튼 레이아웃+압축 송금 플로우, 수동 토글 전용)"`

---

### Task 12: 데모 패널 동작 + 위젯 선물상자

**Files:**
- Modify: `카카오뱅크_솔루션.html`

- [ ] **Step 1: JS** — SOLUTION JS 블록에:

```js
/* ===== SOLUTION: 데모 패널 ===== */
$('#demo-deposit').addEventListener('click',()=>{
  const a=accounts.sang;a.bal+=50000;
  a.tx.unshift({dt:'06.12',nm:'이서연',memo:'#입금',v:'+50,000원',plus:1,bal:a.bal.toLocaleString('ko-KR')+'원'});
  pendingGift=true;
  $('#wg-gift').classList.add('on');
  wgRender(false);
  const ha=document.querySelector('#s-home .acct[data-acct="sang"] .amt');
  if(ha)ha.dataset.full=a.bal.toLocaleString('ko-KR')+'원';
  if(!$('#s-lock').classList.contains('on'))toast('입금 50,000원 · 이서연 💛');
  haptic(20);
});
$('#demo-dup').addEventListener('click',()=>{
  if(!$('#s-amount').classList.contains('on')){toast('금액 입력 화면에서 눌러주세요');return;}
  const key=document.querySelector('#am-keypad .am-key[data-k="5"]');
  key.click();
  setTimeout(()=>key.click(),120);
});
```

- [ ] **Step 2: SYNTAX-CHECK + 브라우저 확인** — ①잠금화면에서 `입금 발생시키기` → 위젯 🎁 흔들림, 위젯 탭 → 인증 → 계좌 상세(입금 내역 최상단) ②앱 화면에서 누르면 토스트 ③금액 화면에서 `중복 입력 시뮬레이션` → 5 한 번만 입력 + 흔들림 + 라벨 ④다른 화면에서 누르면 안내 토스트

- [ ] **Step 3: 커밋** — `git commit -am "feat: 데모 패널 (입금 트리거+중복입력 시뮬레이션)"`

---

### Task 13: 최종 DoD 검증

- [ ] **Step 1: SYNTAX-CHECK** — 기대: `SYNTAX-OK`

- [ ] **Step 2: 전체 시연 동선 수동 통과** (스펙 DoD):
  1. 잠금 위젯(마스킹 토글·5초 링·계좌 복사) → 인증 선택 → 홈
  2. FAB 보내기 → 큰 카드 그리드 → 키패드(중복 방지+한글 표기) → 표기 → 슬라이드 투 센드 → 지문 인증 → 완료 화면(초 단위) → 홈
  3. 계좌 상세 → 카드 스택 → 볼륨키/↑↓ 넘김
  4. 간단 모드 ON → 3버튼 → 압축 송금 완주 → OFF
  5. 데모 패널 2버튼 + 햅틱 프레임 펄스
  6. 브라우저 창을 390px 폭으로 줄여 모바일 뷰포트 깨짐 확인
  7. 기존 클론 화면(혜택/투자/AI/전체메뉴/알림) 진입·복귀 정상

- [ ] **Step 3: 콘솔 에러 0 확인** — 개발자도구 콘솔에 에러 없을 것

- [ ] **Step 4: 발견된 문제 수정 후 최종 커밋** — `git commit -am "fix: 최종 시연 동선 점검 수정"` (수정이 있을 때만)

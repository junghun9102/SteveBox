# 다이어그램 / 시각 자료 가이드

리포트가 구조, 의존 관계, 데이터 흐름, 아키텍처처럼 **"관계"를 설명해야 하는 내용**을 담고 있다면, 문장으로 풀어 쓰지 말고 다이어그램을 그리세요. 사람이 관계를 파악하는 속도는 표/글보다 그림이 압도적으로 빠릅니다. 반대로 회고, 공지, 단순 진행 상황처럼 관계가 없는 내용에 억지로 다이어그램을 넣을 필요는 없습니다.

이 리포트는 Claude Artifact가 아니라 GitHub Pages로 배포되는 정적 HTML이므로 CDN 스크립트(cytoscape, mermaid, chart.js 등)를 자유롭게 `<script src="https://...">`로 불러와도 됩니다. CSP 제약이 없습니다.

## 언제 뭘 쓸지

| 설명하려는 것 | 방법 |
|---|---|
| 고정된 구조/관계 (모듈 의존, 아키텍처, 시스템 구성도) — 노드 10~20개 이하, 한 번 보고 끝 | **정적 인라인 SVG** 직접 그리기 |
| 노드/엣지가 많아서(수십 개) 필터링·검색·클릭 탐색이 필요한 그래프 | **Cytoscape.js + dagre** (CDN) 인터랙티브 그래프 |
| 순서가 있는 흐름, 시퀀스, 플로우차트 | Mermaid(`flowchart`, `sequenceDiagram`) CDN, 또는 노드 수가 적으면 정적 SVG |
| 일정/타임라인 | 표로 충분한 경우가 많음. 단계가 겹치거나 병렬로 진행되면 간단한 가로 바 형태 SVG/CSS 타임라인 |
| 개별 사실/스펙 나열 (API 목록, 필드 목록) | 표. 다이어그램으로 억지로 그리지 말 것 |

## 정적 SVG 다이어그램

가장 자주 쓰게 될 방법입니다. `<svg>`를 인라인으로 문서에 직접 넣고, `viewBox`로 좌표계를 잡은 뒤 박스(`rect`)와 화살표(`path`)를 좌표 계산으로 배치합니다. 스크린샷이나 외부 이미지 대신 이 방식을 쓰는 이유:

- 다크모드에서 색이 깨지지 않도록 CSS 변수(`var(--accent)` 등)를 `fill`/`stroke`에 그대로 쓸 수 있음
- 텍스트가 실제 텍스트라서 리포트 안에서 검색/선택 가능
- 나중에 좌표만 조정하면 되므로 유지보수가 쉬움

최소 예시 (박스 2개 + 화살표):

```html
<div class="card scroll">
<svg viewBox="0 0 480 140" width="100%" style="min-width:420px">
  <style>
    .node-label { font: 600 13px -apple-system, sans-serif; fill: var(--fg); }
    .node-box { fill: var(--accent-soft); stroke: var(--accent); stroke-width: 1.5; }
  </style>
  <rect class="node-box" x="20" y="50" width="140" height="40" rx="8"/>
  <text class="node-label" x="90" y="75" text-anchor="middle">모듈 A</text>

  <rect class="node-box" x="320" y="50" width="140" height="40" rx="8"/>
  <text class="node-label" x="390" y="75" text-anchor="middle">모듈 B</text>

  <path d="M 160 70 L 320 70" fill="none" stroke="var(--muted)" stroke-width="1.5"
        marker-end="url(#arrow)"/>
  <defs>
    <marker id="arrow" markerWidth="8" markerHeight="8" refX="6" refY="4" orient="auto">
      <path d="M0,0 L8,4 L0,8 z" fill="var(--muted)"/>
    </marker>
  </defs>
</svg>
<p class="note">범례: 실선 = 컴파일 의존. 노드가 많아지면 열(column) 단위로 나눠 배치하면 정리됩니다.</p>
</div>
```

- 노드가 한 줄에 다 안 들어가면 `.card.scroll`로 감싸서 가로 스크롤 처리 (`style.css`에 이미 정의됨)
- 노드/엣지가 많을 때는 JS로 좌표를 계산해서 `createElementNS`로 동적 생성하는 것이 손으로 좌표를 다 쓰는 것보다 정확합니다 (열 단위로 배치 → y좌표는 `index * gap`으로 계산)
- 범례는 다이어그램 바로 아래 `<p class="note">` 또는 색상 스와치(`<span>` + `background-color`)로

## 인터랙티브 그래프 (Cytoscape.js)

의존성 그래프처럼 노드가 20개를 넘고, 클릭해서 이웃 노드를 강조하거나 검색으로 필터링하는 게 유용한 경우에 씁니다.

```html
<script src="https://unpkg.com/cytoscape@3.30.2/dist/cytoscape.min.js"></script>
<script src="https://unpkg.com/dagre@0.8.5/dist/dagre.min.js"></script>
<script src="https://unpkg.com/cytoscape-dagre@2.5.0/cytoscape-dagre.js"></script>

<div class="card" style="padding:0; overflow:hidden;">
  <div id="cy" style="width:100%; height:600px; background:#0b1020;"></div>
</div>
<script>
  const cy = cytoscape({
    container: document.getElementById('cy'),
    elements: {
      nodes: [{data:{id:'a', label:'모듈 A'}}, {data:{id:'b', label:'모듈 B'}}],
      edges: [{data:{source:'a', target:'b'}}],
    },
    layout: {name:'dagre', rankDir:'LR'},
    style: [
      {selector:'node', style:{label:'data(label)', 'background-color':'#3b82f6', color:'#e5e7eb',
        'text-valign':'center', 'text-halign':'center', shape:'round-rectangle', padding:'8px'}},
      {selector:'edge', style:{'curve-style':'bezier', 'target-arrow-shape':'triangle',
        'line-color':'#475569', 'target-arrow-color':'#475569'}},
    ],
  });
  cy.on('tap', 'node', evt => {
    const n = evt.target, hood = n.closedNeighborhood();
    cy.elements().not(hood).style('opacity', 0.15);
    hood.style('opacity', 1);
  });
</script>
```

- 그래프는 다크 배경(`#0b1020` 등)을 고정으로 써도 됩니다 — 그래프 캔버스는 문서 전체 라이트/다크와 분리된 별도의 시각 영역으로 취급해도 자연스럽습니다 (레퍼런스 리포트도 이 방식).
- 데이터가 이미 코드/설정 파일에 있다면(예: `settings.gradle`, `package.json`) 그 내용을 파싱해서 `nodes`/`edges` 배열을 실제 데이터로 채우세요. 손으로 나열하지 말고 실제 관계를 반영해야 신뢰할 수 있는 리포트가 됩니다.

## 자잘하지만 효과 큰 것들

- **pill**: 모듈명, 상태값처럼 짧은 라벨을 여러 개 나열할 때 `<span class="pill">이름</span>`. 비활성/삭제 대상은 `pill dead`.
- **card**: 다이어그램, 코드 블록, 범례처럼 "이건 본문과 시각적으로 분리된 덩어리다" 싶은 내용은 `<div class="card">`로 감싸세요.
- **note**: 다이어그램/표 바로 아래 붙는 해설. `<p class="note">`.
- 표 안에 코드가 들어가면 자동으로 줄바꿈되지 않게 처리되어 있습니다 (`td code`).

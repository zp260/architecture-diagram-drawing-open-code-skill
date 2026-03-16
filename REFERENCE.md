This document contains advanced drawio-to-svg file processing features, detailed examples, and additional libraries not covered in the main skill instructions.
~~完整的代码样例参考,当前文件目录下的./scripts/convert.js~~

## JavaScript Libraries
- 当前系统环境需要具备node v24版本，如果当前系统不具备node以及需要的版本，询问并确认是否要安装。
- node的modules需要安装有sax、xml2js、xmlbuilder等node框架。缺少框架请使用npm install安装它们。

### draw-to-cvg-demo.js
```js
const fs = require('fs');
const xml2js = require('xml2js');

const parser = new xml2js.Parser();

const content = fs.readFileSync('{filename}.drawio', 'utf-8');

parser.parseString(content, (err, result) => {
  if (err) {
    console.error('Parse error:', err);
    process.exit(1);
  }

  const diagram = result.mxfile.diagram[0];
  const graphModel = diagram.mxGraphModel[0];
  const root = graphModel.root[0];
  const cells = root.mxCell || [];

  const width = 1600;
  const height = 750;
  
  let svgElements = [];
  const parentPositions = {};

  cells.forEach(cell => {
    const $ = cell.$ || {};
    const style = $.style || '';
    let value = ($.value || '').replace(/&#xa;/g, '\n');
    const id = $.id;
    const parentId = $.parent;

    if (id === '0' || id === '1') return;

    const geo = cell.mxGeometry ? cell.mxGeometry[0].$ : {};
    let x = parseFloat(geo.x || 0);
    let y = parseFloat(geo.y || 0);
    const w = parseFloat(geo.width || 0);
    const h = parseFloat(geo.height || 0);

    if (w === 0 || h === 0) return;

    if (parentId && parentPositions[parentId]) {
      x += parentPositions[parentId].x;
      y += parentPositions[parentId].y;
    }

    const styleObj = {};
    style.split(';').forEach(s => {
      const [k, v] = s.split('=');
      if (k && v) styleObj[k.trim()] = v.trim();
    });

    const fill = styleObj.fillColor || '#ffffff';
    const stroke = styleObj.strokeColor || '#000000';
    const fontColor = styleObj.fontColor || '#333333';
    const fontSize = parseInt(styleObj.fontSize) || 12;
    const isBold = styleObj.fontStyle === '1';
    const fontWeight = isBold ? 'bold' : 'normal';
    
    const isSwimlane = style.includes('swimlane');
    const isText = style.includes('text;') || (styleObj.strokeColor === 'none' && styleObj.fillColor === 'none');
    const isEdge = style.includes('edge=1') || style.includes('endArrow');

    if (isSwimlane) {
      parentPositions[id] = { x, y };
    }

    if (isEdge) {
      if (cell.mxGeometry && cell.mxGeometry[0].mxPoint) {
        const points = cell.mxGeometry[0].mxPoint;
        if (points.length >= 2) {
          const p1 = points[0].$;
          const p2 = points[1].$;
          const strokeClr = styleObj.strokeColor || '#666666';
          svgElements.push({
            type: 'edge',
            y: Math.min(parseFloat(p1.y), parseFloat(p2.y)),
            svg: `  <line x1="${p1.x}" y1="${p1.y}" x2="${p2.x}" y2="${p2.y}" stroke="${strokeClr}" stroke-width="2" marker-end="url(#arrow)"/>\n`
          });
        }
      }
      return;
    }

    if (isText) {
      svgElements.push({
        type: 'text',
        y: y,
        svg: `  <text x="${x}" y="${y + fontSize}" font-family="Arial, sans-serif" font-size="${fontSize}" font-weight="${fontWeight}" fill="${fontColor}">${escapeXml(value)}</text>\n`
      });
      return;
    }

    if (isSwimlane) {
      const startSize = parseInt(styleObj.startSize) || 40;
      const headerFill = fill;
      const bodyFill = lightenColor(fill, 30);
      
      svgElements.push({
        type: 'swimlane',
        y: y,
        svg: `  <g>
    <rect x="${x}" y="${y}" width="${w}" height="${startSize}" fill="${headerFill}" stroke="${stroke}" stroke-width="2"/>
    <rect x="${x}" y="${y + startSize}" width="${w}" height="${h - startSize}" fill="${bodyFill}" stroke="${stroke}" stroke-width="2"/>
    <text x="${x + 15}" y="${y + startSize/2 + 5}" font-family="Arial, sans-serif" font-size="${fontSize}" font-weight="${fontWeight}" fill="${fontColor}">${escapeXml(value)}</text>
  </g>\n`
      });
      return;
    }

    const lines = value.split('\n');
    let textSvg = '';
    
    if (lines.length === 1) {
      textSvg = `    <text x="${x + w/2}" y="${y + h/2 + 4}" text-anchor="middle" font-family="Arial, sans-serif" font-size="${fontSize}" fill="${fontColor}">${escapeXml(value)}</text>\n`;
    } else {
      const lineHeight = fontSize + 4;
      const startY = y + h/2 - (lines.length - 1) * lineHeight / 2 + fontSize/2;
      lines.forEach((line, i) => {
        textSvg += `    <text x="${x + w/2}" y="${startY + i * lineHeight}" text-anchor="middle" font-family="Arial, sans-serif" font-size="${fontSize}" fill="${fontColor}">${escapeXml(line)}</text>\n`;
      });
    }

    svgElements.push({
      type: 'rect',
      y: y,
      svg: `  <g>
    <rect x="${x}" y="${y}" width="${w}" height="${h}" rx="8" fill="${fill}" stroke="${stroke}" stroke-width="1.5"/>
${textSvg}  </g>\n`
    });
  });

  svgElements.sort((a, b) => a.y - b.y);

  const svg = `<?xml version="1.0" encoding="UTF-8"?>
<svg xmlns="http://www.w3.org/2000/svg" width="${width}" height="${height}" viewBox="0 0 ${width} ${height}">
  <defs>
    <marker id="arrow" markerWidth="10" markerHeight="10" refX="9" refY="3" orient="auto" markerUnits="strokeWidth">
      <path d="M0,0 L0,6 L9,3 z" fill="#666666"/>
    </marker>
  </defs>
  <rect width="100%" height="100%" fill="#fafafa"/>
  
${svgElements.map(e => e.svg).join('')}
</svg>`;

  fs.writeFileSync('{fileSaveName}.svg', svg);
  console.log('✓ SVG 文件已保存到: {fileSaveName}.svg');
  console.log(`  - 处理了 ${svgElements.length} 个元素`);
});

function escapeXml(str) {
  if (!str) return '';
  return str
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;')
    .replace(/"/g, '&quot;')
    .replace(/'/g, '&apos;');
}

function lightenColor(hex, percent) {
  // Lighten a hex color
  const num = parseInt(hex.replace('#', ''), 16);
  const amt = Math.round(2.55 * percent);
  const R = (num >> 16) + amt;
  const G = (num >> 8 & 0x00FF) + amt;
  const B = (num & 0x0000FF) + amt;
  return '#' + (0x1000000 + 
    (R < 255 ? R < 1 ? 0 : R : 255) * 0x10000 + 
    (G < 255 ? G < 1 ? 0 : G : 255) * 0x100 + 
    (B < 255 ? B < 1 ? 0 : B : 255)
  ).toString(16).slice(1);
}
```

## how to use
- run  node draw-to-cvg-demo.js 
- then out put "✓ SVG 文件已保存到: 系统架构图.svg .处理了 41 个元素"
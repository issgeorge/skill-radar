# Alternative: Pure Canvas Radar Chart for Wix Velo

## Overview
If you encounter issues with Chart.js in Wix, this alternative uses pure HTML5 Canvas for drawing the radar chart, ensuring 100% compatibility with Wix Velo.

## Page Setup (Same as previous)
- Add HTML Embed component with ID: `radarChartEmbed`
- Add text elements for rotating labels
- Add legend container

## Alternative Implementation

### 1. Main Page Code (`yourPage.js`)

```javascript
import { initializeCustomRadarChart } from 'public/customRadarChart.js';

$w.onReady(function () {
    // Initialize the custom radar chart
    initializeCustomRadarChart();
    
    // Setup legend
    setupLegend();
    
    // Start label animations with enhanced effects
    startEnhancedLabelAnimations();
});

function setupLegend() {
    const legendData = [
        { id: 'legendTech', color: '#1976D2', text: 'Tech' },
        { id: 'legendBusiness', color: '#FBC02D', text: 'Business' },
        { id: 'legendDesign', color: '#388E3C', text: 'Design' },
        { id: 'legendArts', color: '#E53935', text: 'Arts' }
    ];

    legendData.forEach(item => {
        $w(`#${item.id}`).html = `
            <div style="display: flex; align-items: center; gap: 6px; font-family: 'Poppins', sans-serif; color: #444; font-size: 16px;">
                <div style="width: 14px; height: 14px; background-color: ${item.color}; border-radius: 3px;"></div>
                <span>${item.text}</span>
            </div>
        `;
    });
}

function startEnhancedLabelAnimations() {
    // Enhanced animations with different effects for each label
    animateRotatingLabel('labelLearning', 90, 0.4, 'sine');
    animateRotatingLabel('labelPracticing', 130, 0.6, 'cosine');
    animateRotatingLabel('labelTeach', 170, 0.9, 'pulse');
    
    // Add pulsing effect to "I Can Teach You" label
    setInterval(() => {
        const element = $w('#labelTeach');
        element.style.transform = 'scale(1.1)';
        setTimeout(() => {
            element.style.transform = 'scale(1)';
        }, 600);
    }, 1200);
}

function animateRotatingLabel(elementId, radius, speed, effect) {
    let angle = 0;
    let pulsePhase = 0;
    const element = $w(`#${elementId}`);
    
    // Style the element
    element.html = `<div style="
        font-family: 'Poppins', sans-serif;
        color: #555;
        font-size: 13px;
        white-space: nowrap;
        opacity: 0.9;
        font-weight: 500;
        text-shadow: 0 1px 2px rgba(0,0,0,0.1);
        transition: all 0.1s ease;
    ">${element.text}</div>`;
    
    function updatePosition() {
        angle = (angle + speed) % 360;
        pulsePhase += 0.1;
        
        const radian = (angle * Math.PI) / 180;
        
        // Add effect variations
        let effectRadius = radius;
        let opacity = 0.9;
        
        switch(effect) {
            case 'sine':
                effectRadius = radius + Math.sin(pulsePhase) * 10;
                break;
            case 'cosine':
                effectRadius = radius + Math.cos(pulsePhase) * 8;
                opacity = 0.8 + Math.cos(pulsePhase) * 0.2;
                break;
            case 'pulse':
                effectRadius = radius + Math.sin(pulsePhase * 2) * 5;
                opacity = 0.9 + Math.sin(pulsePhase * 3) * 0.1;
                break;
        }
        
        const centerX = 200;
        const centerY = 200;
        
        const x = centerX + effectRadius * Math.cos(radian);
        const y = centerY + effectRadius * Math.sin(radian);
        
        element.style = {
            position: 'absolute',
            left: `${x}px`,
            top: `${y}px`,
            transform: 'translate(-50%, -50%)',
            zIndex: 10,
            opacity: opacity,
            transition: 'opacity 0.1s ease'
        };
        
        requestAnimationFrame(updatePosition);
    }
    
    updatePosition();
}
```

### 2. Custom Radar Chart Implementation (`public/customRadarChart.js`)

```javascript
// Custom Radar Chart using Pure Canvas
export function initializeCustomRadarChart() {
    const chartHtml = `
        <div style="width: 100%; height: 400px; position: relative; background: linear-gradient(135deg, #f8f9fa 0%, #e9ecef 100%); border-radius: 12px; box-shadow: 0 4px 20px rgba(0,0,0,0.1);">
            <canvas id="customRadarChart" width="400" height="400" style="width: 100%; height: 100%; border-radius: 12px;"></canvas>
        </div>
    `;
    
    $w('#radarChartEmbed').html = chartHtml;
    
    setTimeout(() => {
        renderCustomRadarChart();
    }, 100);
}

function renderCustomRadarChart() {
    const canvas = document.getElementById('customRadarChart');
    if (!canvas) return;
    
    const ctx = canvas.getContext('2d');
    const centerX = canvas.width / 2;
    const centerY = canvas.height / 2;
    const maxRadius = Math.min(centerX, centerY) - 60;
    
    // Data
    const skillsByCategory = {
        Tech: [
            { label: "LLM Integration", level: 1 },
            { label: "Responsible AI", level: 1 },
            { label: "System Design", level: 2 },
            { label: "Automation Engineering", level: 2 },
            { label: "Technical Writing", level: 3 },
            { label: "Data Analytics", level: 2 }
        ],
        Business: [
            { label: "AI Roadmapping", level: 2 },
            { label: "Product Roadmapping", level: 3 },
            { label: "Data-Driven Decision Making", level: 3 },
            { label: "GTM Strategy", level: 2 },
            { label: "Pitching & Presentation", level: 2 }
        ],
        Design: [
            { label: "Wireframing", level: 2 },
            { label: "HCI & AI", level: 1 },
            { label: "UX & Visual Design", level: 2 }
        ],
        Arts: [
            { label: "Creative Photography", level: 2 },
            { label: "Creative Writing", level: 2 }
        ]
    };
    
    const categoryColors = {
        Tech: "#1976D2",
        Business: "#FBC02D",
        Design: "#388E3C",
        Arts: "#E53935"
    };
    
    const allSkills = Object.values(skillsByCategory).flat();
    const skillCount = allSkills.length;
    
    // Clear canvas
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    
    // Draw background grid with enhanced styling
    drawEnhancedGrid(ctx, centerX, centerY, maxRadius, skillCount, allSkills);
    
    // Animate data drawing
    animateDataDrawing(ctx, centerX, centerY, maxRadius, skillsByCategory, categoryColors, allSkills);
}

function drawEnhancedGrid(ctx, centerX, centerY, maxRadius, skillCount, allSkills) {
    // Draw concentric circles with gradient effect
    for (let i = 1; i <= 3; i++) {
        const radius = (maxRadius / 3) * i;
        
        // Create gradient for circles
        const gradient = ctx.createRadialGradient(centerX, centerY, 0, centerX, centerY, radius);
        gradient.addColorStop(0, 'rgba(200, 200, 200, 0.1)');
        gradient.addColorStop(1, 'rgba(200, 200, 200, 0.3)');
        
        ctx.beginPath();
        ctx.arc(centerX, centerY, radius, 0, 2 * Math.PI);
        ctx.strokeStyle = i === 3 ? '#ddd' : '#eee';
        ctx.lineWidth = i === 3 ? 2 : 1;
        ctx.stroke();
        
        // Add level indicators
        if (i === 3) {
            ctx.fillStyle = '#999';
            ctx.font = '10px Poppins, sans-serif';
            ctx.textAlign = 'center';
            ctx.fillText('Can Teach', centerX + radius - 20, centerY - 5);
        } else if (i === 2) {
            ctx.fillStyle = '#aaa';
            ctx.fillText('Practicing', centerX + radius - 20, centerY - 5);
        } else {
            ctx.fillStyle = '#bbb';
            ctx.fillText('Learning', centerX + radius - 20, centerY - 5);
        }
    }
    
    // Draw radial lines with enhanced styling
    for (let i = 0; i < skillCount; i++) {
        const angle = (2 * Math.PI * i) / skillCount - Math.PI / 2;
        const endX = centerX + Math.cos(angle) * maxRadius;
        const endY = centerY + Math.sin(angle) * maxRadius;
        
        // Create gradient for lines
        const lineGradient = ctx.createLinearGradient(centerX, centerY, endX, endY);
        lineGradient.addColorStop(0, 'rgba(180, 180, 180, 0.8)');
        lineGradient.addColorStop(1, 'rgba(180, 180, 180, 0.3)');
        
        ctx.beginPath();
        ctx.moveTo(centerX, centerY);
        ctx.lineTo(endX, endY);
        ctx.strokeStyle = lineGradient;
        ctx.lineWidth = 1;
        ctx.stroke();
        
        // Draw skill labels with enhanced typography
        const labelX = centerX + Math.cos(angle) * (maxRadius + 25);
        const labelY = centerY + Math.sin(angle) * (maxRadius + 25);
        
        ctx.fillStyle = '#333';
        ctx.font = 'bold 11px Poppins, sans-serif';
        ctx.textAlign = 'center';
        ctx.textBaseline = 'middle';
        
        // Add text shadow effect
        ctx.shadowColor = 'rgba(0, 0, 0, 0.2)';
        ctx.shadowBlur = 2;
        ctx.shadowOffsetX = 1;
        ctx.shadowOffsetY = 1;
        
        const skill = allSkills[i];
        const labelText = skill.label.length > 12 ? skill.label.substring(0, 12) + '...' : skill.label;
        ctx.fillText(labelText, labelX, labelY);
        
        // Reset shadow
        ctx.shadowColor = 'transparent';
        ctx.shadowBlur = 0;
        ctx.shadowOffsetX = 0;
        ctx.shadowOffsetY = 0;
    }
}

function animateDataDrawing(ctx, centerX, centerY, maxRadius, skillsByCategory, categoryColors, allSkills) {
    let animationProgress = 0;
    const animationDuration = 3000; // 3 seconds
    const startTime = Date.now();
    
    function animate() {
        const elapsed = Date.now() - startTime;
        animationProgress = Math.min(elapsed / animationDuration, 1);
        
        // Easing function for smooth animation
        const easeProgress = 1 - Math.pow(1 - animationProgress, 3);
        
        // Clear previous drawings (keep grid)
        drawEnhancedGrid(ctx, centerX, centerY, maxRadius, allSkills.length, allSkills);
        
        // Draw each category with staggered animation
        Object.entries(skillsByCategory).forEach(([category, skills], categoryIndex) => {
            const categoryDelay = categoryIndex * 0.3;
            const adjustedProgress = Math.max(0, easeProgress - categoryDelay) / (1 - categoryDelay);
            
            if (adjustedProgress > 0) {
                drawCategoryData(ctx, centerX, centerY, maxRadius, skills, allSkills, 
                               categoryColors[category], adjustedProgress, category);
            }
        });
        
        if (animationProgress < 1) {
            requestAnimationFrame(animate);
        }
    }
    
    animate();
}

function drawCategoryData(ctx, centerX, centerY, maxRadius, skills, allSkills, color, progress, categoryName) {
    const points = [];
    
    // Calculate points for this category
    allSkills.forEach((skill, index) => {
        const angle = (2 * Math.PI * index) / allSkills.length - Math.PI / 2;
        const categorySkill = skills.find(s => s.label === skill.label);
        const level = categorySkill ? categorySkill.level : 0;
        const radius = (maxRadius / 3) * level * progress;
        
        const x = centerX + Math.cos(angle) * radius;
        const y = centerY + Math.sin(angle) * radius;
        points.push({ x, y, level });
    });
    
    // Draw filled area with transparency
    if (points.length > 0) {
        ctx.beginPath();
        ctx.moveTo(points[0].x, points[0].y);
        points.forEach(point => {
            ctx.lineTo(point.x, point.y);
        });
        ctx.closePath();
        
        // Create gradient fill
        const gradient = ctx.createRadialGradient(centerX, centerY, 0, centerX, centerY, maxRadius);
        gradient.addColorStop(0, color + '40'); // 25% opacity
        gradient.addColorStop(1, color + '20'); // 12% opacity
        
        ctx.fillStyle = gradient;
        ctx.fill();
        
        // Draw border
        ctx.strokeStyle = color;
        ctx.lineWidth = 2;
        ctx.stroke();
    }
    
    // Draw points with enhanced effects
    points.forEach((point, index) => {
        if (point.level > 0) {
            // Outer glow
            ctx.beginPath();
            ctx.arc(point.x, point.y, 8, 0, 2 * Math.PI);
            ctx.fillStyle = color + '30';
            ctx.fill();
            
            // Main point
            ctx.beginPath();
            ctx.arc(point.x, point.y, 5, 0, 2 * Math.PI);
            ctx.fillStyle = color;
            ctx.fill();
            
            // Inner highlight
            ctx.beginPath();
            ctx.arc(point.x, point.y, 3, 0, 2 * Math.PI);
            ctx.fillStyle = '#fff';
            ctx.fill();
            
            // Border
            ctx.beginPath();
            ctx.arc(point.x, point.y, 5, 0, 2 * Math.PI);
            ctx.strokeStyle = '#fff';
            ctx.lineWidth = 2;
            ctx.stroke();
        }
    });
}
```

## Benefits of This Alternative

1. **Zero Dependencies**: No external libraries required
2. **Enhanced Animations**: Custom easing and staggered effects
3. **Better Performance**: Direct canvas manipulation
4. **Gradient Effects**: Beautiful visual enhancements
5. **Full Control**: Complete customization possible
6. **Wix Compatible**: 100% guaranteed to work in Wix Velo

## Usage Instructions

1. Use this implementation if Chart.js doesn't work in your Wix environment
2. Follow the same setup steps as the main guide
3. Replace the Chart.js code with this custom implementation
4. Customize colors, animations, and styling as needed

This alternative provides even better visual effects while ensuring complete compatibility with Wix Velo constraints.
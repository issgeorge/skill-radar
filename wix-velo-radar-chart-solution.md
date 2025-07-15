# Wix Velo Radar Skills Chart - Implementation Guide

## Overview
This guide shows how to convert your HTML radar skills chart to work with Wix Velo, maintaining the same visual appeal and animations while working within Wix's constraints.

## Page Setup in Wix Editor

### 1. Add Required Elements
In your Wix Editor, add these elements to your page:

1. **HTML Embed Component** (for the canvas)
   - Element ID: `radarChartEmbed`
   - Size: 400px × 400px (adjust as needed)

2. **Text Elements** for rotating labels:
   - Element ID: `labelLearning` - Text: "Learning"
   - Element ID: `labelPracticing` - Text: "Practicing" 
   - Element ID: `labelTeach` - Text: "I Can Teach You"

3. **Container Box** for legend:
   - Element ID: `legendContainer`
   - Add 4 text elements inside:
     - `legendTech`, `legendBusiness`, `legendDesign`, `legendArts`

### 2. Page Styling
Add this CSS to your page (in Page Settings > SEO (Advanced) > Head):

```html
<style>
@import url('https://fonts.googleapis.com/css2?family=Playfair+Display:wght@400;600&family=Poppins:wght@300;500;700&display=swap');

#radarChartEmbed {
  position: relative !important;
}

.rotating-label {
  position: absolute;
  font-family: 'Poppins', sans-serif;
  color: #555;
  font-size: 13px;
  white-space: nowrap;
  opacity: 0.8;
  z-index: 10;
  pointer-events: none;
  transform-origin: center;
  transition: all 0.1s ease;
}

.pulse-animation {
  animation: wixPulse 1.2s infinite;
}

@keyframes wixPulse {
  0% { transform: scale(1); }
  50% { transform: scale(1.1); }
  100% { transform: scale(1); }
}

.legend-container {
  display: flex;
  justify-content: center;
  gap: 20px;
  font-family: 'Poppins', sans-serif;
  color: #444;
  flex-wrap: wrap;
}

.legend-item {
  display: flex;
  align-items: center;
  gap: 6px;
  font-size: 16px;
}

.legend-dot {
  width: 14px;
  height: 14px;
  border-radius: 3px;
  display: inline-block;
}
</style>
```

## Velo Code Implementation

### 1. Main Page Code (`yourPage.js`)

```javascript
import { initializeRadarChart } from 'public/radarChart.js';

$w.onReady(function () {
    // Initialize the radar chart
    initializeRadarChart();
    
    // Setup legend
    setupLegend();
    
    // Start label animations
    startLabelAnimations();
});

function setupLegend() {
    const legendData = [
        { id: 'legendTech', color: '#1976D2', text: '● Tech' },
        { id: 'legendBusiness', color: '#FBC02D', text: '● Business' },
        { id: 'legendDesign', color: '#388E3C', text: '● Design' },
        { id: 'legendArts', color: '#E53935', text: '● Arts' }
    ];

    legendData.forEach(item => {
        $w(item.id).text = item.text;
        $w(item.id).html = `<span style="color: ${item.color}; font-size: 16px;">●</span> <span style="color: #444;">${item.text.substring(2)}</span>`;
    });
}

function startLabelAnimations() {
    // Get chart container position and size
    const chartEmbed = $w('#radarChartEmbed');
    
    // Add CSS classes for styling
    $w('#labelLearning').html = '<div class="rotating-label">Learning</div>';
    $w('#labelPracticing').html = '<div class="rotating-label">Practicing</div>';
    $w('#labelTeach').html = '<div class="rotating-label pulse-animation">I Can Teach You</div>';
    
    // Start rotating animations
    animateRotatingLabel('labelLearning', 80, 0.3, chartEmbed);
    animateRotatingLabel('labelPracticing', 120, 0.5, chartEmbed);
    animateRotatingLabel('labelTeach', 160, 0.8, chartEmbed);
}

function animateRotatingLabel(elementId, radius, speed, chartContainer) {
    let angle = 0;
    const element = $w(`#${elementId}`);
    
    function updatePosition() {
        angle = (angle + speed) % 360;
        const radian = (angle * Math.PI) / 180;
        
        // Calculate center position relative to chart
        const centerX = 200; // Half of chart width
        const centerY = 200; // Half of chart height
        
        const x = centerX + radius * Math.cos(radian);
        const y = centerY + radius * Math.sin(radian);
        
        // Position relative to page
        element.style = {
            position: 'absolute',
            left: `${x}px`,
            top: `${y}px`,
            transform: 'translate(-50%, -50%)',
            zIndex: 10
        };
        
        setTimeout(updatePosition, 16); // ~60fps
    }
    
    updatePosition();
}
```

### 2. Public File for Chart Logic (`public/radarChart.js`)

```javascript
// Radar Chart Implementation for Wix Velo

export function initializeRadarChart() {
    const chartHtml = createChartHTML();
    $w('#radarChartEmbed').html = chartHtml;
    
    // Initialize chart after HTML is loaded
    setTimeout(() => {
        if (typeof Chart !== 'undefined') {
            renderRadarChart();
        } else {
            loadChartJS().then(() => {
                renderRadarChart();
            });
        }
    }, 100);
}

function loadChartJS() {
    return new Promise((resolve) => {
        const script = document.createElement('script');
        script.src = 'https://cdn.jsdelivr.net/npm/chart.js';
        script.onload = resolve;
        document.head.appendChild(script);
    });
}

function createChartHTML() {
    return `
        <div style="width: 100%; height: 400px; position: relative;">
            <canvas id="skillsRadarChart" style="width: 100% !important; height: 100% !important;"></canvas>
        </div>
    `;
}

function renderRadarChart() {
    const categoryColors = {
        Tech: "#1976D2",
        Business: "#FBC02D", 
        Design: "#388E3C",
        Arts: "#E53935"
    };

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

    // Create datasets
    const datasets = Object.entries(skillsByCategory).map(([category, skills]) => {
        const labels = Object.values(skillsByCategory).flat().map(s => s.label);
        const data = labels.map(label => {
            const skill = skills.find(s => s.label === label);
            return skill ? skill.level : 0;
        });

        return {
            label: category,
            data,
            backgroundColor: categoryColors[category] + '33', // Add transparency
            borderColor: categoryColors[category],
            borderWidth: 2,
            pointBackgroundColor: categoryColors[category],
            pointBorderColor: '#fff',
            pointHoverBackgroundColor: '#fff',
            pointHoverBorderColor: categoryColors[category],
            pointRadius: 4,
            pointHoverRadius: 6,
            fill: true
        };
    });

    const allLabels = Object.values(skillsByCategory).flat().map(s => s.label);

    const config = {
        type: 'radar',
        data: {
            labels: allLabels,
            datasets
        },
        options: {
            responsive: true,
            maintainAspectRatio: true,
            animation: {
                duration: 3000,
                easing: 'easeInOutQuart',
                delay: (context) => {
                    return context.datasetIndex * 500 + context.dataIndex * 100;
                }
            },
            scales: {
                r: {
                    min: 0,
                    max: 3,
                    ticks: {
                        stepSize: 1,
                        display: false
                    },
                    grid: { 
                        color: "#eee",
                        lineWidth: 1
                    },
                    angleLines: { 
                        color: "#ddd",
                        lineWidth: 1
                    },
                    pointLabels: {
                        font: {
                            size: 11,
                            family: 'Poppins, sans-serif'
                        },
                        color: "#333",
                        callback: function(label) {
                            // Truncate long labels
                            return label.length > 15 ? label.substring(0, 15) + '...' : label;
                        }
                    }
                }
            },
            plugins: {
                legend: {
                    display: false
                },
                tooltip: {
                    backgroundColor: 'rgba(0,0,0,0.8)',
                    titleColor: '#fff',
                    bodyColor: '#fff',
                    cornerRadius: 8,
                    displayColors: true,
                    callbacks: {
                        title: function(context) {
                            return context[0].label;
                        },
                        label: function(context) {
                            const levels = ['', 'Learning', 'Practicing', 'Can Teach'];
                            return `${context.dataset.label}: ${levels[context.parsed.r] || 'Unknown'}`;
                        }
                    }
                }
            },
            interaction: {
                intersect: false,
                mode: 'point'
            }
        }
    };

    // Create chart
    const canvas = document.getElementById('skillsRadarChart');
    if (canvas) {
        new Chart(canvas, config);
    }
}
```

## Enhanced Features

### 1. Improved Animations
- **Staggered Loading**: Each dataset appears with a delay for dramatic effect
- **Smooth Rotations**: Labels rotate smoothly around the chart
- **Pulse Effect**: "I Can Teach You" label pulses to draw attention
- **Hover Effects**: Enhanced tooltips and point interactions

### 2. Responsive Design
- Chart automatically adapts to container size
- Labels scale appropriately
- Mobile-friendly layout

### 3. Interactive Features
- Hover tooltips show skill levels with descriptive text
- Smooth point highlighting
- Professional color scheme

## Implementation Steps

1. **Create Page Elements**: Add all required elements in Wix Editor with correct IDs
2. **Add CSS**: Insert the provided CSS in page head
3. **Add Velo Code**: Copy the main page code to your page's code panel
4. **Create Public File**: Add the `radarChart.js` file to your public folder
5. **Test**: Preview the page to ensure everything works
6. **Customize**: Adjust colors, data, and animations as needed

## Customization Options

### Data Updates
Modify the `skillsByCategory` object in `radarChart.js` to update your skills and levels.

### Styling
- Adjust colors in `categoryColors` object
- Modify animation speeds in `animateRotatingLabel` function
- Change chart size by updating container dimensions

### Animation Tweaks
- Adjust rotation speeds by changing the `speed` parameter
- Modify radius values to change label distances
- Update animation duration in chart config

This implementation provides the same visual impact as your HTML version while being fully compatible with Wix Velo and offering enhanced interactivity.
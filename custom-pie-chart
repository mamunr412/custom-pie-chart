import React, { useEffect, useRef, useState } from 'react';

const PieChart = ({ data = [
  { label: 'Category A', value: 30, color: '#FF6384' },
  { label: 'Category B', value: 50, color: '#36A2EB' },
  { label: 'Category C', value: 20, color: '#FFCE56' }
]}) => {
  const canvasRef = useRef(null);
  const [hoveredSegment, setHoveredSegment] = useState(null);
  const [hoveredLabel, setHoveredLabel] = useState(null);
  const segmentRefs = useRef([]);
  const labelRefs = useRef([]);
  const animationRef = useRef(null);
  const [animationProgress, setAnimationProgress] = useState(0);
  const [isAnimating, setIsAnimating] = useState(false);
  const previousHoveredSegment = useRef(null);

  useEffect(() => {
    if (hoveredSegment !== previousHoveredSegment.current || hoveredLabel !== null) {
      setIsAnimating(true);
      setAnimationProgress(0);
      previousHoveredSegment.current = hoveredSegment;
    }
  }, [hoveredSegment, hoveredLabel]);

  useEffect(() => {
    const animate = () => {
      setAnimationProgress(prev => {
        const next = prev + 0.1;
        if (next >= 1) {
          setIsAnimating(false);
          return 1;
        }
        animationRef.current = requestAnimationFrame(animate);
        return next;
      });
    };

    if (isAnimating) {
      animationRef.current = requestAnimationFrame(animate);
    }

    return () => {
      if (animationRef.current) {
        cancelAnimationFrame(animationRef.current);
      }
    };
  }, [isAnimating]);

  useEffect(() => {
    const canvas = canvasRef.current;
    const ctx = canvas.getContext('2d');
    
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    
    const total = data.reduce((sum, item) => sum + item.value, 0);
    const centerX = canvas.width / 2;
    const centerY = canvas.height / 2;
    const baseOuterRadius = Math.min(centerX, centerY) - 60;
    const baseInnerRadius = baseOuterRadius * 0.6;

    segmentRefs.current = [];
    labelRefs.current = [];
    
    const drawRoundedArc = (centerX, centerY, radius, startAngle, endAngle, color, isHovered) => {
      const hoverScale = isHovered ? 1 + (0.1 * animationProgress) : 1;
      const outerRadius = radius * hoverScale;
      const innerRadius = baseInnerRadius * hoverScale;
      
      ctx.save();
      ctx.beginPath();
      ctx.arc(centerX, centerY, outerRadius, startAngle, endAngle);
      ctx.arc(centerX, centerY, innerRadius, endAngle, startAngle, true);
      ctx.closePath();
      ctx.clip();
      
      ctx.beginPath();
      ctx.arc(centerX, centerY, outerRadius, startAngle, endAngle);
      ctx.arc(centerX, centerY, innerRadius, endAngle, startAngle, true);
      ctx.closePath();
      
      if (isHovered) {
        const opacity = 0.2 * animationProgress;
        ctx.fillStyle = color;
        ctx.fill();
        ctx.strokeStyle = `rgba(51, 51, 51, ${opacity})`;
        ctx.lineWidth = 2;
        ctx.stroke();
      } else {
        ctx.fillStyle = color;
        ctx.fill();
      }
      
      ctx.restore();
    };
    
    let startAngle = 0;
    data.forEach((item, index) => {
      const sliceAngle = (2 * Math.PI * item.value) / total;
      const endAngle = startAngle + sliceAngle;
      const isHovered = index === hoveredSegment;
      
      segmentRefs.current.push({
        startAngle,
        endAngle,
        item,
        percentage: Math.round((item.value / total) * 100)
      });
      
      drawRoundedArc(centerX, centerY, baseOuterRadius, startAngle, endAngle, item.color, isHovered);
      
      // Draw and store label information
      const labelAngle = startAngle + sliceAngle / 2;
      const labelRadius = (baseOuterRadius + baseInnerRadius) / 2;
      const labelX = centerX + Math.cos(labelAngle) * labelRadius;
      const labelY = centerY + Math.sin(labelAngle) * labelRadius;
      
      labelRefs.current.push({
        x: labelX,
        y: labelY,
        angle: labelAngle,
        percentage: Math.round((item.value / total) * 100)
      });

      // Draw percentage label with hover effect
      const isLabelHovered = index === hoveredLabel;
      const labelScale = isLabelHovered ? 1.4 - (0.4 * (1 - animationProgress)) : 1;
      const labelOpacity = isLabelHovered ? 1 : 0.8;
      
      ctx.save();
      ctx.translate(labelX, labelY);
      ctx.scale(labelScale, labelScale);
      ctx.translate(-labelX, -labelY);
      
      ctx.fillStyle = `rgba(0, 0, 0, ${labelOpacity})`;
      ctx.font = `${isLabelHovered ? 'bold' : 'normal'} 14px Arial`;
      ctx.textAlign = 'center';
      ctx.textBaseline = 'middle';
      ctx.fillText(`${Math.round((item.value / total) * 100)}%`, labelX, labelY);
      ctx.restore();
      
      startAngle = endAngle;
    });

    ctx.beginPath();
    ctx.arc(centerX, centerY, baseInnerRadius, 0, 2 * Math.PI);
    ctx.fillStyle = '#fff';
    ctx.fill();
    ctx.strokeStyle = '#e5e5e5';
    ctx.lineWidth = 1;
    ctx.stroke();

    ctx.fillStyle = '#333';
    ctx.font = 'bold 16px Arial';
    ctx.textAlign = 'center';
    ctx.textBaseline = 'middle';
    
    if (hoveredSegment !== null || hoveredLabel !== null) {
      const segment = data[hoveredSegment !== null ? hoveredSegment : hoveredLabel];
      const percentage = Math.round((segment.value / total) * 100);
      const opacity = Math.min(1, animationProgress * 2);
      ctx.fillStyle = `rgba(51, 51, 51, ${opacity})`;
      ctx.fillText(segment.label, centerX, centerY - 10);
      ctx.fillText(`${percentage}%`, centerX, centerY + 10);
    } else {
      const opacity = animationProgress < 0.5 ? 1 - animationProgress * 2 : 0;
      ctx.fillStyle = `rgba(51, 51, 51, ${opacity})`;
      ctx.fillText('Total', centerX, centerY - 10);
      ctx.fillText(total, centerX, centerY + 10);
    }
  }, [data, hoveredSegment, hoveredLabel, animationProgress]);

  const handleMouseMove = (event) => {
    const canvas = canvasRef.current;
    const rect = canvas.getBoundingClientRect();
    const x = event.clientX - rect.left;
    const y = event.clientY - rect.top;
    
    const centerX = canvas.width / 2;
    const centerY = canvas.height / 2;
    const angle = Math.atan2(y - centerY, x - centerX);
    const normalizedAngle = angle < 0 ? angle + 2 * Math.PI : angle;
    const distance = Math.sqrt((x - centerX) ** 2 + (y - centerY) ** 2);
    
    // Check if mouse is near any label
    const labelHoverRadius = 20;
    const hoveredLabelIndex = labelRefs.current.findIndex(label => {
      const dx = x - label.x;
      const dy = y - label.y;
      return Math.sqrt(dx * dx + dy * dy) <= labelHoverRadius;
    });

    if (hoveredLabelIndex >= 0) {
      setHoveredLabel(hoveredLabelIndex);
      setHoveredSegment(null);
      return;
    } else {
      setHoveredLabel(null);
    }

    const outerRadius = Math.min(centerX, centerY) - 60;
    const innerRadius = outerRadius * 0.6;
    
    if (distance >= innerRadius && distance <= outerRadius) {
      const hoveredIndex = segmentRefs.current.findIndex(
        segment => normalizedAngle >= segment.startAngle && normalizedAngle <= segment.endAngle
      );
      setHoveredSegment(hoveredIndex >= 0 ? hoveredIndex : null);
    } else {
      setHoveredSegment(null);
    }
  };

  const handleMouseLeave = () => {
    setHoveredSegment(null);
    setHoveredLabel(null);
  };

  return (
    <div className="flex flex-col items-center space-y-4">
      <canvas
        ref={canvasRef}
        width={400}
        height={400}
        className="w-full max-w-md"
        onMouseMove={handleMouseMove}
        onMouseLeave={handleMouseLeave}
      />
      <div className="flex flex-wrap justify-center gap-4">
        {data.map((item, index) => (
          <div 
            key={index} 
            className={`flex items-center space-x-2 cursor-pointer p-2 rounded transition-all duration-300 ease-in-out ${
              (hoveredSegment === index || hoveredLabel === index) ? 'bg-gray-100 scale-105' : ''
            }`}
            onMouseEnter={() => setHoveredSegment(index)}
            onMouseLeave={() => setHoveredSegment(null)}
          >
            <div 
              className={`w-4 h-4 rounded-full transition-all duration-300 ease-in-out ${
                (hoveredSegment === index || hoveredLabel === index) ? 'scale-125' : ''
              }`}
              style={{ backgroundColor: item.color }}
            />
            <span className={`text-sm transition-all duration-300 ease-in-out ${
              (hoveredSegment === index || hoveredLabel === index) ? 'font-medium' : ''
            }`}>
              {item.label} ({item.value})
            </span>
          </div>
        ))}
      </div>
    </div>
  );
};

export default PieChart;

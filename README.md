Heart Disease Prediction using PyTorch
A neural network model to predict heart disease risk from patient data

📌 Project Overview
This project trains a PyTorch neural network to predict heart disease risk using clinical features like cholesterol levels, blood pressure, and ECG results.

Key Features:

1. Preprocesses mixed numerical/categorical data

2. Implements a customizable neural network

3. Tracks and visualizes training progress

4. Achieves ~85-90% test accuracy

import { useState, useRef, useEffect } from "react";

// Mock DragDrop hook for demo
const useDragDrop = () => ({
  draggedItem: null,
  isDragging: false,
  setDraggedItem: () => {},
  setIsDragging: () => {}
});

// Mock DraggableElement component for demo
const DraggableElement = ({ element, onUpdateElement, onDelete, isSelected, setIsSelected }) => (
  <div
    className={`absolute border-2 ${isSelected ? 'border-blue-500' : 'border-gray-300'} bg-white rounded p-2 cursor-move`}
    style={{
      left: element.x,
      top: element.y,
      width: element.width,
      height: element.height,
    }}
    onClick={(e) => {
      e.stopPropagation();
      setIsSelected(element.id);
    }}
  >
    {element.type === 'text' ? element.content : element.name}
    <button
      onClick={(e) => {
        e.stopPropagation();
        onDelete(element.id);
      }}
      className="absolute -top-2 -right-2 w-6 h-6 bg-red-500 text-white rounded-full text-xs"
    >
      ×
    </button>
  </div>
);

const Canvas = ({ 
  pageData = { elements: [], background: "#ffffff" }, 
  pageIndex = 0, 
  onUpdatePage = () => {}, 
  format = { name: "A4", width: 794, height: 1123 } 
}) => {
  const { draggedItem, isDragging, setDraggedItem, setIsDragging } = useDragDrop();
  const canvasRef = useRef(null);
  const textAreaRef = useRef(null);
  const textDisplayRef = useRef(null);
  
  const [elements, setElements] = useState(pageData?.elements || []);
  const [background, setBackground] = useState(pageData?.background || "#ffffff");
  const [selectedElementId, setSelectedElementId] = useState(null);
  
  // Lined paper states
  const [showLines, setShowLines] = useState(pageData?.showLines || false);
  const [lineSpacing, setLineSpacing] = useState(pageData?.lineSpacing || 30);
  const [lineColor, setLineColor] = useState(pageData?.lineColor || "#e0e7ff");
  
  // Text editing states
  const [isTextEditing, setIsTextEditing] = useState(false);
  const [textContent, setTextContent] = useState(pageData?.textContent || "Click here to start typing...");
  
  // Text styling states
  const [textColor, setTextColor] = useState(pageData?.textColor || "#000000");
  const [fontSize, setFontSize] = useState(pageData?.fontSize || 16);
  const [fontFamily, setFontFamily] = useState(pageData?.fontFamily || "Arial");
  const [isBold, setIsBold] = useState(pageData?.isBold || false);
  const [isItalic, setIsItalic] = useState(pageData?.isItalic || false);
  const [isUnderline, setIsUnderline] = useState(pageData?.isUnderline || false);
  const [textAlign, setTextAlign] = useState(pageData?.textAlign || "left");
  
  // Text selection state
  const [selectionRange, setSelectionRange] = useState(null);
  const [styledText, setStyledText] = useState(pageData?.styledText || []);
  const [lastClickTime, setLastClickTime] = useState(0);

  const scale = Math.min(1, 800 / format.width);

  // Initialize styledText from pageData
  useEffect(() => {
    if (pageData?.styledText) {
      setStyledText(pageData.styledText);
    }
  }, [pageData]);

  // Simple update function - only call parent when needed
  const updatePageData = (updates) => {
    onUpdatePage(pageIndex, { 
      elements, 
      background, 
      showLines, 
      lineSpacing, 
      lineColor, 
      textContent,
      textColor,
      fontSize,
      fontFamily,
      isBold,
      isItalic,
      isUnderline,
      textAlign,
      styledText,
      ...updates 
    });
  };

  // Toggle lines function
  const toggleLines = () => {
    const newShowLines = !showLines;
    setShowLines(newShowLines);
    updatePageData({ showLines: newShowLines });
  };

  // Change line spacing
  const changeLineSpacing = (newSpacing) => {
    setLineSpacing(newSpacing);
    updatePageData({ lineSpacing: newSpacing });
  };

  // Generate line positions
  const generateLines = () => {
    const lines = [];
    const startY = 40;
    const numLines = Math.floor((format.height - startY - 20) / lineSpacing);
    
    for (let i = 0; i < numLines; i++) {
      lines.push(startY + (i * lineSpacing));
    }
    return lines;
  };

  // Improved click handling
  const handleCanvasClick = (e) => {
    e.preventDefault();
    e.stopPropagation();
    
    const currentTime = Date.now();
    const timeSinceLastClick = currentTime - lastClickTime;
    setLastClickTime(currentTime);
    
    // Check if clicking on an element
    const rect = canvasRef.current?.getBoundingClientRect();
    if (!rect) return;
    
    const clickedX = (e.clientX - rect.left) / scale;
    const clickedY = (e.clientY - rect.top) / scale;
    
    const elementUnderMouse = elements.find(el => {
      return clickedX >= el.x && clickedX <= el.x + el.width && 
             clickedY >= el.y && clickedY <= el.y + el.height;
    });
    
    if (elementUnderMouse) {
      setSelectedElementId(elementUnderMouse.id);
      setIsTextEditing(false);
      setSelectionRange(null);
      return;
    }
    
    // If we have text content and we're not currently editing
    if (textContent && !isTextEditing) {
      // Check if clicking on text area (rough approximation)
      const textAreaBounds = {
        x: 16, // padding
        y: 16, // padding
        width: format.width - 32,
        height: format.height - 32
      };
      
      const isClickingOnTextArea = 
        clickedX >= textAreaBounds.x && 
        clickedX <= textAreaBounds.x + textAreaBounds.width &&
        clickedY >= textAreaBounds.y && 
        clickedY <= textAreaBounds.y + textAreaBounds.height;
      
      if (isClickingOnTextArea) {
        // Double click or single click with existing text enters edit mode
        if (timeSinceLastClick < 300 || textContent.trim() !== "Click here to start typing...") {
          startTextEditing();
          return;
        }
      }
    }
    
    // Single click on empty canvas or outside text area
    if (!textContent || textContent.trim() === "Click here to start typing...") {
      startTextEditing();
    } else {
      // Just clear selection if we have text but not editing
      setSelectedElementId(null);
      setSelectionRange(null);
    }
  };

  // Start text editing
  const startTextEditing = () => {
    setSelectedElementId(null);
    setIsTextEditing(true);
    
    // Clear placeholder text when starting to edit
    if (textContent === "Click here to start typing...") {
      setTextContent("");
    }
    
    // Focus the textarea after state update
    setTimeout(() => {
      if (textAreaRef.current) {
        textAreaRef.current.focus();
        // Place cursor at end if there's content
        if (textContent && textContent !== "Click here to start typing...") {
          textAreaRef.current.setSelectionRange(textContent.length, textContent.length);
        }
      }
    }, 50);
  };

  // Handle text input
  const handleTextInput = (e) => {
    const newText = e.target.value;
    setTextContent(newText);
  };

  // Save text when done editing
  const finishTextEditing = () => {
    setIsTextEditing(false);
    setSelectionRange(null);
    
    // If text is empty, set placeholder
    if (!textContent.trim()) {
      setTextContent("Click here to start typing...");
      updatePageData({ textContent: "Click here to start typing..." });
    } else {
      updatePageData({ textContent });
    }
  };

  // Handle selection change in textarea
  const handleSelectionChange = () => {
    if (!textAreaRef.current || !isTextEditing) return;
    
    const start = textAreaRef.current.selectionStart;
    const end = textAreaRef.current.selectionEnd;
    
    if (start !== end && textContent.trim()) {
      setSelectionRange({ start, end });
    } else {
      setSelectionRange(null);
    }
  };

  // Handle keyboard shortcuts
  const handleKeyDown = (e) => {
    if (e.key === 'Escape') {
      finishTextEditing();
      return;
    }
    
    // Formatting shortcuts
    if (e.ctrlKey || e.metaKey) {
      switch (e.key) {
        case 'b':
          e.preventDefault();
          toggleBold();
          break;
        case 'i':
          e.preventDefault();
          toggleItalic();
          break;
        case 'u':
          e.preventDefault();
          toggleUnderline();
          break;
      }
    }
  };

  // Apply style to selected text
  const applyStyleToSelection = (styleType, value) => {
    if (!selectionRange || !textContent.trim()) {
      // Apply to all text if no selection
      handleStyleChange(styleType, value);
      return;
    }

    const { start, end } = selectionRange;
    
    // Remove any existing styles that overlap with this selection
    const newStyledText = styledText.filter(style => 
      !(style.start < end && style.end > start)
    );
    
    // Only add style if there's actual selected text
    if (start !== end) {
      newStyledText.push({
        start,
        end,
        [styleType]: value,
        id: Date.now() + Math.random()
      });
    }
    
    // Sort by start position for rendering
    newStyledText.sort((a, b) => a.start - b.start);
    
    setStyledText(newStyledText);
    updatePageData({ styledText: newStyledText });
  };

  // Text formatting functions
  const toggleBold = () => {
    if (selectionRange && isTextEditing) {
      applyStyleToSelection('fontWeight', isBold ? 'normal' : 'bold');
    } else {
      const newBold = !isBold;
      setIsBold(newBold);
      updatePageData({ isBold: newBold });
    }
  };

  const toggleItalic = () => {
    if (selectionRange && isTextEditing) {
      applyStyleToSelection('fontStyle', isItalic ? 'normal' : 'italic');
    } else {
      const newItalic = !isItalic;
      setIsItalic(newItalic);
      updatePageData({ isItalic: newItalic });
    }
  };

  const toggleUnderline = () => {
    if (selectionRange && isTextEditing) {
      applyStyleToSelection('textDecoration', isUnderline ? 'none' : 'underline');
    } else {
      const newUnderline = !isUnderline;
      setIsUnderline(newUnderline);
      updatePageData({ isUnderline: newUnderline });
    }
  };

  // Handle text style changes
  const handleStyleChange = (style, value) => {
    switch(style) {
      case 'color':
        if (selectionRange && isTextEditing) {
          applyStyleToSelection('color', value);
        } else {
          setTextColor(value);
          updatePageData({ textColor: value });
        }
        break;
      case 'fontSize':
        if (selectionRange && isTextEditing) {
          applyStyleToSelection('fontSize', value);
        } else {
          setFontSize(value);
          updatePageData({ fontSize: value });
        }
        break;
      case 'fontFamily':
        if (selectionRange && isTextEditing) {
          applyStyleToSelection('fontFamily', value);
        } else {
          setFontFamily(value);
          updatePageData({ fontFamily: value });
        }
        break;
      case 'align':
        setTextAlign(value);
        updatePageData({ textAlign: value });
        break;
    }
  };

  // Clear all text
  const clearText = () => {
    setTextContent("Click here to start typing...");
    setStyledText([]);
    setIsTextEditing(false);
    setSelectionRange(null);
    updatePageData({ textContent: "Click here to start typing...", styledText: [] });
  };

  // Get text styles for a specific span
  const getTextSpanStyles = (styleObj) => {
    const baseStyles = {
      fontFamily: fontFamily,
      fontSize: `${fontSize}px`,
      color: textColor,
      fontWeight: isBold ? 'bold' : 'normal',
      fontStyle: isItalic ? 'italic' : 'normal',
      textDecoration: isUnderline ? 'underline' : 'none',
      lineHeight: `${Math.max(fontSize * 1.4, lineSpacing)}px`,
    };

    // Override with specific styles if they exist
    if (styleObj.fontFamily) baseStyles.fontFamily = styleObj.fontFamily;
    if (styleObj.fontSize) baseStyles.fontSize = `${styleObj.fontSize}px`;
    if (styleObj.color) baseStyles.color = styleObj.color;
    if (styleObj.fontWeight) baseStyles.fontWeight = styleObj.fontWeight;
    if (styleObj.fontStyle) baseStyles.fontStyle = styleObj.fontStyle;
    if (styleObj.textDecoration) baseStyles.textDecoration = styleObj.textDecoration;

    return baseStyles;
  };

  // Render styled text
  const renderStyledText = () => {
    if (!textContent || textContent === "Click here to start typing...") {
      return (
        <span style={{ color: '#999', fontStyle: 'italic' }}>
          {textContent}
        </span>
      );
    }

    if (styledText.length === 0) {
      return <span>{textContent}</span>;
    }

    const result = [];
    let lastIndex = 0;

    // Sort styled text by start position
    const sortedStyles = [...styledText].sort((a, b) => a.start - b.start);

    sortedStyles.forEach((style, index) => {
      // Add text before this style
      if (style.start > lastIndex) {
        result.push(
          <span key={`text-${lastIndex}-${style.start}`}>
            {textContent.substring(lastIndex, style.start)}
          </span>
        );
      }

      // Add styled text
      result.push(
        <span key={`style-${style.id || index}`} style={getTextSpanStyles(style)}>
          {textContent.substring(style.start, style.end)}
        </span>
      );

      lastIndex = style.end;
    });

    // Add any remaining text
    if (lastIndex < textContent.length) {
      result.push(
        <span key={`text-${lastIndex}-end`}>
          {textContent.substring(lastIndex)}
        </span>
      );
    }

    return result;
  };

  // Handle text display click for re-editing
  const handleTextDisplayClick = (e) => {
    e.stopPropagation();
    if (!isTextEditing) {
      startTextEditing();
    }
  };

  // Drag & Drop integration
  const handleDrop = (e) => {
    e.preventDefault();
    if (!draggedItem || !canvasRef.current) return;
    const rect = canvasRef.current.getBoundingClientRect();
    const x = (e.clientX - rect.left) / scale;
    const y = (e.clientY - rect.top) / scale;
    let width = 200;
    let height = 50;
    if (draggedItem.type === "logo") {
      width = 150;
      height = 150;
    } else if (draggedItem.type === "text") {
      width = 250;
      height = 60;
    }
    const finalX = Math.max(0, Math.min(x - width / 2, format.width - width));
    const finalY = Math.max(0, Math.min(y - height / 2, format.height - height));
    const newElement = {
      id: Date.now() + Math.random(),
      type: draggedItem.type,
      x: finalX,
      y: finalY,
      width,
      height,
    };
    if (draggedItem.type === "text") {
      newElement.content = draggedItem.content || "Text content";
      newElement.fontSize = draggedItem.fontSize || 16;
      newElement.fontWeight = draggedItem.fontWeight || "normal";
      newElement.fontFamily = draggedItem.fontFamily || "Arial";
      newElement.fontStyle = draggedItem.fontStyle || "normal";
      newElement.color = draggedItem.color || "#000000";
      newElement.textAlign = draggedItem.textAlign || "left";
      newElement.lineHeight = draggedItem.lineHeight || 1.4;
    } else if (draggedItem.type === "logo") {
      newElement.src = draggedItem.src || "https://via.placeholder.com/150x150/D4AF37/FFFFFF?text=LOGO";
      newElement.alt = draggedItem.alt || draggedItem.name || "Image";
      newElement.name = draggedItem.name || "Image";
    }
    const updatedElements = [...elements, newElement];
    setElements(updatedElements);
    updatePageData({ elements: updatedElements });
    setSelectedElementId(newElement.id);
    setDraggedItem(null);
    setIsDragging(false);
  };

  const handleDragOver = (e) => {
    e.preventDefault();
    if (isDragging) e.dataTransfer.dropEffect = "copy";
  };

  const updateElement = (id, updates) => {
    const newElements = elements.map((el) => (el.id === id ? { ...el, ...updates } : el));
    setElements(newElements);
    updatePageData({ elements: newElements });
  };

  const deleteElement = (id) => {
    const newElements = elements.filter((el) => el.id !== id);
    setElements(newElements);
    updatePageData({ elements: newElements });
    if (selectedElementId === id) setSelectedElementId(null);
  };

  return (
    <div className="relative" style={{ perspective: '1500px' }}>
      <div className="mb-6 text-center">
        <span className="text-sm text-gray-700 bg-gray-100 px-4 py-2 rounded-full border border-gray-200 shadow-sm font-semibold">
          {format.name} - {format.width} × {format.height}px
        </span>
      </div>
      
      {/* Toolbar */}
      <div className="mb-4 flex justify-center items-center gap-4 flex-wrap">
        <button
          onClick={toggleLines}
          className={`px-4 py-2 rounded-lg font-semibold transition-all duration-200 ${
            showLines 
              ? 'bg-blue-500 text-white shadow-lg hover:bg-blue-600' 
              : 'bg-gray-200 text-gray-700 hover:bg-gray-300'
          }`}
        >
          📝 {showLines ? 'Hide Lines' : 'Show Lines'}
        </button>
        
        <div className="flex items-center gap-2">
          <label className="text-sm text-gray-600 font-medium">Line Spacing:</label>
          <select
            value={lineSpacing}
            onChange={(e) => changeLineSpacing(Number(e.target.value))}
            className="px-2 py-1 border border-gray-300 rounded text-sm"
          >
            <option value={20}>Narrow (20px)</option>
            <option value={25}>Medium-Narrow (25px)</option>
            <option value={30}>Normal (30px)</option>
            <option value={35}>Medium-Wide (35px)</option>
            <option value={40}>Wide (40px)</option>
          </select>
        </div>

        {showLines && (
          <div className="flex items-center gap-2">
            <label className="text-sm text-gray-600 font-medium">Line Color:</label>
            <input
              type="color"
              value={lineColor}
              onChange={(e) => {
                setLineColor(e.target.value);
                updatePageData({ lineColor: e.target.value });
              }}
              className="w-8 h-6 border border-gray-300 rounded cursor-pointer"
            />
          </div>
        )}
        
        <div className="flex items-center gap-2">
          <label className="text-sm text-gray-600 font-medium">Text Color:</label>
          <input
            type="color"
            value={textColor}
            onChange={(e) => handleStyleChange('color', e.target.value)}
            className="w-8 h-6 border border-gray-300 rounded cursor-pointer"
          />
        </div>
        
        <div className="flex items-center gap-2">
          <label className="text-sm text-gray-600 font-medium">Font Size:</label>
          <select
            value={fontSize}
            onChange={(e) => handleStyleChange('fontSize', Number(e.target.value))}
            className="px-2 py-1 border border-gray-300 rounded text-sm"
          >
            <option value={12}>12px</option>
            <option value={14}>14px</option>
            <option value={16}>16px</option>
            <option value={18}>18px</option>
            <option value={20}>20px</option>
            <option value={24}>24px</option>
            <option value={28}>28px</option>
          </select>
        </div>
        
        <div className="flex items-center gap-2">
          <label className="text-sm text-gray-600 font-medium">Font:</label>
          <select
            value={fontFamily}
            onChange={(e) => handleStyleChange('fontFamily', e.target.value)}
            className="px-2 py-1 border border-gray-300 rounded text-sm"
          >
            <option value="Arial">Arial</option>
            <option value="Times New Roman">Times New Roman</option>
            <option value="Courier New">Courier New</option>
            <option value="Verdana">Verdana</option>
            <option value="Georgia">Georgia</option>
            <option value="Comic Sans MS">Comic Sans MS</option>
            <option value="Impact">Impact</option>
          </select>
        </div>
        
        <div className="flex items-center gap-1">
          <button
            onClick={toggleBold}
            className={`px-2 py-1 border border-gray-300 rounded text-sm font-bold transition-colors ${
              (selectionRange && isTextEditing) || isBold ? 'bg-blue-500 text-white' : 'bg-white text-gray-700 hover:bg-gray-100'
            }`}
            title="Bold (Ctrl+B)"
          >
            B
          </button>
          <button
            onClick={toggleItalic}
            className={`px-2 py-1 border border-gray-300 rounded text-sm italic transition-colors ${
              (selectionRange && isTextEditing) || isItalic ? 'bg-blue-500 text-white' : 'bg-white text-gray-700 hover:bg-gray-100'
            }`}
            title="Italic (Ctrl+I)"
          >
            I
          </button>
          <button
            onClick={toggleUnderline}
            className={`px-2 py-1 border border-gray-300 rounded text-sm underline transition-colors ${
              (selectionRange && isTextEditing) || isUnderline ? 'bg-blue-500 text-white' : 'bg-white text-gray-700 hover:bg-gray-100'
            }`}
            title="Underline (Ctrl+U)"
          >
            U
          </button>
        </div>
        
        <div className="flex items-center gap-1">
          <button
            onClick={() => handleStyleChange('align', 'left')}
            className={`px-2 py-1 border border-gray-300 rounded text-sm transition-colors ${
              textAlign === 'left' ? 'bg-blue-500 text-white' : 'bg-white text-gray-700 hover:bg-gray-100'
            }`}
            title="Align Left"
          >
            ⬅️
          </button>
          <button
            onClick={() => handleStyleChange('align', 'center')}
            className={`px-2 py-1 border border-gray-300 rounded text-sm transition-colors ${
              textAlign === 'center' ? 'bg-blue-500 text-white' : 'bg-white text-gray-700 hover:bg-gray-100'
            }`}
            title="Align Center"
          >
            ↔️
          </button>
          <button
            onClick={() => handleStyleChange('align', 'right')}
            className={`px-2 py-1 border border-gray-300 rounded text-sm transition-colors ${
              textAlign === 'right' ? 'bg-blue-500 text-white' : 'bg-white text-gray-700 hover:bg-gray-100'
            }`}
            title="Align Right"
          >
            ➡️
          </button>
        </div>
        
        {textContent && textContent !== "Click here to start typing..." && (
          <button
            onClick={clearText}
            className="px-3 py-1 text-xs bg-red-100 text-red-700 rounded hover:bg-red-200 transition-colors"
            title="Clear all text"
          >
            🗑️ Clear Text
          </button>
        )}

        {isTextEditing && (
          <button
            onClick={finishTextEditing}
            className="px-3 py-1 text-xs bg-green-100 text-green-700 rounded hover:bg-green-200 transition-colors"
            title="Finish editing (Esc)"
          >
            ✅ Done Editing
          </button>
        )}

        {selectionRange && isTextEditing && (
          <div className="px-3 py-1 text-xs bg-blue-100 text-blue-700 rounded-full">
            🔥 Text Selected ({selectionRange.end - selectionRange.start} chars)
          </div>
        )}
      </div>
      
      <div className="relative flex justify-center">
        <div
          className="relative border border-gray-300 rounded-2xl overflow-hidden bg-white"
          style={{
            transform: `scale(${scale})`,
            transformOrigin: "top center",
            boxShadow: '0 10px 30px rgba(0,0,0,0.2)',
            marginBottom: scale < 1 ? `${(1 - scale) * format.height}px` : 0,
          }}
        >
          <div
            ref={canvasRef}
            className="relative overflow-hidden"
            style={{
              width: format.width,
              height: format.height,
              backgroundColor: background,
              cursor: isTextEditing ? 'text' : 'pointer',
            }}
            onClick={handleCanvasClick}
            onDrop={handleDrop}
            onDragOver={handleDragOver}
          >
            {/* Horizontal Lines */}
            {showLines && (
              <div className="absolute inset-0 pointer-events-none" style={{ zIndex: 1 }}>
                {generateLines().map((lineY, index) => (
                  <div
                    key={`line-${index}`}
                    className="absolute left-0 right-0"
                    style={{
                      top: `${lineY}px`,
                      height: '1px',
                      backgroundColor: lineColor,
                    }}
                  />
                ))}
                <div
                  className="absolute top-0 bottom-0"
                  style={{
                    left: '60px',
                    width: '1px',
                    backgroundColor: '#ffb3ba',
                    opacity: 0.7,
                  }}
                />
              </div>
            )}

            {/* Text Editor */}
            {isTextEditing ? (
              <textarea
                ref={textAreaRef}
                value={textContent === "Click here to start typing..." ? "" : textContent}
                onChange={handleTextInput}
                onKeyDown={handleKeyDown}
                onSelect={handleSelectionChange}
                onBlur={finishTextEditing}
                placeholder="Start typing... Use Ctrl+B for bold, Ctrl+I for italic, Ctrl+U for underline"
                className="absolute inset-0 w-full h-full resize-none border-none outline-none bg-transparent p-4"
                style={{
                  fontFamily: fontFamily,
                  fontSize: `${fontSize}px`,
                  color: textColor,
                  fontWeight: isBold ? 'bold' : 'normal',
                  fontStyle: isItalic ? 'italic' : 'normal',
                  textDecoration: isUnderline ? 'underline' : 'none',
                  textAlign: textAlign,
                  lineHeight: `${Math.max(fontSize * 1.4, lineSpacing)}px`,
                  caretColor: textColor,
                  whiteSpace: 'pre-wrap',
                  overflow: 'auto',
                  zIndex: 10,
                }}
              />
            ) : (
              /* Display Text */
              textContent && (
                <div
                  ref={textDisplayRef}
                  className="absolute inset-0 p-4 cursor-text hover:bg-blue-50/30 transition-colors"
                  style={{
                    fontFamily: fontFamily,
                    fontSize: `${fontSize}px`,
                    color: textContent === "Click here to start typing..." ? '#999' : textColor,
                  fontWeight: isBold ? 'bold' : 'normal',
                  fontStyle: isItalic ? 'italic' : 'normal',
                  textDecoration: isUnderline ? 'underline' : 'none',
                  textAlign: textAlign,
                  lineHeight: `${Math.max(fontSize * 1.4, lineSpacing)}px`,
                  zIndex: 5,
                  overflow: 'hidden',
                  userSelect: 'text',
                }}
                onClick={handleTextDisplayClick}
                onDoubleClick={startTextEditing}
              >
                {renderStyledText()}
              </div>
            )}
            
            {/* Draggable Elements */}
            {elements.map((element) => (
              <DraggableElement
                key={element.id}
                element={element}
                onUpdateElement={updateElement}
                onDelete={deleteElement}
                canvasWidth={format.width}
                canvasHeight={format.height}
                isSelected={selectedElementId === element.id}
                setIsSelected={setSelectedElementId}
              />
            ))}

            {/* Text Editor Active Indicator */}
            {isTextEditing && (
              <div className="absolute top-2 right-2 bg-green-100 text-green-700 px-3 py-1 rounded-full text-xs font-semibold shadow-sm border border-green-200" style={{ zIndex: 25 }}>
                ✍️ Text Editor Active - Press Esc or click outside to finish
              </div>
            )}

            {/* Drop zone indicator */}
            {isDragging && (
              <div className="absolute inset-0 bg-blue-100/30 border-4 border-blue-400 border-dashed flex items-center justify-center pointer-events-none rounded-2xl" style={{ zIndex: 20 }}>
                <div className="bg-white px-8 py-4 rounded-2xl shadow-xl border border-blue-300 animate-pulse">
                  <span className="text-blue-700 font-bold text-lg">
                    {draggedItem?.type === "logo" ? "✨ Drop your image here" : "📝 Drop element here"}
                  </span>
                </div>
              </div>
            )}
          </div>
        </div>
      </div>
      
      <div className="mt-6 text-center text-sm text-gray-600 font-semibold bg-gray-100 py-2 px-4 rounded-full border border-gray-200">
        Page {pageIndex + 1} of {format.name} 
        {showLines && ' 📝'} 
        {textContent && textContent !== "Click here to start typing..." && ' ✍️'}
        {isTextEditing && ' 🖊️'}
        {selectionRange && ' 🎯'}
      </div>

      {/* Instructions */}
      <div className="mt-4 text-center">
        <div className="inline-block bg-blue-50 px-4 py-2 rounded-lg border border-blue-200 text-xs text-blue-700">
          <strong>How to use:</strong> 
          {!isTextEditing ? (
            <span> Click on text to edit • Double-click for quick edit • Select text with mouse to style specific parts</span>
          ) : (
            <span> Select text with mouse then use toolbar • Ctrl+B (Bold) | Ctrl+I (Italic) | Ctrl+U (Underline) | Esc (Finish)</span>
          )}
        </div>
      </div>

      {/* Selection Info */}
      {selectionRange && isTextEditing && (
        <div className="mt-2 text-center">
          <div className="inline-block bg-yellow-50 px-3 py-1 rounded border border-yellow-200 text-xs text-yellow-800">
            💡 <strong>Tip:</strong> With text selected, use the toolbar buttons or keyboard shortcuts to style just the selected portion
          </div>
        </div>
      )}
    </div>
  );
};

export default Canvas;


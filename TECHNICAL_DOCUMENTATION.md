# Technical Documentation - Nested Drag & Drop List Component

## Overview

A production-ready React component for creating nested lists with full drag-and-drop functionality. Built with TypeScript and Tailwind CSS for type safety and responsive design.

## Features

- **Nested Structure**: Unlimited nesting levels
- **Drag & Drop**: Full HTML5 drag-and-drop with visual feedback
- **Reordering**: Items can be reordered within the same level
- **Cross-level Movement**: Items can be moved between different nesting levels
- **Expand/Collapse**: Collapsible nested items with visual indicators
- **Circular Prevention**: Prevents circular structures (items can't be dropped into descendants)
- **TypeScript**: Full type safety
- **Responsive**: Works on desktop and mobile devices

## Architecture

### Component Structure

```
App
└── NestedList (Main container)
    └── NestedListItem (Recursive item component)
        └── NestedListItem (Children items)
```

### File Organization

```
src/
├── components/
│   ├── NestedList.tsx          # Main list container
│   └── NestedListItem.tsx      # Individual list item
├── hooks/
│   └── useDragAndDrop.ts       # Drag-and-drop logic
├── data/
│   └── sampleData.ts           # Sample data (empty by default)
├── types.ts                    # TypeScript definitions
└── App.tsx                     # Main application
```

## Data Structure

### NestedItem Interface

```typescript
interface NestedItem {
  id: string;           // Unique identifier
  title: string;        // Display text
  children?: NestedItem[]; // Optional nested items
  isExpanded?: boolean; // Optional expand state (default: true)
}
```

### DragState Interface

```typescript
interface DragState {
  draggedItem: NestedItem | null;      // Currently dragged item
  draggedFromPath: number[];           // Original position path
  dragOverPath: number[] | null;       // Current hover target path
  dragOverPosition: 'before' | 'after' | 'inside' | null; // Drop position
}
```

## Core Algorithms

### Path-Based Navigation

Items are identified using array paths representing their position:

```typescript
// Examples:
[0]       // First root item
[0, 2]    // Third child of first root item
[1, 0, 3] // Fourth child of first child of second root item
```

### Item Lookup

```typescript
const findItemByPath = (items: NestedItem[], path: number[]): NestedItem | null => {
  let current = items;
  let item = null;
  
  for (let i = 0; i < path.length; i++) {
    const index = path[i];
    if (index >= current.length) return null;
    
    item = current[index];
    if (i < path.length - 1) {
      current = item.children || [];
    }
  }
  
  return item;
};
```

### Circular Reference Prevention

```typescript
const isDescendant = (ancestorPath: number[], descendantPath: number[]): boolean => {
  if (ancestorPath.length >= descendantPath.length) return false;
  
  for (let i = 0; i < ancestorPath.length; i++) {
    if (ancestorPath[i] !== descendantPath[i]) return false;
  }
  
  return true;
};
```

## Drag and Drop Implementation

### HTML5 Drag and Drop API

The component uses native HTML5 drag-and-drop:

```typescript
// Drag start
onDragStart={(e) => {
  e.dataTransfer.effectAllowed = 'move';
  onDragStart(item, path);
}}

// Drag over with position detection
onDragOver={(e) => {
  e.preventDefault();
  e.stopPropagation();
  onDragOver(path, position);
}}

// Drop handling
onDrop={(e) => {
  e.preventDefault();
  e.stopPropagation();
  onDrop();
}}
```

### Drop Positions

Three drop positions are supported:

1. **Before**: Drop above an item at the same level
2. **After**: Drop below an item at the same level
3. **Inside**: Drop as a child of the target item

### Visual Feedback

```typescript
const getDragOverClass = (position: 'before' | 'after' | 'inside') => {
  switch (position) {
    case 'before':
      return 'border-t-2 border-blue-500';  // Top border
    case 'after':
      return 'border-b-2 border-blue-500';  // Bottom border
    case 'inside':
      return 'bg-blue-50 border border-blue-300'; // Background highlight
  }
};
```

## State Management

### Immutable Updates

All state changes create new objects/arrays:

```typescript
const removeItemByPath = (items: NestedItem[], path: number[]): NestedItem[] => {
  const newItems = [...items]; // Shallow copy
  // ... manipulation logic
  return newItems;
};
```

### Event Flow

1. **Drag Start**: Set dragged item and source path
2. **Drag Over**: Update target path and position
3. **Drop**: Execute move operation and update state
4. **Drag End**: Reset drag state

## Performance Optimizations

### React Optimizations

- **useCallback**: Memoized event handlers prevent unnecessary re-renders
- **Key Props**: Stable `item.id` keys for efficient list updates
- **Immutable Updates**: Prevent deep equality checks

### Drag Performance

- **Event Delegation**: Minimal event listeners through React's synthetic events
- **Throttled Updates**: State updates only on actual position changes
- **Efficient Comparisons**: JSON.stringify for quick path equality checks

## Usage Examples

### Basic Implementation

```tsx
import React, { useState } from 'react';
import { NestedList } from './components/NestedList';
import { NestedItem } from './types';

function App() {
  const [items, setItems] = useState<NestedItem[]>([
    {
      id: '1',
      title: 'Parent Item',
      isExpanded: true,
      children: [
        { id: '1-1', title: 'Child Item 1' },
        { id: '1-2', title: 'Child Item 2' }
      ]
    }
  ]);

  return (
    <NestedList 
      items={items} 
      onItemsChange={setItems} 
    />
  );
}
```

### Adding New Items

```tsx
const addNewItem = () => {
  const newItem: NestedItem = {
    id: `item-${Date.now()}`,
    title: 'New Item',
    isExpanded: true,
  };
  setItems([...items, newItem]);
};
```

## API Reference

### NestedList Props

| Prop | Type | Description |
|------|------|-------------|
| `items` | `NestedItem[]` | Array of nested items to display |
| `onItemsChange` | `(items: NestedItem[]) => void` | Callback when items are reordered |

### NestedListItem Props

| Prop | Type | Description |
|------|------|-------------|
| `item` | `NestedItem` | The item data to display |
| `path` | `number[]` | Path to this item in the tree |
| `level` | `number` | Nesting level (for indentation) |
| `dragState` | `DragState` | Current drag operation state |
| `onToggleExpand` | `(path: number[]) => void` | Toggle expand/collapse |
| `onDragStart` | `(item: NestedItem, path: number[]) => void` | Drag start handler |
| `onDragOver` | `(path: number[], position: string) => void` | Drag over handler |
| `onDrop` | `() => void` | Drop handler |
| `onDragEnd` | `() => void` | Drag end handler |

## Browser Support

- **Desktop**: Chrome 4+, Firefox 3.5+, Safari 3.1+, Edge 12+
- **Mobile**: Limited HTML5 drag support, requires touch polyfills for full functionality

## Error Handling

### Boundary Conditions

- **Empty Lists**: Graceful handling with "No items" message
- **Invalid Paths**: Null checks for safe navigation
- **Missing Children**: Optional chaining for safe access

### Drag Safety

- **Null Checks**: Verify drag state before operations
- **Path Validation**: Ensure paths exist before manipulation
- **Circular Prevention**: Block invalid drop operations

## Testing Strategy

### Unit Tests

- Path algorithms (`findItemByPath`, `removeItemByPath`, `insertItemAtPath`)
- Circular detection (`isDescendant`)
- State management (drag state transitions)

### Integration Tests

- Full drag-and-drop workflows
- Visual feedback correctness
- Data integrity after operations

### Edge Cases

- Single item lists
- Deep nesting performance
- Rapid successive operations

## Customization

### Styling

The component uses Tailwind CSS classes that can be customized:

```typescript
// Basic styling classes used:
'bg-white'           // White background
'border'             // Simple border
'p-2', 'p-4'        // Basic padding
'text-gray-800'      // Dark gray text
'hover:bg-gray-50'   // Light gray hover
'flex items-center'  // Flexbox layout
'rounded'            // Rounded corners
```

### Extending Functionality

- **Custom Item Types**: Extend `NestedItem` interface
- **Additional Drop Positions**: Modify position detection
- **Drag Constraints**: Add validation in drag handlers
- **Persistence**: Add API integration for data saving

## Deployment

The component is ready for production deployment with:

- TypeScript for type safety
- Responsive design
- Cross-browser compatibility
- Performance optimizations
- Error handling
- Clean, maintainable code structure


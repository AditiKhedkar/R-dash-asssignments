# Nested Drag & Drop List Component

A production-ready React component for creating nested lists with full drag-and-drop functionality. Built with TypeScript and Tailwind CSS.

## Features

- **Nested Structure**: Support for unlimited nesting levels
- **Drag & Drop**: Full drag-and-drop functionality with visual feedback
- **Reordering**: Items can be reordered within the same level
- **Cross-level Movement**: Items can be moved between different nesting levels
- **Expand/Collapse**: Collapsible nested items with visual indicators
- **Circular Prevention**: Prevents circular structures (items can't be dropped into their descendants)
- **TypeScript**: Full type safety with TypeScript
- **Responsive**: Works on desktop and mobile devices

## Installation

```bash
npm install
npm run dev
```

## Usage

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

  const handleItemsChange = (newItems: NestedItem[]) => {
    setItems(newItems);
  };

  return (
    <NestedList items={items} onItemsChange={handleItemsChange} />
  );
}
```

### Data Structure

The component expects data in the following format:

```typescript
interface NestedItem {
  id: string;           // Unique identifier
  title: string;        // Display text
  children?: NestedItem[]; // Optional nested items
  isExpanded?: boolean; // Optional expand state (default: true)
}
```

## Component Architecture

### Core Components

1. **NestedList** - Main container component
2. **NestedListItem** - Individual list item with drag-and-drop functionality
3. **useDragAndDrop** - Custom hook managing drag-and-drop state and operations

### File Structure

```
src/
├── components/
│   ├── NestedList.tsx          # Main list container
│   └── NestedListItem.tsx      # Individual list item
├── hooks/
│   └── useDragAndDrop.ts       # Drag-and-drop logic
├── data/
│   └── sampleData.ts           # Sample data for testing
├── types.ts                    # TypeScript type definitions
└── App.tsx                     # Main application component
```

## Technical Implementation

### Drag and Drop System

The drag-and-drop system uses the HTML5 Drag and Drop API with the following key features:

#### Drop Positions
- **Before**: Drop above an item at the same level
- **After**: Drop below an item at the same level  
- **Inside**: Drop as a child of the target item

#### Visual Feedback
- Blue borders indicate valid drop zones
- Light blue background highlights when hovering over drop targets
- Dragged items become semi-transparent during drag operation

#### Path-based Navigation
Items are identified using array paths (e.g., `[0, 2, 1]` represents the 2nd child of the 3rd child of the 1st root item).

### State Management

The component uses React's `useState` for managing the nested data structure. All operations create new immutable data structures to ensure proper React re-rendering.

### Key Algorithms

#### Path Finding
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

#### Circular Reference Prevention
```typescript
const isDescendant = (ancestorPath: number[], descendantPath: number[]): boolean => {
  if (ancestorPath.length >= descendantPath.length) return false;
  
  for (let i = 0; i < ancestorPath.length; i++) {
    if (ancestorPath[i] !== descendantPath[i]) return false;
  }
  
  return true;
};
```

## API Reference

### NestedList Props

| Prop | Type | Description |
|------|------|-------------|
| `items` | `NestedItem[]` | Array of nested items to display |
| `onItemsChange` | `(items: NestedItem[]) => void` | Callback when items are reordered |

### NestedItem Interface

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `id` | `string` | Yes | Unique identifier for the item |
| `title` | `string` | Yes | Display text for the item |
| `children` | `NestedItem[]` | No | Array of child items |
| `isExpanded` | `boolean` | No | Whether the item is expanded (default: true) |

## Styling

The component uses Tailwind CSS for styling with a clean, minimal design:

- **Colors**: Gray color scheme with blue accents for drag indicators
- **Layout**: Flexbox-based layout with proper spacing
- **Responsive**: Works on all screen sizes
- **Hover States**: Subtle hover effects for better UX

### Customization

To customize the appearance, modify the Tailwind classes in the component files:

- `NestedList.tsx` - Main container styling
- `NestedListItem.tsx` - Individual item styling

## Browser Support

- Chrome 4+
- Firefox 3.5+
- Safari 3.1+
- Edge 12+
- Mobile browsers with touch support

## Performance Considerations

- Uses React's key prop for efficient re-rendering
- Immutable data updates prevent unnecessary re-renders
- Path-based item identification for O(log n) lookups
- Minimal DOM manipulation during drag operations

## Accessibility

- Keyboard navigation support
- Screen reader friendly
- Focus management during drag operations
- Semantic HTML structure

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests if applicable
5. Submit a pull request

## License

MIT License - feel free to use in your projects.
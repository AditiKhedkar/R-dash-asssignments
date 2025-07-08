# Simple Drag & Drop List - Easy Documentation

## What This Does
A simple nested list where you can drag items around to reorder them or move them into other items as children.

## How It Works

### 1. Data Structure
Each item looks like this:
```javascript
{
  id: "unique-id",
  title: "Item Name", 
  children: [...], // Optional array of child items
  isExpanded: true // Whether children are shown
}
```

### 2. The Simple Logic

#### Finding Items
```javascript
// Find an item using its path (like [0, 2, 1])
const findItem = (items, path) => {
  let current = items;
  
  for (const index of path) {
    if (path.indexOf(index) === path.length - 1) {
      return current[index]; // Found it!
    }
    current = current[index].children || [];
  }
};
```

#### Moving Items (3 Simple Steps)
1. **Remove** the item from where it was
2. **Adjust** the target position if needed  
3. **Add** the item to the new position

```javascript
// Step 1: Remove from old spot
let newItems = removeItem(items, oldPath);

// Step 2: Adjust target if moving within same parent
if (sameParent && oldIndex < newIndex) {
  newIndex = newIndex - 1; // Account for removed item
}

// Step 3: Add to new spot
newItems = addItem(newItems, item, newPath, position);
```

### 3. Drag and Drop Events

#### When Drag Starts
- Remember what item we're dragging
- Remember where it came from

#### When Dragging Over
- Check if drop is allowed (no loops!)
- Show visual feedback

#### When Dropped
- Do the 3-step move process
- Update the data
- Clear drag state

### 4. Key Safety Checks

#### Prevent Loops
```javascript
// Don't let item be dropped into its own children
const wouldCreateLoop = (dragPath, dropPath) => {
  // If drop path starts with drag path, it's a child
  for (let i = 0; i < dragPath.length; i++) {
    if (dragPath[i] !== dropPath[i]) return false;
  }
  return true;
};
```

## File Structure (Simple)
```
src/
├── components/
│   ├── NestedList.tsx      # Main container
│   └── NestedListItem.tsx  # Each item
├── hooks/
│   └── useDragAndDrop.ts   # All the drag logic
├── types.ts                # Data types
└── App.tsx                 # Main app
```

## How to Use

### Basic Setup
```jsx
function App() {
  const [items, setItems] = useState([
    {
      id: '1',
      title: 'First Item',
      children: [
        { id: '1-1', title: 'Child Item' }
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
```jsx
const addItem = () => {
  const newItem = {
    id: `item-${Date.now()}`,
    title: 'New Item'
  };
  setItems([...items, newItem]);
};
```

## The Magic Explained

### Paths Are Like Addresses
- `[0]` = First item in the list
- `[0, 2]` = Third child of first item  
- `[1, 0, 3]` = Fourth child of first child of second item

### Deep Copy for Safety
```javascript
const newItems = JSON.parse(JSON.stringify(items));
```
This creates a completely new copy so React knows to re-render.

### Three Drop Positions
- **Before**: Drop above an item
- **After**: Drop below an item
- **Inside**: Drop as a child of an item

## Common Gotchas

1. **Always use deep copy** when changing nested data
2. **Adjust indices** when moving within the same parent
3. **Check for loops** before allowing drops
4. **Use stable IDs** for React keys

## That's It!
The core concept is simple: find, remove, adjust, add. Everything else is just React and visual feedback!
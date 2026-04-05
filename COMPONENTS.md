# Component API Documentation

## Overview

The Finance Dashboard is built with a modular component architecture. This document explains each component, its props, and how to use or extend it.

---

## Context & Hooks

### `FinanceContext`

Global state container for the entire application.

**State Structure:**
```javascript
{
  role: 'viewer' | 'admin',
  transactions: Transaction[],
  filters: {
    search: string,
    category: string,
    type: 'all' | 'income' | 'expense'
  }
}
```

### `FinanceProvider`

Wraps the entire app to provide context.

```jsx
<FinanceProvider>
  <Dashboard />
</FinanceProvider>
```

### `useFinance()`

Custom hook to access context anywhere in the app.

**Returns:**
```javascript
{
  role: string,                    // Current user role
  setRole: (role) => void,        // Change role
  transactions: Transaction[],     // All transactions
  addTransaction: (tx) => void,   // Add new transaction
  deleteTransaction: (id) => void, // Delete by ID
  filters: FilterState,           // Current filters
  setFilters: (filters) => void  // Update filters
}
```

**Usage:**
```jsx
function MyComponent() {
  const { transactions, filters, setFilters } = useFinance();
  
  return (
    <div>
      {transactions.map(tx => (...))}
    </div>
  );
}
```

---

## Components

### 1. SummaryCard

Displays a single metric with optional trend indicator.

**Props:**
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| `title` | string | ✓ | Card header text |
| `amount` | number \| string | ✓ | Main value displayed |
| `icon` | Component | ✓ | Lucide icon (e.g., TrendingUp) |
| `trend` | number | | Percentage change (positive/negative) |
| `color` | string | ✓ | Tailwind color class (e.g., 'bg-blue-500') |

**Example:**
```jsx
<SummaryCard
  title="Total Balance"
  amount={5000}
  icon={TrendingUp}
  trend={12.5}
  color="bg-blue-500"
/>
```

**Features:**
- Gradient background with subtle border
- Hover animation on card
- Icon background with transparency
- Trend indicator (↑/↓) with color coding
- Responsive layout

---

### 2. TransactionsTable

Main transactions display with filtering and sorting.

**Internal Hooks:**
- `sortBy` - 'date' or 'amount'
- Computed `filteredTransactions` with memoization

**Features:**
- Real-time search by description
- Category filter dropdown
- Type filter (income/expense/all)
- Sortable columns (Date, Amount)
- Admin-only delete button
- Empty state message

**Filtering Logic:**
```javascript
filteredTransactions = transactions
  .filter(t => {
    if (search && !t.description.includes(search)) return false;
    if (category !== 'all' && t.category !== category) return false;
    if (type !== 'all' && t.type !== type) return false;
    return true;
  })
  .sort(by Date or Amount)
```

**Data Flow:**
1. User types/selects filter
2. `setFilters()` updates context
3. Computed value recalculates
4. Table re-renders with filtered data

---

### 3. AddTransactionModal

Modal form for creating new transactions (Admin only).

**State:**
```javascript
{
  description: string,
  amount: string,
  category: string,
  type: 'income' | 'expense',
  date: string (YYYY-MM-DD)
}
```

**Features:**
- Only visible to Admin role
- Form validation (description + amount required)
- Category dropdown (6 options)
- Type toggle (Income/Expense)
- Date picker with current date default
- Closes on submit or X button

**Form Submission:**
```javascript
const handleSubmit = (e) => {
  e.preventDefault();
  if (formData.description && formData.amount) {
    addTransaction({
      ...formData,
      amount: parseFloat(formData.amount)
    });
    resetForm();
    closeModal();
  }
};
```

**Example Usage:**
```jsx
const [isOpen, setIsOpen] = useState(false);

return (
  <>
    <button onClick={() => setIsOpen(true)}>Add</button>
    <AddTransactionModal 
      isOpen={isOpen} 
      onClose={() => setIsOpen(false)} 
    />
  </>
);
```

---

### 4. Dashboard

Main application component orchestrating all sections.

**Responsibilities:**
- Layout structure
- Summary card calculations
- Chart data preparation
- Header/navigation
- Modal state management
- Analytics computation

**Computed Values:**
```javascript
const income = transactions
  .filter(t => t.type === 'income')
  .reduce((sum, t) => sum + t.amount, 0);

const expenses = transactions
  .filter(t => t.type === 'expense')
  .reduce((sum, t) => sum + t.amount, 0);

const balance = income - expenses;

const categorySpending = transactions
  .filter(t => t.type === 'expense')
  .reduce((acc, t) => {
    // Group by category
  });
```

---

## Data Models

### Transaction
```typescript
interface Transaction {
  id: number,
  date: string,            // YYYY-MM-DD format
  description: string,
  amount: number,
  category: string,
  type: 'income' | 'expense'
}
```

### Filter State
```typescript
interface FilterState {
  search: string,
  category: string,        // 'all' or specific category
  type: 'all' | 'income' | 'expense'
}
```

---

## Charts

### Balance Trend (LineChart)

**Data Format:**
```javascript
[
  { month: 'Nov', balance: 4200 },
  { month: 'Dec 1-5', balance: 4800 },
  ...
]
```

**Customization:**
```jsx
<LineChart data={balanceTrendData}>
  <CartesianGrid strokeDasharray="3 3" stroke="#334155" />
  <XAxis dataKey="month" stroke="#94a3b8" />
  <YAxis stroke="#94a3b8" />
  <Tooltip contentStyle={{...}} />
  <Line 
    type="monotone" 
    dataKey="balance" 
    stroke="#3b82f6"
    strokeWidth={2}
    dot={{ fill: '#3b82f6', r: 5 }}
  />
</LineChart>
```

### Spending by Category (PieChart)

**Data Format:**
```javascript
[
  { name: 'Food', value: 125.50 },
  { name: 'Transport', value: 45.00 },
  ...
]
```

**Colors:**
- Green: #10b981
- Blue: #3b82f6
- Amber: #f59e0b
- Red: #ef4444
- Purple: #8b5cf6
- Pink: #ec4899

---

## Styling System

### Color Palette

**Primary:**
- bg-blue-600 / text-blue-500 - Primary actions
- bg-emerald-500 / text-emerald-400 - Income, positive
- bg-red-500 / text-red-400 - Expenses, negative

**Background:**
- bg-slate-900 - Main background
- bg-slate-800 - Card backgrounds
- bg-slate-700 - Input/hover states

**Text:**
- text-white - Primary text
- text-slate-400 - Secondary text
- text-slate-500 - Tertiary text

### Component Patterns

**Card Pattern:**
```jsx
<div className="bg-gradient-to-br from-slate-800 to-slate-900 rounded-2xl p-6 border border-slate-700 hover:border-slate-600 transition-all">
  {/* Content */}
</div>
```

**Input Pattern:**
```jsx
<input 
  className="px-4 py-2 bg-slate-700 border border-slate-600 rounded-lg text-white placeholder-slate-500 focus:outline-none focus:border-blue-500 focus:ring-1 focus:ring-blue-500"
/>
```

**Badge Pattern:**
```jsx
<span className="px-3 py-1 rounded-full text-xs font-medium bg-emerald-500/20 text-emerald-300">
  Income
</span>
```

---

## State Management Patterns

### Adding Data
```javascript
const addTransaction = (transaction) => {
  const newTransaction = {
    ...transaction,
    id: Math.max(...transactions.map(t => t.id), 0) + 1,
  };
  setTransactions([newTransaction, ...transactions]);
};
```

**Pattern:** Place new items at the start of array for recency.

### Deleting Data
```javascript
const deleteTransaction = (id) => {
  setTransactions(transactions.filter(t => t.id !== id));
};
```

**Pattern:** Immutable filter operation.

### Filtering Data
```javascript
const filteredTransactions = useMemo(() => {
  return transactions
    .filter(/* condition */)
    .sort(/* comparator */);
}, [transactions, filters, sortBy]);
```

**Pattern:** Memoize computed values to avoid recalculation.

---

## Performance Optimizations

### 1. useMemo for Filtered Data
```javascript
const filteredTransactions = useMemo(() => {
  // Only recalculate when dependencies change
  return transactions.filter(...).sort(...);
}, [transactions, filters, sortBy]);
```

### 2. Computed Analytics
```javascript
const income = useMemo(() => {
  return transactions
    .filter(t => t.type === 'income')
    .reduce((sum, t) => sum + t.amount, 0);
}, [transactions]);
```

### 3. Reusable Components
- `SummaryCard` used 3 times for different metrics
- Reduces code duplication and bundle size

---

## Extending the Dashboard

### Adding a New Insight Card

1. **Compute the value:**
```javascript
const avgTransaction = useMemo(() => {
  return transactions.length > 0 
    ? transactions.reduce((sum, t) => sum + t.amount, 0) / transactions.length
    : 0;
}, [transactions]);
```

2. **Create the card:**
```jsx
<div className="bg-gradient-to-br from-slate-800 to-slate-900 rounded-2xl p-6 border border-slate-700">
  <h3 className="text-slate-400 text-sm font-medium mb-2">Average Transaction</h3>
  <p className="text-2xl font-bold text-white">${avgTransaction.toFixed(2)}</p>
</div>
```

### Adding a New Chart

1. **Prepare data:**
```javascript
const monthlyData = [
  { month: 'Jan', income: 5000, expense: 2000 },
  // ...
];
```

2. **Use Recharts:**
```jsx
<ResponsiveContainer width="100%" height={250}>
  <BarChart data={monthlyData}>
    <Bar dataKey="income" fill="#10b981" />
    <Bar dataKey="expense" fill="#ef4444" />
  </BarChart>
</ResponsiveContainer>
```

### Adding Persistence

```javascript
useEffect(() => {
  localStorage.setItem('finance_transactions', JSON.stringify(transactions));
}, [transactions]);

useEffect(() => {
  const saved = localStorage.getItem('finance_transactions');
  if (saved) setTransactions(JSON.parse(saved));
}, []);
```

---

## Testing Checklist

- [ ] Filters work in all combinations
- [ ] Sorting toggles between ascending/descending
- [ ] New transactions appear immediately
- [ ] Deletes update all displays (totals, charts, table)
- [ ] Role switch hides/shows admin features
- [ ] Modal validates required fields
- [ ] Charts update with new data
- [ ] Responsive on all breakpoints
- [ ] No console errors

---

## Common Issues & Solutions

| Issue | Solution |
|-------|----------|
| Charts not updating | Check `data` prop is changing, verify Recharts version |
| Filters not working | Verify filter state updates, check filter conditions |
| Role not switching | Ensure `setRole` is called, page refreshes state |
| Modal won't open/close | Check `isOpen` prop and `onClose` handler |
| Cards misaligned | Verify Tailwind CSS loaded, check grid classes |

---

## References

- [React Hooks Docs](https://react.dev/reference/react)
- [Tailwind CSS Classes](https://tailwindcss.com/docs)
- [Recharts Documentation](https://recharts.org/)
- [Lucide Icons](https://lucide.dev/)

---

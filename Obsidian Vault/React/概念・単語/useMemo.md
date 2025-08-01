useMemo is [[hook]].useMemo does read of [[cache]]

useMemo is doesn't calculation as long as the variable or situation doesn't change.

**use timing**
1,The heavy calculations
Sorting,firtering,big loops,chart data conversion,etc.
Example
```tsx
const sorted = useMemo(() => data.sort(),[data])
```
**Other**
pass props to child [[component]]-Prevent re-renders when object reference changes.

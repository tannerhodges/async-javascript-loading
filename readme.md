# 👨‍🔬 Async JavaScript Loading

How does loading scripts differently affect the order they run in?

- **Red** = Source Order.
- **Green** = Load Order.

## Basic Test

`index.html`

```diff
  1. head inline
  2. head external
  3. head inline
  4. head inline async
- 5. head external async
  6. head inline async
  7. head inline defer
- 8. head external defer
  9. head inline defer
  -- end of head --
  10. body inline
  11. body external
  12. body inline
  13. body inline async
- 14. body external async
  15. body inline async
  16. body inline defer
- 17. body external defer
  18. body inline defer
  -- end of body --
+ 5. head external async
+ 8. head external defer
+ 14. body external async
+ 17. body external defer
```

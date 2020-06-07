# 👨‍🔬 Async JavaScript Loading

How does loading scripts differently affect the order they run in?

## Tests

### Basic

`basic/index.html`

- **Red** = Source Order.
- **Green** = Load Order.

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

### Weighted

`weighted/index.html`

Now it gets interesting...

- If the weights are the same, the order remains consistent.
- `async`/`defer` only affects external files.
	- "Classic scripts are affected by the async and defer attributes, but only when the src attribute is set." —[HTML Spec](https://html.spec.whatwg.org/multipage/scripting.html#script)
- Heavier render-blocking scripts give in-flight scripts more time to load. For example, an `async` script in the `head` can finish loading and execute before other scripts in the `body`.
- As soon as you start loading over HTTP, `async` varies like crazy. Literally, it can load _whenever_. Results vary, even on a local server with caching disabled and no 3rd party code (except Browsersync).
- One thing is certain: 17 will always fire after 8. Otherwise, 5 and 14 can show up in any order.

### Dynamic

`dynamic/index.html`

- Dynamically loaded scripts will always execute after scripts listed in the markup.
- The "source" order you add a dynamic script to doesn't matter at all. All that matters is the order you request them in and whether or not they're `async`.
- If you want to dynamically load a script before other scripts execute (say, conditionally loading a polyfill), you have to dynamically load and defer the other scripts too. (The only other way, as far as I can tell, is to sticky with render-blocking scripts.)

## Possible Execution Orders

If all scripts were `defer` there would only be 1 possible execution order.

But as soon as we throw `async` in the mix, scripts can literally load in any order. They may converge on the most likely order (the "standard" order), but there's no guarantee.

If we give each external script a number (1–4), we can generate a list of all _possible_ orders these scripts can run in (thanks to https://www.free-online-calculator-use.com/combination-calculator.html).

With 4 numbers, there are 4! (4 * 3 * 2 * 1) or 24 possible combinations.

But since some scripts use `defer`, we know 4 will always run after 2.

With a little regex (`.*4.*2`) we can remove all the combinations that where 4 loads before 2.

Now there are only 12 combinations:

```
1 2 3 4
1 2 4 3
1 3 2 4
2 1 3 4
2 1 4 3
2 3 1 4
2 3 4 1
2 4 1 3
2 4 3 1
3 1 2 4
3 2 1 4
3 2 4 1
```

Ideally, I'd run an automated test suite to do at least 100 (if not 500+) runs for each test and get statistics on the likelihood of each combination.

Based on my limited manual tests tonight, it seems like:

- `1 2 4 3` is the most common ("standard") order.
	- `head:async`, `head:defer`, `body:defer`, `body:async`
- Other combinations vary based on file size, latency, etc.

## Questions

1. Why is this the standard order?
2. What exactly causes the variation in `async` loading?
3. Why does `body:async` tend to fire after `body:defer`?
4. Why not just `defer` everything?
5. Why exactly do dynamically added scripts always run after scripts in the source code, regardless of position?

# Finding Format

A pass returns its findings as one message, grouped by file.

## Template

````md
**{path}**
- **possible {Smell}** in `{function or class}`: {why the smell applies here}. → {the fix, in one line}.
  ```{language}
  {the 1 to 3 lines that show it, verbatim}
  ```
````

## Example

````md
**src/render.ts**
- **Bug** in `render()`: an empty `items` throws on the first line. → Return the empty state before reading `items[0]`.
  ```ts
  const heading = items[0].title;
  ```
- **possible Flag Argument** in `render()`: `compact` switches between two layouts. → Split into `renderCompact(items)` and `renderFull(items)`.
  ```ts
  function render(items: Item[], compact: boolean) {
  ```
````

## Rules

- **Bugs** come first in their file, labelled `**Bug**`.
- **The label** of every other finding is `possible {Smell}`, or `possible {Principle} violation` for Command-Query Separation and the Dependency Rule.
- **Repeats** of one smell across functions form one finding that names every location.
- **A contract change** says so after the arrow, and names the callers it touches.
- **The quote** is left out for smells about missing code (Insufficient Tests, Missing Why).
- **A pass with no findings** returns `clean`.

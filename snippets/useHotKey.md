---
title: React useHotKey hook
tags: hook,state,effect,memo
expertise: intermediate
firstSeen: 2022-06-22T05:00:00-04:00
---

Listen and trigger hotkeys you defined.

- the hook receive `hotkey` string as when to trigger, the `callback` function as how to trigger.
- use `keydown`/`keyup` event to listen what pressed, use a `stack` structure to store pressed key.
- when `keydown` event trigger, add the keycode to a stack(maintained by a class, use Set as inner dataStructure to exclude duplication).
- use a `isValid` Getter to auto calculate whether keys matched the hotkeys.
- if matched, `callback` will fire and the keycode `stack` will reset.

```jsx
// keycode alias predefined
const HotkeyDict = {
  backspace: 8,
  tab: 9,
  enter: 13,
  shift: 16,
  ctrl: 17,
  alt: 18,
  command: 91,
  "caps-lock": 20,
  escape: 27,
  space: 32,
  "page-down": 34,
  end: 35,
  "arrow-left": 37,
  "arrow-up": 38,
  "arrow-right": 39,
  "arrow-down": 40,
  insert: 45,
  delete: 46,
  0: 48,
  1: 49,
  2: 50,
  3: 51,
  4: 52,
  5: 53,
  6: 54,
  7: 55,
  8: 56,
  9: 57,
  "numpad-0": 96,
  "numpad-1": 97,
  "numpad-2": 98,
  "numpad-3": 99,
  "numpad-4": 100,
  "numpad-5": 101,
  "numpad-6": 102,
  "numpad-7": 103,
  "numpad-8": 104,
  "numpad-9": 105,
  a: 65,
  b: 66,
  c: 67,
  d: 68,
  e: 69,
  f: 70,
  g: 71,
  h: 72,
  i: 73,
  j: 74,
  k: 75,
  l: 76,
  m: 77,
  n: 78,
  o: 79,
  p: 80,
  q: 81,
  r: 82,
  s: 83,
  t: 84,
  u: 85,
  v: 86,
  w: 87,
  x: 88,
  y: 89,
  z: 90,
};
// use Stack to maintain pressed keycode
class KeyManager {
  queue;
  target;
  constructor(combineAlias, hotKeyDict = HotkeyDict) {
    this.queue = new Set();
    this.target = KeyManager.extractKeyAlias(combineAlias, hotKeyDict);
  }
  // whether keycode list is matched hotkey alias
  get isValid() {
    return (
      this.target.every((targetKeyCode) => this.queue.has(targetKeyCode)) &&
      this.queue.size === this.target.length
    );
  }
  add(keyCode) {
    this.queue.add(keyCode);
  }
  remove(keyCode) {
    this.queue.delete(keyCode);
  }
  reset() {
    this.queue.clear();
  }

  static extractKeyAlias(combineAlias, hotKeyDict) {
    return combineAlias
      .replace(/\s/g, "")
      .split("+")
      .map((keyAlias) => {
        if (isNaN(hotKeyDict[keyAlias])) {
          throw new Error("nonsupport key alias");
        }
        return hotKeyDict[keyAlias];
      });
  }
}
/**
 *
 * @param combineAlias key + key + key + ... + key
 * @param callback execute when combineAlias is matched
 */
function useHotKey(combineAlias, callback) {
  // when hotkey changes, keyManager should reCreate
  const keyManager = React.useMemo(() => {
    return new KeyManager(combineAlias);
  }, [combineAlias]);
  React.useEffect(() => {
    function handleKeyDown(e) {
      keyManager.add(e.keyCode);
      if (keyManager.isValid) {
        callback?.();
      }
    }
    function handleKeyUp(e) {
      keyManager.remove(e.keyCode);
    }
    document.addEventListener("keydown", handleKeyDown);
    document.addEventListener("keyup", handleKeyUp);
    return () => {
      document.removeEventListener("keydown", handleKeyDown);
      document.removeEventListener("keyup", handleKeyUp);
    };
  }, [keyManager, callback]);
  return keyManager;
}

// usages

const useKeyPress = (targetKey) => {
  const [keyPressed, setKeyPressed] = React.useState(false);

  const downHandler = ({ key }) => {
    if (key === targetKey) setKeyPressed(true);
  };

  const upHandler = ({ key }) => {
    if (key === targetKey) setKeyPressed(false);
  };

  React.useEffect(() => {
    window.addEventListener("keydown", downHandler);
    window.addEventListener("keyup", upHandler);

    return () => {
      window.removeEventListener("keydown", downHandler);
      window.removeEventListener("keyup", upHandler);
    };
  }, []);

  return keyPressed;
};

const MyApp = () => {
  const [count, setCount] = React.useState(0);
  const hotKeyCallback = React.useCallback(() => {
    setCount((prev) => prev + 1);
  }, [count]);

  const [count2, setCount2] = React.useState(0);
  const hotKey2Callback = React.useCallback(() => {
    setCount2((prev) => prev + 1);
  }, [count2]);

  useHotKey("ctrl+w", hotKeyCallback);

  useHotKey("ctrl+shift+w", hotKey2Callback);

  return (
    <>
      <p>Press ctrl+w {count} time </p>
      <p>Press ctrl+shift+f {count2} time </p>
    </>
  );
};
```

```jsx
ReactDOM.render(<MyApp />, document.getElementById("root"));
```

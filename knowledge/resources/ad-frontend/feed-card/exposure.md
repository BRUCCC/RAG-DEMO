# 卡片曝光

信息流滚动很快，组件刚挂载或只闪过一条边，都不能算有效曝光。

## 判定

同时满足下面三条才上报一次曝光：

- 卡片可见面积达到一半，也就是 IntersectionObserver 的 threshold 取 0.5。
- 达到一半之后连续停留一秒。中途掉到一半以下，计时作废。
- 同一次挂载只上报一次。滑出再滑回不再重复上报。

## 示例

```jsx
import { useEffect, useRef } from "react";

// 露出一半并停留一秒，才通知上层记一次曝光。
export function useHalfSecondExposure(onExpose) {
  const ref = useRef(null);

  useEffect(() => {
    const node = ref.current;
    if (!node) return undefined;
    let timer = 0;
    let reported = false;
    const observer = new IntersectionObserver(
      ([entry]) => {
        if (reported) return;
        if (entry.intersectionRatio >= 0.5) {
          timer = window.setTimeout(() => {
            reported = true;
            onExpose();
          }, 1000);
        } else {
          window.clearTimeout(timer);
        }
      },
      { threshold: [0, 0.5, 1] }
    );
    observer.observe(node);
    return () => {
      window.clearTimeout(timer);
      observer.disconnect();
    };
  }, [onExpose]);

  return ref;
}
```

上报字段至少带上 creativeId、slotId 和 requestId。事件名使用 impression，字段含义见监测目录，卡片里不要另起事件名。

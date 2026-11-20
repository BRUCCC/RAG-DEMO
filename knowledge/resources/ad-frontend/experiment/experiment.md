# 广告位实验

一个广告位同时只挂一个进行中的实验。服务端下发实验号、分组号和变体名，前端不要自己随机抽签，否则刷新就会换组。

## 分桶展示

```jsx
// 按分组号选择卡片样式。埋点始终带上同一次下发的实验号。
export function ExperimentalSlot({ assignment, creative, onTrack }) {
  const Variant = assignment.variant === "variant" ? NewAdCard : OldAdCard;
  const extra = {
    experimentId: assignment.experimentId,
    bucket: assignment.bucket,
  };

  return (
    <Variant
      creative={creative}
      onExpose={() => onTrack("impression", extra)}
      onClick={() => onTrack("click", extra)}
    />
  );
}
```

`assignment` 为空时渲染默认卡片，并且不要附带实验号。分组号用服务端字段 `bucket`，界面上不要展示给用户。

变体之间保持同一套 props：creative、onExpose、onClick。不要为新样式另做一套回调名字。


#DoTween

### 呼吸效果(scale 变大变小)
```CS
if (isScalingUp)
{
	sun.rectTransform.DOScale(scaleUpTo, duration).OnComplete(() => isScalingUp = false);
}
else
{
	sun.rectTransform.DOScale(scaleDownTo, duration).OnComplete(() => isScalingUp = true);
}
```

### 上下随机来回移动效果
```CS
ball1.rectTransform.DOShakePosition(duration: 2f, strength: new Vector3(0, 20f, 0), vibrato: 1, randomness: 1, snapping: false, fadeOut: true).SetLoops(-1, LoopType.Yoyo); 
```
- DOShakePosition：用于在指定的持续时间内摇晃目标的当前位置。

- duration：摇晃的持续时间。

- strength：摇晃的强度，这里只在x轴上设置了10f。

- vibrato：振动的次数。

- randomness：随机性，这里设置为0表示没有随机性。

- SetLoops(-1, LoopType.Yoyo)：设置为无限循环，并且以Yoyo的方式来回摇晃。
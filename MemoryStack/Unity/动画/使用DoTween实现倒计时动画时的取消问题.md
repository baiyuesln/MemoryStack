#DoTween  

在Tween动画进行中,有时候我们需要随机中断并隐藏,并且下次调用时从新开始,这两个操作间隔可能很短,如果下次调用时,上次的隐藏没有中断,就会出现bug

所以这里我们可以添加一个取消令牌来控制动画的中断,给Tween进程添加一个令牌,如果需要中断时,取消这个令牌,然后tween就会发现令牌为空之后报错,然后被捕获,然后再杀掉这个被捕获的tween,就完成的清理工作

```cs
private CancellationTokenSource _cts;

    public async UniTask PlayCountdown()
    {
        StopCountDown();
        _cts = new CancellationTokenSource();

        try
        {
            gameObject.SetActive(true);
            canvasGroup.alpha = 1;

            for (int i = 0; i < countdownSprites.Length; i++)
            {
                _cts.Token.ThrowIfCancellationRequested();

                countdownImage.sprite = countdownSprites[i];
                countdownImage.transform.localScale = Vector3.one * 2f;
                countdownImage.color = Color.white;

                // 同时执行缩放和淡出效果
                await UniTask.WhenAll(
                    countdownImage.transform.DOScale(1f, numberDuration).SetEase(Ease.OutBack).AsyncWaitForCompletion().AsUniTask().AttachExternalCancellation(_cts.Token),
                    countdownImage.DOFade(0, numberDuration).SetEase(Ease.InQuad).AsyncWaitForCompletion().AsUniTask().AttachExternalCancellation(_cts.Token)
                );

                await UniTask.Delay((int)(delayBetweenNumbers * 1000), cancellationToken: _cts.Token);
            }

        }
        catch (OperationCanceledException)
        {
            // 处理取消操作
            DOTween.Kill(countdownImage.transform);
            DOTween.Kill(countdownImage);
        }
        finally
        {
            gameObject.SetActive(false);
            _cts?.Dispose();
            _cts = null;
        }

    }

    private void OnDisable()
    {
        StopCountDown();
    }

    public void StopCountDown()
    {
        _cts?.Cancel();
        _cts?.Dispose();
        _cts = null;
        gameObject.SetActive(false);
    }
```

主要改动：

1. 添加了取消令牌 CancellationTokenSource
2. 添加了 StopCountdown 方法用于中断动画
3. 使用 try-catch-finally 处理取消操作
4. 在 OnDisable 中自动停止动画
5. 每次开始前都会停止之前的动画
6. 添加了 DOTween 的清理操作
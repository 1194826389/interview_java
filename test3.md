# 标题1


```java
class="lang-objc">#import "UIkit"
-(void)scrollViewDidScroll:(UIScrollView *)scrollView
{
    
    CGFloat scale = scrollView.contentOffset.x / scrollView.frame.size.width;
    //获得左边的label
    NSInteger leftIndex = scale;
    NEHHomeLabel *leftLabel = self.titleScrollView.subviews[leftIndex];
    
    //获得右边的label
    NSInteger rightIndex = 1 + leftIndex;
    NEHHomeLabel *rightLabel = (rightIndex == self.titleScrollView.subviews.count)?nil:self.titleScrollView.subviews[rightIndex];
    
    
    // 算出比例值
    CGFloat rightScale = scale - leftIndex;
    
    CGFloat leftScale = 1 - rightScale;
    
    
    leftLabel.scale = leftScale;
    rightLabel.scale = rightScale;
    
    
    // 黑色变红色
    //         R  G  B
    //  黑色    0  0  0
    //  红色    1  0  0
    //  绿色    0  1  0
    //  蓝色    0  0  1
}    
```


``` c
void quick_sort(int* a, int l, int r)
{
    if (l < r)
    {
        int i = l;
        int j = r;
        int k = a[i];
        while (i < j)
        {
            while(i < j && a[j] > k)
                j--; // 从右向左找第一个小于k的数
            if(i < j)
                a[i++] = a[j];
            while(i < j && a[i] < k)
                i++; // 从左向右找第一个大于k的数
            if(i < j)
                a[j--] = a[i];
        }
        a[i] = k;
        quick_sort(a, l, i-1);
        quick_sort(a, i+1, r);
    }
}

```

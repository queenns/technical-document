# 堆
## 基础数据
- 完全二叉树
- 数组(内存具有连续性,访问速度更快)

* parent = (i - 1) / 2
* left = 2 * i + 1
* right = 2 * i + 2
* 下沉:pop
* 下浮:push
* O(logN)
## 大堆,根为最大值
```java
public class MaxHeap {
    private int[] elements;
    public int size = 0;

    void sink(int index) {
        int j;
        int t = elements[index];
        while ((j = (index << 1) + 1) < size) {
            if (j < size - 1 && elements[j] < elements[j + 1]) {
                j++;
            }
            if (elements[j] > t) {
                elements[index] = elements[j];
                index = j;
            } else {
                break;
            }
        }
        elements[index] = t;
    }

    void swim(int index) {
        int parent;
        int t = elements[index];

        while (index > 0) {
            parent = (index - 1) >> 1;
            if (elements[parent] < t) {
                elements[index] = elements[parent];
                index = parent;
            } else {
                break;
            }
        }
        elements[index] = t;
    }

    void push(int val) {
        elements[size++] = val;
        swim(size - 1);
    }

    int pop() {
        int t = elements[0];
        elements[0] = elements[--size];
        sink(0);
        return t;
    }

    public int[] getLeastNumbers(int[] array, int k) {
        if (k <= 0 || array == null || array.length == 0) {
            return new int[0];
        }

        elements = new int[k + 1];

        for (int element : array) {
            push(element);
            if (size > k) {
                pop();
            }
        }

        int[] results = new int[k];
        int index = 0;
        while (size > 0) {
            results[index++] = pop();
        }

        return results;
    }

    public static void main(String[] args) {
        int[] A = new int[]{9, 2, 4, 10, 3, 2, 1};
        MaxHeap maxHeap = new MaxHeap();
        int[] leastNumbers = maxHeap.getLeastNumbers(A, 4);
        System.out.println(Arrays.toString(leastNumbers));
    }

}
```
## 小堆,根为最小值
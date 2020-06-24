```java
public class App2 {

    public static void main(String[] args) {
        int[] arr = {8, 4, 5, 7, 1, 3, 6, 2};
        int[] temp = new int[arr.length];
        fork(arr, 0, arr.length - 1, temp);
        System.out.println(Arrays.toString(arr));
    }

    public static void fork(int[] arr, int left, int right, int[] temp) {
        if (left < right) {
            int mid = (left + right) / 2;
            fork(arr, left, mid, temp);
            fork(arr, mid + 1, right, temp);
            join(arr, left, right, mid, temp);
        }
    }

    public static void join(int[] arr, int left, int right, int mid, int[] temp) {
        int i = left, j = mid + 1, x = 0;
        for (; i <= mid && j <= right; ) {
            if (arr[i] <= arr[j]) {
                temp[x++] = arr[i++];
            } else {
                temp[x++] = arr[j++];
            }
        }

        for (; i <= mid; ) temp[x++] = arr[i++];
        for (; j <= right; ) temp[x++] = arr[j++];

        System.out.println("temp\t" + Arrays.toString(temp));

        x = 0;
        for (int tmpLeft = left; tmpLeft <= right; ) {
            arr[tmpLeft++] = temp[x++];
        }

        System.out.println("arr\t\t" + Arrays.toString(arr));
        System.out.println();
    }
}

// 控制台输出
temp	[4, 8, 0, 0, 0, 0, 0, 0]
arr		[4, 8, 5, 7, 1, 3, 6, 2]

temp	[5, 7, 0, 0, 0, 0, 0, 0]
arr		[4, 8, 5, 7, 1, 3, 6, 2]

temp	[4, 5, 7, 8, 0, 0, 0, 0]
arr		[4, 5, 7, 8, 1, 3, 6, 2]

temp	[1, 3, 7, 8, 0, 0, 0, 0]
arr		[4, 5, 7, 8, 1, 3, 6, 2]

temp	[2, 6, 7, 8, 0, 0, 0, 0]
arr		[4, 5, 7, 8, 1, 3, 2, 6]

temp	[1, 2, 3, 6, 0, 0, 0, 0]
arr		[4, 5, 7, 8, 1, 2, 3, 6]

temp	[1, 2, 3, 4, 5, 6, 7, 8]
arr		[1, 2, 3, 4, 5, 6, 7, 8]

[1, 2, 3, 4, 5, 6, 7, 8]
```


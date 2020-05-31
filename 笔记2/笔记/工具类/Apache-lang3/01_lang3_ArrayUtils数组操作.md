---
2019-07-29 15:38:32

---





## 1. add添加元素返回新集合

```java
@Test
public void testArrayUtils(){
    int [] arr = new int[]{1,2};
    System.out.println("array = " + Arrays.toString(arr));
    // 添加一个新元素并返回新集合
    int[] newArr = ArrayUtils.add(arr, 3);
    System.out.println("newArr = " + Arrays.toString(newArr));
}
```

输出：

```
array = [1, 2]
newArr = [1, 2, 3]
```



## 2. addAll合并数组或copy操作

```java
@Test
public void testArrayUtils(){
    // 合并数组，可以是null，如果只有一个数组则为copy操作
    int[] arr = new int[]{1,2};
    // copy操作
    int[] copyArr = ArrayUtils.addAll(arr);
    // 添加所有操作
    int[] allArr = ArrayUtils.addAll(arr, copyArr);
    System.out.println("arr = " + Arrays.toString(arr));
    System.out.println("copyArr = " + Arrays.toString(copyArr));
    System.out.println("allArr = " + Arrays.toString(allArr));
}
```

输出：

```
arr = [1, 2]
copyArr = [1, 2]
allArr = [1, 2, 1, 2]
```



## 3. clone克隆

```java
@Test
public void testCloneArrayList(){
    int[] arr = new int[]{1,2,3};
    int[] clone = ArrayUtils.clone(arr);
}
```



## 4. contains

```java
@Test
public void testContains(){
    int[] arr = new int[]{1,2,3};
    System.out.println("ArrayUtils.contains(arr,1) = " + ArrayUtils.contains(arr, 1));
}
```



## 5. hashCode

```java
@Test
public void testHashCode(){
    int[] arr = new int[]{1,2,3};
    System.out.println("ArrayUtils.hashCode(arr) = " + ArrayUtils.hashCode(arr));
}
```



## 6. insert

```java
@Test
public void testInsert(){
    int[] arr = new int[]{1,2,3};
    int[] arr2= new int[]{4,5,6};
    // arr第二个位置开始插入数组arr2
    int[] insert = ArrayUtils.insert(1, arr, arr2);
    System.out.println("arr = " + Arrays.toString(arr));
    System.out.println("insert = " + Arrays.toString(insert));
}
```

结果：

```
arr = [1, 2, 3]
insert = [1, 4, 5, 6, 2, 3]
```



## 7. 判空

null和{}，即null或size为0的，是空。

```java
@Test
public void testEmpty(){
    // isEmpty && isNotEmpty
    int[] arr1 = null;
    int[] arr2 = {};
    System.out.println("ArrayUtils.isEmpty(arr1) = " + ArrayUtils.isEmpty(arr1));
    System.out.println("ArrayUtils.isEmpty(arr2) = " + ArrayUtils.isEmpty(arr2));
}
```



## 8. isSameLength判断长度是否一致

```java
@Test
public void testSameLength(){
    int[] arr1 = new int[]{1,2,3,4,5};
    int[] arr2 = new int[]{2,3,4,5,6};
    System.out.println("ArrayUtils.isSameLength(arr1,arr2) = " + ArrayUtils.isSameLength(arr1, arr2));
}
```



## 9. NullToEmpty

null转[]

```java
@Test
public void testNulltoEmpty(){
    // null转[]
    int[] arr = null;
    System.out.println("Arrays.toString(arr) = " + Arrays.toString(arr));
    int[] newArr = ArrayUtils.nullToEmpty(arr);
    System.out.println("Arrays.toString(arr) = " + Arrays.toString(newArr));
}
```

结果：

```
Arrays.toString(arr) = null
Arrays.toString(arr) = []
```



## 10. remove和removeAll删除并返回新数组

```java
@Test
public void testRemove(){
    int[] arr = {1, 2, 3, 4};
    System.out.println("Arrays.toString(arr) = " + Arrays.toString(arr));
    int[] remove = ArrayUtils.remove(arr, 1);
    System.out.println("Arrays.toString(remove) = " + Arrays.toString(remove));
    int[] removeAll = ArrayUtils.removeAll(arr, 1, 2, 3);
    System.out.println("Arrays.toString(removeAll) = " + Arrays.toString(removeAll));
}
```

结果：

```
Arrays.toString(arr) = [1, 2, 3, 4]
Arrays.toString(remove) = [1, 3, 4]
Arrays.toString(removeAll) = [1]
```



## 11. reverse逆序

```java
@Test
public void testReverse(){
    int[] arr = {1,2,3,4};
    ArrayUtils.reverse(arr);
    System.out.println("Arrays.toString(arr) = " + Arrays.toString(arr));
    // index从1开始到3之间，即第二个位置和第三个位置逆序 左闭右开[)
    ArrayUtils.reverse(arr,1,3);
    System.out.println("Arrays.toString(arr) = " + Arrays.toString(arr));
}
```

结果：

```
Arrays.toString(arr) = [4, 3, 2, 1]
Arrays.toString(arr) = [4, 2, 3, 1]
```



## 12. shuffle乱序

```java
@Test
public void testShuffle(){
    int[] arr = {1,2,3,4,5};
    ArrayUtils.shuffle(arr);
    System.out.println("Arrays.toString(arr) = " + Arrays.toString(arr));
}
```

结果：

```
Arrays.toString(arr) = [2, 3, 4, 5, 1]
```



## 13. subarray截取数组

```java
@Test
public void testSubarray(){
    int[] arr = {1,2,3,4,5,6,7};
    System.out.println("Arrays.toString(arr) = " + Arrays.toString(arr));
    // 截取数组 左闭右开 [)
    int[] subarray = ArrayUtils.subarray(arr, 1, 3);
    System.out.println("Arrays.toString(subarray) = " + Arrays.toString(subarray));
}
```

结果：

```
Arrays.toString(arr) = [1, 2, 3, 4, 5, 6, 7]
Arrays.toString(subarray) = [2, 3]
```



## 14. swap指定位置交换数据

```java
@Test
public void testSwap(){
    // 指定位置交换数据 两边都是闭  [2,5]
    int[] arr = {1,2,3,4,5,6,7,8};
    System.out.println("Arrays.toString(arr) = " + Arrays.toString(arr));
    ArrayUtils.swap(arr,2,5);
    System.out.println("Arrays.toString(arr) = " + Arrays.toString(arr));
}
```

结果：

```
Arrays.toString(arr) = [1, 2, 3, 4, 5, 6, 7, 8]
Arrays.toString(arr) = [1, 2, 6, 4, 5, 3, 7, 8]
```



## 15. toArray创建新数组

```java
@Test
public void testToArray(){
    String[] arr = ArrayUtils.toArray("大", "中", "小");
    System.out.println("Arrays.toString(arr) = " + Arrays.toString(arr));
}
```



## 16. toPrimitive包装类转基本数据类型

```java
@Test
public void testToPrimitive(){
    Integer[] arr = {1, 2, 3, 4, 5, 6, 7};
    // 包装类转基本数据类型
    int[] intArr = ArrayUtils.toPrimitive(arr);
    System.out.println("Arrays.toString(intArr) = " + Arrays.toString(intArr));
}
```



## 17. toString

```java
@Test
public void testToString(){
    int[] arr = {1,2,3,4,5,6};
    String str = ArrayUtils.toString(arr);
    System.out.println(str);
}
```
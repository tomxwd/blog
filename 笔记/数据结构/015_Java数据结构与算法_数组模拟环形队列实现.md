---
2019-07-26 09:31:35

---



```java
public class CircleQueueDemo {

    public static void main(String[] args) {
        // 创建一个队列
        CircleQueue queue = new CircleQueue(4);// 约定最后一位为空，所以这里有效的是3位
        char key = ' ';//接收用户输入
        Scanner scanner = new Scanner(System.in);
        boolean loop =true;
        // 输出一个菜单
        while(loop){
            System.out.println("s(show):显示队列");
            System.out.println("e(exit):退出程序");
            System.out.println("a(add):添加数据到队列");
            System.out.println("g(get):从队列取出数据");
            System.out.println("h(head):查看队列头数据");
            key = scanner.next().charAt(0);
            switch (key){
                case 's':
                    queue.showQueue();
                    break;
                case 'a':
                    System.out.println("请输入一个数字：");
                    int value = scanner.nextInt();
                    queue.addQueue(value);
                    break;
                case 'g': // 取数据
                    try {
                        int res = queue.getQueue();
                        System.out.printf("取出的数据是%d\n",res);
                    } catch (Exception e){
                        System.out.println(e.getMessage());
                    }
                    break;
                case 'h': // 查看队列头数据
                    try {
                        int res = queue.headQueue();
                        System.out.printf("队列头的数据是%d\n",res);
                    }catch (Exception e){
                        System.out.println(e.getMessage());
                    }
                    break;
                case 'e': // 退出
                    scanner.close();
                    loop = false;
                    break;
                default:
                    break;
            }
        }
        System.out.println("程序退出");
    }

}

// 使用数组模拟队列 - 编写一个ArrayQueue类
class CircleQueue{
    private int maxSize; // 表示数组的最大容量
    private int front;   // 队列头
    private int rear;    // 队列尾
    private int[] arr;   // 该数组用于存放数据，模拟队列

    // 创建队列的构造器
    public CircleQueue(int maxSize) {
        this.maxSize = maxSize;
        arr = new int[maxSize];
    }

    // 判断队列是否满
    public boolean isFull(){
        return (rear+1) % maxSize == front;
    }

    // 判断队列是否为空
    public boolean isEmpty(){
        return rear == front;
    }

    // 添加数据到队列
    public void addQueue(int n){
        // 判断队列是否满
        if(isFull()){
            System.out.println("队列满，不能加入数据");
            return;
        }
        arr[rear] = n;
        rear = (rear+1)%maxSize;
    }

    // 获取队列的数据，出队列
    public int getQueue(){
        // 判断队列是否空
        if(isEmpty()){
            // 通过抛出异常来处理
            throw new RuntimeException("队列空，不能取数据");
        }
        // 这里需要分析出front是指向队列的第一个元素
        // 1. 先把front对应的值保存在一个临时变量
        // 2. 把front后移，考虑取模
        // 3. 把临时返回的变量返回
        int result = arr[front];
        front = (front+1)%maxSize;
        return result;
    }

    // 显示队列所有数据
    public void showQueue(){
        // 遍历
        if(isEmpty()){
            System.out.println("队列为空，没有数据！");
            return;
        }
        // 从front开始遍历，遍历多少个元素
        for (int i = front; i < front+size(); i++) {
            System.out.printf("arr[%d]=%d\n",i%maxSize,arr[i%maxSize]);
        }
    }

    // 求出当前队列的有效数据个数，否则无法遍历
    public int size(){
        // rear=1;
        // front=0;
        // maxSize=3
        return (rear+maxSize-front)%maxSize;
    }

    // 显示队列的头数据，注意不是取出数据，而是显示队头数据是什么
    public int headQueue(){
        // 判断
        if(isEmpty()){
            throw new RuntimeException("队列为空，没有数据！");
        }
        return arr[front];
    }

}
```








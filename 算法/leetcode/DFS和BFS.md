##  打印从1到最大的n位数

[剑指 Offer 17.](https://leetcode-cn.com/problems/da-yin-cong-1dao-zui-da-de-nwei-shu-lcof/)

**题目：**输入数字 n，按顺序打印出从 1 到最大的 n 位十进制数。比如输入 3，则打印出 1、2、3 一直到最大的 3 位数 999。当n很大时，你需要考虑到类型范围。

**示例 1:**

```
输入: n = 1
输出: [1,2,3,4,5,6,7,8,9]
```



**题解：**无论是 short / int / long ... 任意变量类型，数字的取值范围都是有限的。因此，大数的表示应用字符串 String 类型。

基于分治思想，先固定高位，向低位递归（向右递归）。当最低位被固定后，将字符串添加为答案里。比如说当n=5时，数字0对应的字符串是00000，为了把高位的0去掉，我们定义一个起始下标start表示字符串里以start为起始节点的字符串是答案之一。此外我们还定义一个变量nineCnt表示当前9数字出现的次数，当以start为起始的字符串里都是9时，则start--，比如说：

- 字符串00009，此时start=4，nineCnt=1，则start--以便之后取到两位数字（00010，00011等）。
- 字符串00099，此时start=3，nineCnt=2，则start--以便之后取到三位数字（00100，00101等）。

```java
class Solution {
    int len, start, curIndx = 0, nineCnt = 0;
    int[] ans;
    char[] num, loop = {'0', '1', '2', '3', '4', '5', '6', '7', '8', '9'};
    public int[] printNumbers(int n) {
        this.len = n;
        this.start = len-1;
        this.num = new char[len];
        this.ans = new int[(int)Math.pow(10, n) - 1];
        dfs(0);
        return ans;
    }

    public void dfs(int curDigit) {
        if(curDigit == len) {
            String str = String.valueOf(num).substring(start);
            if(!str.equals("0")) {
                ans[curIndx++] = Integer.parseInt(str);
            }
            if(len-start == nineCnt) {
                start--;
            }
            return;
        }

        for(char number : loop) {
            if(number == '9') {
                nineCnt++;
            }
            num[curDigit] = number;
            dfs(curDigit+1);
        }
        nineCnt--;
    }
}
```

**复杂度分析：**

- 时间复杂度 O(10^n)： 递归的生成的排列的数量为 10^n10 
- 空间复杂度 O(n)： 



## 课程表

[leetcode207](https://leetcode-cn.com/problems/course-schedule/)

**题目：**你这个学期必须选修 numCourse 门课程，记为 0 到 numCourse-1 。

在选修某些课程之前需要一些先修课程。 例如，想要学习课程 0 ，你需要先完成课程 1 ，我们用一个匹配来表示他们：[0,1]

给定课程总量以及它们的先决条件，请你判断是否可能完成所有课程的学习？

**示例 1:**

```
输入: 2, [[1,0]] 
输出: true
解释: 总共有 2 门课程。学习课程 1 之前，你需要完成课程 0。所以这是可能的。
```

**示例 2:**

```
输入: 2, [[1,0],[0,1]]
输出: false
解释: 总共有 2 门课程。学习课程 1 之前，你需要先完成课程 0；并且学习课程 0 之前，你还应先完成课程 1。这是不可能的。
```

**提示：**

输入的先决条件是由 边缘列表 表示的图形，而不是 邻接矩阵 。详情请参见图的表示法。
你可以假定输入的先决条件中没有重复的边。
1 <= numCourses <= 10^5



**题解：**用一个List<List<Integer>来表示完成课程i 后可选修的课程。每次对一个课程进行深度遍历：

- 如果flag[curIndx]= -1，表明该课程已在本层先前的遍历搜索到，构成了循环，返回false
- 如果flag[curIndx]= 1，表明课程curIndx的后续课程在其他层遍历中搜索到，可以确定不会构成循环，直接返回true
- 将flag[curIndx]置为1，表示该课程已在本层循环中查找到，然后遍历课程courseIndx的后续课程，查看是否构成循环



```java
class Solution {
    public boolean canFinish(int numCourses, int[][] prerequisites) {
        List<List<Integer>> advanceCourse = new ArrayList<>();
        for(int i = 0; i < numCourses; i++) {
            advanceCourse.add(new ArrayList<>());
        }
        int[] flag = new int[numCourses];
        //course[1]是course[0]的先修课程。记录选完course[1]后，可以选择的课程
        for(int[] course : prerequisites) {
            advanceCourse.get(course[1]).add(course[0]);
        }
        for(int i = 0; i < numCourses; i++) {
            if(!dfs(advanceCourse, flag, i)) {
                return false;
            }
        }
        return true;
    }

    public boolean dfs(List<List<Integer>> advanceCourse, int[] flag, int curIndx) {
        if(flag[curIndx] == -1) return false;
        if(flag[curIndx] == 1) return true;
        flag[curIndx] = -1;
        for(Integer nextCourseIndx : advanceCourse.get(curIndx)) {
            if(!dfs(advanceCourse, flag, nextCourseIndx)) {
                return false;
            }
        }
        flag[curIndx] = 1;
        return true;
    }
}
```



## 课程表II

[leetcode210](https://leetcode-cn.com/problems/course-schedule-ii/)

**题目：**现在你总共有 n 门课需要选，记为 0 到 n-1。

在选修某些课程之前需要一些先修课程。 例如，想要学习课程 0 ，你需要先完成课程 1 ，我们用一个匹配来表示他们: [0,1]

给定课程总量以及它们的先决条件，返回你为了学完所有课程所安排的学习顺序。

可能会有多个正确的顺序，你只要返回一种就可以了。如果不可能完成所有课程，返回一个空数组。

**示例 1:**

```
输入: 2, [[1,0]] 
输出: [0,1]
解释: 总共有 2 门课程。要学习课程 1，你需要先完成课程 0。因此，正确的课程顺序为 [0,1] 。
```


**示例 2:**

```
输入: 4, [[1,0],[2,0],[3,1],[3,2]]
输出: [0,1,2,3] or [0,2,1,3]
解释: 总共有 4 门课程。要学习课程 3，你应该先完成课程 1 和课程 2。并且课程 1 和课程 2 都应该排在课程 0 之后。
     因此，一个正确的课程顺序是 [0,1,2,3] 。另一个正确的排序是 [0,2,1,3] 。
```



**题解：**

```java
    public int[] findOrder(int numCourses, int[][] prerequisites) {
        
        //preCourse[i]表示i课程的先修课程数量
        int[] preCourse = new int[numCourses];
        for(int[] course : prerequisites) {
            preCourse[course[0]]++;
        }
        //queue存放没有先修课的课程
        Queue<Integer> queue = new LinkedList<>();
        for(int i = 0; i < numCourses; i++) {
            if(preCourse[i] == 0) {
                queue.add(i);
            }
        }
        int[] ans = new int[numCourses];
        int curIndx = 0;
        while(!queue.isEmpty()) {
            int curCourse = queue.poll();
            ans[curIndx++] = curCourse;
            //找到先修课是curCourse的课程（curCourse->course[0])
            //然后把course[0]的入度数减1，如果course[0]入度数为0，表示当前它的先修课程已遍历完，则加入队列中/
            for(int[] course : prerequisites) {
                if(course[1] == curCourse) {
                    preCourse[course[0]]--;
                    if(preCourse[course[0]] == 0) {
                        queue.add(course[0]);
                    }
                }
            }
        }
        return curIndx == numCourses ? ans : new int[0];
    }
```



## 被围绕的区域

[leetcode130](https://leetcode-cn.com/problems/surrounded-regions/)

**题目：**

给定一个二维的矩阵，包含 'X' 和 'O'（字母 O）。

找到所有被 'X' 围绕的区域，并将这些区域里所有的 'O' 用 'X' 填充。

**示例:**

```
X X X X
X O O X
X X O X
X O X X

运行你的函数后，矩阵变为：

X X X X
X X X X
X X X X
X O X X
```

**解释:**

被围绕的区间不会存在于边界上，换句话说，任何边界上的 'O' 都不会被填充为 'X'。 任何不在边界上，或不与边界上的 'O' 相连的 'O' 最终都会被填充为 'X'。如果两个元素在水平或垂直方向相邻，则称它们是“相连”的。



**题目：**

我们将矩阵划分为边界（第一行或最后一行或第一列或最后一列）和非边界。

对于边界上(i, j)可能出现的0，如果它与里面有联通，比如下图中的(3, 2)的上面可以和里面有联通，这时的0是不替换的，我们将这种情况的0换成#作为标志。遇到 O 替换为 X（和边界不连通的 O）；遇到 #，替换回 O(和边界连通的 O)。

```
X X X X
X O O X
X X O X
X O O X
```

```java
class Solution {
     public void solve(char[][] board) {
        if (board == null || board.length == 0) return;
        int rowLen = board.length;
        int colLen = board[0].length;
        for (int i = 0; i < rowLen; i++) {
            for (int j = 0; j < colLen; j++) {
                boolean isEdge = i == 0 || j == 0 || i == rowLen - 1 || j == colLen - 1;
                if (isEdge && board[i][j] == 'O') {
                    dfs(board, i, j);
                }
            }
        }

        for (int i = 0; i < rowLen; i++) {
            for (int j = 0; j < colLen; j++) {
                if (board[i][j] == 'O') {
                    board[i][j] = 'X';
                }
                if (board[i][j] == '#') {
                    board[i][j] = 'O';
                }
            }
        }
    }

    public void dfs(char[][] board, int i, int j) {
        if (i < 0 || j < 0 || i >= board.length || j >= board[0].length || board[i][j] != 'O') {
            return;
        }
        board[i][j] = '#';
        dfs(board, i - 1, j);
        dfs(board, i + 1, j); 
        dfs(board, i, j - 1); 
        dfs(board, i, j + 1); 
    }

}
```



## 01 矩阵

[leetcode 542](https://leetcode-cn.com/problems/01-matrix/)

**题目：**给定一个由 0 和 1 组成的矩阵，找出每个元素到最近的 0 的距离。

两个相邻元素间的距离为 1 。

**示例 1:**

输入:

```
0 0 0
0 1 0
0 0 0

```

输出:

```
0 0 0
0 1 0
0 0 0
```

**示例 2:**
输入:

```
0 0 0
0 1 0
1 1 1
```


输出:

```
0 0 0
0 1 0
1 2 1
```



**题解：**

首先我们通过一个队列来统计数值零的下标，并把非零数置为-1表示尚未访问过。原数组matrix将作为结果去返回。

遍历队列，此时队列里存储都是大于等于数值0的下标(x, y)，我们以这些下标为起点，上下左右探索到一个位置(newX, newY)，如果(newX, newY)的值是-1，则它到最近数值0的距离等于matrix\[x][y]+1。

```java
    public int[][] updateMatrix(int[][] matrix) {
        int row = matrix.length;
        int col = matrix[0].length;
        Queue<int[]> queue = new LinkedList<>();

        for(int i=0; i<row; i++) {
            for(int j=0; j<col; j++) {
                if(matrix[i][j] == 0) {
                    queue.offer(new int[]{i, j});
                }else {     //对于没有访问过非零数置为-1
                    matrix[i][j] = -1;
                }
            }
        }

        int[] mvX = new int[]{-1, 1, 0, 0};
        int[] mvY = new int[]{0, 0, -1, 1};

        while(!queue.isEmpty()) {
            int[] point = queue.poll();
            int x = point[0], y = point[1];
            //大于等于0的数(x, y)向四周延伸
            for(int i=0; i<4; i++) {
                int newX = x + mvX[i];
                int newY = y + mvY[i];
                //如果(newX, newY)没有被访问过，则它到最近0的距离等于matrix[x][y]+1
                if(newX>=0 && newX<row && newY>=0 && newY<col && matrix[newX][newY]==-1) {
                    matrix[newX][newY] = matrix[x][y] + 1;
                    queue.offer(new int[]{newX, newY});
                }
            }
        }

        return matrix;
    }
```



**时间复杂度**：O(n*m)